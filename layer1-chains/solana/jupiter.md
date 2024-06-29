# Jupiter

## V6 Swap API

Jupiter API is the easiest way for developers to access liquidity on Solana. Simply pass in the desired pairs, amount, and slippage, and the API will return the serialized transactions needed to execute the swap, which can then be passed into the Solana blockchain with the required signatures.

#### Swap and Quote API requests[​](https://station.jup.ag/docs/apis/swap-api#swap-and-quote-api-requests) <a href="#swap-and-quote-api-requests" id="swap-and-quote-api-requests"></a>

#### Try it out\![​](https://station.jup.ag/docs/apis/swap-api#try-it-out) <a href="#try-it-out" id="try-it-out"></a>

<details>

<summary>Click to play video</summary>



</details>

<details>

<summary>POSThttps://quote-api.jup.ag/v6/swap</summary>

#### Request Parameters[​](https://station.jup.ag/docs/apis/swap-api#request-parameters) <a href="#request-parameters" id="request-parameters"></a>

```
# This is only an example of the parameters. You must replace the placeholder keys before the request can work
curl -X POST "https://quote-api.jup.ag/v6/swap" \
     -H "Content-Type: application/json" \
     -d '{
           "userPublicKey": "YourUserPublicKey",
           "quoteResponse": {
             "inAmount": 1000000,
             "outAmount": 2000000,
             "swapTransaction": "YourSwapTransaction"
           },
           "wrapAndUnwrapSol": true,
           "useSharedAccounts": true,
           "feeAccount": "YourFeeAccountPublicKey",
           "computeUnitPriceMicroLamports": 100,
           "asLegacyTransaction": false,
           "useTokenLedger": false,
           "destinationTokenAccount": "YourDestinationTokenAccountPublicKey"
         }'
```

_**Response**_



</details>

Get the best swap routes for a token trade pair sorted by largest output token amount

<details>

<summary>GEThttps://quote-api.jup.ag/v6/quote</summary>

#### Request Parameters[​](https://station.jup.ag/docs/apis/swap-api#request-parameters-1) <a href="#request-parameters-1" id="request-parameters-1"></a>

PLATFORM FEE

If you'd like to charge a fee, pass in `platformFeeBps` as a parameter in the quote.

AMOUNT

The API takes in amount in integer and you have to factor in the decimals for each token by looking up the decimals for that token. For example, USDC has 6 decimals and 1 USDC is 1000000 in integer when passing it in into the API.

```
# Copy and paste this into your terminal!
curl -s 'https://quote-api.jup.ag/v6/quote?inputMint=So11111111111111111111111111111111111111112&outputMint=EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v&amount=1000000&slippageBps=1' | jq '.outAmount'
```

_**Response**_





</details>

Get the best swap routes for a token trade pair sorted by largest output token amount using the quote api above!

### V6 API Reference[​](https://station.jup.ag/docs/apis/swap-api#v6-api-reference) <a href="#v6-api-reference" id="v6-api-reference"></a>

All Jupiter swaps are using versioned transactions and address lookup tables. But not all wallets support Versioned Transactions yet, so if you detect a wallet that does not support versioned transactions, you will need to use the `asLegacyTransaction` parameter.

Learn more about the Jupiter API Documentation at the [OpenAPI documentation](https://station.jup.ag/api-v6). This documentation has a REST request list and a built in API Playground. Use the API Playground to try API calls now!

API DOCUMENTATION

[OpenAPI Documentation](https://station.jup.ag/api-v6)

#### Guide for V6 Swap API (code example)[​](https://station.jup.ag/docs/apis/swap-api#guide-for-v6-swap-api-code-example) <a href="#guide-for-v6-swap-api-code-example" id="guide-for-v6-swap-api-code-example"></a>

**1. Install required libraries**[**​**](https://station.jup.ag/docs/apis/swap-api#1-install-required-libraries)

Running this example requires a minimum of [NodeJS 16](https://nodejs.org/en/). In your command line terminal, install the libraries.

```
npm i @solana/web3.js
npm i cross-fetch
npm i @project-serum/anchor
npm i bs58
```

**2. Import from libraries and setup connection**[**​**](https://station.jup.ag/docs/apis/swap-api#2-import-from-libraries-and-setup-connection)

Next you can copy the following code snippets to a javascript file jupiter-api-example.js. And when you are ready to run the code, just type: _node jupiter-api-example.js_

```
import { Connection, Keypair, VersionedTransaction } from '@solana/web3.js';
import fetch from 'cross-fetch';
import { Wallet } from '@project-serum/anchor';
import bs58 from 'bs58';

// It is recommended that you use your own RPC endpoint.
// This RPC endpoint is only for demonstration purposes so that this example will run.
const connection = new Connection('https://neat-hidden-sanctuary.solana-mainnet.discover.quiknode.pro/2af5315d336f9ae920028bbb90a73b724dc1bbed/');
```

TIP

Always make sure that you are using your own RPC endpoint. The RPC endpoint used by the connection object in the above example may not work anymore. For more information about RPC endpoints see the [official Solana Documentation](https://solana.com/docs/core/clusters) to learn more about their public RPC endpoints.

**3. Setup your wallet**[**​**](https://station.jup.ag/docs/apis/swap-api#3-setup-your-wallet)

You can paste in your private key for testing purposes but this is not recommended for production applications.

```
const wallet = new Wallet(Keypair.fromSecretKey(bs58.decode(process.env.PRIVATE_KEY || '')));
```

**4. Get the route for a swap**[**​**](https://station.jup.ag/docs/apis/swap-api#4-get-the-route-for-a-swap)

Here, we are getting a quote to swap from SOL to USDC.

```
// Swapping SOL to USDC with input 0.1 SOL and 0.5% slippage
const quoteResponse = await (
  await fetch('https://quote-api.jup.ag/v6/quote?inputMint=So11111111111111111111111111111111111111112\
&outputMint=EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v\
&amount=100000000\
&slippageBps=50'
  )
).json();
// console.log({ quoteResponse })
```

**5. Get the serialized transactions to perform the swap**[**​**](https://station.jup.ag/docs/apis/swap-api#5-get-the-serialized-transactions-to-perform-the-swap)

Once we have the quote, we need to serialize the quote into a swap transaction that can be submitted on chain.

```
// get serialized transactions for the swap
const { swapTransaction } = await (
  await fetch('https://quote-api.jup.ag/v6/swap', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      // quoteResponse from /quote api
      quoteResponse,
      // user public key to be used for the swap
      userPublicKey: wallet.publicKey.toString(),
      // auto wrap and unwrap SOL. default is true
      wrapAndUnwrapSol: true,
      // feeAccount is optional. Use if you want to charge a fee.  feeBps must have been passed in /quote API.
      // feeAccount: "fee_account_public_key"
    })
  })
).json();
```

**6. Deserialize and sign the transaction**[**​**](https://station.jup.ag/docs/apis/swap-api#6-deserialize-and-sign-the-transaction)

```
// deserialize the transaction
const swapTransactionBuf = Buffer.from(swapTransaction, 'base64');
var transaction = VersionedTransaction.deserialize(swapTransactionBuf);
console.log(transaction);

// sign the transaction
transaction.sign([wallet.payer]);
```

**7. Execute the transaction**[**​**](https://station.jup.ag/docs/apis/swap-api#7-execute-the-transaction)

```
// Execute the transaction
const rawTransaction = transaction.serialize()
const txid = await connection.sendRawTransaction(rawTransaction, {
  skipPreflight: true,
  maxRetries: 2
});
await connection.confirmTransaction(txid);
console.log(`https://solscan.io/tx/${txid}`);
```

SOLANA NETWORK CONGESTION

Due to the network congestion on Solana, the `sendRawTransaction` method may not be able to help you to land your transaction. You should check out this [`transactionSender`](https://github.com/jup-ag/jupiter-quote-api-node/blob/main/example/utils/transactionSender.ts) file to send transaction.

<details>

<summary>Whole code snippet</summary>

```
import { Connection, Keypair, VersionedTransaction } from '@solana/web3.js';
import fetch from 'cross-fetch';
import { Wallet } from '@project-serum/anchor';
import bs58 from 'bs58';

// It is recommended that you use your own RPC endpoint.
// This RPC endpoint is only for demonstration purposes so that this example will run.
const connection = new Connection('https://neat-hidden-sanctuary.solana-mainnet.discover.quiknode.pro/2af5315d336f9ae920028bbb90a73b724dc1bbed/');

const wallet = new Wallet(Keypair.fromSecretKey(bs58.decode(process.env.PRIVATE_KEY || '')));

// Swapping SOL to USDC with input 0.1 SOL and 0.5% slippage
const quoteResponse = await (
  await fetch('https://quote-api.jup.ag/v6/quote?inputMint=So11111111111111111111111111111111111111112\
&outputMint=EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v\
&amount=100000000\
&slippageBps=50'
  )
).json();
// console.log({ quoteResponse })

// get serialized transactions for the swap
const { swapTransaction } = await (
  await fetch('https://quote-api.jup.ag/v6/swap', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      // quoteResponse from /quote api
      quoteResponse,
      // user public key to be used for the swap
      userPublicKey: wallet.publicKey.toString(),
      // auto wrap and unwrap SOL. default is true
      wrapAndUnwrapSol: true,
      // feeAccount is optional. Use if you want to charge a fee.  feeBps must have been passed in /quote API.
      // feeAccount: "fee_account_public_key"
    })
  })
).json();

// deserialize the transaction
const swapTransactionBuf = Buffer.from(swapTransaction, 'base64');
var transaction = VersionedTransaction.deserialize(swapTransactionBuf);
console.log(transaction);

// sign the transaction
transaction.sign([wallet.payer]);

// Execute the transaction
const rawTransaction = transaction.serialize()
const txid = await connection.sendRawTransaction(rawTransaction, {
  skipPreflight: true,
  maxRetries: 2
});
await connection.confirmTransaction(txid);
console.log(`https://solscan.io/tx/${txid}`);
```

</details>

### Advanced error handling to disable certain AMM from the API[​](https://station.jup.ag/docs/apis/swap-api#advanced-error-handling-to-disable-certain-amm-from-the-api) <a href="#advanced-error-handling-to-disable-certain-amm-from-the-api" id="advanced-error-handling-to-disable-certain-amm-from-the-api"></a>

Sometimes an AMM will throw an error when swapping. To prevent getting a quote from the failed AMM, you can use the `excludeDexes` parameter when getting `/quote`.

Example JS, with the help of `@mercurial-finance/optimist` package:

```
import { parseErrorForTransaction } from '@mercurial-finance/optimist';

// TX ID from last step if the transaction failed.
const transaction = connection.getTransaction(txid, {
  maxSupportedTransactionVersion: 0,
  commitment: 'confirmed'
});

const programIdToLabelHash = await (
  await fetch('https://quote-api.jup.ag/v6/program-id-to-label')
).json();
const { programIds } = parseErrorForTransaction(transaction);

let excludeDexes = new Set();
if (programIds) {
  for (let programId of programIds) {
    let foundLabel = programIdToLabelHash[programId];
    if(foundLabel) {
      excludeDexes.add(foundLabel);
    }
  }
}

// Request another quote with `excludeDexes`.
const { data } = await (
  await fetch(`https://quote-api.jup.ag/v6/quote?inputMint=So11111111111111111111111111111111111111112
&outputMint=EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v
&amount=100000000&excludeDexes=${Array.from(excludeDexes).join(',')}
&slippageBps=50`
  )
).json();
```

### Instructions Instead of Transaction[​](https://station.jup.ag/docs/apis/swap-api#instructions-instead-of-transaction) <a href="#instructions-instead-of-transaction" id="instructions-instead-of-transaction"></a>

Sometimes you may prefer to compose using instructions instead of one transaction that is returned from the `/swap` endpoint. You can post to `/swap-instructions` instead, it takes the same parameters as the `/swap` endpoint.

```
const instructions = await (
  await fetch('https://quote-api.jup.ag/v6/swap-instructions', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      // quoteResponse from /quote api
      quoteResponse,
      userPublicKey: swapUserKeypair.publicKey.toBase58(),
    })
  })
).json();

if (instructions.error) {
  throw new Error("Failed to get swap instructions: " + instructions.error);
}

const {
  tokenLedgerInstruction, // If you are using `useTokenLedger = true`.
  computeBudgetInstructions, // The necessary instructions to setup the compute budget.
  setupInstructions, // Setup missing ATA for the users.
  swapInstruction: swapInstructionPayload, // The actual swap instruction.
  cleanupInstruction, // Unwrap the SOL if `wrapAndUnwrapSol = true`.
  addressLookupTableAddresses, // The lookup table addresses that you can use if you are using versioned transaction.
} = instructions;

const deserializeInstruction = (instruction) => {
  return new TransactionInstruction({
    programId: new PublicKey(instruction.programId),
    keys: instruction.accounts.map((key) => ({
      pubkey: new PublicKey(key.pubkey),
      isSigner: key.isSigner,
      isWritable: key.isWritable,
    })),
    data: Buffer.from(instruction.data, "base64"),
  });
};

const getAddressLookupTableAccounts = async (
  keys: string[]
): Promise<AddressLookupTableAccount[]> => {
  const addressLookupTableAccountInfos =
    await connection.getMultipleAccountsInfo(
      keys.map((key) => new PublicKey(key))
    );

  return addressLookupTableAccountInfos.reduce((acc, accountInfo, index) => {
    const addressLookupTableAddress = keys[index];
    if (accountInfo) {
      const addressLookupTableAccount = new AddressLookupTableAccount({
        key: new PublicKey(addressLookupTableAddress),
        state: AddressLookupTableAccount.deserialize(accountInfo.data),
      });
      acc.push(addressLookupTableAccount);
    }

    return acc;
  }, new Array<AddressLookupTableAccount>());
};

const addressLookupTableAccounts: AddressLookupTableAccount[] = [];

addressLookupTableAccounts.push(
  ...(await getAddressLookupTableAccounts(addressLookupTableAddresses))
);

const blockhash = (await connection.getLatestBlockhash()).blockhash;
const messageV0 = new TransactionMessage({
  payerKey: payerPublicKey,
  recentBlockhash: blockhash,
  instructions: [
    // uncomment if needed: ...setupInstructions.map(deserializeInstruction),
    deserializeInstruction(swapInstructionPayload),
    // uncomment if needed: deserializeInstruction(cleanupInstruction),
  ],
}).compileToV0Message(addressLookupTableAccounts);
const transaction = new VersionedTransaction(messageV0);
```

### Using `maxAccounts`[​](https://station.jup.ag/docs/apis/swap-api#using-maxaccounts) <a href="#using-maxaccounts" id="using-maxaccounts"></a>

Sometimes, if you are composing with Jupiter Swap instruction, you may want to spare some accounts (64 max in 1 Solana transaction) for your own program instruction, you can use `maxAccounts`.

```
// If you know that your instruction will take up 10 accounts, you
// can pass in 54 as `maxAccounts` when quoting.
const { data } = await (
  await fetch('https://quote-api.jup.ag/v6/quote?inputMint=So11111111111111111111111111111111111111112\
&outputMint=EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v\
&amount=100000000\
&slippageBps=50\
&maxAccounts=54'
  )
).json();
const quoteResponse = data;
// console.log(quoteResponse)
```

The `maxAccounts` is an estimation since it doesn't consider account overlapping but it is a good start to control how many accounts you want per transaction.

### Using Token Ledger Instruction[​](https://station.jup.ag/docs/apis/swap-api#using-token-ledger-instruction) <a href="#using-token-ledger-instruction" id="using-token-ledger-instruction"></a>

Sometimes you may not know the exact input amount for the Jupiter swap until an instruction before the swap happens.

For example:

```
const instructions = await (
  await fetch('https://quote-api.jup.ag/v6/swap-instructions', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      // quoteResponse from /quote api
      quoteResponse,
      useTokenLedger: true,
  })
).json();

const {
  tokenLedgerInstruction: tokenLedgerPayload, // If you are using `useTokenLedger = true`.
  swapInstruction: swapInstructionPayload, // The actual swap instruction.
  addressLookupTableAddresses, // The lookup table addresses that you can use if you are using versioned transaction.
} = instructions;

// A withdraw instruction that will increase the user input token account amount.
const withdrawInstruction = ...;

// Coupled with the tokenLedgerInstruction, the swap instruction will use the
// user increased amount of the input token account after the withdrawal as input amount.
const tokenLedgerInstruction = new TransactionInstruction({
  programId: new PublicKey(tokenLedgerPayload.programId),
  keys: tokenLedgerPayload.accounts.map((key) => ({
    pubkey: new PublicKey(key.pubkey),
      isSigner: key.isSigner,
      isWritable: key.isWritable,
    })),
  data: Buffer.from(tokenLedgerPayload.data, "base64"),
});

const swapInstruction = new TransactionInstruction({
  programId: new PublicKey(swapInstructionPayload.programId),
  keys: swapInstructionPayload.accounts.map((key) => ({
    pubkey: new PublicKey(key.pubkey),
      isSigner: key.isSigner,
      isWritable: key.isWritable,
    })),
  data: Buffer.from(swapInstructionPayload.data, "base64"),
});

const getAdressLookupTableAccounts = async (
  keys: string[]
): Promise<AddressLookupTableAccount[]> => {
  const addressLookupTableAccountInfos =
    await connection.getMultipleAccountsInfo(
      keys.map((key) => new PublicKey(key))
    );

  return addressLookupTableAccountInfos.reduce((acc, accountInfo, index) => {
    const addressLookupTableAddress = keys[index];
    if (accountInfo) {
      const addressLookupTableAccount = new AddressLookupTableAccount({
        key: new PublicKey(addressLookupTableAddress),
        state: AddressLookupTableAccount.deserialize(accountInfo.data),
      });
      acc.push(addressLookupTableAccount);
    }

    return acc;
  }, new Array<AddressLookupTableAccount>());
};

const addressLookupTableAccounts: AddressLookupTableAccount[] = [];

addressLookupTableAccounts.push(
  ...(await getAdressLookupTableAccounts(addressLookupTableAddresses))
);

const messageV0 = new TransactionMessage({
  payerKey: payerPublicKey,
  recentBlockhash: blockhash,
  instructions: [tokenLedgerInstruction, withdrawInstruction, swapInstruction],
}).compileToV0Message(addressLookupTableAccounts);
const transaction = new VersionedTransaction(messageV0);
```

This can be useful if you want to withdraw from Solend and immediately convert your withdrawal token into another token with Jupiter.

### Setting Priority Fee for Your Transaction[​](https://station.jup.ag/docs/apis/swap-api#setting-priority-fee-for-your-transaction) <a href="#setting-priority-fee-for-your-transaction" id="setting-priority-fee-for-your-transaction"></a>

If transactions are expiring without confirmation on-chain, this might mean that you have to pay additional fees to prioritize your transaction. To do so, you can set the `computeUnitPriceMicroLamports` parameter.

```
const transaction = await (
  await fetch('https://quote-api.jup.ag/v6/swap', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      // quoteResponse from /quote api
      quoteResponse,
      // user public key to be used for the swap
      userPublicKey: wallet.publicKey.toString(),
      dynamicComputeUnitLimit: true, // allow dynamic compute limit instead of max 1,400,000
      // custom priority fee
      prioritizationFeeLamports: 'auto' // or custom lamports: 1000
    })
  })
).json();
```

If 'auto' is used, Jupiter will automatically set a priority fee for the transaction, it will be capped at 5,000,000 lamports / 0.005 SOL.

### Examples[​](https://station.jup.ag/docs/apis/swap-api#examples) <a href="#examples" id="examples"></a>

For more example scripts please visit the [jupiter-quote-api-node public Git repository](https://github.com/jup-ag/jupiter-quote-api-node). The repository has some further scripts and instructions for you to explore!

* Javascript/Typescript: [https://github.com/jup-ag/jupiter-quote-api-node](https://github.com/jup-ag/jupiter-quote-api-node)
* Rust: [https://github.com/jup-ag/jupiter-api-rust-example](https://github.com/jup-ag/jupiter-api-rust-example)

Having issues? Head to the [Troubleshooting](https://station.jup.ag/docs/apis/troubleshooting) section for some help.
