**15:00 UTC alternate Tuesdays on the #bitcoin-core-dev channel on Freenode**

Join us for a fortnightly (that's every two weeks, folks) IRC meeting to discuss P2P PRs and issues in Bitcoin Core. Meetings will be held on:

- 11 August 2020 ([log](http://www.erisian.com.au/meetbot/bitcoin-core-dev/2020/bitcoin-core-dev.2020-08-11-15.00.html)), ([summary](#11-aug-2020))
- 25 August 2020 ([log](http://www.erisian.com.au/meetbot/bitcoin-core-dev/2020/bitcoin-core-dev.2020-08-25-15.00.log.html)), ([summary](#25-aug-2020))
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

### 8 Sept 2020
_Please update this wiki page with proposed topics!_

Follow-up on "What would a good transaction propagation framework look like? See a first draw Transactions propagation design goals [#19820](https://github.com/bitcoin/bitcoin/issues/19820) (ariard)

## 25 Aug 2020

### Topic: priorities

backports: 
  - 0.19: Add txids with non-standard inputs to reject filter [#19681](http://github.com/bitcoin/bitcoin/pull/19681)
  - 0.20: Add txids with non-standard inputs to reject filter [#19680](http://github.com/bitcoin/bitcoin/pull/19680)
  - 0.20: Backport wtxid relay [#19606](http://github.com/bitcoin/bitcoin/pull/19606)
  - [19569 comment](github.com/bitcoin/bitcoin/pull/19569#issuecomment-667116307) (to be opened by jnewbery)
  - fanquake commented that there is no rush on 0.20.2, but MarcoFalke noted that [#19681](http://github.com/bitcoin/bitcoin/pull/19681) is the last of the p2p 0.19.2 backports and the release could be shipped once that is merged.

jonatack: BIP155 ([#19628](http://github.com/bitcoin/bitcoin/pull/19628)), BIP324 ([#18242](http://github.com/bitcoin/bitcoin/pull/18242)), [#19610](http://github.com/bitcoin/bitcoin/pull/19610)
  - BIP324 has a new detailed comment from [LLFourn to review](https://gist.github.com/jonasschnelli/c530ea8421b8d0e80c51486325587c52#gistcomment-3428675)

hebasto: Unify Send and Receive protocol versions ([#17785](http://github.com/bitcoin/bitcoin/pull/17785)) and Block-relay-only Anchor Connections ([#17428](http://github.com/bitcoin/bitcoin/pull/17428))

troygiorshev: Per-Peer Message Logging ([#19509](http://github.com/bitcoin/bitcoin/pull/19509)), Move all header verification into the network layer, extend logging ([#19107](http://github.com/bitcoin/bitcoin/pull/19107)), and addrv2 ([#19031](http://github.com/bitcoin/bitcoin/pull/19031))

amiti: Allow outbound & block-relay-only connections in functional tests ([#19315](http://github.com/bitcoin/bitcoin/pull/19315)) and progress on Block-relay-only Anchor Connections ([#17428](http://github.com/bitcoin/bitcoin/pull/17428)). Reviewers have agreed on the benefits of #17428, but momentum slowed down on some of the specifics.

nehan: a PR on vRecvGetData and orphan_work_set coming soon.

ajonas: keeping track of [#18044](http://github.com/bitcoin/bitcoin/pull/18044) follow-ups.

sdaftuar:

##### Feature negotiation:
After [discussion on the mailing list](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2020-August/018084.html), sdaftuar commented that he now has clarity on how to unobjectionably propose new feature negotiation: accompany new features with a protocol version bump but otherwise copy what has been done before. While forcing protocol changes to be serial seems not as good as being able to opt into different features, we can deal with that when we actually get protocol version number conflicts (which hasn't happened in practice). It is difficult to force other implementations to adopt anything generic, so it's easy for now to accommodate those preferences.

##### Distinguish incoming block-relay-only connections:
sdaftuar expressed that the above-simplified negotiation of block-relay only connections at peer connection time allows both sides of a connection to devote fewer resources to each other, resulting in per-peer memory savings. This improvement can hopefully lead to an increase in the number of inbounds (specifically carving out extra inbound slots for block-relay peers). In turn, it could lead to an increase in the number of block-relay outbound connections that each node can accommodate. This something that sdaftuar plans to pick up in the near term.

##### Syncing headers with feeler-peers:
Additionally, sdaftuar has small topology improvements in mind, specifically picking up on the implementation of syncing headers with feeler-peers discussed in [#16859](http://github.com/bitcoin/bitcoin/pull/16859). The main idea is to sync headers with more peers, introducing a new peer type (headers-sync-peer), which gets quickly dropped and potentially replace peers with new ones as well. sdaftuar hopes to have a PR to come "soon-ish."

### Topic: Goals for 0.21 (50 days/7 weeks until feature freeze)
  - tx overhaul ([#19184](http://github.com/bitcoin/bitcoin/pull/19184)), sipa will open review again soon
  - addrv2 ([#19031](http://github.com/bitcoin/bitcoin/pull/19031)) - confirmed by the author to be mostly implementation work (desirable for 0.21 essential for 0.22 since Tor v2 deprecation begins Sept 15, 2020 & will be removed July 15, 2021)
  - progress on p2p testing ([#19035](http://github.com/bitcoin/bitcoin/pull/19035))
  - erlay ([#18261](http://github.com/bitcoin/bitcoin/pull/18261)) may have to wait until 0.22

### Topic: service flags on signet and problems with tx relay of time-sensitive transactions

aj has been working on supporting "regular" peers and "taproot-enabled" (and "anyprevout-enabled") peers for signet, indicated via service flags. Allocating service flags on signet seems to solve the problem okay on signet, but he is trying to see if there's a better solution that usefully generalizes to something helpful on mainnet. aj has some code available on [a branch](https://github.com/ajtowns/bitcoin/commits/signet-tr) but isn't yet soliciting review.

sipa added that with better separation between block and transaction connections (or block-only and full connections), it might be possible to apply preferential peering only on the tx-carrying connections. ariard worried about policy requirements knowledge by higher layers and its impact on Lightning. sdaftuar noted that many things can go wrong with respect to poorly propagated transactions, and the addition of automatic tx rebroadcast and tx-relay-peer rotation is the best way to address.

<it was noted that the meeting got a bit sidetracked in this section and there was lots of crosstalk - see [log for more details](http://www.erisian.com.au/meetbot/bitcoin-core-dev/2020/bitcoin-core-dev.2020-08-25-15.00.log.html#l-73)>
 
### Misc
Is anyone working on?
  - overhauling peer misbehavior (e.g., measure resource usage by peers and penalize those peers consuming more of ours?) (asked by jonatack)
  - eviction testing? (gated by [#19035](http://github.com/bitcoin/bitcoin/pull/19035)) (asked by jonatack, endorsed by ariard/amiti)

What would a good transaction propagation framework look like? (asked by amiti)
  - sdaftuar: I think that we don't even know exactly what our tx-relay goals are, let alone how to measure our progress against those goals, or how to improve things. sipa's writeup ([#19031](http://github.com/bitcoin/bitcoin/pull/19031)) is great for what we want a single node to do, but what we want overall network behavior to look like remains a big unknown.

## 11 Aug 2020

Date: 2020-08-11 15:00:07 UTC  
Host: jnewbery

### Topic: Individual priorities

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

### Topic: Opt-in review begging experiment

ajonas read over [core dev tech transcript on priorities from 2018](https://diyhpl.us/wiki/transcripts/bitcoin-core-dev-tech/2018-03-07-priorities/), which lists some possibilities for how to better coordinate review. Since then, he has experimented with asking for reviews directly. [The results](https://docs.google.com/spreadsheets/d/1INEN1RrZTsu-V4GH6kr0aVhFOVY8nGgx0ajHf3NEYlc/) should be taken with a grain of salt. It was also noted that merge speed may not be a very good metric, which ajonas agreed with.

ajonas expressed interest in expanding his experiments if any contributors would like to opt-in to being nagged by him to review. You can DM him if you'd like to volunteer.

A related comment by fanquake was that "ACK recap" comments can sometimes be misleading. There be confusion as to why a PR which looks like it has *lots* of ACKs, maybe after a [PR review-club](https://bitcoincore.reviews/), hasn't been merged. (amiti, jonatack, and ajonas disagreed and/or asked for further clarification)

### Topic: feature negotiation (new bip proposal from sdaftuar)

sdaftuar plans to send a new [BIP proposal](https://github.com/sdaftuar/bips/blob/2020-08-generalized-feature-negotiation/bip-p2p-feature-negotiation.mediawiki) to the mailing list soon [edit: [mailing list post](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2020-August/018084.html)].

To summarize, [wtxid-relay](https://github.com/bitcoin/bitcoin/pull/18044) uses a new feature-negotiation method (exchanging messages between version and verack) that this BIP proposes to codify for future use. To do so, we need to ensure software on the network knows to ignore unknown messages pre-verack, which is the undocumented de facto standard today.

Bitcoin Core historically has disallowed unknown messages pre-verack. This BIP establishes that unknown messages should not cause peer disconnections, which could be problematic for network topology if we get this wrong in the future even if those unknown messages are before verack.

Additionally, this BIP disentangles protocol version bump from feature negotiation by allowing feature signaling between version/verack. The aim is to make this a standard way for features that need to negotiate at connection startup time. In particular, this negotiation method would lend itself to block-relay only connections.

sdaftuar noted that he might update [the draft]((https://github.com/sdaftuar/bips/blob/2020-08-generalized-feature-negotiation/bip-p2p-feature-negotiation.mediawiki)) to NOT further bump the version number to signal support. Software that chooses not to implement wtxid-relay should already be adopting this proposed BIP (if they bump their version number to 70016 or higher at any point).