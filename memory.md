## Memory
The only real limitation. The only concern at Mono. Memory (storage) usage aswell as bandwidth usage.

- [Optimizations](#Optimizations)
	- [Transactions](#Transactions)
	- [Verification](#Verification)
- [Conclusion](#Conclusion)

### Optimizations
#### Transactions
Bitcoin and other currencies use a lot of bloat in their transactions. In the case of bitcoin, there is one great issue in addition to the storage of signatures (33 wasted bytes). Bitcoin also uses a DAG of transactions instead of using balances like ethereum does. Using balances allows having one input instead of plenty of inputs. By having one input, one output, the amount and a fee, the size of one transaction is reduced to 74 byte. 74 byte, because we store both public keys, not the addresses, to increase security of addresses from 160bit to 256bit. To allow verification of the transaction, a signature is needed. When done using BLS, they can be stacked to a total of 33byte, no matter how manyVerification transactions are used.

#### Verification
To save space at verifications, a merkle tree is not used. Instead, sequential hashing is used to verify all transactions stored in the same block at once. All transactions are split into blocks of 4GiB and then hashed using Blake2b-512. The resulting hash is then appended to the next block of data, hashed again using Blake2b-512. The first block contains the stacked BLS signature and the 512bit hash of the previous block in addition to the transaction data. The results is a final hash of constant size containing the signature data, aswell as the previous block hash and all transaction data. Calculating this hash takes about one second per 15 million transactions. A potential issue is that by tampering the transactions, an attacker might be able to receive the same final hash. This would allow the previous hash to be wrong aswell, which means that all previous blocks can be changed. Since nodes already agreed on previous blocks, the only possible scenario where this could cause issues is a sybil attack performed by a computationally speaking big thread. A thread big enough to be able to void 512bit security. In the following, we will assume that 512bit security secures a block more than enough.


### Conclusion
In conclusion, block contain 
1. **Transaction count**: 8 byte, 64 bit unsigned integer.
2. **Transactions**: 74 byte per transaction, mentioned above
3. **Verification**: 64 byte in total, mentioned above
4. **BLS signature**: 33 byte in total, explained [here](https://en.wikipedia.org/wiki/Boneh%E2%80%93Lynn%E2%80%93Shacham) and [here](https://medium.com/cryptoadvance/bls-signatures-better-than-schnorr-5a7fe30ea716)
5. **Timestamp**: 4 byte, 32bit unsigned integer
6. **Nonce**: 8 byte, 64bit unsigned integer to prove the work done in order to mine the block

The difficulty of a current block aswell as the current reward can be calculated using a set of timestamps. To not be forced to recalculate all difficulties at boot, those are stored in a local database, but not shared over the network.

This results in exactly 191 bytes per block (+ coinbase transaction) in addition to 74 byte per transacion. Applying this on the transactions and blocks in bitcoin, this chain would have a total size of 27.7 GiB where bitcoin has with its 400 million transactions and 570,000 blocks a total size of 200GiB. By increasing the block spacing to 32 minutes, the total size gets reduced to 25.9GiB.

To save more space, purging can be implemented. This might cause sharding or security issues due to hash missmatches.
