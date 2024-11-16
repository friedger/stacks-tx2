# Transaction Bundles

Currently, sponsors need to find individual ways to receive compensation for sponsoring transactions. Furthermore, the compensation and the execution of the sponsored transaction might not happen at the same time. If one fails, either the sponsor or the user would loose the fees.

There is a proposal about bundling two or more transactions into one atomic execution ([issue #4235](https://github.com/stacks-network/stacks-core/issues/4235)). For sponsored transactions, a second transaction for compensation could be bundled with the original transaction so that both transactions are executed or both fail.
