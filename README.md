# Movement Token Tutorial

## 🎯 What You'll Learn

This hands-on tutorial will teach you how to:
- Install and configure the Movement CLI
- Understand a fungible asset smart contract
- Build and test Move smart contracts
- Deploy tokens to Movement's Bardock testnet
- Interact with your deployed token using CLI commands
- Deploy to Movement mainnet using environment variables

## 📋 Prerequisites

- Basic command line knowledge
- Git installed on your system
- Rust/Cargo installed (for building Movement CLI)
- A text editor for viewing code

## 🏗️ What We're Building

We'll deploy **FA Coin** - a fungible asset token based on the Aptos fungible asset standard. This token demonstrates:
- Minting new tokens
- Transferring tokens between accounts
- Burning tokens
- Account freezing/unfreezing capabilities

## ⚠️ Important Notes About This Tutorial

### Included Configuration (Reference Only)
This tutorial includes a `.movement/config.yaml` file **for demonstration purposes only**. This shows you what a Movement configuration looks like after running `movement init`. 

**DO NOT use this configuration** - you should create your own by running `movement init`.

### Creating Your Own Projects
When you create your own Movement projects:
1. Run `movement init` to generate your own unique keys and configuration
2. Select "Custom" when prompted for network type
3. Enter the Movement testnet URLs:
   - REST API: `https://full.testnet.movementinfra.xyz/v1`
   - Faucet: `https://faucet.testnet.movementinfra.xyz/`
4. Update `Move.toml` to use Movement's framework:
   ```toml
   [dependencies.AptosFramework]
   git = "https://github.com/movementlabsxyz/aptos-core.git"
   rev = "movement"
   subdir = "aptos-move/framework/aptos-framework"
   ```
   (Not the standard `aptos-labs` repository)

---

## 📚 Tutorial Sections

### Section 1: Install Movement CLI

#### Option A: Build from source (recommended)
```shell
# Clone the Movement-specific Aptos core repository
git clone https://github.com/movementlabsxyz/aptos-core/
cd aptos-core

# Build the Movement CLI
cargo build -p movement

# Install the CLI globally
sudo cp target/debug/movement /usr/local/bin/
```

#### Option B: Download pre-built binaries
Follow the official installation guide for pre-compiled binaries:

📖 **[Movement CLI Quick Install Guide](https://docs.movementnetwork.xyz/devs/movementcli#quick-install-movement-precompiled-binaries)**

This provides platform-specific installation instructions for:
- macOS x86_64  
- macOS ARM64 (Apple Silicon)
- Windows

For other platforms Option A should be followed.

#### Verify installation
```shell
movement --version
```

This would be `aptos 3.5.0` or superior.

---

### Section 2: Understanding the Project Structure

#### Project Layout
```
tutorial/
├── .movement/
│   └── config.yaml       # Pre-configured with testnet keys
├── move/
│   ├── Move.toml         # Package configuration
│   └── sources/
│       └── basic_token.move  # Our FA Coin contract
└── README.md             # This file
```

#### About the Configuration

The included `.movement/config.yaml` file is **for reference only** and shows:
- What a Movement configuration file looks like
- The structure with multiple profiles (`default` and `recipient`)
- Correct network endpoints for Bardock testnet

📝 **Important**: You must create your own configuration by running:
```shell
movement init
```

When prompted:
1. Choose network type: **Custom**
2. Enter REST API URL: `https://full.testnet.movementinfra.xyz/v1`
3. Enter Faucet URL: `https://faucet.testnet.movementinfra.xyz/`

This will generate your own unique keys and account address and fund the account using the Movement Faucet.

Configurations can be named using the `--profile` such as `movement init --profile alice` which would create a new section in `config.yaml` and then can be referenced in future operations using `--profile alice`

---

### Section 3: Build and Test Locally

#### Build the contract
```shell
movement move build
```

This compiles your Move contract and checks for errors.

#### Run tests
```shell
movement move test
```

This executes the test functions in the contract to verify functionality.

---

### Section 4: Deploy to Bardock Testnet

#### Step 1: Check your account balance

```shell
movement account balance
```

**Expected output**:
```json
{
  "Result": [
    {
      "asset_type": "coin",
      "coin_type": "0x1::aptos_coin::AptosCoin",
      "balance": 100000000
    }
  ]
}
```

💡 **Note**: Balance is shown in smallest units (1 MOVE = 100,000,000 units)

#### Step 2: Fund your account (if needed)
```shell
movement account fund-with-faucet
```

This requests testnet tokens from the faucet.

#### Step 3: Deploy your token contract
```shell
movement move publish --named-addresses movement_token_tutorial=d829639b72f98216951fd146e7d0e747993b8a6a20d29b57841e0d91f0fb2581
```

**What this does**:
- Publishes your FA Coin contract to the blockchain
- Uses your account address for the `movement_token_tutorial` named address which is `d829639b72f98216951fd146e7d0e747993b8a6a20d29b57841e0d91f0fb2581` as defined in `config.yaml` for the `Custom` profile.
- Creates a new fungible asset that you control

**Expected output**:
```json
{
  "Result": {
    "transaction_hash": "0x...",
    "gas_used": 5068,
    "gas_unit_price": 100,
    "sender": "d829639b72f98216951fd146e7d0e747993b8a6a20d29b57841e0d91f0fb2581",
    "success": true,
    "vm_status": "Executed successfully"
  }
}
```

✅ **Success indicator**: `"success": true` and `"vm_status": "Executed successfully"`

🔍 **View on Explorer**: 
```
https://explorer.movementlabs.xyz/txn/{YOUR_TRANSACTION_HASH}?network=bardock+testnet
```

#### Step 4: Verify deployment
```shell
movement account list --query modules
```

This shows all modules deployed to your account. You should see the `fa_coin` module listed.

```json
{
  "Result": [
    {
      "bytecode": "...",
      "abi": {
        "address": "0xd829639b72f98216951fd146e7d0e747993b8a6a20d29b57841e0d91f0fb2581",
        "name": "fa_coin",
        "friends": [],
        "exposed_functions": [
          {
            "name": "burn",
            "visibility": "public",
            "is_entry": true,
            "is_view": false,
            "generic_type_params": [],
            "params": [
              "&signer",
              "address",
              "u64"
            ],
            "return": []
          },
          {
            "name": "deposit",
            "visibility": "public",
            "is_entry": false,
            "is_view": false,
            "generic_type_params": [],
            "params": [
              "&signer",
              "address",
              "0x1::fungible_asset::FungibleAsset"
            ],
            "return": []
          },
          {
            "name": "freeze_account",
            "visibility": "public",
            "is_entry": true,
            "is_view": false,
            "generic_type_params": [],
            "params": [
              "&signer",
              "address"
            ],
            "return": []
          },
          {
            "name": "get_metadata",
            "visibility": "public",
            "is_entry": false,
            "is_view": true,
            "generic_type_params": [],
            "params": [],
            "return": [
              "0x1::object::Object<0x1::fungible_asset::Metadata>"
            ]
          },
          {
            "name": "get_name",
            "visibility": "public",
            "is_entry": false,
            "is_view": true,
            "generic_type_params": [],
            "params": [],
            "return": [
              "0x1::string::String"
            ]
          },
          {
            "name": "mint",
            "visibility": "public",
            "is_entry": true,
            "is_view": false,
            "generic_type_params": [],
            "params": [
              "&signer",
              "address",
              "u64"
            ],
            "return": []
          },
          {
            "name": "transfer",
            "visibility": "public",
            "is_entry": true,
            "is_view": false,
            "generic_type_params": [],
            "params": [
              "&signer",
              "address",
              "address",
              "u64"
            ],
            "return": []
          },
          {
            "name": "unfreeze_account",
            "visibility": "public",
            "is_entry": true,
            "is_view": false,
            "generic_type_params": [],
            "params": [
              "&signer",
              "address"
            ],
            "return": []
          },
          {
            "name": "withdraw",
            "visibility": "public",
            "is_entry": false,
            "is_view": false,
            "generic_type_params": [],
            "params": [
              "&signer",
              "u64",
              "address"
            ],
            "return": [
              "0x1::fungible_asset::FungibleAsset"
            ]
          }
        ],
        "structs": [
          {
            "name": "ManagedFungibleAsset",
            "is_native": false,
            "abilities": [
              "key"
            ],
            "generic_type_params": [],
            "fields": [
              {
                "name": "mint_ref",
                "type": "0x1::fungible_asset::MintRef"
              },
              {
                "name": "transfer_ref",
                "type": "0x1::fungible_asset::TransferRef"
              },
              {
                "name": "burn_ref",
                "type": "0x1::fungible_asset::BurnRef"
              }
            ]
          }
        ]
      }
    }
  ]
}

```

---

### Section 5: Interact with Your Token

#### Get token name
```shell
movement move view --function-id 0xd829639b72f98216951fd146e7d0e747993b8a6a20d29b57841e0d91f0fb2581::fa_coin::get_name
```

**Expected output**: `["FA Coin"]`

#### Mint your first tokens

```shell
movement move run --function-id 0xd829639b72f98216951fd146e7d0e747993b8a6a20d29b57841e0d91f0fb2581::fa_coin::mint --args address:d829639b72f98216951fd146e7d0e747993b8a6a20d29b57841e0d91f0fb2581 u64:1000
```

**What this does**: Mints 1000 FA Coin tokens to your account
```json
{
  "Result": {
    "transaction_hash": "0x4f1edab56e5af2449f4a18b589dbed8d9b8d4619b86fdffd95b7ff1aa58dcae5",
    "gas_used": 739,
    "gas_unit_price": 100,
    "sender": "d829639b72f98216951fd146e7d0e747993b8a6a20d29b57841e0d91f0fb2581",
    "sequence_number": 1,
    "success": true,
    "timestamp_us": 1755279061456493,
    "version": 24091123,
    "vm_status": "Executed successfully"
  }
}
```

#### Get the token metadata object
```shell
movement move view --function-id 0xd829639b72f98216951fd146e7d0e747993b8a6a20d29b57841e0d91f0fb2581::fa_coin::get_metadata
```

**Expected output**: 
```json
{
  "Result": [
    {
      "inner": "0xa511db8a546837226ef3d171f77153d6540abef681600506899d345cffcb091c"
    }
  ]
}
```

📝 **Note**: The `inner` value is your token's unique identifier. Save this for checking balances!

🔍 **View on Explorer**:
```
https://explorer.movementlabs.xyz/fungible_asset/{YOUR_TOKEN_ID}?network=bardock+testnet
```

#### Check token balance
```shell
movement move view --function-id 0x1::primary_fungible_store::balance --args address:d829639b72f98216951fd146e7d0e747993b8a6a20d29b57841e0d91f0fb2581 address:0xa511db8a546837226ef3d171f77153d6540abef681600506899d345cffcb091c --type-args 0x1::fungible_asset::Metadata
```

**Command breakdown**:
- `0x1::primary_fungible_store::balance` - Framework function to check token balance
- `address:d829639b72f98216951fd146e7d0e747993b8a6a20d29b57841e0d91f0fb2581` - The account whose balance we're checking (your deployer account)
- `address:0xa511db8a546837226ef3d171f77153d6540abef681600506899d345cffcb091c` - Your token's metadata object ID (from get_metadata)
- `--type-args 0x1::fungible_asset::Metadata` - Specifies we're working with fungible asset metadata

**Expected output**: `["1000"]` (the amount you minted)

💡 **Note**: Replace the addresses with your own:
- Use your account address from your config.yaml
- Use your token ID from the get_metadata result

#### Transfer tokens to another account

To create a recipient account for testing transfers called `alice`:
```shell
movement init --profile alice
```

When prompted, use the same network settings as before:
- Network type: **Custom**
- REST API: `https://full.testnet.movementinfra.xyz/v1`
- Faucet: `https://faucet.testnet.movementinfra.xyz/`

This will create a second profile in your config.yaml with its own keys and address.

**Transfer 100 tokens to Alice**:
```shell
movement move run --function-id 0x1::primary_fungible_store::transfer --args address:0xa511db8a546837226ef3d171f77153d6540abef681600506899d345cffcb091c address:67e945e897db25bbea75ed35887162631a0c199214221a14c3e896f6476e94ae u64:100 --type-args 0x1::fungible_asset::Metadata
```

**Command breakdown**:
- `0x1::primary_fungible_store::transfer` - Framework function for token transfers
- `address:0xa511db8a546837226ef3d171f77153d6540abef681600506899d345cffcb091c` - Your token's metadata object ID (identifies which token to transfer)
- `address:67e945e897db25bbea75ed35887162631a0c199214221a14c3e896f6476e94ae` - Alice's account address (recipient)
- `u64:100` - Amount to transfer (100 tokens)
- `--type-args 0x1::fungible_asset::Metadata` - Type specification for fungible asset operations

💡 **Note**: This transfers tokens FROM your account (the signer) TO Alice's account. Replace with your actual:
- Token ID (from get_metadata)
- Alice's address (from alice profile in config.yaml)

**Verify Alice's balance**:
```shell
movement move view --function-id 0x1::primary_fungible_store::balance --args address:67e945e897db25bbea75ed35887162631a0c199214221a14c3e896f6476e94ae address:0xa511db8a546837226ef3d171f77153d6540abef681600506899d345cffcb091c --type-args 0x1::fungible_asset::Metadata
```

**Command breakdown**:
- `0x1::primary_fungible_store::balance` - Framework function to check token balance
- `address:67e945e897db25bbea75ed35887162631a0c199214221a14c3e896f6476e94ae` - Alice's account address (whose balance we're checking)
- `address:0xa511db8a546837226ef3d171f77153d6540abef681600506899d345cffcb091c` - Your token's metadata object ID (which token to check)
- `--type-args 0x1::fungible_asset::Metadata` - Type specification for fungible asset operations

**Expected output**: `["100"]` (confirming Alice received the tokens)

💡 **Note**: This is the same balance check command as before, but now checking Alice's account instead of yours. Replace with your actual:
- Alice's address (from alice profile in config.yaml)
- Token ID (from get_metadata result)

---

### Section 6: Deploy to Movement Mainnet

⚠️ **Important**: For mainnet deployment, use environment variables to protect your keys!

#### Step 1: Set up environment variables
```bash
export MY_ADDRESS="your_mainnet_address"
export MY_PRIVATE_KEY="your_mainnet_private_key"
```

#### Step 2: Deploy to mainnet
```shell
movement move publish --url https://full.mainnet.movementinfra.xyz/v1 --named-addresses movement_token_tutorial=$MY_ADDRESS --private-key $MY_PRIVATE_KEY
```

🔍 **View on Mainnet Explorer**:
```
https://explorer.movementlabs.xyz/txn/{YOUR_TRANSACTION_HASH}?network=mainnet
```

#### Step 3: Get your mainnet token ID
```shell
movement move view --function-id $MY_ADDRESS::fa_coin::get_metadata --url https://full.mainnet.movementinfra.xyz/v1
```

```json

{
  "Result": [
    {
      "inner": "0xbdfa8e26819b73f5acb4465879255db80bda010dc5d13e28f2c78df7df372f2f"
    }
  ]
}

```

#### Step 4: Mint tokens on mainnet
```shell
movement move run --function-id $MY_ADDRESS::fa_coin::mint --args address:$MY_ADDRESS u64:1000 --url https://full.mainnet.movementinfra.xyz/v1 --private-key $MY_PRIVATE_KEY
```

#### Step 5: Check mainnet balance
```shell
movement move view --function-id 0x1::primary_fungible_store::balance --args address:$MY_ADDRESS address:{YOUR_TOKEN_ID} --type-args 0x1::fungible_asset::Metadata --url https://full.mainnet.movementinfra.xyz/v1
```

**Expected output**: `["1000"]`

---

## 🎉 Congratulations!

You've successfully:
- ✅ Installed the Movement CLI
- ✅ Built and tested a Move smart contract
- ✅ Deployed a fungible token to testnet
- ✅ Minted and transferred tokens
- ✅ Deployed to mainnet with secure key management

## 📖 Additional Resources

- [Contract Overview](docs/CONTRACT_OVERVIEW.md) - Detailed explanation of the FA Coin contract
- [Troubleshooting Guide](docs/TROUBLESHOOTING.md) - Common issues and solutions
- [Advanced Features](docs/ADVANCED.md) - Freeze/unfreeze accounts and more
- [Movement Documentation](https://docs.movementnetwork.xyz/)
- [Movement Explorer](https://explorer.movementlabs.xyz/)

## 🛠️ Quick Reference

### Common Commands
```shell
# Check balance
movement account balance

# Fund account
movement account fund-with-faucet

# Build contract
movement move build

# Run tests
movement move test

# Deploy contract
movement move publish --named-addresses movement_token_tutorial={YOUR_ADDRESS}

# View function (read-only)
movement move view --function-id {ADDRESS}::{MODULE}::{FUNCTION}

# Run function (state-changing)
movement move run --function-id {ADDRESS}::{MODULE}::{FUNCTION} --args {ARGUMENTS}
```

### Contract Functions
- `get_name()` - Returns token name
- `get_metadata()` - Returns token metadata object
- `mint(admin, to, amount)` - Mint new tokens
- `transfer(admin, from, to, amount)` - Transfer tokens
- `burn(admin, from, amount)` - Burn tokens
- `freeze_account(admin, account)` - Freeze an account
- `unfreeze_account(admin, account)` - Unfreeze an account

