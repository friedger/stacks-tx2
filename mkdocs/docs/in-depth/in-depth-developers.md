# In-Depth: Sponsored Transactions for Developers

Sponsored transactions were added to the Stacks blockchain to enable developers and/or infrastructure operators to pay for users to call into their smart contracts, even if users do not hold STX to pay for fees. (See [SIP-005](https://github.com/stacksgov/sips/blob/main/sips/sip-005/sip-005-blocks-and-transactions.md#transaction-authorizations)).

The process consists of two steps: 1) signing by user and 2) signing by sponsor

## Signing Transaction by User

Sponsored transactions are build in the same way as normal transactions. The only difference is that fees are set to 0 and the sponsored flag is set to true. The sponsored transaction is then send to the user's wallet for confirmation. The wallet can't broadcast the transaction, instead it is returned to the app and needs to be handled separately.

An example with [@stacks/connect](https://docs.hiro.so/stacks/connect/packages/connect) library looks like the following (see also [send-many example](https://github.com/friedger/stacks-send-many/blob/main/src/components/SendManyInputContainer.tsx#L532)):

```js
const txOptions = {...};
txOptions.fee = 0;
txOptions.sponsored = true;

openContractCall({...txOptions,
        onFinish: (finishedTxData) => handleSignedSponsoredTransaction(finishTxData)
    }
)
```

## Signing Transaction by Sponsor

The signed transaction is returned from the wallet as part of the finished transaction data. The authentication type of the transaction will be of type `Sponsored`

```js
data.stacksTransaction.auth.authType === AuthType.Sponsored;
```

The serialized transaction (`data.txRaw`) can then be sent to a server for signing by the sponsor. There, the sponsor sets the `fee` for the transaction, add the sponsor's `nonce` and signs the transaction with the private key.

An example using the javascript library [@stacks/transactions](https://docs.hiro.so/stacks/stacks.js/packages/transactions) on a server looks like the following (see also [not-sponsoring example](https://github.com/friedger/stacks-not-sponsoring/blob/main/src/index.ts#L95)):

```js
import {deserializeTransaction, sponsorTransaction} from '@stacks/transactions';

...
const transaction = deserializeTransaction(txRaw);
const feeEstimate = await estimateTransactionFeeWithFallback(transaction, network);
const sponsorNonce = nonce; // next nonce of sponsor account

const sponsoredTx = await sponsorTransaction({
    sponsorPrivateKey: env.SPONSOR_PRIVATE_KEY,
    transaction,
    network,
    fee,
    sponsorNonce: nonce,
});

const result = await broadcastTransaction(sponsoredTx);
```

## Webservices for Sponsoring Transactions

As mentioned above the transaction signing by sponsor usually happens on a server that holds the private key of the sponsor. There are two open source projects that implemented such a webservice that provide both the same API:

- [Tx2 service](https://github.com/friedger/stacks-not-sponsoring/): basic service that allows to sponsor send-many calls for $NOTHING, uses single sponsoring address.
- [Xverse service](https://github.com/secretkeylabs/stacks-transaction-sponsor/): advanced service that allows to sponsor nft transfers, uses a randomly selected address from a list of sponsoring addresses.

### API Reference

#### sponsor

`/v1/sponsor` (POST)

The request body contains the user signed transaction as json property `tx`. The transaction must be created as a sponsored transaction. The service will sign the transaction as the sponsor and broadcast it.

##### Example

Request:

```
curl -X POST http://localhost:3000/v1/sponsor \
 -H "Content-Type: application/json" \
 -d '{"tx": "feedc00d1...user-signed-transaction-hex"}'
```

Result:

```json
{
  "txid": "c001c0de...tx-id",
  "rawTx": "feedc00d2...sponsored-transaction-hex"
}
```

#### info

`/v1/info` (GET)

The request returns information about the service in JSON. The result is a list of addresses used by the service for sponsoring transactions.

For tx2, the result contains also the minimum fees for NOT sponsoring and the balance of the sponsor.

##### Example

```
curl -X GET http://localhost:3000/v1/info'
```

Result:

```json
{
  "active": "true",
  "sponsor_addresses": ["SP1234...", "SP5678..."]
}
```
