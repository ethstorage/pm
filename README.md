# Wishlist / TODO list

- [ ] Study and summary of latest Verkle Tree
  - https://docs.google.com/document/d/1D2GtzI3q9btZd1ZOzCsWPsvzCaA-fCLZdXDtawoPUyM/edit
- [ ] Run a node of Reth

## ZK Fraud Proof

- [ ] Study and summary of other x fraud proof proposals in https:/github.com/ethereun-optinsn/ecesysten-contrbutions/1ssues/61
- [ ] mini-program (like Fib] with host func to imput and verified on ZkWASM
- [ ] External oracle support in WASM (with WA5I?]
- [ ] Explore tiny-go corpiled op-program
  - [ ] Issue of compilatlon e.g., missing crypto library in tiny-go (or more libraries)?
  - [ ] Any issue of 2kMA5M runing tiny-go compiled code, e.g., all syscalls/hostlos to virtual functians? (01)
  - [ ] Need to confirm the time and gfforts with sinka on compilation and zkWASM support (Q1)
- [ ] Explore tiny-go compiled wasm interpreter with go runtine with geth compiled op-program
  - [ ] Implement a fully-functianal interpreter on existing go implementatlan (e.g., https://github.com/mattn/gowasmer/blob/matn/gowasmer.go)
  - [ ] Implenent a nterpreter with hostio IPO5X style)
  - [ ] Replay a simple program wlth I0 (read/write/open/close with sinilar I0 siz as op-progran) compiled with geth
- [ ] Explore Arb rust interpreter
  - [ ] How to replace wasm import with a rust fumc (Yanlong)
  - [ ] Implement a interpreter with hostio (P0SIX style) and replace existing Arb runtines (and replay geth-comp!led op-program)
  - [ ] How Wasm code is replaced by wavm in Arb?

- [x] op-program compilation with WASM target [yanlong / po)
  - [x] [fastcache](https: //github.cam/ethstoragefastcache)
  - [x] https://github.cam/ethstarage/optimism/tree/ap-wasm-Pa/op-program
- [x] Golang host communicat ion In WASM (WASI)
  - [x] https://github.com/Wasner10/asner-g0/tree/masterfhow-to-Tun-go-programs-compiled-to-wbasSembly-module5-1th-wasner-g0
  - [x] https://github.com/wasmerio/wasmer-go/blob/master/examples/exanple_exports_function_test.go
  - [x] https://wazero.1o/languages/tnygo/#unsupported-standard-libraries
- [x] [zkwASM training (Q1)
- [x] WASH emulator used in Arb
  - Arbitrum now uses customized emulator as they use customized WAWM
- [ ] <strike> Find a WASM emulator to replay block transition </strike>
      
