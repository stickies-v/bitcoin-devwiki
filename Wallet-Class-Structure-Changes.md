# Preface

This describes the wallet class structure changes that were discussed at CoreDev Amsterdam. A transcript of most of this discussion can be found [here](http://diyhpl.us/wiki/transcripts/bitcoin-core-dev-tech/2019-06-05-wallet-architecture/).

# The New Structure

The wallet is currently structured as a monolith. The `CWallet` class contains basically everything in the wallet - address generation, key generation, signing, transaction tracking, determining whether a transaction belongs to it, etc. The goal of this change will be to break out the address generation, key generation, signing, and ismine parts into a separate "box" within the wallet. This "box" will provide a standard interface for address generation, key fetching (for signing) and determining ismine and `CWallet` can have one or more of these boxes for the various address types (i.e. a default for legacy, one for bech32, and one to query for p2sh-segwit). Each box's internal implementation can vary which allows us to create one for the current wallet behavior and one for native descriptor wallets without requiring other significant changes to `CWallet`. This will also allow for future expandability

## The "box"

Within every `CWallet`, there will be one or more boxes, also called `ScriptPubKeyManager`. `ScriptPubKeyManager` will provide a `GetSigningProvider()` function which will return the `SigningProvider` for a given script to be used during signing. There will be additional functions added to it that allow for address fetching (e.g. `GetNewAddress()`). Each `ScriptPubKeyManager` will also have it's own `IsMine()` function to determine whether a `CTxOut` belongs to that `ScriptPubKeyManager` (and thus the wallet itself). Each `ScriptPubKeyManager` maintains which address type(s) (legacy/bech32/p2sh-wrapped) it can return and will respond accordingly to `GetNewAddress()` calls.

## `CWallet` changes

`CWallet` will no loner handle keys, addresses, and `IsMine` directly. Instead it will contain one or more `ScriptPubKeyManager`s with a map of address types - every combination of legacy/p2sh-segwit/bech32 and change/receive - to `ScriptPubKeyManager`. So there will be 6 `ScriptPubKeyManager` objects tracked in the wallet for legacy-receive, legacy-change, p2sh-receive, p2sh-change, bech32-receive, and bech32-change. These may all be the same `ScriptPubKeyManager` or different. When a new address is being requested, the default `ScriptPubKeyManager` for that type (legacy/p2sh/bech32, change/receive) is retrieved and a new address fetched from it. The type is usually decided based on the default for either change or non-change in the wallet, but can sometimes be specified explicitly (e.g. in the getnewaddress RPC). `IsMine` will be moved to be part of `ScriptPubKeyManager` so wallets will call `IsMine` for each of its `ScriptPubKeyManager`s to determine whether a transaction belongs to it. For signing, `ScriptPubKeyManager` will be passed into `ProduceSignature` as the `SigningProvider` to be used there. In order to determine which `SPKManger` to use when signing, the wallet will go through each of the `ScriptPubKeyManager`s it has and find the one which returns `IsMine = True` for a scriptPubKey. The `ScriptPubKeyManager` that does will be the one used for signing.

# Prerequisites

## `IsMine` (:heavy_check_mark: done in https://github.com/bitcoin/bitcoin/pull/16226)

Before this new class structure can be implemented, some things need to be moved around and changed. `IsMine` is currently a standalone module because it is used in a couple of non-wallet tests. However it needs to be moved to be part of the wallet module first, possibly as a member function of `CWallet`. Once `SPKManger` is introduced, `IsMine` will become a member function if that.

## `CWallet` Subclass Stack (:heavy_check_mark: Done in https://github.com/bitcoin/bitcoin/pull/16227)

The inheritance chain from `SigningProvider` should be condensed and the scopes slightly changed. The chain currently is:

```
SigningProvider -> CKeyStore -> CBasicKeyStore -> CCryptoKeyStore -> CWallet
```

Instead, `CWallet` will need to be standalone and `ScriptPubKeyManager` will be a `SigningProvider`; there is no need for `CWallet` to be a `SigningProvider`. Additionally `CKeyStore`, `CBasicKeyStore`, and `CCryptoKeyStore` are largely unnecessary. `CKeyStore` can be removed and it's functionality split into `SigningProvider` and `CBasicKeyStore`. The `Add*` functions can go up to `CBasicKeyStore` while the `Have*` go down to `SigningProvider`. The watch only related functions should go to `CWallet` and `CBasicKeyStore` can be renamed to another type of `SigningProvider`. Lastly, `CCryptoKeyStore` should be entirely combined with `CWallet`. At the end of this refactoring, the stack will be:

```
SigningProvider -> CBasicKeyStore (renamed) -> CWallet
```
Combined with the addition of `ScriptPubKeyManager`, `CWallet` will be standalone and `ScriptPubKeyManager` will extend `CBasicKeyStore (renamed)`.

## Give out Destinations from the wallet instead of keys (:heavy_check_mark: Done in https://github.com/bitcoin/bitcoin/pull/16237)

Since `ScriptPubKeyManager` moves the wallet to an addresses and scripts model rather than keys, we need to give out the addresses (aka Destinations) from the wallet instead of keys. So RPCs like `getnewaddress` will fetch a Destinaion from the wallet rather than fetching and converting a key to a Destination. This will allow us to have different `ScriptPubKeyManager`s later that can have arbitrary scripts. To make the transition easier, everything that was getting keys from the wallet needs to be getting Destinations and the wallet needs to give out Destinations instead of keys.

# See Also

Wallet and Segwit: https://gist.github.com/sipa/125cfa1615946d0c3f3eec2ad7f250a2