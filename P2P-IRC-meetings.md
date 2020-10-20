**15:00 UTC alternate Tuesdays on the #bitcoin-core-dev channel on Freenode**

Join us for a fortnightly (that's every two weeks, folks) IRC meeting to discuss P2P PRs and issues in Bitcoin Core. Meetings will be held on:

- 11 August 2020 ([log](http://www.erisian.com.au/meetbot/bitcoin-core-dev/2020/bitcoin-core-dev.2020-08-11-15.00.html)), ([summary](#11-aug-2020))
- 25 August 2020 ([log](http://www.erisian.com.au/meetbot/bitcoin-core-dev/2020/bitcoin-core-dev.2020-08-25-15.00.log.html)), ([summary](#25-aug-2020))
- 8 September 2020 ([log](http://www.erisian.com.au/meetbot/bitcoin-core-dev/2020/bitcoin-core-dev.2020-09-08-15.00.log.html)), ([summary](#8-sept-2020))
- 22 September 2020 ([log](http://www.erisian.com.au/meetbot/bitcoin-core-dev/2020/bitcoin-core-dev.2020-09-22-15.00.log.html)), ([summary](#22-sept-2020)) 
- 6 October 2020 ([log](http://www.erisian.com.au/meetbot/bitcoin-core-dev/2020/bitcoin-core-dev.2020-10-06-15.00.log.html))
- 20 October 2020 ([log](http://www.erisian.com.au/meetbot/bitcoin-core-dev/2020/bitcoin-core-dev.2020-10-20-15.00.log.html)), ([summary]((#20-oct-2020))) 
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

## 03 Nov 2020

_Feel free to propose a topic for the upcoming meeting_

## 20 Oct 2020

### Topic: priorities:

  - jnewbery: Backport of wtxid relay to 0.20, [#19606](http://github.com/bitcoin/bitcoin/pull/19606)

### Topic: Remove timestamps from addr messages? It seems like the timestamp is only used to leak information about our recent connectivity. It doesn't look like we use it to make decisions about who to connect to. (sdaftuar/jnewbery)

sdaftuar and jnewbery were discussing the interaction around block-relay-only peers, where we don't want to leak any information. They observed that right now in master, we directly leak the time we connected to a block-relay-only peer after we disconnect from that peer and include the address in a getaddr response. This can be fixed, but it led to wondering what good does this timestamp provide? Bitcoin Core rarely uses these timestamps -- it is used occasionally to filter out responses to getaddr requests and used sometimes to evict things from the new table, but we do not use it for determining who to connect to. In the places where they are used, they could likely be replaced without much trouble using nlasttry and nlastsuccess.

ariard observed that even if Bitcoin Core's addrman does not use it, that doesn't mean it's not used by some other bitcoin clients to decide its peering. sdaftuar agreed but thought it might be worth asking how much good these timestamps do since they are necessarily unverifiable data. jnewbery offered that using timestamps is already a well-known way of [inferring network topology](https://www.cs.umd.edu/projects/coinscope/coinscope.pdf), and this is a tradeoff between helping a (theoretical) alternative implementation make better decisions about who to connect to versuslot protecting our own privacy. By default, we should always lean towards protecting our privacy unless doing so would be detrimental to the network as a whole. All agreed this change should be advertised on the mailing list.


### Topic: PR to fix some interactions between addrman and block-relay-only peers

After looking into how eviction works from the new and tried tables, sdaftuar decided it makes sense for block-relay-only peers to be moved to the tried table (see [#20187](https://github.com/bitcoin/bitcoin/issues/20187)). This is a reversal from his initial inclination, but there remains lots to consider, particularly related to privacy issues that are hard to reason about. One problem is whether the timestamps returned in getaddr messages for those peers will stick out. sipa added that this is a balance between not updating addrman to minimize detectability of block-only connections and updating it to make sure the node keeps good ones.

### MISC:

jnewbery thought another piece of (almost) useless data that we could stop sharing is the start_height in the version message, but it didn't fit in with today's discussion.

## 06 Oct 2020

No suggested topics.

jnewbery: "The high priority PRs right now are taproot, tx request overhaul and torv3. It'd be great if they could all land before feature freeze."

## 22 Sept 2020

### Topic: rename feelers to probes (gleb)
In #[19958](https://github.com/bitcoin/bitcoin/pull/19958), gleb suggested renaming "feeler" connections to "probes" throughout the codebase. Feelers currently capture two distinct features: feelers and test-before-evict. There was some support of renaming, but also hesitation, so gleb plans to drop the rename and focus on improving the documentation.

### Topic: Transaction propagation design goals - see #19820 (ariard)

ariard expressed concern for higher-level protocols (like Lightning) currently being designed and deployed with a dependency on implicit assumptions of the transaction relay network and mempool behavior.

sipa recognized that anything that relies on specific properties of transaction relay being guaranteed is broken. sdaftuar noted that transaction relay works well for transactions whose inputs are all confirmed though aj remarked that it might not be the case if a transaction is RBF'ing something else that was previously relayed.

sipa thought it is a good idea to better analyze and document the design goals for transactions as well as strive for useful features for common cases in higher-level protocols, but that the results of these efforts will not be guarantees.

Regarding specific solutions, ariard proposed that better documentation of the relay policy and adding something like package relay would be helpful. While sdaftuar thought that package relay still needs to be fleshed out, it led him to wonder whether it is reasonable to ask whether relay policy changes should always be documented in a BIP so that wallet authors can take those changes into account. ariard added that there is no BIP for the [CPFP carve-out](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2018-November/016518.html), for example, and that developers are reusing it beyond Lightning. Since none of the relay policy is documented today (except for BIP125), this leads to the concern of the consequences of tightening a relay policy rule when others have built on it being liberal?

jnewbery did not believe that the p2p policy belongs in BIPs, but rather that the [dev wiki](https://github.com/bitcoin-core/bitcoin-devwiki/wiki) would be an appropriate place for it. luke-jr clarified that strict policy changes don't require a BIP, but that changes that external developers should design around does belong in a BIP. sdaftuar summary of this was that wallet authors should use their best understanding of what policy rules are deployed on the network to generate transactions that will propagate well but shouldn't rely on that for security.

### Topic: Overview of [#19988](https://github.com/bitcoin/bitcoin/pull/19988) - motivation/design philosophy rather than technical details that can be found in the PR (sipa)

Bitcoin Core's transaction fetching logic has grown over time with various data structures needed for coordination and an unclear specification of what they actually implement. There are biases in favor/against fetching from certain nodes, but they are all implemented indirectly through random delays and insertions/lookups in maps that are hard to reason about. Instead, [#19988](https://github.com/bitcoin/bitcoin/pull/19988) creates a clear specification of what should be fetched. This provides something that can be defined in a simple data structure and then has one class that encapsulates an efficient implementation of it. To test it, sipa wrote a fuzz tester containing a naive reimplementation with the exact same behavior as in the efficient boost::multi_index based implementation and tries to find sequences of operations that make them diverge.

The goal of merging this before the 0.21 feature freeze on Oct 15 jnewbery noted was ambitious given that the patch is +2000/-450 LOC. However, aj added that adding this patch early in a cycle would make backporting other p2p things (i.e., from 0.22pre to 0.21) harder.

sipa noted that most of the code is fuzzing and tests, but the non-test code was hairy. sipa encourages reviewers to look at the fuzz test first. Even ignoring the fuzzing aspect, the naive reimplementation provides a good reference for what the patch *should* do.

## 8 Sept 2020

### Topic: priorities

gleb: opened 4 addr-related Prs ([#19843](https://github.com/bitcoin/bitcoin/pull/19843), [#19869](https://github.com/bitcoin/bitcoin/pull/19869), [#19906](https://github.com/bitcoin/bitcoin/pull/19906), [#19860](https://github.com/bitcoin/bitcoin/pull/19860)) over the last 10 days. Addr needs attention, and his future work is blocked. Some of them are refactoring, others have important implications and bug fixes. [#19843](https://github.com/bitcoin/bitcoin/pull/19843) is a little refactor which will unlock his future work.

jnewbery: Still encouraged review of the remaining backport [#19606](https://github.com/bitcoin/bitcoin/pull/19606). It's not a totally clean backport, but it shouldn't be impossible to review.

jonatack: Quick reminder to review the bip155 and bip324 implementation PRs and it would be great to have #19643 in master, and it seems RFM.

hebasto: [#17785](https://github.com/bitcoin/bitcoin/pull/17785).

sdaftuar: I'd love review on [#19858](https://github.com/bitcoin/bitcoin/pull/19858).

### Topic: netgroup diversity of outbound peers (sdaftuar)

This discussion was originally suggested by sdaftuar because some of the suggestions in [#19860](https://github.com/bitcoin/bitcoin/pull/19860) had some non-obvious changes. This patch prevents short-lived ADDR_FETCH and FEELER connections from affecting other peer selection and prevents BLOCK_RELAY connections from affecting other full-relay peer selection. This is because block-relay-connections should be more private (we rely on them for anti-eclipse). The disadvantage of this change is that it reduces diversity as a whole since new full-relay connections may be in the same netgroup with existent block-relay-only connections. This last point is what sdaftuar thought should be the primary consideration, as that seems like it has the most significant effect on Bitcoin Core's partition-resistance since generally speaking, more diversity is assumed to be better.

#### The attack scenario:
An adversary controlling a victim's AddrMan (e.g. by occupying all of the full-relay slots) may learn which netgroups existing block-relay-only connections are occupying. This knowledge might allow the attacker to glean who are the node's peers and possibly evict the victim from them.

PR #19860 attempts to address the scenario where a node has lost all full connections to a poisoned AddrMan, but 2 block-relay-only are still healthy. sdaftuar’s intuition was that if the AddrMan starts healthy, then the node, in the short-term, would be OK due to the [tried-table-collision resolution algorithm](https://github.com/bitcoin/bitcoin/commit/e68172ed9fd0e35d4e848142e9b36ffcc41de254#diff-546affdc61603f38c92524326c9e0bf7R537) (also known as [test-before-evict](https://github.com/bitcoin/bitcoin/pull/9037)). However, in the long-run, if a node has no healthy Addr peers, and therefore never learns of new honest nodes, this could lead to an eventual eclipse as their honest peers turnover. sdaftuar also offered the opinion that it wasn't worth optimizing for the scenario where the AddrMan is already poisoned, but the two block-relay only connections remained healthy. Since block-relay peers ignore Addr messages, healthy block-relay-only connections will not help sanitize the AddrMan.

note: jnewbery wondered whether anchor block-relay-only connections ([#17428](https://github.com/bitcoin/bitcoin/pull/17428)) might make the above scenario of compromised full connections and healthy block-relay-only connections more likely. amiti and ariard thought it might.

current protections: Each node has its own way of dividing the address space into netgroups. This is known as keyed-netgroups.

#### Goals of block-relay-only connections:
This discussion revisited the goals of block-relay-only connections. sdaftuar stated that the goals of block-only connections are twofold: the primary goal is to increase the difficulty/cost of an adversary required to carry out a network partitioning attack against a target node without requiring the bandwidth overhead of full connections. The specific motivation was that Bitcoin Core's existing full-relay connections leak topology information (and fixing that is sort difficult if not eventually hopeless).

The second goal is more amorphous. It's easier to optimize for anti-DoS/robustness/design tradeoffs if there is an individual focus on the different aspects of network traffic. For instance, the design goals around transaction relay may lead to different peering strategies than the design goals around block-only relay. Separating logic for the connection types also reduces potential privacy/exploitability concerns by preventing leak between them. sdaftuar noted, however, that it seems there is a long way to go from truly separating connection types since AddrMan remains firmly in the middle.

#### Connection diversity:
gleb brought up that block-relay-only connections weren't originally intended to improve diversity, but this ended up being a substantial side-effect benefit. sdaftuar contended that taking the diversity benefit away for the sake of privacy is hard to imagine, without a concrete understanding of where the privacy would be lost.

sipa clarified that the value of connection diversity for full + block-only connections are for partition resistance, while full connections alone add actual short-term efficient connectivity. sdaftuar added that achieving maximum diversity for blocks-only and full connections are sufficient to achieve maximum diversity for full connections alone.

#### Ideas to explore:
sipa wondered if addr-only connections would be a useful addition to block-only and transaction-relay connections to which sdaftuar offered that he had an implementation idea for syncing the chain tip with more peers by coupling it with AddrMan updates. This would result in block-relay-only connections for eclipse prevention, addr+blocks connections for low-bandwidth extra diversity, and transaction-relay connections for transaction-relay.

#### Resolution:
In this PR, gleb felt comfortable sacrificing the diversity of block-relay-only connections because he believed it had so little effect. The degradation of diversity would be that the two existing block-relay-only connections _may_ overlap with new full relay connections. If more block-relay-only connections were introduced in the future, 2 of them could be private while n - 2 of them could overlap with the full connections. jnewbery pointed out that ultimately a worse case effect of this change would be that it reduces the level of connection diversity to where it was before block-relay-only peers were introduced.

sdaftuar thought that this change's value ultimately came down to whether there is a belief that the attack of observing keyed-netgroup collisions from observed outbound connections is practical. And so, gleb suggested that we keep the current logic in place until more there are block-relay-only connections and then designate some of them to privacy and some portion of them to connection diversity.

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
  - progress on p2p testing ([#19315](http://github.com/bitcoin/bitcoin/pull/19315))
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