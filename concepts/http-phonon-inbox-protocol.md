# HTTP Phonon Inbox Protocol (HPIP)

A **Phonon Inbox** is a service that manages the issuance, transfer and storage of _posted phonons_. This document presents an HTTP-based implementation of a Phonon Inbox, with the intent to discover and define a standard that governs all implementations (the data required to create and consume posted phonons can be transmitted over any medium, not just HTTP). Like how email clients can interact with email providers who use the same standards - no matter if they implement SMTP, POP3 or IMAP - this HPIP hopes to layout a standard that is flexible, open and makes it easy for developers to make tools that are interoperable and support the phonon network.

The protocol includes three pieces of functionality:

- **Request a Slot** - A sender requests the details required to create a posted phonon for a particular recipient from a phonon inbox
- **Post a Phonon** - A sender sends a posted phonon transfer packet to a phonon inbox
- **Consume Phonons** - A recipient requests phonon transfer packets from a phonon inbox for consumption
- **Mark phonons as consumed** - The recipient informs the phono inbox services that phonon transfer packets have been successfully consumed.

## Endpoints

// Todo - Include details about error responses, Consume a phonon

### Request a slot (POST)

To create a posted phonon a sender requires the recipient’s public key and a valid nonce. The “request a slot” endpoint returns these details as well as a time to live (TTL) that indicates how long these details are valid for. If public facing, the phonon inbox service has the option to confirm the request is coming from a valid phonon card and reject future requests if abuse is detected.

#### Request Body

- inbox:
  - The unique identifier of the recipient
- sender card certifications (optional):
  - The sender’s phonon card’s cert
  - The sender's phonon card’s public key
  - A signature that proves sender owns phonon card’s public key
- sender id (optional):
  - The unique identifier of the sender
    - _Example_: The phonon inbox’s domain signed by sender's phonon card’s key
- memo (optional):
  - Information the inbox can use to screen or relay sender requests
    - _Example_: Burner code issued to sender to be permitted a response

#### Response Body

- nonce - A unique integer
- publicKey - The recipient’s card’s public key
- ttl - Timestamp indicating when the nonce is no longer valid
- postUrl - The url that should be used when sending the phonon transfer packet

### Post a Phonon (POST)

#### Request Body

The user sends the created phonon transfer packet to the “postUrl” received from the “Request a slot” step.
Request Body

- inbox - The unique identifier of the recipient
- packet - The phonon transfer packet
- data (optional) - string that can be used to pass data to service

#### Response Body

null

### Consume Phonons (POST)

The recipient requests the next phonon transfer packets to be consumed. The inbox will only return a phonon if all lower nonces have been consumed or their ttl have expired.

#### Request Body

- authorisation - TBD
- markAsConsumed (optional) - A collection identifying which packets should be marked as consumed before returning the next collection of transfer packets. See "Mark phonons as consumed"
- count - The max number of transfer packets that should be return

#### Response Body

- packets - A collection of transfer packets

### Mark phonons as consumed (POST)

The recipient informs the Phonon Inbox that particular packets have been successfully consumed.

- authorisation - TBD
- markAsConsumed - A collection identifying which packets should be marked as consumed before returning the next collection of transfer packets

## Sharing Phonon Inbox Details

To recieve funds a recipient needs to share their "Request a slot" url with senders. This is not user friendly.

### Phonon Inbox Addresses

[inbox]$[domain]

examples: personal$hinchy.com, hinchy$phononinbox.com

A phonon inbox address identifies:

- what domain should be used to lookup the "Request a slot" endpoint
- the inbox (unique identifier for the recipient)

A phonon wallet uses the domain section of the phonon inbox address to check the DNS records for that domain. The TXT record "phononInboxRequestSlotUrl" determines what url should be used to call the "Request a slot" endpoint

### ENS

Similar to DNS, a user can add a record to their ENS name. A phonon wallet would attempt to look up this record if an ENS name is entered into the recipient input.

## Questions

1. Does a Phonon Inbox Address require the inbox section (unique identifier)? Can a Phonon Inbox identify the intended recipient using the recipient's card's public key in the transfer packet?
