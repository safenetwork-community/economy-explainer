

Unlinkability
So why is it important for a monetary token to be unlinkable? In short, because that is the technical means to achieve the more general and highly desirable monetary property 1 of fungibility 1. Fungibility means that one token is interchangeable with another.

Think of two unmarked gold bars of equal weight. One is as good as the other. They are indistinguishable. However, if these bars are marked with serial numbers and some central entity keeps a list of all gold exchanges tracked by the serial number, then we can have a situation where one bar becomes worth less than another in the minds of potential recipients because it was associated with some “bad” activity in the past. Or another might be even more valuable because it passed through the hands of a famous person. With such a system, participants may begin to lose trust in the gold bars. They may have to check against a centralized “good/bad list” for every transaction to prevent receiving a “bad bar”, and the overall system becomes less efficient. Due to tracking, we have lost fungibility. For more on this, see the above linked article.

There are also many privacy-loss implications when receiving or making a payment with a token that has a long history attached to it. The Safe Network aspires to enable private, safe transactions for all network participants.

How can we determine if a coin is linkable or unlinkable? It’s actually pretty easy, we just look at the inputs and outputs of a pair of transactions.

Note: In the case of DBCs, we often use the words transaction and reissue interchangeably

Here is a simplified example of two reissues using our current DBC system. We can say that Bob paid 50 to Alice (1st reissue) and then Alice paid the same 50 to Jim (2nd reissue). So each reissue has just one input DBC and one output DBC. Each DBC has an associated amount, public key of the owner pubkey, and a mint signature mintsig. In this example pubkey, mintsig and hash are shortened to 3 digits for brevity. In reality they are very large unique numbers.

reissue 1 (r1):
→ input = a {amount=50, pubkey = 567, mintsig=233},  hash(a) = 223
→ output = b {amount = 50, pubkey = 725, mintsig=112}, hash(b) = 787

reissue 2 (r2):
→ input = b {amount = 50, pubkey = 725, mintsig=112 }, hash(b) = 787
→ output = c {amount = 50, pubkey = 212, mintsig=455}, hash(c) = 993
Note: a, b, and c are labels for this example. They do not appear in a reissue. Amounts are now hidden by commitments, but not relevant to this analysis.

We can easily see that input b for r2 matches output b for r1. We can link them together by any of: pubkey == 725 or mintsig == 112, or hash == 787.

This establishes that reissues r1 and r2 are linked together because they share a common DBC. Over a series of reissues this creates a chain that can be followed.

This is what is meant by linkability.

Also if the amount is a very unique value (or value commitment), it could be used to link transactions.

An important point to notice is that even if we remove the pubkey and mintsig fields and just use a random id field (to prevent double spending), the DBCs are still identifiable by their hash value. In other words, if the same data is used as an output to one reissue and as an input to another, then the reissues are linkable. This is generalizable to any cryptocurrency, substituting the word transaction for reissue.

You may ask: what if we use multiple inputs and outputs and varying amounts to split and merge tokens? Well, such techniques can make it harder to be certain of the trail of an individual token. That is the basis of CoinJoin, CoinShuffle, and similar mixing algorithms. They make more possible links to follow/evaluate. But they do not actually break any links… which are preserved indefinitely for anyone to analyze at leisure using statistical analysis, data from third parties (eg exchanges), and so on. Such techniques are generally considered “weak-sauce” by cryptographers.

You may also ask: Why have any unique identifier in the DBC at all? The answer is that an identifier is necessary to prevent double-spending. The mint must be able to record a “coin” as spent, in order to catch any future attempt to spend it again.

Once we understand linkability, we can begin thinking about ways to prevent it.

A system with the property of unlinkability would not have any such links. The big question is, how to achieve that property?

Blind Signatures
The oldest known technique was invented by David Chaum and is known as Blind Signatures. It has the advantage that it is the simplest scheme to implement, and for now at least, the most efficient, which makes it very well suited for mint operations.

Let’s review the basic concept of a blind signature. Chaum’s paper 2 uses the example of an election conducted remotely. Here we present a condensed version, though Chaum’s is worth reading as well.

A blind signature is like putting a slip of paper with a message (eg a vote) inside an envelope which is lined with carbon. Mary sends the envelope with the slip inside and a return address to election official Charles. Charles signs the envelope (which also signs the slip inside) and sends it back. At a later date, Mary removes the vote slip and sends it back to Charles in a new envelope, but without a return address. Charles opens the envelope, removes the slip, verifies it has a valid signature by Charles, and counts the vote – without knowing who cast this particular vote or when it was signed (amongst the set of signed envelopes).

The key point is that there is no link between what is on any of the slips and what was written on any of the envelopes. So the vote cast by each of the voters is unknown to election official Charles. (excluding writing forensics, etc, etc which are not relevant for cryptographic discussions).

Ok, so lets perform our two reissues again, this time using blind signatures. Now it is the DBC Mint that blindly signs the envelope(s). For this example, we assume that all DBCs are equal in value.

reissue 1 (r1):
→ input = a.slip {pubkey = 567, mintsig=233},  hash(a.slip) = 223
→ output = b.envelope {blind(b.slip)}, hash(b.envelope) = 565

reissue 2 (r2):
→ input = b.slip {pubkey = 445, mintsig=112 }, hash(b.slip) = 787
→ output = c.envelope {blind(c.slip)}, hash(c.envelope) = 993
In words:

Bob presents input A (slip) to mint (with valid mint sig over A) and obtains mint’s signature over output B (envelope) with mint sig indicating that B is worth the same as A (slip).
Mint also marks A (slip) as spent.
Bob removes B (slip) from B (envelope) and gives it to Alice, who presents B (slip) to the mint and obtains mint’s sig over C (envelope) worth the same as B (slip).
Mint also marks B (slip) as spent.
Now, the interesting thing here is that B (envelope) has a different hash from B (slip) inside it. 565 != 787

Therefore, the hashes do not match between tx1 and tx2 and neither do any fields. Nothing connects these two transactions together so far as the mint can see. We have unlinkability!

*Zero Knowledge Circuits*
A more modern approach to unlinkable transactions is through the use of Zero-Knowledge (ZK) proof circuits ala ZCash.

With ZK proofs, we are attempting to convince the Mint of something without revealing some sensitive information. ZK circuit proof systems like PLONK have emerged over the last few years as general systems for engineering ZK proofs.

For example, say we implement a ZK circuit that simulates sha3_256. This circuit would allow us to prove to a third party that we know the data that hashes to some hash without revealing the data itself. We can even prove properties about this data!

Say for example that the data we are hashing is actually an entire reissue transaction.

let reissue = Reissue {
   inputs: { Dbc {amount: 9, owner: pk1,..}, Dbc {amount: 1, owner: pk2,..},
   outputs: { Dbc {amount: 10, owner: pk3,..}
   ownership_proofs: {pk1_sig, pk2_sig}
};
I could run this reissue through sha3_256 to produce a hash: tx_hash = sha3_256(reissue).
To convince the mint that this is a hash of a valid transaction, we can generate a ZK proof using our sha3_256 circuit.

A ZK circuit has private and public inputs, in this example, the reissue information will be private inputs and the tx_hash will be a public input.

private: [9, pk1, 1, pk2, 10, pk3, pk1_sig, pk2_sig]
public:  [tx_hash]
We can write down constraints on these private/public inputs, these constraints are encoded into a ZK Circuit that can then be run to produce proofs that these constraints are satisfied

/// Reissue Constraints
private[0] + private[2] = private[4]        // 9 + 1 = 10
private[1].verify(private[0..6], public[6]) // pk1 signed the transaction
private[3].verify(private[0..6], public[7]) // pk2 signed the transaction
public[0] = sha3_256(private)               // reissue data hashes to tx_hash
Note: we are referring to inputs by their index, circuits have a pre-determined size, this size limits the number of input / output DBC’s we can reissue within a single reissue transaction.

Now once we encode these constraints as a ZK circuit, we can insert the reissue details into the private input slots and the tx_hash into the public input slot of the circuit. We then turn the crank to produce a proof that all constraints are satisfied.

Importantly, this proof does not leak any information about the private inputs, we can send this proof along with the tx_hash to a Mint and it would be convinced that this tx_hash is a hash of a valid transaction. The mint could then sign tx_hash to certify the reissue.

This is very cool, but the research in this area is still young and the tooling around these ZK circuit systems still quite immature. Proofs tend to be very slow to produce and verify, especially for complicated circuits like sha3_256.

If you’d like to go deeper this series is quite good 3.

*Ring Signatures*
Ring Signatures, as used in Monero and other cryptocurrencies, are a method to obscure the transaction history by hiding in a group.

In a ring signature, Alice signs a transaction using (1) Alice’s private key, (2) Alice’s public key, and (3) other people’s public keys. Anyone can verify Alice’s signature using the set of public keys, but they’re (usually) unable to determine who the true signer was because there are multiple equivalent public keys.

As such, ring signatures are a form of automatic mixing. Links still exist, but there are so many of them, that it may be too difficult to analyze. In Monero at present, there are 11 possible signers: the real one plus 10 others. So this is like hiding in a crowd of 11 people… better than standing alone, but not super comforting if a bloodhound is sniffing around.

Amount Hiding
Amount Hiding, also known as Confidential Transactions also helps improve privacy, but it does not affect [un]linkability.

Amount Hiding is compatible with zk circuits and ring signatures but not with a blind signature approach, which typically requires the use of fixed denominations.

We discussed Amount Hiding via Pedersen commitments in a previous update.

Fixed Denominations
Fixed denominations are another form of privacy protection, similar in purpose to amount hiding. Only a certain limited number of amounts are allowed by the system. To represent any other amount, one must make change using the known amounts.

Forcing all transactions to use these fixed amounts means that all inputs and outputs to a transaction appear regular. While the total amount of a given transaction is visible, one cannot track a given input or output effectively by its amount because it merges into an ocean of other tokens worth the same amount. As such, all units of a given denomination are fungible with eachother.

It is therefore important to keep the total number of denominations small. One could for example define every integer value to its own denomination. While this would work technically, it is bad for privacy, as people will often use very unique amounts. Instead of merging into an ocean, a single drop wanders by itself.

DBC Mints

DBC Mints have been around since the 90’s but they never became very popular.

One serious issue preventing adoption is that they have typically been centralized and thus require that users of the system (token holders) trust the mint operator completely. A greedy mint operator could inflate the money supply without anyone else knowing, thereby stealing value from all the users. Also, the mint operator could simply disappear, and all the tokens become value-less overnight. So one must trust the mint operator not only today, but also tomorrow, and also that no third party, hacker, insider, or government can impact their operation.

Decentralization via sections (sharding) is a central feature of the Safe Network design. The SN DBC design leverages and builds on that to create a Mint that is both decentralized and scaleable. To date, no deployed cryptocurrency[1] has incorporated decentralized DBC mints that we are aware of, so this is a novel development. It is our aim to scale to worldwide usage with instant (and private) transaction settlements.

The decentralization functionality can be broken up into a few main areas:

MintNode peers within a Section.
Sharding: Each Section is responsible for a subset of DBCs.
Client Aggregation
Distributed Spentbook written by Client
Let’s look at each of these.

MintNode peers within a Section.
Within a section, each Elder is also designated as a MintNode. Thus, there are 7 nodes, with others ready and waiting to replace any node that fails. When the section is formed or membership changes, the nodes perform a round of distributed-key-generation (DKG) to create a shared BLS key. This is a multi-sig key, such that 5 of the 7 (super-majority) nodes must sign a piece of data for it to be considered valid. In the case of DBCs, the signed data is called a ReissueShare. These ReissueShares must be collected from a super-majority of nodes in a section to complete the reissue of DBC’s managed by that section.

Let’s say one of the nodes is malicious. Perhaps a client has colluded with a node to double spend one of it’s DBCs. The malicious node could respond with a fraudulent ReissueShare for the double spend, but this share on it’s own is useless, a client must convince a super-majority of nodes before it can collect enough ReissueShares to certify a double-spend reissue.

Further it is difficult in terms of time and resources to become a Mint Node (section Elder) in the first place. The Safe Network has a concept of Node Age by which Adult nodes must prove themselves to be reliable and honest over time. Only the most senior adult is selected as an Elder whenever an existing elder disappears. On top of this, a node cannot choose which section it will be in. All these strategies make it difficult for an attacker to achieve multiple malicious nodes in a section relevant to their DBC activity.

Sharding: Each Section is responsible for a subset of DBCs.
Sharding is a general term which means breaking up data so that different servers handle subsets of the data.

Our sharding approach is actually quite simple. The DBCs spend key (A universally unique one-time-key) determines which Section is responsible for handling it. The client software is responsible for sending an identical ReissueRequest to all mint nodes in all sections corresponding to DBC inputs in the ReissueRequest.

In the following example, we have 3 inputs, each of which are handled by different sections, each section responds with a super-majority (5 of 7) of reissue shares.

fanout
fanout
1052×1000 126 KB
As a MintNode loops through the input DBCs, for each one it checks if it has responsibility or not. If a MintNode is responsible for a DBC, the Spentbook is consulted on whether this DBC has been spent already. So a MintNode ReissueShare only provides SignatureShare for the DBCs which a particular section has authority for.

Client Aggregation
When a user wants to reissue a DBC, the user’s client software contacts each of the 7 MintNodes for the section(s) responsible for the input DBC(s) and submits a ReissueRequest. Each node validates the request, and if everything is correct, it replies with a ReissueShare which contains a BLS SignatureShare.

The client software collects the ReissueShare replies from all the MintNodes it contacted and inspects them. If at least 5 nodes in a given section have responded correctly then the client can combine their SignatureShare in order to form a final Mint Signature for each output DBC. With the Signature in hand, the client then constructs the actual DBC(s) which can later be sent as input to another reissue.

fanin
fanin
1380×730 136 KB
Distributed Spentbook written by Client
This is a feature in development. The basic idea is that the Client commits to a given ReissueTransaction and publishes that commitment, identified by the DBC’s spend key (or a hash of the key), on the Safe Network before it makes a ReissueRequest. Each MintNode then only has to read the Spentbook entry for each DBC and validate that the entry is correctly signed by the DBC owner. This simplifies operations for the MintNode, which no longer needs to write any data, and keeps the MintNode’s reissue API idempotent. In other words, the Client can call reissue many times with the same ReissueRequest and always get the same response.

We plan to go into more detail about the Spentbook in a later update.

[1] A whitepaper was published in 2019 describing a decentralized DBC Mint system called Scrit. Scrit’s design is substantially different from ours, and it does not appear to have been publicly deployed as of this writing.
