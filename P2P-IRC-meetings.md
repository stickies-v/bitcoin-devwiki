**15:00 UTC alternate Tuesdays on the #bitcoin-core-dev channel on Freenode**

Join us for a fortnightly (that's every two weeks, folks) IRC meeting to discuss P2P PRs and issues in Bitcoin Core. Meetings will be held on:

- 11 August 2020 ([log](http://www.erisian.com.au/meetbot/bitcoin-core-dev/2020/bitcoin-core-dev.2020-08-11-15.00.html))
- 25 August 2020
- 8 September 2020
- 22 September 2020
- 6 October 2020
- 20 October 2020
- 3 November 2020
- 17 November 2020
- 1 December 2020
- 15 December 2020
- 29 December 2020
- etc

[Google calendar](https://calendar.google.com/calendar?cid=MTFwcXZkZ3BkOTlubGliZjliYTg2MXZ1OHNAZ3JvdXAuY2FsZW5kYXIuZ29vZ2xlLmNvbQ) for all Bitcoin Core irc meetings.

### Format

**Maximum**: 1 hour. **Minimum**: much less if there's nothing to discuss.

### Topics

1. **Focus and priorities**: Everyone will have a chance to share what they're working on and prioritizing.

2. **????**: Feel free to suggest topics for the upcoming meeting below.

##### 11 Aug 2020

- Adam Jonas: Opt-in review begging experiment
- aj p2p things:
  - taproot relay (non-standard input, 0.19 #19681 0.20 #19680 ; wtxid relay backport #19606 ; anything else?)
  - any other wtxid cleanups?
  - tx relay overhaul #19184
  - refactors .. so many of them :-/
  - update / automate [https://github.com/users/ajtowns/projects/1](https://github.com/users/ajtowns/projects/1)
  - better p2p testing? #14210 / #19316
  - rest are backlog for me:
    - package relay? #14895
    - erlay #18261 (needs rebase! phew!)
    - does mempool count? epoch mempool stuff...
    - dynamically throttle min-fee-bump for RBF depending on network activity...
    - support absurdly low fee txs #13990
    - parallel connection attempts when starting #15502