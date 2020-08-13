**15:00 UTC alternate Tuesdays on the #bitcoin-core-dev channel on Freenode**

Join us for a fortnightly (that's every two weeks, folks) IRC meeting to discuss P2P PRs and issues in Bitcoin Core. Meetings will be held on:

- 11 August 2020 ([log](http://www.erisian.com.au/meetbot/bitcoin-core-dev/2020/bitcoin-core-dev.2020-08-11-15.00.html)), [summary](#11-aug-2020)
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

#### 25 Aug 2020

_Please update this wiki page with proposed topics!_

#### 11 Aug 2020

Date: 2020-08-11 15:00:07 UTC  
Host: jnewbery

##### Topic: Individual priorities

aj:
  - taproot relay (non-standard input, 0.19 #19681 0.20 #19680 ; wtxid relay backport #19606 ; anything else?)
  - any other wtxid cleanups?
  - tx relay overhaul #19184
  - refactors
  - update / automate [project board](https://github.com/users/ajtowns/projects/1)
  - better p2p testing? (#14210 / #19316)
  - rest are backlog for me:
    - erlay #18261
    - package relay? (#14895)
    - does mempool count? epoch mempool (#18191)
    - dynamically throttle min-fee-bump for RBF depending on network activity...
    - support absurdly low fee txs (#13990)
    - parallel connection attempts when starting (#15502)
 - thinking on big changes priority-wise is: tx relay overhaul (#19184), then erlay (#18261);  package relay (#14895) is back to the drawing board.

sdaftuar:
  - transaction download stuff (#19184)
  - network-topology improvements:
    - more block-relay only peers (which is probably gated on negotiating block-relay connections at connect-time)
    - more improvements to syncing our tips with more peers (possibly including tx-relay-peer rotation, which can help here as well)
    - improved eviction logic (#19670)
    - other stuff is related to transaction relay policy, particularly package relay (#14895), which is a whole beast of a topic by itself

jonatack:
  - refactoring/cleanup
  - for a couple of weeks working on inbound eviction policy methodology was (1) an observation dashboard (#19643), 2) test coverage, and 3) optimization
  - asked by a few devs to consider picking up [bip324 implementation](https://gist.github.com/jonasschnelli/c530ea8421b8d0e80c51486325587c52). Talked with jonas schnelli  and he will be back on it soon.
  - longer term: BIP155/addrv2, BIP324, BIPs340-342, BIP325

troygiorshev:
  - moving header verification from net_processing to net (#19107)
  - reviewing addrv2 (#19031) - tor v2 deprecation (15 sept) and obsolescence (july 2021) is quickly approaching, addrv2 is needed before we can update to tor v3.

dongcarl:
  - focused on the PRs populating the Peer struct (#19607)

amiti:
  - simplifying how we track different types of connections (#19316)
  - enable more p2p testing (#19315)
  - make my way back to the rebroadcast work
  - Reviewing #19670. Also on my list are #17428 (anchors), #19184 (tx logic overhaul). and the per-peer stuff #19509 & #19607 is also interesting

ariard:
  - AltNet (#18988) (pending on ryanofsky multiprocess)
  - package relay (#14895)/RBF pinning
  - tx-relay peer rotation
  - review transaction request overhaul/erlay/others

sipa:
  - addressing some feedback in tx overhaul (#19184)
  - helping with BIP324 efforts and addrv2
  - outbound peer rotation also sounds interesting

jnewbery:
  - short-term focus - backports -- reviewed #19680 and #19681. Backported wtxid relay in #19606 (which should also be backported to v0.20). Have a branch that backports the orphan relay stuff on top of that (can add to v0.20 if that makes it easier for reviewers)
  - Longer-term - progress on #19398, where the main goal is to clarify the interface between net and net_processing, while not expanding the scope of cs_main (and then eventually reduce the scope of cs_main by moving data into the new Peer struct). First PR in that series is #19607.

vasild:
  - BIP155 / addrv2 (#19031)

ajonas also created a [longer list organized by theme](https://gist.github.com/adamjonas/85137e2623f12450f1978d291a28d680).

##### Topic: Opt-in review begging experiment

ajonas read over [core dev tech transcript on priorities from 2018](https://diyhpl.us/wiki/transcripts/bitcoin-core-dev-tech/2018-03-07-priorities/), which lists some possibilities for how to better coordinate review. Since then, he has experimented with asking for reviews directly. [The results](https://docs.google.com/spreadsheets/d/1INEN1RrZTsu-V4GH6kr0aVhFOVY8nGgx0ajHf3NEYlc/) should be taken with a grain of salt. It was also noted that merge speed may not be a very good metric, which ajonas agreed with.

ajonas expressed interest in expanding his experiments if any contributors would like to opt-in to being nagged by him to review. You can DM him if you'd like to volunteer.

A related comment by fanquake was that "ACK recap" comments can sometimes be misleading. There be confusion as to why a PR which looks like it has *lots* of ACKs, maybe after a [PR review-club](https://bitcoincore.reviews/), hasn't been merged. (amiti, jonatack, and ajonas disagreed and/or asked for further clarification)

##### Topic: feature negotiation (new bip proposal from sdaftuar)

sdaftuar plans to send a new [BIP proposal](https://github.com/sdaftuar/bips/blob/2020-08-generalized-feature-negotiation/bip-p2p-feature-negotiation.mediawiki) to the mailing list soon.

To summarize, [wtxid-relay](https://github.com/bitcoin/bitcoin/pull/18044) uses a new feature-negotiation method (exchanging messages between version and verack) that this BIP proposes to codify for future use. To do so, we need to ensure software on the network knows to ignore unknown messages pre-verack, which is the undocumented de facto standard today.

Bitcoin Core historically has disallowed unknown messages pre-verack. This BIP establishes that unknown messages should not cause peer disconnections, which could be problematic for network topology if we get this wrong in the future even if those unknown messages are before verack.

Additionally, this BIP disentangles protocol version bump from feature negotiation by allowing feature signaling between version/verack. The aim is to make this a standard way for features that need to negotiate at connection startup time. In particular, this negotiation method would lend itself to block-relay only connections.

sdaftuar noted that he might update [the draft]((https://github.com/sdaftuar/bips/blob/2020-08-generalized-feature-negotiation/bip-p2p-feature-negotiation.mediawiki)) to NOT further bump the version number to signal support. Software that chooses not to implement wtxid-relay should already be adopting this proposed BIP (if they bump their version number to 70016 or higher at any point).