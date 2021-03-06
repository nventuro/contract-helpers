# Solc 0.4 contract mocks

Usage: `import "@aragon/contract-helpers-test/contracts/0.4/<contract>";`

- [`aragonOS/`](#aragonos)
  - [`EtherTokenConstant`](#ethertokenconstant)
  - [`TimeHelpersMock`](#timehelpersmock)
  - [`SharedTimeHelpersMock`](#sharedtimehelpersmock)
- [`token/`](#tokens)
  - [`TokenMock`](#tokenmock)
  - [`TokenReturnFalseMock`](#tokenreturnfalsemock)
  - [`TokenReturnMissingMock`](#tokenreturnmissingmock)
- [`misc/`](#misc)
  - [`ExecutionTarget`](#executiontarget)
- [`internal/`](#internal)

## aragonOS

Requires `@aragon/os <= 4.x` to also be installed.

### `EtherTokenConstant`

A simple mock to get the "mock" token address of ether for aragonOS and most Aragon app implementations, when the contract supports both ether and a token address.

```solidity
contract EtherTokenConstantMock is EtherTokenConstant {
    function getETHConstant() external pure returns (address);
}
```

### `TimeHelpersMock`

A mock to manipulate aragonOS's `TimeHelpers` construct.

This mock is useful to ensure smart contracts requiring a notion of time can be easily tested on "real" chain implementations like geth or parity, which do not implement "test" APIs such as `evm_increaseTime` or `evm_mine` (and rightly so!).

Most Aragon apps extend and utilize the `TimeHelpers` contract, allowing this mock to manipulate the app's internal concept of time.

You will usually use this mock by "wrapping" it around your actual app contract, like so:

```solidity
contract MyApp is AragonApp {
}


// Automatically expose the time-control utilities from TimeHelpersMock into the mock
contract MyAppMock is MyApp, TimeHelpersMock {
}
```

And in your test files:

```js
// Use the mocked version for testing
const MyApp = artifacts.require('MyAppMock')

const myApp = await MyApp.new()
myApp.mockSetTimestamp(...)
```

There are a number of simple utility methods included in the `TimeHelpersMock` to update, advance, or reverse time, so the best [documentation is the implementation itself](./aragonOS/TimeHelpersMock.sol).

### `SharedTimeHelpersMock`

This is a similar mock to the `TimeHelpersMock`, except it allows you to externally configure a "clock" (a `TimeHelpersMock` instance).

This is useful in situations where you would like multiple contracts to share a single source of mocked time, without having to manipulate each contract's time functions individually.

To use it, you will use it similarly to `TimeHelpersMock`:

```solidity
contract MyApp is AragonApp {
}


// Automatically expose the time-control utilities from TimeHelpersMock into the mock
contract MyAppMock is MyApp, SharedTimeHelpersMock {
}
```

And in your test files:

```js
const MyApp = artifacts.require('MyAppMock')
const TimeHelpersMock = artifacts.require('TimeHelpersMock')

const clock = await TimeHelpersMock.new()
const myApp = await MyApp.new()
myApp.setClock(clock.address)

// Update time in myApp
clock.mockSetTimestamp(...)
```

Note that all time manipulation now occurs through the shared `TimeHelpersMock` clock instance.

## Tokens

Imported via `@aragon/contract-helpers-test/contracts/0.4/token/`.

### `TokenMock`

A standards compliant ERC-20 token, with the ability to specify a specific account's balance on construction:

```solidity
contract TokenMock {
    constructor(address initialAccount, uint256 initialBalance) public;
}
```

Returns `true` on allowed actions and reverts on failed actions.

### `TokenReturnFalseMock`

A standards compliant ERC-20 token, similar to `TokenMock`, but while `TokenMock` reverts on failed actions, `TokenReturnFalseMock` returns `false`.

### `TokenReturnMissingMock`

A **non-compliant** ERC-20 token, which does not return a value on allowed actions.

Useful for testing that a contract correctly handles these non-compliant tokens, of which there are [multiple high-profile ones deployed](https://medium.com/coinmonks/missing-return-value-bug-at-least-130-tokens-affected-d67bf08521ca).

## Misc.

### `ExecutionTarget`

A small mock contract that is meant to be used as the "target" of another action (such as a proxied call).

```solidity
contract ExecutionTarget {
    event TargetExecuted(uint256 counter);
    uint256 public counter;
    function execute() external;
}
```

On each successful call to `execute()`, `counter` will be incremented and `TargetExecuted` will be emitted.

## Internal

Contracts in the `internal/` subdirectory are meant to be private to this package and should not be used directly.

They have been given prefixed names to avoid causing name clashes with real contracts.
