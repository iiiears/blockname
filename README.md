blockname - bitcoin dns cache
=============================

This is a simple bitcoin-based DNS cache, using the blockchain as a backup cache for normal DNS resolution as well as to resolve alternative domains and TLDs (completely distributed, no registrars).

Simply publish your own domain name as a valid `OP_RETURN` output on *any* transaction with the text format `*.myname.com11223344`, these are called `hint` transactions:

* first byte is always the star character: `*` 
* for textual domain name hints, the second byte is always the dot character: `.`
  * followed by up to 30 valid [domain name](http://en.wikipedia.org/wiki/Domain_name) characters
  * the last 8 characters are always the IPv4 address octets hex encoded, this address is used as the dns server to forward the query to
* for binary hashname hints, the second byte is always the hash character: `#`
  * followed by 32 bytes of the hashname
  * followed by 4 bytes of the IPv4 address
  * followed by 2 bytes of the port

The blockchain resolver will attempt to resolve all domains with traditional DNS, and only when they fail will it use any names that come from the cache hints.

In the background the resolver will continuously index all newly broadcast transactions that have a valid hints, storing only the unique hints that have the largest value transactions (larger inputs/outputs will replace smaller ones for the same domain name).

Hashnames are only resolved with a `.public` TLD and are always validated before being used.

## DHT Index

In order to not require every resolver to keep an index of every hint in the blockchain they may connect to a common DHT based on [Kademlia](https://en.wikipedia.org/wiki/Kademlia) and [telehash](http://telehash.org).  Unknown queries may ask peer resolvers on the DHT for hints and their transaction IDs so that they can be independently/locally verified.

Upon being cached from a verified DHT hint the local resolver must monitor new transactions for updates as long as the hint is cached.

> WIP, merging [dotPublic](https://github.com/telehash/dotPublic) into here

## Status

It is [currently working](http://testnet.coinsecrets.org/?to=320107.000001) on [testnet](http://blockexplorer.com/testnet/tx/6b6ea2fffa1ad59dc0eb716bf2a8386fe091eb180486e38c9e4a6c7458ec00fa) and being tested/developed for the main blockchain, these commands are working but expect them to change.

```
git clone https://github.com/quartzjer/blockname.git
cd blockname
npm install
```

Start a local DNS resolver (defaults to port `8053`)

```
node bin/serve.js
```

Start a process to sync and monitor the transactions on the blockchain:

```
node bin/scan.js
```

Register your own domain name hint on the blockchain, passing the domain and the IP address of a nameserver that will resolve it or the IP to return to any `A` queries.  Uses a testnet faucet service by default, may also pass an existing source transaction and destination address to refund to (run command w/ no args to see options)

```
node bin/register.js "somename.tld" 12.34.56.78
```

Now do a test resolution to the local cache server, it will check normal DNS first, then fallback to any indexed hints from the blockchain:

```
dig somename.tld @127.0.0.1 -p 8053
```

## Plans

After some more testing and docs, this will default to mainnet and become a `blocknamed` DNS resolver service and `blockname` registration command that anyone can `npm install -g blockname`.

After some more usage there will be a list of public blockname resolvers that can be used by anyone and a web-based registration tool and a chart of top hints in the blockchain.

## Thanks

Thanks to help from [William Cotton](https://github.com/williamcotton/blockcast).
