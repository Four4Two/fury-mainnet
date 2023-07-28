Blackfury

![Banner!](assets/banner.png)

# Setting Up a Genesis Blackfury Validator

This document includes instructions for validators who intend to participate in the launch of the Blackfury mainnet. Please note:

1. This process is intended for technically inclined people who have participated in other `cosmos-sdk` based blockchain launches. Experience running production IT systems is strongly recommended.
2. FURY staked during genesis will be at risk of 5% slashing if your validator double signs. If you accidentally misconfigure your validator setup, this can easily happen, and slashed FURY are not expected to be recoverable by any means. Additionally, if you double-sign, your validator will be tombstoned and you will be required to change operator and signing keys.
3. You will be creating public key accounts that are restored via their mnemonic. It is vital that you securely backup and store your mnemonic for any accounts that are created during this process. **Failure to do so can result in the irrecoverable loss of all FURY tokens**.


## Instructions for Fan Club-Owner Validator Participants

You will be preparing a document:

`gentx.json` a signed transaction with your validator key that will create a validator at genesis with 20000 self-delegated Fury.

Once the documents are created, you will place them in a new directory in the `gentx` folder of this repo. Choose a unique directory name.

<br>

#### Prepare documents

1. Install `fury` version v0.4.1

##### Requires Go v1.20.2

```sh
git clone https://github.com/Four4Two/fury
cd fury
git checkout v0.4.1
make install
fury init <your-validator-moniker> --chain-id highbury_710-1
```

2. Create a key for your validator account. Back up the mnemonic and store the key information securely.

```sh
fucli keys add <your-validator-key-name>
```

3. Assign 20000 FURY to your validator

```sh
fury add-genesis-account $(fucli keys show <your-validator-key-name> -a) 20000000000ufury
```

4. Sign a genesis transaction

```sh
fury gentx \
  --amount 20000000000ufury \
  --commission-rate <commission_rate> \
  --commission-max-rate <commission_max_rate> \
  --commission-max-change-rate <commission_max_change_rate> \
  --pubkey <consensus_pubkey> \
  --name <key_name>
```

**NOTE:**  If you would like to override the memo field use the `--ip` and `--node-id` flags for the `fury gentx` command above. `pubkey` can be obtained using `fury tendermint show-validator`

5. Copy the gentx you created to a new directory in the `gentx` directory of this repo and name it `gentx.json`

```sh
cp $HOME/.fury/config/gentx/gentx-<node_id>.json ./gentx/<your-directory>/gentx.json
```

6. Submit your directory, with `gentx.json` file, as a PR on this repo.


## Helpful resources

For resources that references `gaiacli`, you can replace `gaiacli` with `fucli`

* [Using the Cosmos app and a Ledger device to store your Fury keys](https://cosmos.network/docs/cosmos-hub/delegator-guide-cli.html#cosmos-accounts)
* [Validator Security](https://cosmos.network/docs/cosmos-hub/validators/security.html#validator-security)
* [Creating a validator after mainnet launch](https://cosmos.network/docs/cosmos-hub/validators/validator-setup.html#create-your-validator)


## Conclusion

See you all at launch! [Join the discord!](https://discord.com/invite/kQzh3Uv)

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

This work, ["Blackfury Genesis Validators
Guide"](https://github.com/Four4Two/mainnet-gentx/blob/gentx/README.md),
is a derivative of ["Agoric Validator
Guide"](https://github.com/Agoric/agoric-sdk/wiki/Validator-Guide) used
under [CC BY](http://creativecommons.org/licenses/by/4.0/)."Fury Validator
Guide" is licensed under [CC BY](http://creativecommons.org/licenses/by/4.0/) by [Blackfury](https://fury.black/). It was extensively modified to be relevant
to the Blackfury Chain.
