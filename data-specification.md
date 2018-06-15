# On data shapes and formats

After a few weeks of tedious research of [many](https://expediaconnectivity.com/apis/product-management/product-api/reference.html)
[existing](https://connect.booking.com/user_guide/site/en-US/user_guide.html?lang=en)
[APIs](https://developer.sabre.com/sabre_hospitality/apis/soap_apis/hotel/channel_connect/ari)
[and](https://www.htng.org/page/technical_specs?)
[standards](https://schemas.liquid-technologies.com/OpenTravel/2008B/),
we are ready to show to the world how the hotel data might look like once
the [Winding Tree](https://windingtree.com/) platform is ready. This is a first draft
that we are putting out for comments and validation by our friends, partners, and community.

So if something is not clear to you, or you think something can be
improved, please, leave us a comment. Your feedback is invaluable and we
are eager to hear all of your ideas. Some pieces are definitely missing
and we already have a [decent TODO list](https://github.com/windingtree/wiki/issues/28)
for this area.

If you don't like reading, jump directly to [the docs](https://windingtree.github.io/docs/swagger-ui/)
where our data format is published in an [OAS 3.0](https://github.com/OAI/OpenAPI-Specification) format.
If you like reading, please open [the docs](https://windingtree.github.io/docs/swagger-ui/)
anyway, so you can directly see what I am talking about.

We have designed the format in a way that plays well with our
[architecture](https://blog.windingtree.com/hotel-distribution-architecture-49ead97f7cff)
for data storage where we plan to allow various storages - [IPFS](https://ipfs.io/),
[Swarm](http://swarm-guide.readthedocs.io/en/latest/introduction.html)
or even plain HTTPS, whatever works the best for each type of data. It
can be divided into four groups:

- *Hotel description* - Where is the hotel, who is the owner, etc.
- *Inventory description* - Which rooms are in the hotel
- *Pricing* - Which prices apply to which rooms
- *Availability* - Which rooms are available at certain points in time

The groups differ by how big the data is and how often it changes.

The entry point for all of hotel-related data is stored on Ethereum blockchain
where each hotel has [its own contract](https://github.com/windingtree/wt-contracts/blob/proposal/next/contracts/hotel/Hotel.sol)
registered via [Winding Tree index](https://github.com/windingtree/wt-contracts/blob/proposal/next/contracts/WTIndex.sol#L63).
In that hotel contract, there is a single URI (`dataUri`) pointing to a document stored in any
supported off-chain storage. And on that URI, there lies the **Hotel data index**.

That is meant as an index of all the four aforementioned data groups. We currently plan
to have three links there (naming may change):

- `descriptionUri` - Hotel + Inventory description
- `pricingUri` - Pricing
- `availabilityUri` - Availability

## What is definitely not yet final?

We still have a large room for improvements and some areas are not yet fully described.
Among these are:

- *Internationalization* - We will eventually support it, stay tuned!
- *Restrictions* - This is a very complex topic and we want to make it right.
We will revisit it in the future, so far we have a very small subset.
- *Amenities* - We are struggling with proper definition since it may vary a lot.

If you'd like to help us with designing these, leave us a comment and we'll get in touch!

## Descriptive data - hotel and inventory

The easiest part is the Hotel and Inventory description. We expect this
data to generally not change at all since it describes a physical reality.
We are right now working only with Room types as inventory. More types of
inventory may come in the future. We are also thinking about all the stuff
happening directly on room level.

- We will eventually add more contacts for a hotel (such as a dedicated person
  for bookings, complaints etc.).
- We allow generic contact options (such as Whatsapp or WeChat contact).
- We do not currently plan to support direct upload of images. We expect only
URIs - they can point for example to Swarm.
- The amenities enumeration (on both hotel and room level) is definitely not
final, more of an example. It will change in the future, feel free to suggest
new possibilities. It is also possible that we will extend the amenities
format in some way.
- The room properties will change. Feel free to suggest what should we include.

Do you want details? Have a look at **Hotel description** and **RoomType**.

## Pricing

The pricing data is closely following the idea of rate plans which should be
widely known.

We would really like to get a feedback from all the people that have experience
with setting up pricing strategies. Can you model your pricing plan within our
data specification? If not, why? Leave us a comment, please. We eventually want
to support as many pricing models as possible.

- We allow base and additional pricing, i. e. some rate applies to a number
of people up to a certain limit and a different one to all people over that
limit.
- For both base and additional pricing we allow different rates based on:
    - The maximum age of a traveler
    - The minimal length of stay
- The restrictions are strictly related to applicability of given rate plan.
So if a length of stay is restricted, the rate plan cannot be applied if the
desired stay is out of bounds. It has no relation to the availability of a room
type.
- We don't support discounts and derived plans as they can be modeled with
the current data shape.
- We don't yet support Occupancy based pricing - i. e. different prices for
a different number of travelers.
- We don't yet support extra pricing for extra services such as breakfast.
- There are probably more restrictions that we can use. Feel free to suggest
them to us!

Do you want details? Have a look at **RatePlan**.

## Availability

Availability is the tricky one since it changes most often and potentially is
the largest dataset - because you have to keep a state of each room type for every day.
That's why it might be really expensive (timewise and resourcewise) to generate
the whole dataset for a hotel all at once.

- We allow update of the whole dataset (**AvailabilitySnapshot**).
- We allow to set some restrictions for each day. These are strictly related
to the availability of given room on given day. There are probably more restrictions
that we can use. Feel free to suggest them to us!
- We are indexing the data by room type. Unfortunately we cannot index them by day
at the same time. That's the responsibility of anyone who will use the data.
- We also allow periodic updates of the snapshot and using **AvailabilityUpdateRecord**s
to tell the world about the changes happening in between snapshots.
    - Every user of the data is then responsible for computing the actual state
    of availability for given hotel at given time.
    - We allow either absolute or incremental updates of the available quantity.
    - Every update message has to contain a serial number to allow replayability
    of all updates in correct order.
    - Updates should be reset once a new snapshot is created.
- We will probably have to split the data into multiple documents. Natural candidates
for separation are:
    - List of updates
    - Availability of every room type

Do you want details? Have a look at **Availability**.

## Concluding remarks

This is a start. We're not saying that in the end, the data will look like this,
but we are pretty confident it will be very similar. We are also not promising
that we will have all the parts implemented in our closest release, but this is a
path we are taking. You can follow our progress on
[our Trello board](https://trello.com/b/ZREEkuOX/winding-tree-work-in-progress).

As this is a heavy work in progress, there might be some inconsistencies (for example
in naming) as well. We are also exploring more features all the time, you can have
a look on [our TODO list](https://github.com/windingtree/wiki/issues/28).

We are also very aware that we will have to create some service that will notify
anyone that some data has changed. It is not feasible to poll many documents in many
storages for their change by many clients.

If you have any comments, questions or would like to help, leave a comment,
post to our brand new [Google Group](https://groups.google.com/forum/#!forum/windingtree)
or contact us via many other channels listed on [our website](https://windingtree.com/).
We value your feedback!
