---
snip: TODO
title: Applicative Paymaster API Standard
description: TODO
author: TODO
discussions-to: 
status: Review
type: Standards Track
category: Core
created: 2025-01-16
---

## Simple Summary

This SNIP proposes a standardized interface between an applicative paymaster service and a user. A user can be a single individual, a wallet, a DApp backend, etc... The specification aims to be as minimal as possible while enabling a full applicative paymaster flow with good UX. Any paymaster service is free to expand on this specification and add more features to it.

## Motivation

The motivation of this SNIP is to make it easier for the Starknet ecosystem to integrate with an applicative paymaster. A neutral specification prevents fragmentation of interfaces of different paymaster services and simplifies paymaster integration for any entity that wishes to do so. It makes it possible for Starknet SDKs to integrate a paymaster specification which is independent of any existing vendor.


## Specification

The full JSON-RPC specification is found in `paymaster_api.json`. The specification is modeled around the following flow:
1. A user submits to the paymaster service a list of calls that they wish to make, along with the token they wish to pay fees with and fee bounds.
2. The paymaster service responds with a typed object (in the sense of SNIP-12) that wraps the calls submitted by the user and which is ready for signature. The purpose of the typed object is to:
    1. Wrap the user calls into a format compatible with outside execution (SNIP-9).
    2. Potentially add additional calls which are necessary for the fee accounting logic of the paymaster service.
3. The user signs on the typed object and sends it (along with the signature) to the paymaster service for execution.
4. The paymaster service finally sends a transaction onchain from its relayer account, containing the signed outside execution payload.

### Version 1

The properties `types` and `domain` of the object `OUTSIDE_EXECUTION_TYPED_DATA_V1` MUST be set as follows:

```json
"types": {
  "StarkNetDomain": [
    { "name": "name", "type": "felt" },
    { "name": "version", "type": "felt" },
    { "name": "chainId", "type": "felt" },
  ],
  "OutsideExecution": [
    { "name": "caller", "type": "felt" },
    { "name": "nonce", "type": "felt" },
    { "name": "execute_after", "type": "felt" },
    { "name": "execute_before", "type": "felt" },
    { "name": "calls_len", "type": "felt" },
    { "name": "calls", "type": "OutsideCall*" },
  ],
  "OutsideCall": [
    { "name": "to", "type": "felt" },
    { "name": "selector", "type": "felt" },
    { "name": "calldata_len", "type": "felt" },
    { "name": "calldata", "type": "felt*" },
  ],
},
"domain": {
    "name": "Account.execute_from_outside",
    "version": "1",
    "chainId": "0x534e5f4d41494e"
    
}

```

The `"chainId"` property of the `"domain"` object is set to the chainId of Starknet mainnet. For Starknet testnet, the corresponding chainId should be used.

### Version 2

The properties `types` and `domain` of the object `OUTSIDE_EXECUTION_TYPED_DATA_V2` MUST be set as follows:
```json
"types": {
    "StarknetDomain": [
        { "name": "name", "type": "shortstring" }, 
        { "name": "version", "type": "shortstring" },
        { "name": "chainId", "type": "shortstring" },
        { "name": "revision", "type": "shortstring" }
    ],
    "OutsideExecution": [
        { "name": "Caller", "type": "ContractAddress" },
        { "name": "Nonce", "type": "felt" },
        { "name": "Execute After", "type": "u128" },
        { "name": "Execute Before", "type": "u128" },
        { "name": "Calls", "type": "Call*" }
      ],
      "Call": [
        { "name": "To", "type": "ContractAddress" },
        { "name": "Selector", "type": "selector" },
        { "name": "Calldata", "type": "felt*" }
      ]
}
"domain": {
    "name": "Account.execute_from_outside",
    "version": "2",
    "chainId": "0x534e5f4d41494e"
}
```
The field `"revision"` of the `"domain` object above is omitted: it should follow the revision of SNIP-12 that is being used (either the integer `1` or the shortstring `"2"`).

The `"chainId"` property of the `"domain"` object is set to the chainId of Starknet mainnet. For Starknet testnet, the corresponding chainId should be used.

### Version 3

TODO: Add version 3 of outside execution which is paymaster "safe". This should be coupled with a modification of SNIP-9 as well.

## Rationale

## Backwards Compatibility

This specification may be breaking for existing paymaster services: more precisely, compliance with this specification may incur breaking upgrades for existing paymaster services.

## Security Considerations

TODO: to mention or not to mention attacks on paymasters.

## Copyright

Copyright and related rights waived via [MIT](../LICENSE).
