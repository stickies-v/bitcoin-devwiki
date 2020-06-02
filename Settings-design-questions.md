In-progress wiki page to summarize design questions raised in various settings pull requests. Meant to serve as discussion summary and informal decision log.

### UI design questions

- Precedence of command line, static config, dynamic settings
- GUI handling conflicting settings
  - Keep "Options set in this dialog are overridden by the command line" overlay and show persistent value, or drop and show active value?
- Static config feature to disable dynamic settings
- Static config feature to override precedence
- Dynamic settings storage
  - ini format, json format, other format?
  - preserve comments?

### Internal or corner case questions
- Default handling empty settings
  - Treat as unset, or treat as errors, or something else?
- Default handling negated int settings
  - Treat as 0, error, or something else?
- Default handling negated string settings
  - Treat as empty string, error, or something else?
- Default handling negated list settings
  - Treat as empty list, error, or something else?
- Drop `IsArgSet` method?
  - No known use cases for calling `IsArgSet` method without also calling `IsArgNegated` method
    - In cases where `IsArgSet` is called correctly with `IsArgNegated`, code can be simplified by using `GetArg*` calls instead
    - In cases where `IsArgSet` is called incorrectly without `IsArgNegated`, there are strange behaviors and arguable bugs like `-norpcwallet` connecting to the `/wallet/0/` endpoint
- Drop `ForceSetArg` and/or `SoftSetArg` methods?
  - Unclear if these help more code clarity more than they hurt. There are no cases where they are ever needed to provide functionality. They are just a way of injecting external data into the ArgsManager object that could be stored equivalently (and maybe with less complexity and ambiguity) by respective ArgsManager clients.

### Pull requests

- [#11082 Add new bitcoin_rw.conf file that is used for settings modified by this software itself](https://github.com/bitcoin/bitcoin/pull/11082) luke-jr
- [#12833 move QSettings to bitcoin_rw.conf where possible](https://github.com/bitcoin/bitcoin/pull/12833) Sjors
- [#13818 More intuitive GUI settings behavior when -proxy is set](https://github.com/bitcoin/bitcoin/pull/13818) Sjors
- [#15454 Remove the automatic creation and loading of the default wallet](https://github.com/bitcoin/bitcoin/pull/15454) achow101
- [#15935 Add <datadir>/settings.json persistent settings storage](https://github.com/bitcoin/bitcoin/pull/15935) ryanofsky
- [#15936 Unify bitcoin-qt and bitcoind persistent settings](https://github.com/bitcoin/bitcoin/pull/15936) ryanofsky
- [#15937 Add loadwallet and createwallet load_on_startup options](https://github.com/bitcoin/bitcoin/pull/15937) ryanofsky