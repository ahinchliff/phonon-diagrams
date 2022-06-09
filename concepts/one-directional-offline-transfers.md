# One Directional Offline Phonon Transfers

## Goal
Make it possible for a person to send a phonon to someone whose card is not online.  Ideally, the transaction is simply POSTed over http.  Creates tons of flexibility for the receiving party.  For example:

1. Online stores - don't need to have cards attached to servers to receive payment
2. Entertainers, like live streamers, could receive tips without having their card processing each transaction in that moment, but accumulating over the show
3. POS - Staff don't need cards plugged into POS machine
4. Simple sending.  Every dev knows how http works, so who knows what cool stuff people can come up with.

## Constraints
We all agree the following must be true for any design to work:

1. Double spending must not be allowed. Receiving card must never be able to consume and spend the same phonon twice
2. Sent phonons must only be consumed by the intended receiving party
3. Avoid losing funds by limiting or eliminating use cases where sent phonons cannot be consumed by receiving party

## Design Assumptions
Assume the following about posted one directional phonons:

1. Must be submitted with a `receiptID` expressed as an integer


Add the following features to the `card APDU`:

1. Allot space on the card (not within each phonon but at the card level) to store an `integer`, and name it something like `lastReceipt` or `witnessedReceipt` or `receiptWatcher` or `receiptTracker`.  Initialize this attribute to `0`.

2. Add a new APDU method and call it `ReceiveOfflinePhonons`:
- It can consume a one directional offline phonon that has been posted by a sender
- It expects a `receiptID` in the offline phonon
- It sets the `receiptWatcher` to the value of the `receiptID` after consuming the offline phonon
- **BUT** it will **NOT** allow an offline phonon to be processed if its `receiptID` is <= the `receiptWatcher`.  

## Talking Through Gotchas
- Write about ways to mitigate lost funds, because of stale receipt IDs
- Write about why collision between vendors who issue identical receipts doesn't matter

## Example Process Flows
