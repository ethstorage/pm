# Wishlist / TODO list

- [ ] Study and summary of the latest Verkle Tree
  - https://docs.google.com/document/d/1D2GtzI3q9btZd1ZOzCsWPsvzCaA-fCLZdXDtawoPUyM/edit
- [ ] Run a node of Reth

## ZK Fraud Proof

- [ ] Study and summary of other x fraud-proof proposals in https:/github.com/ethereun-optinsn/ecesysten-contrbutions/1ssues/61
- [ ] mini-program (like Fib) with host func to input and verified on ZkWASM
- [ ] External Oracle support in WASM (with WASI?)
- [ ] Explore tiny-go compiled op-program
  - [ ] Issue of compilatlon e.g., missing crypto library in tiny-go (or more libraries)?
  - [ ] Any issue of 2kMA5M running tiny-go compiled code, e.g., all syscalls/hostio to virtual functions? (Qi)
  - [ ] Need to confirm the time and efforts with sinka on compilation and zkWASM support (Qi)
- [ ] Explore tiny-go compiled wasm interpreter with go runtime with geth compiled op-program
  - [ ] Implement a fully-functional interpreter on existing go implementation (e.g., https://github.com/mattn/gowasmer/blob/matn/gowasmer.go)
  - [ ] Implement an interpreter with hostio (POSIX style)
  - [ ] Replay a simple program with I0 (read/write/open/close with similar IO siz as op-program) compiled with geth
- [ ] Explore Arb rust interpreter
  - [ ] How to replace wasm import with a rust func (https://github.com/wasmerio/wasmer/blob/master/examples/hello_world.rs) (Yanlong)
  - [ ] Implement an interpreter with hostio (POSIX style) and replace existing Arb runtimes (and replay geth-compiled op-program)
  - [ ] How is Wasm code replaced by wavm in Arb?

- [x] op-program compilation with WASM target [yanlong / po)
  - [x] [fastcache](https: //github.cam/ethstoragefastcache)
  - [x] https://github.cam/ethstarage/optimism/tree/ap-wasm-Pa/op-program
- [x] Golang host communication In WASM (WASI)
  - [x] https://github.com/Wasner10/asner-g0/tree/masterfhow-to-Tun-go-programs-compiled-to-wbasSembly-module5-1th-wasner-g0
  - [x] https://github.com/wasmerio/wasmer-go/blob/master/examples/exanple_exports_function_test.go
  - [x] https://wazero.1o/languages/tnygo/#unsupported-standard-libraries
- [x] [zkwASM training (Q1)
- [x] WASH emulator used in Arb
  - Arbitrum now uses a customized emulator as they use customized WAWM
- [ ] <strike> Find a WASM emulator to replay block transition </strike>
      
