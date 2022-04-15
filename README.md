# COSMOS OMNIBUS - Run Cosmos Nodes on Akash

This is a meta package of [cosmos-sdk](//github.com/cosmos-cosmos-sdk)-based
docker images and configuration meant to make deploying onto [Akash](//github.com/ovrclk/akash)
easy and standardized across cosmos.

The goal is to have a simple way to launch any cosmos chain, with a variety of different bootstrapping options

1. ["normal" boostrap](#shortcuts) - using `fastsync`.
1. [Hand-made snapshots](#snapshot-restore) a la [cosmos-snapshots](https://github.com/c29r3/cosmos-snapshots)
1. [The new `state-sync` mechanism](#statesync).

Configuration is achieved using environment variables, with shortcuts available for common setups. Every aspect of the node configuration can be achieved in this way.

Additional features are included to make running a node as simple as possible

1. [Chain configuration can be sourced from a remote JSON file](#chain-configuration)
1. [Genesis file can be downloaded and unzipped in various ways](#chain-configuration)
1. [Private keys can be backed up and restored](#private-key-backuprestore) from any S3 compatible storage provider, such as Sia or Storj via [Filebase](https://filebase.com/).
1. [Snapshots of a nodes data directory](#snapshot-backup) can be created at a certain time or day and uploaded to an S3 storage provider

## Generic image (binary downloaded at runtime)

Omnibus has a generic base image which can download the required binary at runtime. This is useful for chain upgrades, testnets, or using a different version than Omnibus primarily supports.

This generic image provides the Omnibus scripts and configuration helpers, and nothing else. Set the `BINARY_URL` environment variable to a `.zip`, `.tar` or `.tar.gz` URL, and configure `PROJECT`, `PROJECT_DIR` and `PROJECT_BIN`. Alternatively provide a [Chain Registry](https://github.com/cosmos/chain-registry) `CHAIN_JSON` to configure everything automatically (where data is available).

Image URL: `ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-generic`

```yaml
services:
  node:
    image: ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-generic
    env:
      - MONIKER=my-moniker-1
      - CHAIN_JSON=https://raw.githubusercontent.com/ovrclk/net/master/edgenet/meta.json
      - BINARY_ZIP_PATH=akash_0.15.0-rc14_linux_amd64/akash
```

More information on the generic image can be found at [/generic](./generic/), and configuration is detailed in depth below.

## Networks (pre-built images)

The available docker images can be found [here](https://github.com/orgs/ovrclk/packages/container/package/cosmos-omnibus).  They are
tagged with the form `$COSMOS_OMNIBUS_VERSION-$PROJECT-$PROJECT_VERSION`.

|Project|Version|Image| |
|---|---|---|---|
|[akash](https://github.com/ovrclk/akash)|`v0.14.1`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-akash-v0.14.1`|[Example](./akash)|
|[agoric](https://github.com/Agoric/ag0)|`agoric-3.1`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-agoric-agoric-3.1`|[Example](./agoric)|
|[bandchain](https://github.com/bandprotocol/chain)|`v2.3.3`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-bandchain-v2.3.3`|[Example](./bandchain)|
|[bitcanna](https://github.com/BitCannaGlobal/bcna)|`v.1.3.1`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-bitcanna-v.1.3.1`|[Example](./bitcanna)|
|[bitsong](https://github.com/bitsongofficial/go-bitsong)|`v0.10.0`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-bitsong-v0.10.0`|[Example](./bitsong)|
|[bostrom](https://github.com/cybercongress/go-cyber)|`v0.2.0`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-bostrom-v0.2.0`|[Example](./bostrom)|
|[cerberus](https://github.com/cerberus-zone/cerberus)|`v1.0.1`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-cerberus-v1.0.1`|[Example](./cerberus)|
|[cheqd](https://github.com/cheqd/cheqd-node)|`v0.5.0`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-cheqd-v0.5.0`|[Example](./cheqd)|
|[chihuahua](https://github.com/ChihuahuaChain/chihuahua)|`v1.1.1`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-chihuahua-v1.1.1`|[Example](./chihuahua)|
|[comdex](https://github.com/comdex-official/comdex)|`v0.1.1`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-comdex-v0.1.1`|[Example](./comdex)|
|[cosmoshub](https://github.com/cosmos/gaia)|`v7.0.1`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-cosmoshub-v7.0.1`|[Example](./cosmoshub)|
|[cronos](https://github.com/crypto-org-chain/cronos)|`v0.6.5`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-cronos-v0.6.5`|[Example](./cronos)|
|[cryptoorgchain](https://github.com/crypto-org-chain/chain-main)|`v3.1.1-croeseid`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-cryptoorgchain-v3.1.1-croeseid`|[Example](./cryptoorgchain)|
|[decentr](https://github.com/Decentr-net/decentr)|`v1.5.7`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-decentr-v1.5.7`|[Example](./decentr)|
|[desmos](https://github.com/desmos-labs/desmos)|`v2.3.1`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-desmos-v2.3.1`|[Example](./desmos)|
|[dig](https://github.com/notional-labs/dig)|`v1.1.0`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-dig-v1.1.0`|[Example](./dig)|
|[emoney](https://github.com/e-money/em-ledger)|`v1.1.4`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-emoney-v1.1.4`|[Example](./emoney)|
|[fetchhub](https://github.com/fetchai/fetchd)|`v0.9.1`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-fetchhub-v0.9.1`|[Example](./fetchhub)|
|[gravitybridge](https://github.com/Gravity-Bridge/Gravity-Bridge)|`v1.4.1`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-gravitybridge-v1.4.1`|[Example](./gravitybridge)|
|[impacthub](https://github.com/ixofoundation/ixo-blockchain)|`v0.17.0`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-impacthub-v0.17.0`|[Example](./impacthub)|
|[irisnet](https://github.com/irisnet/irishub)|`v1.0.1`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-irisnet-v1.0.1`|[Example](./irisnet)|
|[juno](https://github.com/CosmosContracts/Juno)|`v2.1.0`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-juno-v2.1.0`|[Example](./juno)|
|[kava](https://github.com/Kava-Labs/kava)|`v0.16.1`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-kava-v0.16.1`|[Example](./kava)|
|[kixchain](https://github.com/KiFoundation/ki-tools)|`2.0.0`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-kichain-2.0.0`|[Example](./kichain)|
|[konstellation](https://github.com/konstellation/konstellation)|`0.5.0`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-konstellation-0.5.0`|[Example](./konstellation)|
|[likecoin](https://github.com/likecoin/likecoin-chain)|`fotan-1.2`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-likecoin-fotan-1.2`|[Example](./likecoin)|
|[lumnetwork](https://github.com/lum-network/chain)|`v1.0.5`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-lumnetwork-v1.0.5`|[Example](./lumnetwork)|
|[osmosis](https://github.com/osmosis-labs/osmosis)|`v7.0.2`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-osmosis-v7.0.2`|[Example](./osmosis)|
|[panacea](https://github.com/medibloc/panacea-core)|`v2.0.2`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-panacea-v2.0.2`|[Example](./panacea)|
|[persistence](https://github.com/persistenceOne/persistenceCore)|`v0.1.3`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-persistence-v0.1.3`|[Example](./persistence)|
|[regen](https://github.com/regen-network/regen-ledger)|`v2.1.0`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-regen-v3.0.0`|[Example](./regen)|
|[rizon](https://github.com/rizon-world/rizon)|`v0.2.8`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-rizon-v0.2.8`|[Example](./rizon)|
|[sentinel](https://github.com/sentinel-official/hub)|`v0.8.3`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-sentinel-v0.8.3`|[Example](./sentinel)|
|[shentu](https://github.com/certikfoundation/shentu)|`v2.2.0`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-shentu-v2.2.0`|[Example](./shentu)|
|[sifchain](https://github.com/Sifchain/sifnode)|`v0.10.1`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-sifchain-v0.10.1`|[Example](./sifchain)|
|[sommelier](https://github.com/PeggyJV/sommelier)|`v3.1.0`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-sommelier-v3.1.0`|[Example](./sommelier)|
|[stargaze](https://github.com/public-awesome/stargaze)|`v1.1.2`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-stargaze-v1.1.2`|[Example](./stargaze)|
|[starname](https://github.com/iov-one/starnamed)|`v0.10.18`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-starname-v0.10.18`|[Example](./starname)|
|[terra](https://github.com/terra-money/core)|`v0.5.9`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-terra-v0.5.9`|[Example](./terra)|
|[thorchain](https://gitlab.com/thorchain/thornode)|`chaosnet-multichain`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-thorchain-chaosnet-multichain`|[Example](./thorchain)|
|[umee](https://github.com/umee-network/umee)|`v1.0.1`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-umee-v1.0.1`|[Example](./umee)|
|[vidulum](https://github.com/vidulum/mainnet)|`v1.0.0`|`ghcr.io/ovrclk/cosmos-omnibus:v0.0.30-vidulum-v1.0.0`|[Example](./vidulum)|

## Running on Akash

See the `deploy.yml` example file in each chain directory which details the minimum configuration required. Use the [configuration options below](#configuration) to add functionality.

## Running locally/any docker host

See the `docker-compose.yml` example file in each chain directory to run each node using `docker-compose up`.

## Snaphots

Akash publish snapshots for the following chains, taken at 12AM UTC every day.

These snapshots are created using Omnibus nodes running on Akash, as shown in the [Snapshot Backup](_examples/snapshot_backup) example.

|Chain|Snapshot JSON|
|---|---|
|`akashnet-2`|https://cosmos-snapshots.s3.filebase.com/akash/snapshot.json|
|`bitcanna-1`|https://cosmos-snapshots.s3.filebase.com/bitcanna/snapshot.json|
|`comdex-1`|https://cosmos-snapshots.s3.filebase.com/comdex/snapshot.json|
|`desmos-mainnet`|https://cosmos-snapshots.s3.filebase.com/desmos/snapshot.json|
|`gravity-bridge-2`|https://cosmos-snapshots.s3.filebase.com/gravitybridge/snapshot.json|
|`juno-1`|https://cosmos-snapshots.s3.filebase.com/juno/snapshot.json|
|`osmosis-1`|https://cosmos-snapshots.s3.filebase.com/osmosis/snapshot.json|
|`sentinelhub-2`|https://cosmos-snapshots.s3.filebase.com/sentinel/snapshot.json|
|`sifchain-1`|https://cosmos-snapshots.s3.filebase.com/sifchain/snapshot.json|

## Examples

See the [_examples](./_examples) directory for some common setups, including:

- [Statesync](./_examples/statesync)
- [Load Balanced RPC Nodes](./_examples/load-balanced-rpc-nodes)
- [Validator and Public Sentries](./_examples/validator-and-public-sentries)
- [Validator with Private Sentries](./_examples/validator-and-private-sentries)
- [Snapshot Backup](./_examples/snapshot_backup)

## Configuration

Cosmos blockchains can be configured entirely using environment variables instead of the config files.
Every chain has it's own prefix, but the format of the configuration is the same.

For example to configure the `seeds` option in the `p2p` section of `config.toml`, for the Akash blockchain:

```
AKASH_P2P_SEEDS=id@node:26656
```

The namespace for each of the supported chains in the cosmos omnibus can be found in the `README` in each project directory.

The omnibus images allow some specific variables and shortcuts to configure extra functionality, detailed below.

### Chain configuration

Chain config can be sourced from a `chain.json` file [as seen in the Cosmos Chain Registry](https://github.com/cosmos/chain-registry).

|Variable|Description|Default|Examples|
|---|---|---|---|
|`CHAIN_JSON`|URL to a `chain.json` file detailing the chain meta| |`https://github.com/cosmos/chain-registry/blob/master/akash/chain.json`
|`CHAIN_ID`|The cosmos chain ID| |`akashnet-2`
|`GENESIS_URL`|URL to the genesis file in `.gz`, `.tar.gz`, or `.zip` format. Can be set by CHAIN_JSON| |`https://raw.githubusercontent.com/ovrclk/net/master/mainnet/genesis.json`
|`DOWNLOAD_GENESIS`|Force download of genesis file. If unset the node will only download if the genesis file is missing| |`1`|
|`VALIDATE_GENESIS`|Set to 0 to disable validation of genesis file|`1`|`0`

### P2P

See [Cosmos docs](https://docs.tendermint.com/master/nodes/configuration.html#p2p-settings) for more information. This can be sourced from a `CHAIN_JSON` URL.

|Variable|Description|Default|Examples|
|---|---|---|---|
|`P2P_SEEDS`|Seed nodes. Can be set by CHAIN_JSON or GENESIS_URL| |`id@node:26656`|
|`P2P_PERSISTENT_PEERS`|Persistent peers. Can be set by CHAIN_JSON or GENESIS_URL| |`id@node:26656`|

### Private key backup/restore

On boot, the container can restore a private key from any S3 storage provider. If this is configured and the key doesn't exist in S3 yet, it will be uploaded from the node.

This allows for a persistent node ID and validator key on Akash's ephemeral storage.

[Filebase](https://filebase.com/) is a great way to use S3 with decentralised hosting such as Sia.

The `node_key.json` and `priv_validator_key.json` are both backed up, and can be encrypted with a password.

|Variable|Description|Default|Examples|
|---|---|---|---|
|`S3_KEY`|S3 access key| | |
|`S3_SECRET`|S3 secret key| | |
|`S3_HOST`|The S3 API host|`https://s3.filebase.com`|`https://s3.us-east-1.amazonaws.com`|
|`KEY_PATH`|Bucket and directory to backup/restore to| |`bucket/nodes/node_1`|
|`KEY_PASSWORD`|An optional password to encrypt your private keys. Shouldn't be optional| | |

### Statesync

Some shortcuts for enabling statesync. Statesync requires 2x nodes with snapshots enabled.

[See an example](_examples/statesync) of a statesync deployment.

|Variable|Description|Default|Examples|
|---|---|---|---|
|`STATESYNC_SNAPSHOT_INTERVAL`|Take a snapshot to provide statesync every X blocks| |`500`|
|`STATESYNC_ENABLE`|Enabling statesyncing from a node. Default `true` if `STATESYNC_RPC_SERVERS` is set| | |
|`STATESYNC_RPC_SERVERS`|Comma separated list of RPC nodes with snapshots enabled| |`ip:26657,ip2:26657`|
|`STATESYNC_TRUSTED_NODE`|A trusted node to obtain trust height and hash from. Defaults to the first `STATESYNC_RPC_SERVERS` if set| |`ip:26657`|
|`STATESYNC_TRUST_PERIOD`|Trust period for the statesync snapshot|`168h0m0s`| |
|`STATESYNC_TRUST_HEIGHT`|Obtained from `STATESYNC_TRUSTED_NODE`| | |
|`STATESYNC_TRUST_HASH`|Obtained from `STATESYNC_TRUSTED_NODE`| | |

### Snapshot restore

The node `data` directory can be restored from a `.tar` or `.tar.gz` file stored on a public URL.
This can be from a specific URL, a [snapshot.json](#snapshot-backup), or from a base URL and matching a given pattern.

|Variable|Description|Default|Examples|
|---|---|---|---|
|`SNAPSHOT_URL`|A URL to a `.tar` or `.tar.gz` file| |`http://135.181.60.250/akash/akashnet-2_2021-06-16.tar`|
|`SNAPSHOT_JSON`|A URL to a `snapshot.json` as detailed in [Snapshot backup](#snapshot-backup)| |`https://cosmos-snapshots.s3.filebase.com/akash/snapshot.json`|
|`SNAPSHOT_FORMAT`|The format of the snapshot file|`tar.gz`|`tar`|
|`SNAPSHOT_BASE_URL`|A base URL to a directory containing backup files| |`http://135.181.60.250/akash`|
|`SNAPSHOT_PATTERN`|The pattern of the file in the `BASE_URL`|`$CHAIN_ID.*$SNAPSHOT_FORMAT`|`foobar.*tar.gz`|
|`DOWNLOAD_SNAPSHOT`|Force bootstrapping from snapshot. If unset the node will only restore a snapshot if the `data` directory is missing| |`1`|

### Snapshot backup

Omnibus includes a script to automatically snapshot a node and upload the resulting archive to any S3 compatible service like [Filebase](https://filebase.com/).
At a specified time (or day), the script will shut down the tendermint server, create an archive of the `data` directory and upload it.
Snapshots older than a specified time can also be deleted. Finally a JSON metadata file is created listing the current snapshots. The server is then restarted and monitored.

[See an example](_examples/snapshot_backup) of a snapshot node deployment.

|Variable|Description|Default|Examples|
|---|---|---|---|
|`S3_KEY`|S3 access key| | |
|`S3_SECRET`|S3 secret key| | |
|`S3_HOST`|The S3 API host|`https://s3.filebase.com`|`s3.us-east-1.amazonaws.com`|
|`SNAPSHOT_PATH`|The S3 path to upload snapshots to, including the bucket| |`cosmos-snapshots/akash`|
|`SNAPSHOT_PREFIX`|The prefix for the snapshot filename|`$CHAIN_ID`|`snapshot`|
|`SNAPSHOT_TIME`|The time the snapshot will run|`00:00:00`|`09:00:00`|
|`SNAPSHOT_DAY`|The numeric day of the week the snapshot will run (Monday = 1)|`*`|`7`|
|`SNAPSHOT_SIZE`|The rough size of the resulting snapshot for the multi-part upload|`107374182400`|`0`|
|`SNAPSHOT_DIR`|The directory on disk to snapshot|`$PROJECT_ROOT/data`|`/root/.akash`|
|`SNAPSHOT_CMD`|The command to run the server|`$START_CMD`|`akash start --someflag`|
|`SNAPSHOT_RETAIN`|How long to retain snapshots for (0 to disable)|`2 days`|`1 week`|
|`SNAPSHOT_METADATA`|Whether to create a snapshot.json metadata file|`1`|`0`|
|`SNAPSHOT_METADATA_URL`|The URL snapshots will be served from (for snapshot.json)| |`https://cosmos-snapshots.s3.filebase.com/akash`|

### Binary download

The node binary can be downloaded at runtime when using the [Generic image](#generic-image-binary-downloaded-at-runtime). All configuration can be sourced from `CHAIN_JSON` if the attributes are available, or configured manually. You will need to set `PROJECT`, `PROJECT_BIN` and `PROJECT_DIR` if these can't be sourced from `CHAIN_JSON`.

|Variable|Description|Default|Examples|
|---|---|---|---|
|`BINARY_URL`|URL to the binary (or `zip`, `tar`, `tar.gz`)| | |
|`BINARY_ZIP_PATH`|Path to the binary in the archive. Can be left blank if correctly named in root| | |
|`PROJECT`|Name of the project, informs other variables| | |
|`PROJECT_BIN`|Binary name|`$PROJECT`|`osmosisd`|
|`PROJECT_DIR`|Name of project directory|`.$PROJECT_BIN`|`.osmosisd`|

### Shortcuts

See [Cosmos docs](https://docs.tendermint.com/master/nodes/configuration.html) for more information

|Variable|Description|Default|Examples|
|---|---|---|---|
|`FASTSYNC_VERSION`|The fastsync version| |`v2`|
|`MINIMUM_GAS_PRICES`|Minimum gas prices| |`0.025uakt`|
|`PRUNING`|How much of the chain to prune| |`nothing`|
|`DEBUG`|Set to `1` to output all environment variables on boot| |`1`|

## Contributing

Adding a new chain is easy:

- Ideally source or setup a `chain.json` to provide a single source of truth for chain info
- Add a project directory based on the existing projects
- Update the [github workflow](https://github.com/ovrclk/cosmos-omnibus/blob/master/.github/workflows/publish.yaml) to create an image for your chain

Submit a PR or an issue if you want to see any specific chains.
