## Keypair

A keypair is consist of `signPublicKey`, `boxPublicKey`, `privateKey`, it's presented in memory like:

```typescript
export class Keypair {
  signPublicKey: Key;
  boxPublicKey: Key;
  privateKey: Key;
}
```

The `signPublicKey` works together with `privateKey` to perform chain database transaction.

If you'd like your asset to be presented as cryptographic in chain database then you should to use
a password to encrypt your asset, the sdk will do the encryption work for you just needs you to provide
a password but it's your responsibility to keep the password is safe.

Once the asset in ethereum is paid, you should to tell the buyer what is the password of your encrypted asset.
You cannot send the plain password over network so you'll try to use the `boxPublicKey` of buyer to encrypt the
password and send the encrypted stuff to buyer.

Buyer will decrypt the encrypted password which is encrypted with his `boxPublicKey` by using his own `privateKey`.

## asset id and mkt id

`asset id` is the identifier of an asset in chain database,
`mkt id` is the identifier of the same asset but in ethereum.

After an asset is published into chain database you will get the `asset id`
then you will use `asset id` to register the asset into ethereum and get the associated `mkt id`.

The asset price should be provided when the asset is being registering in ethereum.

## How to run

Stay at repo root dir and type below command

```
./node_modules/.bin/webpack-dev-server --config examples/seller/webpack.config.js --open
```
