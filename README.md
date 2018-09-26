# Abstract

*sessionproof* will make it easy to generate [non-interactive zero-knowledge proofs](https://en.wikipedia.org/wiki/Non-interactive_zero-knowledge_proof) for TLS sessions.

You can use this to prove that you are the owner of any online account (including online banking),
without revealing your username or password.

#### Note: These proofs will require servers to send [HTTP Signatures](https://tools.ietf.org/html/draft-cavage-http-signatures-09), which are a new IETF specification.

(View the [Non-Repudation Solutions](#non-repudation-solutions) section for some more ideas.)


-------------------------------

## Introduction

Many financial services offer to verify your bank account by asking for your username and password. They sign in on your behalf and check your account info. Some services that do this are
[Coinbase](https://www.coinbase.com/), [TransferWise](https://transferwise.com/),
and [Xero](https://www.xero.com/).

Providing your username and password is generally a terrible idea, but there aren't many
viable alternatives. Some services will send a few transactions and ask you to confirm the amounts,
but it can take a few days before these transactions appear in your account.

A modern bank might provide an [OAuth token](https://oauth.net/2/) with limited read-only permissions.
This would be an ideal solution, but banks don't move very fast, and many banks will probably never
implement this.

*sessionproof* will be a proof-of-concept solution that can verify your account on any website,
without providing your username and password to anyone. You will use the *sessionproof* browser to log in to your banking website on your own computer. The software will then generate a "proof" for the following
information:

* You visited www.xyzbank.com
* Their SSL certificate was valid and signed by a known root authority
* You successfully signed in
* The server sent you an HTML page that included your full name, your phone number, and your bank account number.

It's important to note that this is the **only** information the proof contains. It doesn't reveal
anything about your username or password.

You can then upload that proof to any third-party website, and they can instantly verify your bank account details.


## How would I verify a proof?

We will provide verification libraries for various programming languages.
Any third-party service could use one of our verification libraries to confirm
that a given proof is valid.


## Is this just for online banking?

You will be able to use *sessionproof* to securely verify information on any website that supports
the creation of these proofs by including a digital signature with the response.
You will open the *sessionproof* browser and visit the website, and sign in with your username and password (if necessary.) You will then be able to highlight any information on the page that you would like to include
in the verification. *sessionproof* will produce a small file that cryptographically verifies the presence of this information, the validity of the TLS session, and *nothing else*.

## Could someone create a fake proof?

Only if you can take over the server and change the website's SSL certificate. (If you can manage to do that, the bank has a serious problem.)

A proof will include the complete SSL handshake, including the website's public SSL certificate.
A proof verifier would always make an SSL connection to the same domain, to confirm that the proof includes a matching SSL certificate.

This means that you would be able to install a self-signed SSL certificate to trick the *sessionproof* browser, but your proof wouldn't convince anyone else.

> Note: We would need to add some way to verify SSL certificates if they are expired or revoked
> in the future. It might be better to just check the root cert of the certificate authority.


---------------------------

## Advanced Proofs

You will be able to create a proof that verifies a certain *fact* about the data on your webpage,
without needing to disclose the actual data. For example, you could prove that a bank account
contains *at least* $100, without giving away the actual balance.

Another example: You could create a proof that verifies your account number, full name, email address,
and phone number, without actually disclosing that information. The only way that someone could
verify this proof is if they already know all of your personal information. Otherwise,
the proof file reveals absolutely nothing about your identity.


## Potential Applications

### Anonymous Social Proof

* You could sign into Gmail to prove that you control an email address.
* You could prove that the same email address is associated with a Facebook account that is at least 3 years old, and has at least 100 friends.
* You could prove that the same email address is associated with a Twitter account that is at least 3 years old, and has at least 100 followers.
* You could prove that the same email address is associated with a GitHub account that is at least 3 years old, has at least 10 repositories, and that one of those repositories has at least 100 stars.
* You could prove that you meet 2 out of 3 of the above requirements. (At least 2 of Facebook, Twitter, GitHub.)

Finally, you could prove all of the above (and only the above) *without ever sharing your actual email address*.

> Note: This proof works because it is anchored to the SSL certificates used by
> mail.google.com, facebook.com, twitter.com, and github.com. A verifier would trust that
> those servers had not been compromised, and that they were sending accurate information.

### Mutual Credit Currency as an Ethereum Contract

* Alice and Bob go to the movies, and Alice tells Bob that he can pay her back later. Bob now owes Alice $10 for the movie ticket.
* Jane and Alice go out to get some burritos. Jane pays $10 for Alice's burrito and drink,
and Alice promises to pay Jane back later.
* Jane and Bob go out for coffee. Jane forgets her wallet, and Bob tells Jane that she can pay him back later.
Jane orders a latte and some food for $10.

If each of these debts was recorded on the Ethereum blockchain and tied to distinct [Anonymous Social Proofs](#anonymous-social-proof) with a certain "trust threshold", then all of the above debts would cancel out, and no-one would owe any money. This concept could be extended to a global network based on trust.

This is very similar to [BeerCoin](https://www.reddit.com/r/ethereum/comments/4v7opj/introducing_beercoin/), an Ethereum contract that was created as a joke. *sessionproof* could provide a solution to the [identity problem described in this comment](https://www.reddit.com/r/ethereum/comments/4v7opj/introducing_beercoin/d5w9n8l/).

#### More information:

* [Hawala](https://en.wikipedia.org/wiki/Hawala)
* [Mutual credit](https://en.wikipedia.org/wiki/Mutual_credit)
* [Mutual Credit Cryptocurrencies: Beyond Blockchain Bottlenecks](http://ceptr.org/whitepapers/mutual-credit)


### Anonymous Online Voting

You could include an anonymous vote as one of your social proof outputs.
For elections, the proof might include accounts on
government websites, such as the [IRS Account Information page](https://www.irs.gov/payments/view-your-tax-account).

Enforcing one vote per person would be a difficult problem, especially because it is easy to create
fake online accounts and email addresses. It might be possible for each proof to provide their email address as an encrypted output. A "duplicate checker" circuit could be created that detects any duplicate votes
by checking each pair of encrypted emails. This would require a key ceremony similar to the [Zcash ceremony](https://z.cash/blog/the-design-of-the-ceremony.html), but then anyone
could verify the results by checking that the match the public key.


## Non-Repudation Solutions

### 1. HTTP Signatures

The TLS protocol is a very secure way to protect against eavesdroppers. It also
prevents [Man-in-the-middle attacks](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)
and [Replay attacks](https://en.wikipedia.org/wiki/Replay_attack).
However, TLS can not provide the property of [non-repudiation](https://en.wikipedia.org/wiki/Non-repudiation)
for any data that is transmitted after the TLS handshake.

This can be solved at the [application layer](https://en.wikipedia.org/wiki/Application_layer)
if the server sends [digital signatures](https://en.wikipedia.org/wiki/Digital_signature) that
verify both the request and response data. The IETF is working on a [specification for "Signing HTTP Messages"](https://tools.ietf.org/html/draft-cavage-http-signatures-09), which is a standard for
adding `Signature` headers to HTTP messages.

A web server could implement HTTP signatures to provide [non-repudiation](https://en.wikipedia.org/wiki/Non-repudiation) for the TLS session.
Note that the signature in the response must include the complete request (including headers),
as well as the response body, and all response headers (except the signature).
This verifies that the server produced the given response for a specific request.
(If the server only signed their response, the client would be able to alter their request data.)

An Nginx or Apache plugin could be created that automatically adds these digital signatures.
A server administrator could install this plugin in a few minutes, so this would be
much easier that implementing something like OAuth 2.


### 2. Hosted Proxy Service

*sessionproof* could provide a hosted proxy service that would record TLS sessions,
and sign the recorded session using a private key to verify that the client did not change any data.
*sessionproof* would not be able to view any of the transmitted data, but a third-party service
would have to trust that *sessionproof* did not collude with the user to sign some forged data.

Alternatively, a third-party service could run the proxy software on their own servers.
This solution would provide excellent security for both the user and the third-party service.
The proof would be just as strong as the user providing their username and password,
and the third-party service signing on their behalf.
This technique does not require HTTP Signatures,
and online banking websites would not need to make any changes.


---------------------------


### GC/ZK Libraries

* [obliv-c](https://github.com/samee/obliv-c)
* [libsnark](https://github.com/scipr-lab/libsnark)
* [emmy](https://github.com/xlab-si/emmy)
* [FastGC](https://github.com/yhuang912/FastGC)
* [yao](https://github.com/hcrypt-project/yao)


### References

* [Secure multi-party computation](https://en.wikipedia.org/wiki/Secure_multi-party_computation)
* [Yao's Millionaires' Problem](https://en.wikipedia.org/wiki/Yao%27s_Millionaires%27_Problem)
* [A Gentle Introduction to Yao’s Garbled Circuits](http://web.mit.edu/sonka89/www/papers/2017ygc.pdf)
* [Garbled Circuits (wikipedia)](https://en.wikipedia.org/wiki/Garbled_circuit)
* [ZKBoo: Faster Zero-Knowledge for Boolean Circuits](https://eprint.iacr.org/2016/163.pdf)
* [Ligero: Lightweight Sublinear Arguments Without a Trusted Setup](https://acmccs.github.io/papers/p2087-amesA.pdf)
* [Signing HTTP Messages (IETF Specification)](https://tools.ietf.org/html/draft-cavage-http-signatures-09)
* [The First Few Milliseconds of an HTTPS Connection](http://www.moserware.com/2009/06/first-few-milliseconds-of-https.html)
* [Non-repudiation](https://en.wikipedia.org/wiki/Non-repudiation)
