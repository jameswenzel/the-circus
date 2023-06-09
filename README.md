# The Circus

## Q: What do Seaport Orders and The Circus have in common?
A: They're both intents.

----

## Usage

`TheCircusZone` is a proof-of-concept Seaport Zone that takes `extraData` in the form of an abi-encoded struct:

```solidity
struct ArbitraryIntentData {
    bytes initData;
    bytes32 salt;
}
```

This data is used to CREATE2 an ephemeral smart contract designed to be a container to execute arbitrary logic within its constructor, without actually writing code to the chain.

This allows for expressive, arbitrary assertions to be made by `TheCircusZone`, to emphasize Seaport's potential as an "intent" protocol; a signed Seaport order will not be fulfillable unless all `ConsiderationItem`s are satisified _in addition_ to any arbitrary checks specified by an `ArbitraryIntent`. Since Zones can both read _and_ write arbitrary data, this allows for a wide range of use cases.

### TheCircusZone

A simple zone that reads the `extraData` from the `ZoneParameters` as an `ArbitraryIntentData` struct. It calls `CREATE2` using this data, and reverts if any data returned, otherwise passing the `validateOrder` check. The resulting address is checked against the `zoneHash` of the order, meaning all logic must be set at the time of order creation.

### ArbitraryIntent

An abstract template contract for executing arbitrary code within the context of a Seaport Zone – "intent" logic is implemented in the `expressIntent()` function. If a revert is specified with a non-zero amount of data, `TheCircusZone` will revert with this data, otherwise, the `validateOrder` check will pass.

### `test/helpers/SpecificIntent.sol`

A very simple "specific" Intent that reverts if a particular smart contract has not been deployed.

# FAQ

## What are intents?
idk tbh

## What is Seaport?
[Seaport](https://github.com/ProjectOpenSea/seaport) is the most powerful, extensible, and gas-efficient token settlement platform on EVM chains.

## What are Seaport Zones?
Zones are accounts that must authorize a Seaport order as part of fulfillment. Strictly speaking, they don't _have_ to be smart contracts, but they're a lot more interesting if they are.

Seaport will call `validateOrder` on smart contract Zones specified by orders' `OrderParameters`, after all token transfers have been made. This means Zones can make arbitrary assertions about the state of the chain _after_ the order has been executed, such as:

> This account should have at least X balance of Y token, and at most B balance of C token

without being prescriptive about how exactly that happens.

Since Seaport makes a stateful call, Zones may also perform on-chain update.

## Why CREATE2 "intents?"
To highlight that Seaport Zones can do pretty much anything

## Isn't that pretty gas-intensive?
Yeah

## Can't I just deploy a custom Seaport Zone for my use-case?
Yeah

## How is "The Circus" different from normal Seaport Zones?
It's not