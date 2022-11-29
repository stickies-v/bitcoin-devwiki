# Eclipse attacks

Eclipse attacks occur when a node’s access to information can be filtered by other malicious nodes in the network. That could be because the node is not connected to any honest peers on the network, and instead, its peer connections are controlled by an adversary. Once denied connectivity to the honest network, a victim can be attacked in numerous ways, such as with double-spends or funds loss on layer 2 systems (such as the Lightning Network). Mining nodes attacked in this way can be forced to waste hashpower, be commandeered in selfish mining attacks, or generally aid in causing forks on the network. 

This document attempts to describe the mechanisms implemented in Bitcoin Core to mitigate eclipse attacks followed by open questions and areas of further research.

## Restart-based eclipse attack

A restart-based eclipse attack occurs when the adversary is able to sufficiently saturate the victim's address manager with attacker IPs (a.k.a. addrman flooding), and then the victim restarts. If the attack succeeds, the victim will lose its current outgoing connections due to the restart and be more likely to make all of its connections to the adversary's addresses. Node restarts are common. They could be caused by natural behavior, a software upgrade (which could be predicted in the case of a widely-publicized bug), or could be triggered by an adversary in the presence of a remotely triggered vulnerability.

## The address manager in Bitcoin Core

The Bitcoin peer-to-peer layer supports messages that allow nodes to gossip the network addresses of other nodes on the network. This allows new nodes coming online to learn of other peers, and new listening nodes (that are able to take incoming connections) can have their address gossiped so that other nodes can connect to them later.

Bitcoin Core tracks addresses in an "addresses manager", known as [AddrMan](https://github.com/bitcoin/bitcoin/blob/77a2f5d30c5ecb764b8a7c098492e1f5cdec90f0/src/addrman.h#L25-L54), introduced in [#787](https://github.com/bitcoin/bitcoin/pull/787). AddrMan maintains network addresses (e.g. IP and onion) of potential bitcoin nodes on the network. While the implementation has undergone several changes, namely increasing the number of buckets for each table, the essential design remains the same.

Bitcoin node's sybil resistance requires having at least one honest peer which is not eclipsed itself, which provides connectivity to the honest network, and thus able to learn of newly mined blocks in some timely fashion.

### Assumption of a healthy addrman from the start

When a node first comes online, it is at the mercy of the DNS seeds (or alternatively by placing trust in the developers by using the hard-coded seeds included in each release) and the first connections it makes by querying for addresses. If fed bad addresses from the start, a fledgling node has little chance of collecting honest peers' addresses. Therefore, addrman assumes that the peer tables are unpoisoned before an attack is initiated.

We limit the trust in any single DNS seed by querying multiple DNS seeds when starting up with an empty addrman.

### How addresses are stored

Nodes store the IP addresses discovered by received [addr](https://developer.bitcoin.org/reference/p2p_networking.html#addr) messages in two tables: the "new" table and the "tried" table. The design goal of the address manager is to ensure that no attacker can fill the entire table with their nodes/addresses. A [comment in addrman.h](https://github.com/bitcoin/bitcoin/blob/be3af4f31089726267ce2dbdd6c9c153bb5aeae1/src/addrman.h#L97) outlines both the address manager's design goals and the bucketing method.

Addresses that have not yet been tried go into 1024 "new" buckets. The address group (either the source's AS if using [asmap](https://github.com/bitcoin/bitcoin/pull/16702) is enabled or /16 for IPv4 if not) of the source (the peer that gossiped the addr to us) is used to select 64 buckets out of the possible 1024, and the destination address is used to select the specific bucket and position, using a cryptographic hash function. A single address can occur in up to 8 different buckets to increase selection chances of frequently seen addresses. If the selected bucket and position is occupied, it uses some rules to determine if the new addr should be stored and the old addr should be evicted or if the new addr should be discarded ([code](https://github.com/bitcoin/bitcoin/blob/e08f3193b543017702d000c2263bccbefa981c14/src/addrman.cpp#L314)). The new table stores network addresses that could potentially contain invalid or malicious addresses and could contain network addresses that are not Bitcoin nodes or even unallocated IP addresses.

Addresses of nodes known to be an accessible bitcoin node go into 256 "tried" buckets. Each address range selects eight of these buckets at random. The actual bucket is chosen from one of these, based on the full address. Bucket selection is based on cryptographic hashing, using a randomly-generated 256-bit key. This key should remain private since an adversary with knowledge of a victim’s bucket selection could more easily target peer addresses to be evicted.

When attempting to add a new good address to an occupied bucket/position in the "tried" table, a short-lived connection attempts to connect to the existing entry. If the connection is unsuccessful, then the older address is evicted back to the "new" buckets and the new address is stored in the "tried" table. (For more, see [Countermeasure #3: Test-before-evict](/#countermeasure-3-test-before-evict---9037).)

## Peer seeding sources

Peers are discovered from six sources:

#### 1. Loaded from peers.dat

A node can use its pre-existing knowledge of possible peers recorded in `peers.dat`.

#### 2. DNS Seeds

This is the most common method for a brand new Bitcoin node to discover peers for the first time. A node can use DNS seeds to populate the addrman when its addrman is empty or if passing in the `-forcednsseed` argument. DNS seeds are [domains](https://github.com/bitcoin/bitcoin/blob/86a8b35f321d55bb2381ea56bcc1cdd17c7896e6/src/chainparams.cpp#L121) owned by Bitcoin contributors that do two things: (1) expose subdomains that DNS resolve to IP addresses of potential peers and (2) run minimal nodes capable of `version`/`verack` handshakes and `addr` gossip ([code](https://github.com/bitcoin/bitcoin/blob/094402430925ec5aac6edbbf52d74f10c665da43/src/net.cpp#L1541-L1549)). Usually, a node will query these seeds via the DNS protocol, which uses a different port from the bitcoin protocol.

If our node is behind a [proxy](https://github.com/bitcoin/bitcoin/blob/417f95fa453d97087a33d4176523ab278bef21a1/src/net.cpp#L1742-L1743) (or if the seeder [does not support service bit filtering](https://github.com/bitcoin/bitcoin/blob/417f95fa453d97087a33d4176523ab278bef21a1/src/net.cpp#L1763-L1767) over DNS), the seeders are treated as an [AddrFetch connection](https://github.com/bitcoin/bitcoin/blob/417f95fa453d97087a33d4176523ab278bef21a1/src/net.h#L174-L180). (For more on `AddrFetch` connections, see 4 below). Nodes will engage in the Bitcoin protocol handshake and addr gossip with the seeder. The connection will disconnect soon after.

There are downsides to DNS seeding. DNS seeding leaks to the ISP that the node is a running bitcoin node. Also, DNS sources can provide useless or malicious addresses. Moreover, even if the DNS request results were cryptographically signed, developers are trusted not to attack nodes starting up in the future by, for example, relegating all newcomers to a separate network and taking advantage of them later.

To prevent a single malicious seeder from dominating addrman by announcing large numbers of IP addresses, [the number of IPs each seeder can contribute is limited to 256](https://github.com/bitcoin/bitcoin/blob/086226d98ae8c194a0a38b2fbfffd0dc3773e879/src/net.cpp#L1685).

#### 3. Fixed seeds

In the case that DNS seeds aren’t responding, as a fallback, the node attempts to expand the peers in addrman via fixed seeds hardcoded in [chainparamsseeds.h](https://github.com/bitcoin/bitcoin/blob/417f95fa453d97087a33d4176523ab278bef21a1/src/chainparamsseeds.h) [code](https://github.com/bitcoin/bitcoin/blob/417f95fa453d97087a33d4176523ab278bef21a1/src/net.cpp#L1887-L1900). This hardcoded list contains addresses of recently active nodes on the network and is updated every release cycle. [#18506](https://github.com/bitcoin/bitcoin/pull/18506) is an example of the process to update the fixed seeds. Connections to fixed seeds are established as ADDR_FETCH connections, used for addr collection and then disconnected.

#### 4. AddrFetch connections via `-seednode`

`AddrFetch` connections are short-lived connections that are established for the sole purpose of populating the addrman ([source](https://github.com/bitcoin/bitcoin/blob/417f95fa453d97087a33d4176523ab278bef21a1/src/net.h#L174-L180)). These ADDR_FETCH connections can also be initiated by explicitly passing in `-seednode` addresses. or under certain circumstances with DNS seeds (i.e., [here](https://github.com/bitcoin/bitcoin/blob/417f95fa453d97087a33d4176523ab278bef21a1/src/net.cpp#L1742-L1743) and [here](https://github.com/bitcoin/bitcoin/blob/417f95fa453d97087a33d4176523ab278bef21a1/src/net.cpp#L1763-L1767)).

#### 5. Manual connection with `-connect` configuration

If the user runs the node with this configuration, `bitcoind` will only ever make outbound connections with nodes specified in `-connect`. For example, this might be useful in enterprise settings where internal nodes all only talk to a gateway node, which is responsible for staying in sync with the public Bitcoin network. Manual connections cannot be evicted and do not contribute to the limits of outbound-full-relay and outbound-block-relay.

#### 6. Manual connection with `-addnode` configuration or using the `addnode` RPC

Bitcoind will make an outbound connection to a specified node, and engage in addr relay with the peer. As new peers are discovered, more outbound connections will be established. Manual connections cannot be evicted and do not contribute to the limits of outbound-full-relay and outbound-block-relay.


## Addr gossip

Once a node has peers established through any of the above seeding techniques, it engages in addr relay to grow its addrman.

### Unsolicited `addr` messages*

Peers share data with each other by gossiping unsolicited addresses. After establishing a connection, the initiating node sends its own address to the new peer, which then forwards it to a couple more peers in the network. Additionally, nodes will resend their address to their peers on average once every 24 hours ([source](https://github.com/bitcoin/bitcoin/blob/816314ef0f7bdf50a6596ef893ac1a1d2d8723bf/src/net_processing.cpp#L4130-L4131)). Those peers will batch these in maximum sets of 10 ([source](https://github.com/bitcoin/bitcoin/blob/816314ef0f7bdf50a6596ef893ac1a1d2d8723bf/src/net_processing.cpp#L2619-L2623)) and send them to a limited group of their peers who haven't been relayed that address before ([code](https://github.com/bitcoin/bitcoin/blob/d0852f39a7a3bfbb36437ef20bf94c263cad632a/src/net_processing.cpp#L1636-L1638)).

*Note: this description of addr relay behavior only describes Bitcoin Core. There are no BIPs specifying addr relay, and therefore, the behavior of other Bitcoin software on the network is purely local policy.

### From responses to `getaddr` messages

`getaddr` data provides another way to populate the addrman. After the initial `version`/`verack` handshake, a node sends a `getaddr` message requesting information about known active peers to help find potential nodes in the network ([source](https://github.com/bitcoin/bitcoin/blob/094402430925ec5aac6edbbf52d74f10c665da43/src/net_processing.cpp#L2424-L2425)). The receiving node proportionally samples addresses from the new and tried tables and responds with an addr message containing up to 1,000 addresses capped at 23% of its own addrman size ([code](https://github.com/bitcoin/bitcoin/blob/094402430925ec5aac6edbbf52d74f10c665da43/src/addrman.cpp#L482-L505)).

Past research has exploited `getaddr` messages by sending them to all possible nodes on the network, then comparing these messages to infer the possible edges of each node. The [Coinscope paper](https://www.cs.umd.edu/projects/coinscope/coinscope.pdf) took advantage of long-lived connections and used timestamps proliferated through the network to infer topology. This behavior was addressed in version 10.1 and further improved by [#14897](https://github.com/bitcoin/bitcoin/pull/14897), which randomized the request order and introduced a bias toward outbound connections giving them a higher priority when a node chooses from whom it should request a transaction. [#18991](https://github.com/bitcoin/bitcoin/pull/18991) cached responses to `getaddr` to further prevent topology leaks.

## Timestamps

Addrman also tracks the timestamps of peers, currently connected or announced by other peers. A timestamp is kept for each address to keep track of when the node address was last seen. Nodes keep newly received addresses in memory and wait two hours before adding them to the `peers.dat` database. However, if the address received is in the database, the timestamp will be updated with the new one if it is more recent ([source](https://core.ac.uk/download/pdf/288502346.pdf)).

When successfully connecting to a peer, the `CAddress`'s nTime is updated, which is gossiped to peers. `net_processing` also updates this timestamp when it disconnects from a peer to prevent leaking information about currently connected peers ([code](https://github.com/bitcoin/bitcoin/blob/e669c3156ff88627a9478be8a6cac12723c2614a/src/addrman.h#L284-L294)).
The above timestamps should not be confused with the `nLastSuccess` timestamp on `CAddrInfo` that updates when we mark an [address as Good](https://github.com/bitcoin/bitcoin/blob/e669c3156ff88627a9478be8a6cac12723c2614a/src/addrman.cpp#L215).

Note that timestamps can be modified, and malicious nodes may change them or even set them in the future. As mentioned above, leaking timestamp information has been used to infer topology information by network spies. [Recent discussions](https://github.com/bitcoin-core/bitcoin-devwiki/wiki/P2P-IRC-meetings#topic-remove-timestamps-from-addr-messages-it-seems-like-the-timestamp-is-only-used-to-leak-information-about-our-recent-connectivity-it-doesnt-look-like-we-use-it-to-make-decisions-about-who-to-connect-to-sdaftuarjnewbery) have questioned the usefulness of `nTime` timestamps, raising the possibility that they could be removed altogether.

## Deployed Countermeasures for eclipse attacks

The eclipse attack paper recommends 10 countermeasures. Deployed countermeasures are presented below in chronological order of when they were merged into Bitcoin Core.

### Countermeasure 4 (feeler connections - [#8282](https://github.com/bitcoin/bitcoin/pull/8282))

Heilman, et al. found that a large percentage of addresses in tried tables are stale IP addresses (ranging from 72% to 95% stale), which increases the risk of eclipse attacks.

To mitigate this, Bitcoin Core introduced feeler connections, which is a short-lived outbound connection that pings an address in a new table to check if it is online before transferring it to the tried table. Feelers confirm addresses are Bitcoin nodes by doing the VERSION/VERACK handshake before disconnecting. This test ensures that the tried table is replenished with a steady supply of recent online addresses.

To limit the feeler connections' network impact, nodes make one new connection at most every two minutes. Compared with other networking tasks that bitcoind performs, the bandwidth increase is very slight. Additionally, there is a random sleep of between 0 and 1000 milliseconds prior to making a feeler connection to avoid synchronization issues. To avoid threading issues, feeler connections are made in the same thread as non-feeler connections.

### Countermeasure 3 (test-before-evict - [#9037](https://github.com/bitcoin/bitcoin/pull/9037))
Before storing an address in its deterministically-chosen slot in a bucket in the tried table, check if there is an older address stored in that slot. If so, a short-lived connection, also [referred to as a feeler connection](https://github.com/bitcoin/bitcoin/blob/e669c3156ff88627a9478be8a6cac12723c2614a/src/net.h#L147-L160), attempts to connect to the older address, and if the connection is successful, then the older address is not evicted. The new address is stored in the tried table only if the connection fails.

Another small side advantage of test-before-evict is that no more than ten addresses can be in the test buffer at once. Addresses are only cleared one at a time from the test buffer, so an attacker is forced to wait at least two minutes to insert a new address into the tried table after filling up the test buffer. This rate limits an attacker attempting to launch an eclipse attack.

### Countermeasure 1 (Deterministic random eviction - [#5941](https://github.com/bitcoin/bitcoin/pull/5941))

When adding an address to the new or tried tables, each address deterministically hashes to a single slot in a single bucket. Before this change, an attacker could increase the number of addresses stored by repeatedly inserting the same address in multiple rounds. This update gave each address a single fixed location in the new and tried tables, which become simple fixed-size arrays instead of sets and vectors.

### Countermeasure 2 (Random selection - [214154e](https://github.com/bitcoin/bitcoin/commit/214154e))

The Eclipse paper's attack exploited bitcoin core’s heavy bias towards initiating  outgoing connections to addresses with fresh timestamps. This advantage was eliminated when addresses were selected at random from tried and new tables. With this change, a 50% exploitation success rate requires the adversary to fill 91.7% of the tried table.

### Countermeasure 6 (More buckets - [1d21ba2](https://github.com/bitcoin/bitcoin/commit/1d21ba2))

As more buckets are added, an infrastructure attacker needs to increase the number of groups in order to expect to fill the same fraction of the tried table. More buckets were added, and the addresses saved per group were scaled. The [last increase was in 2015](https://github.com/bitcoin/bitcoin/commit/1d21ba2f5ecbf03086d0b65c4c4c80a39a94c2ee). Revisiting the optimal number of buckets and considering larger new and tried tables may have to scale in proportion to the growth of the network. (See open question #7 for further discussion.)

A note from the eclipse paper is that this countermeasure is helpful only when the tried table already contains many legitimate addresses, so that attacker owns a smaller fraction of the addresses in tried. However, if tried is mostly empty (or contains mostly stale addresses for nodes that are no longer online), the attacker will still own a large fraction of the tried table addresses, even though the number of tried buckets has increased.

### Countermeasure 5 (Anchor connections - [#17326](https://github.com/bitcoin/bitcoin/pull/17326) and [#17428](https://github.com/bitcoin/bitcoin/pull/17428))

Inspired by Tor [entry guard rotation rates](https://www-users.cs.umn.edu/~hoppernj/single_guard.pdf), the eclipse attack paper recommends adding connections that persist between restarts, known as anchor connections. These long-lasting connections make eclipse attack more expensive for eclipse attackers but at the cost of privacy concerns of leaking network topology or transaction sources.

Anchor connections were introduced by PR [#17326](https://github.com/bitcoin/bitcoin/pull/17326) and merged into the 0.21 release. (Note that the eclipse attack paper suggested adding anchor connections for full connections, but this PR scopes anchor connections to block-relay-only connections to help with the privacy concerns stated above.)

Another tradeoff is that persistent long-lived connections would enable a snooping attacker to passively capture persistence of connections, potentially making topology inference more powerful. Strong persistence can contribute to network self-partitioning (e.g., long distance links are less reliable, so they get culled, and eventually disconnected subgraphs might only connect to their own continent) ([source](https://github.com/bitcoin/bitcoin/issues/17326#issuecomment-550521262)).
This added behavior should probably pair with a complementary behavior on the inbound side. As of now, about half the inbound slots are preserved for longest-connected peers. Half of those could be redirected to be preserved for peers with the longest-historically-connected time. Without some measure like this, persistent connection logic could be undermined by an attacker that fills the connection slots up on long-running static IPed nodes in order to cause the eviction of (or prevent connections from) the other hosts they hope to eclipse ([source](https://github.com/bitcoin/bitcoin/issues/17326#issuecomment-550521262)).

### Countermeasure 8: Ban unsolicited ADDR messages

Before, unsolicited ADDR messages of greater than 10 [addresses were accepted](https://github.com/bitcoin/bitcoin/blob/83e4670fd7cde3f9624d91208885e868fda6658f/src/net_processing.cpp#L2628) but not relayed. A node could choose not to accept large unsolicited ADDR messages from incoming peers and only solicit ADDR messages from outgoing connections when its new table is near empty. This prevents adversarial incoming connections from flooding a victim's new table with useless or malicious addresses. The tradeoff would be the slower propagation of addresses from new nodes across the network.

At large, this was addressed by PR [#22387](https://github.com/bitcoin/bitcoin/pull/22387) (included in the 22.0 release), which introduced a rate limit mechanism for unsolicited ADDR messages, making it harder for an attacker to flood the victim's new table. The rate limit does not distinguish between inbound and outbound peers.

### Other Countermeasure: Disallow inbounds from tried tables
While this was not among the countermeasures recommended in the eclipse attack paper, [PR #8594](https://github.com/bitcoin/bitcoin/pull/8594) disabled that incoming peers could add their address to the tried tables just by connecting to the node. The actual attack presented in the eclipse paper used this method to populate the tried tables with attacker-controlled addresses while filling the new table with trash IPs. After #8594, addresses would only be added to tried when the connection was initiated by the node, making it much harder for an attacker to gain influence over the tried tables.

## Undeployed (or partially deployed) recommended countermeasures

### Countermeasure 7: More outgoing connections

The eclipse attack paper recommended adding, "additional outgoing connections without risking that the network will run out of connection capacity."

Version 0.19 added two outbound block-relay-only connections via [PR #15759](https://github.com/bitcoin/bitcoin/pull/15759), which do not relay or process transactions or addr messages. (See related discussion on the impact of [address propagation through the network](https://github.com/bitcoin-core/bitcoin-devwiki/wiki/P2P-IRC-meetings#topic-disabletx-p2p-message-sdaftuar).)These add further robustness by increasing connectivity with minimal CPU/memory/bandwidth tradeoffs. Also, since fingerprinting transactions is a common technique to infer network topology, block-relay-only connections are much harder to observe than their transaction-relay counterparts.

When considering the addition of more outbound connections by default, there exists a fundamental tradeoff between resource minimization and robustness to peer misbehavior. Adding more connectivity to the network graph makes Bitcoin's network more robust (e.g., to eclipse or partition attacks), but at the cost of more resource utilization. The Erlay proposal presents a promising path to reduce the bandwidth required for these connections, allowing more connections while maintaining resource requirements.


### Countermeasure 9: Diversify incoming connections

A Bitcoin node can have all of its incoming connections come from the same IP address, making it far too easy for a single computer to monopolize a victim’s incoming connections during an eclipse attack or connection-starvation attack. The eclipse attack paper suggests a node accepts only a limited number of connections from the same IP address.

The current way incoming connections are diversified is by prioritizing incoming connections evictions from the largest netgroup. Keyed-netgroups is the mechanism by which each node uniquely divides the address space into netgroups. Asmap has been proposed as a way to further improve IP bucketing in addrman (issue: [#16599](https://github.com/bitcoin/bitcoin/issues/16599) and PR: [#16702](https://github.com/bitcoin/bitcoin/pull/16702), [blog post](https://blog.bitmex.com/call-to-action-testing-and-improving-asmap/)). Instead of relying on /16 prefixes to diversify the connections every node creates, a node would rely on the IP to ASN mapping, if such a mapping is provided.

### Countermeasure 10: Anomaly detection

The eclipse attack can have several specific "signatures'' that make it detectable, including: (1) a flurry of short-lived incoming TCP connections from diverse IP addresses that send large ADDR messages containing "trash" IP addresses. (2) An attacker that suddenly connects a large number of nodes to the network. (3) As could one that uses eclipsing to decrease the network's mining power dramatically. Monitoring and anomaly detection systems that look for this behavior would be useful. They would, at the very least, force an eclipse attacker to attack at a lower rate or waste resources on overwriting new, rather than useless, IP addresses.

## Open questions and areas for research

#### 1. Examining the rules for evicting an incoming connection

A DoS connection exhaustion attack is when an adversary fills all available incoming connection slots on the network, then overtakes the outbounds for a victim just starting up. A [patch in 2015](https://github.com/bitcoin/bitcoin/pull/6374) fixed this by enabling new incoming connections to evict old incoming connections. However, this patch introduced the possibility of an attacker purposely evicting connections. For example, because a trusted outgoing connection is another node's untrusted incoming connection, an attacker with knowledge of a node's topology inference could evict this connection. Therefore, an attacker might be able to eclipse a node without reboots via connection eviction logic ([source](https://github.com/bitcoin/bitcoin/issues/17326#issuecomment-548410548)).
Changes to incoming connection eviction logic to mitigate a connection exhaustion attack are described as follows:
  - Bitcoin Core in 2015:
    - Bob has 116 incoming connections (125 total connections minus 8 outgoing connections minus 1 feeler connection).
    - Alice makes an outgoing connection to Bob.
    - Bob has 117 incoming connections
    - Carol attempts to make an outgoing connection to Bob
    - Carol's connection is rejected.
    - Alice still has an outgoing connection to Bob

  - Bitcoin Core 2020:
    - Bob has 114 incoming connections (125 total connections minus 8 outgoing full connections minus 2 block-only relay connections minus 1 feeler connection).
    - Alice makes an outgoing connection to Bob.
    - Bob has 115 incoming connections
    - Carol attempts to make an outgoing connection to Bob
    - There is a chance Alice's connection is evicted, and Carol's connection is established
    - Alice loses her outgoing connection to Bob (sometimes)
    - Carol now has an outgoing connection (sometimes)

The change described above presents a tradeoff. Nodes coming online are able to find receptive peers for their outgoing connections. However, an adversary with network topology inference is potentially more powerful under today's eviction ruleset ([code](https://github.com/bitcoin/bitcoin/blob/a35a3466efd187a2e443aaa230472c8c22f5cfc3/src/net.cpp#L917)) because an attacker could target the eviction of incoming connections of a victim node's peer (i.e., the victim node's outgoing connection).

Further research is needed to determine:

a) How feasible is it for incoming connections to be displaced by new incoming connections by an attacker? If feasible, this has serious consequences for someone who has knowledge of the network topology since they can target a victim's outbound connections via the peer's node.

b) For a connection exhaustion attack, at some point, the attacker may just be evicting itself and (only possibly) locking new connections out. How many IP addresses are needed to perform this connection exhaustion attack, and what netgroups or geographic locations would be required?
 
#### 2. Timestamps on addr messages:

The [Coinscope paper](https://www.cs.umd.edu/projects/coinscope/coinscope.pdf) used addr messages to infer network topology. Since then, changes have been made, including PR #18991, which introduced a mechanism to reduce the ability to scrape the address manager. However, there are likely more leaks and a smarter way to communicate timestamps. There may be the possibility of a spam attack using malicious timestamps as well. Some have proposed doing away with these timestamps altogether. (See [October 20th, 2020, P2P IRC meeting](https://github.com/bitcoin-core/bitcoin-devwiki/wiki/P2P-IRC-meetings#topic-remove-timestamps-from-addr-messages-it-seems-like-the-timestamp-is-only-used-to-leak-information-about-our-recent-connectivity-it-doesnt-look-like-we-use-it-to-make-decisions-about-who-to-connect-to-sdaftuarjnewbery) for more discussion.)

#### 3. Defending against bad actors when a node first comes online

As discussed in the discussion on Assumption of a [Healthy Addrman from the Start](#assumption-of-a-healthy-addrman-from-the-start), nodes are most vulnerable when they first come online. Can we harden the seeders? Should we use authentication? What are the vulnerabilities of the DNS caching?

#### 4. Effects on the network of outbound peer rotation

The effects of outbound peer rotation have both positive and negative effects on peers. Outbound peer rotation is good for transaction-relayed peers as it improves privacy and makes topology inference more difficult. However, outbound peer rotation is bad for block-only-relayed peers as it increases the risk of a node being eclipsed.

Nodes make new full-relay connections when the tip is stale, disconnecting after waiting a short time to see if we learn a new block. Also, [#19858](https://github.com/bitcoin/bitcoin/pull/19858) makes eclipse attacks more difficult by regularly initiating outbound connections and staying connected long enough to sync headers and potentially learn of new blocks. If the node learns of a new block, it rotates out an existing block-relay peer in favor of the new peer. Since block-relay connections use minimal bandwidth, nodes can make these connections regularly and not just when the tip is stale.

Further discussion can be found at [#4723](https://github.com/bitcoin/bitcoin/pull/4723), [#15759](https://github.com/bitcoin/bitcoin/pull/15759), and the [mailing list](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2014-August/006502.html)).

#### 5. Resources estimation given the current mitigations in place

Heilman, et al. cited ~9000 IP addresses were required to complete an attack successfully. Given the suggested countermeasures implemented as well as other changes deployed, a new model is needed to understand the resource requirements of successfully exploiting a victim.
 
#### 6. Heuristics for “quality” peers besides online presence

Should the feeler connection poll the chain tip of peers similar to the outbound peer rotation logic? What is the cost of adding this logic?

#### 7. The optimum bucket sizes and size of the new and tried tables

How is the growth of the network reflected in these bucket sizes and new and tried tables? Are there enough addresses to fill up these tables? Would smaller buckets lend themselves to a more defensible position? 

The problem we face is that the network changes in size and we might not be able to predict these changes. What is a safe number given the unpredictability?