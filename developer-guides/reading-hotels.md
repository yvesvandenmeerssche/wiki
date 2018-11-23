# Listing and reading hotel details

- The address of Winding Tree index used in this example is for demo purposes only. There is only example data.

## REST API

- The results may be different for you. There's no guarantee of data stability.
- You can check from which WTIndex is the data downloaded on https://playground-api.windingtree.com/
- This data is based on `0.3.0` version of the API, for complete docs, see https://playground-api.windingtree.com/docs,
there are more features available.
- The API is deployed on an infrastructure, that puts servers to sleep if there's no traffic.
Your first request might therefore take a lot of time.

### Hotel listing
```sh
$ curl 'https://playground-api.windingtree.com/hotels'

{
   "items" : [
      {
         "location" : {
            "longitude" : "14.414170",
            "latitude" : "50.075388"
         },
         "name" : "Random hotel",
         "id" : "0x36903f12E9BDC70427d0Db2D5A865DB65db92AB8"
      },
      {
         "location" : {
            "latitude" : "50.075388",
            "longitude" : "14.414170"
         },
         "id" : "0xE1383bD3d193eC22d692FA1e5A53d868e5bC20A0",
         "name" : "Random hotel on Swarm"
      }
   ],
   "errors" : [
      {
         "originalError" : "Cannot download data: Error 404.",
         "data" : {
            "id" : "0x49c61B16be16B82c6EF4892b17A3D2E284207118"
         },
         "error" : "Cannot access off-chain data"
      }
   ]
}
```

```sh
$ curl 'https://playground-api.windingtree.com/hotels?limit=1&startWith=0xE1383bD3d193eC22d692FA1e5A53d868e5bC20A0&fields=name,description,images'

{
   "next" : "http://https://playground-api.windingtree.com/hotels?limit=1&startWith=0x49c61B16be16B82c6EF4892b17A3D2E284207118",
   "items" : [
      {
         "images" : [
            "https://raw.githubusercontent.com/windingtree/media/master/logo-variants/tree/png/tree--gradient-on-white.png",
            "https://raw.githubusercontent.com/windingtree/media/master/logo-variants/full-logo/png/logo--black-on-green.png"
         ],
         "id" : "0xE1383bD3d193eC22d692FA1e5A53d868e5bC20A0",
         "description" : "**Beautiful** hotel located in the center of _Prague, Czech Republic_.",
         "name" : "Random hotel on Swarm"
      }
   ],
   "errors" : [
      {
         "error" : "Cannot access off-chain data",
         "originalError" : "Cannot download data: Error 404.",
         "data" : {
            "id" : "0x49c61B16be16B82c6EF4892b17A3D2E284207118"
         }
      }
   ]
}
```

### Hotel detail

```sh
$  curl 'https://playground-api.windingtree.com/hotels/0xE1383bD3d193eC22d692FA1e5A53d868e5bC20A0'

{
   "id" : "0xE1383bD3d193eC22d692FA1e5A53d868e5bC20A0",
   "contacts" : {
      "general" : {
         "url" : "https://jirkachadima.cz",
         "phone" : "00420224371111",
         "email" : "chadima.jiri@gmail.com",
         "additionalContacts" : {
            "twitter" : "@windingtree",
            "Telegram" : "https://t.me/windingtree"
         },
         "ethereum" : "windingtree.eth"
      }
   },
   "address" : {
      "city" : "Prague",
      "country" : "Czech Republic",
      "line2" : "Nové Město",
      "line1" : "Rašínovo nábřeží 1981/80",
      "postalCode" : "12000"
   },
   "currency" : "CZK",
   "name" : "Random hotel on Swarm",
   "location" : {
      "latitude" : "50.075388",
      "longitude" : "14.414170"
   },
   "description" : "**Beautiful** hotel located in the center of _Prague, Czech Republic_.",
   "images" : [
      "https://raw.githubusercontent.com/windingtree/media/master/logo-variants/tree/png/tree--gradient-on-white.png",
      "https://raw.githubusercontent.com/windingtree/media/master/logo-variants/full-logo/png/logo--black-on-green.png"
   ],
   "amenities" : [
      "Beer garden",
      "WiFi",
      "Parking"
   ],
   "updatedAt" : "2018-06-19T15:53:00"
}
```

```sh
$ curl 'https://playground-api.windingtree.com/hotels/0xE1383bD3d193eC22d692FA1e5A53d868e5bC20A0?fields=address,contacts' 

{
   "address" : {
      "city" : "Prague",
      "line2" : "Nové Město",
      "postalCode" : "12000",
      "line1" : "Rašínovo nábřeží 1981/80",
      "country" : "Czech Republic"
   },
   "id" : "0xE1383bD3d193eC22d692FA1e5A53d868e5bC20A0",
   "contacts" : {
      "general" : {
         "url" : "https://jirkachadima.cz",
         "email" : "chadima.jiri@gmail.com",
         "ethereum" : "windingtree.eth",
         "phone" : "00420224371111",
         "additionalContacts" : {
            "Telegram" : "https://t.me/windingtree",
            "twitter" : "@windingtree"
         }
      }
   }
}

```

## Javascript library

### Dependencies
```
  "@windingtree/off-chain-adapter-http": "2.0.0",
  "@windingtree/off-chain-adapter-swarm": "3.2.0",
  "@windingtree/wt-js-libs": "0.3.0"
```
### Example code

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

// 3. Collect the data
// This has to be in an async block due to the nature of distributed data
(async () => {
  // Get an instance of WTIndex wrapper
  const index = await libs.getWTIndex('0x933198455e38925bccb4bfe9fb59bac31d00b4d3');

  // We can get all hotels available in the WTIndex
  // - We will get only hotels on valid and accessible addresses
  // - Every hotel is an abstraction over on-chain and off-chain data
  // - This should be reasonably fast, because only a single method call on WTIndex is performed
  const hotels = await index.getAllHotels();

  // Notice how the approach is totally protocol agnostic
  for (let hotel of hotels) {
    try {
      // You can benefit from a recursive shorthand method that downloads all of hotel data for you
      const serializedHotel = await hotel.toPlainObject();
      console.log(serializedHotel.address);
      // And you can access all of the data in a simple, synchronous way
      const hotelLocation = serializedHotel.dataUri.contents.descriptionUri.contents.location;
      
      // OR with much finer control and a lot of await calls, you can do this:
      
      // This actually fetches data from a hotel smart contract
      const hotelDataUri = await hotel.dataUri;
      // But it gets cached, so the second call is way faster
      const hotelDataUri2 = await hotel.dataUri;

      // Let's initialize the off-chain data index - but no data is downloaded yet
      const hotelDataIndex = await hotel.dataIndex;
      // Only the next command actually initiates the download of the data index document
      const offChainData = await hotelDataIndex.contents;
      const hotelDescriptionUri = offChainData.descriptionUri;
      // And only now the actual contents of descriptionUri gets downloaded
      const hotelDescriptionContents = await hotelDescriptionUri.contents;
      // This is fast, because the document is already cached in memory.
      const hotelName = hotelDescriptionContents.name;
      const hotelDescription = hotelDescriptionContents.description;
    } catch (e) {
      // A single hotel off-chain data access can fail, so it's a good practice to handle that.
      console.log(hotel.address, e);
    }
  }
})();
```
