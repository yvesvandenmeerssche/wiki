# Publicly available WT deployments

We currently have two publicly accessible WT environments -
*playground* and *demo*. Each of them consists of:

- a set of WT smart contracts, deployed to the Ropsten Ethereum network.
- an instance of wt-write-api
- an instance of wt-read-api
- an instance of hotel explorer web application

Just bear in mind that publicly available instances of the write
API have their database purged once a day for security reasons.
Hotels uploaded through them still remain on the blockchain,
though. In practice, this means that you might sometimes need to
re-create an account in order to continue working with
previously created hotels.

In addition to this, there is also an instance of wt-notification-api in
both environments whose database will be purged irregularly. So your
subscriptions might need to be re-created from time to time.

**Note: None of these environments is really stable. It can change at any time,
and your data might be rendered inaccessible, obsolete and useless. However, we
cannot delete your data. It will still be accessible with some effort. If you
are serious with our platform, please follow our GitHub, social channels and
[Google Group](https://groups.google.com/forum/#!forum/windingtree) to be
informed about the upcoming changes.**

## Playground

The "playground" environment is supposed to be used by anyone
who wishes to experiment with the WT platform. It contains some
fake data. These are the key addresses:

- WT index: [0x3b476ac17ffea8dcf2dbd5ef787a5baeeebe9984](https://ropsten.etherscan.io/address/0x3b476ac17ffea8dcf2dbd5ef787a5baeeebe9984)
- WT Read API: https://playground-api.windingtree.com
- WT Write API: https://playground-write-api.windingtree.com
- WT Notification API: https://playground-notification-api.windingtree.com
- WT Search API: https://playground-search-api.windingtree.com
- Hotel explorer: https://hotel-explorer-playground.windingtree.com

## Demo

Supposed to serve for more "serious" (although still somewhat
"beta" in nature) attempts at integration with WT. The available
data should be real. Currently used by the crypto booking app
(booking.windingtree.com).

- WT index: [0x933198455e38925bccb4bfe9fb59bac31d00b4d3](https://ropsten.etherscan.io/address/0x933198455e38925bccb4bfe9fb59bac31d00b4d3)
- WT Read API: https://demo-api.windingtree.com
- WT Write API: https://demo-write-api.windingtree.com
- WT Search API: https://demo-search-api.windingtree.com
- Hotel explorer: https://hotel-explorer-demo.windingtree.com

## Building another environment

If you wish to prepare your own standalone WT environment,
please consult the [Preparing environment](developer-guides/preparing-environment.md) page.
