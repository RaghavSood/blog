---
layout: post
title: Why Bitcoin Changed The Game
categories:
- blog
---

Bitcoin did not spawn out of nothing. The fundamental building blocks of Bitcoin were not new, but instead combined in a novel way to fix shortcomings in the individual components. This post aims to shed a little light on where the basics of Bitcoin originated, and why Bitcoin was able to succeed in creating a peer-to-peer electronic cash system where others before it failed.

The properties that we are interested in are defined in the introduction of the Bitcoin whitepaper as  

 > ... an electronic payment system based on cryptographic proof instead of trust, allowing any two willing parties to transact directly with each other without the need for a trusted third party. Transactions that are computationally impractical to reverse would protect sellers from fraud, and routine escrow mechanisms could easily be implemented to protect buyers. In this paper, we propose a solution to the double-spending problem using a peer-to-peer distributed timestamp server to generate computational proof of the chronological order of transactions. The system is secure as long as honest nodes collectively control more CPU power than any cooperating group of attacker nodes.[^1]

 This introduction contains the essence of what Bitcoin is, and also conveniently lists the properties that we are interested in:

 1. Cryptographic proof of trust (digital signatures) 
 2. No trusted party (peer to peer)
 3. Transactions are irreversible (proof of work)
 4. Chronological ordering of transactions based on computational proof (timestamping proof of work)

## Prior Art 

### Digital Signatures

Digital signatures predate Bitcoin by far, and this post does not aim to delve into cryptography. Instead, we will focus on their applications to digital cash. Prior to Bitcoin, there were numerous attempts at digital cash, some of them even seeing brief real-world use, such as [DigiCash](https://en.wikipedia.org/wiki/DigiCash). However, these system had shortcomings. For one, they relied on a centralized service. If this service was to go down, all digital transactions would be halted. Additionally, they were unable to cleanly solve the double spending problem. Regardless, the base for digital signatures used for digital cash was established long before Bitcoin.

#### Double Spending

Understanding what double spending is is essential to understanding why Bitcoin succeeds. By solving it in a decentralized manner, it has fulfilled one of the major gaps in prior digital cash systems (and even real cash systems).

In early digital cash system proposals, "cash" was simply a value with a digital signature. This led to the problem that the signed value could easily be duplicated and sent multiple times. In order to prevent this, the notion of unique identifiers was introduced, where cash not only had a value, but also a unique identifier. This value+identifier combination was then signed as one unit. This, in turn, led to another problem - before accepting a payment, the recipient would have to contact whoever issued that cash and ask them if the cash with a given identifier had already been spent. This works well if the issuer is contactable. If it has not been spent, the recipient accepts it and the issuer marks it as spent. If it has been spent, the recipient rejects it. However, if the issuer is uncontactable, the validity of the cash cannot be verified, and no transaction can safely take place. Additionally, after collecting cash, the recipient must now go back to the issuer and have the received cash's value reissued with a new unique identifier to be able to spend it further.

Bitcoin utilized the same digital signatures as proof of ownership, but combines them with a distributed ledger maintained via a proof of work algorithm that prevents double spending. I will go into details on this in the proof of work section below.

### Peer to peer

Once again, peer-to-peer (or p2p) systems were nothing new when Bitcoin was announced. In fact, Bitcoin's implementation of it has next to no notable features. It is simply used as a carrier layer for the blocks and transactions. Numerous p2p systems existed before, such as [ARPANET](https://en.wikipedia.org/wiki/ARPANET), [Usenet](https://en.wikipedia.org/wiki/Usenet), [SMTP](https://en.wikipedia.org/wiki/Simple_Mail_Transfer_Protocol), etc.

### Proof of Work

This is where half the magic of Bitcoin happens. Proof of work (or PoW) was first proposed in a 1993 article[^2] by Cynthia Dwork and Moni Naor. It's most well-known use prior to Bitcoin is perhaps in Hashcash[^3], a email spam combatting system designed by Adam Back. It is also likely from where Satoshi got the idea, as it is credited in the Bitcoin whitepaper.

Proof of work is essentially a problem-solving game. In the simplest form, you must repeatedly solve a mathematical puzzle with different inputs until the output matches certain validation criteria. This validation criteria is changed to increase or reduce the difficulty of finding the solution. Moreover, each attempt must have an equal probability of being the solution for a given set of validation criteria. Lastly, proof of work should be hard to find, but easy to verify. This is usually done by employing a relatively inexpensive hash function as the PoW algorithm, which is easy to compute for a single value. Thus, one needs to compute several values to find a valid hash, but only needs to verify input to the valid solution.

Both Bitcoin and Hashcash rely on a very similar proof of work model, where some data (the block header, in the case of Bitcoin) is hashed along with a nonce value repeatedly until the answer falls below a given difficulty level. By including the block hash as an input to the puzzle, Bitcoin prevents past work from being reusable for new blocks, thus ensuring that everyone must do new work each time they wish to mine a block.

The primary issues with proof of work in systems such as hashcash came about in usability. Hashcash was designed as an email spam combating measure, where each email had a proof of work attached to it, with the amount of work needed scaling up as CPUs became more powerful. This was done with the thought that for normal users sending only a few emails, it would pose no limitation. However, for spammers sending thousands or millions of emails, the cost of computation would quickly exceed what was feasible economically or practically.

Unfortunately, most spammers utilized botnets, and would thus be able to rapidly send emails anyways (although Hashcash would certainly slow them down a bit). Moreover, as CPU power increased across average computers, users with older models would fall behind and find it extremely slow to send emails as the hashcash computation requirements went up, forcing otherwise unnecessary upgrades.

### Timestamping

In any financial system, maintaining a chronological order of transactions is essential. Without this order, things quickly breakdown as people are spending money they haven't received, or receiving money from people who have none.

Like everything else so far, timestamping is not a new concept. Timestamping was proposed all the way back in 1990 by Haber and Stornetta[^4]. They proposed a central service to which an end user may send documents. This central service would then affix a digital signature over the document and the current timestamp, along with a point or link to a previous version of the document. A later paper improved upon this design by batching documents into blocks and signing and linking the blocks instead[^5] (sound familiar yet?).

However, once again, this service depended on the availability and honest of a centralized timestamping system. 

## Bitcoin

The above prior art is only a short look at what existed before Bitcoin. There is much more to this history than what is covered above, and further information can be found in the Further Reading section below. Now, let's get on to why Bitcoin is so exceptional.

To see why Bitcoin is able to achieve something new, we should look at it being built layer by layer.

The very first layer is that it is peer-to-peer. This removes reliance on a central authority, as each peer independently stores and validates transactions and blocks. Any peer can start mining the proof of work algorithm and submit new blocks to the network.

Within this peer to peer system, there are transactions. Transactions have inputs and outputs (much like DigiCash). Each input is must correspond to a previous output in the blockchain, and each output can only be used in a single input further along in the chain. This rule is enforced by each peer in the network, and transactions attempting to use an output that has already been used in another transaction again are rejected. Finally, each input must be accompanied by an appropriate digital signature from the address it resides in (this is an oversimplification, but suffices for the essence of this post). This solves the double spending problem by allowing each input to be used only once, and enforcing it via a peer to peer system of validation.

Miners are able to batch these transactions into blocks by solving the mining puzzle. To solve the puzzle, they must make a list of transactions they wish to include in their block, link them all into a [merkle tree](https://en.wikipedia.org/wiki/Merkle_tree), and use the block hash of the previous block, the root of the merkle tree, and a few other metadata fields to build a block header. This block header is unique to each block, authoritatively defines which transactions are in a block via the merkle root, and links it to the previous block in the chain. This block header is hashed in combination with random nonce values until the result of the hash satisfies the current difficulty requirement of the network. The difficulty requirement is adjusted every 2016 blocks (or every two weeks) in a manner that there is, on average, one solution every ten minutes, and thus one block every ten minutes. As more miners join and perform more computation, blocks are found faster than every 10 minutes. When the next difficulty adjustment happens (every 2016 blocks) the difficulty will be increased so that the same computational power produces a solution every 10 minutes again. Any block submitted that does not meet the difficulty requirement is rejected by the network. As a reward for finding a valid block, miners are allowed to credit a certain amount of new Bitcoin (that decreases with time) to themselves, along with any transaction fees for the transactions included in their block.

By using proof of work to limit the creation speed of blocks, Bitcoin creates its own timestamping system. As each block contains transactions, and each block appears after another block, each transaction is timestamped into the blockchain in a specific, verifiable order, without relying on a central timestamping system.

When all of these layers are combined, you have a peer-to-peer electronic cash system.

### References

[^1]: Nakamoto, S. (2008). *Bitcoin: A peer-to-peer electronic cash system*.
[^2]: Dwork, C., & Naor, M. (1992, August). *Pricing via processing or combatting junk mail*. In Annual International Cryptology Conference (pp. 139-147). Springer, Berlin, Heidelberg.
[^3]: Back, A. (2002). *Hashcash-a denial of service counter-measure*.
[^4]: Haber, S., & Stornetta, W. S. (1990, August). *How to time-stamp a digital document*. In Conference on the Theory and Application of Cryptography (pp. 437-455). Springer, Berlin, Heidelberg.
[^5]: Bayer, D., Haber, S., & Stornetta, W. S. (1993). *Improving the efficiency and reliability of digital time-stamping*. In Sequences II (pp. 329-334). Springer, New York, NY.
