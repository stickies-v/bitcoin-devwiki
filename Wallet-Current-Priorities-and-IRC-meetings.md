What are you currently working on with the Bitcoin Core wallet? (Add your GitHub username and the date and a link to the PR, gist or branch where you're working. Don't put your entry below anyone who's more than a month out of date!)

Please update in advance of the wallet meeting held every two weeks on Friday at 19:00 UTC (**#bitcoin-core-dev** on **Libera.chat**). The next meeting is on **June 18th at 19:00 UTC**. Specific topics for discussion can also be preregistered on IRC using `#proposedwalletmeetingtopic`. This will be added to https://gnusha.org/bitcoin-core-dev/proposedwalletmeetingtopics.txt by a bot prior to the meeting.

Notes from previous meeting: #21365 is merged. For the 22.0 milestone, open wallet PRs are #22166 and #21329 (@achow101 priorities), #22154 and #19651 (more bug fix-ish and can be merged after feature freeze). There is also #22275 but that is pretty small and low priority.

### @achow101 2021-06-04
* `OutputType::BECH32M` [#22154](https://github.com/bitcoin/bitcoin/pull/22154)
* Coin selection upgrades
  * Refactor `CreateTransactionInternal` [#22008](https://github.com/bitcoin/bitcoin/pull/22008)
  * Waste metric to choose algo [#22009](https://github.com/bitcoin/bitcoin/pull/22009)

(@achow101 reviewing #21365 on Twitch: https://www.twitch.tv/videos/1048874953)