# Troubleshooting Guide

This guide covers common issues you might encounter while working with the Movement Token Tutorial and their solutions.

## Installation Issues

### Movement CLI Won't Install

**Problem**: `cargo build -p movement` fails

**Solutions**:
1. **Update Rust**: 
   ```bash
   rustup update
   ```

2. **Clear cargo cache**:
   ```bash
   cargo clean
   rm -rf ~/.cargo/registry/cache
   ```

3. **Check system requirements**:
   - Rust 1.70.0 or higher
   - Git installed
   - Sufficient disk space (>2GB)

4. **Try release build** (slower but more stable):
   ```bash
   cargo build -p movement --release
   ```

### Command Not Found After Installation

**Problem**: `movement: command not found`

**Solutions**:
1. **Check installation path**:
   ```bash
   which movement
   ls -la /usr/local/bin/movement
   ```

2. **Add to PATH manually**:
   ```bash
   echo 'export PATH="/usr/local/bin:$PATH"' >> ~/.bashrc
   source ~/.bashrc
   ```

3. **For macOS users**:
   ```bash
   echo 'export PATH="/usr/local/bin:$PATH"' >> ~/.zshrc
   source ~/.zshrc
   ```

## Configuration Issues

### Movement Init Fails

**Problem**: `movement init` returns an error

**Solutions**:
1. **Check network connectivity**:
   ```bash
   curl https://full.testnet.movementinfra.xyz/v1
   ```

2. **Use the provided config**:
   - This tutorial includes `.movement/config.yaml`
   - You don't need to run `init` if using the tutorial config

3. **Manual configuration**:
   - Copy the provided config.yaml
   - Update with your own keys if desired

### Wrong Framework in Move.toml

**Problem**: Build fails with dependency errors

**Solution**: Ensure Move.toml uses Movement's framework:
```toml
[dependencies.AptosFramework]
git = "https://github.com/movementlabsxyz/aptos-core.git"
rev = "movement"
subdir = "aptos-move/framework/aptos-framework"
```

**NOT** the standard Aptos repository!

## Build and Test Issues

### Build Fails with Type Errors

**Problem**: `movement move build` shows type mismatches

**Solutions**:
1. **Clean build directory**:
   ```bash
   rm -rf move/build
   movement move build
   ```

2. **Check Move.toml addresses**:
   ```toml
   [addresses]
   movement_token_tutorial = "_"
   ```

3. **Verify framework version**:
   - Ensure you're using Movement's fork
   - Pull latest changes if needed

### Tests Fail

**Problem**: `movement move test` fails

**Solutions**:
1. **Check test address**:
   ```move
   #[test(creator = @FACoin)]  // Must match expected address
   ```

2. **Run specific test**:
   ```bash
   movement move test --filter test_basic_flow
   ```

3. **Increase verbosity**:
   ```bash
   movement move test --verbose
   ```

## Deployment Issues

### Insufficient Balance

**Problem**: "Insufficient balance for gas"

**Solutions**:
1. **Request testnet tokens**:
   ```bash
   movement account fund-with-faucet
   ```

2. **Check balance**:
   ```bash
   movement account balance
   ```

3. **Wait for funding** (can take 30 seconds):
   ```bash
   sleep 30
   movement account balance
   ```

### Module Already Published

**Problem**: "Module already exists"

**Solutions**:
1. **For testnet**: Deploy to a different account
2. **For learning**: This is OK - your module is already deployed
3. **To upgrade**: Movement doesn't support upgrades yet

### Transaction Failed

**Problem**: Deployment transaction fails

**Common causes and solutions**:

1. **Gas too low**:
   - The network automatically estimates gas
   - If it fails, try again

2. **Network congestion**:
   - Wait and retry
   - Check network status

3. **Invalid bytecode**:
   - Rebuild: `movement move build --skip-fetch-latest-git-deps`

## Function Execution Issues

### Permission Denied Errors

**Problem**: `ENOT_OWNER` error when calling functions

**Solution**: Only the deployer account can call admin functions:
- `mint`, `burn`, `transfer` (admin version)
- `freeze_account`, `unfreeze_account`

Make sure you're using the correct account!

### Cannot Find Module

**Problem**: "Cannot find module fa_coin"

**Solutions**:
1. **Verify deployment**:
   ```bash
   movement account list --query modules
   ```

2. **Check address**:
   - Use the address that deployed the contract
   - Not the token ID (metadata object)

3. **Correct function call**:
   ```bash
   movement move view --function-id YOUR_ADDRESS::fa_coin::get_name
   ```

### Balance Shows Zero After Minting

**Problem**: Minted tokens but balance is 0

**Solutions**:
1. **Check correct token ID**:
   ```bash
   movement move view --function-id ADDRESS::fa_coin::get_metadata
   ```

2. **Use the metadata object for balance**:
   ```bash
   movement move view --function-id 0x1::primary_fungible_store::balance \
     --args address:ACCOUNT address:TOKEN_ID \
     --type-args 0x1::fungible_asset::Metadata
   ```

## Network Issues

### Connection Timeouts

**Problem**: Commands hang or timeout

**Solutions**:
1. **Check network URLs**:
   - Testnet: `https://full.testnet.movementinfra.xyz/v1`
   - Mainnet: `https://full.mainnet.movementinfra.xyz/v1`

2. **Test connectivity**:
   ```bash
   curl -I https://full.testnet.movementinfra.xyz/v1
   ```

3. **Use explicit URL**:
   ```bash
   movement account balance --url https://full.testnet.movementinfra.xyz/v1
   ```

### Faucet Not Working

**Problem**: Cannot get testnet tokens

**Solutions**:
1. **Direct faucet request**:
   ```bash
   curl -X POST https://faucet.testnet.movementinfra.xyz/mint \
     -H "Content-Type: application/json" \
     -d '{"address":"YOUR_ADDRESS","amount":100000000}'
   ```

2. **Use web faucet**:
   Visit: https://faucet.testnet.movementinfra.xyz/

3. **Check faucet status**:
   - Faucets have rate limits
   - Wait 1 hour between requests

## Explorer Issues

### Transaction Not Found

**Problem**: Explorer shows "transaction not found"

**Solutions**:
1. **Wait for confirmation** (10-30 seconds)
2. **Check network parameter**:
   - Testnet: `?network=bardock+testnet`
   - Mainnet: `?network=mainnet`
3. **Correct URL format**:
   ```
   https://explorer.movementlabs.xyz/txn/TX_HASH?network=bardock+testnet
   ```

## Common Error Codes

| Error Code | Meaning | Solution |
|------------|---------|----------|
| 0x1 | Not owner | Use the account that deployed the contract |
| 0x50001 | Permission denied | Check signer authorization |
| 0x60001 | Insufficient balance | Fund account or reduce amount |
| 0x60002 | Account frozen | Unfreeze the account first |
| 0x10003 | Module not found | Deploy the contract first |

## Script Issues

### Scripts Won't Run

**Problem**: `./scripts/install-movement-cli.sh: Permission denied`

**Solution**: Make scripts executable:
```bash
chmod +x scripts/*.sh
```

### Scripts Fail on macOS

**Problem**: Scripts use Linux-specific commands

**Solutions**:
1. **Install GNU tools**:
   ```bash
   brew install coreutils
   ```

2. **Adjust script paths** if needed

## Getting Help

If your issue isn't covered here:

1. **Check the logs**:
   ```bash
   movement move build --verbose
   ```

2. **Movement Resources**:
   - [Movement Discord](https://discord.gg/movementnetwork)
   - [Movement Documentation](https://docs.movementnetwork.xyz/)
   - [GitHub Issues](https://github.com/movementlabsxyz/aptos-core/issues)

3. **Debug tips**:
   - Start with testnet, not mainnet
   - Test each step individually
   - Use view functions to verify state
   - Check explorer for transaction details

## Prevention Tips

1. **Always test on testnet first**
2. **Keep private keys secure**
3. **Verify addresses before transactions**
4. **Check balances before operations**
5. **Read error messages carefully**
6. **Use the provided scripts for consistency**

Remember: Most issues are configuration-related. Double-check your setup against the tutorial!