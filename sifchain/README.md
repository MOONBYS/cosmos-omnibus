# Sifchain

| | |
|---|---|
|Version|`v1.1.0-beta`|
|Binary|`sifnoded`|
|Directory|`.sifnoded`|
|ENV namespace|`SIFNODED`|
|Repository|`https://github.com/Sifchain/sifnode`|
|Image|`ghcr.io/ovrclk/cosmos-omnibus:v0.3.20-sifchain-v1.1.0-beta`|

## Examples

- Run on Akash with the [example deploy.yml](./deploy.yml)
- Run locally using the [example docker-compose.yml](./docker-compose.yml)

## Chain information

The [Cosmos Chain Registry](https://github.com/cosmos/chain-registry) publishes up to date chain info for Sifchain.

|Variable|Value|
|---|---|
|`CHAIN_JSON`|`https://raw.githubusercontent.com/cosmos/chain-registry/master/sifchain/chain.json`|

## Polkachu Chain Services

[Polkachu's Chain Services](https://www.polkachu.com/) make bootstrapping a node extremely easy. They provide live peers, statesync and pruned snapshots.

Note you should choose between statesync and snapshot bootstrapping, snapshot will take precedence.

|Variable|Value|
|---|---|
|`P2P_POLKACHU`|`1`|
|`SNAPSHOT_POLKACHU`|`1`|
|`STATESYNC_POLKACHU`|`1`|
