## Mono
Scalability by design

**Contents**
- [Proof of Work](#Proof-of-Work)
- [Scalability](#Scalability)
	- [Blocks](#Blocks)
	- [Transactions](#Transactions)
	- [Emission](#Emission)
- [Value](#Value)

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
In contrast to other cryptocurrencies, mono utilises a very high block target. With its 32 minutes per block, it takes up to an hour for your transaction to receive its first confirmation. Utilising a 32 minute block time, the network would average at about two transactions per second, assuming the same block size limits were chosen as they are in bitcoin. But in contrast to bitcoin, mono uses real resources as the upper bound. Bandwidth. By having an unlimited block size, the network will accept it if a miner were to include the whole mempool. This of course would require them to have a strong bandwidth to be able to transmit their currently mined blocks. Since plenty of servers already utilise 1GiB/s upload speeds and the mining node will in most cases be a server, not the miner themselves (pool mining), this does not inherit a limitation for either user. Instead, this allows to have a much better bloat to transaction ratio than other cryptos have. With a fixed growth of 3MiB per year, the only thing that impacts the size of the blockchain are transactions. Assuming that all mining nodes have 1GiB/s upload bandwidth which is entirely utilised by submitting a block, the network would have a theoretical maximum of 5.4 million transactions per second. Since not all nodes have this download capacity, lets assume we have the average german node with 2MiB/s down, 200KiB/s. This would conclude in a synchronising speed of ten thousand transactions per second.

Now, with possible peak speeds of 5.4MTPS and constant speeds of 10kTPS, there certainly are issues with this network. Including such a huge amount of transactions would mean that the blockchain would be able to grow above one terabyte within one block. While this itself is not an issue, assuming that all those transactions are legitimate transactions which have to be included in blocks, it might be an issue if those transactions are garbage transactions. Garbage that can not be purged off the chain. This means that space in the blockchain has to cost.

#### Transactions
To reduce transaction sizes, amount obfuscation has to be disabled aswell as denominations. This allows us to have a typical one-in-one-out transaction like ethereum does. Those are typically at 200 bytes, with a minimum of 100 bytes. 

Additionally a has to introduced, which has to be high enough to disallow attackers from spamming the network with their transactions. Such a fee should be at about 10ct to force the sender of a transaction to pay 50ct per KB they use. This would mean that for every MiB an attacker wants to be included, they have to pay 524USD, for every GiB 539'000USD. While those fees still allow everyone to send a transaction, even a micro transaction, they heavily punish spammers and endorse miners. To assure that the fees wont overshoot or drop below a certain point, an [adaptive fee](https://zawy1.blogspot.com/2017/12/using-difficulty-to-get-constant-value.html) has to be introduced.

#### Emission
Since a miner can easily make thousands of dollars by including a few megabytes of transactions into blocks, they are heavily incentivised to do so. Not just that, this also means that all transactions are included every block, resulting in a fight over who gets those precious fees. But fees are not constant. They might come, but can not be calculated. To have a constant reward for miners without relying on fees, an emission of new coins is used. This emission has to be as low as possible to allow fees to have a high impact on the reward, but a constant reward is also not wanted. Instead, a reward based on the difficulty is used. The reward is calculated by dividing the base reward by the square of the second logarithm of the current difficulty. In this particular case a curve would look like [this](https://clashproject.org/images/diff_reward.png) ([svg version](https://clashproject.org/images/diff_reward.svg)). At a difficulty of 1.7P (ethereums current difficulty), the reward would be around 0.0003 coins. 

What are the advantages of an emission curve like this? 
1. **Lower Bound**: If the difficulty drops, the emitted coins will rise. This will naturally bring the difficulty back up, in case a major miner went off the network.
2. **Upper Bound**: If the difficulty rises unproportionally to the price, it becomes less luctrative far faster than it would do in static cases (such as bitcoins emission).
3. **Growth correllation**: If the price increases, the difficulty naturally increases aswell. This means that the reward will decrease resulting in a better approximation of a constant emission (in USD) per block.
4. **Static Difficulty**: Since an increasing difficulty results in drastic reward drops, a decreasing difficulty doing the opposite thing, the difficulty appears to be much more static compared to other coins.

Potential tradeoffs are insider mining, such as pools cooperating that each of them mines a block after the other did. Since this would require full consensus over the network and proof of work communities tend to [communiticate less than other communities](https://proofofwork.news/p/proof-of-work-60), such an attack is highly unlikely.

### Value
To allow the general usage of this coin, there are no decimals. This allows global adoption. As of right now, there is a maximum supply of `18,446,744,073,709,551,615` coins (thats 18.4 Quintillion Coins or 18.4 Million Million Million Coins). This is a huge number, which is why we recommend using GMONO. The maximum supply of MONO is 18.4 billion GMONO. Assuming that MONO becomes as popular as the US Dollar, a transition to MMONO would be possible to allow the usage of 18.4 trillion MMONO. For comparision, there currently is a circulating supply of about 1.8 trillion US Dollar.

