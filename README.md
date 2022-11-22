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

vega - bump cosmos-sdk to v0.44.6, adds authz module, adds feegrant module, makes IBC v2.0.3 standalone module, adds packet-forward-middleware v1.0.1, 

Binary Version gaia v6.0.2

2. Metadata changes from the upgrade

```json
    "authz": {
      "authorization": []
    },
```

```json
    "feegrant": {
      "allowances": []
    },
```

There is only a params change for IBC
```json
    "ibc": {
      ... things from before ...
      "connection_genesis": {
        ... things from before ...
        "params": { 
          "max_expected_time_per_block": "30000000000" 
        }
      },
    },
  ```

3. Code changes and why they were required for the upgrade

This runs the IntGenesis method for eac module that is deleted from fromVM as well as auth which gets run last due to a bug.

```go
app.UpgradeKeeper.SetUpgradeHandler(
		upgradeName,
		func(ctx sdk.Context, _ upgradetypes.Plan, _ module.VersionMap) (module.VersionMap, error) {
			app.IBCKeeper.ConnectionKeeper.SetParams(ctx, ibcconnectiontypes.DefaultParams())

			fromVM := make(map[string]uint64)
			for moduleName := range app.mm.Modules {
				fromVM[moduleName] = 1
			}
			// delete new modules from the map, for _new_ modules as to not skip InitGenesis
			delete(fromVM, authz.ModuleName)
			delete(fromVM, feegrant.ModuleName)
			delete(fromVM, routertypes.ModuleName)

			// make fromVM[authtypes.ModuleName] = 2 to skip the first RunMigrations for auth (because from version 2 to migration version 2 will not migrate)
			fromVM[authtypes.ModuleName] = 2

			// the first RunMigrations, which will migrate all the old modules except auth module
			newVM, err := app.mm.RunMigrations(ctx, app.configurator, fromVM)
			if err != nil {
				return nil, err
			}
			// now update auth version back to 1, to make the second RunMigrations includes only auth
			newVM[authtypes.ModuleName] = 1

			// RunMigrations twice is just a way to make auth module's migrates after staking
			return app.mm.RunMigrations(ctx, app.configurator, newVM)
		},
)
```

This is the exact code from [github.com/cosmos/ibc-go/v2@v2.0.3/modules/core/03-connection/types/params.go](https://github.com/cosmos/ibc-go/blob/main/modules/core/03-connection/types/params.go) that sets ibc.params.max_expected_time_per_block
```go
func (p *Params) ParamSetPairs() paramtypes.ParamSetPairs {
  return paramtypes.ParamSetPairs{
    paramtypes.NewParamSetPair(KeyMaxExpectedTimePerBlock, p.MaxExpectedTimePerBlock, validateParams),
  }
}
```



4. How was the upgrade performed

Look [here](https://github.com/cosmos/gaia/blob/main/docs/migration/cosmoshub-4-vega-upgrade.md#vega-upgrade-steps) for upgrade instructions. For most it is done by using cosmovisor.

## Gaia Antipenultimate Upgrade
1. What was the upgrade

Governance Proposal for Gravity DEX Adoption to the Cosmos Hub - Gravity DEX, proposal 38.

Binary Version 4.0.2

### From the release notes
  Some of the biggest changes to take note on when upgrading as a developer or client are the the following:

* Protocol Buffers: Initially the Cosmos SDK used Amino codecs for nearly all encoding and decoding. In this version a major upgrade to Protocol Buffers have been integrated. It is expected that with Protocol Buffers applications gain in speed, readability, convinience and interoperability with many programming languages. Read more
* CLI: The CLI and the daemon for a blockchain were seperated in previous versions of the Cosmos SDK. This led to a gaiad and gaiacli binary which were seperated and could be used for different interactions with the blockchain. Both of these have been merged into one gaiad which now supports the commands the gaiacli previously supported.
* Node Configuration: Previously blockchain data and node configuration was stored in ~/.gaia/, these will now reside in ~/.gaia/, if you use scripts that make use of the configuration or blockchain data, make sure to update the path.

### The upgrade adds the following modules
* capability
* ibc
* upgrade
* evidence
* transfer
* vesting

### The upgrade changes
* gov

2. Metadata changes from the upgrade

I couldn't get v4.0.2 or any v4 to compile likely due to a newer version of XCode. Therefore, I had to extrapolate from a v5.0.0 genesis.json for module changes to metadata.

In any case, these are the additions to the metadata (genesis.json)

```json
    "ibc": {
      "client_genesis": {
        "clients": [],
        "clients_consensus": [],
        "clients_metadata": [],
        "params": {
          "allowed_clients": [
            "06-solomachine",
            "07-tendermint"
          ]
        },
        "create_localhost": false,
        "next_client_sequence": "0"
      },
      "connection_genesis": {
        "connections": [],
        "client_connection_paths": [],
        "next_connection_sequence": "0"
      },
      "channel_genesis": {
        "channels": [],
        "acknowledgements": [],
        "commitments": [],
        "receipts": [],
        "send_sequences": [],
        "recv_sequences": [],
        "ack_sequences": [],
        "next_channel_sequence": "0"
      }
    }
```

```json
   "capability": {
      "index": "1",
      "owners": []
    },
```

```json
    "upgrade": {},
    "vesting": {}
```

```json
    "evidence": {
      "max_age_num_blocks": "100000",
      "max_age_duration": "172800000000000",
      "max_bytes": "1048576"
    },
```

```json
    "transfer": {
      "port_id": "transfer",
      "denom_traces": [],
      "params": {
        "send_enabled": true,
        "receive_enabled": true
      }
    },
```

cosmoshub-3 -> cosmoshub-4

3. Code changes and why they were required for the upgrade

See [app.go v4.0.2](https://github.com/cosmos/gaia/blob/v4.0.2/app/app.go) for details on where the liquidity module was added as a module and keeper. 

There is no upgrade handler for this release. The modules automatically run their own InitGenesis routines.

4. How was the upgrade performed

The upgrade proceedures are discussed [here](https://github.com/cosmos/gaia/blob/main/docs/migration/cosmoshub-3.md#upgrade-procedure)

## Osmosis Last Upgrade

1. What was the upgrade

Osmosis v5 Boron Upgrade - Enables OFAC, Bech23IBC, Clawback, Whitelists tokens available in the front end, SDK 0.44, implements Gaia Hub proposal 32.

Binary 5.0.0

* Upgrade Cosmos-SDK to SDK v0.44 from SDK v0.42 For a full list of updates in Cosmos-SDK v0.44.3 please see its changelog.

### New modules:
* Authz - allows granting arbitrary privileges from one account (the granter) to another account (the grantee). Authorizations must be granted for a particular Msg service method one by one using an implementation of the Authorization interface.
* Bech32IBC - Allows auto-routing of send msgs to addresses on other chains, once configured by governance. Allows you to do a bank send on Osmosis to a cosmos1... address, and it automatically gets IBC'd there.
* TxFees - Enables validators to easily accept txfees in multiple assets
* Implements Proposal 32 - Clawback of unclaimed uosmo and uion on airdrop end date. (December 15th, 5PM UTC)
* Upgrade IBC from a standalone module in the SDK to IBC v2. This improves the utility of Ethereum Bridges and Cosmwasm bridges.
* Blocking OFAC banned ETH addresses

2. Metadata changes from the upgrade

    max_expected_time_per_block was changed to 30 seconds

```json
 "connection_genesis": {
    "connections": [],
    "client_connection_paths": [],
    "next_connection_sequence": "0",
    "params": {
      "max_expected_time_per_block": "30000000000"
    }
```
    All modules have been upgraded to their most recent version auth and bech32Ibc have had their genesis routines run. I cannot see any changes from the auth InitGenesis. 

```json
"txfees": {
  "basedenom": "stake",
  "feetokens": []
},
```
    The bond_denom was set to 'stake'
    feetokens was set to an empty array 

```json
"bech32ibc": {
  "nativeHRP": "osmo",
  "hrpIBCRecords": []
},
```
    This was added when the bech32-ibc module had its InitGenesis run

```json
"authz": {
  "authorization": []
},
```
  This was added when the authz InitGenesis method was run


3. Code changes and why they were required for the upgrade

```go
    keepers.IBCKeeper.ConnectionKeeper.SetParams(ctx, ibcconnectiontypes.DefaultParams())
```
    See: https://github.com/cosmos/ibc-go/blob/main/docs/migrations/ibc-migration-043.md#in-place-store-migrations		

```go
    ctx.Logger().Info("Setting txfees module genesis with actual v5 desired genesis")
		feeTokens := InitialWhitelistedFeetokens(ctx, keepers.GAMMKeeper)
		keepers.TxFeesKeeper.InitGenesis(ctx, txfeestypes.GenesisState{
			Basedenom: keepers.StakingKeeper.BondDenom(ctx),
			Feetokens: feeTokens,
		})
```
    New desired values is the reason. Chain policy

    This code ensures the auth module is the last one to change its version
    See: https://github.com/cosmos/cosmos-sdk/issues/10591
```go
for moduleName := range mm.Modules {
  fromVM[moduleName] = 1
}

// EXCEPT Auth needs to run AFTER staking.
//
// See: https://github.com/cosmos/cosmos-sdk/issues/10591
//
// So we do this by making auth run last. This is done by setting auth's
// consensus version to 2, running RunMigrations, then setting it back to 1,
// and then running migrations again.
fromVM[authtypes.ModuleName] = 2

// Override versions for authz & bech32ibctypes module as to not skip their
// InitGenesis for txfees module, we will override txfees ourselves.
delete(fromVM, authz.ModuleName)
delete(fromVM, bech32ibctypes.ModuleName)

newVM, err := mm.RunMigrations(ctx, configurator, fromVM)
if err != nil {
  return nil, err
}

// now update auth version back to v1, to run auth migration last
newVM[authtypes.ModuleName] = 1
  
return mm.RunMigrations(ctx, configurator, newVM)
```

  See [app.go v5.0.0](https://github.com/osmosis-labs/osmosis/blob/v5.0.0/app/app.go) for details on where the bech32ibc, authz, and txFees modules was added as modules and keepers. 

  go.mod contains the dependencies for these modules. 

4. How was the upgrade performed

cosmovisor for users ... If someone is running a validator cosmovisor downloads the binary but doesn't automatically restart the node (depending whether or not someone configures this).

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

## Osmosis Antipenultimate Upgrade

There isn't one. There are only two software upgrade proposals for osmosis mainnet. I'm speficially looking at "/osmos-sdk/SoftwareUpgradeProposal" types that are successful.
