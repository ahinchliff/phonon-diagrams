# Posted Phonons: *One Directional Phonon Transfers*
## High Level Process Flows with Design Questions


### 1) The Store
Picture a coffee shop: one vendor who serves coffee to many people

#### Stage 1: Establish Service
1. Store's Card requests to be registered with Store
  - How is this request made?
2. Store registers Store's Card
  - What information does the Store need to know/record about the Store's Card?
  - Does the store need to verify ownership of the Store's Card? If so, how?

#### Stage 2: Process Transactions
1. Person requests order from the Store
  - How is this request made?
2. Store sends the Person a request for payment
  - How is this request sent?
  - Is a nonce, an expiry date for that nonce, and the public key of the registered
    card the only data the Person needs in that request for payment?
  - Will it always issue a nonce that is incremented by one?
  - How long should the expiry date be? Should it depend and be variable according
    to the context of the vendor?
3. Person creates a phonon transfer packet
  - What prevents the Person from creating a phonon after the expiry date has expired?
  - How do we know this packet was created by a valid phonon card?
  - How do we know this packet has not been tampered after having been created?
4. Person posts the payment to the Store
  - Does the store care if the payment comes from a person who is NOT the Person
    issued the request for payment in Step 2?
5. Store verifies the legitimacy of the posted phonon
  - How does it verify the phonon originated from a valid card? not tampered?
  - How does it verify the value of the phonon?
  - How does it verify that the phonon can be consumed by the registered card?

#### Stage 3: Collect Phonons
5. Store's card connects to Store
  - How is this connection made?
6. Store's card requests to withdraw posted phonon(s)
  - What if there are requests for receipt still outstanding?
  - Can posted phonons exist in the service if they were not requested from the service?
7. Store's card withdraws phonon(s)
  - What prevents the card from withdrawing the same phonon more than once?

### 2) The Gallery
Picture an art gallery: multiple vendors who sell art to multiple people
- What would need to be different in the above process flow if the Service were
to manage multiple vendors' cards?


### 3) The Nomad
Picture a traveling artist: different vendors who sell their artwork
- What happens when someone wants to change service providers?
- What are the risks of changing providers?
- Can someone subscribe to multiple providers at once?
