A best-effort collection of all vulns found exclusively via [fuzzing](https://github.com/bitcoin/bitcoin/blob/master/doc/fuzzing.md). That is, the unit and functional tests passed.

This includes vulns found on pull requests. Vulns found on released versions are tracked on https://en.bitcoin.it/wiki/Common_Vulnerabilities_and_Exposures.

_Unique Id_ is:

* _pull-nnnn_ for vulns that exist(ed) on the pull request with given id
* _master-ffff_ for vulns that exist on the master branch as of the given commit id

_Discovery_ is:

* `qa-assets` means the vuln was triggered by a one of the inputs in https://github.com/bitcoin-core/qa-assets
* `dynamic` means the vuln was triggered by none of the fuzz inputs in qa-assets, but can be found with an existing fuzz target and enough CPU time
* `mod` means the vuln was triggered by a fuzz target that isn't publicly available or a fuzz target that is locally modified.


| Unique ID           | Discovery | Severity | Attack is... | Flaw                  |
|---------------------|-----------|----------|--------------|-----------------------|
| pull-18808          | qa-assets | DoS      | easy         | Missing nullptr check https://github.com/bitcoin/bitcoin/pull/18808#discussion_r417307258 |
| master-9efd86a      | mod       | DoS      | easy         | Assert on untrusted input https://github.com/bitcoin/bitcoin/pull/20317#issuecomment-723047111 |
| pull-21043          | mod       | none     | easy         | signed integer overflow in version message processing https://github.com/bitcoin/bitcoin/pull/21043 |