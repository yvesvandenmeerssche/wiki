# Decentralized storage for Winding Tree

As we have described in our [Architecture](https://github.com/windingtree/wiki/blob/master/WT%20Hotel%20Architecture.md#distributed-database-d-db)
document, we need some kind of distributed storage for Winding Tree to work. We would really love to have all data on-chain, but
with the current state of technology, it's just not feasible (it's slow and expensive).

Our initial idea was to store only a link to data on-chain. Everything would then be
accessible through the [library](https://github.com/windingtree/wt-js-libs) or [API](https://github.com/windingtree/wt-nodejs-api)
or maybe some caching layer. By storing data off-chain, we would remedy the lack of speed and higher cost of data stored on-chain.
Obviously, we're [not the first](https://github.com/TiesNetwork/ties-docs/wiki/Where-do-decentralized-applications-store-their-data%3F)
to do such thinking and [this article](https://decentralized.blog/picking-a-decentralized-storage-system.html)
was a good starting point for our evaluation. We haven't used any strict methodology, it was more of a look & feel approach
without any special configuration.

The motivation for storing data off-chain is twofold:

1. *Size* - Larger chunks of data (yes, a kilobyte is a large chunk of data), such as hotel description or images
[might](https://hackernoon.com/ether-purchase-power-df40a38c5a2f) [be](https://ethereum.stackexchange.com/a/896)
[costly](https://www.reddit.com/r/ethereum/comments/6rj8ks/current_storage_costs_and_limits/dl5scwq/) when stored
directly in Ethereum smart contracts.
1. *Mutability* - Every change of data such as a booking of a room (which changes availability data and might happen very often),
would result in an additional transaction of updating the availability data. Even when we only link to the data
itself, we cannot update the link every time data changes. That's why we need either mutable data under the 
same identifier or some kind of naming service standing between blockchain and distributed storage.

I think it's safe to say that every considered storage provides cheaper data storage than blockchain. The speed
factor and maturity level of every technology may vary, however. We also need to access the network programmatically
to be able to handle the expected data load.

## Contenders

### Swarm

[Swarm](https://swarm-guide.readthedocs.io) is the obvious first choice as it will be an integral part of EVM some day.

- *Speed* - Upload speeds through a [public proxy](http://swarm-gateways.net) seem to be very good, download through a proxy is reasonably quick.
- *Full-network consistency* - This seems to take some time anywhere between a minute and 15 minutes. This is unfortunate but
can be mitigated by heavy caching and/or using dedicated nodes with some additional logic.
- *Data loss* - It might eventually happen after implementing the [incentivization system](https://www.youtube.com/watch?v=9Cgyhsjsfbg&feature=youtu.be&list=PLaM7G4Llrb7xNkX0gugDyUm6Z_RrCvWPk),
but highly unlikely.
- *Name service* - That's the culprit. From what I've found, the general idea is to use [ENS](https://ens.domains/). But that's
implemented over Ethereum smart contracts. So every change of data means an on-chain transaction and we're back to square one.

### IPFS

[IPFS](https://ipfs.io/) seems like another serious contender with its history dating back to 2014.

- *Speed* - It seems regularly slow. Publishing a document is fast (as it happens locally). But accessing the data through public gateway
may take eventually somewhere between 20-70 seconds.
- *Full-network consistency* - That's hard to achieve as right now [no-one is incentivized](https://github.com/ipfs/faq/issues/47) to keep
your content on their nodes.
- *Data loss* - It might very well happen once the original node is offline. This can also be mitigated by using dedicated nodes.
- *Name service* - [IPNS](https://medium.com/@yaniv_g/hosting-websites-on-ipfs-with-ipns-b94659c42b52) has a poor documentation. And is really slow
and sometimes stalls. From my understanding, right now it works on the same [DHT](https://github.com/ipfs/go-ipfs/issues/1396) protocol
as regular files, so its speed is consistent with the rest of the network.

### Zeronet

[Zeronet](https://zeronet.io/) uses BitTorrent protocol for file distribution and there's a
[nice presentation](https://docs.google.com/presentation/d/1_2qK1IuOKJ51pgBvllZ9Yu7Au2l551t3XBgyTSvilew/pub?start=false&loop=false&delayms=3000&slide=id.g9a1cce9ee_0_4)
on how it works.

- *Speed* - It's quite nice as everything happens locally. Accessing the data from a different place takes some time, but is reasonably quick
after the initial search. The speed increases with the number of nodes/users hosting your files.
- *Full-network consistency* - Once you publish the data locally (and notify the BitTorrent tracker), everyone with the address can see it.
- *Data loss* - That happens when all nodes with the copy of your data get offline. You can prevent it by using dedicated nodes.
- *Name service* - No need! The address serves as a directory, so you can change its contents in any way you want and the address will stay the same.

### BigchainDB

[BigchainDB](https://www.bigchaindb.com/) is finishing its 2.0 release right now and seems very promising
with its local queryable MongoDB instance. However, there does not seem to be a public network for everybody.
It seems to me like it is oriented more towards private networks. But the API looks nice and pretty generic.

They are also using [Tendermint](https://tendermint.readthedocs.io/en/master/) as [PoS](https://en.wikipedia.org/wiki/Proof-of-stake)
algorithm and had even run [Etheruem on it](http://ethermint.readthedocs.io/en/master/).

### Ties DB

[TiesDB](https://tiesdb.com/) is in early development stages, but it looks like
[they might be up to something](https://github.com/TiesNetwork/ties-docs/wiki/Where-do-decentralized-applications-store-their-data%3F).

### Freenet

[Freenet](https://freenetproject.org/index.html) is a very old project aiming at private decentralized
groups with an elevated trust. It can be used in a trustless setup, but the
[API](https://bluishcoder.co.nz//2017/03/28/introduction-to-the-freenet-api.html) does not really look
very friendly.

## Conclusion

It looks like there is no silver bullet for decentralized storage, every solution has its pros and cons.
But for sure, you need to run a node or a daemon (or maybe multiple of those) to ensure the data is available.
The lack of reliable naming service is also a big disappointment as it kind of leads us to a dead end.

But does it really? When one day, Ethereum is mature enough to hold hundreds of transactions per second, we might
not need a naming service at all. Once IPNS is mature enough, we might be using that. Still, we need to find
some solution for now and not for some hopefully not-so-distant future. Or maybe we could use [Namecoin](https://namecoin.org/).

That's why we have decided to implement a schema-based data links in our smart contracts and provide multiple
drivers based on current demand. How will it work?

We store a [url field](https://github.com/windingtree/wt-contracts/blob/proposal/next/contracts/hotel/Hotel.sol#L22)
for every hotel Winding Tree manages. This url shall look like `ipfs://QmRZEpRMS62xt1Gm6B7PLoXreuxSwX1kDR384PJ6tbCRLA`
or `ipns://QmRZEpRMS62xt1Gm6B7PLoXreuxSwX1kDR384PJ6tbCRLA` or `bzz://446421099bd54d1532b00db29027b5e5e37c9b5d6a9f55809a9ef3880ac43ed5`
or even `https://raw.githubusercontent.com/windingtree/wt-contracts/proposal/next/contracts/hotel/Hotel.sol`. Then
it will be a responsibility of a user of such contract to get this data and verify its integrity and ownership.

Our [library](https://github.com/windingtree/wt-js-libs) and [API](https://github.com/windingtree/wt-nodejs-api)
will do exactly that. We are also considering having multiple data links to different kinds of data. Maybe IPFS
is a better storage for availability data while Zeronet deals better with hotel images. We are still figuring the details
and implementing the first version of this right now.

Regardless of the chosen data storage, it is probably a good idea to introduce a caching server at some point.
Because once we allow private servers to host data, even if it is in the best interest of the provider
to ensure the server's uptime, no one can actually enforce it and downtimes will eventually happen. For some
technologies, caching can also significantly decrease the access times.
