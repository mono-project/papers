## Memory
The only real limitation. The only concern at Mono. Memory (storage) usage aswell as bandwidth usage.

- [Optimizations](#Optimizations)
	- [Transactions](#Transactions)
	- [Verification](#Verification)
	- [Blocks](#Blocks)
- [Light-Wallet](#Light-Wallet)
- [Sharding](#Sharding)

### Optimizations
#### Transactions
Bitcoin and other currencies use a lot of bloat in their transactions. In the case of bitcoin, there is one great issue in addition to the storage of signatures. Bitcoin also uses a DAG of transactions instead of using balances like ethereum does. Using balances allows having one input instead of plenty of inputs. By having one input, one output and an amount to transact, a raw, unsigned transaction is 76 bytes big. While this certainly is a good start, it can be optimised, by using [usernames](#usernames), which allows to save another 48 bytes, by using two 8 byte usernames instead of two 32 byte public keys. Therefore a raw, unsigned transaction has a total size of 24 byte. A signature still has to be added, which would add another 64 byte (resulting in a total of 88 bytes). Using the BLS signature scheme, all signatures can be stacked to one for the whole block, resulting to a constant memory usage, covered in [#verification](#verification).

#### Verification
To save space at verifications, a merkle tree is not used. Instead, sequential hashing is used to verify all transactions stored in the same block at once. All transactions are split into blocks of 4GiB and then hashed using Blake2b-512. The resulting hash is then appended to the next block of data, hashed again using Blake2b-512. The first block contains the stacked BLS signature and the 512bit hash of the previous block in addition to the transaction data. The results is a final hash of constant size the aggregated signature, the previous block hash and all transaction data. Calculating this hash takes about one second per 46 million transactions, using one normal consumer-grade CPU core (3.40GHz, Skylake architecture). A potential issue is that by tampering the transactions, an attacker might be able to receive the same final hash. This would allow the previous hash to be wrong aswell, which means that all previous blocks can be changed. Since nodes already agreed on previous blocks, the only possible scenario where this could cause issues is a sybil attack performed by a computationally speaking big thread. A thread big enough to be able to void 512bit security. In the following, we will assume that 512bit security secures a block more than enough. As a sidenote, this is the same security level as if you had to crack 2^256 hashes in bitcoin.


#### Blocks
In conclusion, blocks contain 
1. **Transaction count**: 8 byte, 64 bit unsigned integer.
2. **Verification**: 64 byte in total, mentioned above
3. **BLS Signature**: 96 byte in total, as stated [here](https://github.com/Chia-Network/bls-signatures)
4. **Timestamp**: 4 byte, 32bit unsigned integer
5. **Nonce**: 8 byte, 64bit unsigned integer to prove the work done in order to mine the block
6. **Coinbase Transaction**: 8 byte (out), since a lot of the transaction is already known
7. **Transactions**: 24 byte per transaction, mentioned above

The difficulty of a current block aswell as the current reward can be calculated using a set of timestamps. To not be forced to recalculate all difficulties at boot, those are stored in a local database, but not shared over the network.

This results in exactly 188 bytes per block in addition to 24 byte per transacion. Applying this on the transactions and blocks in bitcoin, this chain would have a total size of 9.0 GiB where bitcoin has with its 400 million transactions and 570,000 blocks a total size of 200GiB. The actual amount stored on a node can be further reduced by implementing [sharding](#Sharding).

To save more space, purging can be implemented. This might cause sharding or security issues due to hash missmatches.

### Light-Wallet
To ensure the security and independance of a light wallet, a lattice-like system can be utilised. Everyone influenced by a certain transaction saves this transaction to their own lattice. This implies that you only have to broadcast your transactions to known validators (instead of every node). Those validators will then check whether or not they want to include your transaction in the next block. If so, they broadcast the block to eachother and allow every non-validator to make requests to their database. If alice sent ten MMONO to bob, bob will then check in every hour at the validators to see if a new transaction for him came in. If so, he adds it to his own lattice. 

An appoach like this would drastically reduce the security of a network since alice and bob have to trust the validators. Since being a validator is not a privelege but instead a paid burden, alice and bob can become validators as well by asking for whole blocks from the validators and mining themselves. This would reduce the trust factor and create a decentralised opt-in validator network.

**Pro**: 
1. **Independance**: Users have to synchronise only their transactions and not rely on other nodes to tell them their balance while still having only a few bytes of data stored.
2. **Changeless**: The group of validators is still the same size, since the reward stays the same.
3. **Speed**: By hosting the own data, a user can check validate their own balance quickly on startup, which removes the necessity of saving the balance.

**Con**:
1. **Mining Storage**: Since pools are validators and the only known full nodes, miners have to be forced to run the whole chain aswell.

In conclusion, a lattice system would increase security and speed for every casual user while not impacting the rest of the ecosystem. Due to the security and speed changes, it is possible that more people who previously hosted full nodes do not do so anymore, unless they are hosting a masternode now. More on that in [#Sharding](#Sharding).

### Sharding
A variation of RAID 6 can be utilised to save even more space aswell as adding scalability. The more nodes, the faster, smaller and more secure the network. Masternodes, nodes that paid a fee to support the network, can be used to store blockchain data. Every node stores every header (3MB/year) and additionally stores transactions assigned to them. Each transaction is stored by 1024 nodes, which means that a 51% attack is not possible. Additionally 1024 nodes dropping off the network when they get paid for staying online and their payment per minute increases over time is very unlikely. To calculate what nodes store which transactions, the following formula is used `if( (TxID%activeNodeCount)>>10 == (UserID%activeNodeCount)>>10 ) storeTransaction()`. Where the TxID is calculated like this `hash(hash(Tx) + hash(BlockHeader))` and the UserID is the hash of the public key of your UserID-keypair. This assures that only active masternodes (over the last week) get to store transactions while also making it unpredictable which nodes will store a transaction. A reward after successfully. To make sure that nobody pays a significant amount of MONO to host a masternode, while still disallowing the creation of billions of nodes without holding a stake, a fee of 1 MMONO has to be paid (sent to 000..000, effectively burning it). This allows everyone to participate easily as one or many masternodes while disallowing the creation of many nodes with the purpose of attacking the network. Since full nodes, such as blockchain explorers or exchanges, are forced to host the full chain, those nodes are able to use one thousand masternodes simultaniously allowing them to fulfill many requests to the network, therefore gaining MONO from holding the chain, something they would've done anyway. This is a further incentive to hold blockchain explorers, pools or even participate as a masternode since you can easily gain a few MONO without using processing power but instead your bandwidth. To assure that there are not billions of mobile phones on the network which essentially are low-quality nodes (they are only connected to WiFi a few hours per day, they usually have poor upload speeds), the fee of 1 MMONO (1 USD) is used. A masternode would either have to be hosted on a server for a long time (more than a month) to start making profit for the hoster or a bunch of mobile phones or raspberry pi's would have to be connected using LAN to a high-quality network. This is because a masternode gets paid for every request they fulfill. If a node asks for the number of transactions from alice, the node also sends a transaction with it. Assuming that one request is worth 1MONO, bob would have to pay 1 KMONO to find out all data of all 999 transactions alice ever sent or 16.5 KMONO to download the entire chain of the last year. This can be seen as a fee for light nodes.

**Pro**:
1. **Memory**: By using 1024 nodes to store one transaction, memory usage and memory distribution increases the more nodes a network has.
2. **Scalability**: Instead of slowing down the more nodes this network has, it gets faster by having more distributed memory aswell as less storage on each node.
3. **Incentive**: While certain types of nodes already are forced to host full nodes, even more people will get incentivised to use their bandwidth to support the chain.
4. **Reward**: Nodes that already store the full chain will get a reward for doing so. Miners get two rewards at once.
5. **Speed**: By having more high-bandwidth nodes (such as servers, which is usually what a masternode is hosted on) and less low-quality full nodes such as home PCs, picking an active node aswell as downloading the whole chain (synchronising with the network) becomes much faster.

**Con**:
1. **Bandwidth**: A sharded node would still have to download an entire block to verify it and then host the transactions its supposed to host. The data downloaded would be there just to verify once and then drop it. The entire block wont ever be broadcasted again by those shard-nodes which results in a potential for a lower bandwidth in total. Assuming that there are many nodes participating in shards, this becomes less of an issue.

A RAID-like masternode system would therefore reduce the security an insignificant bit while reducing memory usage, increasing the quality of nodes and therefore the network. Overall this means that the user and the full node see a drastic performance increase. So much, that a common user might think about becoming a "full shard". This also implies that, by growing, this network will get faster and increase its own security. 

