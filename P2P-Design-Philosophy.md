## Network partition resistance

For the Bitcoin network to remain in consensus, the network of nodes must not be partitioned. So for an individual node to remain in consensus with the network, it must have at least one connection to that network of peers that share its consensus rules. This document describes how we attempt to achieve this.

We can't rely on inbound peers to be honest, because they are initiated by others. It's impossible for us to know, for example, whether all our inbound peers are controlled by the same adversary.

Therefore, in order to try to be connected to the honest network, we focus on having good outbound peers, as we get to choose who those are.

Note that non-listening nodes have no inbound peers, while all nodes should have 10 outbound peers by default (8 of which are relaying transactions, and 2 of which are only relaying blocks/headers). If we're unable to achieve 10 outbound peers for some reason, there's a good chance we have bigger problems on our hands (ie massive network failure!). This document assumes that we have plenty of outbound peers to choose from, and doesn't address the ways we protect against not having sufficient peers to connect to.

For the sake of this document, we generally operate under the assumption that we have no useful inbound peers, as this is the most generic case (though it's worth analyzing to ensure that the presence of inbound peers can't interfere or attack the logic described).

## Ways that our outbound peers can be bad

### Case 1. Our outbound peers never announce a block to us.

If all our peers do this, then our tip may never update.  Our peers may be honest, and this can just be a case of blocks being found slowly, or our peers may have different consensus rules or are just refusing to relay valid blocks, and we should try to find new peers.  We deal with this in two ways:

#### The stale-tip check, introduced in #11560:

- Our tip is potentially "stale" if it hasn't updated in > 30 minutes. We check this on a 10-minute timer.
- If our tip is stale, we stop using our feeler connection, and instead make a 9th outbound connection.
- Every 45 seconds, check to see if we're over our 8 connection limit, and if so, disconnect the peer who least recently gave us a new block, choosing the newer connection in case of tie (and skipping over a fixed set of 4 outbound peers, which are exempt from this disconnection logic). If the peer has been connected for >30 seconds (which should be long enough to handshake and learn about any new blocks), disconnect it, and turn off using the 9th connection (ie revert to using only for feeler connections again) until the next stale tip check.
- Intuition: we only stay connected to our newest peer if they provide us with a new block after we connect to them, otherwise we close that connection pretty quickly.
- We only check again for a stale tip every 10 minutes, so generally we'll just connect once to a new 9th peer, and then disconnect (assuming our initial 8 peers don't disconnect us).
- We exempt 4 "protected" outbound peers from being disconnected under this logic, as well -- so when we look to evict an outbound peer because we have more than 8, the so-called protected peers don't get chosen.

Design consideration:

- Avoid putting too much load on the network if blocks are slow to be found. The 30 minute lag for stale tip detection is aggressive, particularly since in a period of naturally slow blocks the whole network will detect this in the same 10 minute window, and all try to connect out. The network load is mitigated because we stop using feeler connections when this is active, which means the total number of outbound connections initiated shouldn't really go up, and we stop connecting out until the next check after our first eviction.
- Eviction of the peer with least-recently-announced block is not a great general rule for managing outbound peers. If we always connected to 9 peers, and periodically evicted the one who least recently announced a block, then eventually the whole network will be competing to be connected to the fastest-block-relaying peers (eg the peers connected to a block-relay network (like FIBRE) or something equivalent).  This in turn makes it easier to sybil attack the network, say by setting up a bunch of nodes that scored well on this metric, to eventually take over all outbound connections of the network.

   We address this concern by (a) protecting 4 peers from this logic, (b) only
   require a minimum connection time of 30 seconds in order to decide that peer
   is evictable (so we won't stay connected to 9 peers for long, in theory),
   and (c) only connect and evict a 9th peer once every 10 minutes.

- Note that we do not require that our outbound peers be the ones that advance our chain, in order to remain connected to all of them. If an inbound peer is announcing blocks to us quickly, then we may be the ones that relay blocks to our outbound peers first and keep them in sync. In this situation there's nothing inherently wrong with our peering (and our links to our outbound peers may be keeping the network connected, such as if we're a big miner). Of course if we were to lose that inbound peer, and that was our only connection to the honest network, then at some point (after 30 minutes) this logic will allow us to start looking for new outbound peers to get reconnected to the honest network.

#### Periodic block-relay-only connections for syncing headers

The second way we deal with this is by periodically opening a block-relay-only connection for the purpose of syncing our tip and then disconnecting, regardless of how often our tip has been updating (since #19858):

- Every 5 minutes (on average), prioritize creating a temporary block-relay-only connection to a peer selected from our addrman (in lieu of a feeler connection).
  * If we fail to connect for any reason after picking one address to try from our addrman, give up until our 5 minute (poisson timer) goes off again.
- Stay connected long enough to sync headers (just as we do with the extra full-relay connection described above, when our tip is stale).
- Every 45 seconds, check to see if we're over our maximum number of block-relay-peers, and if we are, we potentially disconnect one:
  * Disconnect the newest block-relay-only peer if we haven't received a new block from that peer.
  * Disconnect the second-youngest block-relay-only peer if, instead, the newest peer has given us a block more recently than it.

Design considerations:

- Avoid putting too much load on the network.
  * We only try to connect once every 5 minutes to avoid holding connection slots open on the network for an inordinate amount of time.
  * We use block-relay-only connections because they are very low bandwidth.
- Allow replacing an existing connection for a new connection if we learn of a new block.  It's fairly unlikely that in normal/honest conditions that we'll learn of a new block from our newest peer (some ways that can happen: all our peers are withholding the best block from us; our peers are themselves partitioned off from the most-work chain somehow; there's a network split and we learn of a competing chain with the same work as our chain).  If it does happen, we probably are better off staying connected to the new peer so that if their chain has more work or ends up with more work that we're more likely to learn about it than from our existing peers.
- Only rotate the youngest peer, to avoid creating attack vectors where both our block-relay-only peer slots could be taken over by an entity that just tries to serve blocks the fastest.  At most one of our two peers could be taken over this way right now.
- Protect against attacks that stale-tip checking woulnd't protect against.
  * As the stale-tip check only fires if our tip hasn't updated in a while, an eclipsing adversary that is feeding us blocks slowly could prevent us from using that logic to look for honest peers.  Since these connections fire every 5 minutes regardless of what our tip is doing, as long as we have a single peer in our addrman that has the most work chain, we should eventually find it.

### Case 2. All outbound peers are on incompatible/invalid chains.

In general, if an outbound peer is on an incompatible chain, we would wish to disconnect them, so that we can make room for an outbound peer that shares our consensus, to stay connected to the honest network.

However there is some nuance to this. For instance, we don't actually need all our outbound peers to enforce the correct consensus rules in order to achieve consensus; we only need one. And there is one case where it is generally beneficial to remain connected to peers that are (temporarily!) on an invalid chain, namely when we are aware of a softfork that our peer may not be aware of, and someone has just mined a more-work but invalid block.

#### Scenario 1. An outbound peer is on an invalid chain with more work than our tip.

We generally respond to this by disconnecting the peer, though the mechanisms for doing this are diffuse. In theory we could be less aggressive (since we're only, in theory, concerned about the case that all our peers are
on invalid chains), but because historical behavior of the node is to disconnect in many cases, we've recently made changes in an attempt to make this disconnection uniform across all the ways we might discover that a peer is
on an actually invalid chain.

This list is not complete, but gives an overview of some of the recently-addressed concerns:

- A peer announces an actually invalid block that we try to connect. When we realize it is invalid, we should disconnect that peer.
- A peer announces a block header that builds on a block that we've detected is invalid. We'll disconnect the peer for building on an invalid chain.
- A peer announces a block header that has some distant parent that we detected as invalid (but the immediate parent header may not be marked invalid, due to implementation details of bad-block-marking). After #11531, this case is detected and the peer will be disconnected.
- A peer announces a block header that we already have and know to be invalid. After #11568, we'll disconnect outbound peers who give us a known-invalid block header, but not inbound peers (eg for the soft-fork reason).

There is an additional complication because under BIP 152 (compact blocks), peers are permitted to relay us blocks before fully validating them.  This means that an honest peer might temporarily appear to be on an invalid chain; however we must not disconnect them in such a case and so we have exceptions in our disconnect behavior to account for compact block relay.

In order to protect against peers using compact block announcements as a way around our protections, we rely on additional logic specific to compact block processing: headers that build on an invalid block header result in
disconnection. Thus while an invalid compact block may be received from an honest peer, a compact block that extends an invalid chain should never be sent by an honest peer under BIP 152, and so that results in disconnection.

Note: it is possible there are still bugs/omissions in our logic for disconnecting a peer who appears to be on an invalid chain, as the detection of invalid blocks/blockheaders is spread throughout the validation code, and we
may be missing a case. For now, such bugs should generally be patched up by ensuring that at least outbound peers get disconnected when relaying (not via compact blocks) a block or header that is known to be necessarily invalid. But this logic will likely be comprehensively re-evaluated in the future, and likely relaxed so that inbound peers are generally not immediately disconnected, and so that we stop banning peers and instead just disconnect them.

#### Subcategory 2. An outbound peer is on a chain with less work than ours.

We only try to switch to a peer's chain if the tip of that chain has more work than our tip.  So if a peer is on a bad chain that has less work than our tip, we may never find that out. However, if an outbound peer is on a chain that has less work than our tip for an extended time period, they are either using different consensus rules (and think our chain is invalid), or are unable to keep up (eg hardware is too slow to process blocks, or they just started up and are still syncing with the network). After #11490, we require that at least 4 of our outbound peers be able to keep up with a chain that has at least as much work as our tip.  We do this as follows:

- For each of the 4 peers not exempt from this logic, we check to see if their  best known block (based on block headers or inv messages we receive from them) has as much work as our tip.  If so, nothing happens and we clear all timers for this peer.
- If not, we set a 20 minute timer and record our current tip's work. After 20 minutes, we check again to see if their best known block has as much work as our tip from 20 minutes ago.  If it does, we set a new timer 20 minutes in the future and record our current tip's work.
- But if not, then we send a single getheaders message (with locator starting  from 20-minute-old-tip) to that peer, and give it 2 minutes to respond. If it fails to respond indicating its on a chain with sufficiently high work, then we disconnect the peer.

Design considerations:

- This logic, of requiring that a given outbound peer keep up with our tip all the time, may be too aggressive. Some reasons for being more lax: if the network is suddenly congested, or a very-slow-validate-block is generated, or if we are old software unaware of some future softfork, then we may wish to remain connected to some peers that may otherwise appear to be failing this test.
 
- Peers are made exempt from this logic (and similarly exempt from the stale tip-based eviction logic described above) if they provide headers with at least as much work as our tip at a time when fewer than 4 peers are marked as exempt.