# Branded Phonons: *immutably marked by their creator*
How do you tell apart a Tesla from a Honda Civic? Do you question the provenance of a $20 bill? How do ranchers find their cows after getting mixed with their neighbors', following the destruction of barbed wire fences in a winter storm?

Brands (symbols, trademarks, distinctive images, ornamental patterns burned into skin) are used to build trust, protect identity, form collections and organize like things.

Like a cow branded at birth, *Branded Phonons* are marked by their creators upon creation with a stamp that cannot be faked or changed. It gives anyone who receives that phonon the trust and guarantee it was created by that creator.

## The Problem
There is no way to prove who created a phonon.  For phonons that are linked to blockchain based assets this not a problem (in fact it's a feature).  But there are use cases where proving provenance of a phonon can add value.  
  - a music venue may want to use phonons to grant recipients of those phonons access to an upcoming show
  - the United States Federal Reserve could guarantee the value of phonons they create

## Solution

Introduce a new class of phonons called **Branded Phonons**.
  - They are attested by the issuer/authority at creation. Accomplished by having the creator pass a private key at creation, whose public key is deduced by the card and tagged to the phonon.
  - Their value is instantiated at creation by the issuer, and thereafter not directly editable.

## Implementation Details

[link to repo with working code](https://github.com/ahinchliff/phonon-client/tree/flexible-phonons-senor)

Process Flow Steps
  0. Launch `repl`: `go run main/phonon.go repl`
  1. Call `mock`
  2. Call `activate` and select card
  3. Call `unlock` as input password `111111`
  4. Call `createSpecial`
  5. Select `Branded`
  6. Input `privateKey` hex and `value`
  7. Phonon is created

Summary of Technical Changes:
  1. Added `Branded` as new `CurrencyType` -> 5
  2. Added `createSpecial` command to `repl`
  3. Added `createBrandedPhonon` to `repl`
    - requests `privatekey` as parameter from issuer
    - requests `value` as parameter from issuer
  4. Added `CreatePhononWithSetDescriptor` to session and card
    - creates `phonon`
    - deduces `publickey` from `privatekey` and tags phonon: `TagCreatorPublicKeyTLV`
    - sets `value` of phonon
  5. Updated `SetDescriptor` on card
    - disables use for `CurrencyType == 5`
