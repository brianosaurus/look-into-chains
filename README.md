# challenge3

## Gaia Last Upgradee 
1. What was the upgrade

v7-Theta - SDK bump v0.45.1, ibc-go bump v3.0.0, ads interchain accounts, liquidity module bump to 1.5.0, bump packet-forward-middleware to v2.1.1.

Binary Version: gaia-7.0.0

2. Metadata changes from the upgrade

cosmoshub-3 -> cosmoshub-4

3. Code changes and why they were required for the upgrade
4. How was the upgrade performed

cosmovisor for users ... If someone is running a validator cosmovisor downloads the binary but doesn't automatically restart the node (depending whether or not someone configures this).

## Gaia Penultimate Upgrade
1. What was the upgrade

vega - bump cosmos-sdk to v0.44.3, adds authz module, adds feegrand module, adds IBC v2.0.0, adds packet-forward-middleware v1.0.1, 
bumps the liquidity module to v1.4.2

Binary Version gaia v6.0.0

2. Metadata changes from the upgrade

cosmoshub-2 -> cosmoshub-3

3. Code changes and why they were required for the upgrade
4. How was the upgrade performed

cosmovisor for users ... If someone is running a validator cosmovisor downloads the binary but doesn't automatically restart the node (depending whether or not someone configures this).

## Gaia Antipenultimate Upgrade
1. What was the upgrade

Governance Proposal for Gravity DEX Adoption to the Cosmos Hub - Gravity DEX, proposal 38.

Binary Version +sha3: v5.0.0-4760cf1f1266accec7a107f440d46d9724c6fd08

2. Metadata changes from the upgrade

cosmoshub-1 -> cosmoshub-2

3. Code changes and why they were required for the upgrade
4. How was the upgrade performed

This is a manual upgrade that includes features to interact with the Gravity DEX. Operators would download either a new binary or source code based upon the tag and build the binary.

## Osmosis Last Upgrade

<<<<<<< Updated upstream
Osmosis v4 Berylium Upgrade - Stability improvements, cheaper TXs, faster computation within nodes, implements prop 12.

Binary 4.0.0

2. Metadata changes from the upgrade
3. Code changes and why they were required for the upgrade
4. How was the upgrade performed

cosmovisor for users ... If someone is running a validator cosmovisor downloads the binary but doesn't automatically restart the node (depending whether or not someone configures this).

## Osmosis Penultimate Upgrade
=======
>>>>>>> Stashed changes
1. What was the upgrade

Osmosis v5 Boron Upgrade - Enables OFAC, Bech23IBC, Clawback, Whitelists tokens available in the front end, SDK 0.44, implements Gaia Hub proposal 32.

Binary 5.0.0

<<<<<<< Updated upstream
2. Metadata changes from the upgrade
=======
2. MetaData changes from the upgrade

>>>>>>> Stashed changes
3. Code changes and why they were required for the upgrade
4. How was the upgrade performed

cosmovisor for users ... If someone is running a validator cosmovisor downloads the binary but doesn't automatically restart the node (depending whether or not someone configures this).

<<<<<<< Updated upstream
## Osmosis Antipenultimate Upgrade
=======
## Osmosis Penultimate Upgrade

1. What was the upgrade

Osmosis v4 Berylium Upgrade - Stability improvements, cheaper TXs, faster computation within nodes, implements prop 12.

Binary 4.0.0

2. MetaData changes from the upgrade

pool_creation_fee was set

```json
"gamm": {
      "pools": [],
      "next_pool_number": "1",
      "params": {
        "pool_creation_fee": [
          {
            "denom": "uosmo",
            "amount": "1000000000"
          }
        ]
      }
    }
```

3. Code changes and why they were required for the upgrade

  This is in app/upgrades/v4/upgrades.go. It changes metadata for the gamm module.

```go
keepers.GAMMKeeper.SetParams(ctx, gammtypes.NewParams(sdk.Coins{sdk.NewInt64Coin("uosmo", 1)})) // 1 uOSMO
```

  There is a call to Prop12 in the v4 upgrade handler but that changes data, not metadata

4. How was the upgrade performed

cosmovisor for users ... If someone is running a validator cosmovisor downloads the binary but doesn't automatically restart the node (depending whether or not someone configures this).

## Osmosis Anti-Penultimate Upgrade

>>>>>>> Stashed changes
There isn't one. There are only two software upgrade proposals for osmosis mainnet. I'm speficially looking at "/osmos-sdk/SoftwareUpgradeProposal" types that are successful.
