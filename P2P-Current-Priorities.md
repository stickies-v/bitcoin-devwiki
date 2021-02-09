What are you currently working on in the P2P realm? (Add your github username and the date, and any projects in bullet points, and a link to the PR, gist, branch etc where you're working)

### @ajtowns 2021-02-09

* Some net_processing clean ups [#20758](https://github.com/bitcoin/bitcoin/pull/20758) - [#20942](https://github.com/bitcoin/bitcoin/pull/20758) removes the next set of globals, after that is splitting out orphan handling to a separate module
* UASF/bip8 safety cf [#19573](https://github.com/bitcoin/pull/19573)
* want to see erlay and dandelion

### @amitiuttarwar 2021-01-27

* tx rebroadcast [#21061](https://github.com/bitcoin/bitcoin/pull/21061)
* small PR that breaks out a unit test helper from 21061 [#21121](https://github.com/bitcoin/bitcoin/pull/21121)

### @jnewbery 2021-01-27

- Move application layer data from net to net_processing:
  - [#19398](https://github.com/bitcoin/bitcoin/issues/19398) describes the high-level design
  - [#20721](https://github.com/bitcoin/bitcoin/pull/20721) is the next PR in the series - moves ping data from net to net_processing
- Clean up addrman:
  - [#20557](https://github.com/bitcoin/bitcoin/pull/20557) fixes some deserialization bugs introduced in [#16702](https://github.com/bitcoin/bitcoin/pull/16702)
  - [#20228](https://github.com/bitcoin/bitcoin/pull/20228) makes addrman a top-level component, allowing ctor arguments to be passed directly from init and removing boilerplate code from CConnman
  - [#20233](https://github.com/bitcoin/bitcoin/pull/20233) makes consistency checks a runtime option, so they can be used in unit/functional/fuzz tests.
- Review and help AJ's general cleanups [#20758](https://github.com/bitcoin/bitcoin/pull/20758)
- Review and help Carl's de-globalize chainstate manager [#20158](https://github.com/bitcoin/bitcoin/pull/20158) (this is mostly in validation but touches the net_processing-validation interface a lot).

### @sdaftuar 2021-01-27

* disabletx
  * [code=#20726](https://github.com/bitcoin/bitcoin/pull/20726)
  * [bip=#1025](https://github.com/bitcoin/bips/pull/1052)
  * [-dev thread](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-January/018340.html)

### @jonatack 2021-20-27

* eviction protection for onion and i2p peers [#20197](https://github.com/bitcoin/bitcoin/pull/20197)
* standardize the outbound full/block relay naming (one commit, scripted diff) [#20729](https://github.com/bitcoin/bitcoin/pull/20729)

### @MarcoFalke 2021-20-26

* Fuzzing p2p:
  * https://github.com/bitcoin/bitcoin/pull/20915  fuzz: Fail if message type is not fuzzed
  * https://github.com/bitcoin/bitcoin/pull/20995  fuzz: Avoid initializing version to less than MIN_PEER_PROTO_VERSION 

### @vasild 2021-01-26

* Code review while waiting input on:
  * [#20685 Add I2P support using I2P SAM](https://github.com/bitcoin/bitcoin/pull/20685)
  * [#20788 net: add RAII socket and use it instead of bare SOCKET](https://github.com/bitcoin/bitcoin/pull/20788)