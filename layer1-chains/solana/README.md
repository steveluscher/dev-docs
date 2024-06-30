# Solana

## Network

## [Solana RPC Methods & Documentation](https://solana.com/docs/rpc)

Interact with Solana nodes directly with the JSON RPC API via the HTTP and Websocket methods.

### Configuring State Commitment [#](https://solana.com/docs/rpc#configuring-state-commitment) <a href="#configuring-state-commitment" id="configuring-state-commitment"></a>

For preflight checks and transaction processing, Solana nodes choose which bank state to query based on a commitment requirement set by the client. The commitment describes how finalized a block is at that point in time. When querying the ledger state, it's recommended to use lower levels of commitment to report progress and higher levels to ensure the state will not be rolled back.

In descending order of commitment (most finalized to least finalized), clients may specify:

* `finalized` - the node will query the most recent block confirmed by supermajority of the cluster as having reached maximum lockout, meaning the cluster has recognized this block as finalized
* `confirmed` - the node will query the most recent block that has been voted on by supermajority of the cluster.
  * It incorporates votes from gossip and replay.
  * It does not count votes on descendants of a block, only direct votes on that block.
  * This confirmation level also upholds "optimistic confirmation" guarantees in release 1.3 and onwards.
* `processed` - the node will query its most recent block. Note that the block may still be skipped by the cluster.

For processing many dependent transactions in series, it's recommended to use `confirmed` commitment, which balances speed with rollback safety. For total safety, it's recommended to use `finalized` commitment.

#### Default Commitment [#](https://solana.com/docs/rpc#default-commitment) <a href="#default-commitment" id="default-commitment"></a>

If commitment configuration is not provided, the node will [default to `finalized` commitment](https://github.com/anza-xyz/agave/blob/aa0922d6845e119ba466f88497e8209d1c82febc/sdk/src/commitment\_config.rs#L199-L203)

Only methods that query bank state accept the commitment parameter. They are indicated in the API Reference below.

### RpcResponse Structure [#](https://solana.com/docs/rpc#rpcresponse-structure) <a href="#rpcresponse-structure" id="rpcresponse-structure"></a>

Many methods that take a commitment parameter return an RpcResponse JSON object comprised of two parts:

* `context` : An RpcResponseContext JSON structure including a `slot` field at which the operation was evaluated.
* `value` : The value returned by the operation itself.

### Parsed Responses [#](https://solana.com/docs/rpc#parsed-responses) <a href="#parsed-responses" id="parsed-responses"></a>

Some methods support an `encoding` parameter, and can return account or instruction data in parsed JSON format if `"encoding":"jsonParsed"` is requested and the node has a parser for the owning program. Solana nodes currently support JSON parsing for the following native and SPL programs:

| Program                      | Account State | Instructions |
| ---------------------------- | ------------- | ------------ |
| Address Lookup               | v1.15.0       | v1.15.0      |
| BPF Loader                   | n/a           | stable       |
| BPF Upgradeable Loader       | stable        | stable       |
| Config                       | stable        |              |
| SPL Associated Token Account | n/a           | stable       |
| SPL Memo                     | n/a           | stable       |
| SPL Token                    | stable        | stable       |
| SPL Token 2022               | stable        | stable       |
| Stake                        | stable        | stable       |
| Vote                         | stable        | stable       |

The list of account parsers can be found [here](https://github.com/solana-labs/solana/blob/master/account-decoder/src/parse\_account\_data.rs), and instruction parsers [here](https://github.com/solana-labs/solana/blob/master/transaction-status/src/parse\_instruction.rs).

### Filter criteria [#](https://solana.com/docs/rpc#filter-criteria) <a href="#filter-criteria" id="filter-criteria"></a>

Some methods support providing a `filters` object to enable pre-filtering the data returned within the RpcResponse JSON object. The following filters exist:

* `memcmp: object` - compares a provided series of bytes with program account data at a particular offset. Fields:
  * `offset: usize` - offset into program account data to start comparison
  * `bytes: string` - data to match, as encoded string
  * `encoding: string` - encoding for filter `bytes` data, either "base58" or "base64". Data is limited in size to 128 or fewer decoded bytes.\
    **NEW: This field, and base64 support generally, is only available in solana-core v1.14.0 or newer. Please omit when querying nodes on earlier versions**
* `dataSize: u64` - compares the program account data length with the provided data size

### [Common JSON Data Structures for Solana RPC Methods](https://solana.com/docs/rpc/json-structures)

Various Solana RPC methods will return more complex responses as structured JSON objects, filled with specific keyed values.

The most common of these JSON data structures include:

* [transactions](https://solana.com/docs/rpc/json-structures#transactions)
* [inner instructions](https://solana.com/docs/rpc/json-structures#inner-instructions)
* [token balances](https://solana.com/docs/rpc/json-structures#token-balances)

#### Transactions [#](https://solana.com/docs/rpc/json-structures#transactions) <a href="#transactions" id="transactions"></a>

Transactions are quite different from those on other blockchains. Be sure to review [Anatomy of a Transaction](https://solana.com/docs/core/transactions) to learn about transactions on Solana.

The JSON structure of a transaction is defined as follows:

* `signatures: <array[string]>` - A list of base-58 encoded signatures applied to the transaction. The list is always of length `message.header.numRequiredSignatures` and not empty. The signature at index `i` corresponds to the public key at index `i` in `message.accountKeys`. The first one is used as the [transaction id](https://solana.com/docs/terminology#transaction-id).
* `message: <object>` - Defines the content of the transaction.
  * `accountKeys: <array[string]>` - List of base-58 encoded public keys used by the transaction, including by the instructions and for signatures. The first `message.header.numRequiredSignatures` public keys must sign the transaction.
  * `header: <object>` - Details the account types and signatures required by the transaction.
    * `numRequiredSignatures: <number>` - The total number of signatures required to make the transaction valid. The signatures must match the first `numRequiredSignatures` of `message.accountKeys`.
    * `numReadonlySignedAccounts: <number>` - The last `numReadonlySignedAccounts` of the signed keys are read-only accounts. Programs may process multiple transactions that load read-only accounts within a single PoH entry, but are not permitted to credit or debit lamports or modify account data. Transactions targeting the same read-write account are evaluated sequentially.
    * `numReadonlyUnsignedAccounts: <number>` - The last `numReadonlyUnsignedAccounts` of the unsigned keys are read-only accounts.
  * `recentBlockhash: <string>` - A base-58 encoded hash of a recent block in the ledger used to prevent transaction duplication and to give transactions lifetimes.
  * `instructions: <array[object]>` - List of program instructions that will be executed in sequence and committed in one atomic transaction if all succeed.
    * `programIdIndex: <number>` - Index into the `message.accountKeys` array indicating the program account that executes this instruction.
    * `accounts: <array[number]>` - List of ordered indices into the `message.accountKeys` array indicating which accounts to pass to the program.
    * `data: <string>` - The program input data encoded in a base-58 string.
  * `addressTableLookups: <array[object]|undefined>` - List of address table lookups used by a transaction to dynamically load addresses from on-chain address lookup tables. Undefined if `maxSupportedTransactionVersion` is not set.
    * `accountKey: <string>` - base-58 encoded public key for an address lookup table account.
    * `writableIndexes: <array[number]>` - List of indices used to load addresses of writable accounts from a lookup table.
    * `readonlyIndexes: <array[number]>` - List of indices used to load addresses of readonly accounts from a lookup table.

#### Inner Instructions [#](https://solana.com/docs/rpc/json-structures#inner-instructions) <a href="#inner-instructions" id="inner-instructions"></a>

The Solana runtime records the cross-program instructions that are invoked during transaction processing and makes these available for greater transparency of what was executed on-chain per transaction instruction. Invoked instructions are grouped by the originating transaction instruction and are listed in order of processing.

The JSON structure of inner instructions is defined as a list of objects in the following structure:

* `index: number` - Index of the transaction instruction from which the inner instruction(s) originated
* `instructions: <array[object]>` - Ordered list of inner program instructions that were invoked during a single transaction instruction.
  * `programIdIndex: <number>` - Index into the `message.accountKeys` array indicating the program account that executes this instruction.
  * `accounts: <array[number]>` - List of ordered indices into the `message.accountKeys` array indicating which accounts to pass to the program.
  * `data: <string>` - The program input data encoded in a base-58 string.

#### Token Balances [#](https://solana.com/docs/rpc/json-structures#token-balances) <a href="#token-balances" id="token-balances"></a>

The JSON structure of token balances is defined as a list of objects in the following structure:

* `accountIndex: <number>` - Index of the account in which the token balance is provided for.
* `mint: <string>` - Pubkey of the token's mint.
* `owner: <string|undefined>` - Pubkey of token balance's owner.
* `programId: <string|undefined>` - Pubkey of the Token program that owns the account.
* `uiTokenAmount: <object>` -
  * `amount: <string>` - Raw amount of tokens as a string, ignoring decimals.
  * `decimals: <number>` - Number of decimals configured for token's mint.
  * `uiAmount: <number|null>` - Token amount as a float, accounting for decimals.

### [Solana RPC HTTP Methods](https://solana.com/docs/rpc/http)

Solana nodes accept HTTP requests using the [JSON-RPC 2.0](https://www.jsonrpc.org/specification) specification.

INFO

For JavaScript applications, use the [@solana/web3.js](https://github.com/solana-labs/solana-web3.js) library as a convenient interface for the RPC methods to interact with a Solana node. For an PubSub connection to a Solana node, use the [Websocket API](https://solana.com/docs/rpc/websocket).

#### RPC HTTP Endpoint [#](https://solana.com/docs/rpc/http#rpc-http-endpoint) <a href="#rpc-http-endpoint" id="rpc-http-endpoint"></a>

Default port: `8899`

* [http://localhost:8899](http://localhost:8899/)
* [http://192.168.1.88:8899](http://192.168.1.88:8899/)

#### Request Formatting [#](https://solana.com/docs/rpc/http#request-formatting) <a href="#request-formatting" id="request-formatting"></a>

To make a JSON-RPC request, send an HTTP POST request with a `Content-Type: application/json` header. The JSON request data should contain 4 fields:

* `jsonrpc: <string>` - set to `"2.0"`
* `id: <number>` - a unique client-generated identifying integer
* `method: <string>` - a string containing the method to be invoked
* `params: <array>` - a JSON array of ordered parameter values

Example using curl:

```
curl https://api.devnet.solana.com -X POST -H "Content-Type: application/json" -d '  {    "jsonrpc": "2.0",    "id": 1,    "method": "getBalance",    "params": [      "83astBRguLMdt2h5U1Tpdq5tjFoJ6noeGwaY3mDLVcri"    ]  }'
```

The response output will be a JSON object with the following fields:

* `jsonrpc: <string>` - matching the request specification
* `id: <number>` - matching the request identifier
* `result: <array|number|object|string>` - requested data or success confirmation

Requests can be sent in batches by sending an array of JSON-RPC request objects as the data for a single POST.

Example Request [#](https://solana.com/docs/rpc/http#example-request)

The commitment parameter should be included as the last element in the `params` array:

```
curl https://api.devnet.solana.com -X POST -H "Content-Type: application/json" -d '  {    "jsonrpc": "2.0",    "id": 1,    "method": "getBalance",    "params": [      "83astBRguLMdt2h5U1Tpdq5tjFoJ6noeGwaY3mDLVcri",      {        "commitment": "finalized"      }    ]  }'
```

#### Definitions [#](https://solana.com/docs/rpc/http#definitions) <a href="#definitions" id="definitions"></a>

* Hash: A SHA-256 hash of a chunk of data.
* Pubkey: The public key of a Ed25519 key-pair.
* Transaction: A list of Solana instructions signed by a client keypair to authorize those actions.
* Signature: An Ed25519 signature of transaction's payload data including instructions. This can be used to identify transactions.

#### Health Check [#](https://solana.com/docs/rpc/http#health-check) <a href="#health-check" id="health-check"></a>

Although not a JSON RPC API, a `GET /health` at the RPC HTTP Endpoint provides a health-check mechanism for use by load balancers or other network infrastructure. This request will always return a HTTP 200 OK response with a body of "ok", "behind" or "unknown":

* `ok`: The node is within `HEALTH_CHECK_SLOT_DISTANCE` slots from the latest cluster confirmed slot
* `behind { distance }`: The node is behind `distance` slots from the latest cluster confirmed slot where `distance > HEALTH_CHECK_SLOT_DISTANCE`
* `unknown`: The node is unable to determine where it stands in relation to the cluster

## [Transaction Confirmation & Expiration](https://solana.com/docs/advanced/confirmation)

Problems relating to [transaction confirmation](https://solana.com/docs/terminology#transaction-confirmations) are common with many newer developers while building applications. This article aims to boost the overall understanding of the confirmation mechanism used on the Solana blockchain, including some recommended best practices.

### Brief background on transactions [#](https://solana.com/docs/advanced/confirmation#brief-background-on-transactions) <a href="#brief-background-on-transactions" id="brief-background-on-transactions"></a>

Before diving into how Solana transaction confirmation and expiration works, let's briefly set the base understanding of a few things:

* what a transaction is
* the lifecycle of a transaction
* what a blockhash is
* and a brief understanding of Proof of History (PoH) and how it relates to blockhashes

#### What is a transaction? [#](https://solana.com/docs/advanced/confirmation#what-is-a-transaction?) <a href="#what-is-a-transaction" id="what-is-a-transaction"></a>

Transactions consist of two components: a [message](https://solana.com/docs/terminology#message) and a [list of signatures](https://solana.com/docs/terminology#signature). The transaction message is where the magic happens and at a high level it consists of three components:

* a **list of instructions** to invoke,
* a **list of accounts** to load, and
* a **“recent blockhash.”**

In this article, we’re going to be focusing a lot on a transaction’s [recent blockhash](https://solana.com/docs/terminology#blockhash) because it plays a big role in transaction confirmation.

#### Transaction lifecycle refresher [#](https://solana.com/docs/advanced/confirmation#transaction-lifecycle-refresher) <a href="#transaction-lifecycle-refresher" id="transaction-lifecycle-refresher"></a>

Below is a high level view of the lifecycle of a transaction. This article will touch on everything except steps 1 and 4.

1. Create a list of instructions along with the list of accounts that instructions need to read and write
2. Fetch a recent blockhash and use it to prepare a transaction message
3. Simulate the transaction to ensure it behaves as expected
4. Prompt user to sign the prepared transaction message with their private key
5. Send the transaction to an RPC node which attempts to forward it to the current block producer
6. Hope that a block producer validates and commits the transaction into their produced block
7. Confirm the transaction has either been included in a block or detect when it has expired

#### What is a Blockhash? [#](https://solana.com/docs/advanced/confirmation#what-is-a-blockhash?) <a href="#what-is-a-blockhash" id="what-is-a-blockhash"></a>

A [“blockhash”](https://solana.com/docs/terminology#blockhash) refers to the last Proof of History (PoH) hash for a [“slot”](https://solana.com/docs/terminology#slot) (description below). Since Solana uses PoH as a trusted clock, a transaction’s recent blockhash can be thought of as a **timestamp**.

#### Proof of History refresher [#](https://solana.com/docs/advanced/confirmation#proof-of-history-refresher) <a href="#proof-of-history-refresher" id="proof-of-history-refresher"></a>

Solana’s Proof of History mechanism uses a very long chain of recursive SHA-256 hashes to build a trusted clock. The “history” part of the name comes from the fact that block producers hash transaction id’s into the stream to record which transactions were processed in their block.

[PoH hash calculation](https://github.com/anza-xyz/agave/blob/aa0922d6845e119ba466f88497e8209d1c82febc/entry/src/poh.rs#L79): `next_hash = hash(prev_hash, hash(transaction_ids))`

PoH can be used as a trusted clock because each hash must be produced sequentially. Each produced block contains a blockhash and a list of hash checkpoints called “ticks” so that validators can verify the full chain of hashes in parallel and prove that some amount of time has actually passed.

### Transaction Expiration [#](https://solana.com/docs/advanced/confirmation#transaction-expiration) <a href="#transaction-expiration" id="transaction-expiration"></a>

By default, all Solana transactions will expire if not committed to a block in a certain amount of time. The **vast majority** of transaction confirmation issues are related to how RPC nodes and validators detect and handle **expired** transactions. A solid understanding of how transaction expiration works should help you diagnose the bulk of your transaction confirmation issues.

#### How does transaction expiration work? [#](https://solana.com/docs/advanced/confirmation#how-does-transaction-expiration-work?) <a href="#how-does-transaction-expiration-work" id="how-does-transaction-expiration-work"></a>

Each transaction includes a “recent blockhash” which is used as a PoH clock timestamp and expires when that blockhash is no longer “recent enough”.

As each block is finalized (i.e. the maximum tick height [is reached](https://github.com/anza-xyz/agave/blob/0588ecc6121ba026c65600d117066dbdfaf63444/runtime/src/bank.rs#L3269-L3271), reaching the "block boundary"), the final hash of the block is added to the `BlockhashQueue` which stores a maximum of the [300 most recent blockhashes](https://github.com/anza-xyz/agave/blob/e0b0bcc80380da34bb63364cc393801af1e1057f/sdk/program/src/clock.rs#L123-L126). During transaction processing, Solana Validators will check if each transaction's recent blockhash is recorded within the most recent 151 stored hashes (aka "max processing age"). If the transaction's recent blockhash is [older than this](https://github.com/anza-xyz/agave/blob/cb2fd2b632f16a43eff0c27af7458e4e97512e31/runtime/src/bank.rs#L3570-L3571) max processing age, the transaction is not processed.

INFO

Due to the current [max processing age of 150](https://github.com/anza-xyz/agave/blob/cb2fd2b632f16a43eff0c27af7458e4e97512e31/sdk/program/src/clock.rs#L129-L131) and the "age" of a blockhash in the queue being [0-indexed](https://github.com/anza-xyz/agave/blob/992a398fe8ea29ec4f04d081ceef7664960206f4/accounts-db/src/blockhash\_queue.rs#L248-L274), there are actually 151 blockhashes that are considered "recent enough" and valid for processing.

Since [slots](https://solana.com/docs/terminology#slot) (aka the time period a validator can produce a block) are configured to last about [400ms](https://github.com/anza-xyz/agave/blob/cb2fd2b632f16a43eff0c27af7458e4e97512e31/sdk/program/src/clock.rs#L107-L109), but may fluctuate between 400ms and 600ms, a given blockhash can only be used by transactions for about 60 to 90 seconds before it will be considered expired by the runtime.

#### Example of transaction expiration [#](https://solana.com/docs/advanced/confirmation#example-of-transaction-expiration) <a href="#example-of-transaction-expiration" id="example-of-transaction-expiration"></a>

Let’s walk through a quick example:

1. A validator is actively producing a new block for the current slot
2. The validator receives a transaction from a user with the recent blockhash `abcd...`
3. The validator checks this blockhash `abcd...` against the list of recent blockhashes in the `BlockhashQueue` and discovers that it was created 151 blocks ago
4. Since it is exactly 151 blockhashes old, the transaction has not expired yet and can still be processed!
5. But wait: before actually processing the transaction, the validator finished creating the next block and added it to the `BlockhashQueue`. The validator then starts producing the block for the next slot (validators get to produce blocks for 4 consecutive slots)
6. The validator checks that same transaction again and finds it is now 152 blockhashes old and rejects it because it’s too old :(

### Why do transactions expire? [#](https://solana.com/docs/advanced/confirmation#why-do-transactions-expire?) <a href="#why-do-transactions-expire" id="why-do-transactions-expire"></a>

There’s a very good reason for this actually, it’s to help validators avoid processing the same transaction twice.

A naive brute force approach to prevent double processing could be to check every new transaction against the blockchain’s entire transaction history. But by having transactions expire after a short amount of time, validators only need to check if a new transaction is in a relatively small set of _recently_ processed transactions.

#### Other blockchains [#](https://solana.com/docs/advanced/confirmation#other-blockchains) <a href="#other-blockchains" id="other-blockchains"></a>

Solana’s approach of prevent double processing is quite different from other blockchains. For example, Ethereum tracks a counter (nonce) for each transaction sender and will only process transactions that use the next valid nonce.

Ethereum’s approach is simple for validators to implement, but it can be problematic for users. Many people have encountered situations when their Ethereum transactions got stuck in a _pending_ state for a long time and all the later transactions, which used higher nonce values, were blocked from processing.

#### Advantages on Solana [#](https://solana.com/docs/advanced/confirmation#advantages-on-solana) <a href="#advantages-on-solana" id="advantages-on-solana"></a>

There are a few advantages to Solana’s approach:

1. A single fee payer can submit multiple transactions at the same time that are allowed to be processed in any order. This might happen if you’re using multiple applications at the same time.
2. If a transaction doesn’t get committed to a block and expires, users can try again knowing that their previous transaction will NOT ever be processed.

By not using counters, the Solana wallet experience may be easier for users to understand because they can get to success, failure, or expiration states quickly and avoid annoying pending states.

#### Disadvantages on Solana [#](https://solana.com/docs/advanced/confirmation#disadvantages-on-solana) <a href="#disadvantages-on-solana" id="disadvantages-on-solana"></a>

Of course there are some disadvantages too:

1. Validators have to actively track a set of all processed transaction id’s to prevent double processing.
2. If the expiration time period is too short, users might not be able to submit their transaction before it expires.

These disadvantages highlight a tradeoff in how transaction expiration is configured. If the expiration time of a transaction is increased, validators need to use more memory to track more transactions. If expiration time is decreased, users don’t have enough time to submit their transaction.

Currently, Solana clusters require that transactions use blockhashes that are no more than 151 blocks old.

INFO

This [Github issue](https://github.com/solana-labs/solana/issues/23582) contains some calculations that estimate that mainnet-beta validators need about 150MB of memory to track transactions. This could be slimmed down in the future if necessary without decreasing expiration time as are detailed in that issue.

### Transaction confirmation tips [#](https://solana.com/docs/advanced/confirmation#transaction-confirmation-tips) <a href="#transaction-confirmation-tips" id="transaction-confirmation-tips"></a>

As mentioned before, blockhashes expire after a time period of only 151 blocks which can pass as quickly as **one minute** when slots are processed within the target time of 400ms.

One minute is not a lot of time considering that a client needs to fetch a recent blockhash, wait for the user to sign, and finally hope that the broadcasted transaction reaches a leader that is willing to accept it. Let’s go through some tips to help avoid confirmation failures due to transaction expiration!

#### Fetch blockhashes with the appropriate commitment level [#](https://solana.com/docs/advanced/confirmation#fetch-blockhashes-with-the-appropriate-commitment-level) <a href="#fetch-blockhashes-with-the-appropriate-commitment-level" id="fetch-blockhashes-with-the-appropriate-commitment-level"></a>

Given the short expiration time frame, it’s imperative that clients and applications help users create transactions with a blockhash that is as recent as possible.

When fetching blockhashes, the current recommended RPC API is called [`getLatestBlockhash`](https://solana.com/docs/rpc/http/getlatestblockhash). By default, this API uses the `finalized` commitment level to return the most recently finalized block’s blockhash. However, you can override this behavior by [setting the `commitment` parameter](https://solana.com/docs/rpc#configuring-state-commitment) to a different commitment level.

**Recommendation**

The `confirmed` commitment level should almost always be used for RPC requests because it’s usually only a few slots behind the `processed` commitment and has a very low chance of belonging to a dropped [fork](https://docs.solanalabs.com/consensus/fork-generation).

But feel free to consider the other options:

* Choosing `processed` will let you fetch the most recent blockhash compared to other commitment levels and therefore gives you the most time to prepare and process a transaction. But due to the prevalence of forking in the Solana blockchain, roughly 5% of blocks don’t end up being finalized by the cluster so there’s a real chance that your transaction uses a blockhash that belongs to a dropped fork. Transactions that use blockhashes for abandoned blocks won’t ever be considered recent by any blocks that are in the finalized blockchain.
* Using the [default commitment](https://solana.com/docs/rpc#default-commitment) level `finalized` will eliminate any risk that the blockhash you choose will belong to a dropped fork. The tradeoff is that there is typically at least a 32 slot difference between the most recent confirmed block and the most recent finalized block. This tradeoff is pretty severe and effectively reduces the expiration of your transactions by about 13 seconds but this could be even more during unstable cluster conditions.

#### Use an appropriate preflight commitment level [#](https://solana.com/docs/advanced/confirmation#use-an-appropriate-preflight-commitment-level) <a href="#use-an-appropriate-preflight-commitment-level" id="use-an-appropriate-preflight-commitment-level"></a>

If your transaction uses a blockhash that was fetched from one RPC node then you send, or simulate, that transaction with a different RPC node, you could run into issues due to one node lagging behind the other.

When RPC nodes receive a `sendTransaction` request, they will attempt to determine the expiration block of your transaction using the most recent finalized block or with the block selected by the `preflightCommitment` parameter. A **VERY** common issue is that a received transaction’s blockhash was produced after the block used to calculate the expiration for that transaction. If an RPC node can’t determine when your transaction expires, it will only forward your transaction **one time** and afterwards will then **drop** the transaction.

Similarly, when RPC nodes receive a `simulateTransaction` request, they will simulate your transaction using the most recent finalized block or with the block selected by the `preflightCommitment` parameter. If the block chosen for simulation is older than the block used for your transaction’s blockhash, the simulation will fail with the dreaded “blockhash not found” error.

**Recommendation**

Even if you use `skipPreflight`, **ALWAYS** set the `preflightCommitment` parameter to the same commitment level used to fetch your transaction’s blockhash for both `sendTransaction` and `simulateTransaction` requests.

#### Be wary of lagging RPC nodes when sending transactions [#](https://solana.com/docs/advanced/confirmation#be-wary-of-lagging-rpc-nodes-when-sending-transactions) <a href="#be-wary-of-lagging-rpc-nodes-when-sending-transactions" id="be-wary-of-lagging-rpc-nodes-when-sending-transactions"></a>

When your application uses an RPC pool service or when the RPC endpoint differs between creating a transaction and sending a transaction, you need to be wary of situations where one RPC node is lagging behind the other. For example, if you fetch a transaction blockhash from one RPC node then you send that transaction to a second RPC node for forwarding or simulation, the second RPC node might be lagging behind the first.

**Recommendation**

For `sendTransaction` requests, clients should keep resending a transaction to a RPC node on a frequent interval so that if an RPC node is slightly lagging behind the cluster, it will eventually catch up and detect your transaction’s expiration properly.

For `simulateTransaction` requests, clients should use the [`replaceRecentBlockhash`](https://solana.com/docs/rpc/http/simulatetransaction) parameter to tell the RPC node to replace the simulated transaction’s blockhash with a blockhash that will always be valid for simulation.

#### Avoid reusing stale blockhashes [#](https://solana.com/docs/advanced/confirmation#avoid-reusing-stale-blockhashes) <a href="#avoid-reusing-stale-blockhashes" id="avoid-reusing-stale-blockhashes"></a>

Even if your application has fetched a very recent blockhash, be sure that you’re not reusing that blockhash in transactions for too long. The ideal scenario is that a recent blockhash is fetched right before a user signs their transaction.

**Recommendation for applications**

Poll for new recent blockhashes on a frequent basis to ensure that whenever a user triggers an action that creates a transaction, your application already has a fresh blockhash that’s ready to go.

**Recommendation for wallets**

Poll for new recent blockhashes on a frequent basis and replace a transaction’s recent blockhash right before they sign the transaction to ensure the blockhash is as fresh as possible.

#### Use healthy RPC nodes when fetching blockhashes [#](https://solana.com/docs/advanced/confirmation#use-healthy-rpc-nodes-when-fetching-blockhashes) <a href="#use-healthy-rpc-nodes-when-fetching-blockhashes" id="use-healthy-rpc-nodes-when-fetching-blockhashes"></a>

By fetching the latest blockhash with the `confirmed` commitment level from an RPC node, it’s going to respond with the blockhash for the latest confirmed block that it’s aware of. Solana’s block propagation protocol prioritizes sending blocks to staked nodes so RPC nodes naturally lag about a block behind the rest of the cluster. They also have to do more work to handle application requests and can lag a lot more under heavy user traffic.

Lagging RPC nodes can therefore respond to [`getLatestBlockhash`](https://solana.com/docs/rpc/http/getlatestblockhash) requests with blockhashes that were confirmed by the cluster quite awhile ago. By default, a lagging RPC node detects that it is more than 150 slots behind the cluster will stop responding to requests, but just before hitting that threshold they can still return a blockhash that is just about to expire.

**Recommendation**

Monitor the health of your RPC nodes to ensure that they have an up-to-date view of the cluster state with one of the following methods:

1. Fetch your RPC node’s highest processed slot by using the [`getSlot`](https://solana.com/docs/rpc/http/getslot) RPC API with the `processed` commitment level and then call the [\`getMaxShredInsertSlot](https://solana.com/docs/rpc/http/getmaxshredinsertslot) RPC API to get the highest slot that your RPC node has received a “shred” of a block for. If the difference between these responses is very large, the cluster is producing blocks far ahead of what the RPC node has processed.
2. Call the `getLatestBlockhash` RPC API with the `confirmed` commitment level on a few different RPC API nodes and use the blockhash from the node that returns the highest slot for its [context slot](https://solana.com/docs/rpc#rpcresponse-structure).

#### Wait long enough for expiration [#](https://solana.com/docs/advanced/confirmation#wait-long-enough-for-expiration) <a href="#wait-long-enough-for-expiration" id="wait-long-enough-for-expiration"></a>

**Recommendation**

When calling the [`getLatestBlockhash`](https://solana.com/docs/rpc/http/getlatestblockhash) RPC API to get a recent blockhash for your transaction, take note of the `lastValidBlockHeight` in the response.

Then, poll the [`getBlockHeight`](https://solana.com/docs/rpc/http/getblockheight) RPC API with the `confirmed` commitment level until it returns a block height greater than the previously returned last valid block height.

#### Consider using “durable” transactions [#](https://solana.com/docs/advanced/confirmation#consider-using-%E2%80%9Cdurable%E2%80%9D-transactions) <a href="#consider-using-durable-transactions" id="consider-using-durable-transactions"></a>

Sometimes transaction expiration issues are really hard to avoid (e.g. offline signing, cluster instability). If the previous tips are still not sufficient for your use-case, you can switch to using durable transactions (they just require a bit of setup).

To start using durable transactions, a user first needs to submit a transaction that [invokes instructions that create a special on-chain “nonce” account](https://docs.rs/solana-program/latest/solana\_program/system\_instruction/fn.create\_nonce\_account.html) and stores a “durable blockhash” inside of it. At any point in the future (as long as the nonce account hasn’t been used yet), the user can create a durable transaction by following these 2 rules:

1. The instruction list must start with an [“advance nonce” system instruction](https://docs.rs/solana-program/latest/solana\_program/system\_instruction/fn.advance\_nonce\_account.html) which loads their on-chain nonce account
2. The transaction’s blockhash must be equal to the durable blockhash stored by the on-chain nonce account

Here’s how these durable transactions are processed by the Solana runtime:

1. If the transaction’s blockhash is no longer “recent”, the runtime checks if the transaction’s instruction list begins with an “advance nonce” system instruction
2. If so, it then loads the nonce account specified by the “advance nonce” instruction
3. Then it checks that the stored durable blockhash matches the transaction’s blockhash
4. Lastly it makes sure to advance the nonce account’s stored blockhash to the latest recent blockhash to ensure that the same transaction can never be processed again

For more details about how these durable transactions work, you can read the [original proposal](https://docs.solanalabs.com/implemented-proposals/durable-tx-nonces) and [check out an example](https://solana.com/developers/guides/advanced/introduction-to-durable-nonces) in the Solana docs.

## [Retrying Transactions](https://solana.com/docs/advanced/retry)

### Retrying Transactions [#](https://solana.com/docs/advanced/retry#retrying-transactions) <a href="#retrying-transactions" id="retrying-transactions"></a>

On some occasions, a seemingly valid transaction may be dropped before it is included in a block. This most often occurs during periods of network congestion, when an RPC node fails to rebroadcast the transaction to the [leader](https://solana.com/docs/terminology#leader). To an end-user, it may appear as if their transaction disappears entirely. While RPC nodes are equipped with a generic rebroadcasting algorithm, application developers are also capable of developing their own custom rebroadcasting logic.

### TLDR; [#](https://solana.com/docs/advanced/retry#tldr;) <a href="#tldr" id="tldr"></a>

* RPC nodes will attempt to rebroadcast transactions using a generic algorithm
* Application developers can implement their own custom rebroadcasting logic
* Developers should take advantage of the `maxRetries` parameter on the `sendTransaction` JSON-RPC method
* Developers should enable preflight checks to raise errors before transactions are submitted
* Before re-signing any transaction, it is **very important** to ensure that the initial transaction’s blockhash has expired

### The Journey of a Transaction [#](https://solana.com/docs/advanced/retry#the-journey-of-a-transaction) <a href="#the-journey-of-a-transaction" id="the-journey-of-a-transaction"></a>

#### How Clients Submit Transactions [#](https://solana.com/docs/advanced/retry#how-clients-submit-transactions) <a href="#how-clients-submit-transactions" id="how-clients-submit-transactions"></a>

In Solana, there is no concept of a mempool. All transactions, whether they are initiated programmatically or by an end-user, are efficiently routed to leaders so that they can be processed into a block. There are two main ways in which a transaction can be sent to leaders:

1. By proxy via an RPC server and the [sendTransaction](https://solana.com/docs/rpc/http/sendtransaction) JSON-RPC method
2. Directly to leaders via a [TPU Client](https://docs.rs/solana-client/latest/solana\_client/tpu\_client/index.html)

The vast majority of end-users will submit transactions via an RPC server. When a client submits a transaction, the receiving RPC node will in turn attempt to broadcast the transaction to both the current and next leaders. Until the transaction is processed by a leader, there is no record of the transaction outside of what the client and the relaying RPC nodes are aware of. In the case of a TPU client, rebroadcast and leader forwarding is handled entirely by the client software.

![Overview of a transactions journey, from client to leader](https://solana-developer-content.vercel.app/assets/docs/rt-tx-journey.png)Overview of a transactions journey, from client to leader

#### How RPC Nodes Broadcast Transactions [#](https://solana.com/docs/advanced/retry#how-rpc-nodes-broadcast-transactions) <a href="#how-rpc-nodes-broadcast-transactions" id="how-rpc-nodes-broadcast-transactions"></a>

After an RPC node receives a transaction via `sendTransaction`, it will convert the transaction into a [UDP](https://en.wikipedia.org/wiki/User\_Datagram\_Protocol) packet before forwarding it to the relevant leaders. UDP allows validators to quickly communicate with one another, but does not provide any guarantees regarding transaction delivery.

Because Solana’s leader schedule is known in advance of every [epoch](https://solana.com/docs/terminology#epoch) (\~2 days), an RPC node will broadcast its transaction directly to the current and next leaders. This is in contrast to other gossip protocols such as Ethereum that propagate transactions randomly and broadly across the entire network. By default, RPC nodes will try to forward transactions to leaders every two seconds until either the transaction is finalized or the transaction’s blockhash expires (150 blocks or \~1 minute 19 seconds as of the time of this writing). If the outstanding rebroadcast queue size is greater than [10,000 transactions](https://github.com/solana-labs/solana/blob/bfbbc53dac93b3a5c6be9b4b65f679fdb13e41d9/send-transaction-service/src/send\_transaction\_service.rs#L20), newly submitted transactions are dropped. There are command-line [arguments](https://github.com/solana-labs/solana/blob/bfbbc53dac93b3a5c6be9b4b65f679fdb13e41d9/validator/src/main.rs#L1172) that RPC operators can adjust to change the default behavior of this retry logic.

When an RPC node broadcasts a transaction, it will attempt to forward the transaction to a leader’s [Transaction Processing Unit (TPU)](https://github.com/solana-labs/solana/blob/cd6f931223181d5a1d47cba64e857785a175a760/core/src/validator.rs#L867). The TPU processes transactions in five distinct phases:

* [Fetch Stage](https://github.com/solana-labs/solana/blob/cd6f931223181d5a1d47cba64e857785a175a760/core/src/fetch\_stage.rs#L21)
* [SigVerify Stage](https://github.com/solana-labs/solana/blob/cd6f931223181d5a1d47cba64e857785a175a760/core/src/tpu.rs#L91)
* [Banking Stage](https://github.com/solana-labs/solana/blob/cd6f931223181d5a1d47cba64e857785a175a760/core/src/banking\_stage.rs#L249)
* [Proof of History Service](https://github.com/solana-labs/solana/blob/cd6f931223181d5a1d47cba64e857785a175a760/poh/src/poh\_service.rs)
* [Broadcast Stage](https://github.com/solana-labs/solana/blob/cd6f931223181d5a1d47cba64e857785a175a760/core/src/tpu.rs#L136)

![Overview of the Transaction Processing Unit (TPU)](https://solana-developer-content.vercel.app/assets/docs/rt-tpu-jito-labs.png)Overview of the Transaction Processing Unit (TPU)

Of these five phases, the Fetch Stage is responsible for receiving transactions. Within the Fetch Stage, validators will categorize incoming transactions according to three ports:

* [tpu](https://github.com/solana-labs/solana/blob/cd6f931223181d5a1d47cba64e857785a175a760/gossip/src/contact\_info.rs#L27) handles regular transactions such as token transfers, NFT mints, and program instructions
* [tpu\_vote](https://github.com/solana-labs/solana/blob/cd6f931223181d5a1d47cba64e857785a175a760/gossip/src/contact\_info.rs#L31) focuses exclusively on voting transactions
* [tpu\_forwards](https://github.com/solana-labs/solana/blob/cd6f931223181d5a1d47cba64e857785a175a760/gossip/src/contact\_info.rs#L29) forwards unprocessed packets to the next leader if the current leader is unable to process all transactions

For more information on the TPU, please refer to [this excellent writeup by Jito Labs](https://jito-labs.medium.com/solana-validator-101-transaction-processing-90bcdc271143).

### How Transactions Get Dropped [#](https://solana.com/docs/advanced/retry#how-transactions-get-dropped) <a href="#how-transactions-get-dropped" id="how-transactions-get-dropped"></a>

Throughout a transaction’s journey, there are a few scenarios in which the transaction can be unintentionally dropped from the network.

#### Before a transaction is processed [#](https://solana.com/docs/advanced/retry#before-a-transaction-is-processed) <a href="#before-a-transaction-is-processed" id="before-a-transaction-is-processed"></a>

If the network drops a transaction, it will most likely do so before the transaction is processed by a leader. UDP [packet loss](https://en.wikipedia.org/wiki/Packet\_loss) is the simplest reason why this might occur. During times of intense network load, it’s also possible for validators to become overwhelmed by the sheer number of transactions required for processing. While validators are equipped to forward surplus transactions via `tpu_forwards`, there is a limit to the amount of data that can be [forwarded](https://github.com/solana-labs/solana/blob/master/core/src/banking\_stage.rs#L389). Furthermore, each forward is limited to a single hop between validators. That is, transactions received on the `tpu_forwards` port are not forwarded on to other validators.

There are also two lesser known reasons why a transaction may be dropped before it is processed. The first scenario involves transactions that are submitted via an RPC pool. Occasionally, part of the RPC pool can be sufficiently ahead of the rest of the pool. This can cause issues when nodes within the pool are required to work together. In this example, the transaction’s [recentBlockhash](https://solana.com/docs/core/transactions#recent-blockhash) is queried from the advanced part of the pool (Backend A). When the transaction is submitted to the lagging part of the pool (Backend B), the nodes will not recognize the advanced blockhash and will drop the transaction. This can be detected upon transaction submission if developers enable [preflight checks](https://solana.com/docs/rpc/http/sendtransaction) on `sendTransaction`.

![Transaction dropped via an RPC Pool](https://solana-developer-content.vercel.app/assets/docs/rt-dropped-via-rpc-pool.png)Transaction dropped via an RPC Pool

Temporarily network forks can also result in dropped transactions. If a validator is slow to replay its blocks within the Banking Stage, it may end up creating a minority fork. When a client builds a transaction, it’s possible for the transaction to reference a `recentBlockhash` that only exists on the minority fork. After the transaction is submitted, the cluster can then switch away from its minority fork before the transaction is processed. In this scenario, the transaction is dropped due to the blockhash not being found.

![Transaction dropped due to minority fork (before processed)](https://solana-developer-content.vercel.app/assets/docs/rt-dropped-minority-fork-pre-process.png)Transaction dropped due to minority fork (before processed)

#### After a transaction is processed and before it is finalized [#](https://solana.com/docs/advanced/retry#after-a-transaction-is-processed-and-before-it-is-finalized) <a href="#after-a-transaction-is-processed-and-before-it-is-finalized" id="after-a-transaction-is-processed-and-before-it-is-finalized"></a>

In the event a transaction references a `recentBlockhash` from a minority fork, it’s still possible for the transaction to be processed. In this case, however, it would be processed by the leader on the minority fork. When this leader attempts to share its processed transactions with the rest of the network, it would fail to reach consensus with the majority of validators that do not recognize the minority fork. At this time, the transaction would be dropped before it could be finalized.

![Transaction dropped due to minority fork (after processed)](https://solana-developer-content.vercel.app/assets/docs/rt-dropped-minority-fork-post-process.png)Transaction dropped due to minority fork (after processed)

### Handling Dropped Transactions [#](https://solana.com/docs/advanced/retry#handling-dropped-transactions) <a href="#handling-dropped-transactions" id="handling-dropped-transactions"></a>

While RPC nodes will attempt to rebroadcast transactions, the algorithm they employ is generic and often ill-suited for the needs of specific applications. To prepare for times of network congestion, application developers should customize their own rebroadcasting logic.

#### An In-Depth Look at sendTransaction [#](https://solana.com/docs/advanced/retry#an-in-depth-look-at-sendtransaction) <a href="#an-in-depth-look-at-sendtransaction" id="an-in-depth-look-at-sendtransaction"></a>

When it comes to submitting transactions, the `sendTransaction` RPC method is the primary tool available to developers. `sendTransaction` is only responsible for relaying a transaction from a client to an RPC node. If the node receives the transaction, `sendTransaction` will return the transaction id that can be used to track the transaction. A successful response does not indicate whether the transaction will be processed or finalized by the cluster.

#### Request Parameters [#](https://solana.com/docs/advanced/retry#request-parameters) <a href="#request-parameters" id="request-parameters"></a>

* `transaction`: `string` - fully-signed Transaction, as encoded string
* (optional) `configuration object`: `object`
  * `skipPreflight`: `boolean` - if true, skip the preflight transaction checks (default: false)
  * (optional) `preflightCommitment`: `string` - [Commitment](https://solana.com/docs/rpc#configuring-state-commitment) level to use for preflight simulations against the bank slot (default: "finalized").
  * (optional) `encoding`: `string` - Encoding used for the transaction data. Either "base58" (slow), or "base64". (default: "base58").
  * (optional) `maxRetries`: `usize` - Maximum number of times for the RPC node to retry sending the transaction to the leader. If this parameter is not provided, the RPC node will retry the transaction until it is finalized or until the blockhash expires.

**Response:**

* `transaction id`: `string` - First transaction signature embedded in the transaction, as base-58 encoded string. This transaction id can be used with [`getSignatureStatuses`](https://solana.com/docs/rpc/http/getsignaturestatuses) to poll for status updates.

### Customizing Rebroadcast Logic [#](https://solana.com/docs/advanced/retry#customizing-rebroadcast-logic) <a href="#customizing-rebroadcast-logic" id="customizing-rebroadcast-logic"></a>

In order to develop their own rebroadcasting logic, developers should take advantage of `sendTransaction`’s `maxRetries` parameter. If provided, `maxRetries` will override an RPC node’s default retry logic, allowing developers to manually control the retry process [within reasonable bounds](https://github.com/solana-labs/solana/blob/98707baec2385a4f7114d2167ef6dfb1406f954f/validator/src/main.rs#L1258-L1274).

A common pattern for manually retrying transactions involves temporarily storing the `lastValidBlockHeight` that comes from [getLatestBlockhash](https://solana.com/docs/rpc/http/getlatestblockhash). Once stashed, an application can then [poll the cluster’s blockheight](https://solana.com/docs/rpc/http/getblockheight) and manually retry the transaction at an appropriate interval. In times of network congestion, it’s advantageous to set `maxRetries` to 0 and manually rebroadcast via a custom algorithm. While some applications may employ an [exponential backoff](https://en.wikipedia.org/wiki/Exponential\_backoff) algorithm, others such as [Mango](https://www.mango.markets/) opt to [continuously resubmit](https://github.com/blockworks-foundation/mango-ui/blob/b6abfc6c13b71fc17ebbe766f50b8215fa1ec54f/src/utils/send.tsx#L713) transactions at a constant interval until some timeout has occurred.

```
import {  Keypair,  Connection,  LAMPORTS_PER_SOL,  SystemProgram,  Transaction,} from "@solana/web3.js";import * as nacl from "tweetnacl"; const sleep = async (ms: number) => {  return new Promise(r => setTimeout(r, ms));}; (async () => {  const payer = Keypair.generate();  const toAccount = Keypair.generate().publicKey;   const connection = new Connection("http://127.0.0.1:8899", "confirmed");   const airdropSignature = await connection.requestAirdrop(    payer.publicKey,    LAMPORTS_PER_SOL,  );   await connection.confirmTransaction({ signature: airdropSignature });   const blockhashResponse = await connection.getLatestBlockhashAndContext();  const lastValidBlockHeight = blockhashResponse.context.slot + 150;   const transaction = new Transaction({    feePayer: payer.publicKey,    blockhash: blockhashResponse.value.blockhash,    lastValidBlockHeight: lastValidBlockHeight,  }).add(    SystemProgram.transfer({      fromPubkey: payer.publicKey,      toPubkey: toAccount,      lamports: 1000000,    }),  );  const message = transaction.serializeMessage();  const signature = nacl.sign.detached(message, payer.secretKey);  transaction.addSignature(payer.publicKey, Buffer.from(signature));  const rawTransaction = transaction.serialize();  let blockheight = await connection.getBlockHeight();   while (blockheight < lastValidBlockHeight) {    connection.sendRawTransaction(rawTransaction, {      skipPreflight: true,    });    await sleep(500);    blockheight = await connection.getBlockHeight();  }})();
```

When polling via `getLatestBlockhash`, applications should specify their intended [commitment](https://solana.com/docs/rpc#configuring-state-commitment) level. By setting its commitment to `confirmed` (voted on) or `finalized` (\~30 blocks after `confirmed`), an application can avoid polling a blockhash from a minority fork.

If an application has access to RPC nodes behind a load balancer, it can also choose to divide its workload amongst specific nodes. RPC nodes that serve data-intensive requests such as [getProgramAccounts](https://solanacookbook.com/guides/get-program-accounts.html) may be prone to falling behind and can be ill-suited for also forwarding transactions. For applications that handle time-sensitive transactions, it may be prudent to have dedicated nodes that only handle `sendTransaction`.

#### The Cost of Skipping Preflight [#](https://solana.com/docs/advanced/retry#the-cost-of-skipping-preflight) <a href="#the-cost-of-skipping-preflight" id="the-cost-of-skipping-preflight"></a>

By default, `sendTransaction` will perform three preflight checks prior to submitting a transaction. Specifically, `sendTransaction` will:

* Verify that all signatures are valid
* Check that the referenced blockhash is within the last 150 blocks
* Simulate the transaction against the bank slot specified by the `preflightCommitment`

In the event that any of these three preflight checks fail, `sendTransaction` will raise an error prior to submitting the transaction. Preflight checks can often be the difference between losing a transaction and allowing a client to gracefully handle an error. To ensure that these common errors are accounted for, it is recommended that developers keep `skipPreflight` set to `false`.

#### When to Re-Sign Transactions [#](https://solana.com/docs/advanced/retry#when-to-re-sign-transactions) <a href="#when-to-re-sign-transactions" id="when-to-re-sign-transactions"></a>

Despite all attempts to rebroadcast, there may be times in which a client is required to re-sign a transaction. Before re-signing any transaction, it is **very important** to ensure that the initial transaction’s blockhash has expired. If the initial blockhash is still valid, it is possible for both transactions to be accepted by the network. To an end-user, this would appear as if they unintentionally sent the same transaction twice.

In Solana, a dropped transaction can be safely discarded once the blockhash it references is older than the `lastValidBlockHeight` received from `getLatestBlockhash`. Developers should keep track of this `lastValidBlockHeight` by querying [`getEpochInfo`](https://solana.com/docs/rpc/http/getepochinfo) and comparing with `blockHeight` in the response. Once a blockhash is invalidated, clients may re-sign with a newly-queried blockhash.

## [Versioned Transactions](https://solana.com/docs/advanced/versions)

Versioned Transactions are the new transaction format that allow for additional functionality in the Solana runtime, including [Address Lookup Tables](https://solana.com/docs/advanced/lookup-tables).

While changes to [onchain](https://solana.com/docs/programs) programs are **NOT** required to support the new functionality of versioned transactions (or for backwards compatibility), developers **WILL** need update their client side code to prevent [errors due to different transaction versions](https://solana.com/docs/advanced/versions#max-supported-transaction-version).

### Current Transaction Versions [#](https://solana.com/docs/advanced/versions#current-transaction-versions) <a href="#current-transaction-versions" id="current-transaction-versions"></a>

The Solana runtime supports two transaction versions:

* `legacy` - older transaction format with no additional benefit
* `0` - added support for [Address Lookup Tables](https://solana.com/docs/advanced/lookup-tables)

### Max supported transaction version [#](https://solana.com/docs/advanced/versions#max-supported-transaction-version) <a href="#max-supported-transaction-version" id="max-supported-transaction-version"></a>

All RPC requests that return a transaction _**should**_ specify the highest version of transactions they will support in their application using the `maxSupportedTransactionVersion` option, including [`getBlock`](https://solana.com/docs/rpc/http/getblock) and [`getTransaction`](https://solana.com/docs/rpc/http/gettransaction).

An RPC request will fail if a Versioned Transaction is returned that is higher than the set `maxSupportedTransactionVersion`. (i.e. if a version `0` transaction is returned when `legacy` is selected)

INFO

WARNING: If no `maxSupportedTransactionVersion` value is set, then only `legacy` transactions will be allowed in the RPC response. Therefore, your RPC requests **WILL** fail if any version `0` transactions are returned.

### How to set max supported version [#](https://solana.com/docs/advanced/versions#how-to-set-max-supported-version) <a href="#how-to-set-max-supported-version" id="how-to-set-max-supported-version"></a>

You can set the `maxSupportedTransactionVersion` using both the [`@solana/web3.js`](https://solana-labs.github.io/solana-web3.js/) library and JSON formatted requests directly to an RPC endpoint.

#### Using web3.js [#](https://solana.com/docs/advanced/versions#using-web3js) <a href="#using-web3js" id="using-web3js"></a>

Using the [`@solana/web3.js`](https://solana-labs.github.io/solana-web3.js/) library, you can retrieve the most recent block or get a specific transaction:

```
// connect to the `devnet` cluster and get the current `slot`const connection = new web3.Connection(web3.clusterApiUrl("devnet"));const slot = await connection.getSlot(); // get the latest block (allowing for v0 transactions)const block = await connection.getBlock(slot, {  maxSupportedTransactionVersion: 0,}); // get a specific transaction (allowing for v0 transactions)const getTx = await connection.getTransaction(  "3jpoANiFeVGisWRY5UP648xRXs3iQasCHABPWRWnoEjeA93nc79WrnGgpgazjq4K9m8g2NJoyKoWBV1Kx5VmtwHQ",  {    maxSupportedTransactionVersion: 0,  },);
```

#### JSON requests to the RPC [#](https://solana.com/docs/advanced/versions#json-requests-to-the-rpc) <a href="#json-requests-to-the-rpc" id="json-requests-to-the-rpc"></a>

Using a standard JSON formatted POST request, you can set the `maxSupportedTransactionVersion` when retrieving a specific block:

```
curl https://api.devnet.solana.com -X POST -H "Content-Type: application/json" -d \'{"jsonrpc": "2.0", "id":1, "method": "getBlock", "params": [430, {  "encoding":"json",  "maxSupportedTransactionVersion":0,  "transactionDetails":"full",  "rewards":false}]}'
```

### How to create a Versioned Transaction [#](https://solana.com/docs/advanced/versions#how-to-create-a-versioned-transaction) <a href="#how-to-create-a-versioned-transaction" id="how-to-create-a-versioned-transaction"></a>

Versioned transactions can be created similar to the older method of creating transactions. There are differences in using certain libraries that should be noted.

Below is an example of how to create a Versioned Transaction, using the `@solana/web3.js` library, to send perform a SOL transfer between two accounts.

**Notes:** [**#**](https://solana.com/docs/advanced/versions#notes:)

* `payer` is a valid `Keypair` wallet, funded with SOL
* `toAccount` a valid `Keypair`

Firstly, import the web3.js library and create a `connection` to your desired cluster.

We then define the recent `blockhash` and `minRent` we will need for our transaction and the account:

```
const web3 = require("@solana/web3.js"); // connect to the cluster and get the minimum rent for rent exempt statusconst connection = new web3.Connection(web3.clusterApiUrl("devnet"));let minRent = await connection.getMinimumBalanceForRentExemption(0);let blockhash = await connection  .getLatestBlockhash()  .then(res => res.blockhash);
```

Create an `array` of all the `instructions` you desire to send in your transaction. In this example below, we are creating a simple SOL transfer instruction:

```
// create an array with your desired `instructions`const instructions = [  web3.SystemProgram.transfer({    fromPubkey: payer.publicKey,    toPubkey: toAccount.publicKey,    lamports: minRent,  }),];
```

Next, construct a `MessageV0` formatted transaction message with your desired `instructions`:

```
// create v0 compatible messageconst messageV0 = new web3.TransactionMessage({  payerKey: payer.publicKey,  recentBlockhash: blockhash,  instructions,}).compileToV0Message();
```

Then, create a new `VersionedTransaction`, passing in our v0 compatible message:

```
const transaction = new web3.VersionedTransaction(messageV0); // sign your transaction with the required `Signers`transaction.sign([payer]);
```

You can sign the transaction by either:

* passing an array of `signatures` into the `VersionedTransaction` method, or
* call the `transaction.sign()` method, passing an array of the required `Signers`

INFO

NOTE: After calling the `transaction.sign()` method, all the previous transaction `signatures` will be fully replaced by new signatures created from the provided in `Signers`.

After your `VersionedTransaction` has been signed by all required accounts, you can send it to the cluster and `await` the response:

```
// send our v0 transaction to the clusterconst txId = await connection.sendTransaction(transaction);console.log(`https://explorer.solana.com/tx/${txId}?cluster=devnet`);
```

INFO

NOTE: Unlike `legacy` transactions, sending a `VersionedTransaction` via `sendTransaction` does **NOT** support transaction signing via passing in an array of `Signers` as the second parameter. You will need to sign the transaction before calling `connection.sendTransaction()`.

### More Resources [#](https://solana.com/docs/advanced/versions#more-resources) <a href="#more-resources" id="more-resources"></a>

* using [Versioned Transactions for Address Lookup Tables](https://solana.com/docs/advanced/lookup-tables#how-to-create-an-address-lookup-table)
* view an [example of a v0 transaction](https://explorer.solana.com/tx/h9WQsqSUYhFvrbJWKFPaXximJpLf6Z568NW1j6PBn3f7GPzQXe9PYMYbmWSUFHwgnUmycDNbEX9cr6WjUWkUFKx/?cluster=devnet) on Solana Explorer
* read the [accepted proposal](https://docs.solanalabs.com/proposals/versioned-transactions) for Versioned Transaction and Address Lookup Tables

