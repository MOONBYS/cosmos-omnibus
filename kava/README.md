# Kava

| | |
|---|---|
|Version|`v0.18.1`|
|Binary|`kava`|
|Directory|`.kava`|
|ENV namespace|`KA`|
|Repository|`https://github.com/Kava-Labs/kava`|
|Image|`ghcr.io/ovrclk/cosmos-omnibus:v0.3.20-kava-v0.18.1`|

## Examples

- Run on Akash with the [example deploy.yml](./deploy.yml)
- Run locally using the [example docker-compose.yml](./docker-compose.yml)

## Chain information

The [Cosmos Chain Registry](https://github.com/cosmos/chain-registry) publishes up to date chain info for Kava.

|Variable|Value|
|---|---|
|`CHAIN_JSON`|`https://raw.githubusercontent.com/cosmos/chain-registry/master/kava/chain.json`|

## Polkachu Chain Services

[Polkachu's Chain Services](https://www.polkachu.com/) make bootstrapping a node extremely easy. They provide live peers, statesync and pruned snapshots.

Note you should choose between statesync and snapshot bootstrapping, snapshot will take precedence.

|Variable|Value|
|---|---|
|`P2P_POLKACHU`|`1`|
|`SNAPSHOT_POLKACHU`|`1`|
|`STATESYNC_POLKACHU`|`1`|

## ChainLayer Quicksync

ChainLayer provide snapshots for Kava as part of their [Quicksync service](https://quicksync.io/networks/kava.html).

|Variable|Value|
|---|---|
|`SNAPSHOT_QUICKSYNC`|`https://quicksync.io/kava.json`|
|`ADDRBOOK_URL`|`https://quicksync.io/addrbook.kava.json`|
