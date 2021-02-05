---
title: A Lesson in Signatures, an Experiment in Revolution | (Simple)Math Behind Bitcoin 
---

When you think about it, signature is a really weird concept to get your head around. When you buy/own something, you scribble some weird looking letters on a paper(that can easily be copied), that paper acts as a document or proof that you own the thing you buy/own. 

In the modern world, signature represents you as a unique identity. 


## Rise of Wax Seals and Signature

The wax seal became more popularized in the Middle Ages. It was typically used by nobility or clergy members, such as monarchs, royal representatives, and bishops. A wax seal often provided a level of authentication to a document, such as a contract or an official letter. Like signatures, each seal was completely unique to its owner. Though the Latin alphabet was already invented, a lot of people remained illiterate. As the wax seal gained more popularity and began to be more frequently used. Among common people, it was a good alternative for those who didn’t know how to read or write.

![image](/img/signature.jpg)

Modern signature did not became what it is today until 1677 with The Statue of Frauds Act, passed by English parliament. The Act required contacts to be signed by parties and that signature was binding of parties to the contract. Later in 1980s, when fax machines became more common in business and Daily life, contracts started being sent electronically. Because of this, laws changed in many countries in order to adapt to this new technology. In order to ensure the legality and authenticity of these electronically sent documents. It was time, for the electronical, cryptographic signatures to enter our lives. Much like the handwritten signature, it is very well possible to impersonate someone elses e-signature. After all, they are a bunch of ones and zeroes, right?

## Digital Signatures and Public Key Infrastructure

Like written signatures, idea is that someone should be able to perform or confirm an action in his/her name and since this is electronical, it should be infeasible for anyone else to forge his/her signature. There are 2 important elements while building a system like this. 
1-	Security and Authenticity: Signatures should be unique and inimitable
2-	Performance: It should be conveniently fast, no one will wait for you to sign something for hours.

In the system of Public Key Infrastructure(PKI), everyone generates a private key and a public key. These keys are both bunch of ones and zeroes and typically they consist of 256 bits. Private keys are also called Secret keys for convenience. Secret keys are used for signing of a digital document and public keys are used for verifying of a digital document. Although algorithms performs this task differently, general idea is something like this.

{% highlight javascript linenos %}
Sign(Message, SecretKey) = Signature
Verify(Message, Signature, Public Key) = True OR False
(If true, it is signed, If false, it is not signed)
{% endhighlight %}

For the curious, formula and complex mathematics behind this is explained [here](https://en.wikipedia.org/wiki/Digital_Signature_Algorithm)
Now, for someone to imitate someone elses signature, there are 2*256 combinations to try. This number is so huge that there are no correct english words to describe it. It would be impossibly low of a chance to “guess” someone elses digital signature. For this reason, using a digital signature is much more secure than using a handwritten one. So, why not build a system out of this?


## Bitcoin Formula

In simplest terms, Bitcoin is a ledger system which keeps transaction between users and uses cryptographic tools to circumvent the need for trust between parties.

Bitcoin = Ledger – Trust + Cryptography

Let’s imagine a group of friends, X, Y, Z. These friends often lend each other money. They decide to come up with a list(ledger) that holds transactions for each payment, so they can observe who lent money to whom and how much. Initialy, they all have $50. To be able to write into this list, they all have to use their signature(digital signature) in order to prove that they are who they claim to be. 
Here are some transactions between them;

X pays $10 to Y.


Y pays $20 to Z.



After these transactions, X has $40, Y has $40 and Z has $60. Bitcoin ledgers also works like this. It follows the transactions that are made in the list to calculate who has how much. Now X, Y and Z could be really good friends and agree on a consensus, but in real world, for a system to be enable reliable currency exchange, trust is the most important aspect to present. What if X decides to spent more than he/she has? What if Y decides to alter the ledger alltogether? Who controls the rules of adding new lines to the list?

## Building Trust or… removing it?

To remove this trust issue, Bitcoin enables all of its users to keep their own copy of the list. Then, when X wants to make a transaction like “X pays $20 to Z”, he/she broadcasts this request to the world for people to hear and record on their own list. But, how can anyone be sure that they all hear the same request? How can everyone agree on what the correct list is? How do you agree on a consensus? For this particular issue, we must first learn about a specific mathematical function called hashing function.

A hash function is a mathematical function that converts a numerical input value into another compressed numerical value. The input to the hash function is of arbitrary length but output is always of fixed length. Values returned by a hash function are called message digest or simply hash values. The following picture illustrated hash function;

![image](/img/hash1.jpg)

SHA-1 is an hashing algorithm. It is described in the Picture below. Note that changing letter a to A completely alters the result. 

![image](/img/hash2.jpg)

In another terms, if you pass a message to a hash function, you get a unique summary of your message. Even changing one letter in a message results in a vastly different summary, this is why hash functions are used for calculating messages authenticity. 



Now, we have a system for authenticating as who we are(signatures) and we have a system that can check messages authenticity(hashes). Last step is reaching a consensus between lots of different ledgers, and agreeing upon what to write in the ledger. Question is, how can you get everyone to agree on what the right ledger is? Imagine you are a node on the system, listening to transactions that are being broadcast. One of them says “X paid Y $50” another one saying “Y paid X $100”, which one are you going to trust? More importantly In what order are you going to trust these? This is a problem that is adressed in the original bitcoin paper.

## Proof of Work

Bitcoin’s solution to this problem is to trust whichever ledger has the most computational work put into system. The idea is, if you use computational work as a basis for what to trust, you can make it so that fraudulent transactions and conflicting ledgers would require an infeasible amount of computation. Let’s dissect this sentences with an example. Imagine a ledger like this,

X pays $40 to Y.

Y pays $30 to Z.

Z pays $25 to T.


Whoever that broadcasted this ledger adds a special number at the end of the transaction like this. 

X pays $40 to Y.

Y pays $30 to Z.

Z pays $25 to T.

7896245679645


And he/she claims that if you calculate the hash of this ledger, first 32 bits are all zeroes, like this.

![image](/img/32bits.png)

To reach a conclusion like this, it would take a lot of guessing work, trying different numbers with ledger and calculating hashes to run into a hash that has 32 zeroes in it’s first 32 bits. Since one has to get 32, probability of it would be;

![image](/img/probability.png)

This allows us to calculate that the number they come up with indeed required a lot of computational work to achieve and we can verify this easily checking the first 32 bits of the ledger’s hash. This is called **proof of work**. And a ledger is called a **block**. A block is only valid if it has a proof of work. Every block has a list of transactions between parties, and a proof of work attached to it. Every block, has the hash of the previous block before it on top of it. That way, if you change anything in any block, whole hash system would be calculated again, which would be infeasible. When you tie these ledgers, or as it called  blocks, you get chains of block, hence the name; **blockchain**.

![image](/img/blockchain.png)

Now, to arrive at the special number that is required for 32 bits of zeroes on a hash, every node can do computational work to come up with that number, and when someone guesses the correct number, they get rewarded with some bitcoin. This process is also called **mining**. This process of guessing the correct number gets more difficult as the system grows. If a node receives 2 or more blockchains that are being transmitted, it selects the longer blockchain which has more blocks because more computational work have beend done for that blockchain. 
Let’s imagine a scenario which we try to fool this system.

## Breaking the Blockchain

Suppose that X is trying to push a block into the network that contains “Y pays $100 X.” X creates this block and by luck, he finds the correct special number before anyone else. He transmits this blockchain into the network. In order to keep his blockchain the valid one, he has to keep continuing to find new blocks which has special number and he needs to be faster than everyone else because remember, in case a longer blockchain enters the game, it is respected as the valid one. 

![image](/img/fooling.png)

So, X has to keep finding correct special number for his blockchain to succeed, and in enough time, it would require more than half of computational work in the system to keep doing this. This is called **%51 attack**. This is why bitcoin is a **decentralized** crypto currency. In case one entity gains control of over %50 of the system(which is incredibly hard at this point), it would just be a centralized currency. 
Maximum amount of achievable bitcoin is 21 000 000. Since first block(which is called **genesis block**) difficulty of finding special number increases periodically and in time, bitcoin will reach it’s maximum amount. Reward for one block was 50 BTC in 2009, it’s 6.25 BTC in 2020. However, this is not the only way miners make money off of bitcoin, they can also pick up transaction fees. Whenever you make a payment, you can optionally include a small transaction fee with it that will go to the miner of whatever block includes that payment. Reason one might do this is to incentivize miners to actually include the transaction you broadcast into the next block. 










