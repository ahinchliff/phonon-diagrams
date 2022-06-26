# Posted Phonons: _One Directional Phonon Transfers_

## Terms

- Posted Phonon - A phonon transfer packet that can be created for a particular phonon card without the sender's and recipient's cards being actively paired.
- Phonon Inbox - A service that stores posted phonons for its users and optionally provides senders the details required to create a posted phonon for an intended recipient. Similar to email, a phonon inbox can be self hosted or accessed through a service provider.

## High Level Process Flows with Design Questions

### 1) Single Coffee Shop

- A shop with a single point of sale system (POS)
- The shop owner doesn't want their staff to have physical access to the shop's phonon card
- The shop owner wants to consume their posted phonons at the end of each day.

#### Stage 1: Setting Up Point of Sale

1. The shop's owner downloads an POS app onto their tablet that has phonon inbox functionality.
2. The shop's owner enters their phonon card's public key and nonce into the POS app using the apps UI or by connecting their phonon card and allowing the app to sync these details automatically.

#### Stage 2: Receiving Phonons As Payment

1. A customer orders a coffee and wishes to the pay using the phonon network.
2. The POS retreives the next nonce from the tablet's storage. The tablet broadcasts an NFC message containing a payment request consisting of the order value, the shop owner's phonon's cards public key and the nonce.
3. The customer taps their device to receive the payment request and accept the payment request. Once accepted the shop owner's card's public key and nonce are used to create a posted phonon.
4. The customer's device broadcasts the posted phonon to shop's POS over NFC.
5. The shop's POS receives the posted phonon and confirms the validity of the phonon and that is contains the expected value. Optionally the POS backups the posted phonon to an external service.
6. The order is completed. The POS increments the nonce in its internal storage and the customer receives their coffee.

#### Stage 3: Store Owner Adds Phonons To Their Card

Note: If the shop's POS was backing up posted phonons to an external service the store's owner could complete the following processes remotely

1. The store's owner connects their phonon card to the POS or backup service and requests to consume the posted phonon with the lowest nonce.
2. The POS or backup service returns the posted phonon and the owner's phonon card consumes it. In the process the card's nonce is increased to match the consumed posted phonons nonce.
3. This process is repeated until all phonons are consumed.

### 2) Chain of Coffee Shops

- Multiple shops with multiple POSs in each
- The owner only wants to use a single phonon card for all their stores
- The owner doesn't want their staff to have physical access to the chain's phonon card
- The owner wants to be able to access a phonon six hours after receiving it.

#### Stage 1: Setting Up Point of Sale

1. The shop's signs up to a hosted phonon inbox service and provides their phonon's cards public key and nonce.
1. The shop's owner downloads an POS app onto their tablet that has support for an external phonon inbox service. This could involve supplying an api key, login details or endpoints.

#### Stage 2: Receiving Phonons As Payment

1. A customer orders a coffee and wishes to the pay using the phonon network.
2. The POS requests the details required to create a posted phonon from the shop's phonon inbox service. The shop's phonon card's public key, a nonce and a TTL equal to the current time plus six hours is returned.
3. The POS display the received data plus the order value as a QR code. The customer scans this QR code and approves the payment request. Once accepted the shop owner's card's public key and nonce are used to create a posted phonon.
4. The posted phonon is sent over HTTP to the phonon inbox service where its validity and value is confirmed.
5. The shop's POS receives a socket message from the phonon inbox service confirming payment has been received. The order is completed.

#### Stage 3: Store Owner Adds Phonons To Their Card

1. The store's owner connects their phonon card to their phonon inbox service and requests to consume the posted phonon with the lowest nonce.
2. The phonon inbox service returns the phonon with the lowest nonce if there is not a pending nonce with a live TTL. If a phonon is returned the owner's phonon card consumes it. In the process the card's nonce is increased to match the consumed posted phonons nonce.
3. This process is repeated until all available phonons are consumed.

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

      ---- Questions ----
    - How does vendor verify phonon is from valid card?

    - How does vendor verify the value of the phonon?
      @3: By leveraging phonon public key, currency type and currency value to
          check on chain assets
      @2: By verifying signature (must include phonon public key)

    - How does vendor verify that the phonon can be consumed by the receiving card?
      @2: By verifying signature (must include receiving card's public key,
          and the cipher text)

    - What prevents the card from withdrawing the same phonon more than once?
      - @1: The cypher text requires the receiving card's private key to
            decrypt, so it cannot be accessed outside of the applet
      - @1: Encrypting the nonce with the private key binds them together.
            While encrypted the nonce cannot be adjusted, and during decryption
            the phonon is consumed and the card's nonce is incremented

      ---- Data Packet Process ----
      1. Sender's card encrypts:
          - Phonon Private Key             
          - Receipt nonce from vendor
         using the public key of the receiving card,
         then returns the cipher text

      2. Sender's card signs across:
          - Cipher text returned from step 1
          - Phonon Public Key
          - Receiving Card's Public Key

      3. Sender compiles entire packet:
          - Cipher text from Step 1
          - Signature from Step 2
          - Phonon Public Key
          - Receiving Card's Public Key
          - Sending Card's Public Key
          - Currency Type of phonon
          - Currency Value of phonon

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
