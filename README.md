Fanfury

![Banner!](assets/banner.png)

# Setting Up a Genesis Fury Validator

This document includes instructions for validators who intend to participate in the launch of the Fury mainnet. Please note:

1. This process is intended for technically inclined people who have participated in Fury testnets and other `cosmos-sdk` based blockchain launches. Experience running production IT systems is strongly recommended.
2. Fury staked during genesis will be at risk of 5% slashing if your validator double signs. If you accidentally misconfigure your validator setup, this can easily happen, and slashed Fury are not expected to be recoverable by any means. Additionally, if you double-sign, your validator will be tombstoned and you will be required to change operator and signing keys.
3. You will be creating public key accounts that are restored via their mnemonic. It is vital that you securely backup and store your mnemonic for any accounts that are created during this process. **Failure to do so can result in the irrecoverable loss of all Fury tokens**.


## Instructions

* [Validator Sale Participants](#for-validator-sale-participants)
* [Founder Rewards Participants](#for-founder-rewards-participants)

## For Validator Sale Participants

You will be preparing two documents:

1. `gentx.json` a signed transaction with your validator key that will create a validator at genesis with 50 self-delegated Fury.
2. `account.json` a json file containing the address and amount of Fury tokens you will receive at this address. The tokens at this address will be subject to vesting terms based on the contract you signed.

Once the documents are created, you will place them in a new directory in the `gentx` folder of this repo. Choose a unique directory name.

<br>

#### Prepare documents

1. Install `fud` version v0.3.0

##### Requires Go 1.13+

```sh
git clone https://github.com/four4two/fury
cd fury
git checkout v0.3.0
make install
fud init <your-validator-moniker> --chain-id fury-1
```

2. Create a key for your validator account. Back up the mnemonic and store the key information securely.

```sh
fucli keys add <your-validator-key-name>
```

3. Assign 50 Fury to your validator

```sh
fud add-genesis-account $(fucli keys show <your-validator-key-name> -a) 50000000ufury
```

4. Sign a genesis transaction

```sh
fud gentx \
  --amount 50000000ufury \
  --commission-rate <commission_rate> \
  --commission-max-rate <commission_max_rate> \
  --commission-max-change-rate <commission_max_change_rate> \
  --pubkey <consensus_pubkey> \
  --name <key_name>
```

**NOTE:**  If you would like to override the memo field use the `--ip` and `--node-id` flags for the `fud gentx` command above. `pubkey` can be obtained using `fud tendermint show-validator`

5. Copy the gentx you created to a new directory in the `gentx` directory of this repo and name it `gentx.json`

```sh
cp $HOME/.fud/config/gentx/gentx-<node_id>.json ./gentx/<your-directory>/gentx.json
```

6. Create a key for your vesting account

```sh
fucli keys add <your-vesting-account-key-name>
```

7. Create an `account.json` file in your new directory

```sh
printf '{"account": "", "amount": ""}\n' > ./gentx/<your-directory>/account.json
```

8. Fill in the address and amount of ufury you expect to receive in the vesting account

Example: If you expect to receive 100,000 Fury

  * Subtract 50 Fury, which was allocated to your validator
  * multiply by 10^6 to convert Fury to ufury
  * (100000 - 50) * 10^6 = 99950000000 ufury

```json
{
  "account": "fury1...",
  "amount": "99950000000ufury"
}
```

9. Submit your directory, with `gentx.json` and `account.json` files, as a PR on this repo.

## For Founder Rewards Participants

Eligible participants in the founder rewards program will submit an account that will be included in genesis:

1. `account.json` a json file containing the address and amount of Fury tokens you will receive at this address. The tokens will be immediately available and not bonded to a particular validator. Founder badge participants are free to create a validator immediately after launch.

Once the `account.json` file is created, you will place it in a new directory in the `gentx` folder of this repo. Choose a unique directory name.

#### Prepare documents

1. Install `fud` version v0.3.0

##### Requires Go 1.13+

```sh
git clone https://github.com/four4two/fury
cd fury
git checkout v0.3.0
make install

fud init <your-moniker> --chain-id fury-1
```

2. Create a key for your account. Back up the mnemonic and store the key information securely.

```sh
fucli keys add <your-account-key-name>
```

3. Create an `account.json` file in a new directory in the `gentx` directory of this repo.


```sh
printf '{"account": "", "amount": ""}\n' > ./gentx/<your-directory>/account.json
```

4. Fill in the address and amount of ufury you expect to receive in the account. Refer to the founder [documentation](https://github.com/Fury-Labs/fury/blob/master/docs/REWARDS.md) for reward amounts and eligibility.

Example: If you expect to receive 3000 Fury

  * multiply by 10^6 to convert Fury to ufury
  * 3000 * 10^6 = 3000000000 ufury

```json
{
  "account": "fury1...",
  "amount": "3000000000ufury"
}
```

5. Submit your directory, with `account.json` file, as a PR on this repo.

## Helpful resources

For resources that references `gaiacli`, you can replace `gaiacli` with `fucli`

* [Using the Cosmos app and a Ledger device to store your Fury keys](https://cosmos.network/docs/cosmos-hub/delegator-guide-cli.html#cosmos-accounts)
* [Validator Security](https://cosmos.network/docs/cosmos-hub/validators/security.html#validator-security)
* [Creating a validator after mainnet launch](https://cosmos.network/docs/cosmos-hub/validators/validator-setup.html#create-your-validator)


## Conclusion

See you all at launch! Join the discord!

---
*Disclaimer: This content is provided for informational purposes only,
and should not be relied upon as legal, business, investment, or tax
advice. You should consult your own advisors as to those matters.
References to any securities or digital assets are for illustrative
purposes only and do not constitute an investment recommendation or
offer to provide investment advisory services. Furthermore, this content
is not directed at nor intended for use by any investors or prospective
investors, and may not under any circumstances be relied upon when
making investment decisions.*

This work, ["Fury Genesis Validators
Guide"](https://github.com/osmosis-labs/networks/genesis-validators.md),
is a derivative of ["Agoric Validator
Guide"](https://github.com/Agoric/agoric-sdk/wiki/Validator-Guide) used
under [CC BY](http://creativecommons.org/licenses/by/4.0/). The Agoric
validator gudie is itself is a derivative of ["Validating Fury
Mainnet"](https://medium.com/four4two/validating-fury-mainnet-72fa1b6ea579)
by [Kevin Davis](https://medium.com/@kevin_35106), used under [CC
BY](http://creativecommons.org/licenses/by/4.0/). "Fury Validator
Guide" is licensed under [CC
BY](http://creativecommons.org/licenses/by/4.0/) by [Fury
Labs](https://fury.zone/). It was extensively modified to be relevant
to the Fury Chain.
