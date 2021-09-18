



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
