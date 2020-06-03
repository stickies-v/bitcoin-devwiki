The [process separation project](https://github.com/bitcoin/bitcoin/projects/10) builds new `bitcoin-node`, `bitcoin-wallet`,
and `bitcoin-gui` executables that isolate node, wallet, and gui functionality in different
processes and communicate with each other across sockets.

A combined branch with all multiprocess changes can be found at
[ryanofsky@`ipc-export`](https://github.com/ryanofsky/bitcoin/commits/ipc-export)
with documentation in
[`doc/multiprocess.md`](https://github.com/ryanofsky/bitcoin/blob/ipc-export/doc/multiprocess.md).

For review purposes, the branch has been broken up into multiple PRs, major ones are listed and grouped below.

### Step 1: Refactoring PRs

Refactoring PRs replace wallet code accessing node state directly, and GUI code accessing wallet and node state directly, with new code accessing state indirectly through explicitly-defined interface classes in [`src/interfaces/`](https://github.com/ryanofsky/bitcoin/tree/ipc-export/src/interfaces) that [don't assume interface callers and callees have access to the same memory space](https://github.com/ryanofsky/bitcoin/blob/ipc-export/doc/developer-notes.md#internal-interface-guidelines).

- [X] [#10244 Refactor: separate gui from wallet and node](https://github.com/bitcoin/bitcoin/pull/10244)
- [X] [#14437 Refactor: Start to separate wallet from node](https://github.com/bitcoin/bitcoin/pull/14437)
- [X] [#15288 Remove wallet -> node global function calls](https://github.com/bitcoin/bitcoin/pull/15288)
- [X] [#17999 refactor: Add ChainClient setMockTime, getWallets methods](https://github.com/bitcoin/bitcoin/pull/17999)
- [X] [#18278 interfaces: Describe and follow some code conventions](https://github.com/bitcoin/bitcoin/pull/18278)

### Step 2: Build support PR

The build PR adds new `bitcoin-gui` and `bitcoin-node` makefile targets, a new travis variant, and new configure and depends changes to build against the [libmultiprocess](https://github.com/chaincodelabs/libmultiprocess) library. These changes only affect build scripts, not C++ code.

- [X] [#16367 Multiprocess build support](https://github.com/bitcoin/bitcoin/pull/16367)
- [X] [#18677 Multiprocess build support](https://github.com/bitcoin/bitcoin/pull/18677)

### Step 3: Blocking fix PRs

Performance improvements or fixes needed for multiprocess support.

- [X] [#17905 gui: Avoid redundant tx status updates](https://github.com/bitcoin/bitcoin/pull/17905)
- [X] [#18587 Avoid wallet tryGetBalances calls in WalletModel::pollBalanceChanged](https://github.com/bitcoin/bitcoin/pull/18587)
### Step 4: Spawned process PR

Minimal change changing `bitcoin-gui` to spawn a `bitcoin-node` process, and
`bitcoin-node` to spawn a `bitcoin-wallet` process and for gui, node, and wallet
functionality to run in the different processes and communicate though pipes.

- [ ] [#19160 Add basic spawn and IPC support](https://github.com/bitcoin/bitcoin/pull/19160)
- [ ] [#10102 Multiprocess bitcoin](https://github.com/bitcoin/bitcoin/pull/10102)

### Step 5: Ad-hoc connection PRs

Changes adding `-ipcconnect` and `-ipcbind` options and allowing `bitcoin-node`
to open a listening socket that allows incoming `bitcoin-gui` and `bitcoin-wallet`
connections.

- [ ] [`c83e46ccb4f` Add bitcoin-wallet -ipcconnect and bitcoin-node -ipcbind options](https://github.com/ryanofsky/bitcoin/commit/c83e46ccb4fb74b5a7b30d278d46dfbf721f9c91)
- [ ] [`1a1c4e0c528` Add bitcoin-gui -ipcconnect option](https://github.com/ryanofsky/bitcoin/commit/1a1c4e0c528306877e24c8302a6f80ebc8c6fb93)

### Followup changes

See [Multiprocess next steps](https://github.com/ryanofsky/bitcoin/blob/ipc-export/doc/multiprocess.md#next-steps)

Steps 1, 2, and 3 above can proceed simultaneously, but steps 4 and 5 depend on all earlier PRs to be merged before they are merged.

### Links
- [Presentation](https://docs.google.com/presentation/d/1AeJ-7gD-dItUgs5yH-HoEzLvXaEWe_2ZiGUUxYIXcws/edit) as of [pr/ipc.21](https://github.com/ryanofsky/bitcoin/releases/tag/pr%2Fipc.21) ([commits](https://github.com/ryanofsky/bitcoin/compare/master...pr/ipc.21)) 
- [Update](https://docs.google.com/presentation/d/1h47dU2g3__0YQ_bd0pxZ_eYqpxC4JkrnrAsV032HXGU/edit)