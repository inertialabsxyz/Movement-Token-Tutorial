# Advanced Features and Customization

This guide covers advanced features of the FA Coin contract and how to customize it for your specific needs.

## Advanced Contract Features

### Account Freezing System

The contract includes a powerful account management system:

#### Freeze an Account
```bash
movement move run --function-id ADDRESS::fa_coin::freeze_account \
  --args address:TARGET_ACCOUNT
```

**Use cases**:
- Compliance with regulations
- Responding to security incidents
- Preventing wash trading
- Temporary account suspension

#### Unfreeze an Account
```bash
movement move run --function-id ADDRESS::fa_coin::unfreeze_account \
  --args address:TARGET_ACCOUNT
```

**Important notes**:
- Only the contract owner can freeze/unfreeze
- Frozen accounts cannot send OR receive tokens
- The admin can still force transfers from frozen accounts

#### Check if Account is Frozen
```bash
movement move view --function-id 0x1::primary_fungible_store::is_frozen \
  --args address:ACCOUNT address:TOKEN_ID \
  --type-args 0x1::fungible_asset::Metadata
```

### Admin Transfer Capabilities

The contract owner has special transfer powers:

```bash
movement move run --function-id ADDRESS::fa_coin::transfer \
  --args address:FROM address:TO u64:AMOUNT
```

**Key differences from user transfers**:
- Can transfer from ANY account
- Bypasses frozen account restrictions
- Useful for recovery operations

### Direct Asset Management

For advanced users, you can interact with the underlying fungible asset system:

#### Withdraw Tokens (Admin)
```move
public fun withdraw(admin: &signer, amount: u64, from: address): FungibleAsset
```

#### Deposit Tokens (Admin)
```move
public fun deposit(admin: &signer, to: address, fa: FungibleAsset)
```

These functions return/accept `FungibleAsset` objects, allowing for complex operations.

## Contract Customization

### Changing Token Parameters

To create your own token, modify these in `basic_token.move`:

```move
// Change the token name and symbol
const ASSET_NAME: vector<u8> = b"My Custom Token";
const ASSET_SYMBOL: vector<u8> = b"MCT";

// In init_module, adjust parameters:
primary_fungible_store::create_primary_store_enabled_fungible_asset(
    constructor_ref,
    option::none(),
    utf8(ASSET_NAME),
    utf8(ASSET_SYMBOL),
    18,  // Change decimals (18 for ETH-like, 8 for BTC-like)
    utf8(b"https://yourproject.com/icon.png"),
    utf8(b"https://yourproject.com"),
);
```

### Adding Maximum Supply Cap

Add supply control to prevent unlimited minting:

```move
// Add to module
const MAX_SUPPLY: u64 = 1000000000000000; // 1 million tokens with 8 decimals
const ERROR_MAX_SUPPLY: u64 = 2;

// Modify mint function
public entry fun mint(admin: &signer, to: address, amount: u64) acquires ManagedFungibleAsset {
    let asset = get_metadata();
    
    // Check current supply
    let current_supply = fungible_asset::supply(asset);
    assert!(current_supply + amount <= MAX_SUPPLY, ERROR_MAX_SUPPLY);
    
    // Continue with normal minting...
}
```

### Implementing Transfer Fees

Add a fee mechanism for transfers:

```move
const TRANSFER_FEE_BPS: u64 = 25; // 0.25% in basis points
const FEE_COLLECTOR: address = @0xFEE;

public entry fun transfer_with_fee(
    sender: &signer, 
    to: address, 
    amount: u64
) acquires ManagedFungibleAsset {
    let fee = (amount * TRANSFER_FEE_BPS) / 10000;
    let transfer_amount = amount - fee;
    
    // Transfer main amount
    primary_fungible_store::transfer(sender, get_metadata(), to, transfer_amount);
    
    // Transfer fee to collector
    primary_fungible_store::transfer(sender, get_metadata(), FEE_COLLECTOR, fee);
}
```

### Adding Minting Roles

Implement role-based access control:

```move
struct MintingRoles has key {
    minters: vector<address>,
}

public entry fun add_minter(admin: &signer, minter: address) acquires MintingRoles {
    // Verify admin
    assert!(signer::address_of(admin) == @movement_token_tutorial, ENOT_OWNER);
    
    let roles = borrow_global_mut<MintingRoles>(@movement_token_tutorial);
    vector::push_back(&mut roles.minters, minter);
}

public entry fun mint_with_role(minter: &signer, to: address, amount: u64) 
acquires ManagedFungibleAsset, MintingRoles {
    let roles = borrow_global<MintingRoles>(@movement_token_tutorial);
    let minter_addr = signer::address_of(minter);
    
    assert!(vector::contains(&roles.minters, &minter_addr), ERROR_NOT_MINTER);
    
    // Proceed with minting...
}
```

### Implementing Vesting

Add token vesting functionality:

```move
struct VestingSchedule has key {
    beneficiary: address,
    total_amount: u64,
    released_amount: u64,
    start_time: u64,
    duration: u64,
}

public entry fun create_vesting(
    admin: &signer,
    beneficiary: address,
    amount: u64,
    duration: u64
) {
    let start_time = timestamp::now_seconds();
    
    move_to(admin, VestingSchedule {
        beneficiary,
        total_amount: amount,
        released_amount: 0,
        start_time,
        duration,
    });
}

public entry fun release_vested(beneficiary: &signer) acquires VestingSchedule {
    let schedule = borrow_global_mut<VestingSchedule>(signer::address_of(beneficiary));
    let elapsed = timestamp::now_seconds() - schedule.start_time;
    
    let vested_amount = if (elapsed >= schedule.duration) {
        schedule.total_amount
    } else {
        (schedule.total_amount * elapsed) / schedule.duration
    };
    
    let to_release = vested_amount - schedule.released_amount;
    // Mint or transfer to_release tokens to beneficiary
}
```

## Integration Patterns

### Building a DEX Integration

To integrate with a decentralized exchange:

```move
public fun get_reserves(): (u64, u64) acquires TokenPair {
    let pair = borrow_global<TokenPair>(@dex_address);
    (pair.token_a_reserve, pair.token_b_reserve)
}

public entry fun add_liquidity(
    provider: &signer,
    amount_a: u64,
    amount_b: u64
) acquires TokenPair {
    // Transfer tokens to DEX
    // Mint LP tokens
    // Update reserves
}
```

### Creating a Staking System

Basic staking implementation:

```move
struct StakeInfo has key {
    amount: u64,
    reward_debt: u64,
    last_update: u64,
}

public entry fun stake(user: &signer, amount: u64) {
    // Lock tokens
    // Record stake info
    // Start earning rewards
}

public entry fun claim_rewards(user: &signer) acquires StakeInfo {
    // Calculate rewards
    // Mint reward tokens
    // Update stake info
}
```

## Testing Advanced Features

### Unit Testing Best Practices

```move
#[test]
fun test_complex_scenario() {
    // Setup
    let admin = create_test_account(@movement_token_tutorial);
    let user1 = create_test_account(@0x1);
    let user2 = create_test_account(@0x2);
    
    // Initialize
    init_module(&admin);
    
    // Test minting
    mint(&admin, address_of(&user1), 1000);
    assert!(balance_of(address_of(&user1)) == 1000, 1);
    
    // Test freezing
    freeze_account(&admin, address_of(&user1));
    assert!(is_frozen(address_of(&user1)), 2);
    
    // Test admin transfer from frozen account
    transfer(&admin, address_of(&user1), address_of(&user2), 500);
    assert!(balance_of(address_of(&user2)) == 500, 3);
}
```

### Integration Testing

Test with multiple modules:

```bash
# Deploy all contracts
movement move publish --named-addresses token=ADDRESS1
movement move publish --named-addresses dex=ADDRESS2

# Test interaction
movement move run --function-id ADDRESS2::dex::add_liquidity \
  --args address:TOKEN1 address:TOKEN2 u64:1000 u64:1000
```

## Performance Optimization

### Gas Optimization Tips

1. **Batch operations**:
```move
public entry fun batch_transfer(
    admin: &signer,
    recipients: vector<address>,
    amounts: vector<u64>
) {
    let i = 0;
    while (i < vector::length(&recipients)) {
        transfer(admin, *vector::borrow(&recipients, i), *vector::borrow(&amounts, i));
        i = i + 1;
    };
}
```

2. **Use inline functions** for frequently called code
3. **Minimize storage operations**
4. **Avoid unnecessary assertions**

### Storage Patterns

Efficient data structure usage:

```move
// Bad: Storing unnecessary data
struct UserInfo has key {
    address: address,  // Redundant
    balance: u64,      // Duplicate of primary store
    name: String,
}

// Good: Only store unique data
struct UserMetadata has key {
    nickname: String,
    join_date: u64,
}
```

## Security Considerations

### Common Vulnerabilities to Avoid

1. **Integer overflow**: Use checked arithmetic
2. **Reentrancy**: Order operations correctly
3. **Access control**: Always verify permissions
4. **Front-running**: Consider commit-reveal schemes

### Audit Checklist

Before mainnet deployment:

- [ ] All tests pass
- [ ] No hardcoded addresses (except constants)
- [ ] Proper error codes and messages
- [ ] Gas limits considered
- [ ] Admin functions protected
- [ ] No infinite loops
- [ ] Decimal handling correct
- [ ] Supply constraints enforced

## Monitoring and Analytics

### On-chain Analytics

Track token metrics:

```move
struct TokenMetrics has key {
    total_transfers: u64,
    total_burns: u64,
    total_mints: u64,
    unique_holders: u64,
}

// Update in each function
public entry fun transfer(...) acquires TokenMetrics {
    // ... transfer logic ...
    
    let metrics = borrow_global_mut<TokenMetrics>(@movement_token_tutorial);
    metrics.total_transfers = metrics.total_transfers + 1;
}
```

### Off-chain Monitoring

Use Movement's APIs to track:
- Transaction volume
- Holder distribution
- Transfer patterns
- Gas usage trends

## Migration Strategies

### Upgrading Contracts

Since Movement doesn't support direct upgrades yet:

1. **Deploy new version**
2. **Implement migration function**:
```move
public entry fun migrate_balances(
    admin: &signer,
    old_token: Object<Metadata>,
    users: vector<address>
) {
    // For each user
    // Burn old tokens
    // Mint new tokens
}
```

3. **Coordinate community migration**
4. **Deprecate old contract**

## Next Steps

1. **Experiment on testnet** with these advanced features
2. **Join the Movement developer community**
3. **Review other production contracts** for patterns
4. **Consider formal verification** for critical contracts
5. **Plan your token economics** carefully

Remember: With great power comes great responsibility. Admin functions should be used carefully and transparently!