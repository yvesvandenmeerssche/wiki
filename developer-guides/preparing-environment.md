# Preparing the whole WT environment

To work with WT, you can use one of our [publicly accessible WT
environments](../developer-resources.md##publicly-available-wt-deployments).
If you wish to prepare your own environment instead, read
further.

## Creating a new environment

At the very least, the smart contracts that form WT's very core
need to be deployed somewhere. In addition to that, you might
want to setup wt-read-api and / or wt-write-api for convenience
purposes.

To achieve that, you can follow the following sequence of
actions.

1. Prepare a private key with a corresponding mnemonic.
2. Sign up for [Infura](https://infura.io/) to get an API key for communicating with
   their Ethereum nodes.
3. Check out the [wt-contracts](https://github.com/windingtree/wt-contracts) repository
   and deploy a new set of WT contracts by following instructions in the repository
   README. Note the resulting WT index address somewhere.
4. Configure and deploy a [wt-read-api](https://github.com/windingtree/wt-read-api) instance.
5. Configure and deploy a [wt-write-api](https://github.com/windingtree/wt-write-api) instance. 
6. As a final step, you might use [wt-fixtures](https://github.com/windingtree/wt-fixtures) to
   populate your WT instance with some test data (hand-crafted or generated).

## Resetting an existing environment

If you'd like to bring an existing environment back to the
initial state, you can do the following:

1. Deploy a new set of WT contracts, thus creating a new, clean
   WT index.
2. Re-configure the APIs to use the new index.
3. (Optional) Reset the wt-write-api database.
4. (Optional) Reupload initial data using [wt-fixtures](https://github.com/windingtree/wt-fixtures).
