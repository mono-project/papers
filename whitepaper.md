# Mono
Scalability by design

**Contents**
- [General](#General)
	- [Proof of Work](#Proof-of-Work)
	- [Scalability](#Scalability)
		- [Blocks](#Blocks)
		- [Transactions](#Transactions)
		- [Emission](#Emission)
	- [Value](#Value)
- [Memory](#Memory)
	- [Optimizations](#Optimizations)
		- [Transaction](#Transaction)
		- [Verification](#Verification)
		- [Block](#Block)
	- [Light-Wallet](#Light-Wallet)
	- [Sharding](#Sharding)
- [Usernames](#Usernames)
	- [Database](#Database)
	- [Blockhain](#Blockchain)
	- [Specifications](#Specifications)
	- [Security](#Security)
- [CRC256](#CRC256)
	- [Design](#Design)
		- [Speed](#Speed)
		- [Safety](#Safety)
	- [Advantages](#Advantages)
- [ICO-Instructions](#ICO-Instructions)
	- [Conditions](#Conditions)
		- [Presale](#Presale)
		- [ICO](#ICO)
	- [Instructions](#Instructions)
	- [Price](#Price)
- [Resources](#Resources)

## General
In the following there will be improvements to the classic blockchain protocol listed. Those certainly are necessary to achieve true scalability aswell as providing a strong resistance against centralisation and attacks.
### Proof-of-Work
Many currently existing proof of work schemes have a few major issues. They are easily implementable on ASICs, depend on computational power or do not allow a decentralised network by favoring GPU miners and farms too much. To counter those issues, the following design considerations have to apply for a given algorithm:
1. **IO saturation**: The algorithm should consume nearly the entire available memory access bandwidth (this is a strategy toward achieving ASIC resistance, the argument being that commodity RAM, especially in GPUs, is much closer to the theoretical optimum than commodity computing capacity)
2. **CPU friendliness**: Targeting CPUs is difficult due to existing issues such as botnets, GPUs and ASICs. To decrease their performance, the algorithm uses special CPU features that already are there in modern CPUs but are missing in modern GPUs and are costly to implement in ASICs. Since GPUs have a bigger memory bus, the tradeoff is an almost equal situation between GPUs and CPUs.
3. **Light client verifiability**: a light client should be able to verify a round of mining in under 0.03 seconds on a desktop in C. With 1-hour blocktargets, this would mean that a validator is able to validate the blocks of a day in one second.
4. **Light client slowdown**: the process of running the algorithm with a light client should be much slower than the process with a full client, to the point that the light client algorithm is not an economically viable route toward making a mining implementation, including via specialized hardware.
5. **Light client fast startup**: a light client should be able to become fully operational and able to verify blocks within 2 seconds in C.

To suit those needs, mono utilises a new proof of work scheme called [SquashPoW](https://github.com/ClashLuke/Squash-Hash). It allows high hashrates, asymmetry and [ASIC resistance](https://github.com/ClashLuke/Squash-Hash/blob/master/docs/asic.md), just like its bigger brother [Ethash](https://github.com/ethereum/wiki/wiki/Ethash) does. The difference between Ethash and Squash is that Squash is an entirely new algorithm, designed to be as resistant against ASICs as possible, while Ethash utilises algorithms that are designed to be as powerful as possible on an ASIC.

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


## Memory
The only real limitation. The main concern at Mono. Memory (storage) usage aswell as bandwidth usage.

### Optimizations
#### Transaction
Bitcoin and other currencies use a lot of bloat in their transactions. In the case of bitcoin, there is one great issue in addition to the storage of signatures. Bitcoin also uses a DAG of transactions instead of using balances like ethereum does. Using balances allows having one input instead of plenty of inputs. By having one input, one output and an amount to transact, a raw, unsigned transaction is 76 bytes big. While this certainly is a good start, it can be optimised, by using [usernames](#usernames), which allows to save another 48 bytes, by using two 8 byte usernames instead of two 32 byte public keys. Therefore a raw, unsigned transaction has a total size of 24 byte. A signature still has to be added, which would add another 64 byte (resulting in a total of 88 bytes). Using the BLS signature scheme, all signatures can be stacked to one for the whole block, resulting to a constant memory usage, covered in [#verification](#verification).

#### Verification
To save space at verifications, a merkle tree is not used. Instead, sequential hashing is used to verify all transactions stored in the same block at once. All transactions are split into blocks of 4GiB and then hashed using CRC512, a variant of [CRC256](#CRC256). The resulting hash is then appended to the next block of data, hashed again using CRC512. The first block contains the stacked BLS signature and the 512bit hash of the previous block in addition to the transaction data. The results is a final hash of constant size the aggregated signature, the previous block hash and all transaction data. Calculating this hash takes about one second per 46 million transactions, using one normal consumer-grade CPU core (3.40GHz, Skylake architecture). A potential issue is that by tampering the transactions, an attacker might be able to receive the same final hash. This would allow the previous hash to be wrong aswell, which means that all previous blocks can be changed. Since nodes already agreed on previous blocks, the only possible scenario where this could cause issues is a sybil attack performed by a computationally speaking big thread. A thread big enough to be able to void 512bit security. In the following, we will assume that 512bit security secures a block more than enough. As a sidenote, this is the same security level as if you had to crack 2^256 hashes in bitcoin.


#### Block
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
1. **Bandwidth**: A sharded node would still have to download an entire block to verify it and then host the transactions its supposed to host. The data downloaded would be there just to verify once and then drop it. The entire block wont ever be broadcasted again by those shard-anodes which results in a potential for a lower bandwidth in total. Assuming that there are many nodes participating in shards, this becomes less of an issue.
2. **Size:** A minimum network size is necessary to securely allow this variation of sharding.

A RAID-like masternode system would therefore reduce the security an insignificant bit while reducing memory usage, increasing the quality of nodes and therefore the network. Overall this means that the user and the full node see a drastic performance increase. So much, that a common user might think about becoming a "full shard". This also implies that, by growing, this network will get faster and increase its own security. Since a minimum network size is required, this will be implemented using a hardfork after one year.


## Usernames
The advantages and disadvantages of an on-chain username system allowing users to pick names rather than getting an unspeakable address assigned

### Database
Allowing users to pick usernames themselves certainly has its issues since usernames do not represent the public key, like a Base32 encoding would. This means, instead of displaying a pseudorandom address thats difficult to transfer to friends and impossible to memorize, you display a username. To ensure everyone knows the same usernames, database of `UserName:PublicKey` mapping is needed. Such a database would need to contain signatures of the `UserName:PublicKey` pairs to make sure a public key wont be changed. It also has to be ensured that a pairing wont be removed from the database as well as adding a way to reward broadcasting nodes. It can be done by broadcasting those pairs as transactions to the network. By having the public key of the sender as an input and the username as the output as well as no amount. The amount of 0 shows that this is the first transaction of the user and is saved as the leading bytes, to allow verifying nodes to know that the size is different.

### Blockchain
This means that usernames will be broadcasted and saved in the blockchain before they are saved as a database entry. A user registration therefore takes up 120 bytes in the blockchain. Why the transaction requires only 112 bytes compared to the 144 of an ordinary transaction can be seen in the [specifications](#Specifications). Additionally as mentioned in [#Database](#Database), a user has to pay 500 KMONO (variations depending on the difficulty might be the case, see [here](https://zawy1.blogspot.com/2017/12/using-difficulty-to-get-constant-value.html))to register themselves. This allows an ordinary human to participate (registration fees are 50ct, one-time) while disallowing spammers to fill the network with useless transactions or registering many names. Spamming 1GiB (registering 9.5M usernames) of useless data to the network would cost more than 45 million US dollar, a constant price.

### Specifications
To be accepted by the network, a username has to fulfill the following requirements:
1. **Uniqueness**: Nobody may have ever used the same username. 
2. **Size**: While a username might be used directly if it has less than nine characters, it has to be hashed if it is longer than that. Internal storage will not display this username, but it is possible to retrieve the internal username from the real username.
3. **Primal Transaction**: A registration transaction has to be the first one broadcasted. Any other transaction containing the "0"-amount will be rejected.
The size of eight bytes is chosen to allow a maximum number of 18 quintillion (18 billion billion) usernames to be registered. This allows every living human on earth to own one billion addresses at the same time. This of course assumes that the human population will stop growing at about 10 billion citizen.

### Security
Since registrations are logged on the blockchain, they are immutable. This allows saying that a `UserName:PublicKey` pair is known to every participant, while being sure that they wont be altered by a third party. A doubled registration can not happen because a block containing a doubled registration would be dropped (since it is invalid), this would force a new block to be generated and the miner of this block to not get any reward. Therefore it is not possible to take over a username nor a public key as well as changing a database entry, thanks to blockchain technology.

## CRC256
A secure 256 bit cyclic redundancy check, faster than blake2.

### Design
A CRC32 has 16 bit collision resistance, therefore two CRC32s of different data have 32 bit collision resistance, eight CRC32s with a total size of 256 bit have a collision resistance of 128 bit. Accordingly by using eight CRC32s to calculate one 256 bit block, this block posesses similar properties to CRC. Internally, the code calculates eight CRC32s like this 
`for(int i=0;i<8;i++) temp_32[i] = crc32(in_32[i]);` and then aggregates the results to the previous result by performing `out_64[i] ^= temp_64[i]` for every 256 bit (32 byte) block of the input. This implies that this implementation still posesses all features of CRC32C, which means that it works very well as a hash function.

#### Speed
Since CRC32s already are very fast, a CRC256 is very fast aswell. Why the speed of the CRC32 matters can be seen in the [design](#design) section below. To get to the numbers, with about 54GiB/s processing speed on a [Xeon E3-1225v2](https://ark.intel.com/content/www/us/en/ark/products/65733/intel-xeon-processor-e3-1225-v2-8m-cache-3-20-ghz.html), we have an average speed of 16.9 bytes per CPU cycle (or 0.06 cycles per byte). On a [Huawei P9 Lite](https://en.wikipedia.org/wiki/Huawei_P9) which utilises the [Cortex A53](https://www.arm.com/products/silicon-ip-cpu/cortex-a/cortex-a53), the total throughput is 30% lower with 30.1 GiB/s with its 1.4GHz ARMv8 CPU. This results in 0.043 cycles per byte on an optimised architecture (or 23 bytes per cycle).

#### Safety
Whenever a drastic speed improvement is proposed, the security of it has to be proven aswell. Luckily this is relatively simple, because this algorithm relies entirely on CRC32 and does nothing but optimising throughput by utilising parallel calculations (optimising CPU-delays) aswell as increasing the size to 256 bit by having eight simultanious streams of CRCs. This allows a parallel execution on two threads to further optimise the speed (more threads would require more memory to be used. Proportional growth). Since the security aswell as the limitiations of CRC256 depends on its [design](#design), it is covered in there.

### Advantages
The usage of CRC256 in favor of Blake2b allows to process 50GB/s on an average low-end server CPU, per thread. This allows to hashing 2.2 billion transactions per second (2.2GTPS), which is necessary to validate a block hash quickly or to mine a block without having to wait for the block template to be processed. Validators can therefore validate much quicker (~50x) than they could with Blake2b or other fast hashing functions, allowing them to synchronize more easily.


## ICO Instructions
Instructions for the ICO and Presale held by Mono. The successor of Clash.

### Conditions
#### Presale
This is a private presale. Known investors and miners are allowed to invest ethereum to get their MONO Token. The presale allows investors to buy tokens 10% cheaper. In addition, another 20% discount 
can be achieved by trading in one Clash [CLS], totalling in a 30% discount for pre-sale investors. One CLS reduces the price of one MMONO (Million MONO) by exactly 20%. The funds gaines in the presale will be spent towards a company registration, UX/UI design and legal support.

#### ICO
The MONO ICO is public, therefore regulations are needed. Additionally, previously mentioned bonusses do not apply. MONO utilises a standard KYC model. A verification of an investors using their full name, date of birth, nationality and in, case more than 150 ETH are used to purchase MONO, an AML aswell. The general process of an KYC can be seen [here](https://medium.com/theacorncollective/a-step-by-step-guide-to-completing-acorns-kyc-process-5a619c13b2a6).

### Instructions
1. **Opening form**: The google form can be found in [here](https://docs.google.com/forms/d/1Khexwfsz0-VoomFEkisO18JznrXqlU4CjI1clBN1kXM). Enter your data there.
2. **Entering CLS Preferences**: If you are interested in using your CLS for an additional discount, enter the amount of (whole) CLS you want to use for a discount. Then send your CLS to `1FvgmBLpAgrKVXMeLzG7PVhFDevQULZ4GUaFDS6vWZPwMiXgYsf8KeAcfTYwNT1cpcRr4uRLnM51cd8mWpBUBqDu77FrNDs` and enter the transaction ID spit out by your wallet. Make sure you use mixin so that nobody can use your TxID. Note that you have to pay the fees. 
3. **Steps using Ethereum**: First, send your prefered amount of ETH to `0x4b09b4aeA5f9C616ebB6Ee0097B62998Cb332275` and enter your ethereum transaction ID. Afterwords sign this string `a247209c0fa3fef7d82deed7dcf44e4a670531a288884c64f705aa15bd2c33a1` with your ethereum private key. If you do not know how, get your private key from your ethereum wallet and use a tool like [this](https://github.com/warner/python-ecdsa). The signature has be submitted.

### Value
Since there is no discount in the ICO, the price is set to 0.01 ETH per MMONO. During the presale, it is 0.009 ETH or 0.007 ETH respectively. There are six months premined, with the emission discussed [here](https://github.com/ClashLuke/Mono-Papers/blob/master/whitepaper.md), this results in 8,235,000 MMONO that are premined. 8.1 Million of these are available for the ICO resulting in a hardcap of 82,350 ETH for this ICO to rise.


