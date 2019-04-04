## Usernames
The advantages and disadvantages of an on-chain username system allowing users to pick names rather than getting an unspeakable address assigned.

- [Database](#Database)
- [Blockhain](#Blockchain)
- [Specifications](#Specifications)
- [Security](#Security)

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
