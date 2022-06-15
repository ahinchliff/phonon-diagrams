# Posted Phonons: *One Directional Phonon Transfers*

## Goal

Make it possible to send phonons without the sender's and receiver's cards being actively paired. Ideally, the transaction data is simply POSTed over HTTP. Creates tons of flexibility for the receiving party. For example:

1. Online stores - don't need to have cards attached to servers to receive payment
2. Entertainers, like live streamers, could receive tips without having their card processing each transaction in that moment, but accumulating over the show
3. POS - Staff don't need cards plugged into POS machine
4. Simple sending. Every dev knows how HTTP works, so who knows what cool stuff people can come up with.
5. Allow for cold storage and for a user to have more phonons than a single card can hold.

## Constraints

We all agree the following must be true for any design to work:

1. Double spending must not be allowed. Receiving card must never be able to consume and spend the same phonon twice
2. Sent phonons must only be consumed by the intended receiving party
3. Avoid losing funds by limiting or eliminating use cases where sent phonons cannot be consumed by receiving party

## Overview of a possible solution

### Protocol
- Add to the protocol an *integer nonce* which exists globally on the card.  Initialize to zero.

- Add to the protocol a method that **transfers** a phonon *from* a card *to* a "Posted Phonon" packet. A "Posted Phonon" packet is a phonon that exists outside of a phonon card (can be posted to servers, websites, etc), but has the following additional features:
  - Contains an integer nonce
  - Can only be collected by a known and singular receiving card (signs by public key)

- Add to the protocol a method that **collects** a "Posted Phonon".
  - The method rejects and disallows the consumption of a "Posted Phonon" packet if:
    1. The packet is not signed for the receiving card
    2. The packet's integer nonce > the card's integer nonce  
  - If successful, the card's integer nonce *is set* to the packet's integer nonce value

### Client
- Add to the client a check that prevents the sender from executing the above *transfer* method from executing if TTL (time to live) is breached.  (Note: we could move this to the protocol but then the TTL threshold must be defined and hard coded)

### Service
- Card holders who want/need to collect "Posted Phonons" enlist the support of a service to:
  - Provide senders the details required to create a "Posted Phonon" packet:
    1. Receiver's card's public key
    2. Nonce - Increments by 1.  
    3. TTL - time to live
  - Validate incoming transfer packets (asset value)
  - Store the packets until the receiver is ready to consume them.
    1. The service should prevent a receiving card from consuming a "Posted Phonon" whose nonce is greater than a nonce for an outstanding request packet with a live TTL.
    2. The service sorts "Posted Phonons" in ascending FIFO order for consumption, so that the receiving card consumes packets with lower integers first.
- When a receiving card establishes/changes service providers, it provides the service the value of its card's integer nonce.  


## Example Process Flows

**User A and User B want to buy from an online store.**

Success

1. User A sends an order request and the store responses with a nonce of 1 and a expiry date for that nonce
2. User B sends an order request and the store responses with a once of 2 and an expiry date for that nonce
3. User A creates a phonon transfer packet and posts it to the store over HTTP
4. User B creates a phonon transfer packet and posts it to the store over HTTP
5. The store's card consumes User A's phonon transfer packet and increments its nonce to 1.
6. The store's card consumes User B's phonon transfer packet and increments its nonce to 2.

Success

1. User A sends an order request and the store responses with a nonce of 1 and a expiry date for that nonce
2. User B sends an order request and the store responses with a once of 2 and an expiry date for that nonce
3. User B creates a phonon transfer packet and posts it to the store over HTTP
4. The store waits until the nonce it sent to User A expires. Once nonce A has expired the store is safe to consume User B's phonon transfer packet. The store's card consumes User B's phonon transfer packet and increments its nonce to 2.

Failure

1. User A sends an order request and the store responses with a nonce of 1 and a expiry date for that nonce
2. User B sends an order request and the store responses with a once of 2 and an expiry date for that nonce
3. User B creates a phonon transfer packet and posts it to the store over HTTP
4. The store fails to wait for the nonce it sent to User A to expire. The store's card consumes User B's phonon transfer packet and increments its nonce to 2.
5. User A creates a phonon transfer packet and posts it to the store over HTTP.
6. The store's card attempts to consume User A's transfer packet but it is rejected as the nonce is too low.

Failure Prevention

1. User A sends an order request and the store responses with a nonce of 1 and a expiry date for that nonce
2. User B sends an order request and the store responses with a once of 2 and an expiry date for that nonce
3. User B creates a phonon transfer packet and posts it to the store over HTTP
4. The store waits for the nonce it sent to User A to expire. The store's card consumes User B's phonon transfer packet and increments its nonce to 2.
5. User A is prevented from creating a phonon transfer packet because TTL has expired; and has no packet to post.

Success
1. User A sends an order request and the store responses with a nonce of 1 and a expiry date for that nonce
2. User B sends an order request and the store responses with a once of 2 and an expiry date for that nonce
3. User B creates a phonon transfer packet and posts it to the store over HTTP
4. User A creates a phonon transfer packet and posts it to the store over HTTP
5. The stored phonon transfer packets are stored and sorted by nonce
5. The store's card consumes User A's phonon transfer packet and increments its nonce to 1.
6. The store's card consumes User B's phonon transfer packet and increments its nonce to 2.


## Questions

1. Will the service receiving the phonon transfer packets be able to validate a phonons content and that it comes from a valid phonon card?
2. Are there any attack vectors outside your typical web service attacks?
