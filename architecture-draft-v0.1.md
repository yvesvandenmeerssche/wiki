# WT architecture proposal

Here is summary of brainstorming done with me (Robin Gottfried), Hynek Urban, Jiří Chadima and Martin Bílek.

## Tools

### Blockchain

Blockchain is the only technology which is able to keep immutable shared history in IT world. This ability is payed by high cost of any operation and data on a blockchain as each node in the network needs to repeat any operation requested on respective blockchain and keep all data forever.

Unique properties:
- keeps immutable history

Not unique properties - alternatives:
- censorship resistancy - any distributed system, onion routing
- zero down time - any distributed / scalable system
- efficient data retrieval - any scalable database

### Distributed database (D-DB)

By distributed database I mean a network system where tha data are spread across  nodes in a manner that requester can contact any node to successfully receive the content associated with a key. The database keeps running even when most of the network nodes disappear. Examples of such database is IPFS or Swarm.

I do not distinguish between database and filesystem because for our needs filesystem is just specific use of a key-value storage.

*Unique properties:*
- zero down time
- censorship resistancy

*Not unique properties* - alternatives:
- zero down time - any  scalable system
- efficient data retrieval - any scalable system

Blockchain is distributed database but with added immutable history and the associated costs per operation and data stored.

## The operations on WT

To make thinking about the architecture easier I splitted WT into simplified operations:

1) Get list of existing hotels (index)
2) Get details (description, pictures, ...)
3) List inventories (rooms, services, ...)
4) Search data by attributes
5) Get current availability
6) Book
7) Pay
8) Trust (reputational system)


## The architecture proposal


        --------------      --------------
        |  Hotel A   |      |  WT API    |<----- Pension
        | .......... |      | .......... |<----- Hotel b
        | : WT lib : |      | : WT lib : |       ...
        | .......... |      | .......... |
        --------------      --------------
            |   |                |   |
            |   ---------------------|----
            |                    |   |   |
            |      ---------------   |   |
            V      V                 |   |
        -----------------          V   V
        | Blockchain    |      *   *   *   *   *   *   *
        |               |      *                       *                  
        # hotel A | url |         D-DB (Swarm/IPFS)                  
        # hotel B | url |      *                       *
        -----------------      *   *   *   *   *   *   *
                 |               |             
                 V               V

             |       WT lib          | 
             |       cache / db      |

             -------  -------  -------
             | OTA |  | OTA |  | OTA |
             -------  -------  -------
                        ^
                        |
                        v
                  -------------
                    U S E R S
                  -------------


### Scenario

- Hotel signs it's data in computer readable standardized format and exports them to D-DB. 
- Hotel writes data locator to blockchain together with it's public key.
- OTAs (or any consumers) read list of hotels from blockchain, retrieve the corresponding files from D-DB, verify the signatures and build index to fulfill search requests of users.

### Properties of the system

- Any hotel can join (only knowledge of WT open data spec or WT lib is required).
- Updating inventory, prices and availability is not expensive.
- Anybody can read hotel and inventory data and verify it is written by a particular hotel.
- Nobody can take ownership of the whole system.

### Properties with adverse effect

- Anybody can copy hotel's data and sign it to pretend he/she is owner of the hotel.
 

## Other points

### Availability data

Keep track of changes of availability by polling the data files of all hotels for chages is possible but could spent unnecessary high amount of resources on client as well as D-DB site. It is desirable to implement event based tracking of changes. Whisper or other messaging service seems like a good candidate.

### Booking / payment

The scheme above does not solve booking / payment process. This process seems to fit perfectly to blockchain domain. The base scheme could be something like this:

1) Client selects a pack of services (room with breakfast from 1st to 5th may, with view to a sea and mahagony doors), adds price and any contract details to one document.
2) Client and hotel signs the document confirming, that they want to make this contract, stores the full content for later evidence and makes hash.
4) Client sends Lífs (Ethers, Bitcoins) to hotel with the hash in data.

This way, client can prove the exact agreement he/she payed for. The same for hotel.
Everybody else knows only hash and price.

I expect that there is already a service / framework for secure digital payment for digital contract OTAs can use.

### Reputation system

For the whole system to work, a reputation system should exist which allow space of trust. I believe that for the first iteration the reputation system is out of scope of WT work, but WT could ease implementation by providing examples and documentation how to implement existing systems created by other parties. In theory there could / should be many concurent such systems in future.


