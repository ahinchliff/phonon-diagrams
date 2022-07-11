# Flexible Phonons: _attested phonons with a flexible value_

The purpose of _Flexible Phonons_ is to make it possible for two parties to settle transactions without the internet, using assets both parties trust, and not relying on a 3rd party to issue change.

## The Problem

Neither native nor backed phonons can be combined or divided into new denominations. This makes it impossible for two cards to complete an offline transaction if they do not have exact change. The concept of a "change maker" service has been discussed but this requires one of the parties to be online and to interact with a third party.

## Solution

Introduce a new class of phonons called **Flexible Phonons**.

- They are attested by the issuer/authority, just like [BrandedPhonons](https://github.com/ahinchliff/phonon-diagrams/blob/main/concepts/branded-phonon.md)
- Their value is _not fixed_ (like a backed phonon whose asset immutably awaits on chain) but can **increment** and **decrement**
- After being instantiated at creation by the issuer, the `value` cannot be edited by a user. Instead, the phonon's value can only be updated by the card itself when making payments.

## Implementation Details

[link to repo with working code](https://github.com/ahinchliff/phonon-client/tree/flexible-phonons-senor)

Process Flow Steps

0. Launch `repl`: `go run main/phonon.go repl`
1. Call `mock`
2. Call `activate` and select card # 1
3. Call `unlock` as input password `111111`
4. Call `connectLocal`
5. Call `mock`
6. Call `activate` and select card # 2
7. Call `unlock` as input password `111111`
8. Call `connectLocal`
9. Call `createSpecial`
10. Select `Flexible`
11. Input `privateKey` hex and `value`
12. Call `listPhonons` and get index of your flexible phonon
13. Call `pairCounterparty` with card # 2
14. Call `sendFlex` with index from step 12, and value to send
15. Call `listPhonons` and verify value has decremented
16. Call `activate` and select card # 2
17. Call `listPhonons` and verify card received phonon with value

Summary of Technical Changes:

0. Leverages functionality from _BrandedPhonons_
1. Added `Flexible` as new `CurrencyType` -> 4
2. Added `createFlexiblePhonon` to `repl`
   - requests `privatekey` as parameter from issuer
   - requests `value` as parameter from issuer
3. Added `sendFlex` to `repl`
   - requests `keyIndex` as parameter from issuer
   - requests `value` as parameter from issuer
4. Added `SendFlex` to `session`
   - expects `keyIndex` for flexible phonon and `value` as parameters
   - calls `CreatePhononCopyDescriptorNoValue`
   - calls `UpdateFlexPhonons`
5. Added `CreatePhononCopyDescriptorNoValue` to card
   - expects a flexible `phonon` as a parameter
   - creates a new phonon
   - sets tags based on phonon passed to method
   - sets value of phonon to 0
   - currently only allowed for CurrencyCode == 4
6. Added `UpdateFlexPhonons` to card
   - expects two flexible phonons, and value to balance as parameters
   - increments the value of one flexible phonon
   - decrements the value by the same amount of the other phonon
   - checks various things (equivalent tags, sufficient value, currency code)
7. Added `ConsolidateFlex` to session
   - expects two flexible phonons
   - calls `UpdateFlexPhonons` to empty the value from one phonon, and adds it to the other
   - should call `destroyPhonon` but still fixing bugs
8. Updated `SetDescriptor` on card
   - disables use for `CurrencyType == 4`
