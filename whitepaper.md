# Mono
Scalability by design

**Contents**
- [General](#General)
	- [Scalability](#Scalability)
		- [Blocks](#Blocks)
		- [Transactions](#Transactions)
		- [Emission](#Emission)
	- [Value](#Value)
- [Blockchain](#Blockchain)
	- [Consensus Models](#Consensus-Models)
		- [Proof of Work](#Proof-of-Work)
		- [Proof of Stake Reputation](#Proof-of-Stake-Reputation)
		- [Proof of Weighted Work](#Proof-of-Weighted-Work)
			- [Concept](#Concept)
			- [Reputation-Variant](#Reputation-Variant)
		- [Proof of Progress](#Proof-of-Progress)
	- [Memory](#Memory)
		- [Optimizations](#Optimizations)
			- [Transaction](#Transaction)
			- [Verification](#Verification)
			- [Block](#Block)
		- [Light-Wallet](#Light-Wallet)
		- [Sharding](#Sharding)
	- [Usernames](#Usernames)
		- [Database](#Database)
		- [On-Chain](#On-Chain)
		- [Specifications](#Specifications)
		- [Security](#Security)
	- [Token](#Token)
- [Crypto](#Crypto)
	- [Squash](#Squash)
		- [Squash 0](#Squash_0)
			- [1-CRC32](#1-CRC32)
			- [2-IntegerMath](#2-IntegerMath)
			- [3-BitMoving](#3-BitMoving)
			- [4-Rotation](#4-Rotation)
			- [5-Encryption](#5-Encryption)
		- [Squash 1](#Squash_1)
			- [Differences](#Differences)
			- [1.5-Scratchpad](#1.5-Scratchpad)
			- [2-IntegerMath](#2-IntegerMath)
		- [Conclusions](#Conclusions)
	- [SquashPoW](#SquashPoW)
		- [Concept](#Concept)
		- [Reference](#Reference)
		- [CRC32](#CRC32)
		- [Parameters](#parameters)
		- [ASICs](#ASICs)
			- [Light-Evaluation-Method](#Light-Evaluation-Method)
			- [Full-Evaluation-Method](#Full-Evaluation-Method)
	- [CRC256](#CRC256)
		- [Design](#Design)
			- [Speed](#Speed)
			- [Safety](#Safety)
		- [Advantages](#Advantages)
- [ICO](#ICO)

## General
In the following there will be improvements to the classic blockchain protocol listed. Those certainly are necessary to achieve true scalability aswell as providing a strong resistance against centralisation and attacks.
### Scalability
#### Blocks
In contrast to other cryptocurrencies, mono utilises a very high block target. With its 2 minutes per block, it takes less than five minutes for your transaction to receive its first confirmation. Utilising a 2 minute block time, the network would average at about 35 transactions per second, assuming the same block size limits were chosen as they are in bitcoin. But in contrast to bitcoin, mono uses real resources as the upper bound. Bandwidth and processing speed (validation). By having an unlimited block size, the network will accept it if a miner were to include the whole mempool. This of course would require them to have a strong bandwidth to be able to transmit their currently mined blocks. Since plenty of servers already utilise 1GiB/s upload speeds and the mining node will in most cases be a server, not the miner themselves (pool mining), this does not inherit a limitation for either user. Instead, this allows to have a much better bloat to transaction ratio than other cryptos have. With a fixed growth of 48MiB per year, the only thing that impacts the size of the blockchain are transactions. Assuming that all mining nodes have 1GiB/s upload bandwidth which is entirely utilised by submitting a block, the network would have a theoretical maximum of 11.2 million transactions per second. Since not all nodes have this download capacity, lets assume we have the average german node with 2MiB/s down, 200KiB/s. This would conclude in a synchronising speed of ten thousand transactions per second.

Now, with possible peak speeds of 11.2MTPS and constant speeds of 22.8kTPS, there certainly are issues with this network. Including such a huge amount of transactions would mean that the blockchain would be able to grow above one terabyte within one block. While this itself is not an issue, assuming that all those transactions are legitimate transactions which have to be included in blocks, it might be an issue if those transactions are garbage transactions. Garbage that can not be purged off the chain. This means that space in the blockchain has to cost.

#### Transactions
Transactions and their memory optimisations are described in memory.md 

Additionally a fee has to introduced, which has to be high enough to disallow attackers from spamming the network with their transactions. Such a fee should be at about 10ct to force the sender of a transaction to pay 1 USD per KB they use. This would mean that for every MiB an attacker wants to be included, they have to pay 1048USD, for every GiB over a million USD. While those fees still allow everyone to send a transaction, even a micro transaction, they heavily punish spammers and endorse miners. To assure that the fees wont overshoot or drop below a certain point, an [adaptive fee](https://zawy1.blogspot.com/2017/12/using-difficulty-to-get-constant-value.html) has to be introduced.

#### Emission
Since a miner can easily make thousands of dollars by including a few megabytes of transactions into blocks, they are heavily incentivised to do so. Not just that, this also means that all transactions are included every block, resulting in a fight over who gets those precious fees. But fees are not constant. They might come, but can not be calculated. To have a constant reward for miners without relying on fees, an emission of new coins is used. This emission has to be as low as possible to allow fees to have a high impact on the reward, but a constant reward is also not wanted. Instead, a reward based on the difficulty is used. The reward is calculated by dividing the base reward by the square of the second logarithm of the current difficulty. In this particular case a curve would look like [this](https://clashproject.org/images/diff_reward.png) ([svg version](https://clashproject.org/images/diff_reward.svg)). At a difficulty of 1.7P (ethereums current difficulty), the reward would be around 0.3 MMONO. (The starting point was 1GMONO per block)

What are the advantages of an emission curve like this? 
1. **Lower Bound**: If the difficulty drops, the emitted coins will rise. This will naturally bring the difficulty back up, in case a major miner went off the network.
2. **Upper Bound**: If the difficulty rises unproportionally to the price, it becomes less luctrative far faster than it would do in static cases (such as bitcoins emission).
3. **Growth correllation**: If the price increases, the difficulty naturally increases aswell. This means that the reward will decrease resulting in a better approximation of a constant emission (in USD) per block.
4. **Static Difficulty**: Since an increasing difficulty results in drastic reward drops, a decreasing difficulty doing the opposite thing, the difficulty appears to be much more static compared to other coins.

Potential tradeoffs are insider mining, such as pools cooperating that each of them mines a block after the other did. Since this would require full consensus over the network and proof of work communities tend to [communiticate less than other communities](https://proofofwork.news/p/proof-of-work-60), such an attack is highly unlikely.

### Value
To allow the general usage of this coin, there are no decimals. This allows global adoption. As of right now, there is a maximum supply of `18,446,744,073,709,551,615` coins (thats 18.4 Quintillion Coins or 18.4 Million Million Million Coins). This is a huge number, which is why we recommend using GMONO. The maximum supply of MONO is 18.4 billion GMONO. Assuming that MONO becomes as popular as the US Dollar, a transition to MMONO would be possible to allow the usage of 18.4 trillion MMONO. For comparision, there currently is a circulating supply of about 1.8 trillion US Dollar.

## Blockchain
### Consensus-Models
#### Proof-of-Work
Many currently existing proof of work schemes have a few major issues. They are easily implementable on ASICs, depend on computational power or do not allow a decentralised network by favoring GPU miners and farms too much. To counter those issues, the following design considerations have to apply for a given algorithm:
1. **IO saturation**: The algorithm should consume nearly the entire available memory access bandwidth (this is a strategy toward achieving ASIC resistance, the argument being that commodity RAM, especially in GPUs, is much closer to the theoretical optimum than commodity computing capacity)
2. **CPU friendliness**: Targeting CPUs is difficult due to existing issues such as botnets, GPUs and ASICs. To decrease their performance, the algorithm uses special CPU features that already are there in modern CPUs but are missing in modern GPUs and are costly to implement in ASICs. Since GPUs have a bigger memory bus, the tradeoff is an almost equal situation between GPUs and CPUs.
3. **Light client verifiability**: a light client should be able to verify a round of mining in under 0.03 seconds on a desktop in C. With 1-hour blocktargets, this would mean that a validator is able to validate the blocks of a day in one second.
4. **Light client slowdown**: the process of running the algorithm with a light client should be much slower than the process with a full client, to the point that the light client algorithm is not an economically viable route toward making a mining implementation, including via specialized hardware.
5. **Light client fast startup**: a light client should be able to become fully operational and able to verify blocks within 2 seconds in C.

To suit those needs, mono utilises a new hash called [SquashPoW](#SquashPoW). It allows high hashrates, asymmetry and [ASIC resistance](https://github.com/ClashLuke/Squash-Hash/blob/master/docs/asic.md), just like its bigger brother [Ethash](https://github.com/ethereum/wiki/wiki/Ethash) does. The difference between Ethash and Squash is that Squash is an entirely new algorithm, designed to be as resistant against ASICs as possible, while Ethash utilises algorithms that are designed to be as powerful as possible on an ASIC.

#### Proof-of-Stake-Reputation
To fight against the energy usage of the proof of work consensus model, an different consensus model is needed. A consensus model purely dependant on intrinsic values, ignoring all changes outside of itself. Proof of Stake does exactly that, but while it allows a reduction of power usage, it also decreases the security of a currency and, by having an inflation, also lowers the value of a currency. To increase the security of the network, an incentive for constant staking has to be found. Additionally honest stakers have to be rewarded for their honesty, increasing the number of honest stakers. To allow both changes, PoSR (Proof of Stake Reputation) is being used. 

In contrast to PoSV, PoSR increases the reward over time for a long-time staker. This implies that honest stakers with a high reputation have an increased total stake. The basic formula to calculate ones stake is `stake = coins staked * stake age`. This allows for a linear growth of stake over time, allowing new stakers to participate, while also giving old stakers a huge reward. As an example, one coin which has been staked for three years is worth as much as 1000 coins which have been staked for one day. This implies that an attacker would either have to buy over 90% of the currently active coin supply or stake for many years. Assuming that there is a constant group of old stakers (with three years advantage), an attacker would have to stake nine years to reduce the required funds from 99.9% to 64%. The probability of this happenening is quite low, therefore providing additional resistance against 51% or even 90% attacks. The downside is that old stake is able to dominate the network, if the staking node is constantly online and providing services to the network. Considering the added security, this does not appear to be an issue.

One last issue with Proof of Stake that has to be resolved is the annual inflation. Instead of an exponential inflation, a static inflation and a reduction of emission over time is wanted. In bitcoin this was done using block reward halving, but MONO uses its own specifically designed [Emission Curve](#Emission). To continue having an emission like this, a value from the outside has to be taken to create a difficulty which is dependant on the price. Since the value of holding a [masternode](#sharding) increases if the price of the coin increases and the total number of active masternodes is known to the network and the activity status is easily checkable, the emission curve takes this number in account, instead of the proof of work difficulty. Therefore the block reward goes to the one validator and the emission curve continues progressing just as normal.

A general issue with the Proof of Stake consensus model is the "Nothing at Stake" problem, where one can vote on all chains, in case the chain splits. This issue is resolved by having [masternodes](#sharding) which always vote on one chain and act as checkpointers. To achieve a chain split, 50% of the masternodes would have to vote on each chain, which is a highly unlikely edgecase. PoSR will be actived after masternodes have been activated.

#### Proof-of-Weighted-Work
##### Concept
Many cryptocurrencies tried combining proof of stake with proof of work. A common idea is splitting the blocks, so that there is one block PoW, one block PoS. A different variation is buying the oppurturnity to be allowed to mine. Both of these reduce decentralization and security, therefore are not an option. This means that something else has to be found, to achieve a security similar to proof of work, or higher, while solving the problem of energy usage. Aiwe from karbowanets proposed [Proof of Work with Stake](https://github.com/Karbovanets/papers/blob/master/PoW%20with%20Stake.md), which increases the security of the network by adding stake to proof of work and forcing an attacker to have both 51% of the hashrate as well as 51% of the emitted and active coins. While this already is a strong solution, it adds a minimum requirement reducint availability and decentralization to achieve a higher security. Instead of having a minimum (changing) requirement to be able to submit a block, a bonus for staking more coins might be ideal. By having a difficulty equal to `personalDifficulty = globalDifficulty*activeState/personalStake`, a higher stake equals to a lower difficulty. This means that an increase of reward can be achieved by either buying more coins or increasing the hashrate. Meaning that if there are miners, there is demand for the coin, increasing price in early stages and reducing energy usage later on. Therefore the work provided to the proof of work consensus is weighted by adding stake to the provided hashrate.

##### Reputation-Variant
The issue with using a PoS variant like PoSR for this, is that the stake has a relatively high impact on the consensus without providing additional security. To add security, the funds have to be locked up when using as stake, as proposed [here](https://github.com/Karbovanets/papers/blob/master/PoW%20with%20Stake.md). This can be done by using PoSR and ignoring currently locked up funds. The advantage of PoSR+Lock-up+PoW in favor of PoS+Lock-up+PoW is that PoSR heavily rewards stake time. This results in a behaviour which removes the incentive of supporing a pool, since those have a lower staking reward and require a higher trust and more stake. Therefore PoWW-SR has strong decentralization and security since it combines both [Proof of Stake Reputation](#Proof-of-Stake-Reputation) and [Proof of Work](#Proof-of-Work) additionally it reduces the energy consumption and increases demand as shown in the [Concept](#Concept) description.

#### Proof-of-Progress
The last consensus algorithm used in MONO is PoP (proof of progress). Its a cross-chain algorithm allowing to add another block to its top when a specific number of blocks on another chain is passed. Assuming that the other chain continously progresses and also is secure, the PoP chain is as secure and as fast as the "main chain" without adding any kind of bloat to the chain. This consensus algorithm will be used to deploy [token](#Token) on the MONO network.

### Memory
The only real limitation. The main concern at Mono. Memory (storage) usage aswell as bandwidth usage.

#### Optimizations
##### Transaction
Bitcoin and other currencies use a lot of bloat in their transactions. In the case of bitcoin, there is one great issue in addition to the storage of signatures. Bitcoin also uses a DAG of transactions instead of using balances like ethereum does. Using balances allows having one input instead of plenty of inputs. By having one input, one output and an amount to transact, a raw, unsigned transaction is 76 bytes big. While this certainly is a good start, it can be optimised, by using [usernames](#usernames), which allows to save another 48 bytes, by using two 8 byte usernames instead of two 32 byte public keys. Therefore a raw, unsigned transaction has a total size of 24 byte. A signature still has to be added, which would add another 64 byte (resulting in a total of 88 bytes). Using the BLS signature scheme, all signatures can be stacked to one for the whole block, resulting to a constant memory usage, covered in [#verification](#verification).

##### Verification
To save space at verifications, a merkle tree is not used. Instead, sequential hashing is used to verify all transactions stored in the same block at once. All transactions are split into blocks of 4GiB and then hashed using CRC512, a variant of [CRC256](#CRC256). The resulting hash is then appended to the next block of data, hashed again using CRC512. The first block contains the stacked BLS signature and the 512bit hash of the previous block in addition to the transaction data. The results is a final hash of constant size the aggregated signature, the previous block hash and all transaction data. Calculating this hash takes about one second per 46 million transactions, using one normal consumer-grade CPU core (3.40GHz, Skylake architecture). A potential issue is that by tampering the transactions, an attacker might be able to receive the same final hash. This would allow the previous hash to be wrong aswell, which means that all previous blocks can be changed. Since nodes already agreed on previous blocks, the only possible scenario where this could cause issues is a sybil attack performed by a computationally speaking big thread. A thread big enough to be able to void 512bit security. In the following, we will assume that 512bit security secures a block more than enough. As a sidenote, this is the same security level as if you had to crack 2^256 hashes in bitcoin.


##### Block
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

#### Light-Wallet
To ensure the security and independance of a light wallet, a lattice-like system can be utilised. Everyone influenced by a certain transaction saves this transaction to their own lattice. This implies that you only have to broadcast your transactions to known validators (instead of every node). Those validators will then check whether or not they want to include your transaction in the next block. If so, they broadcast the block to eachother and allow every non-validator to make requests to their database. If alice sent ten MMONO to bob, bob will then check in every hour at the validators to see if a new transaction for him came in. If so, he adds it to his own lattice. 

An appoach like this would drastically reduce the security of a network since alice and bob have to trust the validators. Since being a validator is not a privelege but instead a paid burden, alice and bob can become validators as well by asking for whole blocks from the validators and mining themselves. This would reduce the trust factor and create a decentralised opt-in validator network.

**Pro**: 
1. **Independance**: Users have to synchronise only their transactions and not rely on other nodes to tell them their balance while still having only a few bytes of data stored.
2. **Changeless**: The group of validators is still the same size, since the reward stays the same.
3. **Speed**: By hosting the own data, a user can check validate their own balance quickly on startup, which removes the necessity of saving the balance.

**Con**:
1. **Mining Storage**: Since pools are validators and the only known full nodes, miners have to be forced to run the whole chain aswell.

In conclusion, a lattice system would increase security and speed for every casual user while not impacting the rest of the ecosystem. Due to the security and speed changes, it is possible that more people who previously hosted full nodes do not do so anymore, unless they are hosting a masternode now. More on that in [#Sharding](#Sharding).

#### Sharding
A variation of RAID 6 can be utilised to save even more space aswell as adding scalability. The more nodes, the faster, smaller and more secure the network. Masternodes, nodes that paid a fee to support the network, can be used to store blockchain data. Every node stores every header (3MB/year) and additionally stores transactions assigned to them. Each transaction is stored by 1024 nodes, which means that a 51% attack is not possible. Additionally 1024 nodes dropping off the network when they get paid for staying online and their payment per minute increases over time is very unlikely. To calculate what nodes store which transactions, the following formula is used `if( (TxID%activeNodeCount)>>10 == (UserID%activeNodeCount)>>10 ) storeTransaction()`. Where the TxID is calculated like this `hash(hash(Tx) + hash(BlockHeader))` and the UserID is the hash of the public key of your UserID-keypair. This assures that only active masternodes (over the last week) get to store transactions while also making it unpredictable which nodes will store a transaction. A reward after successfully. To make sure that nobody pays a significant amount of MONO to host a masternode, while still disallowing the creation of billions of nodes without holding a stake, a fee of 1 MMONO has to be paid (sent to 000..000, effectively burning it). This allows everyone to participate easily as one or many masternodes while disallowing the creation of many nodes with the purpose of attacking the network. Since full nodes, such as blockchain explorers or exchanges, are forced to host the full chain, those nodes are able to use one thousand masternodes simultaniously allowing them to fulfill many requests to the network, therefore gaining MONO from holding the chain, something they would've done anyway. This is a further incentive to hold blockchain explorers, pools or even participate as a masternode since you can easily gain a few MONO without using processing power but instead your bandwidth. To assure that there are not billions of mobile phones on the network which essentially are low-quality nodes (they are only connected to WiFi a few hours per day, they usually have poor upload speeds), the fee of 1 MMONO (1 USD) is used. A masternode would either have to be hosted on a server for a long time (more than a month) to start making profit for the hoster or a bunch of mobile phones or raspberry pi's would have to be connected using LAN to a high-quality network. This is because a masternode gets paid for every request they fulfill. If a node asks for the number of transactions from alice, the node also sends a transaction with it. Assuming that one request is worth 1MONO, bob would have to pay 1 KMONO to find out all data of all 999 transactions alice ever sent or 16.5 KMONO to download the entire chain of the last year. This can be seen as a fee for light nodes.

**Pro**:
1. **Memory**: By using 1024 nodes to store one transaction, memory usage and memory distribution increases the more nodes a network has.
2. **Scalability**: Instead of slowing down the more nodes this network has, it gets faster by having more distributed memory aswell as less storage on each node.
3. **Incentive**: While certain types of nodes already are forced to host full nodes, even more people will get incentivised to use their bandwidth to support the chain.
4. **Reward**: Nodes that already store the full chain will get a reward for doing so. Miners get two rewards at once.
5. **Speed**: By having more high-bandwidth nodes (such as servers, which is usually what a masternode is hosted on) and less low-quality full nodes such as home PCs, picking an active node aswell as downloading the whole chain (synchronising with the network) becomes much faster.

**Con**:
1. **Bandwidth**: A sharded node would still have to download an entire block to verify it and then host the transactions its supposed to host. The data downloaded would be there just to verify once and then drop it. The entire block wont ever be broadcasted again by those shard-anodes which results in a potential for a lower bandwidth in total. Assuming that there are many nodes participating in shards, this becomes less of an issue.
2. **Size:** A minimum network size is necessary to securely allow this variation of sharding.

A RAID-like masternode system would therefore reduce the security an insignificant bit while reducing memory usage, increasing the quality of nodes and therefore the network. Overall this means that the user and the full node see a drastic performance increase. So much, that a common user might think about becoming a "full shard". This also implies that, by growing, this network will get faster and increase its own security. Since a minimum network size is required, this will be implemented using a hardfork after one year.


### Usernames
The advantages and disadvantages of an on-chain username system allowing users to pick names rather than getting an unspeakable address assigned

#### Database
Allowing users to pick usernames themselves certainly has its issues since usernames do not represent the public key, like a Base32 encoding would. This means, instead of displaying a pseudorandom address thats difficult to transfer to friends and impossible to memorize, you display a username. To ensure everyone knows the same usernames, database of `UserName:PublicKey` mapping is needed. Such a database would need to contain signatures of the `UserName:PublicKey` pairs to make sure a public key wont be changed. It also has to be ensured that a pairing wont be removed from the database as well as adding a way to reward broadcasting nodes. It can be done by broadcasting those pairs as transactions to the network. By having the public key of the sender as an input and the username as the output as well as no amount. The amount of 0 shows that this is the first transaction of the user and is saved as the leading bytes, to allow verifying nodes to know that the size is different.

#### On-Chain
This means that usernames will be broadcasted and saved in the blockchain before they are saved as a database entry. A user registration therefore takes up 120 bytes in the blockchain. Why the transaction requires only 112 bytes compared to the 144 of an ordinary transaction can be seen in the [specifications](#Specifications). Additionally as mentioned in [#Database](#Database), a user has to pay 500 KMONO (variations depending on the difficulty might be the case, see [here](https://zawy1.blogspot.com/2017/12/using-difficulty-to-get-constant-value.html))to register themselves. This allows an ordinary human to participate (registration fees are 50ct, one-time) while disallowing spammers to fill the network with useless transactions or registering many names. Spamming 1GiB (registering 9.5M usernames) of useless data to the network would cost more than 45 million US dollar, a constant price.

#### Specifications
To be accepted by the network, a username has to fulfill the following requirements:
1. **Uniqueness**: Nobody may have ever used the same username. 
2. **Size**: While a username might be used directly if it has less than nine characters, it has to be hashed if it is longer than that. Internal storage will not display this username, but it is possible to retrieve the internal username from the real username.
3. **Primal Transaction**: A registration transaction has to be the first one broadcasted. Any other transaction containing the "0"-amount will be rejected.
The size of eight bytes is chosen to allow a maximum number of 18 quintillion (18 billion billion) usernames to be registered. This allows every living human on earth to own one billion addresses at the same time. This of course assumes that the human population will stop growing at about 10 billion citizen.

#### Security
Since registrations are logged on the blockchain, they are immutable. This allows saying that a `UserName:PublicKey` pair is known to every participant, while being sure that they wont be altered by a third party. A doubled registration can not happen because a block containing a doubled registration would be dropped (since it is invalid), this would force a new block to be generated and the miner of this block to not get any reward. Therefore it is not possible to take over a username nor a public key as well as changing a database entry, thanks to blockchain technology.

### Token
In ETH, EOS and most currently existing cryptocurrencies, token are saved on the main chain. The advantages of a token are simple, it allows easy interaction with the main chain and adds security since the token doesn't need its own chain. Yet it bloats the main chain, something MONO does not want. Therefore token and other second-layer applications are out-sourced using [PoP](#Proof-of-Progress). This means token can still utilise the security of the main network and have easy interaction, without adding any unecessary information to full nodes or forcing miners to validate transactions they dont care about. Additionally it allows everyone to use a language of their liking. Most importantly it allows other chains to have other rules, such as its own fees that are unrelated to MONO, changed block rewards or add ring signatures. Thanks to the token allowing any language and absolute freedom, there are literally no limitations. To allow the best performance in interaction with the MONO cryptocurrency, an API will be provided, working similarly to most currently known payment APIs. 

## Crypto
### Squash
#### Squash_0
##### 1-CRC32
Assuming that there is a perfect CRC, a 32bit redundancy check (with random 32bit inputs) has 2^16 collisions. We calculate the CRC four times to have delays according to the inner-CPU delays, as seen [here](https://github.com/JuliaLang/julia/blob/master/src/crc32c.c#L111). Now 128bit of the input are processed, resulting in 128bit output with 64bit collision-resistance. Code: `crc_32[0] = crc32(data_32[0]);` CRCs are heavily optimised on ARM CPUs since they are an addition to the instructionset. They are relatively expensive for ASICs to implement, according to [this](https://www.slideshare.net/bschn2/the-rainforest-algorithm) post.

##### 2-IntegerMath
The next 128bit are calculated using two times two 64bit integer arithmetics. First, the next 64bit of input data are used to be added to the first two CRCs. Since an addition is a XOR with carry, it can be assumed that the first bit is a casual XOR, all following bits are a doubled XOR. This results in a slightly increased probability for high results - which, since the highest bit is cut off, does not matter. To even the first few bits out, a division of the previous data (3rd 64bit of the input, 1st 64bit of the CRC results) is added to the previous result using a XOR. The resulting expression looks like this `crc_64[2] = (data_64[2] + crc_64[0]) ^ (data_64[2] / crc_64[0]);`. This results in another 128bit with 64bit collision-resistance (assuming that the ADD operator is cryptographically insecure). While those 64bit operations are easy for a modern CPU, it is [difficult](https://www.slideshare.net/bschn2/the-rainforest-algorithm) for ASICs to implement those.

##### 3-BitMoving
A bit reverse and a byteswap of existing data are additionally added, since those require one cycle but significantly increase GPU and ASIC resistance. None of these increases entropy nor collision resistance, but the byteswap is optimised for x86 and ARM instructionsets, which results in a slowdown for GPUs. Additionally the bit reverse is optimised in the ARM instructionset as [rbit](http://www.keil.com/support/man/docs/armasm/armasm_dom1361289889382.htm), which increases performance for low-power devices such as [ARM Server CPUs](https://www.arm.com/products/silicon-ip-cpu/neoverse/neoverse-n1) or smartphones.

##### 4-Rotation
Compared to the previous operations, the rotations are relatively cheap. `key[1][0] = (out_64[0]>>shift[0]) | (out_64[0]<<(shift[1]));` Two 64bit rotations in each direction are used to obtain keys to encrypt the previously generated 256bit with. Those are variable rotations, the number of bits to shift is obtained by reading from the first/last byte of the previous 256bit. Since those dont change fundamentally change data but instead are a fancy copy mechanism which provides additional protection against ASICs but cost nothing on a [casual CPU](https://www.slideshare.net/bschn2/the-rainforest-algorithm).

##### 5-Encryption
The last step, an AES ECB encryption, is there to increase entropy, GPU resistance and ASIC resistance as seen [here](https://www.slideshare.net/bschn2/the-rainforest-algorithm). It does not fundamentally increase collision resistance or security. The encryption is performed using the first 128bit of the previous results (the CRC results) and encrypts them with the key obtained from the integer math. The code looks like this: `aes(out, (uint8_t*)key[0]);`. The same thing goes for the second 128bit encrypted with the rotated CRC.

#### Squash_1
##### Differences
Squash 1 has the same steps as Squash 0, just another step inserted after the first CRC and a slightly changed integer math.

##### 1.5-Scratchpad
Right after the first step, four CRC32s, their data is taken to read from a read-only scratchpad. In Squash 1, the first 8bit are used to read from a 256 Byte scratchpad, Squash 2 uses the first 16bit of each CRC to read from a 64KiB scratchpad, Squash 3 uses the full 32bit of each CRC to read from a 4GiB scratchpad. The data obtained from Cache/RAM is then stored as the next 128bit of the data, like this `crc_32[4] = ((uint32_t*)&scratchpad[crc_8[ 0]])[0];`, so that there are two 128bit blocks totalling in 256bit. This step is solely used to force an attacker to store the (ideally pseudorandom) scratchpad. In case of Squash 3, there are iterations of reading, to have the hash be I/O bound instead of processing bound. Since this step heavily relies on the previous CRC, it cannot be better than the CRC32, which results in 64bit collision resistance for the read 128bit.

##### 2-IntegerMath
Since the second 128bit already are assigned, it would not be useful to reassign them, ignoring their data. Thats why the result of the two 64bit math operations is calculated using 128bit of the previous data `divr[0] = (data_64[2] + crc_64[2]) ^ (data_64[2] / crc_64[0]);` and then XORed with two 64bit blocks `out_64[0] = crc_64[0]^divr[0]; out_64[1] = crc_64[1]^divr[0];`. There are different blocks XORed than used for calculation to increase the variaty of results.

#### Conclusions
To sum everything up, Squash is a hash with 128bit collision resistance, a high speed of 5.8 cycles per byte (Ivy Bridge, 4.0 cpb on Zen). In contrast to other hashes, Squash relies on 256bit inputs but is also able to provide a decent level of ASIC resistance. To further increase the ASIC resistance without decreasing the speeds below a [certain level](https://github.com/ethereum/wiki/wiki/Ethash-Design-Rationale), SquashPoW was introduced.


### SquashPoW
#### Abstract
The same Xeon E3-1225v2 as mentioned above has about 100kH/s (single-threaded, using DDR3-1333 RAM) using the 4GiB dataset, 11H/s using the light mode and a 64MiB cache. An IoT device such as a rapsberry pi 3B+ achieves an average hashrate of 0.9H/s on one thread. This is because they utilise much slower CPUs to allow huge power savings as well as tiny amounts of RAM, forcing the device to run in validation mode - even while mining. Therefore this speed difference can be seen as a way to lock out low-power devices, which would otherwise create rising prices for users of these as well as increasing the risk of an attack by a botnet. Mobile phones are not the most secure devices.

#### CRC32
CRC32 was used to provide a data aggregation function which is (i) non-associative, and (ii) easy to compute. A commutative and associative alternative to CRC32 would be XOR. Furthermore, a CRC has the advantage of already being implemented in CPU hardware which makes it difficult to compute for GPUs and expensive for specialised hardware.

#### Parameters
* A 64 MB cache was chosen because a smaller cache would allow for an ASIC to be produced far too easily using the light-evaluation method. The 64 MB cache still requires a very high bandwidth of cache reading, whereas a smaller cache could be much more easily optimized. A larger cache would lead to validation nodes requiring more RAM which is not feasable at certain points. 
* 256 parents per DAG item was chosen in order to ensure that time-memory tradeoffs can only be made at a worse-than-1:1 ratio. It is not higher to assure a fast verification for validators. It is possible to reduce the accesses and increase the dataset parents to have the same resulting speed for a validator, increased speed for the miner but lower ASIC resistance.
* The 4GiB DAG size was chosen in order to require a level of memory larger than the size at which most specialized memories and caches are built, but still small enough for ordinary computers to be able to mine with it.
* In contrast to ethash, there is no multiplator to suit the changes associated with moores law. This is done since 1) most people dont upgrade their computers, and this applies to ASICs only and 2) moores law seems to not apply anymore, which means that there is more observation over the next three years needed. Especially considering that intels [3D stacked CPUs](https://en.wikipedia.org/wiki/Three-dimensional_integrated_circuit) might speed up everything up once they are released.
* 1024 accesses was chosen to bound the speed of the algorithm completely to the IO. This was done in order to reduce the risk of ASICs which may not improve the memory speed further than what is available to GPUs.
* The epoch length cannot be infinite (ie. constant dataset) because then the algorithm could be optimized via ROM, and very long epoch lengths make it easier to create memory which is designed to be updated very infrequently and only read often. Excessively short epochs would increase barriers to entry as weak machines would need to spend much of their time on a fixed cost of updating the dataset. The epoch length can probably be reduced or increased substantially if design considerations require it.

#### ASICs

Asic Resistance is one of the main concerns in the cryptocurrency industry. To provide a strong security against them while also providing several [design considerations](#Proof-of-Work) is the core achievement of the SquashPoW algorithm. The following two methods can be used to create an ASIC for this algorithm:

##### Light-Evaluation-Method
The first method an ASIC might use, is called the "Light Evaluation Method" because an ASIC uses many concurrent threads and stores a 64MiB cache instead of the full 4GiB dataset per thread. This method relies entirely on the speed of the generation of one dataset item. If an ASIC is able to generate an item faster than a CPU/GPU can, so much faster that it is not needed to store the 4GiB scratchpad but instead calculate every item when needed. To fight against this method being used on CPU/GPU, the dataset generation is made to be computationally expensive. For an ASIC, a repetitive algorithm can be implemented easily. Which is why the dataset generation uses an algorithm which already is ASICed for CPUs. CRC32 is hardware-implemented in ARMv8 CPUs, which means that an ASIC wont get be faster than a modern CPU. Since a CPU needs about 4'600x more time computing each dataset item at runtime compared to reading from the RAM, we can assume that the same goes for an ASIC. If an ASIC would use 4GiB RAM, it would be 72 times slower using the Light Evaluation Method and need 64x more computational energy compared to it using the Full Evaluation Method.

##### Full-Evaluation-Method
Using this variation, an ASIC wont run the code a validator will but instead precomputate the dataset to iterate over it and read from it. Since the dataset changes every epoch, and epochs are designed to happen every hour, it is unlikely to be some kind of ROM. Instead, an ASIC would have to use RAM such as HBM2.1. Assuming that GPU vendors already use top-of-the-line RAM, an ASIC will not get any advantages there. Which means the only possible speed improval would be the computational part. Since the computational part of the algorithm takes about one one-hundreth (on ARMv8) of the total time, there is a potential gain of 1% (assuming a manufacturer would be able to reduce this part to literally nothing). Since this is not the case, an ASIC would most likely be an ARMv8 CPU with fast HBM RAM.


### CRC256
A secure 256 bit cyclic redundancy check, faster than blake2.

#### Design
A CRC32 has 16 bit collision resistance, therefore two CRC32s of different data have 32 bit collision resistance, eight CRC32s with a total size of 256 bit have a collision resistance of 128 bit. Accordingly by using eight CRC32s to calculate one 256 bit block, this block posesses similar properties to CRC. Internally, the code calculates eight CRC32s like this 
`for(int i=0;i<8;i++) temp_32[i] = crc32(in_32[i]);` and then aggregates the results to the previous result by performing `out_64[i] ^= temp_64[i]` for every 256 bit (32 byte) block of the input. This implies that this implementation still posesses all features of CRC32C, which means that it works very well as a hash function.

##### Speed
Since CRC32s already are very fast, a CRC256 is very fast aswell. Why the speed of the CRC32 matters can be seen in the [design](#design) section below. To get to the numbers, with about 54GiB/s processing speed on a [Xeon E3-1225v2](https://ark.intel.com/content/www/us/en/ark/products/65733/intel-xeon-processor-e3-1225-v2-8m-cache-3-20-ghz.html), we have an average speed of 16.9 bytes per CPU cycle (or 0.06 cycles per byte). On a [Huawei P9 Lite](https://en.wikipedia.org/wiki/Huawei_P9) which utilises the [Cortex A53](https://www.arm.com/products/silicon-ip-cpu/cortex-a/cortex-a53), the total throughput is 30% lower with 30.1 GiB/s with its 1.4GHz ARMv8 CPU. This results in 0.043 cycles per byte on an optimised architecture (or 23 bytes per cycle).

##### Safety
Whenever a drastic speed improvement is proposed, the security of it has to be proven aswell. Luckily this is relatively simple, because this algorithm relies entirely on CRC32 and does nothing but optimising throughput by utilising parallel calculations (optimising CPU-delays) aswell as increasing the size to 256 bit by having eight simultanious streams of CRCs. This allows a parallel execution on two threads to further optimise the speed (more threads would require more memory to be used. Proportional growth). Since the security aswell as the limitiations of CRC256 depends on its [design](#design), it is covered in there.

#### Advantages
The usage of CRC256 in favor of Blake2b allows to process 50GB/s on an average low-end server CPU, per thread. This allows to hashing 2.2 billion transactions per second (2.2GTPS), which is necessary to validate a block hash quickly or to mine a block without having to wait for the block template to be processed. Validators can therefore validate much quicker (~50x) than they could with Blake2b or other fast hashing functions, allowing them to synchronize more easily.


## ICO
The MONO ICO is public, therefore regulations are needed. Additionally, previously mentioned bonusses do not apply. MONO utilises a standard KYC model. A verification of an investors using their full name, date of birth, nationality and in, case more than 150 ETH are used to purchase MONO, an AML aswell. The general process of an KYC can be seen [here](https://medium.com/theacorncollective/a-step-by-step-guide-to-completing-acorns-kyc-process-5a619c13b2a6). The only thing an investor has to do is sending ETH to the [crowdsale contract](https://etherscan.io/address/0x1a9ecb05376bf8bb32f7f038a845dbafb22041cd). The MONO Token will be sent out afterwords, so that it can be traded in using an escrow for MONO on the main net.

Since there is no discount in the ICO, the price is set to 0.01 ETH per MMONO. During the presale, it is 0.009 ETH or 0.007 ETH respectively. There are six months premined, with the emission discussed [here](https://github.com/ClashLuke/Mono-Papers/blob/master/whitepaper.md), this results in 8,235,000 MMONO that are premined. 8.2 Million of these are available for the ICO resulting in a hardcap of 82,350 ETH for this ICO to rise.


