## Devnet

Entrance Info:
```bash
Explorer：http://142.132.154.16/
RPC: http://142.132.154.16:8545
Custom Gas Token: 0xe6ABD81D16a20606a661D4e075cdE5734AB62519
PORTAL_ADDRESS: 0x9eD5C0fD5b1e6852744819a9F33573f69D5345AF
```


Deploy Config:

```json
{
  "l1StartingBlockTag": "0xc29c621cfc4331e834db10de0c6e6c733f28dd8c5a4ddf95b0f320ebef6841d1",

  "l1ChainID": 11155111,
  "l2ChainID": 42069,
  "l2BlockTime": 2,
  "l1BlockTime": 12,

  "maxSequencerDrift": 600,
  "sequencerWindowSize": 3600,
  "channelTimeout": 300,

  "p2pSequencerAddress": "0x6f8Ef8427484a12B0Ca3Ba9e40D34f1e7FbfC9dc",
  "batchInboxAddress": "0x27504265a9bc4330e3fe82061a60cd8b6369b4dc",
  "batchSenderAddress": "0xf16FE8a2BA6bfaACf11c8bbf16E5Ccc2f718474b",

  "l2OutputOracleSubmissionInterval": 21600,
  "l2OutputOracleStartingBlockNumber": 0,
  "l2OutputOracleStartingTimestamp": 1716476628,

  "l2OutputOracleProposer": "0x1dA2DA8c61805b2D0d27E48718C6Ab1E0988bc10",
  "l2OutputOracleChallenger": "0x21A380157A6ACdc19c0e411c7Ad1143bd6b1a443",

  "finalizationPeriodSeconds": 12,

  "proxyAdminOwner": "0x21A380157A6ACdc19c0e411c7Ad1143bd6b1a443",
  "baseFeeVaultRecipient": "0x21A380157A6ACdc19c0e411c7Ad1143bd6b1a443",
  "l1FeeVaultRecipient": "0x21A380157A6ACdc19c0e411c7Ad1143bd6b1a443",
  "sequencerFeeVaultRecipient": "0x21A380157A6ACdc19c0e411c7Ad1143bd6b1a443",
  "finalSystemOwner": "0x21A380157A6ACdc19c0e411c7Ad1143bd6b1a443",
  "superchainConfigGuardian": "0x21A380157A6ACdc19c0e411c7Ad1143bd6b1a443",

  "baseFeeVaultMinimumWithdrawalAmount": "0x8ac7230489e80000",
  "l1FeeVaultMinimumWithdrawalAmount": "0x8ac7230489e80000",
  "sequencerFeeVaultMinimumWithdrawalAmount": "0x8ac7230489e80000",
  "baseFeeVaultWithdrawalNetwork": 0,
  "l1FeeVaultWithdrawalNetwork": 0,
  "sequencerFeeVaultWithdrawalNetwork": 0,

  "gasPriceOracleOverhead": 0,
  "gasPriceOracleScalar": 1000000,

  "enableGovernance": true,
  "governanceTokenSymbol": "OP",
  "governanceTokenName": "Optimism",
  "governanceTokenOwner": "0x21A380157A6ACdc19c0e411c7Ad1143bd6b1a443",

  "l2GenesisBlockGasLimit": "0x1c9c380",
  "l2GenesisBlockBaseFeePerGas": "0x3b9aca00",
  "l2GenesisRegolithTimeOffset": "0x0",

  "eip1559Denominator": 50,
  "eip1559DenominatorCanyon": 250,
  "eip1559Elasticity": 6,

  "l2GenesisEcotoneTimeOffset": "0x0",
  "l2GenesisDeltaTimeOffset": "0x0",
  "l2GenesisCanyonTimeOffset": "0x0",

  "systemConfigStartBlock": 0,

  "requiredProtocolVersion": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "recommendedProtocolVersion": "0x0000000000000000000000000000000000000000000000000000000000000000",

  "faultGameAbsolutePrestate": "0x03c7ae758795765c6664a5d39bf63841c71ff191e9189522bad8ebff5d4eca98",
  "faultGameMaxDepth": 44,
  "faultGameClockExtension": 0,
  "faultGameMaxClockDuration": 600,
  "faultGameGenesisBlock": 0,
  "faultGameGenesisOutputRoot": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "faultGameSplitDepth": 14,

  "fundDevAccounts": false,
  "useFaultProofs": false,
  "faultGameWithdrawalDelay": 120,
  "proofMaturityDelaySeconds": 120,
  "disputeGameFinalityDelaySeconds": 120,
  "useCustomGasToken": true,
  "customGasTokenAddress": "0xe6ABD81D16a20606a661D4e075cdE5734AB62519",

  "preimageOracleMinProposalSize": 1800000,
  "preimageOracleChallengePeriod": 86400
}
```



L1 contracts:

```json
{
  "AddressManager": "0xd37a90FD4ed35B4b782B6E620ccaAA8cc49Ae11f",
  "AnchorStateRegistry": "0xadDFC07176EE27a0a09cA869aCe056114E9B14CA",
  "AnchorStateRegistryProxy": "0xBE90dFcc0e75Cd87d613befF36f48fEFC3366667",
  "DelayedWETH": "0xaF1B38a056cfBA7E09bde970CeA07812B0395F30",
  "DelayedWETHProxy": "0xD4cE0F38b8341f693b008C65957dEE0d97B51779",
  "DisputeGameFactory": "0x6cd63086476E12e0638B53d8f05D69D17Aeb54b0",
  "DisputeGameFactoryProxy": "0x4D08013D127Ceb8fb7a308B04faa28Fe1C1E34dD",
  "L1CrossDomainMessenger": "0x40503CDBdC58BeF5ed87077Ad40Bfd0a7Fadd978",
  "L1CrossDomainMessengerProxy": "0x7AB850bf4c1979Ad92183C23cC5c9168C011D023",
  "L1ERC721Bridge": "0xbDE5CCaD83bC553d885d64878D9F446AC5881bc8",
  "L1ERC721BridgeProxy": "0x43A07fBBF904ed36110B98deA02FBCBBb6aBFC18",
  "L1StandardBridge": "0xB136cadE741aAB0b6938930Cb25548039EE8A6F5",
  "L1StandardBridgeProxy": "0xFafD9BB92C37d147b99EfA1D5B60005D995Ee80c",
  "L2OutputOracle": "0xCDaD209a741a49246d5cab8e38c1D8953Ec1C157",
  "L2OutputOracleProxy": "0x05D060E72B23C324c2CE1d904339686B31830f4F",
  "Mips": "0x43C54334E7dcDee3F240a4081ba1271ccEdac3a1",
  "OptimismMintableERC20Factory": "0xe0026e449DE3cD7f36FEDb67fe5A33B1A1eF9E3D",
  "OptimismMintableERC20FactoryProxy": "0x2d57FF050911A378FD2aE9778C1e1871EdBa7884",
  "OptimismPortal": "0x1A611fFC5905815ba3f57CD6C57406F98Bf4841e",
  "OptimismPortal2": "0x82F8747F797980fd01a2f5B710FC2270A5A7C48D",
  "OptimismPortalProxy": "0x9eD5C0fD5b1e6852744819a9F33573f69D5345AF",
  "PreimageOracle": "0xCC7fBA9B5B3A5E6Aeeb63B47265c4a41E7c9E339",
  "ProtocolVersions": "0xcAf91aDf94dEc71858309c3Cd25Dc8Db59BAbEFD",
  "ProtocolVersionsProxy": "0x335fc936AF3144D2e93CB5abFD548a1a6cf6178D",
  "ProxyAdmin": "0xaD2dB2044D517e05c40dA5a0700aB0922071A5ed",
  "SafeProxyFactory": "0xa6B71E26C5e0845f74c812102Ca7114b6a896AB2",
  "SafeSingleton": "0xd9Db270c1B5E3Bd161E8c8503c55cEABeE709552",
  "SuperchainConfig": "0xb3F2a8e081E57280eD64045e625b058d79586C31",
  "SuperchainConfigProxy": "0x4B9814Cf674188316Ed029336cACdde8D404e458",
  "SystemConfig": "0x121750C6725D9597A397c086d7aA87630253561F",
  "SystemConfigProxy": "0xC09AaA3E5F27BD49411063BB8456e627857f3d3E",
  "SystemOwnerSafe": "0x05935310dFDF83B0693489425041AA0e0e426A73"
}
```