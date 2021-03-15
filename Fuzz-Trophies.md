A best-effort collection of all vulns found exclusively via [fuzzing](https://github.com/bitcoin/bitcoin/blob/master/doc/fuzzing.md). That is, the unit and functional tests passed.

This includes vulns found on pull requests. Vulns found on released versions are tracked on https://en.bitcoin.it/wiki/Common_Vulnerabilities_and_Exposures.

_Unique Id_ is:

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
| pull-18808             | qa-assets | DoS      | easy         | MarcoFalke     | Missing nullptr check ([details](https://github.com/bitcoin/bitcoin/pull/18808#discussion_r417307258)) |
| master-9efd86a         | mod       | DoS      | easy         | practicalswift | Assert on untrusted input ([details](https://github.com/bitcoin/bitcoin/pull/20317#issuecomment-723046620), [details](https://github.com/bitcoin/bitcoin/pull/20317#issuecomment-723047111)) |
| undisclosed-2020-10-09 | mod       | Netsplit | Very hard    | practicalswift | Undisclosed flaw |

## Non-Exploitable Issues

Issues without Severity

| Unique ID              | Discovery | Found by       | Flaw                  |
|------------------------|-----------|----------------|-----------------------|
| pull-20867             | qa-assets | darosior       | implicit-integer-sign-change in multisig policy ([details](https://github.com/bitcoin/bitcoin/pull/20867#issuecomment-782474611)) |
| pull-21043             | mod       | Crypt-iQ       | signed integer overflow in version message processing ([details](https://github.com/bitcoin/bitcoin/pull/21043)) |
| pull-19237             | qa-assets | practicalswift | CPubKey deserialization reads uninitialized memory ([details](https://github.com/bitcoin/bitcoin/issues/19235)) |
| pull-18162             | qa-assets | practicalswift | Uninitialized read in FormatISO8601DateTime ([details](https://github.com/bitcoin/bitcoin/pull/18162)) |