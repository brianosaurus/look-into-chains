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

This is where the bech32ibc, authz, and txFees modules were added to the module list
```go
 ModuleBasics = module.NewBasicManager(
    auth.AppModuleBasic{},
    genutil.AppModuleBasic{},
    bank.AppModuleBasic{},
    capability.AppModuleBasic{},
    staking.AppModuleBasic{},
    mint.AppModuleBasic{},
    distr.AppModuleBasic{},
    gov.NewAppModuleBasic(
      paramsclient.ProposalHandler, distrclient.ProposalHandler, upgradeclient.ProposalHandler, upgradeclient.CancelProposalHandler,
      poolincentivesclient.UpdatePoolIncentivesHandler,
      ibcclientclient.UpdateClientProposalHandler, ibcclientclient.UpgradeProposalHandler,
    ),
    params.AppModuleBasic{},
    crisis.AppModuleBasic{},
    slashing.AppModuleBasic{},
    authzmodule.AppModuleBasic{},
    ibc.AppModuleBasic{},
    upgrade.AppModuleBasic{},
    evidence.AppModuleBasic{},
    transfer.AppModuleBasic{},
    vesting.AppModuleBasic{},
    gamm.AppModuleBasic{},
    txfees.AppModuleBasic{},
    incentives.AppModuleBasic{},
    lockup.AppModuleBasic{},
    poolincentives.AppModuleBasic{},
    epochs.AppModuleBasic{},
    claim.AppModuleBasic{},
    bech32ibc.AppModuleBasic{},
  )
  ```

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
