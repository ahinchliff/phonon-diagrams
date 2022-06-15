# One Directional Phonon Transfers

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

- Add to the protocol the concept of a "one directional" transfer packet which will contain an integer nonce and a time to live (ttl)
- Add ability for phonon cards to consume "one directional" transfer packets if a packet is signed for that card and the packet has a nonce > the card's "one directional" nonce
- A service outside of the protocol is responsible for providing senders with the details required to create a "one directional" transfer packet for a receiver (receiver's card's public key, nonce, ttl). The same service would also be required to validate incoming transfer packets and to store these packets until the receiver is ready to consume them. The service should not allow a user to consume a transfer packets if there exists a lower nonce with a live ttl.

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

## Questions

1. Will the service receiving the phonon transfer packets be able to validate a phonos content and that it comes from a valid phonon card?
2. Are there any attack vectors outside your typical web service attacks?

## Implementation Notes

Assume the following about posted one directional phonons:

1. Must be submitted with a `receiptID` expressed as an integer

Add the following features to the `card APDU`:

1. Allot space on the card (not within each phonon but at the card level) to store an `integer`, and name it something like `lastReceipt` or `witnessedReceipt` or `receiptWatcher` or `receiptTracker`. Initialize this attribute to `0`.

2. Add a new APDU method and call it `ReceiveOfflinePhonons`:

- It can consume a one directional offline phonon that has been posted by a sender
- It expects a `receiptID` in the offline phonon
- It sets the `receiptWatcher` to the value of the `receiptID` after consuming the offline phonon
- **BUT** it will **NOT** allow an offline phonon to be processed if its `receiptID` is <= the `receiptWatcher`.
