## Memory
The only real limitation. The only concern at Mono. Memory (storage) usage aswell as bandwidth usage.

- [Optimizations](#Optimizations)
	- [Transactions](#Transactions)
	- [Verification](#Verification)
	- [Blocks](#Blocks)
- [Sharding](#Sharding)
	- [Lattice](#Lattice)
	- [RAID](#RAID)

### Optimizations
#### Transactions
Bitcoin and other currencies use a lot of bloat in their transactions. In the case of bitcoin, there is one great issue in addition to the storage of signatures (33 wasted bytes). Bitcoin also uses a DAG of transactions instead of using balances like ethereum does. Using balances allows having one input instead of plenty of inputs. By having one input, one output, the amount and a fee, the size of one transaction is reduced to 74 byte. 74 byte, because we store both public keys, not the addresses, to increase security of addresses from 160bit to 256bit. To allow verification of the transaction, a signature is needed. When done using BLS, they can be stacked to a total of 33byte, no matter how manyVerification transactions are used.

#### Verification
To save space at verifications, a merkle tree is not used. Instead, sequential hashing is used to verify all transactions stored in the same block at once. All transactions are split into blocks of 4GiB and then hashed using Blake2b-512. The resulting hash is then appended to the next block of data, hashed again using Blake2b-512. The first block contains the stacked BLS signature and the 512bit hash of the previous block in addition to the transaction data. The results is a final hash of constant size containing the signature data, aswell as the previous block hash and all transaction data. Calculating this hash takes about one second per 15 million transactions. A potential issue is that by tampering the transactions, an attacker might be able to receive the same final hash. This would allow the previous hash to be wrong aswell, which means that all previous blocks can be changed. Since nodes already agreed on previous blocks, the only possible scenario where this could cause issues is a sybil attack performed by a computationally speaking big thread. A thread big enough to be able to void 512bit security. In the following, we will assume that 512bit security secures a block more than enough. (Just a sidenote, this is the same security level as if you had to crack 2^256 hashes in bitcoin.)


#### Blocks
In conclusion, blocks contain 
1. **Transaction count**: 8 byte, 64 bit unsigned integer.
2. **Transactions**: 74 byte per transaction, mentioned above
3. **Verification**: 64 byte in total, mentioned above
4. **BLS signature**: 33 byte in total, explained [here](https://en.wikipedia.org/wiki/Boneh%E2%80%93Lynn%E2%80%93Shacham) and [here](https://medium.com/cryptoadvance/bls-signatures-better-than-schnorr-5a7fe30ea716)
5. **Timestamp**: 4 byte, 32bit unsigned integer
6. **Nonce**: 8 byte, 64bit unsigned integer to prove the work done in order to mine the block

The difficulty of a current block aswell as the current reward can be calculated using a set of timestamps. To not be forced to recalculate all difficulties at boot, those are stored in a local database, but not shared over the network.

This results in exactly 191 bytes per block (+ coinbase transaction) in addition to 74 byte per transacion. Applying this on the transactions and blocks in bitcoin, this chain would have a total size of 27.7 GiB where bitcoin has with its 400 million transactions and 570,000 blocks a total size of 200GiB. By increasing the block spacing to 32 minutes, the total size gets reduced to 25.9GiB.

To save more space, purging can be implemented. This might cause sharding or security issues due to hash missmatches.

### Sharding
All of the following solutions give up parts of the security of a blockchain in favor of saving memory space and bandwidth to allow everyone to participate. All of the following solutions require some kind of validator or master node, which implies that a tiered system is implemented. Tiered systems have a strong impact on decentralisation aswell as security.
#### Lattice
To save even more memory space, a lattice system can be utilised. Everyone influenced by a certain transaction saves this transaction to their own lattice. This implies that you only have to broadcast your transactions to known validators (instead of every node). Those validators will then check whether or not they want to include your transaction in the next block. If so, they broadcast the block to eachother and allow every non-validator to make requests to their database. If alice sent ten MMONO to bob, bob will then check in every hour at the validators to see if a new transaction for him came in. If so, he adds it to his own lattice. 

An appoach like this would drastically reduce the security of a network since alice and bob have to trust the validators. Since being a validator is not a privelege but instead a paid burden, alice and bob can become validators as well by asking for whole blocks from the validators and mining themselves. This would reduce the trust factor and create a decentralised opt-in validator network.

**Pro**: 
1. **Independance**: Users have to synchronise only their transactions and not rely on other nodes to tell them their balance while still having only a few bytes of data stored.
2. **Changeless**: The group of validators is still the same size, since the reward stays the same.
3. **Speed**: By hosting the own data, a user can check validate their own balance quickly on startup, which removes the necessity of saving the balance.

**Con**:
1. **Mining Storage**: Since pools are validators and the only known full nodes, miners have to be forced to run the whole chain aswell.

In conclusion, a lattice system would increase security and speed for every casual user while not impacting the rest of the ecosystem. Due to the security and speed changes, it is possible that more people who previously hosted full nodes do not do so anymore, unless they are mining.

#### RAID
More specific, a variation of [RAID 6](https://en.wikipedia.org/wiki/Standard_RAID_levels#RAID_6). Masternodes, nodes that paid a fee to support the network, can be used to store blockchain data. By implementing a RAID-like system where each node has a number {0, 1, 2, 3}, and every number is set to store specific blocks (0 - {0, 1}, 1 - {1, 2},..), every node will always store half of the blockchain instead of the whole blockchain and drop the rest. Masternodes are then paid for every request they fulfill, which means that well-known and active masternodes will earn much more than an idling masternode hosted on a raspberry pi. Since masternodes are the memory of the chain, while validators are its processor, validators will either have to make requests to masternodes which in turn would heavily increase their delays (because they have to download a few whole blocks per hash), host the whole chain themselves without being rewarded for it, or become a masternode to get two payments at once. Note that you can host two masternodes on the same device on the same blockchain, which would allow miners to host a 0 and a 2 masternode while also mining. This is highly recommended for nodes that already have to be a full node, such as miners, explorers or exchanges.

**Pro**:
1. **Memory**: By using half the space, every full node would host 13GiB in previously mentioned bitcoin example. Compared to bitcoins 200GiB thats a lot less.
2. **Incentive**: While certain types of nodes already are forced to host full nodes, even more people will get incentivised to use their bandwidth to support the chain.
3. **Reward**: Nodes that already store the full chain will get a reward for doing so. Miners get two rewards at once.
4. **Speed**: By having more high-bandwidth nodes (such as servers, which is usually what a masternode is hosted on) and less low-quality full nodes such as home PCs, picking an active node aswell as downloading the whole chain (synchronising with the network) becomes much faster.

**Con**:
1. **51% Attack**: By holding 51% of all masternodes considered active, trustworthy and well-known, an attacker might cause a chain split out of malicious intend. This is unlikely and results in no gain, but a possible attack scenario.
2. **Sybil Attack**: By having only "verified" (via burning) nodes as masternodes, a sybil attack becomes much more possible since an attacker needs 51% of the capital invested in masternodes aswell as trust by the users instead of 51% of the nodes and the best ping times.

A RAID-like masternode system would therefore reduce the security an insignificant bit while reducing memory usage, increasing the quality of nodes and therefore the network. Overall this means that the user sees a drastic performance increase. This also implies that, by growing, this network will get faster and increase its own security. 

