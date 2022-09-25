# Fungible Token (FT)

Example implementation of a [Fungible Token] contract which uses [near-contract-standards].

[fungible token]: https://nomicon.io/Standards/Tokens/FungibleToken/Core
[near-contract-standards]: https://github.com/near/near-sdk-rs/tree/master/near-contract-standards

NOTES:

- The maximum balance value is limited by U128 (2\*\*128 - 1).
- JSON calls should pass U128 as a base-10 string. E.g. "100".
- This does not include escrow functionality, as `ft_transfer_call` provides a superior approach. An escrow system can, of course, be added as a separate contract.

## Pre-requisites

To develop Rust contracts you would need to:

- Install [Rustup](https://rustup.rs/):

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

- Add wasm target to your toolchain:

```bash
rustup target add wasm32-unknown-unknown
```

## Building

To build run:

```bash
./build.sh
```

## Testing

To test run:

```bash
cargo test --workspace -- --nocapture
```

# Using this contract

## Pre-requisites

Ensure `near-cli` is installed by running `near --version`. If not installed, install with:

```bash
npm install -g near-cli
```

## Deploy

### Quickest deploy

You can build and deploy this smart contract to a development account. Dev accounts are auto-generated accounts to assist in developing and testing smart contracts. Please see the [Standard deploy](#standard-deploy) section for creating a more personalized account to deploy to.

```bash
near dev-deploy --wasmFile res/fungible_token.wasm
```

Behind the scenes, this is creating an account and deploying a contract to it. On the console, notice a message like:

> Done deploying to dev-1234567890123

In this instance, the account is `dev-1234567890123`. A file has been created containing a key pair to
the account, located at `neardev/dev-account`. To make the next few steps easier, we're going to set an
environment variable containing this development account id and use that when copy/pasting commands.
Run this command to set the environment variable:

```bash
source neardev/dev-account.env
```

You can tell if the environment variable is set correctly if your command line prints the account name after this command:

```bash
echo $CONTRACT_NAME
```

### Standard deploy

This smart contract will get deployed to your NEAR account. For this example, please create a new NEAR account. Because NEAR allows the ability to upgrade contracts on the same account, initialization functions must be cleared. If you'd like to run this example on a NEAR account that has had prior contracts deployed, please use the `near-cli` command `near delete`, and then recreate it in Wallet. To create (or recreate) an account, please follow the directions on [NEAR Wallet](https://wallet.near.org/).

Switch to `mainnet`. You can skip this step to use `testnet` as a default network.

    export NEAR_ENV=mainnet

In the project root, log in to your newly created account with `near-cli` by following the instructions after this command:

    near login

Now we can deploy the compiled contract in this example to your account. Replace `<MY_ACCOUNT_NAME>` with the account name you just logged in with, including the `.near`:

    near deploy --wasmFile res/fungible_token.wasm --accountId <MY_ACCOUNT_NAME>

## Intermediate step

To make this tutorial easier to copy/paste, we're going to set an environment variable for your account id. If you deployed to a development account, execute:

    ID=$CONTRACT_NAME

Otherwise, replace `<MY_ACCOUNT_NAME>` with the account name you logged in with:

    ID=<MY_ACCOUNT_NAME>

You can tell if the environment variable is set correctly if your command line prints the account name after this command:

    echo $ID

## Initialization

FT contract should be initialized before usage. You can read more about metadata at ['nomicon.io'](https://nomicon.io/Standards/FungibleToken/Metadata.html#reference-level-explanation). Modify the parameters and create a token:

    near call $ID new '{"owner_id": "'$ID'", "total_supply": "1000000000000000", "metadata": { "spec": "ft-1.0.0", "name": "Mute DAO", "symbol": "MTDAO", "decimals": 8 }}' --accountId $ID

Get metadata:

    near view $ID ft_metadata

## Transfer example

Let's create an account to transfer some tokens to. This account will be a sub-account of the NEAR account you set up previously.

    near create-account bob.$ID --masterAccount $ID --initialBalance 1

Add storage deposit for Bob's account:

    near call $ID storage_deposit '' --accountId bob.$ID --amount 0.00125

Check balance of Bob's account, it should be `0` for now:

    near view $ID ft_balance_of '{"account_id": "'bob.$ID'"}'

Transfer tokens to Bob from the contract that minted these fungible tokens, exactly 1 yoctoNEAR of deposit should be attached:

    near call $ID ft_transfer '{"receiver_id": "'bob.$ID'", "amount": "19"}' --accountId $ID --amount 0.000000000000000000000001

Check the balance of Bob again with the command from before and it will now return `19`.
