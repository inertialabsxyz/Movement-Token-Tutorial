# Movement Token  Tutorial

### Setup Movement CLI

```shell
git clone https://github.com/movementlabsxyz/aptos-core/ && cd aptos-core`
cargo build -p movement
sudo cp target/debug/movement /usr/local/bin/
```

`movement --version`

Should give -> 

Setup development environment
`movement init`

Select, custom

#### URLs for Bardock

https://full.testnet.movementinfra.xyz/v1
https://faucet.testnet.movementinfra.xyz/

`.movement/config.yaml`

```yaml
---
profiles:
  default:
    network: Custom
    private_key: "0xf6208279efe5a3d3b464ca3ccd53d0c7f977135f3f2bfaf6f139ed146d2d8698"
    public_key: "0x4f79ba761002a999cb6b0d1aa16bf98bdee953f342ad4a44406b0923034d5ea0"
    account: d829639b72f98216951fd146e7d0e747993b8a6a20d29b57841e0d91f0fb2581
    rest_url: "https://full.testnet.movementinfra.xyz/v1"
    faucet_url: "https://faucet.testnet.movementinfra.xyz/"
```

Confirm we have tokens

`movement account balance`

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

If you need more you can run this

`movement account fund-with-faucet`

Build the contract

movement move build

Test the contract

movement move test

Deploy to Bardock(Testnet)
movement move publish --named-addresses movement_token_tutorial=d829639b72f98216951fd146e7d0e747993b8a6a20d29b57841e0d91f0fb2581

```json
Transaction submitted: https://explorer.movementlabs.xyz/txn/0x0b0609b4bf67b136e3437c9a8799e5163d657b7c29baf3936912e589e9bc8d38?network=custom
{
  "Result": {
    "transaction_hash": "0x0b0609b4bf67b136e3437c9a8799e5163d657b7c29baf3936912e589e9bc8d38",
    "gas_used": 5068,
    "gas_unit_price": 100,
    "sender": "d829639b72f98216951fd146e7d0e747993b8a6a20d29b57841e0d91f0fb2581",
    "sequence_number": 0,
    "success": true,
    "timestamp_us": 1755278058956615,
    "version": 24091010,
    "vm_status": "Executed successfully"
  }
}
```

The link above is erroneous and should be:
https://explorer.movementlabs.xyz/txn/0x0b0609b4bf67b136e3437c9a8799e5163d657b7c29baf3936912e589e9bc8d38?network=bardock+testnet

We can see the module deployed with:

`movement account list --query modules`

```json
{
  "Result": [
    {
      "bytecode": "0xa11ceb0b060000000c0100100210300340b10104f10118058902de0207e704be0508a50a4006e50a7510da0bad010a870d0c0c930db8040dcb11060000010101020103010401050106010700080800020a0000030d07010001020e0b000710070002180600021a0600021c0600021d0800032a0200042c070100000009000100000b020100000c030100000f010400001101050000120601000013000100001407010000150301000016080900061e060b00031f0d0e010801200f0f000321100b01080522111201080223140101080524111201080225160101080226180101080327190b0003280b1a010802291a050108032b1c1d00042d011f0100072e200500052f2101000230222300023122240002322225000333222600021328090002342a01010802352c0901080b0c0d0c0e0c0f13100c11131213140c150c171e1f13201303060c05030003060c05080102060c05010b0201080301080401060c04060c05050303060c0305010801050b020108030b020108030608070b02010808060c0105010803020b02010900050101010301060b0201090002050b02010900010b02010808010808030608070b0201090003050b020108030b02010803060c0b02010808060806030608060b020109000801050b020108030b02010803060c0608060b02010808030608060b02010900010206050a02010b0201090006080908070608090c0805080602060c0a020108090104010b0a010900010a02070608090b0a010408040804020804080401060809010805010807010806010c060b020108030b020108030801060800060c0b020108080206080503060b020108030b020108030b02010808060c0b02010808060806040608060b020109000b0201090003050b020108030b020108030b02010808060c060806030608060b02010900030766615f636f696e056572726f720e66756e6769626c655f6173736574066f626a656374066f7074696f6e167072696d6172795f66756e6769626c655f73746f7265067369676e657206737472696e67144d616e6167656446756e6769626c654173736574046275726e0d46756e6769626c654173736574076465706f7369740e667265657a655f6163636f756e74064f626a656374084d657461646174610c6765745f6d6574616461746106537472696e67086765745f6e616d650b696e69745f6d6f64756c65046d696e74087472616e7366657210756e667265657a655f6163636f756e74087769746864726177086d696e745f726566074d696e745265660c7472616e736665725f7265660b5472616e73666572526566086275726e5f726566074275726e5265660d46756e6769626c6553746f72650a616464726573735f6f660869735f6f776e6572117065726d697373696f6e5f64656e6965640e6f626a6563745f616464726573730d7072696d6172795f73746f7265096275726e5f66726f6d1b656e737572655f7072696d6172795f73746f72655f657869737473106465706f7369745f776974685f7265660f7365745f66726f7a656e5f666c6167156372656174655f6f626a6563745f6164647265737311616464726573735f746f5f6f626a656374046e616d650e436f6e7374727563746f72526566136372656174655f6e616d65645f6f626a656374064f7074696f6e046e6f6e6504757466382b6372656174655f7072696d6172795f73746f72655f656e61626c65645f66756e6769626c655f61737365741167656e65726174655f6d696e745f7265661167656e65726174655f6275726e5f7265661567656e65726174655f7472616e736665725f7265660f67656e65726174655f7369676e6572117472616e736665725f776974685f7265661177697468647261775f776974685f726566d829639b72f98216951fd146e7d0e747993b8a6a20d29b57841e0d91f0fb258100000000000000000000000000000000000000000000000000000000000000010a020807464120436f696e0a0203024641030801000000000000000520d829639b72f98216951fd146e7d0e747993b8a6a20d29b57841e0d91f0fb25810a021f1e687474703a2f2f6578616d706c652e636f6d2f66617669636f6e2e69636f0a021312687474703a2f2f6578616d706c652e636f6d126170746f733a3a6d657461646174615f763198010101000000000000000a454e4f545f4f574e4552344f6e6c792066756e6769626c65206173736574206d65746164617461206f776e65722063616e206d616b65206368616e6765732e01144d616e6167656446756e6769626c654173736574010301183078313a3a6f626a6563743a3a4f626a65637447726f757002086765745f6e616d650101000c6765745f6d657461646174610101000002031708051908061b080700010401000a1d11030c030b000a030c040c070a040b07110a3800040c050f0702110c270e0438012b0010000c050b010b0338020c060b050b060b023803020101000100151d11030c030b000a030c040c050a040b05110a3800040c050f0702110c270e0438012b0010010c070b010b0338040c060b070b060b023805020201040100171d11030c020b000a020c030c040a030b04110a3800040c050f0702110c270e0338012b0010010c050b010b0238040c060b050b0608380602030100000b0707030c000e00070111133807020401000001031103380802050000001b250b00070111160c010e010c030a03380907001118070111183108070411180705111811190a03111a0c050a03111b0c020a03111c0c060b03111d0c040e040b050b060b0212002d00020601040100272211030c030b000a030c040c070a040b07110a3800040c050f0702110c270e0438012b000c060b010b0338040c080a0610020b02111e0c050b0610010b080b053805020701040100292211030c040b000a040c050c070a050b07110a3800040c050f0702110c270e0538012b0010010c090b010a0438020c060b020b0438040c080b090b060b080b03380a020801040100171d11030c020b000a020c030c040a030b04110a3800040c050f0702110c270e0338012b0010010c050b010b0238040c060b050b060938060209010001002b1d11030c030b000a030c040c060a040b06110a3800040c050f0702110c270e0438012b0010010c070b020b0338020c050b070b050b01380b0200020001000000",
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

We can now start calling functions on the contract

`movement move view --function-id 0xd829639b72f98216951fd146e7d0e747993b8a6a20d29b57841e0d91f0fb2581::fa_coin::get_name`

```json
{
  "Result": [
    "FA Coin"
  ]
}

```

Mint tokens

`movement move run --function-id 0xd829639b72f98216951fd146e7d0e747993b8a6a20d29b57841e0d91f0fb2581::fa_coin::mint --args address:d829639b72f98216951fd146e7d0e747993b8a6a20d29b57841e0d91f0fb2581 u64:1000`

Transaction submitted: https://explorer.movementlabs.xyz/txn/0x4f1edab56e5af2449f4a18b589dbed8d9b8d4619b86fdffd95b7ff1aa58dcae5?network=custom
{
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

The asset is an object with the identifier

`movement move view --function-id 0xd829639b72f98216951fd146e7d0e747993b8a6a20d29b57841e0d91f0fb2581::fa_coin::get_metadata`

```json
{
  "Result": [
    {
      "inner": "0xa511db8a546837226ef3d171f77153d6540abef681600506899d345cffcb091c"
    }
  ]
}
```

You can view the asset on https://explorer.movementlabs.xyz/fungible_asset/0xa511db8a546837226ef3d171f77153d6540abef681600506899d345cffcb091c?network=bardock+testnet

To view the balance for the account we have minted the tokens to we can use:

`movement move view --function-id 0x1::primary_fungible_store::balance --args address:d829639b72f98216951fd146e7d0e74movement move view --function-id 0x1::primary_fungible_store::balance --args address:d829639b72f98216951fd146e7d0e747993b8a6a20d29b57841e0d91f0fb2581 address:0xa511db8a546837226ef3d171f77153d6540abef681600506899d345cffcb091c --type-args 0x1::fungible_asset::Metadata`

Create another account under profile 'recipient'

`movement init --profile recipient`

```yaml
  recipient:
    network: Custom
    private_key: "0xe4ece15ef02a3d6ce6594ea7911684c09dcefee2d2c8f0b6d96c72702b19c0b1"
    public_key: "0x60c424c8de8a4116553a5753bd657747c381bb95c78efcee1a456b70319d3981"
    account: 67e945e897db25bbea75ed35887162631a0c199214221a14c3e896f6476e94ae
    rest_url: "https://full.testnet.movementinfra.xyz/v1"
    faucet_url: "https://faucet.testnet.movementinfra.xyz/"
```

Transfer 100 to account `67e945e897db25bbea75ed35887162631a0c199214221a14c3e896f6476e94ae`

`movement move run --function-id 0x1::primary_fungible_store::transfer --args address:0xa511db8a546837226ef3d171f77153d6540abef681600506899d345cffcb091c address:67e945e897db25bbea75ed35887162631a0c199214221a14c3e896f6476e94ae u64:100 --type-args 0x1::fungible_asset::Metadata`

We can check the balance for the new account:

` movement move view --function-id 0x1::primary_fungible_store::balance --args address:67e945e897db25bbea75ed35887162631a0c199214221a14c3e896f6476e94ae  address:0xa511db8a546837226ef3d171f77153d6540abef681600506899d345cffcb091c --type-args 0x1::fungible_asset::Metadata`

### Movement Mainnet

We can now deploy the contract to mainnet with the following:

`movement move publish --url https://full.mainnet.movementinfra.xyz/v1 --named-addresses movement_token_tutorial=$MY_ADDRESS --private-key $MY_PRIVATE_KEY`

You can see the transaction here https://explorer.movementlabs.xyz/txn/0x6196bee28badadb57e885c6f9f25dcc77db3e76650660e36570c0ae5c5f5c54e?network=mainnet

We can view the asset id with

`movement move view --function-id $MY_ADDRESS::fa_coin::get_metadata --url https://full.mainnet.movementinfra.xyz/v1`

```json

{
  "Result": [
    {
      "inner": "0xbdfa8e26819b73f5acb4465879255db80bda010dc5d13e28f2c78df7df372f2f"
    }
  ]
}

```

Mint some tokens

`movement move run --function-id $MY_ADDRESS::fa_coin::mint --args address:d829639b72f98216951fd146e7d0e747993b8a6a20d29b57841e0d91f0fb2581 u64:1000 --url https://full.mainnet.movementinfra.xyz/v1 --private-key $MY_PRIVATE_KEY`

and get the balance of them

`movement move view --function-id 0x1::primary_fungible_store::balance --args address:d829639b72f98216951fd146e7d0e747993b8a6a20d29b57841e0d91f0fb2581 address:0xbdfa8e26819b73f5acb4465879255db80bda010dc5d13e28f2c78df7df372f2f --type-args 0x1::fungible_asset::Metadata --url https://full.mainnet.movementinfra.xyz/v1`

```json
{
  "Result": [
    "1000"
  ]
}

```

