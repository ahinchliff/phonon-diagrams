# HTTP Phonon Inbox Protocol (HPIP)

A phonon inbox is a service that stores posted phonons for its users and optionally provides senders the details required to create a posted phonon for an intended recipient. The goal of this document is to begin describing a protocol that could be adapted by future phonon inboxes and phonon wallets to create a standard. In the same way that any email clients that implement SMTP, POP3 or IMAP can interact with any email providers that implement the same standards, it is hoped that HPIP can provide similar levels of flexibility and openness. Although posted phonons, and the data required to create and consume them, can be transmitted over any medium, HPIP focuses on transmission over HTTP.

The protocol includes has three pieces of functionality:
Request a slot

- The sender requesting from a phonon inbox the details required to created a posted phonon for a particular recipient
- Post a phonon - The sender sending a posted phonon transfer packet to a phonon inbox
- Consume a phonon - The recipient requesting phonon transfer packets so they can be consumed

## Endpoints

// Todo - Include details about error responses, Consume a phonon

### Request a slot (POST)

To create a posted phonon a sender requires the recipient’s public key and a valid nonce. The “request a slot” endpoint returns these details as well as a time to live (TTL) that indicates how long these details are valid for. If public facing, the phonon inbox service has the option to confirm the request is coming from a valid phonon card and reject future requests if abuse is detected.

#### Request Body

- inbox - The unique identifier of the recipient
- cert (optional) - The sender’s phonon’s card’s cert including the phonon card’s public key
- id (optional) - The phonon inbox’s domain signed with the phonon card’s public key

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

### Phonon Inbox Addresses

[inbox]$[domain]

examples: personal@hinchy.com, hinchy@phononinbox.com

A phonon inbox address identifies:

- what domain should be used to lookup the "Request a slot" endpoint
- the inbox (unique identifier for the recipient)

A phonon wallet uses the domain section of the phonon inbox address to check the dns records for that domain. The TXT record "phononInboxRequestSlot" determines what url should be used call the "Request a slot" endpoint
