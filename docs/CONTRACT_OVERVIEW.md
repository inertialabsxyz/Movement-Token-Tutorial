# FA Coin Contract Overview

## Introduction

The FA Coin contract (`move/sources/basic_token.move`) is a fungible asset implementation based on the Aptos fungible asset standard. This document provides a detailed explanation of how the contract works, making it easier for developers to understand and modify.

## What is a Fungible Asset?

A **fungible asset** is a type of token where each unit is interchangeable with another. Think of it like currency - one dollar bill is worth the same as any other dollar bill. In blockchain terms, fungible assets are used for:
- Cryptocurrencies
- Stablecoins
- Governance tokens
- Gaming currencies
- Loyalty points

## Contract Architecture

### Module Declaration
```move
module movement_token_tutorial::fa_coin {
```
- **movement_token_tutorial**: The address where the module is deployed
- **fa_coin**: The name of our module

### Key Imports
The contract uses several important framework modules:

1. **fungible_asset**: Core functionality for creating and managing fungible tokens
2. **object**: Movement's object model for resource management
3. **primary_fungible_store**: Manages token storage in user accounts
4. **signer**: Validates transaction signers
5. **string**: String manipulation utilities

## Core Components

### 1. The ManagedFungibleAsset Struct

```move
struct ManagedFungibleAsset has key {
    mint_ref: MintRef,
    transfer_ref: TransferRef,
    burn_ref: BurnRef,
}
```

This struct holds three critical "references" that give the contract owner special powers:

- **mint_ref**: Permission to create new tokens
- **transfer_ref**: Permission to transfer tokens (even from frozen accounts)
- **burn_ref**: Permission to destroy tokens

These references are created once during initialization and stored on-chain.

### 2. Initialization Function

```move
fun init_module(admin: &signer) {
```

This function runs automatically when the contract is deployed. It:

1. **Creates a named object** using the token symbol
2. **Sets up the fungible asset** with:
   - Name: "FA Coin"
   - Symbol: "FA"
   - Decimals: 8 (like Bitcoin)
   - Icon and project URLs
3. **Generates control references** for minting, burning, and transferring
4. **Stores these references** in the ManagedFungibleAsset struct

**Key Concept**: The `init_module` function can only run once - during deployment. This ensures token parameters can't be changed later.

## Public Functions

### View Functions (Read-Only)

#### get_metadata()
```move
public fun get_metadata(): Object<Metadata>
```
- Returns the token's metadata object
- This object ID is used to identify the token in other operations
- No gas cost (it's a view function)

#### get_name()
```move
public fun get_name(): string::String
```
- Returns the token's display name ("FA Coin")
- Useful for UI display

### State-Changing Functions

#### mint()
```move
public entry fun mint(admin: &signer, to: address, amount: u64)
```
**Purpose**: Create new tokens

**How it works**:
1. Verifies the signer is the contract owner
2. Creates `amount` new tokens
3. Deposits them into the `to` address
4. Automatically creates a token store for the recipient if needed

**Who can call**: Only the contract owner

#### transfer()
```move
public entry fun transfer(admin: &signer, from: address, to: address, amount: u64)
```
**Purpose**: Move tokens between accounts (admin function)

**Special power**: Can transfer even if accounts are frozen

**Use case**: Admin recovery or intervention

#### burn()
```move
public entry fun burn(admin: &signer, from: address, amount: u64)
```
**Purpose**: Permanently destroy tokens

**Effect**: Reduces total supply by removing tokens from circulation

**Common uses**:
- Deflationary tokenomics
- Removing tokens from circulation
- Error correction

#### freeze_account() / unfreeze_account()
```move
public entry fun freeze_account(admin: &signer, account: address)
public entry fun unfreeze_account(admin: &signer, account: address)
```
**Purpose**: Control account access

**When frozen**: Account cannot send or receive tokens

**Use cases**:
- Compliance requirements
- Security incidents
- Preventing malicious activity

## Security Model

### Authorization Check
```move
inline fun authorized_borrow_refs(owner: &signer, asset: Object<Metadata>)
```

This critical function:
1. Checks if the signer owns the metadata object
2. Returns the control references if authorized
3. Aborts the transaction if unauthorized

**Key Security Feature**: Only the address that deployed the contract can mint, burn, or use admin functions.

## Token Lifecycle

```
1. Deployment
   └─> init_module() creates token with fixed parameters

2. Minting
   └─> Admin creates new tokens
       └─> Tokens deposited to user accounts

3. Transfers
   ├─> Users transfer via primary_fungible_store
   └─> Admin can force transfers if needed

4. Burning
   └─> Admin destroys tokens
       └─> Reduces total supply

5. Account Management
   ├─> Freeze: Temporarily disable account
   └─> Unfreeze: Re-enable account
```

## Primary Fungible Store

The contract uses Movement's **primary fungible store** system:

- Each account can have one primary store per token type
- Stores are created automatically when needed
- Users interact with stores via framework functions
- No need to manually manage token balances in the contract

## Testing

The contract includes two test functions:

### test_basic_flow()
Tests the happy path:
1. Initialize the module
2. Mint tokens
3. Freeze an account
4. Transfer tokens (even while frozen)
5. Unfreeze the account
6. Burn tokens

### test_permission_denied()
Tests security:
- Verifies that non-owners cannot mint tokens
- Expects the transaction to abort with error code

## Best Practices for Modification

When modifying this contract:

1. **Never remove security checks** - The authorization system is critical
2. **Test thoroughly** - Add tests for any new functions
3. **Consider decimals** - Remember amounts are in smallest units
4. **Document changes** - Update comments and documentation
5. **Audit references** - Be careful with mint/burn/transfer refs

## Common Modifications

### Change Token Parameters
Modify these constants in the contract:
```move
const ASSET_NAME: vector<u8> = b"Your Token Name";
const ASSET_SYMBOL: vector<u8> = b"YTN";
```

### Add Supply Cap
Add a maximum supply check in the mint function:
```move
let current_supply = fungible_asset::supply(asset);
assert!(current_supply + amount <= MAX_SUPPLY, ERROR_EXCEEDS_CAP);
```

### Add Fees
Implement transfer fees:
```move
let fee = amount * FEE_PERCENTAGE / 100;
let transfer_amount = amount - fee;
```

### Public Minting
Remove the authorization check to allow anyone to mint (use carefully!):
```move
public entry fun public_mint(user: &signer, amount: u64)
```

## Movement vs Aptos Differences

While this contract is based on Aptos standards, remember:

1. **Network endpoints** are different (Movement-specific)
2. **Framework location**: Use `movementlabsxyz/aptos-core` not `aptos-labs`
3. **Explorer URLs** use Movement's explorer
4. **Gas costs** may differ between networks

## Further Learning

- [Movement Documentation](https://docs.movementnetwork.xyz/)
- [Aptos Move Tutorial](https://aptos.dev/tutorials/)
- [Move Language Book](https://move-language.github.io/move/)

## Questions?

If you have questions about this contract:
1. Check the [Troubleshooting Guide](TROUBLESHOOTING.md)
2. Review the test functions for examples
3. Experiment on testnet before mainnet
4. Join the Movement community for support