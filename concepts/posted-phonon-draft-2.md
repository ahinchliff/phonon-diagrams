# Posted Phonons: One Directional Phonon Transfers

## Goal

Make it possible to send phonons without the sender's and recipient's phonon cards being actively paired.

## Terms

- Posted Phonon Transfer Packet - A phonon transfer packet that has been created for a particular recipient without the sender's and recipient's cards being actively paired.
- Phonon Inbox - Any service that stores posted phonons for its users and optionally provides senders the details required to create a posted phonon for an intended recipient. Similar to email, a phonon inbox can be self hosted or accessed through a service provider.

## Requirements

A "posted phonon" transfer packet must be:

1. only consumable by the intended recipient
2. only able to be consumed once
3. verifiable by any party

## Potential Solution

### Assumptions

1. A phonon card's identity private key can't ever be revealed
2. A phonon card can export it's certificate

### Properties of a posted phonon transfer packet

- Sender's card's certificate
- Recipient’s card's public key
- Nonce
- Collection of phonon data
  - Public key
  - Currency type
  - Currency value
  - Metadata
  - Private key encrypted with the Recipient's cards public key.
- Message signed by the sender's card's private identity key containing:
  - Recipient's cards public key
  - Nonce
  - Hash of the phonon collection

### New phonon card functionality

The phonon card will need the ability to:

- construct a posted phonon transfer packet
- consume a posted phonon transfer packet
- return its next valid nonce

### How are the requirements met?

**How is a posted phonon transfer packet verifiable by any interested party?** It can be verified that the transfer packet was produced by a valid phonon card by checking 1) the sender's certificate is signed by a valid issuer 2) checking the signed message was produced by the public key listed in the sender's certificate. These checks can be conducted by any party with a list of known valid issuers. If these two checks are passed then it is guaranteed by the Phonon Protocol that the phonon transfer packet is valid and it contains the data it claims (recipient, nonce, phonons).

**How is a posted phonon transfer packet only consumable by the intended recipient?**
The phonons (private keys) within the transfer packet are encrypted using the recipient's card's public key and therefore can only be decrypted by the recipient's card.

**How is it enforced that a posted phonon transfer packet is only consumed once?**
A transfer packet contains an integer nonce. When a transfer packet is successfully consumed the consuming card will set its internal nonce to match that of the successfully consumed transfer packet. A card will refuse to consume any transfer packets that have a nonce less than or equal to its internal nonce. It is the responsibility of a party external to the Phonon Protocol to coordinate the issuance of nonces.

## Nonce issuance and consumption ordering

To prevent certain attack vectors it is essential that phonon cards

- can only consume phonon transfer packets with a nonce greater than its current internal nonce
- are not required to consume nonces in consecutive order

The above requirements have the potential for cards being unable to consume valid posted phonon transfer packets. Consider the following example where the recipient has a nonce of 0.

1. The Recipient asks Sender A to construct a transfer packet with a nonce of 1 and asks Sender B to construct a transfer packet with a nonce of 2.
2. Sender B sends their transfer packet.
3. The recipient consumes Sender B's transfer packet.
4. Sender A sends their transfer packet.
5. The recipient attempts to consume Sender A's transfer packet but it is rejected by the card.

In the above example the funds from Sender A are lost forever. This could be avoided by not consuming the transfer packet from Sender B until the transfer packet from Sender A has been consumed. The issue is the recipient has no idea how long to wait or if the packet will ever be received.

To overcome this it is essential that the sender and recipient agree on how long a nonce is valid for. This is outside the scope of the Phonon Protocol but whenever a nonce is issued a TTL should be provided.

**User A and User B want to buy from an online store.**

1. User A sends an order request and the store responses with a nonce of 1 and a expiry date for that nonce
2. User B sends an order request and the store responses with a once of 2 and an expiry date for that nonce
3. User B creates a phonon transfer packet and posts it to the store over HTTP
4. The store waits until the nonce it sent to User A expires. Once nonce A has expired the store is safe to consume User B's phonon transfer packet. The store's card consumes User B's phonon transfer packet and increments its nonce to 2.
