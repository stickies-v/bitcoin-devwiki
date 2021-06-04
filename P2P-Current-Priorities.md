What are you currently working on in the P2P realm? (Add your github username and the date, and any projects in bullet points, and a link to the PR, gist, branch etc where you're working. Don't put your entry below anyone who's more than a month out of date!)

### @ajtowns 2021-05-17

* Some net_processing clean ups [#20758](https://github.com/bitcoin/bitcoin/pull/20758) - current is removing g_cs_orphans lock and related changes [#21527](https://github.com/bitcoin/bitcoin/pull/21527)
* dandelion
  - considering "dandelion-lite" (stem for exactly one hop, the flood -- no p2p changes required)
  - question is whether that provides enough benefits to be worthwhile? answer: simulate and see!
  - Giulia's updated https://github.com/gfanti/dandelion-simulations/tree/dandelion-lite ; some ensuing discussion on twitter https://twitter.com/giuliacfanti/status/1362963585471815680
* want to see erlay
* UASF/bip8 safety cf [#19573](https://github.com/bitcoin/pull/19573)

### @jonatack 2021-06-04

* [#21261](https://github.com/bitcoin/bitcoin/pull/21261) rewriting the inbound peer eviction to easily protect peers connecting via multiple special networks (onion, "localhost", I2P, CJDNS) and begin protecting I2P peers.

### @vasild 2021-06-04

* Minor I2P tweaks
  * [pull/21514 p2p: Ignore ports on I2P addresses](https://github.com/bitcoin/bitcoin/pull/21514)
  * [pull/22112 Force port 0 in I2P](https://github.com/bitcoin/bitcoin/pull/22112)
  * (no PR yet) Add I2P to `CNetAddr::IsRelayable()`
  * Ditch [prefer-8333 ports](https://github.com/bitcoin/bitcoin/blob/6e22b522f9505d6a3c71ef9972aea6ae3fb10d2e/src/net.cpp#L2020-L2026) for I2P; done as part of [pull/22112 Force port 0 in I2P](https://github.com/bitcoin/bitcoin/pull/22112)
  * (no PR yet) Add some basic docs/howto at `doc/i2p.md`, similarly to `doc/tor.md` ([hint](https://bitcoin.stackexchange.com/questions/103402/how-can-i-use-bitcoin-core-with-the-anonymous-network-protocol-i2p))
* [pull/21878 Make all networking code mockable](https://github.com/bitcoin/bitcoin/pull/21878)
* Implement CJDNS support

### @amitiuttarwar 2021-01-27

* tx rebroadcast [#21061](https://github.com/bitcoin/bitcoin/pull/21061)

### @ariard 2021-02-09

- stop to process unrequested txn : https://github.com/bitcoin/bitcoin/pull/20277
   - see ML proposal : https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-February/018391.html
- review erlay
- altnet : move personal notes under some wiki project page, work on a production branch on top of [#19160](https://github.com/bitcoin/bitcoin/pull/19160), headers over DNS as first integration

### @jnewbery 2021-05-30

- Move application layer data from net to net_processing:
  - [#19398](https://github.com/bitcoin/bitcoin/issues/19398) describes the high-level design
  - The next PRs in the series are built on [#21527](https://github.com/bitcoin/bitcoin/pull/21527) (net_processing: lock clean up)
- Review tx rebroadcast [#21061](https://github.com/bitcoin/bitcoin/pull/21061)
- Clean up addrman:
  - [#20233](https://github.com/bitcoin/bitcoin/pull/20233) makes consistency checks a runtime option, so they can be used in unit/functional/fuzz tests (currently WIP)
- Review AJ's general cleanups [#20758](https://github.com/bitcoin/bitcoin/pull/20758)
- Review Carl's de-globalize chainstate manager [#20158](https://github.com/bitcoin/bitcoin/pull/20158) (this is mostly in validation but touches the net_processing-validation interface a lot).

### @sdaftuar 2021-01-27

* disabletx
  * [code=#20726](https://github.com/bitcoin/bitcoin/pull/20726)
  * [bip=#1025](https://github.com/bitcoin/bips/pull/1052)
  * [-dev thread](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-January/018340.html)

### @MarcoFalke 2021-01-26

* Fuzzing p2p:
  * https://github.com/bitcoin/bitcoin/pull/20915  fuzz: Fail if message type is not fuzzed
  * https://github.com/bitcoin/bitcoin/pull/20995  fuzz: Avoid initializing version to less than MIN_PEER_PROTO_VERSION 

### @naumenkogs 2021-02-23
* [Erlay](https://github.com/bitcoin/bitcoin/pull/18261)