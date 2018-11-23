# Removing a hotel

- The address of Winding Tree index used in this example is for demo purposes only. There is only example data.
- We don't guarantee any hotel to exist, so before running this example you might first need to
[register your own hotel](registering-hotel.md).

## REST API

- In order to work with the [wt-write-api](https://github.com/windingtree/wt-write-api),
you need to have an account registered. (See [registering hotel](registering-hotel.md) to find out how to do that).
- The sample deployment on https://playground-write-api.windingtree.com is for demonstration
purposes only and is re-deployed every 24 hours. Your accounts will be lost after re-deployment.
- This code works with `0.1.0` version deployed on https://playground-write-api.windingtree.com although
the data will be different.
- Hotel deletion notification will be published automatically by
the API server to the given notifications address, if available.

### Deleting a hotel

- You are only allowed to delete your own hotel.

```sh
# Replace X-Access-Key with the result of account creation above
$ curl -X DELETE https://playground-write-api.windingtree.com/hotels -H 'Content-Type: application/json' \
  -H 'X-Access-Key: usgq6tSBW+wDYA/MBF367HnNp4tGKaCTRPy3JHPEqJmFBuxq1sA7UhFOpuV80ngC' \
  -H 'X-Wallet-Password: windingtree'

# Response will be 204
```


## Javascript library

- Dependencies
```
  "@windingtree/off-chain-adapter-http": "2.0.0",
  "@windingtree/off-chain-adapter-swarm": "3.2.0",
  "@windingtree/wt-js-libs": "0.3.0"
```

```js
// 1. Load the libs
const WTLibs = require('@windingtree/wt-js-libs');
const SwarmAdapter = require('@windingtree/off-chain-adapter-swarm');
const HttpAdapter = require('@windingtree/off-chain-adapter-http');

// 2. Configure the wt-js-libs
const libs = WTLibs.createInstance({
  dataModelOptions: {
    // We are using Ropsten as testnet, our demo index is on 0x933198455e38925bccb4bfe9fb59bac31d00b4d3
    provider: 'https://ropsten.infura.io/',
  },
  offChainDataOptions: {
    adapters: {
      // WT is using bzz-raw protocol to access data on swarm
      'bzz-raw': {
        options: {
          swarmProviderUrl: 'https://swarm-gateways.net/',
        },
        create: (options) => {
          return new SwarmAdapter(options);
        },
      },
      // WT is not using http at the moment, we only resolve https
      https: {
        create: () => {
          return new HttpAdapter();
        },
      },
    },
  },
});

// !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
// !!!!!!!! NEVER USE THIS WALLET !!!!!!!!
// !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
const WALLET_FILE = {"version":3,"id":"7fe84016-4686-4622-97c9-dc7b47f5f5c6","address":"d037ab9025d43f60a31b32a82e10936f07484246","crypto":{"ciphertext":"ef9dcce915eeb0c4f7aa2bb16b9ae6ce5a4444b4ed8be45d94e6b7fe7f4f9b47","cipherparams":{"iv":"31b12ef1d308ea1edacc4ab00de80d55"},"cipher":"aes-128-ctr","kdf":"scrypt","kdfparams":{"dklen":32,"salt":"d06ccd5d9c5d75e1a66a81d2076628f5716a3161ca204d92d04a42c057562541","n":8192,"r":8,"p":1},"mac":"2c30bc373c19c5b41385b85ffde14b9ea9f0f609c7812a10fdcb0a565034d9db"}};
const PASSWORD = 'windingtree';
// This is for demo purposes only, it's the same
// wallet used in https://github.com/windingtree/wt-demo-app
// Never use this wallet for any production data,
// everyone would have access to it.
// However, this Wallet has some Ether on Ropsten,
// so this example should work.

// 3. Remove your hotel from Winding Tree platform
(async () => {
  // Get an instance of WTIndex wrapper
  const index = await libs.getWTIndex('0x933198455e38925bccb4bfe9fb59bac31d00b4d3');

  // Create a Wallet abstraction and unlock it.
  const wallet = await libs.createWallet(WALLET_FILE);
  wallet.unlock(PASSWORD);

  // Get a hotel instance - for obvious reasons, you have to
  // set the hotel address to some existing one.
  const hotel = await index.getHotel('existing-hotel-address');
  try {
    // Remove the hotel
    // a. Get ready transaction data
    const { transactionData, eventCallbacks } = await index.removeHotel(hotel);
    // b. Sign and the transaction. You probably don't have to use our wallet abstraction.
    // This signs a transaction and sends it to be mined. You can get finer control
    // of this by using your own eventCallbacks, not awaiting the Promise etc.
    const receipt = await wallet.signAndSendTransaction(transactionData, eventCallbacks);
    // After the transaction is mined, you get
    // a receipt which contains a transaciontHash, among other useful things.
    console.log('transaction to check: ', receipt.transactionHash);
    // Ideally, publish a deletion notification for subscribers
    // to immediately find out about the change. For more
    // details, please have a look at https://github.com/windingtree/wt-notification-api.
  } finally {
    // Don't forget to lock your wallet after you are done, you
    // don't want to leave your private keys lying around.
    wallet.lock();
  }
})();
```
