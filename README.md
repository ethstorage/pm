# Wishlist / TODO list

- [ ] Study and summary of the latest Verkle Tree
  - https://docs.google.com/document/d/1D2GtzI3q9btZd1ZOzCsWPsvzCaA-fCLZdXDtawoPUyM/edit
- [ ] Run a node of Reth

## ZK Fraud Proof

### Materials

- Fraud Proof Comparison https://norswap.com/bedrock-vs-nitro/
- Replayable geth-compiled op-program on top of wasm_exec.js https://github.com/ethstorage/optimism/tree/js-io/op-program#build-js-wasm-and-replay
- Replayable geth-compiled op-program with GOOS=wasip1 https://github.com/ethstorage/optimism/tree/js-io/op-program#build-wasi-and-replay-without-op-host-program

### Tasks
- [ ] Study and summary of other x fraud-proof proposals in https:/github.com/ethereun-optinsn/ecesysten-contrbutions/1ssues/61
- [x] mini-program (like Fib) with host func to input and be verified on zkWASM
  - [x] A program with target=wasm-freestanding is [here](https://github.com/LiuJiazheng/go-to-wasm-example) with libc support (memory)
  - [ ] <strike> **A program with simple scheduler support (Frank) or with `fmt.Println()?`**
- [ ] Explore geth compiled op-program
  - [x] Replayable GOOS=js GOARCH=wasm with both fd/syscall and hostio [here](https://github.com/ethstorage/optimism/tree/js-io/op-program#build-js-wasm-and-replay)
  - [x] Replayable GOOS=wasip1 GOARCH=wasm with both fd/syscall and hostio [here](https://github.com/ethstorage/optimism/tree/js-io/op-program#build-wasi-and-replay-without-op-host-program)
  - [ ] Softfloat support [issue](https://github.com/golang/go/issues/62470)
  - [ ] Replayble zkWASM
- [ ] Explore Arb rust interpreter
  - [x] How to replace wasm import with a rust func (https://github.com/wasmerio/wasmer/blob/master/examples/hello_world.rs) (Yanlong)
  - [x] (Skipped) Implement an interpreter with hostio (POSIX style) and replace existing Arb runtimes (and replay geth-compiled op-program)
  - [ ] How is Wasm code replaced by wavm in Arb?
- [ ] zkWASM adapter
  - [ ] Replace floating point opcode to softfloat?
  - [ ] Or even better, remove them?
  - [x] Memory size issue: solved by removing cache in client (and be completed moved to server)
  - [ ] External Oracle support in WASM (with WASI?) (Qi)
  - [x] <strike> tiny-go compiled wasm with target=zkwasm
  - [ ] zkWASM emulator with provable state/trace


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
- [ ] <strike> Explore tiny-go compiled wasm interpreter with go runtime with geth compiled op-program
  - [ ] Implement a fully-functional interpreter on existing go implementation (e.g., https://github.com/mattn/gowasmer/blob/matn/gowasmer.go)
  - [ ] Implement an interpreter with hostio (POSIX style)
  - [ ] Replay a simple program with I0 (read/write/open/close with similar IO size as op-program) compiled with geth
- [ ] <strike> Explore tiny-go compiled op-program
  - [x] Issue of compilatlon e.g., missing crypto library in tiny-go (or more libraries)?  ([code]([url](https://github.com/ethstorage/optimism/pull/5)))
  - [x] Any issue of zkWASM running tiny-go compiled code, e.g., all syscalls/hostio to virtual functions? A: Expose 3-5 external functions
  - [ ] Need to confirm the time and efforts with sinka on compilation and zkWASM support (Qi)
  - [ ] **Replay the tiny-go compiled op-program in any emulator (emulator can be node+wasm_exec.js, target=wasm/wasi)**
    - [ ] Replay op-program native binary
    - [ ] Check memory size usage
    - [ ] Check # instructions using Cannon example command line
    - [ ] Replace hint/preimage IO with external functions (Qi)
