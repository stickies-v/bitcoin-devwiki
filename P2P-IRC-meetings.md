**21:00 UTC alternate Tuesdays on the #bitcoin-core-dev channel on Freenode**

Join us for an IRC meeting every two weeks to discuss P2P PRs and issues in Bitcoin Core. The next meeting will be on:

- 09 March 2021 at 21:00 UTC

Previous meetings:

<details><summary>Meeting Archive</summary>

- 23 February 2021 ([log](https://bitcoin.jonasschnelli.ch/ircmeetings/logs/bitcoin-core-dev/2021/bitcoin-core-dev.2021-02-23-21.00.log.html))
- 09 February 2021 ([log](http://www.erisian.com.au/bitcoin-core-dev/log-2021-02-09.html#l-325)), ([summary](#09-feb-2021))
- 26 January 2021 ([log](https://bitcoin.jonasschnelli.ch/ircmeetings/logs/bitcoin-core-dev/2021/bitcoin-core-dev.2021-01-26-15.00.moin.txt))
- 12 January 2021 ([log](https://bitcoin.jonasschnelli.ch/ircmeetings/logs/bitcoin-core-dev/2021/bitcoin-core-dev.2021-01-12-15.00.moin.txt)), ([summary](#12-jan-2021)) 
- 15 December 2020 ([log](https://bitcoin.jonasschnelli.ch/ircmeetings/logs/bitcoin-core-dev/2020/bitcoin-core-dev.2020-12-15-15.01.moin.txt))
- 1 December 2020 ([log](https://bitcoin.jonasschnelli.ch/ircmeetings/logs/bitcoin-core-dev/2020/bitcoin-core-dev.2020-12-01-15.00.moin.txt)), ([summary](#1-dec-2020)) 
- 17 November 2020 ([log](https://bitcoin.jonasschnelli.ch/ircmeetings/logs/bitcoin-core-dev/2020/bitcoin-core-dev.2020-11-17-15.00.moin.txt)), ([summary](#17-nov-2020)) 
- 3 November 2020 ([log](http://www.erisian.com.au/meetbot/bitcoin-core-dev/2020/bitcoin-core-dev.2020-11-03-15.00.log.html)), ([summary](#04-nov-2020)) 
- 20 October 2020 ([log](http://www.erisian.com.au/meetbot/bitcoin-core-dev/2020/bitcoin-core-dev.2020-10-20-15.00.log.html)), ([summary](#20-oct-2020)) 
- 6 October 2020 ([log](http://www.erisian.com.au/meetbot/bitcoin-core-dev/2020/bitcoin-core-dev.2020-10-06-15.00.log.html))
- 22 September 2020 ([log](http://www.erisian.com.au/meetbot/bitcoin-core-dev/2020/bitcoin-core-dev.2020-09-22-15.00.log.html)), ([summary](#22-sept-2020)) 
- 8 September 2020 ([log](http://www.erisian.com.au/meetbot/bitcoin-core-dev/2020/bitcoin-core-dev.2020-09-08-15.00.log.html)), ([summary](#8-sept-2020))
- 25 August 2020 ([log](http://www.erisian.com.au/meetbot/bitcoin-core-dev/2020/bitcoin-core-dev.2020-08-25-15.00.log.html)), ([summary](#25-aug-2020))
- 11 August 2020 ([log](http://www.erisian.com.au/meetbot/bitcoin-core-dev/2020/bitcoin-core-dev.2020-08-11-15.00.html)), ([summary](#11-aug-2020))
</details>

[Google calendar](https://calendar.google.com/calendar?cid=MTFwcXZkZ3BkOTlubGliZjliYTg2MXZ1OHNAZ3JvdXAuY2FsZW5kYXIuZ29vZ2xlLmNvbQ) for all Bitcoin Core irc meetings.

### Format

**Maximum**: 1 hour. **Minimum**: much less if there's nothing to discuss.

### Topics

1. **[Focus and priorities](https://github.com/bitcoin-core/bitcoin-devwiki/wiki/P2P-Current-Priorities)**: We'll discuss what people are working on and prioritizing.

2. **????**: Feel free to suggest topics for the upcoming meeting below.

## 9 Mar 2021

- **addrman and eclipse attack [doc](https://github.com/bitcoin-core/bitcoin-devwiki/wiki/Addrman-and-eclipse-attacks)** (ajonas)

## 23 Feb 2021

- **Erlay update** (gleb)
- **Orphan reprocessing** (jnewbery)
  - see discussion about orphan processing in PR 21224 ([here](https://github.com/bitcoin/bitcoin/pull/21224#issuecomment-781741925) onwards)
    - [Previous IRC discussion](http://www.erisian.com.au/bitcoin-core-dev/log-2020-07-02.html#l-325)
    - [Move orphan reprocessing to global](https://github.com/bitcoin/bitcoin/pull/19364)
    - [Add txorphanage](https://github.com/bitcoin/bitcoin/pull/21148)
- **Peer rate-limiting** (jnewbery)
  - see discussion about peer rate-limiting in PR 21224 ([here](https://github.com/bitcoin/bitcoin/pull/21224#issuecomment-781567152) onwards)
- **Dandelion update** (ajtowns)

## 09 Feb 2021


## Topic: State of Erlay implementation (gleb)

[#18261](https://github.com/bitcoin/bitcoin/pull/18261) needs a couple more high-level code review to evaluate the approach. The next step would be to split the PR into five or so parts and merge one by one, but gleb would prefer a high-level check first to ensure the code structure overall makes sense and there aren't any implementation-specific blockers.

sipa mentioned that he addressed some of the comments in [minisketch#28](https://github.com/sipa/minisketch/pull/28) (updated: merged Feb 19, 2021).

## Other updates and requested review

amiti announced that she has a PR open to introduce new node-level rebroadcast functionality at [#21061](https://github.com/bitcoin/bitcoin/pull/21061) and would appreciate review.

jonatack encouraged reviewers to test and review the [I2P PR](https://github.com/bitcoin/bitcoin/pull/20685) that adds the invisible internet project to the P2P networks. jonatack and wumpus have been running I2P nodes without any problems. vasild planned on getting [#20788](https://github.com/bitcoin/bitcoin/pull/20788) merged first (update: merged on Feb 11, 2021), which would simplify [#20685](https://github.com/bitcoin/bitcoin/pull/20685).

aj asked for review on [#20942](https://github.com/bitcoin/bitcoin/issues/20942) (update: merged on Feb 15, 2021)

## 26 Jan 2021

_Nothing discussed_

## 12 Jan 2021

### Topic: P2P priorities

vasild: [#20788](https://github.com/bitcoin/bitcoin/issues/20788), which would help #19203 and #20685 to move forward  
jonatack: [#20197](https://github.com/bitcoin/bitcoin/pull/20197) and adding more unit test coverage to the eviction logic  
jnewbery: progress [#19398](https://github.com/bitcoin/bitcoin/issues/19398). He is currently waiting for [#20811](https://github.com/bitcoin/bitcoin/issues/20811) because he thinks the remaining changes are less disruptive after that.  
ariard: reviewing erlay/package testmempoolaccept and updating [#20277](https://github.com/bitcoin/bitcoin/issues/20277) with the latest feedback from sdaftuar

### Topic: disabletx P2P message (sdaftuar)

[#20726](https://github.com/bitcoin/bitcoin/pull/20726), [mailing list post](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2021-January/018340.html), [Optech Newletter](https://bitcoinops.org/en/newsletters/2021/01/13/#proposed-disabletx-message). (sdaftuar plans to mark #20726 ready for review once a BIP number is assigned.)

This proposal's overarching goal is to increase the number of inbound connection slots on the network to feel good about increasing the number of block-relay-only connections. Block-relay-only connections add security to the network by increasing its partition resistance at relatively low resource cost. But to do that, we need to give nodes a way to know that an inbound peer is a block-relay-only peer. Currently, block-relay-only peers use the `fRelay` flag from BIP37 to instruct their peers they don't want transactions, but BIP37 allows for transaction relay to resume with a FILTERCLEAR message.

This proposes adding a new p2p message (`disabletx`), which indicates that a connection will not relay transactions for its lifetime. This, in turn, allows us to write code to increase the number of connections by reserving additional slots for disabletx-peers.

amiti wondered whether the current proposal is a disabletx message that explicitly disables transactions and implicitly disables addresses with the thinking is that in the future, we could introduce a message that explicitly enables addresses. sdaftuar responded that block-relay-only connections currently have no way to communicate that they also don't want addr messages, and his proposal is to have the BIP "RECOMMEND" that we not send addrs to peers sending disabletx
with the idea being that in the future, we should adopt some kind of addr relay negotiation protocol, which would then take precedence. However, sdaftuar believes that designing an addr relay protocol will take some work, and he isn't ready to propose one now until a few questions around what the goals of addr relay should be and how best to achieve them are more clearly defined. For example, the long term design might be more of a toggle (like with transactions), and we might want to communicate information about particular networks (as described in BIP 155). It remains unclear what the different handling should be for different networks (i.e., addresses we understand versus ones that we don't).

vasild pointed out that transaction relay is unrelated to addr relay and that linking both together under disabletx would be confusing. sdaftuar responded that both are related to block-relay-only peer logic, which is the only thing that will be using this at the start. Software that ignores the addr-relay suggestion will not be in violation of the design, but it accommodates our desired behavior today. vasild wondered whether relayblocksonly instead of disabletx more clear (relay only blocks and nothing else - no tx, no addr). sdaftuar pointed out that was his first approach, but [received feedback](https://github.com/bitcoin/bitcoin/pull/20726#discussion_r548352366) that having a BIP that governed too much behavior was also confusing or bad protocol design for future compatibility. The current approach is narrow and just specifies required behavior for transaction-relay but provides an implication for other behavior in the absence of protocol support. Ultimately, sdaftuar thinks neither proposal would inhibit his implementation nor future protocol extensions.

sdaftuar posed to the group the question of whether nodes should relay addrs to software sending disabletx. He opined that relaying addrs to inbound-block-relay-only peers would be a problem and strictly worse than not relaying addrs. One of the drawbacks to increasing the number of inbound-block-relay-only peers is that we create addr-relay black holes (peers that don't connect the graph), which will prevent the addition of more block-relay connections. This was originally discussed in [#15759](https://github.com/bitcoin/bitcoin/pull/15759#issuecomment-480400958), which was one of the reasons that only two outbound block-relay-only connections were added at that time.

jnewbery asked whether sdaftuar had thoughts about what a future addr relay negotiation method would look. sdaftuar's naive guess is that adding some kind of feature negotiation where we signal support (not sure 0/1 or if more precision is necessary) for different networks (as defined in BIP 155) would be desirable. However, he thinks there needs to be a discussion on addr relay goals, come up with some relay policies to address those goals, and then make sure the design supports those relay policies. There is currently no writeup anywhere on how addr relay should work, and addr relay remains an understudied topic. Without further research, his guess about a future addr relay negotiation method would be difficult to defend. That said, he thought progress on adding block-relay-only connections would be possible while deferring the design of an actual addr relay protocol to the future. In the meantime, he wants to make sure that people are comfortable with not relaying addrs to disabletx peers.

## 15 Dec 2020

### Topic: Review of the P2P Review Process (ariard)

Which PR reviews stand-out as productive and which were unproductive ? What can we learn from these examples ?

Review-intensive P2P PRs on last 12 months (non-exhaustive):
* [#16702](https://github.com/bitcoin/bitcoin/pull/16702)
* [#17428](https://github.com/bitcoin/bitcoin/pull/17428)
* [#18038](https://github.com/bitcoin/bitcoin/pull/18038)
* [#18044](https://github.com/bitcoin/bitcoin/pull/18044)
* [#18876](https://github.com/bitcoin/bitcoin/pull/18876)
* [#18991](https://github.com/bitcoin/bitcoin/pull/18991)
* [#19107](https://github.com/bitcoin/bitcoin/pull/19107)
* [#19219](https://github.com/bitcoin/bitcoin/pull/19219)
* [#19301](https://github.com/bitcoin/bitcoin/pull/19031)
* [#19316](https://github.com/bitcoin/bitcoin/pull/19316)
* [#19398](https://github.com/bitcoin/bitcoin/issues/19398)
* [#19858](https://github.com/bitcoin/bitcoin/pull/19858)
* [#19954](https://github.com/bitcoin/bitcoin/pull/19954)
* [#19988](https://github.com/bitcoin/bitcoin/pull/19988)


## 1 Dec 2020

### Topic: p2p fuzzing (michaelfolkson)

### Topic: Transaction pinning / package relay (ariard)

Another case of transaction-pinning [was recently found](https://bitcoinops.org/en/newsletters/2020/09/16/#stealing-onchain-fees-from-ln-htlcs) after the merging of [new anchor spec](https://github.com/lightningnetwork/lightning-rfc/pull/803). The attack is explained in detail [here](https://github.com/t-bast/lightning-docs/blob/master/pinning-attacks.md).

sipa opined that DoS protection will always result in an inability to accept transactions in some scenarios where that transaction would have been accepted from p2p in another state. We can reduce the set of situations that can happen using better algorithms, but the problem in a generic sense seems inherent. ariard agreed and pointed to his [June mailing list post](https://lists.linuxfoundation.org/pipermail/lightning-dev/2020-June/002758.html#:~:text=Scenario%203).

cdecker added that alternative transport over the lightning overlay is not intended to be a complete solution. It just reduces the effectiveness of pinning by making it less likely to succeed. He hoped that it'd be attractive for miners to listen to the overlay since the feerate is higher (even though they don't beat the absolute fee, which is not the miner's primary motivation). If there's interest from miners to get access cdecker offered to write up a faux-bitcoin-node that acts as a bridge between bitcoin p2p and the ln overlay relay network.

### Topic: Erlay BIP updates (gleb)

Sometimes reconciliation fails initially because the first sketch is insufficient or because it's too small and allows to decode up to N differences. If it fails, the Erlay protocol makes another attempt. There are two independent alternatives to do this while still being 100% efficient: bisection and extension — the original proposal used bisection because it spent fewer CPU cycles on computing sketches. In practice, the implementation turned out to be too complicated on the Bitcoin Core p2p protocol side. So gleb switched the code to do sketch extensions instead. It's much less code, the code complexity is now more aligned with general Bitcoin project complexity, and, if need be, it's also easier to extend to allow to make multiple attempts. It's a bit more CPU expensive doesn't matter because we expect sketches of low capacity. For more rationale, see the [updated BIP](https://github.com/bitcoin/bips/pull/899).

## 17 Nov 2020

### Priorities for 0.22

sdaftuar suggested erlay to which ariard, jonatack agreed  

ariard is waiting for [#19160](https://github.com/bitcoin/bitcoin/pull/19160] before proceeding with altnet  

jonatack mentioned BIP 324 implementation to which troygiorshev agreed  

jnewbery seeks to progress clarifying the net/net_processing and net_processing  

sdaftuar plans to work on block-relay-only peering, though he is stuck on some annoying addr-relay things that might hold him up  

troygiorshev reminded reviewers about per-Peer Message logging [#19509](https://github.com/bitcoin/bitcoin/issues/19509)  

ariard plans to work on a better version of [#18797](https://github.com/bitcoin/bitcoin/issues/18797) and better understand transaction-standardness before going back to package relay  

amiti hopes to make progress on [#19315](https://github.com/bitcoin/bitcoin/issues/19315) (adding full-relay and block-relay-only to tests) and reviving transaction rebroadcast  

(Note: these were individual's [priorities in August](https://github.com/bitcoin-core/bitcoin-devwiki/wiki/P2P-IRC-meetings#topic-individual-priorities))

### Topic: Touch base on wtxid backport (aj)

[#20399](https://github.com/bitcoin/bitcoin/issues/20399) proposes to revert wtxid relay from 0.20 ([#19606](https://github.com/bitcoin/bitcoin/pull/19606)) and [#20317](https://github.com/bitcoin/bitcoin/issues/20317) is a fix up of the orphan handling regression.

sdaftuar thought that he wasn't sure that we should have backported wtxid-relay to the 0.20 branch. Knowing that there was a crashing bug in that line of work heightens the sense that backporting a feature is not a great idea. He didn't believe that backporting it was a requirement in the first place, but saw the reasoning that it was a nice-to-have. Given that it indeed turned out to be risky, he was in favor of dropping it, as there's no compelling reason for backport after [#19620](https://github.com/bitcoin/bitcoin/issues/19620) either. Falling back on the principle of only backporting bugfixes seems like the prudent thing to do.

jnewbery noted that people agreed in the original meeting that it should have been backported, and then he has raised reviewing the PR in several meetings since then without objection. He didn't think the presence of a bug that was caught in review and fixed in the follow-up changes the facts but could understand that it makes people nervous.

Even so, it was decided that [#20399](https://github.com/bitcoin/bitcoin/issues/20399) should be done to revert the backport and [#20317](https://github.com/bitcoin/bitcoin/issues/20317) would be closed.

### Topic: Reducing [CVE-2020-26895](https://cve.mitre.org/cgi-bin/cvename.cgi?name=2020-26895) class of bugs and Tx-standardness. (ariard)

_Prior to v0.10.0-beta, a malicious peer could force an lnd node to accept a high-S ECDSA signature when updating new off-chain states. Though the signatures are valid according to consensus rules, the mempool policy would reject transactions containing high-S values, potentially leading to loss of funds if time-sensitive transactions cannot be relayed and confirmed. Transaction-relay policy is an area of high-concern for off-chain protocols. How to best mitigate against this class of bugs in the future remains an open question. Building out a libtxstandardness library to make the tx-standardness verification available to other applications might be a solution._

In some cases, LND would accept a transaction signature that Bitcoin Core would not relay or mine by default. When the transaction containing the unrelayable signature fails to confirm, a timelock eventually expires, and the attacker is able to steal funds they previously paid to the vulnerable user. ([source bitcoin optech newsletter](https://bitcoinops.org/en/newsletters/2020/10/28/#cve-2020-26895-acceptance-of-non-standard-signatures))

ariard started by apologizing for the previous discussion of this issue since it was difficult to explain before the public disclosure. This is not the first time that a Lightning Network implementation has had issues with tx-standardness. C-Lightning [had an issue](https://github.com/bitcoin/bitcoin/issues/13283) a couple of years ago with minimum relay fees. ariard contended that there is a class of vulnerabilities with any time-sensitive protocols built on top of the base layer, and ways to mitigate it correctly remain unclear.

luke-jr proposed be strict on the transaction form. One cannot rely on node/relay policies, so doing what one can to pass them seems like the best option.

While `testmempoolaccept` only tells you about your own policy, sdaftuar offered a belt and suspenders approach to enforce a strict transaction form and double-check against testmempoolaccept. Though luke-jr pointed out that double-checking against your own node makes sense, it ultimately shouldn't be your security check.

ariard also brought up that mobile LN nodes won't have a mempool but are required to verify that the chain of transactions built from a particular UTXO (commitment transaction + HTLC transaction) are transaction-relay compliant. He worried that your full-node ultimately decides the validity of LN transactions that both you and your counter-party must agree to.

ariard speculated that in the future, we might have fixed-fee commitment transactions and their feerate adjusted by a CPFP+package relay, to which aj added that we'd expect testmempoolaccept to support packages (which glozow is currently working on). testmempoolaccept currently requires all inputs to be available in mempool or UTXO set, so if you're testing validity further down a chain, it won't work. jnewbery agreed that testmempoolaccept does require the full UTXO set, but one could imagine a version where you provide the inputs, like `signrawtransaction`. Still, the concern remains whether all the mempool checks would be satisfied if the checks were abstracted into its own module/library, especially since some important ones are inside the script interpreter.

luke-jr argued that this is not possible to have certainty since nodes will relay what they want, and there are no consensus rules forcing relay rules. He and sdaftuar were clear that there is no such thing as "bitcoin's tx relay rules." Their suggestion was to narrow what is accepted to a small enough subset that it will likely always be relayable both now and in the future. Testing validity against various full-node implementations would be a nice backup to make sure the assumptions aren't broken or that the code hasn't changed. aj pointed out that this isn't very robust against hostile peers who have access to your codebase to find checks you forgot to implement and is also hard to fuzz test for valid-but-non-standard sigs and the like.

The meeting closed with ariard announcing his intention to rework [#18797](https://github.com/bitcoin/bitcoin/issues/18797) as a check on top of ATMP.

## 03 Nov 2020

### Topic: addrman file versioning (jnewbery)

While reviewing [#19954](https://github.com/bitcoin/bitcoin/pull/19954#discussion_r515607619), jnewbery realized that old versions of peers.dat were not guaranteed to fail to parse after the upgrade to 0.21. vasild opened #20284 to address the issue. This mitigation is needed so that a user can upgrade, make changes to peers.dat and then downgrade (at least one version) and still parse the peers.dat. sdaftuar suggested to rename the peers.dat file for 0.21, and migrate data from the old file to the new one though this strategy would require updating a lot of documentation. jnewbery floated the possibility of moving the peers.dat to sqlite.

A previous change [#16702](https://github.com/bitcoin/bitcoin/pull/16702) partially broke backward-compatibility (users upgrading to v0.20 had records from the new table deleted). #16702 also means that whenever a new asmap is provided, records from the new table will be deleted.

### Topic: I2P support, some background at [vasild/bitcoin/wiki/I2P-connectivity](https://github.com/vasild/bitcoin/wiki/I2P-connectivity) (vasild)

Sdaftuar asked for clarification on how I2P and Tor addresses get gossiped to peers that support the network type. Sipa replied that I2P/cjdns do not get rumored by the current code (see [#20119](https://github.com/bitcoin/bitcoin/issues/20119)).

Reachable networks get rumored 2x, unreachable ipv4/ipv6/torv2/torv3 get rumored 1.5x, and unreachable others do not get rumored at all (see [#19728](https://github.com/bitcoin/bitcoin/issues/19728)). [#20254](https://github.com/bitcoin/bitcoin/issues/20254) makes I2P reachable. Tor remains unreachable and has no from address. (It was also mentioned that I2P has P2P encryption by default!)

Sdaftuar asked how it is decided what address to advertise as given the node is reachable in multiple ways? Sipa responded that among the local addresses compatible with that peer, the one that has received the most mentions in incoming version messages. (This reminded sipa that Tor v3/I2P/cjdns can't be put in version messages, so those will never receive any mentions). Wumpus added that there was the idea to add that address in the 'sendaddrv2' message. Sdaftuar suggested that if there is an attempt to bootstrap a new type of address, it is necessary to actively advertise that address even to peers who don't understand it so that eventually, peers that do support it will receive it. vasild noted that it was decided not to relay I2P addresses by nodes that are connected to I2P (at least in the 0.21 release).

## 20 Oct 2020

### Topic: priorities:

  - jnewbery: Backport of wtxid relay to 0.20, [#19606](http://github.com/bitcoin/bitcoin/pull/19606)

### Topic: Remove timestamps from addr messages? It seems like the timestamp is only used to leak information about our recent connectivity. It doesn't look like we use it to make decisions about who to connect to. (sdaftuar/jnewbery)

sdaftuar and jnewbery were discussing the interaction around block-relay-only peers, where we don't want to leak any information. They observed that right now in master, we directly leak the time we connected to a block-relay-only peer after we disconnect from that peer and include the address in a getaddr response. This can be fixed, but it led to wondering what good does this timestamp provide? Bitcoin Core rarely uses these timestamps -- it is used occasionally to filter out responses to getaddr requests and used sometimes to evict things from the new table, but we do not use it for determining who to connect to. In the places where they are used, they could likely be replaced without much trouble using nlasttry and nlastsuccess.

ariard observed that even if Bitcoin Core's addrman does not use it, that doesn't mean it's not used by some other bitcoin clients to decide its peering. sdaftuar agreed but thought it might be worth asking how much good these timestamps do since they are necessarily unverifiable data. jnewbery offered that using timestamps is already a well-known way of [inferring network topology](https://www.cs.umd.edu/projects/coinscope/coinscope.pdf), and this is a tradeoff between helping a (theoretical) alternative implementation make better decisions about who to connect to versus protecting our own privacy. By default, we should always lean towards protecting our privacy unless doing so would be detrimental to the network as a whole. All agreed this change should be advertised on the mailing list.


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