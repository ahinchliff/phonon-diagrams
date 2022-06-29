# Phonon Token

## Phonon types

- **Native Phonon** - A phonon that is mined on a phonon card.
- **Backed Phonon** - A phonon that is created and then assigned a value. The Phonon Protocol has no concept of how the value is derived.
- **Eccumbering Phonon** - A phonon thats key pair encumbers something of value.
- **Attested Phonon** - A phonon that derives its value from an authority assigning it legitimacy and traits. Similar to the US Gov backing USD notes and coins.

## Problem

Both native and backed phonon cannot be combined or divided into new denominations. This means when transacting either the sender needs to be able to provide exact payment or the receiver needs to be able to issue exact change. The concept of a "change maker" service has been discussed but this requires one of the parties to be online and to interact with a third party.

## Goal

To design a way in which attested phonons can be combined and divided to allow for exact payment of any value.

## Solution

The concept of a "Phonon Token" would be introduced to the phonon applet. A phonon token is not a key pair and no part of it is ever "sent" to other phonon cards. It is a data structure that includes an issuer public key as an identifier and a "balance" property that can be altered.

### Creation

Anyone can mint "Phonon Tokens” by calling `createPhononTokens` with a private key and a value. The phonon applet will derive the tokenId from the private key. If the card is tracking a "phonon token" with that tokenId it will increase the balance by the value argument. If not, the applet will create a new data structure setting the tokenId to the public key and the balance to the value argument.

### Sending

`sendPhononTokens` is called with the tokenId and value to be sent. The applet decreases the token balance by the value and creates a transfer packet. If the token balance is zero after creating the transfer packet the token structure can be deleted from the card.

### Consumption

`consumePhononToken` is called with the transfer packet. The recipient's balance is adjusted in the same way as during the creation step.

### Data Structures

#### On Card

- **tokenId** - The public key of the issuer/authority
- **balance** - A integrer in the assets smallest denomination

#### Transfer Packet

- **tokenId** - The public key of the issuer/authority
- **value** - The amount that has been subtracted from the sender's token balance, and when consumed, the amount to be added to the recipient's token balance.

## Example

The US Gov wants to issue phonon tokens representing a million dollars. To enable micro transactions they decide the denomination of a token should be 1/100 of a cent. They generate a key pair and make the following information public:

- tokenId: Their public key
- token Name: pUSD
- decimals: 4

The US Gov wants to pay 100 pUSD Senor for some work he has done. They call `sendPhononTokens` with a value of 1000000 after pairing with Senor's phonon card. The US Gov's pUSD balance is reduced by 100 and a phonon transfer packet is created with a value of 100 pUSD. Senor's card receives payment and his app searches a public registry for the token details. It finds the token name and decimal information is able to correctly display his new balance.
