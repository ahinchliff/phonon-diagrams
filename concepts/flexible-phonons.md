# Flexible Phonons: *attested phonons with a flexible value*
The purpose of *Flexible Phonons* is to make it possible for two parties to settle transactions without the internet, using assets both parties trust, and not relying on a 3rd party to issue change.

## The Problem

Neither native nor backed phonons can be combined or divided into new denominations. This makes it impossible for two cards to complete an offline transaction if they do not have exact change. The concept of a "change maker" service has been discussed but this requires one of the parties to be online and to interact with a third party.

## Solution

Introduce a new class of phonons called **Flexible Phonons**.
  - They are attested by the issuer/authority
  - Their value is not fixed (like a backed phonon whose asset immutably awaits on chain) but can increment and decrement
  - Their value is instantiated at creation by the issuer, but thereafter not directly editable.  After creation, the phonon's value can only be updated by the card itself when making payments.

The above is made possible by:
  - Making a few changes to the `create` process (see creation)
  - Disabling the `SetDescriptor` method after instantiation for *Flexible Phonons* (makes it impossible for anyone to change the `CurrencyValue`, `CurrencyType` and `Tags` after creation)
  - Create two new methods on the applet called `SendValue` and `ReceiveValue` which manage the changes to the `CurrencyValue` (see sending and consumption)

### Creation

When an issuer/authority creates a *Flexible Phonon* they must provide a denomination (i.e. penny), declare its value (i.e. 10 cents) and provide proof that they created it.

This is made possible by:
  - Requiring a `CurrencyValue` be passed as a parameter to the `Create` method
  - Requiring a private key (known only to the issuer/authority) be passed as a parameter to the `Create` method. The card would deduce the private key's public key, then store it in an `id` tag in the phonon's TLV meta data.
    - The public key would be posted publicly by the issuer/authority, so could be used by phonon clients to prove a *Flexible Phonon's* authenticity
    - The public key would exist in the TLV of other *Flexible Phonon's* (on many other cards) that have been created by the issuer with the same private key. I'm imagining cards would receive these phonons by pairing with the authority through various means.
  - Because the `id` tag is a unique identifier, and would be shared by all *Flexible Phonons* that were seeded with the same private key, the `id` can function as the denomination too. The issuer/authority posts publicly a mapping of their their public keys to names (i.e. xxxxx -> penny).
  - The `CurrencyType` would be `FlexiblePhonon`

After creation, the `SetDescriptor` can never be called again for *Flexible Phonons*. The `CurrencyType` and `Tags` would be immutable. And the `CurrencyValue` would only be editable by the card when making payments (see next section).

### Sending

Users must be able to send *Flexible Phonons* like any other phonons. And they must be able to send only a portion of their value (i.e. I have 10 cents but only need to send 3 cents).

This is made possible by:
  - Creating a new method called `SendValue` which takes a value as a parameter
    - This method is be used for *Flexible Phonons* only
    - This method calls the `SetDescriptor` function and decrements the *Flexible Phonon* by the passed value
    - This method creates a transfer packet to send to receiving card. Packet includes:
      - public key listed in `id` tag
      - value (for receiving card to increment)
  - Update the `Send` method to take a `value` parameter for *Flexible Phonons*.
    - If no parameter value is provided, the *Flexible Phonon* is sent between cards like any other phonon
    - If a parameter value is included, the `SendValue` method is called, and the *Flexible Phonon* remains in the senders card

### Consumption

Users must be able to receive *Flexible Phonons* like any other phonons. And they must be able to receive portions of sent value (i.e. they have 10 cents but only need to send me 3 cents).

This is made possible by:
  - Creating a new method called `ReceiveValue` which takes the packet sent by the sending card as a parameter (see above)
    - This method parses the packet, gets the public address listed in the `id` tag, then filters its own phonon list for a *Flexible Phonon* with the same public key in its `id` field.
      - If it returns 0 phonons, it aborts the transaction.
      - If it returns a phonon, it calls the `SetDescriptor` function and increments the returned *Flexible Phonon* by the value in the packet
    - This method can only be called while actively paired with the sending valid phonon card

Like adding an ERC20 contract to ones meta mask, I'm picturing cards interested in handling a specific *Flexible Phonon* denomination would proactively have one added to their card (even if its value is initialized as 0 by the issuer/authority).  This way they can fully trust the value being sent from a card in an offline situation (their client can't audit the public key) because they'd have a matching key already.

## Example

The US Gov wants to issue digital cash, and they want to minimize the possibility that two parties (who have neither access to internet nor a change maker) cannot complete a transaction because they do not have exact change.  The US Gov decides to issue the penny since it's the lowest common denominator for most transactions.  Though it seems silly to deal in pennies, paperbacks solve physical problems (who wants to lug around 10,000 pennies) which are non issues in a digital world.

They generate a key pair and make the following information public:
- id: Their public key
- name: USDPenny

[to be continued I'm very tired]

## Notes
- Other names I've come up with include *counting phonons*, or *purse phonons*
- `CurrencyValue` I believe is currently an int64 which would be limiting if penny were selected as a denomination I believe.  One option it to leverage a TLV to track value for *Flexible Phonons* to avoid increasing the size of the CurrencyValue struct which would have memory ramifications across all phonon types
