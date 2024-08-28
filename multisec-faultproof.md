## Tasks
- [x] FaultDisputeGame contract [draft](https://github.com/ethstorage/optimism/pull/17/files)
- [x] [FaultDisputeGame contract @Bill @Qi](https://github.com/ethstorage/optimism/blob/develop/packages/contracts-bedrock/src/dispute/FaultDisputeGameN.sol)
- [x] [4844 point evaluation @Po](https://github.com/dajuguan/solidity/blob/main/eip-4844-kzg/hardhat/test/Lock.js)
- [x] [contract test of 4-ary dispute game @Bill @Po](https://github.com/ethstorage/optimism/blob/develop/packages/contracts-bedrock/test/dispute/FaultDisputeGameN.t.sol)
- [ ] E2E test of 4-ary dispute game (@Bill @Po)
  - [ ] Alphabet game
    - [ ] fetch counter's sub claims from DA and upgrade [CalculateNextActions](https://github.com/ethstorage/optimism/blob/78e1084ec14d3003cb9e546b9eb5a22db7408ac2/op-challenger/game/fault/agent.go#L101) for [TraceAccessor interface](https://github.com/ethstorage/optimism/blob/c41bb739f15005412f227a130474433e168faf8c/op-challenger/game/fault/trace/alphabet/provider.go#L46) and [test it] (https://github.com/ethstorage/optimism/blob/78e1084ec14d3003cb9e546b9eb5a22db7408ac2/op-challenger/game/fault/solver/game_solver_test.go) -- 3 Weeks
    - [ ] upgrade [PerformAction](https://github.com/ethstorage/optimism/blob/78e1084ec14d3003cb9e546b9eb5a22db7408ac2/op-challenger/game/fault/responder/responder.go) and [test it](https://github.com/ethstorage/optimism/blob/fdf5bd0632e5e044292013e9212d3f249c444980/op-challenger/game/fault/solver/solver_test.go) -- 1 Weeks
    - [ ] [e2e test](https://github.com/ethstorage/optimism/blob/2586a82f6c270e86c766b2f895182c2c963e9ef5/op-e2e/faultproofs/output_alphabet_test.go) -- 3 Weeks
  - [ ] adapt for Cannon -- 2 Weeks
  - [ ] integrate [op-dispute-mon](https://github.com/ethereum-optimism/optimism/tree/develop/op-dispute-mon)
## Materials
- [op proposal & milestones](https://app.charmverse.io/op-grants/page-29596258544520615)

- Multi-sec analysis[ppt]: https://docs.google.com/presentation/d/1Wvk4IFiiE7mZ-PxWq7MX-BeWXJzJmGr5NkbRc9RBq3M/edit#slide=id.g266f97ce0a2_0_11
- How to run fault proof: https://hackmd.io/Meq0Qn77QDKKmjHusNP9TQ
- Anatomy of fault proof test: https://hackmd.io/GK_Fv9fQQmmbW3MW-b69Gw?view#Challenger
