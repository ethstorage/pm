[TOC]

# Bolt Research

## Flow

https://chainbound.github.io/bolt-docs/flows/inclusion-flow

### bolt_requestInclusion

#### Structure

```rust
/// Request to include a transaction at a specific slot.
#[derive(Debug, Clone, Serialize, Deserialize, PartialEq, Eq)]
pub struct InclusionRequest {
    /// The consensus slot number at which the transaction should be included.
    pub slot: u64,
    /// The transaction to be included.
    pub txs: Vec<FullTransaction>,
    /// The signature over the "slot" and "tx" fields by the user.
    /// A valid signature is the only proof that the user actually requested
    /// this specific commitment to be included at the given slot.
    #[serde(skip)]
    pub signature: Option<Signature>,
    #[serde(skip)]
    pub signer: Option<Address>,
}
```

#### Validation

###### Consensus Validation

1. Check if the slot is in the current epoch.
2. If the request is for the next slot, check if it's within the commitment deadline. Default commitment deadline
   duration is 8 seconds.

```rust
   /// This function validates the state of the chain against a block. It checks 2 things:
    /// 1. The target slot is one of our proposer slots. (TODO)
    /// 2. The request hasn't passed the slot deadline.
    ///
    /// TODO: Integrate with the registry to check if we are registered.
    pub fn validate_request(&self, request: &CommitmentRequest) -> Result<u64, ConsensusError> {
        let CommitmentRequest::Inclusion(req) = request;

        // Check if the slot is in the current epoch
        if req.slot < self.epoch.start_slot || req.slot >= self.epoch.start_slot + SLOTS_PER_EPOCH {
            return Err(ConsensusError::InvalidSlot(req.slot));
        }

        // If the request is for the next slot, check if it's within the commitment deadline
        if req.slot == self.latest_slot + 1 &&
            self.latest_slot_timestamp + self.commitment_deadline_duration < Instant::now()
        {
            return Err(ConsensusError::DeadlineExceeded);
        }

        // Find the validator index for the given slot
        let validator_index = self.find_validator_index_for_slot(req.slot)?;

        Ok(validator_index)
    }
    
    /// Default commitment deadline duration.
    ///
    /// The sidecar will stop accepting new commitments for the next block
    /// after this deadline has passed. This is to ensure that builders and
    /// relays have enough time to build valid payloads.
    pub const DEFAULT_COMMITMENT_DEADLINE_IN_MILLIS: u64 = 8_000;    
```

###### Execution Validation

1. Validate the chain ID
2. Check if there is room for more commitments, default maximum number of commitments is 128.
3. Check if the committed gas exceeds the maximum gas limit(accumulate), default maximum gas limit is 10_000_000.
4. Check if the transaction size exceeds the maximum, default maximum transaction size is 4 * 32 * 1024 bytes.
5. Check if the transaction is a contract creation and the init code size exceeds the maximum, default maximum init code
   size is 2 * 24576 bytes.
6. Check if the gas limit is higher than the maximum block gas limit, default maximum block gas limit is 30_000_000.(
   seems no need)
7. Ensure max_priority_fee_per_gas is less than max_fee_per_gas
8. https://www.blocknative.com/blog/eip-1559-fees, Check if the max_fee_per_gas would cover the maximum possible
   basefee (target slot - current slot)
9.     // Validate each transaction in the request against the account state,
       // keeping track of the nonce and balance diffs, including:
       // - any existing state in the account trie
       // - any previously committed transactions
       // - any previous transaction in the same request
       //
       // NOTE: it's also possible for a request to contain multiple transactions
       // from different senders, in this case each sender will have its own nonce
       // and balance diffs that will be applied to the account state.
10. Check blob count, default maximum blob count is 786432 / 131072 = 6.
11. max possible increase in blob basefee, similar with 8.
12. Validate blob against KZG settings

```rust
    /// Validates the commitment request against state (historical + intermediate).
    ///
    /// NOTE: This function only simulates against execution state, it does not consider
    /// timing or proposer slot targets.
    ///
    /// If the commitment is invalid because of nonce, basefee or balance errors, it will return an
    /// error. If the commitment is valid, its account state
    /// will be cached. If this is succesful, any callers can be sure that the commitment is valid
    /// and SHOULD sign it and respond to the requester.
    ///
    /// TODO: should also validate everything in https://github.com/paradigmxyz/reth/blob/9aa44e1a90b262c472b14cd4df53264c649befc2/crates/transaction-pool/src/validate/eth.rs#L153
    pub async fn validate_request(
        &mut self,
        request: &mut CommitmentRequest,
    ) -> Result<(), ValidationError> {
        let CommitmentRequest::Inclusion(req) = request;

        let signer = req.signer().expect("Set signer");
        req.recover_signers()?;

        let target_slot = req.slot;

        // Validate the chain ID
        if !req.validate_chain_id(self.chain_id) {
            return Err(ValidationError::ChainIdMismatch);
        }

        // Check if there is room for more commitments
        if let Some(template) = self.get_block_template(target_slot) {
            if template.transactions_len() >= self.limits.max_commitments_per_slot.get() {
                return Err(ValidationError::MaxCommitmentsReachedForSlot(
                    self.slot,
                    self.limits.max_commitments_per_slot.get(),
                ));
            }
        }

        // Check if the committed gas exceeds the maximum
        let template_committed_gas =
            self.get_block_template(target_slot).map(|t| t.committed_gas()).unwrap_or(0);

        if template_committed_gas + req.gas_limit() >= self.limits.max_committed_gas_per_slot.get()
        {
            return Err(ValidationError::MaxCommittedGasReachedForSlot(
                self.slot,
                self.limits.max_committed_gas_per_slot.get(),
            ));
        }

        // Check if the transaction size exceeds the maximum
        if !req.validate_tx_size_limit(self.validation_params.max_tx_input_bytes) {
            return Err(ValidationError::TransactionSizeTooHigh);
        }

        // Check if the transaction is a contract creation and the init code size exceeds the
        // maximum
        if !req.validate_init_code_limit(self.validation_params.max_init_code_byte_size) {
            return Err(ValidationError::TransactionSizeTooHigh);
        }

        // Check if the gas limit is higher than the maximum block gas limit
        if req.gas_limit() > self.validation_params.block_gas_limit {
            return Err(ValidationError::GasLimitTooHigh);
        }

        // Ensure max_priority_fee_per_gas is less than max_fee_per_gas
        if !req.validate_priority_fee() {
            return Err(ValidationError::MaxPriorityFeePerGasTooHigh);
        }

        // Check if the max_fee_per_gas would cover the maximum possible basefee.
        let slot_diff = target_slot.saturating_sub(self.slot);

        // Calculate the max possible basefee given the slot diff
        let max_basefee = calculate_max_basefee(self.basefee, slot_diff)
            .ok_or(ValidationError::MaxBaseFeeCalcOverflow)?;

        debug!(%slot_diff, basefee = self.basefee, %max_basefee, "Validating basefee");

        // Validate the base fee
        if !req.validate_basefee(max_basefee) {
            return Err(ValidationError::BaseFeeTooLow(max_basefee));
        }

        if target_slot < self.slot {
            debug!(%target_slot, %self.slot, "Target slot lower than current slot");
            return Err(ValidationError::SlotTooLow(self.slot));
        }

        // Validate each transaction in the request against the account state,
        // keeping track of the nonce and balance diffs, including:
        // - any existing state in the account trie
        // - any previously committed transactions
        // - any previous transaction in the same request
        //
        // NOTE: it's also possible for a request to contain multiple transactions
        // from different senders, in this case each sender will have its own nonce
        // and balance diffs that will be applied to the account state.
        let mut bundle_nonce_diff_map = HashMap::new();
        let mut bundle_balance_diff_map = HashMap::new();
        for tx in req.txs.iter() {
            let sender = tx.sender().expect("Recovered sender");

            // From previous preconfirmations requests retrieve
            // - the nonce difference from the account state.
            // - the balance difference from the account state.
            // - the highest slot number for which the user has requested a preconfirmation.
            //
            // If the templates do not exist, or this is the first request for this sender,
            // its diffs will be zero.
            let (nonce_diff, balance_diff, highest_slot_for_account) =
                self.block_templates.iter().fold(
                    (0, U256::ZERO, 0),
                    |(nonce_diff_acc, balance_diff_acc, highest_slot), (slot, block_template)| {
                        let (nonce_diff, balance_diff, slot) = block_template
                            .get_diff(sender)
                            .map(|(nonce, balance)| (nonce, balance, *slot))
                            .unwrap_or((0, U256::ZERO, 0));

                        (
                            nonce_diff_acc + nonce_diff,
                            balance_diff_acc.saturating_add(balance_diff),
                            u64::max(highest_slot, slot),
                        )
                    },
                );

            if target_slot < highest_slot_for_account {
                debug!(%target_slot, %highest_slot_for_account, "There is a request for a higher slot");
                return Err(ValidationError::SlotTooLow(highest_slot_for_account));
            }

            trace!(?signer, nonce_diff, %balance_diff, "Applying diffs to account state");

            let account_state = match self.account_state(sender).copied() {
                Some(account) => account,
                None => {
                    // Fetch the account state from the client if it does not exist
                    let account = match self.client.get_account_state(sender, None).await {
                        Ok(account) => account,
                        Err(err) => {
                            return Err(ValidationError::Internal(format!(
                                "Error fetching account state: {:?}",
                                err
                            )))
                        }
                    };

                    self.account_states.insert(*sender, account);
                    account
                }
            };

            debug!(?account_state, ?nonce_diff, ?balance_diff, "Validating transaction");

            let sender_nonce_diff = bundle_nonce_diff_map.entry(sender).or_insert(0);
            let sender_balance_diff = bundle_balance_diff_map.entry(sender).or_insert(U256::ZERO);

            // Apply the diffs to this account according to the info fetched from the templates
            // and the current bundle diffs for this sender.
            let account_state_with_diffs = AccountState {
                transaction_count: account_state
                    .transaction_count
                    .saturating_add(nonce_diff)
                    .saturating_add(*sender_nonce_diff),

                balance: account_state
                    .balance
                    .saturating_sub(balance_diff)
                    .saturating_sub(*sender_balance_diff),

                has_code: account_state.has_code,
            };

            // Validate the transaction against the account state with existing diffs
            validate_transaction(&account_state_with_diffs, tx)?;

            // Check EIP-4844-specific limits
            if let Some(transaction) = tx.as_eip4844() {
                if let Some(template) = self.block_templates.get(&target_slot) {
                    if template.blob_count() >= MAX_BLOBS_PER_BLOCK {
                        return Err(ValidationError::Eip4844Limit);
                    }
                }

                let PooledTransactionsElement::BlobTransaction(ref blob_transaction) = tx.deref()
                else {
                    unreachable!("EIP-4844 transaction should be a blob transaction")
                };

                // Calculate max possible increase in blob basefee
                let max_blob_basefee = calculate_max_basefee(self.blob_basefee, slot_diff)
                    .ok_or(ValidationError::MaxBaseFeeCalcOverflow)?;

                debug!(%max_blob_basefee, blob_basefee = blob_transaction.transaction.max_fee_per_blob_gas, "Validating blob basefee");
                if blob_transaction.transaction.max_fee_per_blob_gas < max_blob_basefee {
                    return Err(ValidationError::BlobBaseFeeTooLow(max_blob_basefee));
                }

                // Validate blob against KZG settings
                transaction.validate_blob(&blob_transaction.sidecar, self.kzg_settings.get())?;
            }

            // Increase the bundle nonce and balance diffs for this sender for the next iteration
            *sender_nonce_diff += 1;
            *sender_balance_diff += max_transaction_cost(tx);
        }

        Ok(())
    }
```

##### Request to Constraints

```rust
/// A message that contains the constraints that need to be signed by the proposer sidecar.
///
/// Reference: https://chainbound.github.io/bolt-docs/api/builder#constraints
#[derive(Serialize, Deserialize, Debug, Clone, PartialEq, Default)]
pub struct ConstraintsMessage {
    /// The validator index of the proposer sidecar.
    pub validator_index: u64,
    /// The consensus slot at which the constraints are valid
    pub slot: u64,
    /// Indicates whether these constraints are only valid on the top of the block.
    /// NOTE: Per slot, only 1 top-of-block bundle is valid.
    pub top: bool,
    /// The constraints that need to be signed.
    #[serde(deserialize_with = "deserialize_txs", serialize_with = "serialize_txs")]
    pub constraints: Vec<FullTransaction>,
}
```

##### Update state diff and store constraints

```rust
/// A block template that serves as a fallback block, but is also used
/// to keep intermediary state for new commitment requests.
///
/// # Roles
/// - Fallback block template.
/// - Intermediary state for new commitment requests.
/// - Simulate new commitment requests.
/// - Update state every block, to invalidate old commitments.
/// - Make sure we DO NOT accept invalid commitments in any circumstances.
#[derive(Debug, Default)]
pub struct BlockTemplate {
    /// The state diffs per address given the list of commitments.
    pub(crate) state_diff: StateDiff,
    /// The signed constraints associated to the block
    pub signed_constraints_list: Vec<SignedConstraints>,
}
```


### /eth/v1/builder/constraints

[mev-boost](https://flashbots.notion.site/Running-MEV-Boost-Relay-at-scale-4040ccd5186c425d9a860cbb29bbfe09#23defbc17b0f40369957fa4836a313ba)

Provided by the modified mev-boost which is a proxy to mev-boost-relay, cache the constraints and forward to all the
relays by same /eth/v1/builder/constraints endpoint.
The constraints will be broadcast to all the builders who have subscribed.

#### /relay/v1/builder/constraints

Builders subscribe to this endpoint to get the constraints through SSE.

### /eth/v1/events?topics=payload_attributes(trigger block building) 
Builder create block according to the constraints. Constraints have index or not.
```go
// commitTransactions applies sorted transactions to the current environment, updating the state
// and creating the resulting block
//
// Assumptions:
//   - there are no nonce-conflicting transactions between `plainTxs`, `blobTxs` and the constraints
//   - all transaction are correctly signed
func (w *worker) commitTransactions(env *environment, plainTxs, blobTxs *transactionsByPriceAndNonce, constraints types.HashToConstraintDecoded, interrupt *atomic.Int32) error {
	gasLimit := env.header.GasLimit
	if env.gasPool == nil {
		env.gasPool = new(core.GasPool).AddGas(gasLimit)
	}
	var coalescedLogs []*types.Log

	// Here we initialize and track the constraints left to be executed along
	// with their gas requirements
	constraintsOrderedByIndex,
		constraintsWithoutIndex,
		constraintsTotalGasLeft,
		constraintsTotalBlobGasLeft := types.ParseConstraintsDecoded(constraints)

	for {
		// `env.tcount` starts from 0 so it's correct to use it as the current index
		currentTxIndex := uint64(env.tcount)

		// Check interruption signal and abort building if it's fired.
		if interrupt != nil {
			if signal := interrupt.Load(); signal != commitInterruptNone {
				return signalToErr(signal)
			}
		}
		// If we don't have enough gas for any further transactions then we're done.
		if env.gasPool.Gas() < params.TxGas {
			log.Trace("Not enough gas for further transactions", "have", env.gasPool, "want", params.TxGas)
			break
		}

		blobGasLeft := uint64(params.MaxBlobGasPerBlock - env.blobs*params.BlobTxBlobGasPerBlob)

		// If we don't have enough blob space for any further blob transactions,
		// skip that list altogether
		if !blobTxs.Empty() && blobGasLeft <= 0 {
			log.Trace("Not enough blob space for further blob transactions")
			blobTxs.Clear()
			// Fall though to pick up any plain txs
		}
		// Retrieve the next transaction and abort if all done.
		var (
			lazyTx      *txpool.LazyTransaction
			txs         *transactionsByPriceAndNonce
			plainLazyTx *txpool.LazyTransaction
			plainTxTip  *uint256.Int
			blobLazyTx  *txpool.LazyTransaction
			blobTxTip   *uint256.Int
		)

		if pTxWithMinerFee := plainTxs.Peek(); pTxWithMinerFee != nil {
			plainLazyTx = pTxWithMinerFee.Tx()
			plainTxTip = pTxWithMinerFee.fees
		}

		if bTxWithMinerFee := blobTxs.Peek(); bTxWithMinerFee != nil {
			blobLazyTx = bTxWithMinerFee.Tx()
			blobTxTip = bTxWithMinerFee.fees
		}

		switch {
		case plainLazyTx == nil:
			txs, lazyTx = blobTxs, blobLazyTx
		case blobLazyTx == nil:
			txs, lazyTx = plainTxs, plainLazyTx
		default:
			if plainTxTip.Lt(blobTxTip) {
				txs, lazyTx = blobTxs, blobLazyTx
			} else {
				txs, lazyTx = plainTxs, plainLazyTx
			}
		}

		type candidateTx struct {
			tx           *types.Transaction
			isConstraint bool
		}
		// candidate is the transaction we should execute in this cycle of the loop
		var candidate struct {
			tx           *types.Transaction
			isConstraint bool
		}

		var constraintTx *types.ConstraintDecoded
		if len(constraintsOrderedByIndex) > 0 {
			constraintTx = constraintsOrderedByIndex[0]
		}

		isSomePoolTxLeft := lazyTx != nil

		isThereConstraintWithThisIndex := constraintTx != nil && constraintTx.Index != nil && *constraintTx.Index == currentTxIndex
		if isThereConstraintWithThisIndex {
			// we retrieve the candidate constraint by shifting it from the list
			candidate = candidateTx{tx: common.Shift(&constraintsOrderedByIndex).Tx, isConstraint: true}
		} else {
			if isSomePoolTxLeft {
				// Check if there enough gas left for this tx
				if constraintsTotalGasLeft+lazyTx.Gas > env.gasPool.Gas() || constraintsTotalBlobGasLeft+lazyTx.BlobGas > blobGasLeft {
					// Skip this tx and try to fit one with less gas.
					// Drop all consecutive transactions from the same sender because of `nonce-too-high` clause.
					log.Debug("Could not find transactions gas with the remaining constraints, account skipped", "hash", lazyTx.Hash)
					txs.Pop()
					// Edge case:
					//
					// Assumption: suppose sender A sends tx T_1 with nonce 1, and T_2 with nonce 2, and T_2 is a constraint.
					//
					//
					// When running the block building algorithm I first have to make sure to reserve enough gas for the constraints.
					// This implies that when a pooled tx comes I have to check if there is enough gas for it while taking into account
					// the rest of the remaining constraint gas to allocate.
					// Suppose there is no gas for the pooled tx T_1, then I have to drop it and consequently drop every tx from the same
					// sender with higher nonce due to "nonce-too-high" issues, including T_2.
					// But then, I have dropped a constraint which means my bid is invalid.
					//
					// FIXME: for the PoC we're not handling this

					// Repeat the loop to try to find another pool transaction
					continue
				}
				// We can safely consider the pool tx as the candidate,
				// since by assumption it is not nonce-conflicting
				tx := lazyTx.Resolve()
				if tx == nil {
					log.Trace("Ignoring evicted transaction", "hash", candidate.tx.Hash())
					txs.Pop()
					continue
				}
				candidate = candidateTx{tx: tx, isConstraint: false}
			} else {
				// No more pool tx left, we can add the unindexed ones if available
				if len(constraintsWithoutIndex) == 0 {
					// To recap, this means:
					// 1. there are no more pool tx left
					// 2. there are no more constraints without an index
					// 3. the remaining indexes inside `constraintsOrderedByIndex`, if any, cannot be satisfied
					// As such, we can safely exist
					break
				}
				candidate = candidateTx{tx: common.Shift(&constraintsWithoutIndex).Tx, isConstraint: true}
			}
		}

		// Error may be ignored here, see assumption
		from, _ := types.Sender(env.signer, candidate.tx)

		// Check whether the tx is replay protected. If we're not in the EIP155 hf
		// phase, start ignoring the sender until we do.
		if candidate.tx.Protected() && !w.chainConfig.IsEIP155(env.header.Number) {
			log.Trace("Ignoring replay protected transaction", "hash", candidate.tx.Hash(), "eip155", w.chainConfig.EIP155Block)
			txs.Pop()
			continue
		}
		// Start executing the transaction
		env.state.SetTxContext(candidate.tx.Hash(), env.tcount)

		logs, err := w.commitTransaction(env, candidate.tx)
		switch {
		case errors.Is(err, core.ErrNonceTooLow):
			// New head notification data race between the transaction pool and miner, shift
			log.Trace("Skipping transaction with low nonce", "hash", candidate.tx.Hash(), "sender", from, "nonce", candidate.tx.Nonce())
			if candidate.isConstraint {
				log.Warn(fmt.Sprintf("Skipping constraint with low nonce, hash %s, sender %s, nonce %d", candidate.tx.Hash(), from, candidate.tx.Nonce()))
			} else {
				txs.Shift()
			}

		case errors.Is(err, nil):
			// Everything ok, collect the logs and shift in the next transaction from the same account
			coalescedLogs = append(coalescedLogs, logs...)
			env.tcount++
			if candidate.isConstraint {
				// Update the amount of gas left for the constraints
				constraintsTotalGasLeft -= candidate.tx.Gas()
				constraintsTotalBlobGasLeft -= candidate.tx.BlobGas()

				constraintTip, _ := candidate.tx.EffectiveGasTip(env.header.BaseFee)
				log.Info(fmt.Sprintf("Executed constraint %s at index %d with effective gas tip %d", candidate.tx.Hash().String(), currentTxIndex, constraintTip))
			} else {
				txs.Shift()
			}

		default:
			// Transaction is regarded as invalid, drop all consecutive transactions from
			// the same sender because of `nonce-too-high` clause.
			log.Debug("Transaction failed, account skipped", "hash", candidate.tx.Hash(), "err", err)
			if candidate.isConstraint {
				log.Warn("Constraint failed, account skipped", "hash", candidate.tx.Hash(), "err", err)
			} else {
				txs.Pop()
			}
		}
	}
	if !w.isRunning() && len(coalescedLogs) > 0 {
		// We don't push the pendingLogsEvent while we are sealing. The reason is that
		// when we are sealing, the worker will regenerate a sealing block every 3 seconds.
		// In order to avoid pushing the repeated pendingLog, we disable the pending log pushing.

		// make a copy, the state caches the logs and these logs get "upgraded" from pending to mined
		// logs by filling in the block hash when the block was mined by the local miner. This can
		// cause a race condition if a log was "upgraded" before the PendingLogsEvent is processed.
		cpy := make([]*types.Log, len(coalescedLogs))
		for i, l := range coalescedLogs {
			cpy[i] = new(types.Log)
			*cpy[i] = *l
		}
		w.pendingLogsFeed.Send(cpy)
	}
	return nil
}
```
### /relay/v1/builder/blocks_with_proofs
Submit the block with inclusion proofs(merkel proof) to the relay. Relay checks the submission is valid or not and save the db.

```go
func (api *RelayAPI) handleSubmitNewBlockWithProofs(w http.ResponseWriter, req *http.Request) {
	var pf common.Profile
	var prevTime, nextTime time.Time

	headSlot := api.headSlot.Load()
	receivedAt := time.Now().UTC()
	prevTime = receivedAt

	args := req.URL.Query()
	isCancellationEnabled := args.Get("cancellations") == "1"

	log := api.log.WithFields(logrus.Fields{
		"method":                "submitNewBlockWithPreconfs",
		"contentLength":         req.ContentLength,
		"headSlot":              headSlot,
		"cancellationEnabled":   isCancellationEnabled,
		"timestampRequestStart": receivedAt.UnixMilli(),
	})

	// Log at start and end of request
	log.Info("request initiated")
	defer func() {
		log.WithFields(logrus.Fields{
			"timestampRequestFin": time.Now().UTC().UnixMilli(),
			"requestDurationMs":   time.Since(receivedAt).Milliseconds(),
		}).Info("request finished")
	}()

	// If cancellations are disabled but builder requested it, return error
	if isCancellationEnabled && !api.ffEnableCancellations {
		log.Info("builder submitted with cancellations enabled, but feature flag is disabled")
		api.RespondError(w, http.StatusBadRequest, "cancellations are disabled")
		return
	}

	var err error
	var reader io.Reader = req.Body
	isGzip := req.Header.Get("Content-Encoding") == "gzip"
	log = log.WithField("reqIsGzip", isGzip)
	if isGzip {
		reader, err = gzip.NewReader(req.Body)
		if err != nil {
			log.WithError(err).Warn("could not create gzip reader")
			api.RespondError(w, http.StatusBadRequest, err.Error())
			return
		}
	}

	limitReader := io.LimitReader(reader, 10*1024*1024) // 10 MB
	requestPayloadBytes, err := io.ReadAll(limitReader)
	if err != nil {
		log.WithError(err).Warn("could not read payload")
		api.RespondError(w, http.StatusBadRequest, err.Error())
		return
	}

	nextTime = time.Now().UTC()
	pf.PayloadLoad = uint64(nextTime.Sub(prevTime).Microseconds())
	prevTime = nextTime

	// BOLT: new payload type
	payload := new(common.VersionedSubmitBlockRequestWithProofs)

	// Check for SSZ encoding
	contentType := req.Header.Get("Content-Type")
	if contentType == "application/octet-stream" {
		// TODO: (BOLT) implement SSZ decoding
		panic("SSZ decoding not implemented for preconfs yet")
	} else {
		log = log.WithField("reqContentType", "json")
		if err := json.Unmarshal(requestPayloadBytes, payload); err != nil {
			api.boltLog.WithError(err).Warn("Could not decode payload - JSON")
			api.RespondError(w, http.StatusBadRequest, err.Error())
			return
		}
	}

	log.Infof("Received block bid with proofs from builder: %s", payload)

	// BOLT: Send an event to the web demo
	slot, _ := payload.Inner.Slot()
	message := fmt.Sprintf("received block bid with %d preconfirmations for slot %d", len(payload.Proofs.TransactionHashes), slot)
	EmitBoltDemoEvent(message)

	nextTime = time.Now().UTC()
	pf.Decode = uint64(nextTime.Sub(prevTime).Microseconds())
	prevTime = nextTime

	isLargeRequest := len(requestPayloadBytes) > fastTrackPayloadSizeLimit
	// getting block submission info also validates bid trace and execution submission are not empty
	submission, err := common.GetBlockSubmissionInfo(payload.Inner)
	if err != nil {
		log.WithError(err).Warn("missing fields in submit block request")
		api.RespondError(w, http.StatusBadRequest, err.Error())
		return
	}
	log = log.WithFields(logrus.Fields{
		"timestampAfterDecoding": time.Now().UTC().UnixMilli(),
		"slot":                   submission.BidTrace.Slot,
		"builderPubkey":          submission.BidTrace.BuilderPubkey.String(),
		"blockHash":              submission.BidTrace.BlockHash.String(),
		"proposerPubkey":         submission.BidTrace.ProposerPubkey.String(),
		"parentHash":             submission.BidTrace.ParentHash.String(),
		"value":                  submission.BidTrace.Value.Dec(),
		"numTx":                  len(submission.Transactions),
		"payloadBytes":           len(requestPayloadBytes),
		"isLargeRequest":         isLargeRequest,
	})
	// deneb specific logging
	if payload.Inner.Deneb != nil {
		log = log.WithFields(logrus.Fields{
			"numBlobs":      len(payload.Inner.Deneb.BlobsBundle.Blobs),
			"blobGasUsed":   payload.Inner.Deneb.ExecutionPayload.BlobGasUsed,
			"excessBlobGas": payload.Inner.Deneb.ExecutionPayload.ExcessBlobGas,
		})
	}

	ok := api.checkSubmissionSlotDetails(w, log, headSlot, payload.Inner, submission)
	if !ok {
		return
	}

	builderPubkey := submission.BidTrace.BuilderPubkey
	builderEntry, ok := api.checkBuilderEntry(w, log, builderPubkey)
	if !ok {
		return
	}

	log = log.WithField("builderIsHighPrio", builderEntry.status.IsHighPrio)

	gasLimit, ok := api.checkSubmissionFeeRecipient(w, log, submission.BidTrace)
	if !ok {
		return
	}

	// Don't accept blocks with 0 value
	if submission.BidTrace.Value.ToBig().Cmp(ZeroU256.BigInt()) == 0 || len(submission.Transactions) == 0 {
		log.Info("submitNewBlock failed: block with 0 value or no txs")
		w.WriteHeader(http.StatusOK)
		return
	}

	// Sanity check the submission
	err = SanityCheckBuilderBlockSubmission(payload.Inner)
	if err != nil {
		log.WithError(err).Info("block submission sanity checks failed")
		api.RespondError(w, http.StatusBadRequest, err.Error())
		return
	}

	attrs, ok := api.checkSubmissionPayloadAttrs(w, log, submission)
	if !ok {
		return
	}

	// Verify the signature
	log = log.WithField("timestampBeforeSignatureCheck", time.Now().UTC().UnixMilli())
	signature := submission.Signature
	ok, err = ssz.VerifySignature(submission.BidTrace, api.opts.EthNetDetails.DomainBuilder, builderPubkey[:], signature[:])
	log = log.WithField("timestampAfterSignatureCheck", time.Now().UTC().UnixMilli())
	if err != nil {
		log.WithError(err).Warn("failed verifying builder signature")
		api.RespondError(w, http.StatusBadRequest, "failed verifying builder signature")
		return
	} else if !ok {
		log.Warn("invalid builder signature")
		api.RespondError(w, http.StatusBadRequest, "invalid signature")
		return
	}

	log = log.WithField("timestampBeforeCheckingFloorBid", time.Now().UTC().UnixMilli())

	// Create the redis pipeline tx
	tx := api.redis.NewTxPipeline()

	// channel to send simulation result to the deferred function
	simResultC := make(chan *blockSimResult, 1)
	var eligibleAt time.Time // will be set once the bid is ready

	submission, err = common.GetBlockSubmissionInfo(payload.Inner)
	if err != nil {
		log.WithError(err).Warn("missing fields in submit block request")
		api.RespondError(w, http.StatusBadRequest, err.Error())
		return
	}

	bfOpts := bidFloorOpts{
		w:                    w,
		tx:                   tx,
		log:                  log,
		cancellationsEnabled: isCancellationEnabled,
		simResultC:           simResultC,
		submission:           submission,
	}
	floorBidValue, ok := api.checkFloorBidValue(bfOpts)
	if !ok {
		return
	}

	log = log.WithField("timestampAfterCheckingFloorBid", time.Now().UTC().UnixMilli())

	// Deferred saving of the builder submission to database (whenever this function ends)
	defer func() {
		savePayloadToDatabase := !api.ffDisablePayloadDBStorage
		var simResult *blockSimResult
		select {
		case simResult = <-simResultC:
		case <-time.After(10 * time.Second):
			log.Warn("timed out waiting for simulation result")
			simResult = &blockSimResult{false, false, nil, nil}
		}

		submissionEntry, err := api.db.SaveBuilderBlockSubmission(
			payload.Inner,
			simResult.requestErr,
			simResult.validationErr,
			receivedAt,
			eligibleAt,
			simResult.wasSimulated,
			savePayloadToDatabase,
			pf,
			simResult.optimisticSubmission,
			payload.Proofs, // BOLT: add merkle proofs to the submission
		)
		if err != nil {
			log.WithError(err).WithField("payload", payload).Error("saving builder block submission to database failed")
			return
		}

		err = api.db.UpsertBlockBuilderEntryAfterSubmission(submissionEntry, simResult.validationErr != nil)
		if err != nil {
			log.WithError(err).Error("failed to upsert block-builder-entry")
		}
	}()

	// ---------------------------------
	// THE BID WILL BE SIMULATED SHORTLY
	// ---------------------------------

	log = log.WithField("timestampBeforeCheckingTopBid", time.Now().UTC().UnixMilli())

	// Get the latest top bid value from Redis
	bidIsTopBid := false
	topBidValue, err := api.redis.GetTopBidValue(context.Background(), tx, submission.BidTrace.Slot, submission.BidTrace.ParentHash.String(), submission.BidTrace.ProposerPubkey.String())
	if err != nil {
		log.WithError(err).Error("failed to get top bid value from redis")
	} else {
		bidIsTopBid = submission.BidTrace.Value.ToBig().Cmp(topBidValue) == 1
		log = log.WithFields(logrus.Fields{
			"topBidValue":    topBidValue.String(),
			"newBidIsTopBid": bidIsTopBid,
		})
	}

	log = log.WithField("timestampAfterCheckingTopBid", time.Now().UTC().UnixMilli())

	nextTime = time.Now().UTC()
	pf.Prechecks = uint64(nextTime.Sub(prevTime).Microseconds())
	prevTime = nextTime

	// Simulate the block submission and save to db
	fastTrackValidation := builderEntry.status.IsHighPrio && bidIsTopBid && !isLargeRequest
	timeBeforeValidation := time.Now().UTC()

	log = log.WithFields(logrus.Fields{
		"timestampBeforeValidation": timeBeforeValidation.UTC().UnixMilli(),
		"fastTrackValidation":       fastTrackValidation,
	})

	// Construct simulation request
	opts := blockSimOptions{
		isHighPrio: builderEntry.status.IsHighPrio,
		fastTrack:  fastTrackValidation,
		log:        log,
		builder:    builderEntry,
		req: &common.BuilderBlockValidationRequest{
			VersionedSubmitBlockRequest: payload.Inner,
			RegisteredGasLimit:          gasLimit,
			ParentBeaconBlockRoot:       attrs.parentBeaconRoot,
		},
	}
	// With sufficient collateral, process the block optimistically.
	if builderEntry.status.IsOptimistic &&
		builderEntry.collateral.Cmp(submission.BidTrace.Value.ToBig()) >= 0 &&
		submission.BidTrace.Slot == api.optimisticSlot.Load() {
		go api.processOptimisticBlock(opts, simResultC)
	} else {
		// Simulate block (synchronously).
		requestErr, validationErr := api.simulateBlock(context.Background(), opts) // success/error logging happens inside
		simResultC <- &blockSimResult{requestErr == nil, false, requestErr, validationErr}
		validationDurationMs := time.Since(timeBeforeValidation).Milliseconds()
		log = log.WithFields(logrus.Fields{
			"timestampAfterValidation": time.Now().UTC().UnixMilli(),
			"validationDurationMs":     validationDurationMs,
		})
		if requestErr != nil { // Request error
			if os.IsTimeout(requestErr) {
				api.RespondError(w, http.StatusGatewayTimeout, "validation request timeout")
			} else {
				api.RespondError(w, http.StatusBadRequest, requestErr.Error())
			}
			return
		} else {
			if validationErr != nil {
				api.RespondError(w, http.StatusBadRequest, validationErr.Error())
				return
			}
		}
	}

	nextTime = time.Now().UTC()
	pf.Simulation = uint64(nextTime.Sub(prevTime).Microseconds())
	prevTime = nextTime

	// If cancellations are enabled, then abort now if this submission is not the latest one
	if isCancellationEnabled {
		// Ensure this request is still the latest one. This logic intentionally ignores the value of the bids and makes the current active bid the one
		// that arrived at the relay last. This allows for builders to reduce the value of their bid (effectively cancel a high bid) by ensuring a lower
		// bid arrives later. Even if the higher bid takes longer to simulate, by checking the receivedAt timestamp, this logic ensures that the low bid
		// is not overwritten by the high bid.
		//
		// NOTE: this can lead to a rather tricky race condition. If a builder submits two blocks to the relay concurrently, then the randomness of network
		// latency will make it impossible to predict which arrives first. Thus a high bid could unintentionally be overwritten by a low bid that happened
		// to arrive a few microseconds later. If builders are submitting blocks at a frequency where they cannot reliably predict which bid will arrive at
		// the relay first, they should instead use multiple pubkeys to avoid uninitentionally overwriting their own bids.
		latestPayloadReceivedAt, err := api.redis.GetBuilderLatestPayloadReceivedAt(context.Background(), tx, submission.BidTrace.Slot, submission.BidTrace.BuilderPubkey.String(), submission.BidTrace.ParentHash.String(), submission.BidTrace.ProposerPubkey.String())
		if err != nil {
			log.WithError(err).Error("failed getting latest payload receivedAt from redis")
		} else if receivedAt.UnixMilli() < latestPayloadReceivedAt {
			log.Infof("already have a newer payload: now=%d / prev=%d", receivedAt.UnixMilli(), latestPayloadReceivedAt)
			api.RespondError(w, http.StatusBadRequest, "already using a newer payload")
			return
		}
	}
	redisOpts := redisUpdateBidOpts{
		w:                    w,
		tx:                   tx,
		log:                  log,
		cancellationsEnabled: isCancellationEnabled,
		receivedAt:           receivedAt,
		floorBidValue:        floorBidValue,
		payload:              payload.Inner,
	}
	updateBidResult, getPayloadResponse, ok := api.updateRedisBidWithProofs(redisOpts, payload.Proofs)
	if !ok {
		return
	}

	// Add fields to logs
	log = log.WithFields(logrus.Fields{
		"timestampAfterBidUpdate":    time.Now().UTC().UnixMilli(),
		"wasBidSavedInRedis":         updateBidResult.WasBidSaved,
		"wasTopBidUpdated":           updateBidResult.WasTopBidUpdated,
		"topBidValue":                updateBidResult.TopBidValue,
		"prevTopBidValue":            updateBidResult.PrevTopBidValue,
		"profileRedisSavePayloadUs":  updateBidResult.TimeSavePayload.Microseconds(),
		"profileRedisUpdateTopBidUs": updateBidResult.TimeUpdateTopBid.Microseconds(),
		"profileRedisUpdateFloorUs":  updateBidResult.TimeUpdateFloor.Microseconds(),
	})

	if updateBidResult.WasBidSaved {
		// Bid is eligible to win the auction
		eligibleAt = time.Now().UTC()
		log = log.WithField("timestampEligibleAt", eligibleAt.UnixMilli())

		// Save to memcache in the background
		if api.memcached != nil {
			go func() {
				err = api.memcached.SaveExecutionPayload(submission.BidTrace.Slot, submission.BidTrace.ProposerPubkey.String(), submission.BidTrace.BlockHash.String(), getPayloadResponse)
				if err != nil {
					log.WithError(err).Error("failed saving execution payload in memcached")
				}
			}()
		}
	}

	nextTime = time.Now().UTC()
	pf.RedisUpdate = uint64(nextTime.Sub(prevTime).Microseconds())
	pf.Total = uint64(nextTime.Sub(receivedAt).Microseconds())

	// All done, log with profiling information
	log.WithFields(logrus.Fields{
		"profileDecodeUs":    pf.Decode,
		"profilePrechecksUs": pf.Prechecks,
		"profileSimUs":       pf.Simulation,
		"profileRedisUs":     pf.RedisUpdate,
		"profileTotalUs":     pf.Total,
	}).Info("received block from builder")
	w.WriteHeader(http.StatusOK)
}
```

### /eth/v1/builder/header 

[Block proposal](https://docs.flashbots.net/flashbots-mev-boost/architecture-overview/block-proposal)
Sidecar will forward to the boost and relay /eth/v1/builder/header_with_proofs endpoint.
The relay will return bid and proofs to the sidecar. Proposer will check the proof,if not valid, it will use the block with local builder.

```go
func (api *RelayAPI) handleGetHeaderWithProofs(w http.ResponseWriter, req *http.Request) {
	vars := mux.Vars(req)
	slotStr := vars["slot"]
	parentHashHex := vars["parent_hash"]
	proposerPubkeyHex := vars["pubkey"]
	ua := req.UserAgent()
	headSlot := api.headSlot.Load()

	slot, err := strconv.ParseUint(slotStr, 10, 64)
	if err != nil {
		api.RespondError(w, http.StatusBadRequest, common.ErrInvalidSlot.Error())
		return
	}

	requestTime := time.Now().UTC()
	slotStartTimestamp := api.genesisInfo.Data.GenesisTime + (slot * common.SecondsPerSlot)
	msIntoSlot := requestTime.UnixMilli() - int64((slotStartTimestamp * 1000))

	log := api.log.WithFields(logrus.Fields{
		"method":           "getHeaderWithProofs",
		"headSlot":         headSlot,
		"slot":             slotStr,
		"parentHash":       parentHashHex,
		"pubkey":           proposerPubkeyHex,
		"ua":               ua,
		"mevBoostV":        common.GetMevBoostVersionFromUserAgent(ua),
		"requestTimestamp": requestTime.Unix(),
		"slotStartSec":     slotStartTimestamp,
		"msIntoSlot":       msIntoSlot,
	})

	if len(proposerPubkeyHex) != 98 {
		api.RespondError(w, http.StatusBadRequest, common.ErrInvalidPubkey.Error())
		return
	}

	if len(parentHashHex) != 66 {
		api.RespondError(w, http.StatusBadRequest, common.ErrInvalidHash.Error())
		return
	}

	if slot < headSlot {
		api.RespondError(w, http.StatusBadRequest, "slot is too old")
		return
	}

	api.boltLog.Info("getHeaderWithProofs request received")

	if slices.Contains(apiNoHeaderUserAgents, ua) {
		log.Info("rejecting getHeaderWithProofs by user agent")
		w.WriteHeader(http.StatusNoContent)
		return
	}

	if api.ffForceGetHeader204 {
		log.Info("forced getHeaderWithProofs 204 response")
		w.WriteHeader(http.StatusNoContent)
		return
	}

	// Only allow requests for the current slot until a certain cutoff time
	if getHeaderRequestCutoffMs > 0 && msIntoSlot > 0 && msIntoSlot > int64(getHeaderRequestCutoffMs) {
		log.Info("getHeaderWithProofs sent too late")
		w.WriteHeader(http.StatusNoContent)
		return
	}

	bid, err := api.redis.GetBestBid(slot, parentHashHex, proposerPubkeyHex)
	if err != nil {
		log.WithError(err).Error("could not get bid")
		api.RespondError(w, http.StatusBadRequest, err.Error())
		return
	}

	bidBlockHash, err := bid.BlockHash()
	if err != nil {
		api.boltLog.WithError(err).Error("could not get bid block hash")
		api.RespondError(w, http.StatusBadRequest, err.Error())
		return
	}

	// BOLT: get preconfirmations proof of the best bid if available
	proof, err := api.redis.GetInclusionProof(slot, proposerPubkeyHex, bidBlockHash.String())
	if err != nil {
		api.boltLog.WithError(err).Error("failed getting preconfirmation proofs", proof)
		// We don't respond with an error and early return since proofs might be missing
	}

	if proof != nil {
		api.boltLog.Infof("Got inclusion proof from cache")
	}

	if bid == nil || bid.IsEmpty() {
		api.boltLog.Info("Bid is nill or is empty")
		w.WriteHeader(http.StatusNoContent)
		return
	}

	value, err := bid.Value()
	if err != nil {
		log.WithError(err).Info("could not get bid value")
		api.RespondError(w, http.StatusBadRequest, err.Error())
	}
	blockHash, err := bid.BlockHash()
	if err != nil {
		log.WithError(err).Info("could not get bid block hash")
		api.RespondError(w, http.StatusBadRequest, err.Error())
	}

	// Error on bid without value
	if value.Cmp(uint256.NewInt(0)) == 0 {
		api.boltLog.Info("Bid has 0 value")
		w.WriteHeader(http.StatusNoContent)
		return
	}

	// BOLT: Include the proofs in the final bid
	bidWithProofs := &common.BidWithPreconfirmationsProofs{
		Bid:    bid,
		Proofs: proof,
	}

	log.WithFields(logrus.Fields{
		"value":     value.String(),
		"blockHash": blockHash.String(),
	}).Info("bid delivered with proof")

	api.RespondOK(w, bidWithProofs)
}

```

### /eth/v1/builder/blinded_blocks

Relay receives a signed blinded block and get unblinded execution payload.

## Challenges