Text taken directly from slides used for talk by Suhas:

Mempool and Mining 
The dynamic duo 

The Mempool: Storage for unconfirmed transactions 
Why do we have a mempool? 
* Miners need it for transaction selection 
* Non-mining nodes can also benefit, from: 
* * Fee estimation 
* * Insight into future blockchain (“is this transaction I received likely to confirm at some point?”) 
* * Participating in transaction relay, while limiting relay-DoS attacks 
* But non-mining nodes don’t “need it”. Only need blocks to stay in consensus. 

What should the Bitcoin Core mempool do? 
My opinion: 
* Primary purpose is to serve mining 
* Secondary purpose is to help with fee estimation, and performance optimizations around block relay/validation 
* Avoid making it consensus critical (so that at worst, bugs may cause a miner to have problems constructing new blocks, but wouldn’t cause the whole network to split). 
I’m sure there are others out there who disagree with this view, so take it with a grain of salt! 

What does the Bitcoin Core mempool do? 
Originally the mempool was just a dumb storage container (just a map: txid -> CTransaction), and its consistency with the tip was not always guaranteed. 
Now the mempool is always consistent with the current chain tip: 
* Every tx in mempool should be valid to include in next block 
* If we have a tx we must also have any unconfirmed ancestors that the transaction depends on. 
The mining code relies on this consistency for performance optimization during transaction selection (“CreateNewBlock”). 

Other mempool features 
* Updating consistency as block chain advances 
* * Remove transactions from mempool when they are confirmed 
* * Remove conflicting transactions from mempool (if any) after every new block 
* * Remove no-longer-valid transactions that could result from reorgs 
* Dynamic memory tracking: how much memory is it using? 
* Mempool limiting: an algorithm for determining what transactions to accept and which ones to evict when the mempool is using too much memory 
* Fee estimation hooks -- provide the fee estimator with the data it needs 
* Multiple tx sort orders, particularly: 
* * Feerate with descendants 
* * Feerate with ancestors 
* * Arrival time 

What problem does mempool limiting try to solve? 
In the old days, there was no cap on the size of the mempool. 
In the summer of 2015(?), we went through a “spam” attack, where someone appeared to be generating tons of low feerate transactions on the network, tying up memory and causing some low-memory nodes to crash. 
We needed a way to limit the memory used by the mempool.... Sounds easy! Just (a) measure the size, and (b) when it gets full, just, um, do what exactly? 
What should get added to the mempool, and what should get removed, when the mempool fills up? 

One approach 
Bitcoin XT (if I remember right) took this rough approach: 
When the mempool is at its transaction limit, randomly evict an existing one when a new one arrives. 
* Network DoS-able: adversary can use network’s relay resources for low cost 
* Gamable: An adversary can cheaply get everyone’s transactions evicted 
* Incompatible with miner incentives: no miner would ever want to randomly evict a high feerate transaction in favor of a low one! 

Bitcoin Core’s approach 
Limit the mempool while trying to still maximize miner income. 
Also avoid network relay attacks, where an adversary can cheaply/freely relay large amounts of traffic on the p2p network. 

Mempool limiting: high-level strategy 
If size(mempool) > limit after accepting a new transaction: 
* Trim the mempool down to its limit by evicting transactions with lowest feerate, including all descendant transactions. Repeat until mempool is within its size limit. 
* Idea: the lowest feerate transaction (including descendants), is very unlikely to be mined anytime soon. 
* Then don’t allow new transactions in to the mempool, for a while, unless their feerate is above the feerate of the transactions you most recently evicted. 

How could this algorithm be attacked? 
Free relay attack: 
* Create a low feerate transaction T. 
* Send zillions of child transactions that are slightly higher feerate than T until mempool is full. 
* Create one small transaction with feerate just higher than T’s, and watch T and all its children get evicted. Total fees in mempool drops dramatically! 
* Attacker just relayed (say) 300MB of data across the whole network but only pays small feerate on one small transaction. 

How can we mitigate the attack? 
“Package” limits 
* Limit number/size of descendants of a single transaction (101kb, 25 transactions) to prevent the amount of “free relay” from being too large. 
* If package size is small compared to mempool size, then attacker bears more costs to do this kind of attack. 

Mining: optimizing transaction selection 
Why do we have code that generates candidate blocks in Bitcoin Core? Surely miners have the money to solve this problem...? 
Goal is for mining to be as easy and commodity-like as possible, to promote mining decentralization. 
If having a better transaction selection algorithm is valuable for mining success, then that will promote centralization (only the really smart miners will be profitable, perhaps). 
So: let’s get some hopefully smart people to make such transaction selection software freely available to all. 

Ancestor fee-rate mining 
This is the current algorithm that we use in Bitcoin Core (as of 0.13.0). 
Some people call this “child-pays-for-parent” mining. CPFP is really a wallet strategy though; the goal of any optimal miner strategy would be to maximize fee income for a miner, taking into account transaction dependencies. 
So all reasonable mining strategies should be “CPFP” strategies -- hence it’s not a great name. 
“Ancestor feerate mining” is what I call the algorithm that selects transactions based on their feerate with ancestors... 

How does ancestor feerate mining work? 
High-level algorithm: 
1. Sort the mempool by feerate-with-unconfirmed-ancestors (yay, the mempool does this; not a coincidence). 
1. Look at the highest-scoring transaction. Add it and all unconfirmed ancestors to the block (sorting those ancestors topologically), if they fit in the block, and go to step 3. Otherwise look at next highest-scoring transaction and continue until we run out of transactions or we are able to add something. 
1. Remove those transactions from the mempool, and re-sort. Go back to step 2. 

Annoying implementation details 
We can’t actually remove things from the mempool during CreateNewBlock. 
Instead, when we add something to a candidate block, we walk the descendants of the added transactions, and stick them in a special holding cell with a score that is modified, as though the parent transactions were confirmed. 
We look at the best transaction in the holding cell and compare it with the next best mempool transaction, and pick the one that has higher feerate. 
See code for more details... 

In what situations does ancestor feerate mining fail? 
Ancestor feerate mining does well when wallets implement simple CPFP schemes, where a single child transaction pays for one or more parents, bumping the feerate of the package to being high enough to mine. 
In what situations would this algorithm do poorly? How poorly can it do, compared to the optimal mining strategy? 
Open research problem I have been meaning to tackle: write code to compare optimal mining behavior with ancestor feerate mining behavior. Monitor network to see if we deviate too much. 

