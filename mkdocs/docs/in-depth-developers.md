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
