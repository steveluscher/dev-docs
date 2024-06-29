# Helius

## RPC Proxy — Protect Your Keys

How to stop your Helius API keys from leaking with 1 click.

A common problem when working with RPCs or APIs on the client side is your API keys leaking. Malicious actors can run up your quota or rate limits if they get access to your keys. Helius does provide access controls for this (for example, you can lockdown your endpoints to only serve requests for certain IPs and Domains) — but using a proxy is the better solution.

We've setup a simple open-source RPC proxy that you can deploy with 1-click to Cloudflare, you can check it out below:

[GitHub - helius-labs/helius-rpc-proxy: This repo hosts a one-click-deploy Cloudflare worker that proxies RPC requests to Helius.](https://github.com/helius-labs/helius-rpc-proxy)



## Priority Fee API

### What Are Priority Fees on Solana? <a href="#what-are-priority-fees-on-solana" id="what-are-priority-fees-on-solana"></a>

Priority fees are an optional fee you can add to your transactions to incentivize block producers (leaders) to include your transaction in the next block. Priority fees are priced in [micro-lamports](https://solana.com/docs/terminology#lamport) per [compute unit](https://solana.com/docs/terminology#compute-units). It is recommended to include a priority fee in your transactions, but how much should you pay?

### Helius Priority Fee API <a href="#helius-priority-fee-api" id="helius-priority-fee-api"></a>

The `getPriorityFeeEstimate` is a new RPC method that provides fee recommendations based on historical data. Most importantly, _it considers both global and local fee markets_.

For example, say you wanted to submit a swap on Jupiter. You could make the following request:

Copy

```
{
    "jsonrpc": "2.0",
    "id": "helius-example",
    "method": "getPriorityFeeEstimate",
    "params": [{
        "accountKeys": ["JUP6LkbZbjS1jKKwapdHNy74zcZ3tLUZoi5QNyVTaV4"],
        "options": {
            "recommended": true
        }
    }]
}
```

And the API will provide you with a recommended fee:

Copy

```
{
    "jsonrpc": "2.0",
    "result": {
        "priorityFeeEstimate": 33549.0
    },
    "id": "helius-example"
}
```

The estimate is priced in micro-lamports per compute unit.

You can also provide a serialized transaction instead of manually specifying the accounts. For example:

Copy

```
{
    "jsonrpc": "2.0",
    "id": "helius-example",
    "method": "getPriorityFeeEstimate",
    "params": [{
        "transaction": "AtP+61DWChFJcicQqntFaf...",
        "options": {
            "recommended": true,
            "transactionEncoding": "base64"
        }
    }]
}
```

### How it Works <a href="#how-it-works" id="how-it-works"></a>

The method uses a set of predefined priority levels (percentiles) to dictate the returned estimate. Users can optionally specify to receive all the priority levels and adjust the window with which these are calculated via `lookbackSlots`

Copy

```
fn get_recent_priority_fee_estimate(request: GetPriorityFeeEstimateRequest) -> GetPriorityFeeEstimateResponse

struct GetPriorityFeeEstimateRequest {
  transaction: Option<String>,   // estimate fee for a serialized txn
  account_keys: Option<Vec<String>>, // estimate fee for a list of accounts
  options: Option<GetPriorityFeeEstimateOptions>
}

struct GetPriorityFeeEstimateOptions {
	priority_level: Option<PriorityLevel>, // Default to MEDIUM
	include_all_priority_fee_levels: Option<bool>, // Include all priority level estimates in the response
	transaction_encoding: Option<UiTransactionEncoding>, // Default Base58
	lookback_slots: Option<u8>, // The number of slots to look back to calculate the estimate. Valid numbers are 1-150, default is 150
	recommended: Option<bool>, // The Helius recommended fee for landing transactions
	include_vote: Option<bool>, // Include vote transactions in the priority fee estimate calculation. Default to true 
}

enum PriorityLevel {
	MIN, // 0th percentile
	LOW, // 25th percentile
	MEDIUM, // 50th percentile
	HIGH, // 75th percentile
	VERY_HIGH, // 95th percentile
  // labelled unsafe to prevent people from using and draining their funds by accident
	UNSAFE_MAX, // 100th percentile 
	DEFAULT, // 50th percentile
}

struct GetPriorityFeeEstimateResponse {
  priority_fee_estimate: Option<MicroLamportPriorityFee>
  priority_fee_levels: Option<MicroLamportPriorityFeeLevels>
}

type MicroLamportPriorityFee = f64

struct MicroLamportPriorityFeeLevels {
	min: f64,
	low: f64,
	medium: f64,
	high: f64,
	very_high: f64,
	unsafe_max: f64,
}
```

### Examples <a href="#examples" id="examples"></a>

**Request all priority fee levels for Jup v6**

Copy

```
{
    "jsonrpc": "2.0",
    "id": "1",
    "method": "getPriorityFeeEstimate",
    "params": [{
        "accountKeys": ["JUP6LkbZbjS1jKKwapdHNy74zcZ3tLUZoi5QNyVTaV4"],
        "options": {
            "includeAllPriorityFeeLevels": true
        }
    }]
}
```

Response

Copy

```
{
    "jsonrpc": "2.0",
    "result": {
        "priorityFeeLevels": {
            "min": 0.0,
            "low": 2.0,
            "medium": 10082.0,
            "high": 100000.0,
            "veryHigh": 1000000.0,
            "unsafeMax": 50000000.0
        }
    },
    "id": "1"
}
```

**Request the high priority level for Jup v6**

Copy

```
{
    "jsonrpc": "2.0",
    "id": "1",
    "method": "getPriorityFeeEstimate",
    "params": [{
        "accountKeys": ["JUP6LkbZbjS1jKKwapdHNy74zcZ3tLUZoi5QNyVTaV4"],
        "options": {
            "priority_level": "HIGH"
        }
    }]
}
```

Response

Copy

```
{
    "jsonrpc": "2.0",
    "result": {
        "priorityFeeEstimate": 1200.0
    },
    "id": "1"
}
```

### Sending a transaction with the Priority Fee API (Javascript) <a href="#sending-a-transaction-with-the-priority-fee-api-javascript" id="sending-a-transaction-with-the-priority-fee-api-javascript"></a>

This code snippet showcases how one can transfer SOL from one account to another. In this code, the transaction is passed to the priority fee API which then determines the specified priority fee from all the accounts involved in the transaction.

Copy

```
const {
  Connection,
  SystemProgram,
  Transaction,
  sendAndConfirmTransaction,
  Keypair,
  ComputeBudgetProgram,
} = require("@solana/web3.js");
const bs58 = require("bs58");

const HeliusURL = "https://mainnet.helius-rpc.com/?api-key=<YOUR_API_KEY>";
const connection = new Connection(HeliusURL);
const fromKeypair = Keypair.fromSecretKey(Uint8Array.from("[Your secret key]")); // Replace with your own private key
const toPubkey = "CckxW6C1CjsxYcXSiDbk7NYfPLhfqAm3kSB5LEZunnSE"; // Replace with the public key that you want to send SOL to

async function getPriorityFeeEstimate(priorityLevel, transaction) {
  const response = await fetch(HeliusURL, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      jsonrpc: "2.0",
      id: "1",
      method: "getPriorityFeeEstimate",
      params: [
        {
          transaction: bs58.encode(transaction.serialize()), // Pass the serialized transaction in Base58
          options: { priorityLevel: priorityLevel },
        },
      ],
    }),
  });
  const data = await response.json();
  console.log(
    "Fee in function for",
    priorityLevel,
    " :",
    data.result.priorityFeeEstimate
  );
  return data.result;
}
async function sendTransactionWithPriorityFee(priorityLevel) {
  const transaction = new Transaction();
  const transferIx = SystemProgram.transfer({
    fromPubkey: fromKeypair.publicKey,
    toPubkey,
    lamports: 100,
  });
  transaction.add(transferIx);

  transaction.recentBlockhash = (
    await connection.getLatestBlockhash()
  ).blockhash;
  transaction.sign(fromKeypair);

  let feeEstimate = { priorityFeeEstimate: 0 };
  if (priorityLevel !== "NONE") {
    feeEstimate = await getPriorityFeeEstimate(priorityLevel, transaction);
    const computePriceIx = ComputeBudgetProgram.setComputeUnitPrice({
      microLamports: feeEstimate.priorityFeeEstimate,
    });
    transaction.add(computePriceIx);
  }

  try {
    const txid = await sendAndConfirmTransaction(connection, transaction, [
      fromKeypair,
    ]);
    console.log(`Transaction sent successfully with signature ${txid}`);
  } catch (e) {
    console.error(`Failed to send transaction: ${e}`);
  }
}

sendTransactionWithPriorityFee("High"); // Choose between "Min", "Low", "Medium", "High", "VeryHigh", "UnsafeMax"
```

#### (Appendix) Calculating the Percentiles <a href="#appendix-calculating-the-percentiles" id="appendix-calculating-the-percentiles"></a>

To calculate the percentiles, we need to consider the global and local fee market over transactions in the last N slots

For example,

Copy

```
priority_estimate(p: Percentile, accounts: Accounts) = 
	max(percentile(txn_fees, p), percentile(account_fees(accounts), p))
```

where txn\_fees are the txn\_fees from the last 150 blocks, and account\_fees(accounts) are the fees for transactions containing these accounts from the last 150 blocks. Here, we are considering the total set of fees seen for accounts and transactions as opposed to the minimum.

**Global Fee Market Estimate**

The global fee market estimate is a percentile of priority fees paid for transactions in the last N slots.

**Local Fee Market Estimate**

The local fee market is influenced by the number of people trying to obtain a lock on an account. We can estimate this similarly to the global fee market but instead use the percentile of fees paid for transactions involving a given account(s). If a user requests an estimate for multiple accounts in the same transaction, we will take the max of the percentiles across those accounts.

**Priority Fee Estimate**

The `priority_fee_estimate` will be the max of the global and local fee market estimates.

#### Extensions <a href="#extensions" id="extensions"></a>

This method could also be integrated into `simulateTransaction` and returned with the response context. This way, developers using `simulateTransaction` can eliminate an extra RPC call.



## Sending Transactions on Solana

Optimize your transactions to minimize confirmation latency and maximize delivery rates.

**Helius' staked connections ensure almost 100% transaction delivery and minimize confirmation times.**

### Summary <a href="#summary" id="summary"></a>

Solana has recently been congested. Ensuring your transactions are delivered and included in confirmed blocks (landed) can be hard. Optimize your landing rates by following these best practices:

1. Use staked connections.
2. Set competitive priority fees.
3. Set minimal compute units.
4. Rebroadcast transactions until confirmation.

We offer two forms of staked connections:

1. **Shared staked connections**
   * Recommended for everyday users, businesses, and startups.
   * Accessible to all **paid** plans if their priority fees meet or exceed the `recommended` value provided by the [Priority Fee API](https://docs.helius.dev/solana-rpc-nodes/priority-fee-api).
2. **Dedicated staked connections**
   * Guarantees staked connection bandwidth for additional peace of mind.
   * Recommended for enterprises, quants, and trading firms.
   * Click [here](https://form.typeform.com/to/STE2XEDw) to contact sales.

We have written a guide on optimizing your priority fees, compute units, retry strategy, and access to staked connections below. We wrote an SDK in NodeJS and Rust for your convenience.

Want to go deeper? We cover all fundamentals in this [blog post](https://www.helius.dev/blog/how-to-land-transactions-on-solana) if you prefer to optimize transactions yourself.

### Best Practices <a href="#best-practices" id="best-practices"></a>

We recommend the following:

* [Optimizing a transaction's compute unit (CU) usage](https://solana.com/developers/guides/advanced/how-to-optimize-compute)
  * [As of 1.18, transactions requiring fewer CUs will be given higher priority](https://github.com/solana-labs/solana/pull/34888). In our Node.js SDK, the [`getComputeUnits` helper method](https://github.com/helius-labs/helius-sdk/blob/018be8486c85cdd0e7afec572ef4d3d137a81bc0/src/RpcClient.ts#L436-L472) can be used to simulate a transaction based on a set of instructions and address lookup tables, returning the amount of CUs consumed. Similarly, our Rust SDK has the [`get_compute_units` helper method](https://github.com/helius-labs/helius-rust-sdk/blob/f0fa49e35057b433f9bd56cc75a1190aad1f8b8e/src/optimized\_transaction.rs#L29-L75) providing the same functionality.
* Implementing [priority fees](https://www.helius.dev/blog/priority-fees-understanding-solanas-transaction-fee-mechanics) and [calculating them dynamically](https://docs.helius.dev/solana-rpc-nodes/priority-fee-api)
* [Implementing robust retry logic](https://www.helius.dev/blog/how-to-land-transactions-on-solana#implement-a-robust-retry-logic)
* Sending transactions via [staked connections](https://www.helius.dev/blog/stake-weighted-quality-of-service-everything-you-need-to-know)
  * Paid plans that set their priority fees using the "recommended" value from the [Priority Fee API](https://docs.helius.dev/solana-rpc-nodes/priority-fee-api) will be routed to staked connections.
* Using [durable nonces](https://solana.com/developers/guides/advanced/introduction-to-durable-nonces) for transactions that are time-insensitive

### Sending Smart Transactions <a href="#sending-smart-transactions" id="sending-smart-transactions"></a>

Both the Helius [Node.js](https://www.npmjs.com/package/helius-sdk/v/1.3.1?activeTab=readme) and [Rust](https://crates.io/crates/helius) SDKs can send smart transactions. This new method builds and sends an optimized transaction while handling its confirmation status. Users can configure the [transaction's send options](https://solana.com/docs/rpc/http/sendtransaction#parameters), such as whether the transaction should skip preflight checks.

At the most basic level, users must supply their keypair and the instructions they wish to execute, and we handle the rest.

We:

* Fetch the latest blockhash
* Build the initial transaction
* Simulate the initial transaction to fetch the compute units consumed
* Set the compute unit limit to the compute units consumed in the previous step, with some margin
* Get the Helius recommended priority fee via our [Priority Fee API](https://docs.helius.dev/solana-rpc-nodes/priority-fee-api)
* Set the priority fee (microlamports per compute unit) as the Helius recommended fee
  * Adds a small safety buffer fee in case the recommended value changes in the next few seconds
* Build and send the optimized transaction
* Return the transaction signature, if successful

Requiring the recommended value (or higher) for our staked connections ensures that Helius sends high-quality transactions and that we wont be rate-limited by validators.

This method is designed to be the easiest way to build, send, and land a transaction on Solana. Note that by using the Helius recommended fee, _transactions sent by **Helius users on one of our paid shared plans will be routed through our staked connections, guaranteeing near 100% transaction delivery**_**.**

#### Node.js SDK <a href="#node.js-sdk" id="node.js-sdk"></a>

The `sendSmartTransaction` method is available in our [Helius Node.js SDK](https://github.com/helius-labs/helius-sdk) for [versions >= 1.3.2](https://www.npmjs.com/package/helius-sdk/v/1.3.2?activeTab=readme). To update to a more recent version of the SDK, run `npm update helius-sdk`.

The following example transfers SOL to an account of your choice. It leverages `sendSmartTransaction` to send an optimized transaction that does not skip preflight checks

Copy

```
import { Helius } from "helius-sdk";
import {
  Keypair,
  SystemProgram,
  LAMPORTS_PER_SOL,
  TransactionInstruction,
} from "@solana/web3.js";

const helius = new Helius("YOUR_API_KEY");

const fromKeypair = /* Your keypair goes here */;
const fromPubkey = fromKeypair.publicKey;
const toPubkey = /* The person we're sending 0.5 SOL to */;

const instructions: TransactionInstruction[] = [
  SystemProgram.transfer({
    fromPubkey: fromPubkey,
    toPubkey: toPubkey,
    lamports: 0.5 * LAMPORTS_PER_SOL, 
  }),
];

const transactionSignature = await helius.rpc.sendSmartTransaction(instructions, [fromKeypair]);
console.log(`Successful transfer: ${transactionSignature}`);
```

#### Rust SDK <a href="#rust-sdk" id="rust-sdk"></a>

The `send_smart_transaction` method is available in our [Rust SDK](https://github.com/helius-labs/helius-rust-sdk) for [versions >= 0.1.5](https://crates.io/crates/helius/versions). To update to a more recent version of the SDK, run `cargo update helius`.

The following example transfers 0.01 SOL to an account of your choice. It leverages `send_smart_transaction` to send an optimized transaction that skips preflight checks and retries twice, if necessary:

Copy

```
use helius::types::*;
use helius::Helius;
use solana_sdk::{
    pubkey::Pubkey,
    signature::Keypair,
    system_instruction
};

#[tokio::main]
async fn main() {
    let api_key: &str = "YOUR_API_KEY";
    let cluster: Cluster = Cluster::MainnetBeta;
    let helius: Helius = Helius::new(api_key, cluster).unwrap();
    
    let from_keypair: Keypair = /* Your keypair goes here */;
    let from_pubkey: Pubkey = from_keypair.pubkey();
    let to_pubkey: Pubkey = /* The person we're sending 0.01 SOL to */;

    // Create a simple instruction (transfer 0.01 SOL from from_pubkey to to_pubkey)
    let transfer_amount = 100_000; // 0.01 SOL in lamports
    let instruction = system_instruction::transfer(&from_pubkey, &to_pubkey, transfer_amount);

    // Create the SmartTransactionConfig
    let config = SmartTransactionConfig {
        instructions,
        signers: vec![&from_keypair],
        send_options: RpcSendTransactionConfig {
            skip_preflight: true,
            preflight_commitment: None,
            encoding: None,
            max_retries: Some(2),
            min_context_slot: None,
        },
        lookup_tables: None,
    };

    // Send the optimized transaction
    match helius.send_smart_transaction(config).await {
        Ok(signature) => {
            println!("Transaction sent successfully: {}", signature);
        }
        Err(e) => {
            eprintln!("Failed to send transaction: {:?}", e);
        }
    }
}
```

### Sending Transactions Without the SDK <a href="#sending-transactions-without-the-sdk" id="sending-transactions-without-the-sdk"></a>

We recommend sending smart transactions with one of our SDKs but the same functionality can be achieved without using one. Both the [Node.js SDK](https://github.com/helius-labs/helius-sdk/blob/main/src/RpcClient.ts) and [Rust SDK](https://github.com/helius-labs/helius-rust-sdk/blob/dev/src/optimized\_transaction.rs) are open-source, so the underlying code for the send smart transaction functionality can be viewed anytime.

#### Prepare and Build the Initial Transaction <a href="#prepare-and-build-the-initial-transaction" id="prepare-and-build-the-initial-transaction"></a>

First, prepare and build the initial transaction. This includes creating a new transaction with a set of instructions, adding the recent blockhash, and assigning a fee payer. For versioned transactions, create a `TransactionMessage` and compile it with lookup tables if any are present. Then, create a new versioned transaction and sign it — this is necessary for the next step when we simulate the transaction, as the transaction must be signed.

For example, if we wanted to prepare a versioned transaction:

Copy

```
// Prepare your instructions and set them to an instructions variable
// The payerKey is the public key that will be paying for this transaction
// Prepare your lookup tables and set them to a lookupTables variable

let recentBlockhash = (await this.connection.getLatestBlockhash()).blockhash;

const v0Message = new TransactionMessage({
    instructions: instructions,
    payerKey: pubKey,
    recentBlockhash: recentBlockhash,
}).compileToV0Message(lookupTables);

versionedTransaction = new VersionedTransaction(v0Message);
versionedTransaction.sign([fromKeypair]);
```

#### Optimize the Transaction's Compute Unit (CU) Usage <a href="#optimize-the-transactions-compute-unit-cu-usage" id="optimize-the-transactions-compute-unit-cu-usage"></a>

To optimize the transaction's compute unit (CU) usage, we can use the [`simulateTransaction` RPC method](https://solana.com/docs/rpc/http/simulatetransaction) to simulate the transaction. Simulating the transaction will return the amount of CUs used, so we can use this value to set our compute limit accordingly. It's recommended to use a test transaction with the desired instructions first, plus an instruction that sets the compute limit to 1.4m CUs. This is done to ensure the transaction simulation succeeds. For example:

Copy

```
const testInstructions = [
    ComputeBudgetProgram.setComputeUnitLimit({ units: 1_400_000 }),
    ...instructions,
];

const testTransaction = new VersionedTransaction(
    new TransactionMessage({
        instructions: testInstructions,
        payerKey: payer,
        recentBlockhash: (await this.connection.getLatestBlockhash()).blockhash,
    }).compileToV0Message(lookupTables)
);

const rpcResponse = await this.connection.simulateTransaction(testTransaction, {
    replaceRecentBlockhash: true,
    sigVerify: false,
});

const unitsConsumed = rpcResponse.value.unitsConsumed;
```

It is also recommended to add a bit of margin to ensure the transaction executes without any issues. We can do so by setting the following:

Copy

```
let customersCU = Math.ceil(unitsConsumed * 1.1);
```

Then, create an instruction that sets the compute unit limit to this value and add it to your array of instructions:

Copy

```
const computeUnitIx = ComputeBudgetProgram.setComputeUnitLimit({
    units: customersCU
});

instructions.push(computeUnitIx);
```

#### Serialize and Encode the Transaction <a href="#serialize-and-encode-the-transaction" id="serialize-and-encode-the-transaction"></a>

This is relatively straightforward. First, to serialize the transaction, both `Transaction` and `VersionedTransaction` types have a `.serialize()` method. Then use the [bs58 package](https://www.npmjs.com/package/bs58) to encode the transaction. Your code should look something like `bs58.encode(txt.serialize());`

#### Setting the Right Priority Fee <a href="#setting-the-right-priority-fee" id="setting-the-right-priority-fee"></a>

First, use the [Priority Fee API](https://docs.helius.dev/solana-rpc-nodes/priority-fee-api) to get the priority fee estimate. We want to pass in our transaction and get the Helius recommended fee via the `recommended` parameter:

Copy

```
const response = await fetch(HeliusURL, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
        jsonrpc: "2.0",
        id: "1",
        method: "getPriorityFeeEstimate",
        params: [
            {
                transaction: bs58.encode(versionedTransaction), // Pass the serialized transaction in
                options: { recommended: true },
            },
        ],   
    }),
});

const data = await response.json();
const priorityFeeRecommendation = data.result.priorityFeeEstimate;
```

Then, create an instruction that sets the compute unit price to this value, and add that instruction to your previous instructions:

Copy

```
const computeBudgetIx = ComputeBudgetProgram.setComputeUnitPrice({
    microLamports: priorityFeeRecommendation,
});

instructions.push(computeBudgetIx);
```

#### Build and Send the Optimized Transaction <a href="#build-and-send-the-optimized-transaction" id="build-and-send-the-optimized-transaction"></a>

This step is almost a repeat of the first step. However, the array of initial instructions has been altered to add two instructions to set the compute unit limit and price optimally. Now, send the transaction. It doesn't matter if you send with or without preflight checks or change any other send options — the transaction will be routed through our staked connections.

#### Polling the Transaction's Status and Rebroadcasting <a href="#polling-the-transactions-status-and-rebroadcasting" id="polling-the-transactions-status-and-rebroadcasting"></a>

While staked connections will forward a transaction directly to the leader, it is still possible for the transaction to be dropped in the Banking Stage. It is recommended that users employ their own rebroadcasting logic rather than rely on the RPC to retry the transaction for them.

The [`sendTransaction` RPC method](https://solana.com/docs/rpc/http/sendtransaction) has a `maxRetries` parameter that can be set to override the RPC's default retry logic, giving developers more control over the retry process. It is a common pattern to fetch the current blockhash via [`getLatestBlockhash`](https://solana.com/docs/rpc/http/getlatestblockhash), store the `lastValidBlockHeight`, and retry the transaction until the blockhash expires. It is crucial to only re-sign a transaction when the blockhash is no longer valid, or else it is possible for both transactions to be accepted by the network.

Once a transaction is sent, it is important to poll its confirmation status to see whether the network has processed and confirmed it before retrying. Use the [`getSignatureStatuses` RPC method](https://solana.com/docs/rpc/http/getsignaturestatuses) to check a list of transactions' confirmation status. The [@solana/web3.js SDK](https://solana-labs.github.io/solana-web3.js/) also has a [`getSignatureStatus` method](https://solana-labs.github.io/solana-web3.js/classes/Connection.html#getSignatureStatus) on its [`Connection` class](https://solana-labs.github.io/solana-web3.js/classes/Connection.html) to fetch the current status of a given signature.

**How `sendSmartTransaction` Handles Polling and Rebroadcasting**

The `sendSmartTransaction` method has a timeout period of 60 seconds. Since a blockhash is valid for 150 slots, and assuming perfect 400ms slots, we can reasonably assume a transaction's blockhash will be invalid after one minute. The method sends the transaction and polls its transaction signature using this timeout period:

Copy

```
try {
   // Create a smart transaction
   const transaction = await this.createSmartTransaction(instructions, signers, lookupTables, sendOptions);
  
   const timeout = 60000;
   const startTime = Date.now();
   let txtSig;
  
   while (Date.now() - startTime < timeout) {
     try {
       txtSig = await this.connection.sendRawTransaction(transaction.serialize(), {
         skipPreflight: sendOptions.skipPreflight,
         ...sendOptions,
       });
  
       return await this.pollTransactionConfirmation(txtSig);
     } catch (error) {
       continue;
     }
   }
} catch (error) {
   throw new Error(`Error sending smart transaction: ${error}`);
}
```

`txtSig` is set to the signature of the transaction that was just sent. The method then uses the `pollTransactionConfirmation()` method to poll the transaction's confirmation status. This method checks a transaction's status every five seconds for a maximum of three times. If the transaction is not confirmed during this time, an error is returned:

Copy

```
async pollTransactionConfirmation(txtSig: TransactionSignature): Promise<TransactionSignature> {
    // 15 second timeout
    const timeout = 15000;
    // 5 second retry interval
    const interval = 5000;
    let elapsed = 0;

    return new Promise<TransactionSignature>((resolve, reject) => {
      const intervalId = setInterval(async () => {
        elapsed += interval;

        if (elapsed >= timeout) {
          clearInterval(intervalId);
          reject(new Error(`Transaction ${txtSig}'s confirmation timed out`));
        }

        const status = await this.connection.getSignatureStatus(txtSig);

        if (status?.value?.confirmationStatus === "confirmed") {
          clearInterval(intervalId);
          resolve(txtSig);
        }
      }, interval);
   });
}
```

We continue sending the transaction, polling its confirmation status, and retrying it until a minute has elapsed. If the transaction has not been confirmed at this time, an error is thrown.



## What are Webhooks?

Setup powerful event-driven workflows in seconds.

#### What are Webhooks? <a href="#what-are-webhooks" id="what-are-webhooks"></a>

Webhooks let you listen to on-chain Solana events (e.g., sales, listings, swaps) and trigger actions when these events happen. Rather than continuously polling blocks, transactions, or accounts on the blockchain — our webhooks serve on-chain events to any URL that you provide as soon as they happen on-chain.

No need to worry about setting up and maintaining expensive & performant infrastructure to keep up with the latest transactions happening on-chain — we provide you both programmatic API access and a front-end UI to create and manage your webhooks. Here's a quick video to demonstrate.

Note: It may take up to **2 minutes** for webhook changes to take effect!

#### Types of Webhooks <a href="#types-of-webhooks" id="types-of-webhooks"></a>

We currently have multiple types of webhooks, these are:

* Enhanced transaction webhooks (trigger updates when a certain type of transaction happens, for example an NFT sale, for the addresses your're watching)
* Raw transaction webhooks (trigger updates when any transaction happens for the addresses you're watching)
* Discord webhooks (stream updates directly to a Discord channel)

#### Automatic Event Detection <a href="#automatic-event-detection" id="automatic-event-detection"></a>

Helius webhooks aren't just standard webhooks — we also give developers the power to select from a number of pre-configured event types. These event types include NFT sales, NFT listings, DeFi swaps, DAO votes, NFT mints, balance changes, and much more. For a more comprehensive list of supported types, please check out [Transaction Types](https://docs.helius.dev/resources/transaction-types). Whereas before you had to build your own parsers and reverse engineer smart contracts on-chain to detect certain events, we do the hard work for you.



Currently we offer both transactions webhooks (listening to transactions for a set of accounts) and account change webhooks (listen to account changes for a set of accounts).

Helius lets you interact with webhooks in three ways.

#### Helius UI (Recommended) <a href="#helius-ui-recommended" id="helius-ui-recommended"></a>

If you'd rather not bother with code and want additional methods for viewing logs and sending test webhook events, the Helius UI is what you need. The UI can be accessed at [dev.helius.xyz](https://dev.helius.xyz/).

#### Helius REST API <a href="#helius-rest-api" id="helius-rest-api"></a>

If you're not working with Typescript or Javascript, you'll need to interact with our webhooks through REST:

[📘PAGEAPI Reference](https://docs.helius.dev/webhooks-and-websockets/api-reference)

#### Helius SDK <a href="#helius-sdk" id="helius-sdk"></a>

The easiest (and most fun) way to interact with Helius webhooks is to use the official SDK:

[GitHub - helius-labs/helius-sdkGitHub](https://github.com/helius-labs/helius-sdk#webhooks)

The SDK contains abstractions to enhance what webhooks have to offer, including the ability to create collection webhooks (i.e., a webhook that tracks all the NFTs within an NFT collection)!

#### Example Uses <a href="#example-uses" id="example-uses"></a>

* **Bots**
  * When an NFT is listed on marketplace X, trigger an "nft buy" action.
  * When a margin position is unhealthy, trigger a "liquidation" action.
* **Monitoring & Alerts**
  * When a program emits a certain log, trigger PagerDuty integration.
  * When a token account balance changes by more than X%, use Dialect to communicate a warning action.
* **Event-driven Indexing**
  * When any transaction occurs for a given program, send it directly to your database or backend.
* **Notifications & Activity Tracking**
  * When a transfer occurs from wallet X to wallet Y — send a Slack notification or email.
* **Analytics & Logs**
  * When event X happens, send it to an ETL pipeline or persist it directly on Helius to view trends over time.
* **Workflow Automation**
  * When event X happens, trigger any set of actions.

## API Reference

Learn how to use Helius webhooks.

For any questions or help regarding our Webhooks, please ask us for help on [Discord](http://discord.gg/6GXdee3gBj)!

Webhook events are charged at 1 credit. To edit/add/delete a webhook via the API, this will cost 100 credits/request.

#### (NEW) Helius Mock Server for Local Webhook Testing <a href="#new-helius-mock-server-for-local-webhook-testing" id="new-helius-mock-server-for-local-webhook-testing"></a>

#### Endpoints <a href="#endpoints" id="endpoints"></a>

[PAGECreate Webhook](https://docs.helius.dev/webhooks-and-websockets/api-reference/create-webhook)[PAGEGet All Webhooks](https://docs.helius.dev/webhooks-and-websockets/api-reference/get-all-webhooks)[PAGEGet Webhook](https://docs.helius.dev/webhooks-and-websockets/api-reference/get-webhook)[PAGEEdit Webhook](https://docs.helius.dev/webhooks-and-websockets/api-reference/edit-webhook)[PAGEDelete Webhook](https://docs.helius.dev/webhooks-and-websockets/api-reference/delete-webhook)



## Create Webhook

Programatically create a Helius webhook.

Note: It may take up to **2 minutes** for webhook changes to take effect!

#### POST /webhooks <a href="#post-webhooks" id="post-webhooks"></a>

Creates a webhook given account addresses, transaction types, and a webhook URL. You can optionally provide an authorization header to verify that the webhook came from Helius.

**Important!** For a full list of supported `transactionTypes —`please see [Transaction Types](https://docs.helius.dev/resources/transaction-types)

If you'd like a raw transaction payload instead of our enhanced transaction object, please input "raw" for the `webhookType`. If you'd like the enhanced version, input "enhanced". Note that raw transactions have much lower latencies due to us not parsing the event types.

### Creates a webhook.

POSThttps://api.helius.xyz/v0/webhooksQuery parametersapi-key\*string

The api key.

Bodyapplication/jsonwebhookURLstringtransactionTypesarray of TransactionType (enum)accountAddressesarray of stringwebhookTypestringauthHeaderstringResponse200

The created webhook.

Bodyapplication/jsonwebhookIDstringwalletstringwebhookURLstringtransactionTypesarray of TransactionType (enum)accountAddressesarray of stringwebhookTypestringauthHeaderstringRequestJavaScriptCurlCopy

```
const response = await fetch('https://api.helius.xyz/v0/webhooks', {
    method: 'POST',
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify({}),
});
const data = await response.json();
```

ResponseCopy

```
{
  "webhookID": "text",
  "wallet": "text",
  "webhookURL": "text",
  "transactionTypes": [
    "UNKNOWN"
  ],
  "accountAddresses": [
    "text"
  ],
  "webhookType": "text",
  "authHeader": "text"
}
```

#### Webhook Types <a href="#webhook-types" id="webhook-types"></a>

Raw Transactions (Sample Payload)

```
{
     "webhookURL": "https://TestServer.test.repl.co/webhooks",
     "transactionTypes": ["Any"],
     // Use ["ACCOUNT_ADDRESS", "ACCOUNT_ADDRESS"] for multiple accountAddresses. 
     "accountAddresses": ["ACCOUNT_ADDRESS"],
     "webhookType": "raw", // "rawDevnet"
     "txnStatus": "all", // success/failed
     "authHeader": "<Optional_AuthHeader>"
}
```

**Enhanced Transactions (Sample Payload)**

```
{
     "webhookURL": "https://TestServer.test.repl.co/webhooks",
     "transactionTypes": ["NFT_SALE"],
     // Use ["ACCOUNT_ADDRESS", "ACCOUNT_ADDRESS"] for multiple accountAddresses. 
     "accountAddresses": ["ACCOUNT_ADDRESS"],
     "webhookType": "enhanced", // "enhancedDevnet"
     "authHeader": "<Optional_AuthHeader>"
}
```

**Discord Transactions (Sample Payload)**

```
{
     "webhookURL": "https://discord.com/api/webhooks/<WebhookID>/<TokenID>",
     "transactionTypes": ["NFT_SALE"],
     // Use ["ACCOUNT_ADDRESS", "ACCOUNT_ADDRESS"] for multiple accountAddresses. 
     "accountAddresses": ["ACCOUNT_ADDRESS"], 
     "webhookType": "discord" // "discordDevnet"
}
```

#### Basic Example <a href="#basic-example" id="basic-example"></a>

```
const createWebhook = async () => {
    try {
      const response = await fetch(
        "https://api.helius.xyz/v0/webhooks?api-key=<PASTE YOUR API KEY HERE>",
        {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({
          "webhookURL": "https://TestServer.test.repl.co/webhooks",
          "transactionTypes": ["Any"],
          "accountAddresses": ["2PP32Vmuzgo1UC847Yzdd9CkowXWhMLrJ47Gfr4KDAyN"],
          "webhookType": "raw", // "rawDevnet"
          "txnStatus": "all", // success/failed
       }),
        }
      );
      const data = await response.json();
      console.log({ data });
    } catch (e) {
      console.error("error", e);
    }
  };
  createWebhook();
```

#### Testing Environments <a href="#testing-environments" id="testing-environments"></a>

**Here are some testing environments that are quick to set up for posting webhook events:**

[Webhook.site - Test, process and transform emails and HTTP requests](https://webhook.site/)

[TypedWebhook.tools: a tool to test webhooks and generate payload types](https://typedwebhook.tools/)



## Payload/Response

Payload for Webhook types available.

#### Payload <a href="#payload" id="payload"></a>

**Enhanced / Account Payload**

Copy

```
[
  {
    "accountData": [
      {
        "account": "CKs1E69a2e9TmH4mKKLrXFF8kD3ZnwKjoEuXa6sz9WqX",
        "nativeBalanceChange": -72938049280,
        "tokenBalanceChanges": []
      },
      {
        "account": "NTYeYJ1wr4bpM5xo6zx5En44SvJFAd35zTxxNoERYqd",
        "nativeBalanceChange": 0,
        "tokenBalanceChanges": []
      },
      {
        "account": "AAaTGaA3uVqikfVEwoSG7EwkCb4bBDsMEyueiVUS5CaU",
        "nativeBalanceChange": 0,
        "tokenBalanceChanges": []
      },
      {
        "account": "autMW8SgBkVYeBgqYiTuJZnkvDZMVU2MHJh9Jh7CSQ2",
        "nativeBalanceChange": 0,
        "tokenBalanceChanges": []
      },
      {
        "account": "D8TxfGwdu9MiNMoJmUoC9wQfNfNT7Lnm6DzifQHRTy6B",
        "nativeBalanceChange": 0,
        "tokenBalanceChanges": []
      },
      {
        "account": "5DxD5ViWjvRZEkxQEaJHZw2sBsso6xoXx3wGFNKgXUzE",
        "nativeBalanceChange": 71860273440,
        "tokenBalanceChanges": []
      },
      {
        "account": "25DTUAd1roBFoUQaxJQByL6Qy2cKQCBp4bK9sgfy9UiM",
        "nativeBalanceChange": -2039280,
        "tokenBalanceChanges": [
          {
            "mint": "FdsNQE5EeCe57tbEYCRV1JwW5dzNCof7MUTaGWhmzYqu",
            "rawTokenAmount": {
              "decimals": 0,
              "tokenAmount": "-1"
            },
            "tokenAccount": "25DTUAd1roBFoUQaxJQByL6Qy2cKQCBp4bK9sgfy9UiM",
            "userAccount": "1BWutmTvYPwDtmw9abTkS4Ssr8no61spGAvW1X6NDix"
          }
        ]
      },
      {
        "account": "DTYuh7gAGGZg2okM7hdFfU1yMY9LUemCiPyD5Z5GCs6Z",
        "nativeBalanceChange": 2039280,
        "tokenBalanceChanges": [
          {
            "mint": "FdsNQE5EeCe57tbEYCRV1JwW5dzNCof7MUTaGWhmzYqu",
            "rawTokenAmount": {
              "decimals": 0,
              "tokenAmount": "1"
            },
            "tokenAccount": "DTYuh7gAGGZg2okM7hdFfU1yMY9LUemCiPyD5Z5GCs6Z",
            "userAccount": "CKs1E69a2e9TmH4mKKLrXFF8kD3ZnwKjoEuXa6sz9WqX"
          }
        ]
      },
      {
        "account": "rFqFJ9g7TGBD8Ed7TPDnvGKZ5pWLPDyxLcvcH2eRCtt",
        "nativeBalanceChange": 1080000000,
        "tokenBalanceChanges": []
      },
      {
        "account": "CgXS5xC3qAGSg9txD9bS7BUgugZwshivGXpCJcGmdwrd",
        "nativeBalanceChange": -2234160,
        "tokenBalanceChanges": []
      },
      {
        "account": "M2mx93ekt1fmXSVkTrUL9xVFHkmME8HTUi5Cyc5aF7K",
        "nativeBalanceChange": 0,
        "tokenBalanceChanges": []
      },
      {
        "account": "E8cU1WiRWjanGxmn96ewBgk9vPTcL6AEZ1t6F6fkgUWe",
        "nativeBalanceChange": 0,
        "tokenBalanceChanges": []
      },
      {
        "account": "11111111111111111111111111111111",
        "nativeBalanceChange": 0,
        "tokenBalanceChanges": []
      },
      {
        "account": "FdsNQE5EeCe57tbEYCRV1JwW5dzNCof7MUTaGWhmzYqu",
        "nativeBalanceChange": 0,
        "tokenBalanceChanges": []
      },
      {
        "account": "AYZsWahcrSnkwqbA1ji7wEzgAnGjLNJhVUMDPfACECZf",
        "nativeBalanceChange": 0,
        "tokenBalanceChanges": []
      },
      {
        "account": "TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA",
        "nativeBalanceChange": 0,
        "tokenBalanceChanges": []
      },
      {
        "account": "SysvarRent111111111111111111111111111111111",
        "nativeBalanceChange": 0,
        "tokenBalanceChanges": []
      },
      {
        "account": "ATokenGPvbdGVxr1b2hvZbsiqW5xWH25efTNsLJA8knL",
        "nativeBalanceChange": 0,
        "tokenBalanceChanges": []
      },
      {
        "account": "1BWutmTvYPwDtmw9abTkS4Ssr8no61spGAvW1X6NDix",
        "nativeBalanceChange": 0,
        "tokenBalanceChanges": []
      }
    ],
    "description": "5DxD5ViWjvRZEkxQEaJHZw2sBsso6xoXx3wGFNKgXUzE sold Fox #7637 to CKs1E69a2e9TmH4mKKLrXFF8kD3ZnwKjoEuXa6sz9WqX for 72 SOL on MAGIC_EDEN.",
    "events": {
      "nft": {
        "amount": 72000000000,
        "buyer": "CKs1E69a2e9TmH4mKKLrXFF8kD3ZnwKjoEuXa6sz9WqX",
        "description": "5DxD5ViWjvRZEkxQEaJHZw2sBsso6xoXx3wGFNKgXUzE sold Fox #7637 to CKs1E69a2e9TmH4mKKLrXFF8kD3ZnwKjoEuXa6sz9WqX for 72 SOL on MAGIC_EDEN.",
        "fee": 10000,
        "feePayer": "CKs1E69a2e9TmH4mKKLrXFF8kD3ZnwKjoEuXa6sz9WqX",
        "nfts": [
          {
            "mint": "FdsNQE5EeCe57tbEYCRV1JwW5dzNCof7MUTaGWhmzYqu",
            "tokenStandard": "NonFungible"
          }
        ],
        "saleType": "INSTANT_SALE",
        "seller": "5DxD5ViWjvRZEkxQEaJHZw2sBsso6xoXx3wGFNKgXUzE",
        "signature": "5nNtjezQMYBHvgSQmoRmJPiXGsPAWmJPoGSa64xanqrauogiVzFyGQhKeFataHGXq51jR2hjbzNTkPUpP787HAmL",
        "slot": 171942732,
        "source": "MAGIC_EDEN",
        "staker": "",
        "timestamp": 1673445241,
        "type": "NFT_SALE"
      }
    },
    "fee": 10000,
    "feePayer": "CKs1E69a2e9TmH4mKKLrXFF8kD3ZnwKjoEuXa6sz9WqX",
    "nativeTransfers": [
      {
        "amount": 72936000000,
        "fromUserAccount": "CKs1E69a2e9TmH4mKKLrXFF8kD3ZnwKjoEuXa6sz9WqX",
        "toUserAccount": "AAaTGaA3uVqikfVEwoSG7EwkCb4bBDsMEyueiVUS5CaU"
      },
      {
        "amount": 2011440,
        "fromUserAccount": "CKs1E69a2e9TmH4mKKLrXFF8kD3ZnwKjoEuXa6sz9WqX",
        "toUserAccount": "D8TxfGwdu9MiNMoJmUoC9wQfNfNT7Lnm6DzifQHRTy6B"
      },
      {
        "amount": 71856000000,
        "fromUserAccount": "AAaTGaA3uVqikfVEwoSG7EwkCb4bBDsMEyueiVUS5CaU",
        "toUserAccount": "5DxD5ViWjvRZEkxQEaJHZw2sBsso6xoXx3wGFNKgXUzE"
      },
      {
        "amount": 1080000000,
        "fromUserAccount": "AAaTGaA3uVqikfVEwoSG7EwkCb4bBDsMEyueiVUS5CaU",
        "toUserAccount": "rFqFJ9g7TGBD8Ed7TPDnvGKZ5pWLPDyxLcvcH2eRCtt"
      },
      {
        "amount": 2039280,
        "fromUserAccount": "CKs1E69a2e9TmH4mKKLrXFF8kD3ZnwKjoEuXa6sz9WqX",
        "toUserAccount": "DTYuh7gAGGZg2okM7hdFfU1yMY9LUemCiPyD5Z5GCs6Z"
      }
    ],
    "signature": "5nNtjezQMYBHvgSQmoRmJPiXGsPAWmJPoGSa64xanqrauogiVzFyGQhKeFataHGXq51jR2hjbzNTkPUpP787HAmL",
    "slot": 171942732,
    "source": "MAGIC_EDEN",
    "timestamp": 1673445241,
    "tokenTransfers": [
      {
        "fromTokenAccount": "25DTUAd1roBFoUQaxJQByL6Qy2cKQCBp4bK9sgfy9UiM",
        "fromUserAccount": "1BWutmTvYPwDtmw9abTkS4Ssr8no61spGAvW1X6NDix",
        "mint": "FdsNQE5EeCe57tbEYCRV1JwW5dzNCof7MUTaGWhmzYqu",
        "toTokenAccount": "DTYuh7gAGGZg2okM7hdFfU1yMY9LUemCiPyD5Z5GCs6Z",
        "toUserAccount": "CKs1E69a2e9TmH4mKKLrXFF8kD3ZnwKjoEuXa6sz9WqX",
        "tokenAmount": 1,
        "tokenStandard": "NonFungible"
      }
    ],
    "type": "NFT_SALE"
  }
]
```

**Raw Payload**

Copy

```
[
  {
    "blockTime": 1673445241,
    "indexWithinBlock": 2557,
    "meta": {
      "err": null,
      "fee": 10000,
      "innerInstructions": [
        {
          "index": 0,
          "instructions": [
            {
              "accounts": [
                0,
                2
              ],
              "data": "3Bxs3zs3x6pg4XWo",
              "programIdIndex": 12
            }
          ]
        },
        {
          "index": 1,
          "instructions": [
            {
              "accounts": [
                0,
                4
              ],
              "data": "11112nba6qLH4BKL4MW8GP9ayKApZeYn3LQKJdPdeSXbRW1n6UPeJ8y77ps6sAVwAjdxzh",
              "programIdIndex": 12
            }
          ]
        },
        {
          "index": 2,
          "instructions": [
            {
              "accounts": [
                2,
                5
              ],
              "data": "3Bxs3zx147oWJQej",
              "programIdIndex": 12
            },
            {
              "accounts": [
                2,
                8
              ],
              "data": "3Bxs3zwT1TGLhiT9",
              "programIdIndex": 12
            },
            {
              "accounts": [
                0,
                7,
                0,
                13,
                12,
                15
              ],
              "data": "1",
              "programIdIndex": 17
            },
            {
              "accounts": [
                13
              ],
              "data": "84eT",
              "programIdIndex": 15
            },
            {
              "accounts": [
                0,
                7
              ],
              "data": "11119os1e9qSs2u7TsThXqkBSRVFxhmYaFKFZ1waB2X7armDmvK3p5GmLdUxYdg3h7QSrL",
              "programIdIndex": 12
            },
            {
              "accounts": [
                7
              ],
              "data": "P",
              "programIdIndex": 15
            },
            {
              "accounts": [
                7,
                13
              ],
              "data": "6YTZgAHgNKVRJ2mAHQUYC1DgXF6dPCgbSWA5P4gZoSfGV",
              "programIdIndex": 15
            },
            {
              "accounts": [
                6,
                7,
                18
              ],
              "data": "3DdGGhkhJbjm",
              "programIdIndex": 15
            },
            {
              "accounts": [
                6,
                5,
                18
              ],
              "data": "A",
              "programIdIndex": 15
            }
          ]
        }
      ],
      "loadedAddresses": {
        "readonly": [],
        "writable": []
      },
      "logMessages": [
        "Program M2mx93ekt1fmXSVkTrUL9xVFHkmME8HTUi5Cyc5aF7K invoke [1]",
        "Program log: Instruction: Deposit",
        "Program 11111111111111111111111111111111 invoke [2]",
        "Program 11111111111111111111111111111111 success",
        "Program M2mx93ekt1fmXSVkTrUL9xVFHkmME8HTUi5Cyc5aF7K consumed 10148 of 600000 compute units",
        "Program M2mx93ekt1fmXSVkTrUL9xVFHkmME8HTUi5Cyc5aF7K success",
        "Program M2mx93ekt1fmXSVkTrUL9xVFHkmME8HTUi5Cyc5aF7K invoke [1]",
        "Program log: Instruction: Buy",
        "Program 11111111111111111111111111111111 invoke [2]",
        "Program 11111111111111111111111111111111 success",
        "Program log: {\"price\":72000000000,\"buyer_expiry\":0}",
        "Program M2mx93ekt1fmXSVkTrUL9xVFHkmME8HTUi5Cyc5aF7K consumed 30501 of 589852 compute units",
        "Program M2mx93ekt1fmXSVkTrUL9xVFHkmME8HTUi5Cyc5aF7K success",
        "Program M2mx93ekt1fmXSVkTrUL9xVFHkmME8HTUi5Cyc5aF7K invoke [1]",
        "Program log: Instruction: ExecuteSaleV2",
        "Program 11111111111111111111111111111111 invoke [2]",
        "Program 11111111111111111111111111111111 success",
        "Program 11111111111111111111111111111111 invoke [2]",
        "Program 11111111111111111111111111111111 success",
        "Program ATokenGPvbdGVxr1b2hvZbsiqW5xWH25efTNsLJA8knL invoke [2]",
        "Program log: Create",
        "Program TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA invoke [3]",
        "Program log: Instruction: GetAccountDataSize",
        "Program TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA consumed 1622 of 497733 compute units",
        "Program return: TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA pQAAAAAAAAA=",
        "Program TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA success",
        "Program 11111111111111111111111111111111 invoke [3]",
        "Program 11111111111111111111111111111111 success",
        "Program log: Initialize the associated token account",
        "Program TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA invoke [3]",
        "Program log: Instruction: InitializeImmutableOwner",
        "Program log: Please upgrade to SPL Token 2022 for immutable owner support",
        "Program TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA consumed 1405 of 491243 compute units",
        "Program TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA success",
        "Program TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA invoke [3]",
        "Program log: Instruction: InitializeAccount3",
        "Program TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA consumed 4241 of 487361 compute units",
        "Program TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA success",
        "Program ATokenGPvbdGVxr1b2hvZbsiqW5xWH25efTNsLJA8knL consumed 21793 of 504630 compute units",
        "Program ATokenGPvbdGVxr1b2hvZbsiqW5xWH25efTNsLJA8knL success",
        "Program TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA invoke [2]",
        "Program log: Instruction: Transfer",
        "Program TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA consumed 4645 of 475696 compute units",
        "Program TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA success",
        "Program TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA invoke [2]",
        "Program log: Instruction: CloseAccount",
        "Program TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA consumed 3033 of 456654 compute units",
        "Program TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA success",
        "Program log: {\"price\":72000000000,\"seller_expiry\":-1,\"buyer_expiry\":0}",
        "Program M2mx93ekt1fmXSVkTrUL9xVFHkmME8HTUi5Cyc5aF7K consumed 109266 of 559351 compute units",
        "Program M2mx93ekt1fmXSVkTrUL9xVFHkmME8HTUi5Cyc5aF7K success"
      ],
      "postBalances": [
        371980779080,
        0,
        0,
        100129388687,
        0,
        81872924494,
        0,
        2039280,
        993583055919,
        0,
        1141440,
        3654000,
        1,
        1461600,
        5616720,
        934087680,
        1009200,
        731913600,
        457953014766
      ],
      "postTokenBalances": [
        {
          "accountIndex": 7,
          "mint": "FdsNQE5EeCe57tbEYCRV1JwW5dzNCof7MUTaGWhmzYqu",
          "owner": "CKs1E69a2e9TmH4mKKLrXFF8kD3ZnwKjoEuXa6sz9WqX",
          "programId": "TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA",
          "uiTokenAmount": {
            "amount": "1",
            "decimals": 0,
            "uiAmount": 1,
            "uiAmountString": "1"
          }
        }
      ],
      "preBalances": [
        444918828360,
        0,
        0,
        100129388687,
        0,
        10012651054,
        2039280,
        0,
        992503055919,
        2234160,
        1141440,
        3654000,
        1,
        1461600,
        5616720,
        934087680,
        1009200,
        731913600,
        457953014766
      ],
      "preTokenBalances": [
        {
          "accountIndex": 6,
          "mint": "FdsNQE5EeCe57tbEYCRV1JwW5dzNCof7MUTaGWhmzYqu",
          "owner": "1BWutmTvYPwDtmw9abTkS4Ssr8no61spGAvW1X6NDix",
          "programId": "TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA",
          "uiTokenAmount": {
            "amount": "1",
            "decimals": 0,
            "uiAmount": 1,
            "uiAmountString": "1"
          }
        }
      ],
      "rewards": []
    },
    "slot": 171942732,
    "transaction": {
      "message": {
        "accountKeys": [
          "CKs1E69a2e9TmH4mKKLrXFF8kD3ZnwKjoEuXa6sz9WqX",
          "NTYeYJ1wr4bpM5xo6zx5En44SvJFAd35zTxxNoERYqd",
          "AAaTGaA3uVqikfVEwoSG7EwkCb4bBDsMEyueiVUS5CaU",
          "autMW8SgBkVYeBgqYiTuJZnkvDZMVU2MHJh9Jh7CSQ2",
          "D8TxfGwdu9MiNMoJmUoC9wQfNfNT7Lnm6DzifQHRTy6B",
          "5DxD5ViWjvRZEkxQEaJHZw2sBsso6xoXx3wGFNKgXUzE",
          "25DTUAd1roBFoUQaxJQByL6Qy2cKQCBp4bK9sgfy9UiM",
          "DTYuh7gAGGZg2okM7hdFfU1yMY9LUemCiPyD5Z5GCs6Z",
          "rFqFJ9g7TGBD8Ed7TPDnvGKZ5pWLPDyxLcvcH2eRCtt",
          "CgXS5xC3qAGSg9txD9bS7BUgugZwshivGXpCJcGmdwrd",
          "M2mx93ekt1fmXSVkTrUL9xVFHkmME8HTUi5Cyc5aF7K",
          "E8cU1WiRWjanGxmn96ewBgk9vPTcL6AEZ1t6F6fkgUWe",
          "11111111111111111111111111111111",
          "FdsNQE5EeCe57tbEYCRV1JwW5dzNCof7MUTaGWhmzYqu",
          "AYZsWahcrSnkwqbA1ji7wEzgAnGjLNJhVUMDPfACECZf",
          "TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA",
          "SysvarRent111111111111111111111111111111111",
          "ATokenGPvbdGVxr1b2hvZbsiqW5xWH25efTNsLJA8knL",
          "1BWutmTvYPwDtmw9abTkS4Ssr8no61spGAvW1X6NDix"
        ],
        "addressTableLookups": null,
        "header": {
          "numReadonlySignedAccounts": 1,
          "numReadonlyUnsignedAccounts": 9,
          "numRequiredSignatures": 2
        },
        "instructions": [
          {
            "accounts": [
              0,
              1,
              2,
              3,
              11,
              12
            ],
            "data": "3GyWrkssW12wSfxjTynBnbif",
            "programIdIndex": 10
          },
          {
            "accounts": [
              0,
              1,
              13,
              14,
              2,
              3,
              11,
              4,
              3,
              15,
              12,
              16
            ],
            "data": "3Jmjmsq2jyrch5iz612vBLZCRB498owPe7qezQVetRZhiMu",
            "programIdIndex": 10
          },
          {
            "accounts": [
              0,
              5,
              1,
              6,
              13,
              14,
              2,
              7,
              3,
              11,
              8,
              4,
              3,
              9,
              3,
              15,
              12,
              17,
              18,
              16
            ],
            "data": "B2rqPwAgvj3t35y6HpdumfhdhsZMLNFLmXMC9Uz2HX4nNAwirTk3o98vTazFB1y",
            "programIdIndex": 10
          }
        ],
        "recentBlockhash": "3NRncb7FJuDruQjMDxnHvJBQkvkHa7KSUBqBsxG21roZ"
      },
      "signatures": [
        "5nNtjezQMYBHvgSQmoRmJPiXGsPAWmJPoGSa64xanqrauogiVzFyGQhKeFataHGXq51jR2hjbzNTkPUpP787HAmL",
        "4dWBkbLHGvU2jw9Sjj6YETtKfaVKAAN1M8aWzXRNC4aHBckUzM73n3FddNbWTtfUvkU2vFRQ7bKHMwKZQ5dGy1iH"
      ]
    }
  }
]
```



## Websockets

Stream real-time data directly to your applications with our websocket integration.

#### **What are Websockets?** <a href="#what-are-websockets" id="what-are-websockets"></a>

Websockets allow for two-way communication between a client and a server. Unlike traditional request-response models, Websockets keep a persistent connection open, enabling real-time data exchange. This is perfect for applications that require instant updates like chat apps, online games, trading bots, and marketplaces. Helius supports all stable Solana Websockets. You can find a list of all these Websockets in the [Solana documentation](https://docs.solana.com/api/websocket). You can use these with your Helius WSS URL:. **Mainnet:** `wss://mainnet.helius-rpc.com/?api-key=<YOUR_API_KEY>` **Devnet:** `wss://devnet.helius-rpc.com/?api-key=<YOUR_API_KEY>`

### Helius' Geyser Enhanced Websockets (beta) <a href="#helius-geyser-enhanced-websockets-beta" id="helius-geyser-enhanced-websockets-beta"></a>

In addition to supporting Solana's standard Websocket methods, Helius provides access to Geyser-enhanced Websockets. These Websockets boast faster response speeds when compared to the standard RPC Websocket methods.

These Websockets are currently in beta. We would love your feedback on Discord!

#### **Why Use Helius Websockets?** <a href="#why-use-helius-websockets" id="why-use-helius-websockets"></a>

* **Direct Streaming**: No more polling or periodic checks. Get data directly when changes happen.
* **Seamless Integration**: Easily integrate our websockets into your application at any layer. Websockets can be used directly in your frontend!
* **Cost-Effective**: Reduce the overhead and expenses that come with frequently polling data.
* **Built on top of Geyser**: Instantly react to live on-chain data changes by getting a direct feed into Geyser data. Read more about Geyser [here](https://www.helius.dev/blog/solana-geyser-plugins-streaming-data-at-the-speed-of-light).

This feature is only available for Business and Professional plans while still in beta.

#### Subscription Endpoints <a href="#subscription-endpoints" id="subscription-endpoints"></a>

Helius websockets are currently available in `mainnet` and `devnet` with the following URL's

**Mainnet:** `wss://atlas-mainnet.helius-rpc.com?api-key=<API_KEY>`

**Devnet:** `wss://atlas-devnet.helius-rpc.com?api-key=<API_KEY>`

#### Transaction Subscribe <a href="#transaction-subscribe" id="transaction-subscribe"></a>

While Solana offers several websocket methods for various functionalities, it notably lacks a dedicated method for subscribing to transaction updates. To bridge this gap, Helius introduces the **`transactionSubscribe`** websocket method. This allows developers to tune in to real-time transaction events. In order to subscribe to transactions, you need to provide a **`TransactionSubscribeFilter`** and can optionally provide **`TransactionSubscribeOptions`**

**Example Payload:**

Copy

```
{
  "jsonrpc": "2.0",
  "id": 420,
  "method": "transactionSubscribe",
  "params": [
      {
        "vote": false,
        "failed": false,
        "signature": "2dd5zTLrSs2udfNsegFRCnzSyQcPrM9svX6m1UbEM5bSdXXFj3XpqaodtKarLYFP2mTVUsV27sRDdZCgcKhjeD9S",
        "accountInclude": ["pqx3fvvh6b2eZBfLhTtQ5KxzU3CginmgGTmDCjk8TPP"],
        "accountExclude": ["FbfwE8ZmVdwUbbEXdq4ofhuUEiAxeSk5kaoYrJJekpnZ"],
        "accountRequired": ["As1XYY9RdGkjs62isDhLKG3yxMCMatnbanXrqU85XvXW"]
      },
      {
	"commitment": "processed",
    	"encoding": "base64",
    	"transaction_details": "full",
    	"showRewards": true,
    	"maxSupportedTransactionVersion": 0
      }
  ]
}
```

**Example Notification:**

Copy

```
{
    "jsonrpc": "2.0",
    "method": "transactionNotification",
    "params": {
        "subscription": 4743323479349712,
        "result": {
            "transaction": {
                "transaction": [
                    "Ae6zfSExLsJ/E1+q0jI+3ueAtSoW+6HnuDohmuFwagUo2BU4OpkSdUKYNI1dJfMOonWvjaumf4Vv1ghn9f3Avg0BAAEDGycH0OcYRpfnPNuu0DBQxTYPWpmwHdXPjb8y2P200JgK3hGiC2JyC9qjTd2lrug7O4cvSRUVWgwohbbefNgKQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA0HcpwKokfYDDAJTaF/TWRFWm0Gz5/me17PRnnywHurMBAgIAAQwCAAAAoIYBAAAAAAA=",
                    "base64"
                ],
                "meta": {
                    "err": null,
                    "status": {
                        "Ok": null
                    },
                    "fee": 5000,
                    "preBalances": [
                        28279852264,
                        158122684,
                        1
                    ],
                    "postBalances": [
                        28279747264,
                        158222684,
                        1
                    ],
                    "innerInstructions": [],
                    "logMessages": [
                        "Program 11111111111111111111111111111111 invoke [1]",
                        "Program 11111111111111111111111111111111 success"
                    ],
                    "preTokenBalances": [],
                    "postTokenBalances": [],
                    "rewards": null,
                    "loadedAddresses": {
                        "writable": [],
                        "readonly": []
                    },
                    "computeUnitsConsumed": 0
                }
            },
            "signature": "5moMXe6VW7L7aQZskcAkKGQ1y19qqUT1teQKBNAAmipzdxdqVLAdG47WrsByFYNJSAGa9TByv15oygnqYvP6Hn2p"
        }
    }
}
```

**TransactionSubscribeFilter:**

**`vote: Option<bool>`**: A flag to include/exclude vote-related transactions.

**`failed: Option<bool>`**: A flag to include/exclude transactions that failed.

**`signature: Option<String>`**: Filters updates to a specific transaction based on its signature.

**`accountInclude: Option<Vec<String>>`**: A list of accounts for which you want to receive transaction updates. This means that only one of the accounts must be included in the transaction updates (e.g., Account 1 OR Account 2).

**`accountExclude: Option<Vec<String>>`**: A list of accounts you want to exclude from transaction updates.

**`accountRequired: Option<Vec<String>>`**: Transactions must involve these specified accounts to be included in updates. This means that all of the accounts must be included in the transaction updates (e.g., Account 1 AND Account 2).

You can include up to 50 000 addresses in the accountsInclude, accountExclude and accountRequired arrays.

**Optional - TransactionSubscribeOptions:**

**`commitment: Option<Commitment>`**: Specifies the commitment level for fetching data, dictating at what stage of the transactions lifecycle updates are sent. The possible values are

* processed
* confirmed
* finalized

**`encoding: Option<UiTransactionEncoding>`**: Sets the encoding format of the returned transaction data. The possible values are

* base58
* base64
* base64+zstd
* jsonParsed

**`transactionDetails: Option<TransactionDetails>`** : Determines the level of detail for the returned transaction data. The possible values are

* full
* signatures
* accounts
* none

**`showRewards: Option<bool>`**: A boolean flag indicating if reward data should be included in the transaction updates.

**`maxSupportedTransactionVersion: Option<u8>`**: Specifies the highest version of transactions you want to receive updates for.

Note, `maxSupportedTransactionVersion` is required for returning the accounts and full level details of a given transaction (i.e., `transactionDetails: "accounts" | "full"`).

#### Transaction Subscribe code example: <a href="#transaction-subscribe-code-example" id="transaction-subscribe-code-example"></a>

In this example we are subscribing to transactions that contain the account `8uDncGPHZJtFnyCktVSbdWmgt7xAiHgwq8ZmxKaK5ZJ1` . Whenever a transaction occurs that contains `8uDncGPHZJtFnyCktVSbdWmgt7xAiHgwq8ZmxKaK5ZJ1` in the `accountKeys` of the transaction, we will receive a websocket notification. Based on the subscription options, the transaction notification will be sent at the `processed` commitment level, with `base64` encoding, `full` transaction details, and will show rewards.

Copy

```
const WebSocket = require('ws');

// Create a WebSocket connection
const ws = new WebSocket('wss://atlas-mainnet.helius-rpc.com?api-key=<API_KEY>');

// Function to send a request to the WebSocket server
function sendRequest(ws) {
    const request = {
        jsonrpc: "2.0",
        id: 420,
        method: "transactionSubscribe",
        params: [
            {
                accountInclude: ["8uDncGPHZJtFnyCktVSbdWmgt7xAiHgwq8ZmxKaK5ZJ1"]
            },
            {
                commitment: "processed",
                encoding: "base64",
                transactionDetails: "full",
                showRewards: true,
                maxSupportedTransactionVersion: 0
            }
        ]
    };
    ws.send(JSON.stringify(request));
}


// Define WebSocket event handlers

ws.on('open', function open() {
    console.log('WebSocket is open');
    sendRequest(ws);  // Send a request once the WebSocket is open
});

ws.on('message', function incoming(data) {
    const messageStr = data.toString('utf8');
    try {
        const messageObj = JSON.parse(messageStr);
        console.log('Received:', messageObj);
    } catch (e) {
        console.error('Failed to parse JSON:', e);
    }
});

ws.on('error', function error(err) {
    console.error('WebSocket error:', err);
});

ws.on('close', function close() {
    console.log('WebSocket is closed');
}); 
```

#### Account Subscribe <a href="#account-subscribe" id="account-subscribe"></a>

Helius Websockets supports a method to subscribe to an account and receive notifications over the websocket connection when the lamports or data for a matching account public key changes. This method matches the Solana Websocket API `accountSubscribe` spec exactly. See the Solana docs for more: [https://docs.solana.com/api/websocket#accountsubscribe](https://docs.solana.com/api/websocket#accountsubscribe)

**Example Payload:**

Copy

```
{
  "jsonrpc": "2.0",
  "id": 420,
  "method": "accountSubscribe",
   "params": [
    "SysvarC1ock11111111111111111111111111111111",
    {
      "encoding": "jsonParsed",
      "commitment": "finalized"
    }
  ]
}
```

**Example Notification:**

Copy

```
{
    'jsonrpc': '2.0', 
    'method': 'accountNotification', 
    'params': 
        {
          'subscription': 237508762798666, 
          'result': 
           {
            'context': {'slot': 235781083}, 
            'value': 
             {
             'lamports': 1169280, 
             'data': 'BvEhEb6hixL3QPn41gHcyi2CDGKt381jbNKFFCQr6XDTzCTXCuSUG9D', 
             'owner': 'Sysvar1111111111111111111111111111111111111', 
             'executable': False, 
             'rentEpoch': 361, 
             'space': 40
             }
           }
        }
}
```

**Parameters:**

**`String`**: Account public key, sent in base58 format, required

**`object: Option<RpcAccountInfoConfig>:`** Optional object used to pass in `encoding` for the data returned in the `AccountNotification` and `commitment`

If nothing is passed into the object, the response will default to base58 encoding and a `finalized` commitment level.

Copy

```
pub struct RpcAccountInfoConfig {
    pub encoding: Option<UiAccountEncoding>, // supported values: base58, base64, base64+zstd, jsonParsed
    pub commitment: Option<CommitmentConfig>, // supported values: finalized, confirmed, processed - defaults to finalized
}
```

#### Account Subscribe code example: <a href="#account-subscribe-code-example" id="account-subscribe-code-example"></a>

In this example we are subscribing to account changes for the account `SysvarC1ock11111111111111111111111111111111` . Whenever a change occurs to the account data or the lamports for this account, we will see an update. This happens at a frequent interval for this specific account as the `slot` and `unixTimestamp` are both a part of the returned account data.

Copy

```
// Create a WebSocket connection
const ws = new WebSocket('wss://atlas-mainnet.helius-rpc.com?api-key=<API_KEY>');

// Function to send a request to the WebSocket server
function sendRequest(ws) {
    const request = {
        jsonrpc: "2.0",
        id: 420,
        method: "accountSubscribe",
        params: [
            "SysvarC1ock11111111111111111111111111111111", // pubkey of account we want to subscribe to
            {
                encoding: "jsonParsed", // base58, base64, base65+zstd, jsonParsed
                commitment: "confirmed", // defaults to finalized if unset
            }
        ]
    };
    ws.send(JSON.stringify(request));
}


// Define WebSocket event handlers

ws.on('open', function open() {
    console.log('WebSocket is open');
    sendRequest(ws);  // Send a request once the WebSocket is open
});

ws.on('message', function incoming(data) {
    const messageStr = data.toString('utf8');
    try {
        const messageObj = JSON.parse(messageStr);
        console.log('Received:', messageObj);
    } catch (e) {
        console.error('Failed to parse JSON:', e);
    }
});

ws.on('error', function error(err) {
    console.error('WebSocket error:', err);
});

ws.on('close', function close() {
    console.log('WebSocket is closed');
});
```



## Enhanced Transactions API

Transactions APIs to parse Solana data.

Enhanced Transaction API V1 is under maintenance while we work on V2.

Attention: we **only** parse NFT, Jupiter, and SPL related transactions so far. Do not rely on these parsers for DeFi or non-NFT, Jupiter, and SPL transactions.

Obtain enriched historical context about _any_ Solana address.

#### Parse A Single Transaction <a href="#parse-a-single-transaction" id="parse-a-single-transaction"></a>

[PAGEParse Transaction(s)](https://docs.helius.dev/solana-apis/enhanced-transactions-api/parse-transaction-s)

#### Get Enriched Transaction History <a href="#get-enriched-transaction-history" id="get-enriched-transaction-history"></a>

[PAGEParsed Transaction History](https://docs.helius.dev/solana-apis/enhanced-transactions-api/parsed-transaction-history)



## Parse Transaction(s)

Parse individual Solana transactions.

Enhanced Transaction API V1 is under maintenance while we work on V2.

The max number of transactions you can pass in to this endpoint is 100. We **only** parse NFT actions, Jupiter swaps, and SPL related transactions so far. Do not rely on these parsers for DeFi or non-NFT, Jupiter, and SPL transactions.

#### v0/transactions <a href="#v0-transactions" id="v0-transactions"></a>

Returns an array of enriched, human-readable transactions of the given transaction signatures. For a full list of Transaction Types and Sources, please see [Transaction Types](https://docs.helius.dev/resources/transaction-types).

Mainnet URL: [https://api.helius.xyz/v0/transactions](https://api.helius.xyz/v0/transactions) Devnet URL: https://api-devnet.helius.xyz/v0/transactions

### Returns an array of enriched, human-readable versions of the given transactions.

POSThttps://api.helius.xyz/v0/transactionsQuery parametersBodyapplication/jsontransactionsarray of stringResponse200400401403404429500

Returns an array of enriched transactions.

Bodyapplication/jsondescriptionstringExample: `"Human readable interpretation of the transaction"`typeTransactionType (enum)UNKNOWNNFT\_BIDNFT\_BID\_CANCELLEDNFT\_LISTINGNFT\_CANCEL\_LISTINGNFT\_SALENFT\_MINTNFT\_AUCTION\_CREATEDNFT\_AUCTION\_UPDATEDNFT\_AUCTION\_CANCELLEDNFT\_PARTICIPATION\_REWARDNFT\_MINT\_REJECTEDCREATE\_STOREWHITELIST\_CREATORADD\_TO\_WHITELISTREMOVE\_FROM\_WHITELISTAUCTION\_MANAGER\_CLAIM\_BIDEMPTY\_PAYMENT\_ACCOUNTUPDATE\_PRIMARY\_SALE\_METADATAADD\_TOKEN\_TO\_VAULTACTIVATE\_VAULTINIT\_VAULTINIT\_BANKINIT\_STAKEMERGE\_STAKESPLIT\_STAKESET\_BANK\_FLAGSSET\_VAULT\_LOCKUPDATE\_VAULT\_OWNERUPDATE\_BANK\_MANAGERRECORD\_RARITY\_POINTSADD\_RARITIES\_TO\_BANKINIT\_FARMINIT\_FARMERREFRESH\_FARMERUPDATE\_FARMAUTHORIZE\_FUNDERDEAUTHORIZE\_FUNDERFUND\_REWARDCANCEL\_REWARDLOCK\_REWARDPAYOUTVALIDATE\_SAFETY\_DEPOSIT\_BOX\_V2SET\_AUTHORITYINIT\_AUCTION\_MANAGER\_V2UPDATE\_EXTERNAL\_PRICE\_ACCOUNTAUCTION\_HOUSE\_CREATECLOSE\_ESCROW\_ACCOUNTWITHDRAWDEPOSITTRANSFERBURNBURN\_NFTPLATFORM\_FEELOANRESCIND\_LOANOFFER\_LOANREPAY\_LOANTAKE\_LOANFORECLOSE\_LOANADD\_TO\_POOLREMOVE\_FROM\_POOLCLOSE\_POSITIONUNLABELEDCLOSE\_ACCOUNTWITHDRAW\_GEMDEPOSIT\_GEMSTAKE\_TOKENUNSTAKE\_TOKENSTAKE\_SOLUNSTAKE\_SOLCLAIM\_REWARDSBUY\_SUBSCRIPTIONSWAPINIT\_SWAPCANCEL\_SWAPREJECT\_SWAPINITIALIZE\_ACCOUNTTOKEN\_MINTCREATE\_APPARAISALFUSEDEPOSIT\_FRACTIONAL\_POOLFRACTIONALIZECREATE\_RAFFLEBUY\_TICKETSUPDATE\_ITEMLIST\_ITEMDELIST\_ITEMADD\_ITEMCLOSE\_ITEMBUY\_ITEMFILL\_ORDERUPDATE\_ORDERCREATE\_ORDERCLOSE\_ORDERCANCEL\_ORDERKICK\_ITEMUPGRADE\_FOXUPGRADE\_FOX\_REQUESTLOAN\_FOXBORROW\_FOXSWITCH\_FOX\_REQUESTSWITCH\_FOXCREATE\_ESCROWACCEPT\_REQUEST\_ARTISTCANCEL\_ESCROWACCEPT\_ESCROW\_ARTISTACCEPT\_ESCROW\_USERPLACE\_BETPLACE\_SOL\_BETCREATE\_BETNFT\_RENT\_UPDATE\_LISTINGNFT\_RENT\_ACTIVATENFT\_RENT\_CANCEL\_LISTINGNFT\_RENT\_LISTINGFINALIZE\_PROGRAM\_INSTRUCTIONUPGRADE\_PROGRAM\_INSTRUCTIONNFT\_GLOBAL\_BIDNFT\_GLOBAL\_BID\_CANCELLEDEXECUTE\_TRANSACTIONAPPROVE\_TRANSACTIONACTIVATE\_TRANSACTIONCREATE\_TRANSACTIONREJECT\_TRANSACTIONCANCEL\_TRANSACTIONADD\_INSTRUCTIONsourceTransactionSource (enum)FORM\_FUNCTIONEXCHANGE\_ARTCANDY\_MACHINE\_V2CANDY\_MACHINE\_V1UNKNOWNSOLANARTSOLSEAMAGIC\_EDENHOLAPLEXMETAPLEXOPENSEASOLANA\_PROGRAM\_LIBRARYANCHORW\_SOLPHANTOMSYSTEM\_PROGRAMSTAKE\_PROGRAMCOINBASECORAL\_CUBEHEDGELAUNCH\_MY\_NFTGEM\_BANKGEM\_FARMDEGODSBLOCKSMITH\_LABSYAWWWATADIASOLPORTHYPERSPACEDIGITAL\_EYESELIXIRELIXIR\_LAUNCHPADTENSORBIFROSTJUPITERMERCURIAL\_STABLE\_SWAPSABERSERUMSTEP\_FINANCECROPPERRAYDIUMALDRINCREMALIFINITYCYKURAORCAMARINADESTEPNSENCHA EXCHANGESAROSENGLISH\_AUCTION\_AUCTIONFOXYFOXY\_STAKINGFOXY\_RAFFLEFOXY\_TOKEN\_MARKETFOXY\_COINFLIPZETAHADESWAPCARDINAL\_RENTCARDINAL\_STAKINGBPF\_UPGRADEABLE\_LOADERBPF\_LOADERSQUADSOPEN\_CREATOR\_PROTOCOLfeeintegerExample: `5000`feePayerstringExample: `"8cRrU1NzNpjL3k2BwjW3VixAcX6VFc29KHr4KZg8cs2Y"`signaturestringExample: `"yy5BT9benHhx8fGCvhcAfTtLEHAtRJ3hRTzVL16bdrTCWm63t2vapfrZQZLJC3RcuagekaXjSs2zUGQvbcto8DK"`slotintegerExample: `148277128`timestampintegerExample: `1656442333`nativeTransfersarray of NativeTransfer (object)tokenTransfersarray of TokenTransfer (object)accountDataarray of AccountData (object)transactionErrorinline\_response\_400 (object)instructionsarray of Instruction (object)eventsEnrichedTransaction\_events (object)

Events associated with this transaction. These provide fine-grained information about the transaction. They match the types returned from the event APIs.

RequestJavaScriptCurl

```
const response = await fetch('https://api.helius.xyz/v0/transactions', {
    method: 'POST',
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify({}),
});
const data = await response.json();
```

Test itResponse

```
[
  {
    "description": "Human readable interpretation of the transaction",
    "type": "UNKNOWN",
    "source": "FORM_FUNCTION",
    "fee": 5000,
    "feePayer": "8cRrU1NzNpjL3k2BwjW3VixAcX6VFc29KHr4KZg8cs2Y",
    "signature": "yy5BT9benHhx8fGCvhcAfTtLEHAtRJ3hRTzVL16bdrTCWm63t2vapfrZQZLJC3RcuagekaXjSs2zUGQvbcto8DK",
    "slot": 148277128,
    "timestamp": 1656442333,
    "nativeTransfers": [
      {
        "fromUserAccount": "text",
        "toUserAccount": "text"
      }
    ],
    "tokenTransfers": [
      {
        "fromUserAccount": "text",
        "toUserAccount": "text",
        "fromTokenAccount": "text",
        "toTokenAccount": "text",
        "tokenAmount": 0,
        "mint": "DsfCsbbPH77p6yeLS1i4ag9UA5gP9xWSvdCx72FJjLsx"
      }
    ],
    "accountData": [
      {
        "account": "text",
        "nativeBalanceChange": 0,
        "tokenBalanceChanges": [
          {
            "userAccount": "F54ZGuxyb2gA7vRjzWKLWEMQqCfJxDY1whtqtjdq4CJ",
            "tokenAccount": "2kvmbRybhrcptDnwyNv6oiFGFEnRVv7MvVyqsxkirgdn",
            "mint": "DUSTawucrTsGU8hcqRdHDCbuYhCPADMLM2VcCb8VnFnQ",
            "rawTokenAmount": {
              "tokenAmount": "text"
            }
          }
        ]
      }
    ],
    "transactionError": {
      "error": "text"
    },
    "instructions": [
      {
        "accounts": [
          "8uX6yiUuH4UjUb1gMGJAdkXorSuKshqsFGDCFShcK88B"
        ],
        "data": "kdL8HQJrbbvQRGXmoadaja1Qvs",
        "programId": "MEisE1HzehtrDpAAT8PnLHjpSSkRYakotTuJRPjTpo8",
        "innerInstructions": [
          {
            "accounts": [
              "text"
            ],
            "data": "text",
            "programId": "text"
          }
        ]
      }
    ],
    "events": {
      "nft": {
        "description": "text",
        "type": "NFT_SALE",
        "source": "FORM_FUNCTION",
        "amount": 1000000,
        "fee": 5000,
        "feePayer": "8cRrU1NzNpjL3k2BwjW3VixAcX6VFc29KHr4KZg8cs2Y",
        "signature": "4jzQxVTaJ4Fe4Fct9y1aaT9hmVyEjpCqE2bL8JMnuLZbzHZwaL4kZZvNEZ6bEj6fGmiAdCPjmNQHCf8v994PAgDf",
        "slot": 148277128,
        "timestamp": 1656442333,
        "saleType": "AUCTION",
        "buyer": "text",
        "seller": "text",
        "staker": "text",
        "nfts": [
          {
            "mint": "DsfCsbbPH77p6yeLS1i4ag9UA5gP9xWSvdCx72FJjLsx",
            "tokenStandard": "NonFungible"
          }
        ]
      },
      "swap": {
        "nativeInput": {
          "account": "2uySTNgvGT2kwqpfgLiSgeBLR3wQyye1i1A2iQWoPiFr",
          "amount": "100000000"
        },
        "nativeOutput": {
          "account": "2uySTNgvGT2kwqpfgLiSgeBLR3wQyye1i1A2iQWoPiFr",
          "amount": "100000000"
        },
        "tokenInputs": [
          {
            "userAccount": "F54ZGuxyb2gA7vRjzWKLWEMQqCfJxDY1whtqtjdq4CJ",
            "tokenAccount": "2kvmbRybhrcptDnwyNv6oiFGFEnRVv7MvVyqsxkirgdn",
            "mint": "DUSTawucrTsGU8hcqRdHDCbuYhCPADMLM2VcCb8VnFnQ",
            "rawTokenAmount": {
              "tokenAmount": "text"
            }
          }
        ],
        "tokenOutputs": [
          {
            "userAccount": "F54ZGuxyb2gA7vRjzWKLWEMQqCfJxDY1whtqtjdq4CJ",
            "tokenAccount": "2kvmbRybhrcptDnwyNv6oiFGFEnRVv7MvVyqsxkirgdn",
            "mint": "DUSTawucrTsGU8hcqRdHDCbuYhCPADMLM2VcCb8VnFnQ",
            "rawTokenAmount": {
              "tokenAmount": "text"
            }
          }
        ],
        "tokenFees": [
          {
            "userAccount": "F54ZGuxyb2gA7vRjzWKLWEMQqCfJxDY1whtqtjdq4CJ",
            "tokenAccount": "2kvmbRybhrcptDnwyNv6oiFGFEnRVv7MvVyqsxkirgdn",
            "mint": "DUSTawucrTsGU8hcqRdHDCbuYhCPADMLM2VcCb8VnFnQ",
            "rawTokenAmount": {
              "tokenAmount": "text"
            }
          }
        ],
        "nativeFees": [
          {
            "account": "2uySTNgvGT2kwqpfgLiSgeBLR3wQyye1i1A2iQWoPiFr",
            "amount": "100000000"
          }
        ],
        "innerSwaps": [
          {
            "tokenInputs": [
              {
                "fromUserAccount": "text",
                "toUserAccount": "text",
                "fromTokenAccount": "text",
                "toTokenAccount": "text",
                "tokenAmount": 0,
                "mint": "DsfCsbbPH77p6yeLS1i4ag9UA5gP9xWSvdCx72FJjLsx"
              }
            ],
            "tokenOutputs": [
              {
                "fromUserAccount": "text",
                "toUserAccount": "text",
                "fromTokenAccount": "text",
                "toTokenAccount": "text",
                "tokenAmount": 0,
                "mint": "DsfCsbbPH77p6yeLS1i4ag9UA5gP9xWSvdCx72FJjLsx"
              }
            ],
            "tokenFees": [
              {
                "fromUserAccount": "text",
                "toUserAccount": "text",
                "fromTokenAccount": "text",
                "toTokenAccount": "text",
                "tokenAmount": 0,
                "mint": "DsfCsbbPH77p6yeLS1i4ag9UA5gP9xWSvdCx72FJjLsx"
              }
            ],
            "nativeFees": [
              {
                "fromUserAccount": "text",
                "toUserAccount": "text"
              }
            ],
            "programInfo": {
              "source": "ORCA",
              "account": "whirLbMiicVdio4qvUfM5KAg6Ct8VwpYzGff3uctyCc",
              "programName": "ORCA_WHIRLPOOLS",
              "instructionName": "whirlpoolSwap"
            }
          }
        ]
      }
    }
  }
]
```

#### Code Samples <a href="#code-samples" id="code-samples"></a>

parseTransaction.js

```
const url = "https://api.helius.xyz/v0/transactions/?api-key=<your-key>";

const parseTransaction = async () => {
  const response = await fetch(url, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      transactions: ["your-txn-id-here"],
    }),
  });

  const data = await response.json();
  console.log("parsed transaction: ", data);
};

parseTransaction();
```



## Parsed Transaction History

Parsed Transaction History for any given address.

Enhanced Transaction API V1 is under maintenance while we work on V2.

Attention: we **only** parse NFT, Jupiter, and SPL related transactions so far. Do not rely on these parsers for DeFi or non-NFT, Jupiter, and SPL transactions.

#### v0/addresses/:address/transactions <a href="#v0-addresses-address-transactions" id="v0-addresses-address-transactions"></a>

Returns the enriched transaction history for a given address. For a full list of Transaction Types and Sources, please see [Transaction Types](https://docs.helius.dev/resources/transaction-types).

Use https://api-devnet.helius.xyz/v0/addresses/{address}/transactions for devnet transactions

_With no parameters set, this will return the latest transactions for an address._

### Returns an enriched transaction history for a given address

GEThttps://api.helius.xyz/v0/addresses/{address}/transactionsPath parametersaddress\*string

The address to query for.

Query parametersResponse200400401403404429500

Returns an array of enriched transactions.

Bodyapplication/jsondescriptionstringExample: `"Human readable interpretation of the transaction"`typeTransactionType (enum)UNKNOWNNFT\_BIDNFT\_GLOBAL\_BIDNFT\_GLOBAL\_BID\_CANCELLEDNFT\_BID\_CANCELLEDNFT\_LISTINGNFT\_CANCEL\_LISTINGNFT\_SALENFT\_MINTNFT\_AUCTION\_CREATEDNFT\_AUCTION\_UPDATEDNFT\_AUCTION\_CANCELLEDNFT\_PARTICIPATION\_REWARDNFT\_MINT\_REJECTEDCREATE\_STOREWHITELIST\_CREATORADD\_TO\_WHITELISTREMOVE\_FROM\_WHITELISTAUCTION\_MANAGER\_CLAIM\_BIDEMPTY\_PAYMENT\_ACCOUNTUPDATE\_PRIMARY\_SALE\_METADATAADD\_TOKEN\_TO\_VAULTACTIVATE\_VAULTINIT\_VAULTINIT\_BANKINIT\_STAKEMERGE\_STAKESPLIT\_STAKESET\_BANK\_FLAGSSET\_VAULT\_LOCKUPDATE\_VAULT\_OWNERUPDATE\_BANK\_MANAGERRECORD\_RARITY\_POINTSADD\_RARITIES\_TO\_BANKINIT\_FARMINIT\_FARMERREFRESH\_FARMERUPDATE\_FARMAUTHORIZE\_FUNDERDEAUTHORIZE\_FUNDERFUND\_REWARDCANCEL\_REWARDLOCK\_REWARDPAYOUTVALIDATE\_SAFETY\_DEPOSIT\_BOX\_V2SET\_AUTHORITYINIT\_AUCTION\_MANAGER\_V2UPDATE\_EXTERNAL\_PRICE\_ACCOUNTAUCTION\_HOUSE\_CREATECLOSE\_ESCROW\_ACCOUNTWITHDRAWDEPOSITTRANSFERBURNBURN\_NFTPLATFORM\_FEELOANRESCIND\_LOANOFFER\_LOANCANCEL\_OFFERLEND\_FOR\_NFTREQUEST\_LOANCANCEL\_LOAN\_REQUESTBORROW\_SOL\_FOR\_NFTCLAIM\_NFTREBORROW\_SOL\_FOR\_NFTREPAY\_LOANTAKE\_LOANFORECLOSE\_LOANUPDATE\_OFFERADD\_TO\_POOLREMOVE\_FROM\_POOLCLOSE\_POSITIONUNLABELEDCLOSE\_ACCOUNTWITHDRAW\_GEMDEPOSIT\_GEMSTAKE\_TOKENUNSTAKE\_TOKENSTAKE\_SOLUNSTAKE\_SOLCLAIM\_REWARDSBUY\_SUBSCRIPTIONSWAPINIT\_SWAPCANCEL\_SWAPREJECT\_SWAPINITIALIZE\_ACCOUNTTOKEN\_MINTCREATE\_APPRAISALCANDY\_MACHINE\_WRAPCANDY\_MACHINE\_UNWRAPCANDY\_MACHINE\_UPDATECANDY\_MACHINE\_ROUTEFRACTIONALIZEDEPOSIT\_FRACTIONAL\_POOLFUSECREATE\_RAFFLEBUY\_TICKETSUPDATE\_ITEMLIST\_ITEMDELIST\_ITEMADD\_ITEMCLOSE\_ITEMBUY\_ITEMFILL\_ORDERUPDATE\_ORDERCREATE\_ORDERCLOSE\_ORDERCANCEL\_ORDERKICK\_ITEMUPGRADE\_FOXUPGRADE\_FOX\_REQUESTLOAN\_FOXBORROW\_FOXSWITCH\_FOX\_REQUESTSWITCH\_FOXCREATE\_ESCROWACCEPT\_REQUEST\_ARTISTCANCEL\_ESCROWACCEPT\_ESCROW\_ARTISTACCEPT\_ESCROW\_USERPLACE\_BETPLACE\_SOL\_BETCREATE\_BETINIT\_RENTNFT\_RENT\_LISTINGNFT\_RENT\_CANCEL\_LISTINGNFT\_RENT\_UPDATE\_LISTINGNFT\_RENT\_ACTIVATENFT\_RENT\_ENDUPGRADE\_PROGRAM\_INSTRUCTIONFINALIZE\_PROGRAM\_INSTRUCTIONEXECUTE\_TRANSACTIONAPPROVE\_TRANSACTIONACTIVATE\_TRANSACTIONCREATE\_TRANSACTIONCANCEL\_TRANSACTIONREJECT\_TRANSACTIONADD\_INSTRUCTIONCREATE\_MASTER\_EDITIONATTACH\_METADATAREQUEST\_PNFT\_MIGRATIONSTART\_PNFT\_MIGRATIONMIGRATE\_TO\_PNFTUPDATE\_RAFFLECREATE\_MERKLE\_TREEDELEGATE\_MERKLE\_TREECOMPRESSED\_NFT\_MINTCOMPRESSED\_NFT\_TRANSFERCOMPRESSED\_NFT\_REDEEMCOMPRESSED\_NFT\_CANCEL\_REDEEMCOMPRESSED\_NFT\_BURNCOMPRESSED\_NFT\_VERIFY\_CREATORCOMPRESSED\_NFT\_UNVERIFY\_CREATORCOMPRESSED\_NFT\_VERIFY\_COLLECTIONCOMPRESSED\_NFT\_UNVERIFY\_COLLECTIONCOMPRESSED\_NFT\_SET\_VERIFY\_COLLECTIONDECOMPRESS\_NFTCOMPRESS\_NFTCOMPRESSED\_NFT\_DELEGATECREATE\_POOLDISTRIBUTE\_COMPRESSION\_REWARDSCHANGE\_COMIC\_STATEUPDATE\_RECORD\_AUTHORITY\_DATACREATE\_AVATAR\_CLASSCREATE\_AVATARCREATE\_TRAITCREATE\_PAYMENT\_METHODEQUIP\_TRAITEQUIP\_TRAIT\_AUTHORITYREMOVE\_TRAITREMOVE\_TRAIT\_AUTHORITYUPDATE\_TRAIT\_VARIANTUPDATE\_TRAIT\_VARIANT\_AUTHORITYUPDATE\_CLASS\_VARIANT\_AUTHORITYUPDATE\_TRAIT\_VARIANT\_METADATAUPDATE\_CLASS\_VARIANT\_METADATABEGIN\_VARIANT\_UPDATEBEGIN\_TRAIT\_UPDATECANCEL\_UPDATEUPDATE\_VARIANTTRANSFER\_PAYMENTBURN\_PAYMENTBURN\_PAYMENT\_TREETRANSFER\_PAYMENT\_TREEADD\_PAYMENT\_MINT\_PAYMENT\_METHODADD\_TRAIT\_CONFLICTSVERIFY\_PAYMENT\_MINTVERIFY\_PAYMENT\_MINT\_TESTBOUND\_HADO\_MARKET\_TO\_FRAKT\_MARKETDEPOSIT\_TO\_BOND\_OFFER\_STANDARDWITHDRAW\_FROM\_BOND\_OFFER\_STANDARDINITIALIZE\_HADO\_MARKETFINISH\_HADO\_MARKETUPDATE\_HADO\_MARKET\_FEEREMOVE\_BOND\_OFFER\_V2REPAY\_FBOND\_TO\_TRADE\_TRANSACTIONSEXIT\_VALIDATE\_AND\_SELL\_TO\_BOND\_OFFERS\_V2REFINANCE\_TO\_BOND\_OFFERS\_V2CREATE\_BOND\_AND\_SELL\_TO\_OFFERSLIQUIDATE\_BOND\_ON\_AUCTION\_PNFTCLAIM\_NFT\_BY\_LENDER\_PNFTCREATE\_BOND\_AND\_SELL\_TO\_OFFERS\_FOR\_TESTINITIALIZE\_FLASH\_LOAN\_POOLDEPOSIT\_SOL\_TO\_FLASH\_LOAN\_POOLWITHDRAW\_SOL\_FROM\_FLASH\_LOAN\_POOLTAKE\_FLASH\_LOANREPAY\_FLASH\_LOANCREATE\_BOND\_OFFER\_STANDARDUPDATE\_BOND\_OFFER\_STANDARDSTAKE\_BANXUNSTAKE\_BANXUNSUB\_OR\_HARVEST\_WEEKSUNSUB\_OR\_HARVEST\_WEEKS\_ENHANCEDUPDATE\_STAKING\_SETTINGSMAP\_BANX\_TO\_POINTSPATCH\_BROKEN\_USER\_STAKESDEPOSIT\_TO\_REWARDS\_VAULTWITHDRAW\_REWARDS\_FROM\_VAULTREFINANCE\_FBOND\_BY\_LENDERSELL\_STAKED\_BANX\_TO\_OFFERSREPAY\_STAKED\_BANXCREATE\_PERPETUAL\_BOND\_OFFERREMOVE\_PERPETUAL\_OFFERREPAY\_PERPETUAL\_LOANREFINANCE\_PERPETUAL\_LOANCREATE\_BOND\_AND\_SELL\_TO\_OFFERS\_CNFTREPAY\_FBOND\_TO\_TRADE\_TRANSACTIONS\_CNFTREFINANCE\_TO\_BOND\_OFFERS\_V2\_CNFTCLAIM\_NFT\_BY\_LENDER\_CNFTLIQUIDATE\_BOND\_ON\_AUCTION\_CNFTMAKE\_PERPETUAL\_MARKETUPDATE\_PERPETUAL\_MARKETUPDATE\_PERPETUAL\_OFFERUPDATE\_INTEREST\_PERPETUAL\_MARKETBORROW\_PERPETUALCLAIM\_PERPETUAL\_LOANTERMINATE\_PERPETUAL\_LOANINSTANT\_REFINANCE\_PERPETUAL\_LOANBORROW\_STAKED\_BANX\_PERPETUALREPAY\_STAKED\_BANX\_PERPETUAL\_LOANBORROW\_CNFT\_PERPETUALREPAY\_CNFT\_PERPETUAL\_LOANCLAIM\_CNFT\_PERPETUAL\_LOANREPAY\_PARTIAL\_PERPETUAL\_LOANCREATE\_COLLECTIONUPDATE\_COLLECTIONUPDATE\_COLLECTION\_OR\_CREATORUPDATE\_FLOORDELETE\_COLLECTIONCREATE\_STATSUPDATE\_STATSCLEANEXPIREFIX\_POOLADMIN\_SYNC\_LIQUIDITYCLOSE\_POOLUPDATE\_POOLUPDATE\_POOL\_COLLECTIONSUPDATE\_POOL\_STATUSUPDATE\_POOL\_MORTGAGEUPDATE\_USABLE\_AMOUNTUPDATE\_POOL\_WHITELISTADD\_LIQUIDITYSYNC\_LIQUIDITYWITHDRAW\_LIQUIDITYTAKE\_COMPRESSED\_LOANREPAYREPAY\_COMPRESSEDLIQUIDATETAKE\_MORTGAGEFREEZEUNFREEZESELL\_LOANBUY\_LOANEXTEND\_LOANSELL\_NFTPROPOSE\_LOANCANCEL\_PROPOSALPOOL\_CANCEL\_PROPOSALACCEPT\_PROPOSALEXECUTE\_LOANEXECUTE\_MORTGAGELIST\_NFTDELIST\_NFTCLAIM\_SALEBOT\_LIQUIDATEBOT\_UNFREEZEBOT\_LIQUIDATE\_SELLBOT\_DELISTBOT\_CLAIM\_SALEsourceTransactionSource (enum)FORM\_FUNCTIONEXCHANGE\_ARTCANDY\_MACHINE\_V3CANDY\_MACHINE\_V2CANDY\_MACHINE\_V1UNKNOWNSOLANARTSOLSEAMAGIC\_EDENHOLAPLEXMETAPLEXOPENSEASOLANA\_PROGRAM\_LIBRARYANCHORPHANTOMSYSTEM\_PROGRAMSTAKE\_PROGRAMCOINBASECORAL\_CUBEHEDGELAUNCH\_MY\_NFTGEM\_BANKGEM\_FARMDEGODSBSLYAWWWATADIADIGITAL\_EYESHYPERSPACETENSORBIFROSTJUPITERMERCURIALSABERSERUMSTEP\_FINANCECROPPERRAYDIUMALDRINCREMALIFINITYCYKURAORCAMARINADESTEPNSENCHASAROSENGLISH\_AUCTIONFOXYHADESWAPFOXY\_STAKINGFOXY\_RAFFLEFOXY\_TOKEN\_MARKETFOXY\_MISSIONSFOXY\_MARMALADEFOXY\_COINFLIPFOXY\_AUCTIONCITRUSZETAELIXIRELIXIR\_LAUNCHPADCARDINAL\_RENTCARDINAL\_STAKINGBPF\_LOADERBPF\_UPGRADEABLE\_LOADERSQUADSSHARKY\_FIOPEN\_CREATOR\_PROTOCOLBUBBLEGUMNOVAD\_READERRAINDROPSW\_SOLDUSTSOLIUSDCFLWRHDGMEANUXDSHDWPOLISATLASUSHTRTLSRUNNERINVICTUSfeeintegerExample: `5000`feePayerstringExample: `"8cRrU1NzNpjL3k2BwjW3VixAcX6VFc29KHr4KZg8cs2Y"`signaturestringExample: `"yy5BT9benHhx8fGCvhcAfTtLEHAtRJ3hRTzVL16bdrTCWm63t2vapfrZQZLJC3RcuagekaXjSs2zUGQvbcto8DK"`slotintegerExample: `148277128`timestampintegerExample: `1656442333`nativeTransfersarray of NativeTransfer (object)tokenTransfersarray of TokenTransfer (object)accountDataarray of AccountData (object)transactionErrorobjectinstructionsarray of Instruction (object)eventsobject

Events associated with this transaction. These provide fine-grained information about the transaction. They match the types returned from the event APIs.

RequestJavaScriptCurlCopy

```
const response = await fetch('https://api.helius.xyz/v0/addresses/{address}/transactions', {
    method: 'GET',
    headers: {},
});
const data = await response.json();
```

Test itResponseCopy

```
[
  {
    "description": "Human readable interpretation of the transaction",
    "type": "UNKNOWN",
    "source": "FORM_FUNCTION",
    "fee": 5000,
    "feePayer": "8cRrU1NzNpjL3k2BwjW3VixAcX6VFc29KHr4KZg8cs2Y",
    "signature": "yy5BT9benHhx8fGCvhcAfTtLEHAtRJ3hRTzVL16bdrTCWm63t2vapfrZQZLJC3RcuagekaXjSs2zUGQvbcto8DK",
    "slot": 148277128,
    "timestamp": 1656442333,
    "nativeTransfers": [
      {
        "fromUserAccount": "text",
        "toUserAccount": "text"
      }
    ],
    "tokenTransfers": [
      {
        "fromUserAccount": "text",
        "toUserAccount": "text",
        "fromTokenAccount": "text",
        "toTokenAccount": "text",
        "tokenAmount": 0,
        "mint": "DsfCsbbPH77p6yeLS1i4ag9UA5gP9xWSvdCx72FJjLsx"
      }
    ],
    "accountData": [
      {
        "account": "text",
        "nativeBalanceChange": 0,
        "tokenBalanceChanges": [
          {
            "userAccount": "F54ZGuxyb2gA7vRjzWKLWEMQqCfJxDY1whtqtjdq4CJ",
            "tokenAccount": "2kvmbRybhrcptDnwyNv6oiFGFEnRVv7MvVyqsxkirgdn",
            "mint": "DUSTawucrTsGU8hcqRdHDCbuYhCPADMLM2VcCb8VnFnQ",
            "rawTokenAmount": {
              "tokenAmount": "text"
            }
          }
        ]
      }
    ],
    "transactionError": {
      "error": "text"
    },
    "instructions": [
      {
        "accounts": [
          "8uX6yiUuH4UjUb1gMGJAdkXorSuKshqsFGDCFShcK88B"
        ],
        "data": "kdL8HQJrbbvQRGXmoadaja1Qvs",
        "programId": "MEisE1HzehtrDpAAT8PnLHjpSSkRYakotTuJRPjTpo8",
        "innerInstructions": [
          {
            "accounts": [
              "text"
            ],
            "data": "text",
            "programId": "text"
          }
        ]
      }
    ],
    "events": {
      "nft": {
        "description": "text",
        "type": "NFT_BID",
        "source": "FORM_FUNCTION",
        "amount": 1000000,
        "fee": 5000,
        "feePayer": "8cRrU1NzNpjL3k2BwjW3VixAcX6VFc29KHr4KZg8cs2Y",
        "signature": "4jzQxVTaJ4Fe4Fct9y1aaT9hmVyEjpCqE2bL8JMnuLZbzHZwaL4kZZvNEZ6bEj6fGmiAdCPjmNQHCf8v994PAgDf",
        "slot": 148277128,
        "timestamp": 1656442333,
        "saleType": "AUCTION",
        "buyer": "text",
        "seller": "text",
        "staker": "text",
        "nfts": [
          {
            "mint": "DsfCsbbPH77p6yeLS1i4ag9UA5gP9xWSvdCx72FJjLsx",
            "tokenStandard": "NonFungible"
          }
        ]
      },
      "swap": {
        "nativeInput": {
          "account": "2uySTNgvGT2kwqpfgLiSgeBLR3wQyye1i1A2iQWoPiFr",
          "amount": "100000000"
        },
        "nativeOutput": {
          "account": "2uySTNgvGT2kwqpfgLiSgeBLR3wQyye1i1A2iQWoPiFr",
          "amount": "100000000"
        },
        "tokenInputs": [
          {
            "userAccount": "F54ZGuxyb2gA7vRjzWKLWEMQqCfJxDY1whtqtjdq4CJ",
            "tokenAccount": "2kvmbRybhrcptDnwyNv6oiFGFEnRVv7MvVyqsxkirgdn",
            "mint": "DUSTawucrTsGU8hcqRdHDCbuYhCPADMLM2VcCb8VnFnQ",
            "rawTokenAmount": {
              "tokenAmount": "text"
            }
          }
        ],
        "tokenOutputs": [
          {
            "userAccount": "F54ZGuxyb2gA7vRjzWKLWEMQqCfJxDY1whtqtjdq4CJ",
            "tokenAccount": "2kvmbRybhrcptDnwyNv6oiFGFEnRVv7MvVyqsxkirgdn",
            "mint": "DUSTawucrTsGU8hcqRdHDCbuYhCPADMLM2VcCb8VnFnQ",
            "rawTokenAmount": {
              "tokenAmount": "text"
            }
          }
        ],
        "tokenFees": [
          {
            "userAccount": "F54ZGuxyb2gA7vRjzWKLWEMQqCfJxDY1whtqtjdq4CJ",
            "tokenAccount": "2kvmbRybhrcptDnwyNv6oiFGFEnRVv7MvVyqsxkirgdn",
            "mint": "DUSTawucrTsGU8hcqRdHDCbuYhCPADMLM2VcCb8VnFnQ",
            "rawTokenAmount": {
              "tokenAmount": "text"
            }
          }
        ],
        "nativeFees": [
          {
            "account": "2uySTNgvGT2kwqpfgLiSgeBLR3wQyye1i1A2iQWoPiFr",
            "amount": "100000000"
          }
        ],
        "innerSwaps": [
          {
            "tokenInputs": [
              {
                "fromUserAccount": "text",
                "toUserAccount": "text",
                "fromTokenAccount": "text",
                "toTokenAccount": "text",
                "tokenAmount": 0,
                "mint": "DsfCsbbPH77p6yeLS1i4ag9UA5gP9xWSvdCx72FJjLsx"
              }
            ],
            "tokenOutputs": [
              {
                "fromUserAccount": "text",
                "toUserAccount": "text",
                "fromTokenAccount": "text",
                "toTokenAccount": "text",
                "tokenAmount": 0,
                "mint": "DsfCsbbPH77p6yeLS1i4ag9UA5gP9xWSvdCx72FJjLsx"
              }
            ],
            "tokenFees": [
              {
                "fromUserAccount": "text",
                "toUserAccount": "text",
                "fromTokenAccount": "text",
                "toTokenAccount": "text",
                "tokenAmount": 0,
                "mint": "DsfCsbbPH77p6yeLS1i4ag9UA5gP9xWSvdCx72FJjLsx"
              }
            ],
            "nativeFees": [
              {
                "fromUserAccount": "text",
                "toUserAccount": "text"
              }
            ],
            "programInfo": {
              "source": "ORCA",
              "account": "whirLbMiicVdio4qvUfM5KAg6Ct8VwpYzGff3uctyCc",
              "programName": "ORCA_WHIRLPOOLS",
              "instructionName": "whirlpoolSwap"
            }
          }
        ]
      },
      "compressed": {
        "type": "COMPRESSED_NFT_MINT",
        "treeId": "text",
        "assetId": "text",
        "newLeafOwner": "text",
        "oldLeafOwner": "text"
      },
      "distributeCompressionRewards": {},
      "setAuthority": {
        "account": "text",
        "from": "text",
        "to": "text"
      }
    }
  }
]
```

#### Code Samples <a href="#code-samples" id="code-samples"></a>

**Basic Example**

parseTransactions.jsCopy

```
const url = "https://api.helius.xyz/v0/addresses/M2mx93ekt1fmXSVkTrUL9xVFHkmME8HTUi5Cyc5aF7K/transactions?api-key=<your-key>";

const parseTransactions = async () => {
  const response = await fetch(url);
  const data = await response.json();
  console.log("parsed transactions: ", data);
};

parseTransactions();
```

**NFT Sale Transaction Example**

Copy

```
const tokenAddress = "GjUG1BATg5V4bdAr1csKys1XK9fmrbntgb1iV7rAkn94"
// NFT TRANSACTION SEARCH
const url = `https://api.helius.xyz/v0/addresses/${tokenAddress}/transactions?api-key=${apiKey}&type=NFT_SALE`

const parseNFT = async () => {
  const response = await fetch(url);
  const data = await response.json();
  console.log("nft transactions: ", data);
};

parseNFT();
```

**Wallet Transaction Example**

Copy

```
const walletAddress = "2k5AXX4guW9XwRQ1AKCpAuUqgWDpQpwFfpVFh3hnm2Ha"
// WALLET TRANSACTION SEARCH
const url = `https://api.helius.xyz/v0/addresses/${walletAddress}/transactions?api-key=${apiKey}`

const parseWallet = async () => {
  const response = await fetch(url);
  const data = await response.json();
  console.log("wallet transactions: ", data);
};

parseWallet();
```

**All Transactions Using Pagination**

Copy

```
let base_url = `https://api.helius.xyz/v0/addresses/2k5AXX4guW9XwRQ1AKCpAuUqgWDpQpwFfpVFh3hnm2Ha/transactions?api-key=<api-key>`;
let url = base_url;
let lastSignature = null;

const fetchAndParseTransactions = async () => {
  while (true) {
    if (lastSignature) {
      url = base_url + `&before=${lastSignature}`;
    }
    const response = await fetch(url);
    const transactions = await response.json();

    if (transactions && transactions.length > 0) {
      console.log("Fetched transactions: ", transactions);
      lastSignature = transactions[transactions.length - 1].signature;
    } else {
      console.log("No more transactions available.");
      break;
    }
  }
};
fetchAndParseTransactions();
```



## Token Balances API

Get a list of assets owned by a Solana address.

#### **Overview** <a href="#overview" id="overview"></a>

This will return a list of assets for the specified owner. Supported assets include NFTs and compressed NFTs (regular DAS), as well as fungible tokens and Token22 (Helius extension).

This method is the fastest way to return **all** assets belonging to a wallet.

The **`page`** parameter in starts at **1 .**

#### Fungible Token Extension <a href="#fungible-token-extension" id="fungible-token-extension"></a>

DAS API traditionally has returned data for NFTs — at Helius, we've also added support for all tokens. To learn more, please visit [Fungible Token Extension (Beta)](https://docs.helius.dev/compression-and-das-api/digital-asset-standard-das-api/fungible-token-extension-beta).

This feature can be enabled via the `showFungible` filter. When enabled, the response will include fungible tokens and their associated information (supply, balance, and price). Token22 tokens are supported and their extensions are parsed. For richer options, consider using [Search Assets](https://docs.helius.dev/compression-and-das-api/digital-asset-standard-das-api/search-assets).

#### Suggested Use Cases <a href="#suggested-use-cases" id="suggested-use-cases"></a>

The Fungible Token Extension also returns the USD prices of token holdings.

Using 'Get Assets by Owner' to return all tokens from a user wallet enables the creation of::

* A wallet tracker.
* A portfolio viewer for both fungible and non-fungible tokens.
* A token-gated dApp.

#### Example <a href="#example" id="example"></a>

**Get NFTs & compressed NFTs from the toly.sol wallet:**

Copy

```
const url = `https://mainnet.helius-rpc.com/?api-key=<api_key>`

const getAssetsByOwner = async () => {
  const response = await fetch(url, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      jsonrpc: '2.0',
      id: 'my-id',
      method: 'getAssetsByOwner',
      params: {
        ownerAddress: '86xCnPeV69n6t3DnyGvkKobf9FdN2H9oiVDdaMpo2MMY',
        page: 1, // Starts at 1
        limit: 1000,
      },
    }),
  });
  const { result } = await response.json();
  console.log("Assets by Owner: ", result.items);
};
getAssetsByOwner(); 
```

**Get ALL assets from the toly.sol wallet:**

Copy

```

const getAssetsByOwner = async () => {
  const response = await fetch(url, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      jsonrpc: '2.0',
      id: 'my-id',
      method: 'getAssetsByOwner',
      params: {
        ownerAddress: '86xCnPeV69n6t3DnyGvkKobf9FdN2H9oiVDdaMpo2MMY',
        page: 1, // Starts at 1
        limit: 1000,
	displayOptions: {
	    showFungible: true //return both fungible and non-fungible tokens
	}
      },
    }),
  });
  const { result } = await response.json();
  console.log("Assets by Owner: ", result.items);
};
getAssetsByOwner(); 
```



## SDKs

SDKs for building the future of Solana

At Helius, we've developed a [Node.js](https://github.com/helius-labs/helius-sdk) and a [Rust SDK](https://github.com/helius-labs/helius-rust-sdk) to make developing on Solana easier. The following page includes information on installing and using these SDKs. It also covers common error handling, where to find the latest documentation, and how to contribute to these SDKs. We also outline a list of unofficial community SDKs made by our wonderful community. Note that those SDKs are not officially maintained by our team — only the Node.js and Rust SDKs are

### Node.js SDK <a href="#node.js-sdk" id="node.js-sdk"></a>

#### Installation <a href="#installation" id="installation"></a>

The Helius Node.js SDK can be installed with any of the following package managers:

| Package Manager               | Command                   |
| ----------------------------- | ------------------------- |
| [npm](https://www.npmjs.com/) | `npm install helius-sdk`  |
| [pnpm](https://pnpm.io/)      | `pnpm install helius-sdk` |
| [Yarn](https://yarnpkg.com/)  | `yarn add helius-sdk`     |

#### Quick Start <a href="#quick-start" id="quick-start"></a>

Here's a straightforward example of how to use the Node.js SDK to fetch a list of assets owned by a given address:

Copy

```
import { Helius } from "helius-sdk";

const helius = new Helius("YOUR_API_KEY");
const response = await helius.rpc.getAssetsByOwner({
  ownerAddress: "86xCnPeV69n6t3DnyGvkKobf9FdN2H9oiVDdaMpo2MMY",
  page: 1,
});

console.log(response.items);
```

#### Documentation <a href="#documentation" id="documentation"></a>

The [README file](https://github.com/helius-labs/helius-sdk/blob/main/README.md) is filled with in-depth code examples covering each method and basic usage. For API reference documentation, refer to our [documentation](https://docs.helius.dev/) and the [official Solana documentation](https://solana.com/docs/rpc) for general Solana JSON RPC API help.

### Rust SDK <a href="#rust-sdk" id="rust-sdk"></a>

#### Installation <a href="#installation-1" id="installation-1"></a>

To start using the Helius Rust SDK in your project, add it as a dependency via [`cargo`](https://doc.rust-lang.org/cargo/). Open your project's `Cargo.toml` and add the following line under `[dependencies]`:

Copy

```
helius = "x.y.z"
```

where `x.y.z` is your desired version. Alternatively, use `cargo add helius` to add the dependency directly via the command line. This will automatically find the latest version compatible with your project and add it to your `Cargo.toml`.

Remember to run `cargo update` regularly to fetch the latest version of the SDK.

#### Quick Start <a href="#quick-start-1" id="quick-start-1"></a>

Here is a straightforward example of using the [Enhanced Transactions API](https://docs.helius.dev/solana-apis/enhanced-transactions-api) to [parse a given transaction](https://docs.helius.dev/solana-apis/enhanced-transactions-api/parse-transaction-s):

Copy

```
use helius::error::Result;
use helius::types::*;
use helius::Helius;

#[tokio::main]
async fn main() -> Result<()> {
    let api_key: &str = "your_api_key";
    let cluster: Cluster = Cluster::MainnetBeta;

    let helius: Helius = Helius::new(api_key, cluster).unwrap();

    let request: ParseTransactionsRequest = ParseTransactionsRequest {
        transactions: vec![
            "2sShYqqcWAcJiGc3oK74iFsYKgLCNiY2DsivMbaJGQT8pRzR8z5iBcdmTMXRobH8cZNZgeV9Ur9VjvLsykfFE2Li".to_string(),
        ],
    };

    let response: Result<Vec<EnhancedTransaction>, HeliusError> = helius.parse_transactions(request).await;
    println!("Assets: {:?}", response);

    Ok(())
}
```

#### Documentation <a href="#documentation-1" id="documentation-1"></a>

The latest documentation can be found [here on docs.rs](https://docs.rs/helius/latest/helius/). For API reference documentation, refer to our [documentation](https://docs.helius.dev/) and the [official Solana documentation](https://solana.com/docs/rpc) for general Solana JSON RPC API help.

More examples of how to use the SDK can be found in the [`examples`](https://github.com/helius-labs/helius-rust-sdk/tree/dev/examples) directory.

### Error Handling <a href="#error-handling" id="error-handling"></a>

An error message will be thrown when the API returns a non-success (i.e., 4xx or 5xx status code). For example:

Copy

```
// From our Node.js SDK
try {
  const response = await helius.rpc.getAssetsByOwner({
    ownerAddress: "86xCnPeV69n6t3DnyGvkKobf9FdN2H9oiVDdaMpo2MMY",
    page: 1,
  });
  console.log(response.items);
} catch (error) {
  console.log(error);
}
```

#### Common Error Codes <a href="#common-error-codes" id="common-error-codes"></a>

When working with the Helius SDK, you may encounter several error codes. Below is a table detailing some of the common error codes along with additional information to help you troubleshoot:

| Error Code | Error Message         | More Information                                                                                       |
| ---------- | --------------------- | ------------------------------------------------------------------------------------------------------ |
| 401        | Unauthorized          | This occurs when an invalid API key is provided or access is restricted due to RPC rules.              |
| 429        | Too Many Requests     | This indicates that the user has exceeded the request limit in a given timeframe or is out of credits. |
| 5XX        | Internal Server Error | This is a generic error message for server-side issues. Please contact Helius support for assistance.  |

If you encounter any of these errors:

* Refer to [`errors.rs`](https://github.com/helius-labs/helius-rust-sdk/blob/dev/src/error.rs) for a list of all possible errors returned by the `Helius` client, if using the Rust SDK
* Refer to the [Helius documentation](https://docs.helius.dev/) for further guidance
* Reach out to the Helius support team for more detailed assistance

### Contribution to Our SDKs <a href="#contribution-to-our-sdks" id="contribution-to-our-sdks"></a>

We welcome all contributions to our SDKs! If you're interested, here are our GitHub Repositories:

* [Node.js SDK](https://github.com/helius-labs/helius-sdk)
* [Rust SDK](https://github.com/helius-labs/helius-rust-sdk)
  * Interested in contributing to the Helius Rust SDK specifically? Read the following [contributions guide](https://github.com/helius-labs/helius-rust-sdk/blob/dev/CONTRIBUTIONS.md) before opening up a pull request!

### Unofficial Community SDKs <a href="#unofficial-community-sdks" id="unofficial-community-sdks"></a>

Our amazing community members have also created their own SDKs to interact with our REST APIs. Please note these are not officially maintained by our team. Unofficial community SDKs in other languages include:

* [Kotlin SDK](https://github.com/dlgrech/khelius)
* [PHP SDK](https://github.com/HowRareIs/helius-php-sdk)
* [Python SDK](https://github.com/vmpyre/helius\_sdk)
* [Rust SDK (Synchronous)](https://github.com/bgreni/helius-rust-sdk)
* [Rust SDK (Asynchronous)](https://github.com/dougEfresh/selene-helius-sdk)

### Start Building Today <a href="#start-building-today" id="start-building-today"></a>

[Helius - The Developer Platform for Solanaheliuslabs](https://dev.helius.xyz/dashboard/app)



## Transaction Types

Decoded transactions.

#### Request a Transaction Type <a href="#request-a-transaction-type" id="request-a-transaction-type"></a>

If you'd like a certain transaction type parsed that's not in our list below, [please shoot us a message on our Discord](https://t.co/uvMcyIhFpQ) and we'll be more than happy to take a look!

#### Parsed Transaction Types <a href="#parsed-transaction-types" id="parsed-transaction-types"></a>

See below for the full list of transaction types.

<details>

<summary>Transaction Types</summary>

Copy

```
- UNKNOWN
- NFT_BID 
- NFT_BID_CANCELLED
- NFT_LISTING
- NFT_CANCEL_LISTING
- NFT_SALE
- NFT_MINT
- NFT_AUCTION_CREATED
- NFT_AUCTION_UPDATED
- NFT_AUCTION_CANCELLED
- NFT_PARTICIPATION_REWARD 
- NFT_MINT_REJECTED
- CREATE_STORE
- WHITELIST_CREATOR
- ADD_TO_WHITELIST
- REMOVE_FROM_WHITELIST
- AUCTION_MANAGER_CLAIM_BID
- EMPTY_PAYMENT_ACCOUNT
- UPDATE_PRIMARY_SALE_METADATA
- ADD_TOKEN_TO_VAULT
- ACTIVATE_VAULT
- INIT_VAULT
- INIT_BANK
- INIT_STAKE
- MERGE_STAKE
- SPLIT_STAKE
- SET_BANK_FLAGS
- SET_VAULT_LOCK
- UPDATE_VAULT_OWNER
- UPDATE_BANK_MANAGER
- RECORD_RARITY_POINTS
- ADD_RARITIES_TO_BANK
- INIT_FARM
- INIT_FARMER
- REFRESH_FARMER
- UPDATE_FARM
- AUTHORIZE_FUNDER
- DEAUTHORIZE_FUNDER
- FUND_REWARD
- CANCEL_REWARD
- LOCK_REWARD
- PAYOUT
- VALIDATE_SAFETY_DEPOSIT_BOX_V2
- SET_AUTHORITY
- INIT_AUCTION_MANAGER_V2
- UPDATE_EXTERNAL_PRICE_ACCOUNT
- AUCTION_HOUSE_CREATE
- CLOSE_ESCROW_ACCOUNT
- WITHDRAW
- DEPOSIT 
- TRANSFER
- BURN
- BURN_NFT
- PLATFORM_FEE
- LOAN
- REPAY_LOAN
- ADD_TO_POOL
- REMOVE_FROM_POOL
- CLOSE_POSITION
- UNLABELED
- CLOSE_ACCOUNT
- WITHDRAW_GEM
- DEPOSIT_GEM
- STAKE_TOKEN
- UNSTAKE_TOKEN
- STAKE_SOL
- UNSTAKE_SOL
- CLAIM_REWARDS
- BUY_SUBSCRIPTION
- SWAP
- INIT_SWAP
- CANCEL_SWAP
- REJECT_SWAP
- INITIALIZE_ACCOUNT
- TOKEN_MINT
- CREATE_APPARAISAL
- FUSE
- DEPOSIT_FRACTIONAL_POOL
- FRACTIONALIZE
- CREATE_RAFFLE
- BUY_TICKETS
- UPDATE_ITEM
- LIST_ITEM
- DELIST_ITEM
- ADD_ITEM
- CLOSE_ITEM
- BUY_ITEM
- FILL_ORDER
- UPDATE_ORDER
- CREATE_ORDER
- CLOSE_ORDER
- CANCEL_ORDER
- KICK_ITEM
- UPGRADE_FOX
- UPGRADE_FOX_REQUEST
- LOAN_FOX
- BORROW_FOX
- SWITCH_FOX_REQUEST
- SWITCH_FOX
- CREATE_ESCROW
- ACCEPT_REQUEST_ARTIST
- CANCEL_ESCROW
- ACCEPT_ESCROW_ARTIST
- ACCEPT_ESCROW_USER
- PLACE_BET
- PLACE_SOL_BET
- CREATE_BET
- NFT_RENT_UPDATE_LISTING
- NFT_RENT_ACTIVATE
- NFT_RENT_CANCEL_LISTING
- NFT_RENT_LISTING
- FINALIZE_PROGRAM_INSTRUCTION
- UPGRADE_PROGRAM_INSTRUCTION
- NFT_GLOBAL_BID
- NFT_GLOBAL_BID_CANCELLED
- EXECUTE_TRANSACTION
- APPROVE_TRANSACTION
- ACTIVATE_TRANSACTION
- CREATE_TRANSACTION
- REJECT_TRANSACTION
- CANCEL_TRANSACTION
- ADD_INSTRUCTION
- ATTACH_METADATA
- REQUEST_PNFT_MIGRATION
- START_PNFT_MIGRATION
- MIGRATE_TO_PNFT
- UPDATE_RAFFLE
- CREATE_POOL
- ADD_LIQUIDITY
- WITHDRAW_LIQUIDITY
```

</details>

#### Relation between Transaction Type and Source <a href="#relation-between-transaction-type-and-source" id="relation-between-transaction-type-and-source"></a>

The mappings below show which sources (programs) generate a specific transaction type and vice-versa.

For example, the **Type to Source** mapping tells us that NFT\_GLOBAL\_BID is generated by MAGIC\_EDEN or SOLANART. And the **Source to Type** mapping says that CANDY\_MACHINE\_V2 only generates NFT\_MINT events.

<details>

<summary>Type to Source</summary>

Copy

```
{
  "NFT_MINT": [
    "CANDY_MACHINE_V2",
    "CANDY_MACHINE_V1",
    "CANDY_MACHINE_V3",
    "FORM_FUNCTION",
    "MAGIC_EDEN",
    "LAUNCH_MY_NFT",
    "BIFROST",
    "ATADIA",
    "ELIXIR_LAUNCHPAD",
    "SOLANA_PROGRAM_LIBRARY",
    "METAPLEX"
  ],
  "TOKEN_MINT": [
    "CANDY_MACHINE_V1",
    "ATADIA",
    "SOLANA_PROGRAM_LIBRARY"
  ],
  "CANDY_MACHINE_UPDATE": [
    "CANDY_MACHINE_V3"
  ],
  "CANDY_MACHINE_ROUTE": [
    "CANDY_MACHINE_V3"
  ],
  "CANDY_MACHINE_WRAP": [
    "CANDY_MACHINE_V3"
  ],
  "CANDY_MACHINE_UNWRAP": [
    "CANDY_MACHINE_V3"
  ],
  "NFT_BID": [
    "FORM_FUNCTION",
    "EXCHANGE_ART",
    "SOLANART",
    "MAGIC_EDEN",
    "ENGLISH_AUCTION",
    "YAWWW",
    "HYPERSPACE",
    "METAPLEX",
    "FOXY_AUCTION"
  ],
  "NFT_SALE": [
    "FORM_FUNCTION",
    "EXCHANGE_ART",
    "SOLANART",
    "MAGIC_EDEN",
    "ENGLISH_AUCTION",
    "SOLSEA",
    "YAWWW",
    "DIGITAL_EYES",
    "HYPERSPACE",
    "TENSOR",
    "METAPLEX",
    "FOXY_AUCTION"
  ],
  "NFT_LISTING": [
    "FORM_FUNCTION",
    "EXCHANGE_ART",
    "SOLANART",
    "MAGIC_EDEN",
    "SOLSEA",
    "YAWWW",
    "HYPERSPACE",
    "TENSOR",
    "METAPLEX"
  ],
  "NFT_CANCEL_LISTING": [
    "EXCHANGE_ART",
    "SOLANART",
    "MAGIC_EDEN",
    "SOLSEA",
    "YAWWW",
    "HYPERSPACE",
    "TENSOR"
  ],
  "NFT_BID_CANCELLED": [
    "EXCHANGE_ART",
    "SOLANART",
    "MAGIC_EDEN",
    "YAWWW",
    "HYPERSPACE",
    "METAPLEX"
  ],
  "NFT_GLOBAL_BID": [
    "SOLANART",
    "MAGIC_EDEN"
  ],
  "NFT_GLOBAL_BID_CANCELLED": [
    "SOLANART",
    "MAGIC_EDEN"
  ],
  "WITHDRAW": [
    "MAGIC_EDEN",
    "BIFROST",
    "STAKE_PROGRAM"
  ],
  "DEPOSIT": [
    "MAGIC_EDEN"
  ],
  "NFT_AUCTION_CREATED": [
    "ENGLISH_AUCTION",
    "METAPLEX",
    "FOXY_AUCTION"
  ],
  "NFT_AUCTION_UPDATED": [
    "ENGLISH_AUCTION"
  ],
  "NFT_AUCTION_CANCELLED": [
    "ENGLISH_AUCTION",
    "FOXY_AUCTION"
  ],
  "TRANSFER": [
    "PHANTOM",
    "SOLANA_PROGRAM_LIBRARY",
    "SYSTEM_PROGRAM"
  ],
  "INIT_BANK": [
    "GEM_BANK",
    "DEGODS",
    "BLOCKSMITH_LABS"
  ],
  "SET_BANK_FLAGS": [
    "GEM_BANK",
    "DEGODS",
    "BLOCKSMITH_LABS"
  ],
  "INIT_VAULT": [
    "GEM_BANK",
    "DEGODS",
    "BLOCKSMITH_LABS",
    "METAPLEX"
  ],
  "SET_VAULT_LOCK": [
    "GEM_BANK",
    "DEGODS",
    "BLOCKSMITH_LABS"
  ],
  "UPDATE_VAULT_OWNER": [
    "GEM_BANK",
    "DEGODS",
    "BLOCKSMITH_LABS"
  ],
  "DEPOSIT_GEM": [
    "GEM_BANK",
    "GEM_FARM",
    "DEGODS",
    "BLOCKSMITH_LABS"
  ],
  "WITHDRAW_GEM": [
    "GEM_BANK",
    "DEGODS",
    "BLOCKSMITH_LABS"
  ],
  "ADD_TO_WHITELIST": [
    "GEM_BANK",
    "GEM_FARM",
    "DEGODS",
    "BLOCKSMITH_LABS"
  ],
  "REMOVE_FROM_WHITELIST": [
    "GEM_BANK",
    "GEM_FARM",
    "DEGODS",
    "BLOCKSMITH_LABS"
  ],
  "UPDATE_BANK_MANAGER": [
    "GEM_BANK",
    "DEGODS",
    "BLOCKSMITH_LABS"
  ],
  "RECORD_RARITY_POINTS": [
    "GEM_BANK",
    "DEGODS",
    "BLOCKSMITH_LABS"
  ],
  "INIT_FARM": [
    "GEM_FARM",
    "DEGODS",
    "BLOCKSMITH_LABS"
  ],
  "UPDATE_FARM": [
    "GEM_FARM",
    "DEGODS",
    "BLOCKSMITH_LABS"
  ],
  "PAYOUT": [
    "GEM_FARM",
    "DEGODS",
    "BLOCKSMITH_LABS"
  ],
  "STAKE_TOKEN": [
    "GEM_FARM",
    "DEGODS",
    "BLOCKSMITH_LABS",
    "FOXY_STAKING",
    "CARDINAL_STAKING"
  ],
  "UNSTAKE_TOKEN": [
    "GEM_FARM",
    "DEGODS",
    "BLOCKSMITH_LABS",
    "FOXY_STAKING",
    "CARDINAL_STAKING"
  ],
  "CLAIM_REWARDS": [
    "GEM_FARM",
    "DEGODS",
    "BLOCKSMITH_LABS"
  ],
  "INIT_FARMER": [
    "GEM_FARM",
    "DEGODS",
    "BLOCKSMITH_LABS"
  ],
  "REFRESH_FARMER": [
    "GEM_FARM",
    "DEGODS",
    "BLOCKSMITH_LABS"
  ],
  "AUTHORIZE_FUNDER": [
    "GEM_FARM",
    "DEGODS",
    "BLOCKSMITH_LABS"
  ],
  "DEAUTHORIZE_FUNDER": [
    "GEM_FARM",
    "DEGODS",
    "BLOCKSMITH_LABS"
  ],
  "FUND_REWARD": [
    "GEM_FARM",
    "DEGODS",
    "BLOCKSMITH_LABS"
  ],
  "CANCEL_REWARD": [
    "GEM_FARM",
    "DEGODS",
    "BLOCKSMITH_LABS"
  ],
  "LOCK_REWARD": [
    "GEM_FARM",
    "DEGODS",
    "BLOCKSMITH_LABS"
  ],
  "ADD_RARITIES_TO_BANK": [
    "GEM_FARM",
    "DEGODS",
    "BLOCKSMITH_LABS"
  ],
  "BUY_SUBSCRIPTION": [
    "YAWWW"
  ],
  "FUSE": [
    "ELIXIR"
  ],
  "SWAP": [
    "ELIXIR",
    "JUPITER",
    "FOXY",
    "ALDRIN",
    "HADESWAP",
    "RAYDIUM"
  ],
  "DEPOSIT_FRACTIONAL_POOL": [
    "ELIXIR"
  ],
  "FRACTIONALIZE": [
    "ELIXIR"
  ],
  "CREATE_APPRAISAL": [
    "ELIXIR"
  ],
  "addCollateralType": [
    "HEDGE"
  ],
  "AUCTION_HOUSE_CREATE": [
    "METAPLEX"
  ],
  "CLOSE_ESCROW_ACCOUNT": [
    "METAPLEX"
  ],
  "AUCTION_MANAGER_CLAIM_BID": [
    "METAPLEX"
  ],
  "EMPTY_PAYMENT_ACCOUNT": [
    "METAPLEX"
  ],
  "NFT_PARTICIPATION_REWARD": [
    "METAPLEX"
  ],
  "VALIDATE_SAFETY_DEPOSIT_BOX_V2": [
    "METAPLEX"
  ],
  "INIT_AUCTION_MANAGER_V2": [
    "METAPLEX"
  ],
  "SET_AUTHORITY": [
    "METAPLEX"
  ],
  "CREATE_STORE": [
    "METAPLEX"
  ],
  "WHITELIST_CREATOR": [
    "METAPLEX"
  ],
  "CREATE_RAFFLE": [
    "FOXY_RAFFLE"
  ],
  "UPDATE_RAFFLE": [
    "FOXY_RAFFLE"
  ],
  "BUY_TICKETS": [
    "FOXY_RAFFLE"
  ],
  "ADD_ITEM": [
    "FOXY_TOKEN_MARKET"
  ],
  "UPGRADE_FOX": [
    "FOXY_MISSIONS"
  ],
  "CREATE_ESCROW": [
    "FOXY_MARMALADE"
  ],
  "CREATE_BET": [
    "FOXY_COINFLIP"
  ],
  "NFT_RENT_LISTING": [
    "CARDINAL_RENT"
  ],
  "NFT_RENT_ACTIVATE": [
    "CARDINAL_RENT"
  ],
  "NFT_RENT_CANCEL_LISTING": [
    "CARDINAL_RENT"
  ],
  "NFT_RENT_UPDATE_LISTING": [
    "CARDINAL_RENT"
  ],
  "EXECUTE_TRANSACTION": [
    "SQUADS"
  ],
  "CREATE_TRANSACTION": [
    "SQUADS"
  ],
  "APPROVE_TRANSACTION": [
    "SQUADS"
  ],
  "ACTIVATE_TRANSACTION": [
    "SQUADS"
  ],
  "REJECT_TRANSACTION": [
    "SQUADS"
  ],
  "CANCEL_TRANSACTION": [
    "SQUADS"
  ],
  "ADD_INSTRUCTION": [
    "SQUADS"
  ],
  "BURN": [
    "SOLANA_PROGRAM_LIBRARY"
  ],
  "UPDATE_PRIMARY_SALE_METADATA": [
    "METAPLEX"
  ],
  "BURN_NFT": [
    "METAPLEX"
  ],
  "ADD_TOKEN_TO_VAULT": [
    "METAPLEX"
  ],
  "ACTIVATE_VAULT": [
    "METAPLEX"
  ],
  "UPDATE_EXTERNAL_PRICE_ACCOUNT": [
    "METAPLEX"
  ],
  "STAKE_SOL": [
    "STAKE_PROGRAM"
  ],
  "UNSTAKE_SOL": [
    "STAKE_PROGRAM"
  ],
  "INIT_STAKE": [
    "STAKE_PROGRAM"
  ],
  "MERGE_STAKE": [
    "STAKE_PROGRAM"
  ],
  "SPLIT_STAKE": [
    "STAKE_PROGRAM"
  ],
  "UPGRADE_PROGRAM_INSTRUCTION": [
    "BPF_UPGRADEABLE_LOADER"
  ],
  "FINALIZE_PROGRAM_INSTRUCTION": [
    "BPF_LOADER"
  ],
  "REQUEST_PNFT_MIGRATION": [
    "METAPLEX"
  ],
  "START_PNFT_MIGRATION": [
    "METAPLEX"
  ],
  "MIGRATE_TO_PNFT": [
    "METAPLEX"
  ],
  "OFFER_LOAN": [
    "SHARKY_FI",
    "CITRUS"
  ],
  "RESCIND_LOAN": [
    "SHARKY_FI"
  ],
  "REPAY_LOAN": [
    "SHARKY_FI"
  ],
  "TAKE_LOAN": [
    "SHARKY_FI"
  ],
  "FORECLOSE_LOAN": [
    "SHARKY_FI"
  ],
  "CANCEL_OFFER": [
    "CITRUS"
  ],
  "LEND_FOR_NFT": [
    "CITRUS"
  ],
  "REQUEST_LOAN": [
    "CITRUS"
  ],
  "CANCEL_LOAN_REQUEST": [
    "CITRUS"
  ],
  "BORROW_SOL_FOR_NFT": [
    "CITRUS"
  ],
  "REPAY_LOAN": [
    "CITRUS"
  ],
  "CLAIM_NFT": [
    "CITRUS"
  ],
  "REBORROW_SOL_FOR_NFT": [
    "CITRUS"
  ],
  "UPDATE_OFFER": [
    "CITRUS"
  ],
  "CREATE_POOL":[
    "RAYDIUM"
  ],
  "ADD_LIQUIDITY":[
    "RAYDIUM"
  ],
  "CREATE_POOL":[
    "RAYDIUM"
  ],
  "WITHDRAW_LIQUIDITY":[
    "RAYDIUM"
  ]
}
```



</details>

* [Request a Transaction Type](https://docs.helius.dev/resources/transaction-types#request-a-transaction-type)
* [Parsed Transaction Types](https://docs.helius.dev/resources/transaction-types#parsed-transaction-types)
* [Relation between Transaction Type and Source](https://docs.helius.dev/resources/transaction-types#relation-between-transaction-type-and-source)
