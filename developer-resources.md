# Development resources

Would you like to start with development on top of Winding Tree platform?
This is the place to be then! In this document, we try to lay out all the
current technical ideas and solutions that will help you in the beginning.

If there are any questions, don't hesitate to contact us (see below), we are
very happy to help you.

If you want to see some code, head over to our [Developer guides](developer-guides/README.md).

## Overall architecture

The overall architecture (right now for Hotels only) is described in
[WT Hotel Architecture](https://github.com/windingtree/wiki/blob/master/architecture-hotel.md).

After reading that, you should have an idea of how we would like to
build a reasonable version of our platform.

## Technical solution

If you would like to make yourself familiar with our many repositories here on GitHub,
check out the [Development and release handbook](https://github.com/windingtree/wiki/blob/master/development-handbooks/development.md).

After reading that you should have a good understanding of what's happening
where and how we operate during development.

If you would like to start working on your own application using the Winding Tree
platform, the [wt-read-api](https://github.com/windingtree/wt-read-api),
the [wt-write-api](https://github.com/windingtree/wt-write-api)
or [wt-js-libs](https://github.com/windingtree/wt-js-libs) is a probably a
good place to start. But please, read on to find out what branch you should
be working with.

### Documentation

As we are evolving very quickly we don't have a dedicated documentation or
developer portal. Right now we can offer live links to automatically generated
documentation which might be sometimes outdated, change over time and
might not be always available.

- **LífToken** - https://github.com/windingtree/LifToken/tree/develop/docs
- **wt-contracts** - https://github.com/windingtree/wt-contracts/tree/develop/docs
- **wt-js-libs** - https://github.com/windingtree/wt-js-libs/blob/develop/docs/reference.md
- **wt-read-api** - https://github.com/windingtree/wt-read-api/blob/master/docs/swagger.yaml or [more readable here](https://playground-api.windingtree.com/docs)
- **wt-write-api** - https://github.com/windingtree/wt-write-api/blob/master/docs/swagger.yaml or [more readable here](https://playground-write-api.windingtree.com/docs)
- **wt-update-api** - https://github.com/windingtree/wt-update-api/blob/master/docs/swagger.yaml or [more readable here](https://playground-update-api.windingtree.com/docs)
- **Data format** - https://github.com/windingtree/wiki/blob/master/hotel-data-swagger.yaml or [more readable here](https://windingtree.github.io/docs/swagger-ui/)

## What is the current version?

There's a lot going on so you might be a little confused.

In [Q1 of 2018](https://github.com/windingtree/wiki/blob/master/dev-updates/2018-Q1.md)
we have published all we had as a `0.1.x` version. That included a set of smart
contracts modelling a hotel reservation system. Every piece of data was stored on-chain.
It was possible to make bookings with
[LífToken](https://github.com/windingtree/LifToken/tree/0ce08793184795e528cf38560d7a6b07f49bb0f4)
in its [ERC-827](https://github.com/ethereum/EIPs/issues/827) flavour. We've also released
a [demo app](https://demo.windingtree.com) where you can play around with it on Ropsten testnet.
We've also released a [REST API](http://playground-api.windingtree.com/) (docs
[here](https://playground-api.windingtree.com/docs/)) that was very closely following the aforementioned
smart contracts. All of these can be found under respective tags on GitHub.
If you'd like to make yourself familiar with basic blockchain concepts such as transactions
and get some experience with its behaviour and limitations, it can be a good learning material.

During [Q2 of 2018](https://github.com/windingtree/wiki/blob/master/dev-updates/2018-Q2.md)
we started to implement a [new architecture](https://github.com/windingtree/wiki/blob/master/architecture-hotel.md).
That resulted into multiple releases in `0.2.x` series marked as `@next` on NPM. The development
happened mainly in `proposal/next` branch which was eventually merged back to develop and with
a next release back to master.

In Q3 of 2018, we're working hard on making the [Summer release of 2018](https://github.com/orgs/windingtree/projects/1)
happen. You can usually track a more granular progress in the respective repositories under simliarly
named milestone. Our goal for that release is to version everything as `1.0.0` tag and move on from there.
If you are serious about integrating with Winding Tree platform, that's a place to start for you. But beware,
things might change at any time as we move forward to the end of the Summer.

## Publicly available WT deployments

See a [dedicated page](deployments.md) where you will find URLs, Ethereum addresses and more.

## What are you working on?

You could either follow our repositories here on GitHub or for a very quick
overview, you can check our [roadmap](https://github.com/windingtree/wiki/blob/master/roadmap.md).
We're trying hard to make everything happen on time and you can check our quarterly
[dev updates](https://github.com/windingtree/wiki/tree/master/dev-updates).

If you'd like to follow the development on a day-to-day basis or would like to give us a hand,
you can check our public Trello boards - for [backend](https://trello.com/b/ZREEkuOX/winding-tree-work-in-progress)
and for [frontend](https://trello.com/b/5V9jP2uD/winding-tree-apps). We're still figuring out
if this is a best way to collaborate with the community, so a few things might be missing there.
But you should be able to get a feeling of what we're dealing with now and in the near future
in greater detail than in the Roadmap document.

## I still have questions

If you have really technical questions, probably the best place to go is our
[Google Group](https://groups.google.com/forum/#!forum/windingtree). You'll usually find someone
with a hands-on experience there. Or of course open up an issue here on GitHub.

We're also active on [Telegram](https://t.me/windingtree) and many other channels
that you can find on [windingtree.com](https://windingtree.com/).
