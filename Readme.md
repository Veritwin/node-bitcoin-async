# node-bitcoin-async
[![npm][npm-image]][npm-url]
[![downloads][downloads-image]][downloads-url]
[![js-standard-style][standard-image]][standard-url]

[npm-image]: https://img.shields.io/npm/v/@veritwin/bitcoin-async.svg?style=flat
[npm-url]: https://npmjs.org/package/@veritwin/bitcoin-async

[downloads-image]: https://img.shields.io/npm/dm/@veritwin/bitcoin-async.svg?style=flat
[downloads-url]: https://npmjs.org/package/@veritwin/bitcoin-async

[standard-image]: https://img.shields.io/badge/code%20style-standard-brightgreen.svg?style=flat
[standard-url]: http://standardjs.com

This is a Bitcoin Core JSON-RPC API client with async processing in the style of the [Async](https://github.com/caolan/async.git)
Node.js module.

> Note: it is a modified version of the [bitcoin](https://github.com/jb55/node-bitcoin.git) Node.js module. In
> particular it changes the way how batch RPC method calls are processed. See [below](#batch-multiple-rpc-calls-into-single-http-request) for more details.

The API is equivalent to the API document [here](https://en.bitcoin.it/wiki/Original_Bitcoin_client/API_Calls_list).
The methods are exposed as lower camelcase methods on the `bitcoin.Client`
object, or you may call the API directly using the `cmd` method.

## Install

`npm install @veritwin/bitcoin-async`

## Examples

### Create client
```js
// all config options are optional
var client = new bitcoin.Client({
  host: 'localhost',
  port: 8332,
  user: 'username',
  pass: 'password',
  wallet: 'walletname',
  timeout: 30000
});
```

### Get balance across all accounts with minimum confirmations of 6

```js
client.getBalance('*', 6, function(err, balance, resHeaders) {
  if (err) return console.log(err);
  console.log('Balance:', balance);
});
```
### Getting the balance directly using `cmd`

```js
client.cmd('getbalance', '*', 6, function(err, balance, resHeaders){
  if (err) return console.log(err);
  console.log('Balance:', balance);
});
```

### Batch multiple RPC calls into single HTTP request

```js
var batch = [];
for (var i = 0; i < 10; ++i) {
  batch.push({
    method: 'getnewaddress',
    params: ['myaccount']
  });
}
client.cmd(batch, function(address, resHeaders, cb) {
  // This function is called after a successful call for each of the RPC methods
  //  in the batch
  console.log('Address:', address);
  cb();
}, function (err) {
  // This method is called after calling all the RPC methods or whenever an error takes
  //  place while calling ony of the methods
  if (err) return console.log(err);
});
```

## SSL
See [Enabling SSL on original client](https://en.bitcoin.it/wiki/Enabling_SSL_on_original_client_daemon).

If you're using this to connect to bitcoind across a network it is highly
recommended to enable `ssl`, otherwise an attacker may intercept your RPC credentials
resulting in theft of your bitcoins.

When enabling `ssl` by setting the configuration option to `true`, the `sslStrict`
option (verifies the server certificate) will also be enabled by default. It is
highly recommended to specify the `sslCa` as well, even if your bitcoind has
a certificate signed by an actual CA, to ensure you are connecting
to your own bitcoind.

```js
var client = new bitcoin.Client({
  host: 'localhost',
  port: 8332,
  user: 'username',
  pass: 'password',
  ssl: true,
  sslStrict: true,
  sslCa: fs.readFileSync(__dirname + '/myca.cert')
});
```
