# SRS-20: Simple Restricted Token Standard

SRS-20 is an easily extendable standard for issuing tokens with transfer restrictions. See [EIP-XXX](#) to learn more.

## Motivation

There are many exciting use-cases for tokens with transfer restrictions.

A few emergent examples:

- Enforcing Token Lock-Up Periods
- Enforcing Passed AML/KYC Checks
- Private Real-Estate Investment Trusts
- Delaware General Corporations Law Shares

Furthermore, standards adoption amongst token issuers has the potential to evolve into a dynamic and interoperable landscape of automated compliance.

However, we feel the current trend of ~~proposing~~ marketing token standards as solutions for individual use-cases poses a threat to future interoperability within the tokenized ecosystem.

Specifically this compromises the usuability of security tokens and the exchanges that wish to support them.

It is our belief that a simplistic underlying standard, which may be easily extended for varying compliance needs, is a far more forward-thinking approach.

This design gives greater freedom / upgradability to token issuers and simultaneously decreases the burden of integration for developers and exchanges.

Additionally, we see fit to provide a framework by which human-readable messages may be returned when token transfers are rejected.

Visibility into _why_ a token's transfer was rejected is of equal importance to the successful enforcement of the transfer restriction itself.

A widely adopted pattern for handling rejections and messaging within token transfers will highly convenience the exchanges, wallets, and interface builders of the future.

## The Standard

As a standard, SRS-20 is dead simple to implement.

It adds _just two_ public functions on top of the tried and true ERC-20 standard.

```solidity
/// @notice Detects if a transfer will be rejected and if so returns an appropriate reference code
/// @param {address} from - Sending address
/// @param {address} to - Receiving address
/// @param {uint} value - Amount of tokens being transferred
/// @return {uint} restrictionCode - Identifier by which to reference message for rejection reasoning
function detectTransferRestriction (address from, address to, uint value)
  public
  view
  returns (uint restrictionCode)
{
  /* ... */
}

/// @notice Returns a human-readable message for a given restriction code
/// @param {uint} restrictionCode - Identifier for looking up a message
/// @return {string} message - Text showing the restriction's reasoning
function messageForTransferRestriction (uint restrictionCode)
  public
  view
  returns (string message)
{
  /* ... */
}
```

The logic of `detectTransferRestriction()` and `messageForTransferRestriction()` are left up to the issuer, with just two requirements respectively:

1.  SRS-20 tokens must perform a `detectTransferRestriction()` check inside `transfer` and `transferFrom` methods. If a value other than `0` is returned, revert the transaction.
2.  SRS-20 tokens must implement `messageForTransferRestriction()` in such a way that a `restrictionCode` of `0` will always return the message string `"SUCCESS"`.

**That's it. Seriously.**

Our [reference implementation](https://github.com/tokensoft/simple-restricted-token-standard/blob/master/contracts/SimpleRestrictedToken.sol) respects both these constraints and can easily be extended to handle more advanced use cases.

### Basic Usage

Install

In your
`$ npm install @tokensoft/simple-restricted-token`

View on [Remix](#)

```solidity
contract MyRestrictedToken is SimpleRestrictedToken {
  constructor () public {
    /* ... */
  }

  // Detect a restriction and return an appropriate code
  function detectTransferRestriction (address from, address to, uint value)
    public
    constant
    returns (uint restrictionCode)
  {
    if (to == 0x0) {
      restrictionCode = 1; // illegal transfer to zero address
    } else if (to == address(this)) {
      restrictionCode = 2; // illegal transfer to own token contract
    } else {
      restrictionCode = 0; // successful transfer (required)
    }
  }

  // Return message for given restriction code
  function messageForTransferRestriction (uint restrictionCode)
    public
    view
    return (string message)
  {
    if (restrictionCode == 0) {
      message = "SUCCESS"; // required
    } else if (restrictionCode == 1) {
      message = "ILLEGAL_TRANSFER_TO_ZERO_ADDRESS";
    } else if (restrictioNCode == 2) {
      message = "ILLEGAL_TRANSFER_TO_OWN_TOKEN_CONTRACT";
    } else {
      message = "UNKNOWN"; // fallback
    }
  }
}
```

## Common Patterns

Several common transfer restriction patterns implemented under SRS-20:

1.  [Cap Tables](#)
2.  [Lock-Up Periods](#)
3.  [Maximum Shareholder Accounts](#)
4.  [Whitelisted Investor Addresses](#)

## Prominent Examples

SRS-20 is so easy to extend that we have included example implementations of the following high-profile standards on top of it:

1.  [Harbor's R-Token](#)
2.  [Polymath's ST-20 Token](#)
3.  [Delaware General Corporations Law (DCGL) ERC-884 Token](#)
