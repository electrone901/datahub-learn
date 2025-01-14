---
description: Learn how to build a scavenger game (continued)
---

# Module

## Module <a id="module"></a>

Our `scaffold` tool has done most of the work for us in generating our `module.go` file inside `./x/scavenge/`. One way that our module is different than the simplest form of a module, is that it uses it's own `Keeper` as well as the `Keeper` from the `bank` module. The only real changes needed are under the `AppModule` and `NewAppModule`, where the `bank.Keeper` needs to be added and referenced. The file should look as follows afterwards:

```javascript
package scavenge

import (
    "encoding/json"

    "github.com/gorilla/mux"
    "github.com/spf13/cobra"

    abci "github.com/tendermint/tendermint/abci/types"

    "github.com/cosmos/cosmos-sdk/client/context"
    "github.com/cosmos/cosmos-sdk/codec"
    sdk "github.com/cosmos/cosmos-sdk/types"
    "github.com/cosmos/cosmos-sdk/types/module"
    "github.com/cosmos/cosmos-sdk/x/bank"
    "github.com/cosmos/sdk-tutorials/scavenge/x/scavenge/client/cli"
    "github.com/cosmos/sdk-tutorials/scavenge/x/scavenge/client/rest"
    "github.com/cosmos/sdk-tutorials/scavenge/x/scavenge/types"
)

var (
    _ module.AppModule      = AppModule{}
    _ module.AppModuleBasic = AppModuleBasic{}
)

// AppModuleBasic defines the basic application module used by the scavenge module.
type AppModuleBasic struct{}

var _ module.AppModuleBasic = AppModuleBasic{}

// Name returns the scavenge module's name.
func (AppModuleBasic) Name() string {
    return types.ModuleName
}

// RegisterCodec registers the scavenge module's types for the given codec.
func (AppModuleBasic) RegisterCodec(cdc *codec.Codec) {
    RegisterCodec(cdc)
}

// DefaultGenesis returns default genesis state as raw bytes for the scavenge
// module.
func (AppModuleBasic) DefaultGenesis() json.RawMessage {
    return ModuleCdc.MustMarshalJSON(DefaultGenesisState())
}

// ValidateGenesis performs genesis state validation for the scavenge module.
func (AppModuleBasic) ValidateGenesis(bz json.RawMessage) error {
    var data GenesisState
    err := ModuleCdc.UnmarshalJSON(bz, &data)
    if err != nil {
        return err
    }
    return ValidateGenesis(data)
}

// RegisterRESTRoutes registers the REST routes for the scavenge module.
func (AppModuleBasic) RegisterRESTRoutes(ctx context.CLIContext, rtr *mux.Router) {
    rest.RegisterRoutes(ctx, rtr)
}

// GetTxCmd returns the root tx command for the scavenge module.
func (AppModuleBasic) GetTxCmd(cdc *codec.Codec) *cobra.Command {
    return cli.GetTxCmd(cdc)
}

// GetQueryCmd returns no root query command for the scavenge module.
func (AppModuleBasic) GetQueryCmd(cdc *codec.Codec) *cobra.Command {
    return cli.GetQueryCmd(StoreKey, cdc)
}

//____________________________________________________________________________

// AppModule implements an application module for the scavenge module.
type AppModule struct {
    AppModuleBasic
    keeper     Keeper
    coinKeeper bank.Keeper
}

// NewAppModule creates a new AppModule object
func NewAppModule(k Keeper, bankKeeper bank.Keeper) AppModule {
    return AppModule{
        AppModuleBasic: AppModuleBasic{},
        keeper:         k,
        coinKeeper:     bankKeeper,
    }
}

// Name returns the scavenge module's name.
func (AppModule) Name() string {
    return ModuleName
}

// RegisterInvariants registers the scavenge module invariants.
func (am AppModule) RegisterInvariants(_ sdk.InvariantRegistry) {}

// Route returns the message routing key for the scavenge module.
func (AppModule) Route() string {
    return RouterKey
}

// NewHandler returns an sdk.Handler for the scavenge module.
func (am AppModule) NewHandler() sdk.Handler {
    return NewHandler(am.keeper)
}

// QuerierRoute returns the scavenge module's querier route name.
func (AppModule) QuerierRoute() string {
    return QuerierRoute
}

// NewQuerierHandler returns the scavenge module sdk.Querier.
func (am AppModule) NewQuerierHandler() sdk.Querier {
    return NewQuerier(am.keeper)
}

// InitGenesis performs genesis initialization for the scavenge module. It returns
// no validator updates.
func (am AppModule) InitGenesis(ctx sdk.Context, data json.RawMessage) []abci.ValidatorUpdate {
    var genesisState GenesisState
    ModuleCdc.MustUnmarshalJSON(data, &genesisState)
    InitGenesis(ctx, am.keeper, genesisState)
    return []abci.ValidatorUpdate{}
}

// ExportGenesis returns the exported genesis state as raw bytes for the scavenge
// module.
func (am AppModule) ExportGenesis(ctx sdk.Context) json.RawMessage {
    gs := ExportGenesis(ctx, am.keeper)
    return ModuleCdc.MustMarshalJSON(gs)
}

// BeginBlock returns the begin blocker for the scavenge module.
func (am AppModule) BeginBlock(ctx sdk.Context, req abci.RequestBeginBlock) {
    BeginBlocker(ctx, req, am.keeper)
}

// EndBlock returns the end blocker for the scavenge module. It returns no validator
// updates.
func (AppModule) EndBlock(sdk.Context, abci.RequestEndBlock) []abci.ValidatorUpdate {
    return nil
}
```

Congratulations you have completed the `scavenge` module!

This module is now able to be incorporated into any Cosmos SDK application.

Since we don't want to _just_ build a module but want to build an application that also uses that module, let's go through the process of configuring an app.

If you had any difficulties following this tutorial or simply want to discuss Cosmos tech with us you can [**join our community today**](https://discord.gg/fszyM7K)!

