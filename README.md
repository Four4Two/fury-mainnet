# mainnet-gentx
# Setting Up a Genesis Fury Validator

This guide will provide instructions on setting up a node, submitting a gentx, and other
tasks needed to participate in the launch of the Fury mainnet.

The primary point of communication for the genesis process and future
updates will be the \#validators channel on the [Fury
Discord](https://discord.gg/FAarwSC8Tr). This channel is private by
default in order to keep it free of spam and unnecessary noise. To join
the channel, please send a message to @Meow#6669 to add yourself and any
team members.

Some important notes on joining as a genesis validator:

1. **Gentxs must be submitted by End of Day UTC on June 30.**
2. We highly recommend only experienced validators who have run on past
    Cosmo SDK chains and have participated in a genesis ceremony before
    become genesis validators on Fury.
3. Furthermore, Fury intends to adopt many new custom low-level features such as
    threshold decryption, custom bridges, and price oracles. Some of
    these future upgrades may require validators to run additional
    software beyond the normal node software, and validators should be
    prepared to learn and run these.
4. To be a genesis validator, you must have MER at genesis via the
    fairdrop. Every address that had ATOMs during the Stargate upgrade
    of the Cosmo Hub from `cosmoshub-3` to `cosmoshub-4` will have
    recieve fairdrop MER. You can verify that a Cosmo address has
    received coins in the fairdrop by inputting an address here:
    <https://airdrop.fury.zone/>.

## Hardware

We recommend selecting an all-purpose server with:

- 4 or more physical`<sup>`{=html}\[1\]`</sup>`{=html} CPU cores
- At least 500GB of SSD disk storage
- At least 32GB of memory
- At least 100mbps network bandwidth

As the usage of the blockchain grows, the server requirements may
increase as well, so you should have a plan for updating your server as
well.

`<sup>`{=html}\[1\]`</sup>`{=html}: You'll often see 4 distinct physical
cores as a machine with 8 logical cores due to hyperthreading. The
distinct logical cores are helpful for things that are I/O bound, but
threshold decryption will have validators running significant, non-I/O
bound, computation, hence the need for physical cores. We are not
launching with this parallelism, but we include the requirement as we
expect parallelism in some form to be needed by validators in a
not-so-distant future.

## Instructions

These instructions are written targeting an Ubuntu 20.04 system.
Relevant changes to commands should be made depending on the
OS/architecture you are running on.

### Install Go

Fury is built using Go and requires Go version 1.20 In this
example, we will be installing Go on the above Ubuntu 20.04:

``` {.sh}
# First remove any existing old Go installation
sudo rm -rf /usr/local/go

# Install the latest version of Go using this helpful script 
curl https://raw.githubusercontent.com/canha/golang-tools-install-script/master/goinstall.sh | bash

# Update environment variables to include go
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF
source $HOME/.profile
```

To verify that Go is installed:

``` {.sh}
go version
# Should return go version go1.20.2 linux/amd64
```

### Get Fury Source Code

Use git to retrieve Fury source code from the [official
repo](https://github.com/four4two/fury), and checkout the
`gentx-launch` tag, which contains the latest stable release.

``` {.sh}
git clone https://github.com/four4two/fury
cd fury
git checkout v0.4.1
```

## Install fury

You can now build Fury node software. Running the following command
will install the executable fury (Fury node daemon) to your
GOPATH.

``` {.sh}
make install
```

### Verify Your Installation

Verify that everything is OK. If you get something *like* the following,
you've successfully installed Fury on your system.

``` {.sh}
fury version --long

name: fury
server_name: fury
version: '"0.4.1
go: go version go1.20.2 darwin/amd64
```

If the software version does not match, then please check your `$PATH`
to ensure the correct `fury` is running.

### Save your Chain ID in fury config

We recommend saving the mainnet `chain-id` into your `fury`'s
client.toml. This will make it so you do not have to manually pass in
the chain-id flag for every CLI command.

``` {.sh}
fury config chain-id highbury_710-1
```

### Initialize your Node

Now that your software is installed, you can initialize the directory
for fury.

``` {.sh}
fury init --chain-id=highbury_710-1 <your_moniker>
```

This will create a new `.fury` folder in your HOME directory.

### Download Pregenesis File

You can now download the "pregenesis" file for the chain. This is a
genesis file with the chain-id and airdrop balances.

``` {.sh}
cd $HOME/.fury/config/
curl https://raw.githubusercontent.com/osmosis-labs/networks/main/highbury_710-1/pregenesis.json > $HOME/.fury/config/genesis.json
```

### Import Validator Key

The create a gentx, you will need the private key to an address that
received an allocation in the airdrop.

There are a couple options for how to import a key into `fury`.

You can import such a key into `fury` via a mnemonic or exporting
and importing a keyfile from an existing CLI.

#### Import Via Mnemonic

To import via mnemonic, you can do so using the following command and
then input your mnemonic when prompted.

``` {.sh}
fury keys add <key_name> --recover
```

#### Import From Another CLI

If you have the private key saved in the keystore of another CLI (such
as gaiad), you can easily import it into `fury` using the following
steps.

1. Export the key from an existing keystore. In this example we will
    use gaiad. When prompted, input a password to encrypt the key file
    with.

``` {.sh}
gaiad keys export <original_key_name>
```

2. Copy the output starting from the line that says
    `BEGIN TENDERMINT PRIVATE KEY` and ending with the line that says
    `END TENDERMINT PRIVATE KEY` into a txt file somewhere on your
    machine.
3. Import the key into `fury` using the following command. When
    prompted for a password, use the same password used in step 1 to
    encrypt the keyfile.

``` {.sh}
fury keys import <new_key_name> ./path/to/key.txt 
```

4. Delete the keyfile from your machine.

#### Import via Ledger

To import a key stored on a ledger, the process will be exactly the same
as adding a ledger key to the CLI normally. You can connect a Ledger
device with the Cosmo app open and then run:

``` {.sh}
fury keys add <key_name> --ledger
```

and follow any prompts.

#### Get your Tendermint Validator Pubkey

You must get your validator's consensus pubkey as it will be necessary
to include in the transaction to create your validator.

If you are using Tendermint's native `priv_validator.json` as your
consensus key, you display your validator public key using the following
command

    fury tendermint show-validator

The pubkey should be formatted with the bech32 prefix `furyvalconspub1`.

If you are using a custom signing mechanism such as `tmkms`, please
refer to their relevant docs to retrieve your validator pubkey.

### Create GenTx

Now that you have you key imported, you are able to use it to create
your gentx.

To create the genesis transaction, you will have to choose the following
parameters for your validator:

- moniker
- commission-rate
- commission-max-rate
- commission-max-change-rate
- min-self-delegation (must be \>1)
- website (optional)
- details (optional)
- identity (keybase key hash, this is used to get validator logos in
    block explorers. optional)
- pubkey (gotten in previous step)

Note that your gentx will be rejected if you use an amount greater than
what you have as liquid from the fairdrop. Recall only 20% of your
fairdrop allocation is liquid at genesis. Also, note that Fury has a
chain-mandated minimum commission rate of 5%.

If you would like to override the memo field, use the `--ip` and
`--node-id` flags.

An example genesis command would thus look like:

``` {.sh}
fury gentx <key_name> 1000000ufury \
  --chain-id="highbury_710-1" \
  --moniker=furywhale \
  --website="https://fury.zone" \
  --details="We love the Arsenal" \
  --commission-rate="0.1" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --identity="5B5AB9D8FBBCEDC6" \
  --pubkey="furyvalconspub1zcjduepqnxl4ntf8wjn0275smfll4n4lg9cwcurz2qt6dkhrjzf94up8g4cspyyzn9"
```

It will show an output something similar to:

``` {.sh}
Genesis transaction written to "/Users/ubuntu/.fury/config/gentx/gentx-eb3b1768d00e66ef83acb1eee59e1d3a35cf76fc.json"
```
