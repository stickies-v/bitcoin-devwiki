A best-effort collection of all vulns found exclusively via [fuzzing](https://github.com/bitcoin/bitcoin/blob/master/doc/fuzzing.md). That is, the unit and functional tests passed.

This includes vulns found on pull requests. Vulns found on released versions are tracked on https://en.bitcoin.it/wiki/Common_Vulnerabilities_and_Exposures.

_Unique Id_ is:

* _cve-yyyy-nnn_ for vulns with assigned CVE (Common Vulnerabilities and Exposures) identifiers
* _pull-nnnn_ for vulns that exist(ed) on the pull request with given id
* _master-ffff_ for vulns that exist on the master branch as of the given commit id
* _undisclosed-yyyy-mm-dd_ for undisclosed vulns that have been reported on that day

_Discovery_ is:

* `qa-assets` means the vuln was triggered by one of the inputs in https://github.com/bitcoin-core/qa-assets
* `dynamic` means the vuln was triggered by none of the fuzz inputs in qa-assets, but can be found with an existing fuzz target and enough CPU time
* `mod` means the vuln was triggered by a fuzz target that isn't publicly available or a fuzz target that is locally modified.

The remaining columns follow the definitions from https://en.bitcoin.it/wiki/Common_Vulnerabilities_and_Exposures


| Unique ID              | Discovery | Severity | Attack is... | Found by       | Flaw                 |
|------------------------|-----------|----------|--------------|----------------|----------------------|
| cve-2017-18350         | qa-assets | DoS      | easy         | practicalswift | SOCKS5 buffer overflow ([details](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2019-November/017453.html)) |
| cve-2018-20586         | mod       | log injection | easy         | practicalswift | Log injection vulnerability ([details](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2019-November/017487.html)) |
| cve-2019-18936         | qa-assets | DoS      | easy         | practicalswift | DoS in UniValue which is triggerable via RPC interface ([details](https://nvd.nist.gov/vuln/detail/CVE-2019-18936)) |
| pull-18808             | qa-assets | DoS      | easy         | MarcoFalke     | Missing nullptr check ([details](https://github.com/bitcoin/bitcoin/pull/18808#discussion_r417307258)) |
| master-9efd86a         | mod       | DoS      | easy         | practicalswift | Assert on untrusted input ([details](https://github.com/bitcoin/bitcoin/pull/20317#issuecomment-723046620), [details](https://github.com/bitcoin/bitcoin/pull/20317#issuecomment-723047111)) |
| undisclosed-2020-10-09 | mod       | Netsplit | Very hard    | practicalswift | Undisclosed flaw |

## Non-Exploitable Issues

Issues without Severity

| Unique ID              | Discovery | Found by       | Flaw                  |
|------------------------|-----------|----------------|-----------------------|
| pull-10977             | mod       | practicalswift | Use of uninitialized memory (UUM) in RPC command getnetworkinfo ([details](https://github.com/bitcoin/bitcoin/pull/10977)) |
| pull-13546             | mod       | practicalswift | Use of uninitialized memory (UUM) in CWallet::CreateTransaction ([details](https://github.com/bitcoin/bitcoin/pull/13546)) |
| pull-13712             | qa-assets | practicalswift | Use of uninitialized memory (UUM) in ParseHDKeypath ([details](https://github.com/bitcoin/bitcoin/pull/13712))
| pull-16800             | qa-assets | practicalswift | Multiple Miniscript issues: heap out-of-bounds read, stack depth DoS, assertion failure, unhandled exception ([details](https://github.com/bitcoin/bitcoin/pull/16800#issuecomment-527797423), [details](https://github.com/bitcoin/bitcoin/pull/16800#issuecomment-530808234), [details](https://github.com/bitcoin/bitcoin/pull/16800#issuecomment-530876527))
| pull-17149             | qa-assets | practicalswift | Multiple PSBT issues: heap use after free, signed integer overflows, etc. ([details](https://github.com/bitcoin/bitcoin/issues/17149)) |
| pull-17501             | qa-assets | practicalswift | Base58 decoding is done without checking that the input size is reasonable ([details](https://github.com/bitcoin/bitcoin/issues/17501)) |
| pull-17642             | mod       | practicalswift | Use of uninitialized memory (UUM) in RPC command bumpfee ([details](https://github.com/bitcoin/bitcoin/issues/17642)) |
| pull-17624             | mod       | practicalswift | Use of uninitialized memory (UUM) when receiving a transaction we already have ([details](https://github.com/bitcoin/bitcoin/issues/17624)) |
| pull-17718             | qa-assets | practicalswift | DecodeBase58 is too liberal when decoding ([details](https://github.com/bitcoin/bitcoin/issues/17718)) |
| pull-18033             | qa-assets | practicalswift | Heap buffer-overflow in GetMappedAS ([details](https://github.com/bitcoin/bitcoin/issues/18033)) |
| pull-18162             | qa-assets | practicalswift | Use of uninitialized memory (UUM) in FormatISO8601DateTime ([details](https://github.com/bitcoin/bitcoin/pull/18162)) |
| pull-18242             | qa-assets | practicalswift | Use of uninitialized memory (UUM) in case of invalid P2P command name ([details](https://github.com/bitcoin/bitcoin/pull/18242#issuecomment-593674721)) |
| pull-18261             | qa-assets | practicalswift | Use of uninitialized memory (UUM) in Erlay P2P code ([details](https://github.com/bitcoin/bitcoin/pull/18261#issuecomment-596803815))
| pull-18858             | qa-assets | practicalswift | Signed integer overflow in CCoinsViewCache::GetValueIn ([details](https://github.com/bitcoin/bitcoin/issues/18858)) |
| pull-19237             | qa-assets | practicalswift | Use of uninitialized memory (UUM) in CPubKey deserialization code ([details](https://github.com/bitcoin/bitcoin/issues/19235)) |
| pull-19930             | qa-assets | guidovranken   | Signed integer overflow in SipHasher ([details](https://github.com/bitcoin/bitcoin/issues/19930)) |
| pull-20135             | qa-assets | practicalswift | Invalid integer negation in abs64 ([details](https://github.com/bitcoin/bitcoin/issues/20135)) |
| pull-20402             | qa-assets | practicalswift | Invalid integer negation in FormatMoney reachable via RPC call decoderawtransaction ([details](https://github.com/bitcoin/bitcoin/issues/20402)) |
| pull-20607             | qa-assets | practicalswift | Signed integer overflow in CFeeRate::GetFee reachable via RPC call analyzepsbt ([details](https://github.com/bitcoin/bitcoin/issues/20607)) |
| pull-20626             | qa-assets | practicalswift | Signed integer overflow in CTxMemPool::PrioritiseTransaction reachable via RPC call prioritisetransaction ([details](https://github.com/bitcoin/bitcoin/issues/20626)) |
| pull-20867             | qa-assets | darosior       | implicit-integer-sign-change in multisig policy ([details](https://github.com/bitcoin/bitcoin/pull/20867#issuecomment-782474611)) |
| pull-20914             | qa-assets | practicalswift | Null pointer derefence in CBlockIndexWorkComparator::operator() reachable via RPC call invalidateblock ([details](https://github.com/bitcoin/bitcoin/issues/20914)) |
| pull-21043             | mod       | Crypt-iQ       | Signed integer overflow in version message processing ([details](https://github.com/bitcoin/bitcoin/pull/21043)) |