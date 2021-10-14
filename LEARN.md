# Testing and understanding NEAR function calls
We’ve seen how to create contracts using NEAR. 

It’s always a good practice to write tests. Not only if you are following a TDD paradigm, but also because deploying to Near blockchain is an expensive operation. It costs some money and also time. You have to deploy it to blockchain, wait for the contract to be deployed and then call the functions that are deployed in that contract. 

All function calls cost some near tokens. This is called a transaction fees. There are 100s of validators around the world who are processing the function calls. In order to have these function calls processed, one must pay the validators for their services.

We’ll see how we can quickly write tests so that we can run these tests without deploying to the blockchain.
## Simple test
First up we’ll create the stub for writing tests. 

```

\#\[cfg(not(target_arch = "wasm32"))\]

\#\[cfg(test)\]

mod tests {

}

```

We’ll write the test cases here. 

Let’s write a test case to check if the user has access to a given account

```

\#\[test\]

fn test_check_access() {

    let contract = NonFungibleTokenBasic::default();

    assert_eq!(false, contract.check_access("madhavan.testnet".to_string()));

}

```

We first initialize the contract and then call the function check_access in this contract. 

To check_access, we send in a parameter the function expects - i.e. the account_id of a user.
## Not just a function call
We’re however not yet ready to run the functions. On near every function call not only has parameters, but also has what is called a context. Near sets up the context for every function call automatically. This context tells who is calling the function, how much money she’s sent to run this function, how much transaction fees she’s paying to execute the code in this function and some more details. 

When we deploy the contract to the blockchain and call functions on that contract using `near call`, the near commandline tool creates this context automatically and attaches it to the function call. The near runtime ensures that the context is legal. It checks that it is sent by the account that claims to have sent it, by verifying the cryptographic signature. The run time also checks for things like, if the user is sending money - the sender has the amount of money that they’re looking to send. 

When we’re running test cases, we need to mimic this context building exercise that Near does under the hood.

```

    use super::\*;

    use near_sdk::MockedBlockchain;

    use near_sdk::{testing_env, VMContext};

    fn get_context(input: Vec, is_view: bool) -> VMContext {

        VMContext {

            current_account_id: "alice_near".to_string(),

            signer_account_id: "bob_near".to_string(),

            signer_account_pk: vec!\[0, 1, 2\],

            predecessor_account_id: "carol_near".to_string(),

            input,

            block_index: 0,

            block_timestamp: 0,

            account_balance: 0,

            account_locked_balance: 0,

            storage_usage: 0,

            attached_deposit: 0,

            prepaid_gas: 10u64.pow(18),

            random_seed: vec!\[0, 1, 2\],

            is_view,

            output_data_receivers: vec!\[\],

            epoch_height: 0,

        }

    }

```

Here, we ask Near to use a MockedBlockchain. 

There are a