# Block hacked account

Sponsored transactions have been seen in the wild to block a hacked account from doing any further transactions. This is possible due to the dependency between user and sponsor.

When a third party has access to the private key of the user's account, the user can try to block their own account. The user can initiate a sponsored transaction and the sponsoring account can set a nonce for the sponsor with a gap. This prevents that the transaction is confirmed by the network. The user account cannot send any transactions anymore.

The third party could try to replace the transaction by higher fees. However, the sponsor can do the same. The difference is that the sponsor can get back their fees when the transaction in mempool expires after around 2 days.
