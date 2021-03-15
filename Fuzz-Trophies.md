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
| pull-17642             | mod       | practicalswift | Use of uninitialized memory (UUM) in RPC command bumpfee ([details](https://github.com/bitcoin/bitcoin/issues/17642)) |
| pull-17624             | mod       | practicalswift | Use of uninitialized memory (UUM) when receiving a transaction we already have ([details](https://github.com/bitcoin/bitcoin/issues/17624)) |
| pull-18162             | qa-assets | practicalswift | Use of uninitialized memory (UUM) in FormatISO8601DateTime ([details](https://github.com/bitcoin/bitcoin/pull/18162)) |
| pull-18242             | qa-assets | practicalswift | Use of uninitialized memory (UUM) in case of invalid P2P command name ([details](https://github.com/bitcoin/bitcoin/pull/18242#issuecomment-593674721)) |
| pull-18261             | qa-assets | practicalswift | Use of uninitialized memory (UUM) in Erlay P2P code ([details](https://github.com/bitcoin/bitcoin/pull/18261#issuecomment-596803815))
| pull-19237             | qa-assets | practicalswift | Use of uninitialized memory (UUM) in CPubKey deserialization code ([details](https://github.com/bitcoin/bitcoin/issues/19235)) |
| pull-20867             | qa-assets | darosior       | implicit-integer-sign-change in multisig policy ([details](https://github.com/bitcoin/bitcoin/pull/20867#issuecomment-782474611)) |
| pull-21043             | mod       | Crypt-iQ       | Signed integer overflow in version message processing ([details](https://github.com/bitcoin/bitcoin/pull/21043)) |
