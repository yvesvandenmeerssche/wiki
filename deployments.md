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


# Smart contracts

## Ethereum Mainnet

| Name                        | Address                                                                                                               | ENS                                                                             | Status     |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- | ---------- |
| Lif Token                   | [0xeb9951021698b42e4399f9cbb6267aa35f82d59d](https://etherscan.io/token/0xeb9951021698b42e4399f9cbb6267aa35f82d59d)   | [token.windingtree.eth](https://etherscan.io/enslookup?q=token.windingtree.eth) | Active     |
| ICO                         | [0x9df3a24d738ae98dea766cd89c3aef16583a4daf](https://etherscan.io/address/0x9df3a24d738ae98dea766cd89c3aef16583a4daf) | [ico.windingtree.eth](https://etherscan.io/enslookup?q=ico.windingtree.eth)     | Deprecated |
| Foundation Main Multisig    | [0xdad697274f95f909ad12437c516626d65263ce47](https://etherscan.io/address/0xdad697274f95f909ad12437c516626d65263ce47) | [windingtree.eth](https://etherscan.io/enslookup?q=windingtree.eth)             | Active     |
| Market Validation Mechanism | [0x27218c41e1054dc0484acd2ad35def0ffd17782a](https://etherscan.io/address/0x27218c41e1054dc0484acd2ad35def0ffd17782a) | [mvm.windingtree.eth](https://etherscan.io/enslookup?q=mvm.windingtree.eth)     | Active     |
| Foundation Quick Multisig   | [0xe7dfdf4b7dd5e9bd0bb94d59379219b625a6433d](https://etherscan.io/address/0xe7dfdf4b7dd5e9bd0bb94d59379219b625a6433d) |                                                                                 | Active     |
| Founders Vesting            | [0xed7eabc86201e647e37bb53419a7ad69bfc5b944](https://etherscan.io/address/0xed7eabc86201e647e37bb53419a7ad69bfc5b944) |                                                                                 | Active     |
| Foundation Vesting          | [0x41a09d330f3d9bc4d269bfa872bc06db1c5a04fc](https://etherscan.io/address/0x41a09d330f3d9bc4d269bfa872bc06db1c5a04fc) |                                                                                 | Active     |

## Ethereum Ropsten

| Name                        | Address                                                                                                               | ENS | Status     |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------- | --- | ---------- |
| Lif Token                   | [0xb6e225194a1c892770c43d4b529841c99b3da1d7](https://ropsten.etherscan.io/address/0xb6e225194a1c892770c43d4b529841c99b3da1d7)   |     | Active     |
