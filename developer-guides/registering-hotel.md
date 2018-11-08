# Registering a hotel

- The address of Winding Tree index used in this example is for demo purposes only. There is only example data
and anyone can delete them, because the originating wallet with password is publicly available.

## Javascript library

- We don't provide an easy way in wt-js-libs how to store your off-chain data.
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

// 3. Get your off-chain data ready
// This is not provided by wt-js-libs out of the box
const offChainDataUri = 'https://jirkachadima.cz/wt/hotel-data-index.json';

// 4. Register your hotel to Winding Tree platform
(async () => {
  // Get an instance of WTIndex wrapper
  const index = await libs.getWTIndex('0x933198455e38925bccb4bfe9fb59bac31d00b4d3');

  // Create a Wallet abstraction and unlock it.
  const wallet = await libs.createWallet(WALLET_FILE);
  wallet.unlock(PASSWORD);

  try {
    // Register the hotel itself
    // a. Get ready transaction data
    const { hotel, transactionData, eventCallbacks } = await index.addHotel({
      manager: wallet.getAddress(),
      dataUri: offChainDataUri,
    });
    // b. Sign and send the transaction. You probably don't have to use our wallet abstraction.
    // This signs a transaction and waits for it to be mined. You can get finer control
    // of this by using your own eventCallbacks, not waiting for the promise to be resolved etc.
    const receipt = await wallet.signAndSendTransaction(transactionData, eventCallbacks);
    // After the transaction is mined, one of the eventCallbacks
    // sets the address of the freshly created hotel.
    const newHotelAddress = hotel.address;
    console.log('hotel address: ', newHotelAddress);
    // Ideally, publish a creation notification for subscribers
    // to immediately find out about the new hotel. For more
    // details, please have a look at https://github.com/windingtree/wt-update-api.
  } finally {
    // Don't forget to lock your wallet after you are done, you
    // don't want to leave your private keys lying around.
    wallet.lock();
  }
})();
```

## REST API

- In order to work with the [wt-write-api](https://github.com/windingtree/wt-write-api),
you need to have an account registered.
- The sample deployment on https://playground-write-api.windingtree.com is for demonstration
purposes only and is re-deployed every 24 hours. Your accounts will be lost after re-deployment.
- This code works with `0.3.0` version deployed on https://playground-write-api.windingtree.com although
the data might be different.

### Creating an account

- An account consists of your Ethereum wallet (password-protected) and a configuration of *uploaders*,
which tell the API where to upload the off-chain data.
- The JSON-format wallet can easily be created locally with [mycrypto](https://download.mycrypto.com/).
- In this example, we are using our demo wallet with password `windingtree`.
- In this example, we are using Swarm as our place for data and we are using a public gateway.
- For different configuration of data storage, please go see [wt-write-api documentation](https://github.com/windingtree/wt-write-api/tree/develop#uploaders)
- This `json` should be saved as `create-account.json` in a directory where you will be running the `curl` command.
- **Never use this wallet for anything in production. Others would have access to your assets as well.**

```json
{
  "wallet": {"version":3,"id":"7fe84016-4686-4622-97c9-dc7b47f5f5c6","address":"d037ab9025d43f60a31b32a82e10936f07484246","crypto":{"ciphertext":"ef9dcce915eeb0c4f7aa2bb16b9ae6ce5a4444b4ed8be45d94e6b7fe7f4f9b47","cipherparams":{"iv":"31b12ef1d308ea1edacc4ab00de80d55"},"cipher":"aes-128-ctr","kdf":"scrypt","kdfparams":{"dklen":32,"salt":"d06ccd5d9c5d75e1a66a81d2076628f5716a3161ca204d92d04a42c057562541","n":8192,"r":8,"p":1},"mac":"2c30bc373c19c5b41385b85ffde14b9ea9f0f609c7812a10fdcb0a565034d9db"}},
  "uploaders": {
    "root": {
      "swarm": {}
    }
  }
}
```

```sh
$ curl -X POST https://playground-write-api.windingtree.com/accounts -H 'Content-Type: application/json' --data @create-account.json

# Response values are randomly generated and will be different for you
{"accountId":"aa43edaf8266e8f8","accessKey":"usgq6tSBW+wDYA/MBF367HnNp4tGKaCTRPy3JHPEqJmFBuxq1sA7UhFOpuV80ngC"}
```

### Registering a hotel

- This is just an example data that passes the validation inside of the API.
- This `json` should be saved as `hotel-description.json` in a directory where you will be running the `curl` command.
- The data format might be slightly different to what we have in our [data structure](http://windingtree.github.io/docs/swagger-ui/).
- Hotel creation notification will be published automatically by the API server to the given notifications address, if available.

```json
{
  "notifications": "https://playground-update-api.windingtree.com/",
  "description": {
    "name": "Random hotel",
    "description": "**Beautiful** hotel located in the center of _Prague, Czech Republic_.",
    "location": {
      "latitude": 50.075388,
      "longitude": 14.414170
    },
    "contacts": {
      "general": {
        "email": "chadima.jiri@gmail.com",
        "phone": "00420224371111",
        "url": "https://windingtree.com",
        "ethereum": "windingtree.eth"
      }
    },
    "address": {
      "line1": "Rašínovo nábřeží 1981/80",
      "line2": "Nové Město",
      "postalCode": "12000",
      "city": "Prague",
      "country": "CZ"
    },
    "timezone": "Europe/Prague",
    "currency": "CZK",
    "amenities": [
      "minibar",
      "coffee machine"
    ],
    "images": [
      "https://raw.githubusercontent.com/windingtree/media/master/logo-variants/tree/png/tree--gradient-on-white.png",
      "https://raw.githubusercontent.com/windingtree/media/master/logo-variants/full-logo/png/logo--black-on-green.png"
    ],
    "roomTypes": {
      "twin-bed": {
        "name": "Twin bed",
        "description": "Room with a **twin bed**",
        "totalQuantity": 10,
        "occupancy": {
          "min": 1,
          "max": 2
        },
        "amenities": [
          "TV"
        ],
        "images": [
          "https://raw.githubusercontent.com/windingtree/media/web-assets/logo-variants/full-logo/png/logo--white.png"
        ],
        "properties": {
          "nonSmoking": "some"
        }
      },
      "king-size-bed": {
        "name": "King size bed",
        "description": "Room with a **king size bed**",
        "totalQuantity": 10,
        "occupancy": {
          "min": 1,
          "max": 2
        },
        "amenities": [
          "TV"
        ],
        "images": [
          "https://raw.githubusercontent.com/windingtree/media/web-assets/logo-variants/full-logo/png/logo--white.png"
        ],
        "properties": {
          "nonSmoking": "some"
        }
      }
    },
    "defaultCancellationAmount": 0,
    "cancellationPolicies": [
      {
        "from": "2018-01-01",
        "to": "2018-12-31",
        "deadline": 30,
        "amount": 50
      },
      {
        "from": "2018-01-01",
        "to": "2018-12-31",
        "deadline": 7,
        "amount": 100
      }
    ]
  },
  "ratePlans": {
    "basic-twin": {
      "name": "Basic twin bed",
      "description": "Off season price for twin bed",
      "currency": "EUR",
      "price": 80,
      "roomTypeIds": [
        "twin-bed"
      ],
      "availableForReservation": {
        "from": "2018-01-01",
        "to": "2018-12-31"
      },
      "availableForTravel": {
        "from": "2018-10-31",
        "to": "2019-04-30"
      },
      "modifiers": [
        {
          "adjustment": -10,
          "conditions": {
            "maxAge": 13
          }
        }
      ],
      "restrictions": {
        "bookingCutOff": {
          "min": 7,
          "max": 120
        },
        "lengthOfStay": {
          "min": 2,
          "max": 14
        }
      }
    },
    "summer-season-twin": {
      "name": "Summer season twin bed",
      "description": "Summer price for twin bed",
      "currency": "EUR",
      "price": 100,
      "roomTypeIds": [
        "twin-bed"
      ],
      "availableForReservation": {
        "from": "2018-01-01",
        "to": "2018-12-31"
      },
      "availableForTravel": {
        "from": "2019-05-01",
        "to": "2019-10-31"
      },
      "modifiers": [
        {
          "adjustment": -10,
          "conditions": {
            "maxAge": 13
          }
        }
      ],
      "restrictions": {
        "bookingCutOff": {
          "min": 7,
          "max": 120
        },
        "lengthOfStay": {
          "min": 2,
          "max": 14
        }
      }
    },
    "basic-king": {
      "name": "Basic king bed",
      "description": "Basic for king bed",
      "currency": "EUR",
      "price": 60,
      "roomTypeIds": [
        "king-size-bed"
      ],
      "availableForReservation": {
        "from": "2018-01-01",
        "to": "2018-12-31"
      },
      "availableForTravel": {
        "from": "2019-05-01",
        "to": "2019-10-31"
      },
      "modifiers": [
        {
          "adjustment": -10,
          "conditions": {
            "maxAge": 13
          }
        }
      ],
      "restrictions": {
        "bookingCutOff": {
          "min": 10,
          "max": 20
        },
        "lengthOfStay": {
          "min": 3,
          "max": 10
        }
      }
    },
    "summer-season-king": {
      "name": "Summer season king bed",
      "description": "Summer season for king bed",
      "currency": "EUR",
      "price": 120,
      "roomTypeIds": [
        "king-size-bed"
      ],
      "availableForReservation": {
        "from": "2018-01-01",
        "to": "2018-12-31"
      },
      "availableForTravel": {
        "from": "2019-05-01",
        "to": "2019-10-31"
      },
      "modifiers": [
        {
          "adjustment": -10,
          "conditions": {
            "maxAge": 13
          }
        }
      ],
      "restrictions": {
        "bookingCutOff": {
          "min": 10,
          "max": 20
        },
        "lengthOfStay": {
          "min": 3,
          "max": 10
        }
      }
    }
  },
  "availability": {
    "roomTypes": {
      "twin-bed": [
        {
          "date": "2018-07-07",
          "quantity": 2,
          "restrictions": {
            "noArrival": true
          }
        },
        {
          "date": "2018-07-08",
          "quantity": 2,
          "restrictions": {
            "noDeparture": true
          }
        },
        {
          "date": "2018-07-09",
          "quantity": 2
        },
        {
          "date": "2018-07-10",
          "quantity": 1
        },
        {
          "date": "2018-07-11",
          "quantity": 0
        },
        {
          "date": "2018-07-12",
          "quantity": 0
        },
        {
          "date": "2018-07-13",
          "quantity": 0
        },
        {
          "date": "2018-07-14",
          "quantity": 0,
          "restrictions": {
            "noArrival": true
          }
        },
        {
          "date": "2018-07-15",
          "quantity": 0,
          "restrictions": {
            "noDeparture": true
          }
        }
      ],
      "king-size-bed": [
        {
          "date": "2018-07-07",
          "quantity": 2,
          "restrictions": {
            "noArrival": true
          }
        },
        {
          "date": "2018-07-08",
          "quantity": 2,
          "restrictions": {
            "noDeparture": true
          }
        },
        {
          "date": "2018-07-09",
          "quantity": 2
        },
        {
          "date": "2018-07-10",
          "quantity": 1
        },
        {
          "date": "2018-07-11",
          "quantity": 0
        },
        {
          "date": "2018-07-12",
          "quantity": 0
        },
        {
          "date": "2018-07-13",
          "quantity": 0
        },
        {
          "date": "2018-07-14",
          "quantity": 0,
          "restrictions": {
            "noArrival": true
          }
        },
        {
          "date": "2018-07-15",
          "quantity": 0,
          "restrictions": {
            "noDeparture": true
          }
        }
      ]
    }
  }
}
```

```sh
# Replace X-Access-Key with the result of account creation above
$ curl -X POST https://playground-write-api.windingtree.com/hotels -H 'Content-Type: application/json' \
  -H 'X-Access-Key: usgq6tSBW+wDYA/MBF367HnNp4tGKaCTRPy3JHPEqJmFBuxq1sA7UhFOpuV80ngC' \
  -H 'X-Wallet-Password: windingtree' \
  --data @hotel-description.json

# This value will be different
{"address":"0xA603FF7EA9A1B81FB45EF6AeC92A323a88211f40"}
```

---
Contacts:
[Google Group](https://groups.google.com/forum/#!forum/windingtree) |
[Twitter](https://twitter.com/windingtree) |
[Medium](http://blog.windingtree.com/) |
[YouTube](https://www.youtube.com/channel/UCFuemEOhCfenYMoNdjD0Aew) |
[Telegram](https://t.me/windingtree) |
[Reddit](https://reddit.com/r/windingtree) |
[BitcoinTalk](https://bitcointalk.org/index.php?topic=1946065)
