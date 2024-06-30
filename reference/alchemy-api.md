# Alchemy API

## Getting Started Instructions

### 1. Choose a package manager (npm or yarn)

For this guide, we will be using npm or yarn as our package manager to install either `alchemy-sdk` or any other packages.

#### npm

To get started with `npm`, follow the documentation to install Node.js and `npm` for your operating system: [https://docs.npmjs.com/downloading-and-installing-node-js-and-npm](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)

#### yarn

To get started with `yarn`, follow these steps: [https://classic.yarnpkg.com/lang/en/docs/install](https://classic.yarnpkg.com/lang/en/docs/install/#mac-stable)

### 2. Set up your project (npm or yarn)

Shell (npm)Shell (yarn)

```
mkdir alchemy-solana-api
cd alchemy-solana-api
npm init --yes
```

### 3. Install Solana Web3.js Library

Run the following command to install the Solana Web3.js library with npm or yarn.

npmyarn

```
npm install --save @solana/web3.js
```

### 4. Make your first request

You are all set now to use Solana API and make your first request. For instance, lets make a request to `get latest slot`. Create an `index.js` file and paste the following code snippet into the file.

index.js

```javascript
const solanaWeb3 = require("@solana/web3.js");

const main = async () => {
  rpc = "https://solana-mainnet.g.alchemy.com/v2/<Your-Alchemy-API-Key>"; // RPC URL for connecting with a Solana node
  connection = new solanaWeb3.Connection(rpc, "confirmed"); // confirming the connection

  let slot = await connection.getSlot(); // getting the most recent slot number
  console.log("The latest slot number is", slot); // logging the most recent slot number
};

main();
```

### 5. Run script

To run the above node script, use cmd `node index.js`, and you should see the output.

shell

```
The latest slot number is 164150510
```

***

## Leverage Alchemy's AccountsDB Infrastructure for Solana RPC Requests!

The most expensive Solana RPC requests involve account scans, such as `getProgramAccounts` and `getLargestTokenAccounts`. These methods are incredibly useful but non-performant methods since they induce a heavy RPC load on Solana validator nodes, often resulting in a 5XX response due to timeout or a response with high latency.

Alchemy has built out core infrastructure that sits atop our Solana validator nodes to support these methods at scale, through what we call the AccountsDB Infrastructure. This infrastructure allows for **faster**, **scalable**, and more **reliable** responses to these methods by paginating the response with a `pageKey`. You could then loop through your that same request with at scan and aggregate the full response from our validator nodes.

You can see `pageKey` is now an optional parameter in each account-scanning method in our Solana [docs](https://alchemyenterprisegroup.readme.io/reference/getprogramaccounts), and you may also include an `order` optional parameter that would sort the accounts in the response by their `pubkey` field.

Here's an example with `getProgramAccounts`:

TypeScript

```typescript
const axios = require("axios");

function getProgramAccountsExample() {
 let gPAExampleRequest = {
   "method": "alchemy_getProgramAccounts",
   "params": [
     "ZETAxsqBRek56DhiGXrn75yj2NHU3aYUnxvHXpkf3aD",
     {
       "encoding": "base64",
       "withContext": true,
       "order": "desc"
     }
   ],
   "id": 0,
   "jsonrpc": "2.0"
 }
 let programAccounts = []

 const alchemyRPCUrl = "https://solana-mainnet.g.alchemy.com/v2/<YOUR-API-KEY>"
 try {
   let response = await axios.post(url, gPAExampleRequest);
   let responseData = response.data["result"]
   
   // continue aggregating if there's a new pageKey present in the latest response
   while (responseData["pageKey"]) {
     programAccounts = programAccounts.concat(responseData["value"]);
     
     // place the pagekey within the optional config object
     // (you may need to create that config object if you didn't have it originally)
     gPAExampleRequest["params"][1]["pageKey"] = responseData["pageKey"];
     
     // make another call to getProgramAccounts with the pageKey
     response = await axios.post(url, gPAExampleRequest);
     responseData = response.data["result"]
   }

    programAccounts = programAccounts.concat(responseData["value"]);
    return programAccounts;
  } catch (err) {
    console.error(`Error in Response, Data is: ${err.data}`);
    return [];
  }
}
```

## Best Practices When Using Alchemy

Here is a list of best practices to reduce [compute unit usage costs](https://alchemyenterprisegroup.readme.io/reference/compute-units) to make sure you’re getting the most out of Alchemy’s platform!

We will continue updating these as we receive customer feedback.

### 1. Send Requests Concurrently

Depending on your background with blockchain nodes, you might expect that requests need to be sent sequentially to function properly. That’s not the case!

**Don’t treat Alchemy like a single node or a group of nodes** - treat it like an automated, scalable service that can handle concurrent request patterns.

You don’t need to be concerned about overloading Alchemy with concurrent requests at scale. Alchemy is built specifically to process high rates of concurrent requests for [all of our web3 customers](https://www.alchemy.com/all-case-studies).

### 2. Avoid High Batch Cardinality

When sending batch requests, **aim for batches under 50 requests per call**.

If you need hundreds or thousands of responses quickly, send more batched requests concurrently rather than placing all your requests in a single call.

Blockchain responses tend to be heavy, which means that responses for certain requests sent to the nodes (like [eth\_getLogs](https://alchemyenterprisegroup.readme.io/docs/deep-dive-into-eth\_getlogs) can have an unbounded size or time to execute.

By batching smaller sets of requests, you can minimize time-outs as the result of unbounded response sizes, and indefinite execution times, and guarantee higher throughputs.

Additionally, it’ll be easier to identify and solve requests that are failing. Instead of having to retry a batch with 100+ requests, you can quickly retry a subset of requests without the failing query.

### 3. Retry (w/ Exponential Backoff) on Failures, Not On Client-Side Timeouts

Many types of common node requests require long processing times or unbounded response sizes, leading to slower response times (oftentimes from 1 - 10+ seconds).

If you’re canceling and re-sending requests on client-side timeouts that are too short, you could end up never receiving the response you’re looking for and spamming the node infrastructure with expensive requests that waste your compute units.

Instead, **retry your requests with exponential backoff on response failures**, which will reduce your compute unit costs, increase the success rate of your requests, and allow you to handle failures properly based on their respective error messages.

If you need to handle timeouts, you can also retry your requests on Alchemy-based timeouts, which will prevent you from accidentally retrying requests before nodes have finished processing them. Alternatively, increase your client-side timeouts to see if your request success rate improves.

To save development overhead, you can use the [Alchemy SDK](https://github.com/alchemyplatform/alchemy-sdk-js) which handles exponential backoff retries on failure for you automatically, among other benefits!

### 4. Send Requests over HTTPS, not WebSockets

Though it may be tempting to use WebSockets for all node requests because it’s a newer technology, the [industry best practice for using WebSockets](https://alchemyenterprisegroup.readme.io/docs/best-practices-for-using-websockets-in-web3) is still primarily for push-based notifications. In the case of EVM chains, `eth_subscribe` and `eth_unsubscribe` to certain events.

HTTPS is a better option for standard JSON-RPC node requests for several reasons:

* **Silent failures**: WebSockets client-side handling has many tricky edge cases and silent failure modes.
* **Load balancing**: When making requests to distributed systems such as Alchemy, individual HTTP requests are load-balanced to the fastest possible server, whereas with WebSockets you incur additional latency by sending JSON-RPC requests only to a single node.
* **Retries**: In most common request frameworks, support for retrying failed HTTP requests comes automatically, and can be configured easily. Conversely, in WebSockets retrying failed requests typically requires custom JSON-RPC id-based tracking.
* **HTTP status codes**: When web3 developers use WebSockets they won't receive HTTP status codes in WebSockets responses, which can be useful for debugging or sorting responses.

### 4. Avoid Large Request / Response Sizes

We recommend keeping the vast majority of requests to be under 100 KB and avoiding response sizes above 10 MB.

Though we permit sending large requests (currently up to 2.5 MB) and receiving large responses (currently up to 150 MB), we strongly suggest avoiding these limits as much as possible. This is for several reasons: Larger requests and responses are more likely to hit our size limits, which will result in failing API calls that you’ll have to retry. Heavy API calls have higher likelihood of timing out, failing while in flight, and causing nodes to become unstable. Smaller API calls are easier to debug and identify issues that arise.

By keeping your API calls an order of magnitude smaller than our hard limits, your infrastructure will become more reliable, responsive, and you’ll spend less time debugging your dApp.

### 5. Use gZip Compression to Speed Up Large Requests

At Alchemy, many of our developers have brought up slow response times as a major blocker to providing their customers with a good web3 user experience.

To provide users with better product experiences, we updated our internal infrastructure to offer Alchemy developers **support for gzip compression on all responses larger than 1kb in size.**

In practice, we’ve seen roughly a **75% improvement in the total latency of typical JSON-RPC replayTransaction calls.**

Go to the article below to learn how to implement gZip compression:

## How to Enable Compression to Speed Up JSON-RPC Blockchain Requests

Adding an "Accept-Encoding: gzip" header to JSON-RPC requests results in roughly a 75% speedup for requests over 100kb. Use this single code change to speed up JSON-RPC requests!

**TL;DR -** Adding an "Accept-Encoding: gzip" header to JSON-RPC requests results in roughly a 75% speedup for requests over 100kb. Use this single code change to speed up JSON-RPC requests for Ethereum, Polygon Optimism, Arbitrum, and more!

Because of the structure of data storage in [blockchain nodes](https://www.alchemy.com/blog/what-is-a-node-provider), RPC endpoints that provide access to blockchain nodes can have extremely slow requests and response times. Many requests, such as a simple `getLogs` or `replayTransaction` call, can take anywhere from 1 to 30 seconds to process

#### What causes slow response times?

The round-trip time of these requests can take a while for two primary reasons: (1) the request or response is large; (2) the requests are complex.

**1. Large Requests or Responses**

The size of the request or response can become quite large - up to 1MB for certain requests - and from 10MB - 250MB for certain response patterns. The latency to send or receive these calls can take seconds to process, depending on the connection speeds of the client or server.

**2. Complex Requests**

The requests themselves can take the nodes seconds to process, as they oftentimes involve replaying complicated transactions from scratch or scanning thousands of blocks to identify relevant transactions.

Because of this, the [best practice for making calls to RPC node providers](https://docs.alchemy.com/alchemy/documentation/best-practices-when-using-alchemy) includes:

1. Keeping request sizes under 100KB
2. Keeping response sizes under 10MB
3. Sending requests in batches no larger than 50

Together, these best practices help developers avoid client-side timeouts and unreliable behavior

***

### Compressing RPC Responses to Speed Up Blockchain Node Requests

At Alchemy, many of our developers have brought up slow response times as a major blocker to providing their customers with a good web3 user experience.

To provide users with better product experiences, we updated our internal infrastructure to offer **Alchemy developers support for gzip compression on all responses larger than 1kb in size.**

Gzip compression offers up to a 95% decrease in the size of files sent over a streaming connection. However, the actual latency and bandwidth savings are dependent on the structure of data being broadcast, the connection speeds of the client and server, and the size of the response.

In practice, we’ve seen roughly a **75% improvement in the total latency of typical JSON-RPC `replayTransaction` calls.**

#### Can JSON-RPC responses be compressed on Optimism, Arbitrum, Polygon, Starknet, or Solana?

Beyond support for gZip compression for Ethereum, this method of compressing JSON-RPC requests works on all the above blockchains. To implement this latency optimization, simply enable Gzip compression using your Alchemy endpoint

***

### How to Enable Gzip Compression on Node Requests

#### Step 1: Set up an Alchemy Account

To enable gzip compression, first, you’ll need an Alchemy endpoint. [If you don’t already have an Alchemy account, sign up for free here.](https://alchemy.com/?a=gzip-compression)

If you’re already building applications on Alchemy’s developer platform, sign in to your account and skip to step 3.

Alchemy is a blockchain developer platform and suite of APIs that allow developers to communicate with multiple blockchains without having to run their own nodes. Alchemy comes with 300 million compute units per month for free, which is equivalent to roughly 12 million free requests per month.

#### Step 2: Create your app and API key

Once you’ve created an Alchemy account, you can generate an API key by creating an app. This will allow you to make requests to the mainnet.

Navigate to the “Create App” page in your Alchemy Dashboard by hovering over “Apps” in the nav bar. Then, click “Create App.”

Then:

* Name your app “Hello World”
* Offer a short description
* Click “Create App”!

Your app should appear in the table. Finally, click on “View Key” on the right-hand side and copy the HTTPS URL.

#### Step 3: Make a command-line node request with gzip enabled

In order to enable gzip compression, **you’ll simply need to provide the additional field "Accept-Encoding: gzip" to the header of the JSON-RPC request.**

When making a curl request in the terminal, your request with gzip might look like this:

cURL

```curl
curl https://eth-mainnet.g.alchemy.com/v2/demo
-v -X POST
-H "Content-Type: application/json"
-H "Accept-Encoding: gzip"
-d '{"method":"trace_replayTransaction","params":["0x3277c743c14e482243862c03a70e83ccb52e25cb9e54378b20a8303f15cb985d",["trace"]],"id":1,"jsonrpc":"2.0"}'
```

***

### Test the JSON-RPC Response Latency Decrease using Gzip Compression

To compare the latency improvements you’ll get using gzip on a single request, we’ll set up a script to record the total time a [`trace_replayTransaction`](https://docs.alchemy.com/alchemy/enhanced-apis/trace-api/trace\_replaytransaction) request takes, and then run the same request with gzip compression and without gzip compression

#### Step 1: Create a curl-format file

Create a new file named `curl-format.txt` and add the following lines:

curl-format.txt

```
time_namelookup:  %{time_namelookup}s\n
time_connect:  %{time_connect}s\n
time_appconnect:  %{time_appconnect}s\n
time_pretransfer:  %{time_pretransfer}s\n
time_redirect:  %{time_redirect}s\n
time_starttransfer:  %{time_starttransfer}s\n
—------------------------------------------------------\n
time_total:  %{time_total}s\n
```

#### Step 2: Run a test JSON-RPC request script without gzip compression

Next, run the following script without gzip compression on an arbitrary node request:

cURL

```curl
curl -w "@curl-format.txt" -o /dev/null -s https://eth-mainnet.g.alchemy.com/v2/demo
-v -X POST
-H "Content-Type: application/json"
-d '{"method":"trace_replayTransaction","params":["0x3277c743c14e482243862c03a70e83ccb52e25cb9e54378b20a8303f15cb985d",["trace"]],"id":1,"jsonrpc":"2.0"}'
```

On our pass, **we got the following output with approximately a 4-second response time:**

JSON

```json
time_namelookup:  0.004295s
time_connect:  0.015269s
time_appconnect:  0.055590s
time_pretransfer:  0.056517s
time_redirect:  0.000000s
time_starttransfer:  0.056595s
----------
time_total:  4.017589s
```

#### Step 3: Run a test JSON-RPC request with gzip compression enabled

Run the same script, but this time with gzip compression enabled on the same node request:

curl-format

```curl
curl -w "@curl-format.txt" -o /dev/null -s https://eth-mainnet.g.alchemy.com/v2/demo
-v -X POST
-H "Content-Type: application/json"
-H "Accept-Encoding: gzip"
-d '{"method":"trace_replayTransaction","params":["0x3277c743c14e482243862c03a70e83ccb52e25cb9e54378b20a8303f15cb985d",["trace"]],"id":1,"jsonrpc":"2.0"}'
```

On our pass, **we got the following output with a roughly 1 second response time:**

response

```
time_namelookup: 0.030062s
time_connect: 0.046659s
time_appconnect: 0.099016s
time_pretransfer: 0.099198s
time_redirect: 0.000000s
time_starttransfer: 0.099243s
----------
time_total: 0.984284s
```

As you can see, in just two responses we’ve seen a **75% decrease in total latency!**

JSON-RPC response latency improvements are dependent on many factors, including connection speed from the client, type of request, and size of the response.

Though this number will vary dramatically based on these factors, you’ll typically see a non-trivial decrease in latency using gzip compression for speeding up large response packages.

## How to Implement Retries

Learn how to implement retries in your code to handle errors and improve application reliability.

[Suggest Edits](https://docs.alchemy.com/edit/how-to-implement-retries)

## Introduction

Alchemy is a powerful platform that provides developers with advanced blockchain tools, such as APIs, monitoring, and analytics, to build their blockchain applications faster and more efficiently. Alchemy's [Elastic Throughput system](https://docs.alchemy.com/reference/throughput) guarantees a given [throughput](https://docs.alchemy.com/reference/throughput#what-is-throughput) limit measured in [compute units per second](https://docs.alchemy.com/reference/throughput#what-are-compute-units-per-second-cups), but you may still hit your throughput capacity in some cases. In this tutorial, we will explore how to implement retries to handle [Alchemy 429 errors](https://docs.alchemy.com/reference/throughput#error-response).

## Option 1: Alchemy SDK

The Alchemy SDK is the easiest way to connect your dApp to the blockchain. It automatically handles retry logic for you. To use the Alchemy SDK, follow these steps:

1. Create a new node.js project and Install the Alchemy SDK using npm or yarn:

npmyarn

```shell
mkdir my-project
cd my-project
npm install alchemy-sdk
```

2. Import and configure the Alchemy SDK with your API key and choice of network.

JavaScript

```javascript
// Importing the Alchemy SDK
const { Network, Alchemy } = require('alchemy-sdk');

// Configuring the Alchemy SDK
const settings = {
  apiKey: 'demo', // Replace with your Alchemy API Key.
  network: Network.ETH_MAINNET, // Replace with your network.
};

// Creating an instance to make requests
const alchemy = new Alchemy(settings);
```

3. Start making requests to the blockchain:

JavaScript

```javascript
// getting the current block number and logging to the console
alchemy.core.getBlockNumber().then(console.log);
```

4. Here's the complete code:

JavaScript

```javascript
// Importing the Alchemy SDK
const { Network, Alchemy } = require("alchemy-sdk");

// Configuring the Alchemy SDK
const settings = {
  apiKey: "demo", // Replace with your Alchemy API Key.
  network: Network.ETH_MAINNET, // Replace with your network.
};

// Creating an instance to make requests
const alchemy = new Alchemy(settings);

// getting the current block number and logging to the console
alchemy.core.getBlockNumber().then(console.log);
```

The Alchemy SDK automatically handles retries for you, so you don't need to worry about implementing retry logic.

## Option 2: Exponential Backoff

Exponential backoff is a standard error-handling strategy for network applications. It is a similar solution to retries, however, instead of waiting random intervals, an exponential backoff algorithm retries requests exponentially, increasing the waiting time between retries up to a maximum backoff time.

Here is an example of an exponential backoff algorithm:

1. Make a request.
2. If the request fails, wait `1 + random_number_milliseconds` seconds and retry the request.
3. If the request fails, wait `2 + random_number_milliseconds` seconds and retry the request.
4. If the request fails, wait `4 + random_number_milliseconds` seconds and retry the request.
5. And so on, up to a maximum\_backoff time...
6. Continue waiting and retrying up to some maximum number of retries, but do not increase the wait period between retries.

Where:

* The wait time is `min(((2^n)+random_number_milliseconds), maximum_backoff)`, with `n` incremented by 1 for each iteration (request).
* `random_number_milliseconds` is a random number of milliseconds less than or equal to 1000. This helps to avoid cases in which many clients are synchronized by some situation and all retry at once, sending requests in synchronized waves. The value of `random_number_milliseconds` is recalculated after each retry request.
* `maximum_backoff` is typically 32 or 64 seconds. The appropriate value depends on the use case.
* The client can continue retrying after it has reached the `maximum_backoff` time. Retries after this point do not need to continue increasing backoff time. For example, suppose a client uses a `maximum_backoff` time of 64 seconds. After reaching this value, the client can retry every 64 seconds. At some point, clients should be prevented from retrying indefinitely.

To implement exponential backoff in your Alchemy application, you can use a library such as [`retry`](https://www.npmjs.com/package/retry) or [`async-retry`](https://www.npmjs.com/package/async-retry) for handling retries in a more structured and scalable way.

Here's an example implementation of exponential backoff using the [`async-retry`](https://www.npmjs.com/package/async-retry) library in a Node.js application where we call the `eth_blockNumber` API using Alchemy:

JavaScript

```javascript
// Setup: npm install node-fetch@2.4.0 | npm install async-retry

// Import required modules
const fetch = require("node-fetch");
const retry = require("async-retry");

// Set your API key
const apiKey = "demo"; // Replace with your Alchemy API key

// Set the endpoint and request options
const url = `https://eth-mainnet.g.alchemy.com/v2/${apiKey}`;
const options = {
  method: "POST",
  headers: { accept: "application/json", "content-type": "application/json" },
  body: JSON.stringify({ id: 1, jsonrpc: "2.0", method: "eth_blockNumber" }),
};

// Create a function to fetch with retries
const fetchWithRetries = async () => {
  const result = await retry(
    async () => {
      // Make the API request
      const response = await fetch(url, options);

      // Parse the response JSON
      let json = await response.json();

      // If we receive a 429 error (Too Many Requests), log an error and retry
      if (json.error && json.error.code === 429) {
        console.error("HTTP error 429: Too Many Requests, retrying...");
        throw new Error("HTTP error 429: Too Many Requests, retrying...");
      }

      // Otherwise, return the response JSON
      return json;
    },
    {
      retries: 5, // Number of retries before giving up
      factor: 2, // Exponential factor
      minTimeout: 1000, // Minimum wait time before retrying
      maxTimeout: 60000, // Maximum wait time before retrying
      randomize: true, // Randomize the wait time
    }
  );

  // Return the result
  return result;
};

// Call the fetchWithRetries function and log the result, or any errors
fetchWithRetries()
  .then((json) => console.log(json))
  .catch((err) => console.error("error:" + err));
```

In this example, we define a new function called `fetchWithRetries` that uses the `async-retry` library to retry the fetch request with exponential backoff. The retry function takes two arguments:

1. An async function that performs the fetch request and returns a response object or throws an error.
2. An options object that specifies the retry behavior. We set the number of retries to 5, the exponential factor to 2, and the minimum and maximum wait times to 1 second and 60 seconds, respectively.

Finally, we call the `fetchWithRetries` function and log the result or the error to the console.

## Option 3: Simple Retries

If exponential backoff poses a challenge to you, a simple retry solution is to wait a random interval between 1000 and 1250 milliseconds after receiving a 429 response and sending the request again, up to some maximum number of attempts you are willing to wait.

Here's an example implementation of simple retries in a node.js application where we call the `eth_blocknumber` API using Alchemy:

JavaScript

```javascript
// Setup: npm install node-fetch@2.4.0

// Import required modules
const fetch = require("node-fetch");

// Set your API key
const apiKey = "demo"; // Replace with your Alchemy API key

// Set the endpoint and request options
const url = `https://eth-mainnet.g.alchemy.com/v2/${apiKey}`;
const options = {
  method: "POST",
  headers: { accept: "application/json", "content-type": "application/json" },
  body: JSON.stringify({ id: 1, jsonrpc: "2.0", method: "eth_blockNumber" }),
};

const maxRetries = 5; // Maximum number of retries before giving up
let retries = 0; // Current number of retries

// Create a function to make the request
function makeRequest() {
  fetch(url, options)
    .then((res) => {
      if (res.status === 429 && retries < maxRetries) {
        // If we receive a 429 response, wait for a random amount of time and try again
        const retryAfter = Math.floor(Math.random() * 251) + 1000; // Generate a random wait time between 1000ms and 1250ms
        console.log(`Received 429 response, retrying after ${retryAfter} ms`);
        retries++;
        setTimeout(() => {
          makeRequest(); // Try the request again after the wait time has elapsed
        }, retryAfter);
      } else if (res.ok) {
        return res.json(); // If the response is successful, return the JSON data
      } else {
        throw new Error(`Received ${res.status} status code`); // If the response is not successful, throw an error
      }
    })
    .then((json) => console.log(json)) // Log the JSON data if there were no errors
    .catch((err) => {
      if (retries < maxRetries) {
        console.error(`Error: ${err.message}, retrying...`);
        retries++;
        makeRequest(); // Try the request again
      } else {
        console.error(`Max retries reached, exiting: ${err.message}`);
      }
    });
}

makeRequest(); // Call the function to make the initial request.
```

* In this example, we define a `maxRetries` constant to limit the number of retries we're willing to wait. We also define a `retries` variable to keep track of how many times we've retried so far.
* We then define the `makeRequest()` function, which is responsible for making the API request. We use the `fetch` function to send the request with the specified `url` and options.
* We then check the response status: if it's a `429 (Too Many Requests)` response and we haven't reached the `maxRetries` limit, we wait a random interval between 1000 and 1250 milliseconds before calling `makeRequest()` again. Otherwise, if the response is `OK`, we parse the JSON response using `res.json()` and log it to the console. If the response status is anything else, we throw an error.
* If an error is caught, we check if we've reached the `maxRetries` limit. If we haven't, we log an error message and call `makeRequest()` again after waiting a random interval between 1000 and 1250 milliseconds. If we have reached the `maxRetries` limit, we log an error message and exit the function.

Finally, we call makeRequest() to start the process.

## Option 4: Retry-After

If you're using HTTP instead of WebSockets, you might come across a 'Retry-After' header in the HTTP response. This header serves as the duration you should wait before initiating a subsequent request. Despite the utility of the 'Retry-After' header, we continue to advise the use of exponential backoff. This is because the 'Retry-After' header only provides a fixed delay duration, while exponential backoff offers a more adaptable delay scheme. By adjusting the delay durations, exponential backoff can effectively prevent a server from being swamped with a high volume of requests in a short time frame.

Here's an example implementation of "Retry-After" in a node.js application where we call the `eth_blocknumber` API using Alchemy:

Go

```go
// Setup: npm install node-fetch@2.4.0

// Import required modules
const fetch = require("node-fetch");

// Set your API key
const apiKey = "demo"; // Replace with your Alchemy API key

// Set the endpoint and request options
const url = `https://eth-mainnet.g.alchemy.com/v2/${apiKey}`;
const options = {
  method: "POST",
  headers: {
    accept: "application/json",
    "content-type": "application/json",
  },
  body: JSON.stringify({ id: 1, jsonrpc: "2.0", method: "eth_blockNumber" }),
};

const maxRetries = 5; // maximum number of retries
let retries = 0; // number of retries

// Create a function to fetch with retries
function makeRequest() {
  fetch(url, options)
    .then((res) => {
      if (res.status === 429 && retries < maxRetries) {
        // check for 429 status code and if max retries not reached
        const retryAfter = res.headers.get("Retry-After"); // get the value of Retry-After header in the response
        if (retryAfter) {
          // if Retry-After header is present
          const retryAfterMs = parseInt(retryAfter) * 1000; // convert Retry-After value to milliseconds
          console.log(
            `Received 429 response, retrying after ${retryAfter} seconds`
          );
          retries++;
          setTimeout(() => {
            makeRequest(); // call the same function after the delay specified in Retry-After header
          }, retryAfterMs);
        } else {
          // if Retry-After header is not present
          const retryAfterMs = Math.floor(Math.random() * 251) + 1000; // generate a random delay between 1 and 250 milliseconds
          console.log(
            `Received 429 response, retrying after ${retryAfterMs} ms`
          );
          retries++;
          setTimeout(() => {
            makeRequest(); // call the same function after the random delay
          }, retryAfterMs);
        }
      } else if (res.ok) {
        // if response is successful
        return res.json(); // parse the response as JSON
      } else {
        throw new Error(`Received ${res.status} status code`); // throw an error for any other status code
      }
    })
    .then((json) => console.log(json)) // log the JSON response
    .catch((err) => {
      if (retries < maxRetries) {
        // if max retries not reached
        console.error(`Error: ${err.message}, retrying...`);
        retries++;
        makeRequest(); // call the same function again
      } else {
        // if max retries reached
        console.error(`Max retries reached, exiting: ${err.message}`);
      }
    });
}

makeRequest(); // call the makeRequest function to start the retry loop
```

* The code starts by defining the API endpoint URL and the request options. It then sets up a function `makeRequest()` that uses `fetch()` to make a POST request to the API.
* If the response status code is `429 (Too Many Requests)`, the code checks for a `Retry-After` header in the response.
* If the header is present, the code retries the request after the number of seconds specified in the header.
* If the header is not present, the code generates a random retry time between 1 and 250ms and retries the request after that time.
* If the response status code is not `429` and is not `OK`, the code throws an error.
* If the response is `OK`, the code returns the response JSON. If there is an error, the code catches the error and retries the request if the number of retries is less than the maximum number of retries.
* If the number of retries is equal to the maximum number of retries, the code logs an error message and exits.

By implementing retries in your Alchemy application, you can help ensure that your application can handle errors and continue to function reliably even in the face of unexpected errors and network disruptions.

## getLargestAccounts

POST https://{solana-mainnet}.g.alchemy.com/v2/{apiKey}

Returns the 20 largest accounts, by lamport balance (results may be cached up to two hours).

import sdk from '@api/alchemy-docs';

sdk.getLargestAccounts({ id: 1, jsonrpc: '2.0', method: 'getLargestAccounts' }, {apiKey: 'docs-demo'}) .then(({ data }) => console.log(data)) .catch(err => console.error(err));

```
import sdk from '@api/alchemy-docs';

sdk.getLargestAccounts({
  id: 1,
  jsonrpc: '2.0',
  method: 'getLargestAccounts'
}, {apiKey: 'docs-demo'})
  .then(({ data }) => console.log(data))
  .catch(err => console.error(err));
```

## getTokenAccountBalance

POST https://{network}.g.alchemy.com/v2/{apiKey}

Returns the token balance of an SPL Token account..

#### Parameters

***

* `<base-58 encoded string>` - Pubkey of queried token account
* `<object>` - (optional) Config object:
  * `commitment:` \<object> - (optional) Configures the commitment level of the blocks queried\
    Accepts one of the following strings: \[`"finalized"`, `"confirmed"`, `"processed"]`\
    For more info, refer to this [doc](https://docs.solana.com/developing/clients/jsonrpc-api#configuring-state-commitment).

***

#### Results

***

* `amount:` \<u64 string> - the raw balance without decimals
* `decimals:` \<u8> - number of base 10 digits to the right of the decimal place
* `uiAmountString: <string>` - the balance as a string, using mint-prescribed decimals

***

#### Example

***

**Request**

***

cURL

```curl
curl --location --request POST 'https://solana-mainnet.g.alchemy.com/v2/alch-demo/' \
--header 'Content-Type: application/json' \
--data-raw '{
    "method": "getTokenAccountBalance",
    "jsonrpc": "2.0",
    "params": [
        "3Lz6rCrXdLybFiuJGJnEjv6Z2XtCh5n4proPGP2aBkA1"
    ],
    "id": "017a141e-9a15-4ce3-b039-865e7dc7da00"
}'
```

***

**Response**

***

JavaScript

```javascript
{
    "jsonrpc": "2.0",
    "result": {
        "context": {
            "slot": 137567036
        },
        "value": {
            "amount": "301922375078",
            "decimals": 6,
            "uiAmount": 301922.375078,
            "uiAmountString": "301922.375078"
        }
    },
    "id": "017a141e-9a15-4ce3-b039-865e7dc7da00"
}
```

```
// project example

import sdk from '@api/alchemy-docs';

sdk.getTokenAccountBalance({
  id: 1,
  jsonrpc: '2.0',
  method: 'getTokenAccountBalance'
}, {apiKey: 'docs-demo'})
  .then(({ data }) => console.log(data))
  .catch(err => console.error(err));
```

## getBalance

POST https://{network}.g.alchemy.com/v2/{apiKey}

Returns the balance of the account of provided Pubkey.

#### Parameters

* `<base-58 encoded string>` - Pubkey of account to query
* `<object>` - (optional) Config object:
  * `commitment:` (optional) \<string> - Configures the commitment level of the blocks queried\
    Accepts one of the following strings: \[`"finalized"`, `"confirmed"`, `"processed"]`\
    For more info, refer to this [doc](https://docs.solana.com/developing/clients/jsonrpc-api#configuring-state-commitment).
  * `minContextSlot:` (optional) \<number> - set the minimum slot that the request can be evaluated at.

#### Result

* `RpcResponse:`\<u64> - RpcResponse JSON object with `value` field set to the balance

#### Example

**Request**

cURL

```curl
curl --location --request POST 'https://solana-mainnet.g.alchemy.com/v2/demo' \
--header 'Content-Type: application/json' \
--data-raw '  {
    "jsonrpc": "2.0",
    "id": 1,
    "method":"getBalance", 
    "params":
    [
    "83astBRguLMdt2h5U1Tpdq5tjFoJ6noeGwaY3mDLVcri"
    ]
  }'
```

**Response**

JavaScript

```javascript
{
  "jsonrpc": "2.0",
  "result": { "context": { "slot": 1 }, "value": 0 },
  "id": 1
}
```

```
// project example

import sdk from '@api/alchemy-docs';

sdk.getBalance({
  id: 1,
  jsonrpc: '2.0',
  method: 'getBalance'
}, {apiKey: 'docs-demo'})
  .then(({ data }) => console.log(data))
  .catch(err => console.error(err));
```

## getTokenAccountsByOwner

POST https://{network}.g.alchemy.com/v2/{apiKey}

Returns all SPL Token accounts by token owner.

#### Parameters

***

* \<base-58 encoded string> - Pubkey of queried SPL token account owner
* `<object>` - Either:
  *   `mint:` \<base-58 encoded string> - Pubkey of the specific token Mint to limit accounts to

      OR
  * `programId:`\<base-58 encoded string> - Pubkey of the Token program that owns the accounts
* `<object>` - (optional) Config object:
  * `commitment:` (optional) Configures the commitment level of the blocks queried\
    Accepts one of the following strings: \[`"finalized"`, `"confirmed"`, `"processed"]`\
    For more info, refer to this [doc](https://docs.solana.com/developing/clients/jsonrpc-api#configuring-state-commitment).
  *   `encoding:` (optional) _\<string>_ - data encoding for each returned transaction

      Accepts one of the following strings:\
      \[`"json"` _(Default)_, `"jsonParsed"`, `"base58"` (_slow_), `"base64"`]\
      `"jsonParsed"` encoding attempts to use program-specific parsers to make the `transaction.message.instructions` list more human-readable; if a parser cannot be found, the instruction falls back to default JSON.
  * `dataSlice:` (optional) \<object> - limits the returned account data using the provided `offset: <usize>` and `length: <usize>` fields; only available for "base58", "base64" or "base64+zstd" encodings.
  * `minContextSlot:` (optional) \<number> - sets the minimum slot that the request can be evaluated at.

***

#### Results

***

* `pubkey:` \<base-58 encoded string> - the account Pubkey
* `account:` \<object> - a JSON object, with the following sub fields:
  * `lamports:` \<u64>, number of lamports assigned to this account
  * `owner:`\<base-58 encoded string>, Pubkey of the program this account has been assigned to
  * `data:` \<object>, Token state data associated with the account, either as encoded binary data or in JSON format `{<program>: <state>}`
  * `executable:` \<bool>, boolean indicating if the account contains a program (and is strictly read-only)
  * `rentEpoch:` \<u64>, the epoch at which this account will next owe rent

***

#### Example

***

**Request**

***

cURL

```curl
curl --location --request POST 'https://solana-mainnet.g.alchemy.com/v2/alch-demo/' \
--header 'Content-Type: application/json' \
--data-raw '{
    "method": "getTokenAccountsByOwner",
    "id": 1,
    "jsonrpc": "2.0",
    "params": [
        "J27ma1MPBRvmPJxLqBqQGNECMXDm9L6abFa4duKiPosa",
        {
            "mint": "2FPyTwcZLUg1MDrwsyoP4D6s1tM7hAkHYRjkNb5w6Pxk"
        },
        {
            "encoding": "jsonParsed"
        }
    ]
}'
```

***

#### Response

***

JavaScript

```javascript
{
    "jsonrpc": "2.0",
    "result": {
        "context": {
            "slot": 137568828
        },
        "value": [
            {
                "account": {
                    "data": {
                        "parsed": {
                            "info": {
                                "isNative": false,
                                "mint": "2FPyTwcZLUg1MDrwsyoP4D6s1tM7hAkHYRjkNb5w6Pxk",
                                "owner": "J27ma1MPBRvmPJxLqBqQGNECMXDm9L6abFa4duKiPosa",
                                "state": "initialized",
                                "tokenAmount": {
                                    "amount": "821",
                                    "decimals": 6,
                                    "uiAmount": 8.21E-4,
                                    "uiAmountString": "0.000821"
                                }
                            },
                            "type": "account"
                        },
                        "program": "spl-token",
                        "space": 165
                    },
                    "executable": false,
                    "lamports": 2039280,
                    "owner": "TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA",
                    "rentEpoch": 318
                },
                "pubkey": "Exo9AH6fNchE43GaJB85FT7ToYiuKnKzYDyW5mFeTXRR"
            }
        ]
    },
    "id": 1
}
```

```
// project example

import sdk from '@api/alchemy-docs';

sdk.getTokenAccountsByOwner({
  id: 1,
  jsonrpc: '2.0',
  method: 'getTokenAccountsByOwner'
}, {apiKey: 'docs-demo'})
  .then(({ data }) => console.log(data))
  .catch(err => console.error(err));
```

## getRecentPrioritizationFees

POST https://{network}.g.alchemy.com/v2/{apiKey}

Returns a list of prioritization fees from recent blocks. Currently, a node's prioritization-fee cache stores data from up to 150 blocks.

```
// project example

import sdk from '@api/alchemy-docs';

sdk.getRecentPrioritizationFees({
  id: 1,
  jsonrpc: '2.0',
  method: 'getRecentPrioritizationFees'
}, {apiKey: 'docs-demo'})
  .then(({ data }) => console.log(data))
  .catch(err => console.error(err));
```

## getBlockHeight

POST https://{network}.g.alchemy.com/v2/{apiKey}

Returns the current block height of the node.

```
// project example

import sdk from '@api/alchemy-docs';

sdk.getBlockHeight({
  id: 1,
  jsonrpc: '2.0',
  method: 'getBlockHeight'
}, {apiKey: 'docs-demo'})
  .then(({ data }) => console.log(data))
  .catch(err => console.error(err));
```

## getSignatureStatuses

POSThttps://{network}.g.alchemy.com/v2/{apiKey}

Returns the statuses of a list of signatures.

#### Parameters

***

* \<array of base-58 encoded string> - An array of transaction signatures to confirm
* `<object>` - (optional) Configuration object containing the following field:
  * `searchTransactionHistory:` \<bool> - if true, a Solana node will search its ledger cache for any signatures not found in the recent status cache

***

#### Results

***

**Known Tx**

***

* `<object>`
  * `slot:` \<u64> - The slot the transaction was processed
  * `confirmations:` \<usize | null> - Number of blocks since signature confirmation, null if rooted, as well as finalized by a supermajority of the cluster
  * `err:` \<object | null> - Error if transaction failed, null if transaction succeeded.
  * `confirmationStatus:` \<string | null> - The transaction's cluster confirmation status; either `processed`, `confirmed`, or `finalized`. See [Commitment](https://docs.solana.com/developing/clients/jsonrpc-api#configuring-state-commitment) for more on optimistic confirmation.

***

**Unknown Tx**

***

* `<null>` - Unknown transaction

***

#### Example

***

**Request**

***

cURL

```curl
curl --location --request POST 'https://solana-mainnet.g.alchemy.com/v2/alch-demo/' \
--header 'Content-Type: application/json' \
--data-raw '{
    "method": "getSignatureStatuses",
    "jsonrpc": "2.0",
    "params": [
        [
            "28P1gdVq52uEbCHns4EL5DCMjU5PtcBo5M3Gju4FX8DLwjLPDchudttnQapAxYy5dkdVZ6sqa6pvtgC5mbKLqfQA"
        ],
        {
            "searchTransactionHistory": true
        }
    ]
}'
```

***

#### Response

***

JavaScript

```javascript
{
    "jsonrpc": "2.0",
    "result": {
        "context": {
            "slot": 137569378
        },
        "value": [
            {
                "confirmationStatus": "finalized",
                "confirmations": null,
                "err": null,
                "slot": 137529522,
                "status": {
                    "Ok": null
                }
            }
        ]
    }
}
```

```
// project example

import sdk from '@api/alchemy-docs';

sdk.getSignatureStatuses({
  id: 1,
  jsonrpc: '2.0',
  method: 'getSignatureStatuses'
}, {apiKey: 'docs-demo'})
  .then(({ data }) => console.log(data))
  .catch(err => console.error(err));
```

## How to Subscribe to Pending Transactions via WebSocket Endpoints

Learn how to subscribe to pending transactions via WebSockets, and filters the transactions based on specified from and/or to addresses.

In this tutorial, you'll utilize the alchemy\_pendingTransactions subscription type API endpoint. If you require the script or further details, refer to the following articles or continue reading for more.

[![docs.alchemy.com](https://files.readme.io/0c06bc6-small-alchemy-circle-logo.png)docs.alchemy.comalchemy\_pendingTransactions](https://docs.alchemy.com/reference/alchemy-pendingtransactions)

Alchemy provides the most effective method to subscribe to pending transactions, log events, and new blocks using WebSockets on Ethereum, Polygon, Arbitrum, and Optimism. By leveraging the Alchemy SDK, you're able to access direct subscription types by simply connecting to each endpoint.

In this tutorial, we will test and create a sample project using the `alchemy_pendingTransactions` method offered by the Alchemy SDK.

**What relevance does the `alchemy_pendingTransactions` provide to users?**

* Watching for pending transactions sent to a set of NFT owners to track the most recent floor price
* Watching transactions sent from a whale trader to track trading patterns

**How does `alchemy_pendingTransactions` compare to `newPendingTransactions`?**\
Both these subscription types enable developers to receive transaction hashes that are sent to the network and marked as "pending". However, `alchemy_pendingTransactions`enhance the developer experience by providing filters that can specify based on to/from addresses. This greatly improves the readability of the transaction requests received.

It allows for strengthened requests with specific parameters given by the user including:

* `toAddress`(optional): Singular address or array of addresses **to** receive pending transactions sent from this address.
* `fromAddress`(optional): Singular address or array of addresses **from** receive pending transactions sent from this address.
* `hashesOnly`(optional - default set to `false`): The response matches the payload of [eth\_getTransactionByHash](https://docs.alchemy.com/reference/eth-gettransactionbyhash). This is information about a transaction by the transaction hash including `blockHash`, `blockNumber` and `transactionIndex`.

### Step 0: Configure your developer environment

Before you begin, complete the following steps to [set up your web3 developer environment](https://docs.alchemy.com/docs/how-to-set-up-core-web3-developer-tools).

1\. Install [Node.js](https://nodejs.org/en/)(> 14) on your local machine

2\. Install [npm](https://www.npmjs.com/) on your local machine

3\. Install [wscat](https://www.npmjs.com/package/wscat) on your local machine

To check your Node version, run the following command in your terminal:

Bash

```shell
node -v
```

### Step 1: Open your Alchemy App

Once your Alchemy account is created, there will also be a default app that is also created.

To create another Alchemy app, check out [this video](https://www.youtube.com/watch?time\_continue=1\&v=tfggWxfG9o0\&feature=emb\_logo).

### Step 2: Get WebSocket URL from Alchemy App

Once you have created your app, get your WebSocket URL that we will use later in this tutorial.

1. Click on your app's **View Key** button in the dashboard
2. Copy and save the **WebSocket URL**

### Step 3: Output Pending Transactions Using wscat

**Wscat** is a terminal or shell tool used to connect to the WebSockets server. Each Alchemy application will provide a WebSocket URL that can be used directly with the wscat command.

1. Initiate the WebSocket stream
2. Enter the specific call command

From your terminal, run the following commands:

wscat

```shell
// initiate websocket stream first
wscat -c wss://eth-mainnet.g.alchemy.com/v2/demo

// then call subscription 
{"jsonrpc":"2.0","id": 2, "method": "eth_subscribe", "params": ["alchemy_pendingTransactions", {"toAddress": ["0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48", "0xdAC17F958D2ee523a2206206994597C13D831ec7"], "hashesOnly": false}]}
```

If successful, you should see output that looks something like this:

Results

```json
{"id":1,"result":"0xf13f7073ddef66a8c1b0c9c9f0e543c3","jsonrpc":"2.0"}

{
  "jsonrpc": "2.0",
  "method": "eth_subscription",
  "params": {
    "result": {
      "blockHash": null,
      "blockNumber": null,
      "from": "0x098bdcdc84ab11a57b7c156557dca8cef853523d",
      "gas": "0x1284a",
      "gasPrice": "0x6fc23ac00",
      "hash": "0x10466101bd8979f3dcba18eb72155be87bdcd4962527d97c84ad93fc4ad5d461",
      "input": "0xa9059cbb00000000000000000000000054406f1ec84f89532f83768f3f159b73b237257f0000000000000000000000000000000000000000000000000000000001c9c380",
      "nonce": "0x11",
      "to": "0xdac17f958d2ee523a2206206994597c13d831ec7",
      "transactionIndex": null,
      "value": "0x0",
      "type": "0x0",
      "v": "0x26",
      "r": "0x93ddd646056f365352f7e53dfe5dc81bde53f5b7c7bbe5deea555a62540d6995",
      "s": "0x79ed82a681930feb11eb68feccd1df2e53e1b96cf9171ae4ffcf53e9b2a40e8e"
    },
    "subscription": "0xf13f7073ddef66a8c1b0c9c9f0e543c3"
  }
}
```

By using **wscat**, you are able to verify the transaction immediately via the computer's terminal or shell.

### Step 4: Create a Node project

To build out a kick starter that leverages the **alchemy-sdk**, let's create an empty repository and install all node dependencies.

To make requests to the SDK WebSockets Endpoints, we recommend using the [Alchemy SDK Quickstart](https://docs.alchemy.com/reference/sdk-websockets-endpoints).

From your terminal, run the following commands:

Alchemy SDK

```shell
mkdir pending-transactions && cd pending-transactions
npm init -y
npm install --save alchemy-sdk
touch main.js
```

This will create a repository named `pending-transactions` that holds all the files and dependencies we need.

Open this repo in your preferred code editor, where we'll write our code in the `main.js` file.

### Step 5: Output Pending Transactions using alchemy-sdk

Next, we’ll demonstrate how to use the Alchemy SDK to create an **alchemy\_pendingTransactions** subscription.

To make requests using Alchemy's pendingTransactions API, we recommend reviewing the [alchemy\_pendingTransactions docs](https://docs.alchemy.com/reference/alchemy-pendingtransactions).

Next, add the following code to the `main.js` file, using your Alchemy API key:

Alchemy SDK

```javascript
// Installation: npm install alchemy-sdk
import { Alchemy, Network, AlchemySubscription } from "alchemy-sdk";

const settings = {
  apiKey: "<-- ALCHEMY APP API KEY -->", // Replace with your Alchemy API Key
  network: Network.ETH_MAINNET, // Replace with your network
};

const alchemy = new Alchemy(settings);

// Subscription for Alchemy's pendingTransactions API
alchemy.ws.on(
  {
    method: AlchemySubscription.PENDING_TRANSACTIONS,
  },
  (tx) => console.log(tx)
);
```

Run this script by running the following command in your terminal:

`node main.js`

If successful, you should see a stream of transactions as the result. This stream of output indicates the latest (pending or mined) transactions hitting the Ethereum Mainnet. It should look something like this:

Results

```json
{
  blockHash: null,
  blockNumber: null,
  from: '0x0dd25571522e0ac38712b56834dc6081cde33325',
  gas: '0x8b5d7',
  gasPrice: '0xd09dc30c',
  hash: '0x9575c90c4923d7d1981029bfcfc3f23b91ec78683b881b571763dff0c00a72da',
  input: '0x0357371d0000000000000000000000000dd25571522e0ac38712b56834dc6081cde3332500000000000000000000000000000000000000000000000000b1a2bc2ec50000',
  nonce: '0x14c',
  to: '0xfebfd3467c4362eee971c433e3613c009ab55ce4',
  transactionIndex: null,
  value: '0x0',
  type: '0x0',
  v: '0x27125',
  r: '0xdc427da8df7cd9a4e831734a9ec4d8127d2c068497e667138b0092635157c5db',
  s: '0x5d4b996fd467702e7602030de3517e024a31085d57cfe2ff064e98460bc2da28'
}
{
  blockHash: null,
  blockNumber: null,
  from: '0x61141bce5352fc9b5ff648468676e356518d86ab',
  gas: '0x55730',
  gasPrice: '0x77359410',
  hash: '0xc81a148daba8bb67daca8b3e645d07c9caaf2977ebdd46245f97f12cc3737dd2',
  input: '0x29dd214d00000000000000000000000000000000000000000000000000000000000000c00000000000000000000000000000000000000000000000000000000000000005000000000000000000000000af28cb0d9e045170e1642321b964740784e7dc640000000000000000000000000000000000000000000000000ddf17ab47bc33b40000000000000000000000000000000000000000000000000000000000000100ee569c3895715793e17640229cd2462886335a0940e299921bb2eef69618bb730000000000000000000000000000000000000000000000000000000000000014805fe47d1fe7d86496753bb4b36206953c1ae660000000000000000000000000000000000000000000000000000000000000000000000000000000000000016065f49bd49de252a7f0d9100776c70f0da398368ef9866f8e21fbb0e3e630e74f00000000000000000000000090294dca1861b78d8aaa4acd1abf4c9b535c490e000000000000000000000000cc7bb2d219a0fc08033e130629c2b854b7ba919500000000000000000000000000000000000000000000000029a2241af62c00000000000000000000000000000000000000000000000000000000000000000120000000000000000000000000000080383847bd75f91c168269aa74004877592f000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001490294dca1861b78d8aaa4acd1abf4c9b535c490e000000000000000000000000',
  nonce: '0x3f927',
  to: '0x000054d3a0bc83ec7808f52fcdc28a96c89f6c5c',
  transactionIndex: null,
  value: '0x0',
  type: '0x0',
  v: '0x27125',
  r: '0x5466a7a7d1823c2290fc4803cac0340dcab34e843a8c0681763ca60fd7ccc7b2',
  s: '0x3030ce868c4d09b71009af597175bdecace2f081a0f78f0e4705e318d19076a9'
}
....
```

### Step 6: Filter Pending Transactions

Next, we’ll demonstrate how to filter pending transactions using the **alchemy\_pendingTransactions** subscription based on an address or array of addresses to receive pending transactions sent **from** this address (`fromAddress`) and **to** this address (`toAddress`).

Add the following code to the `main.js` file, using your Alchemy API key:

> ### 🚧Notice new filters
>
> Within the Subscription API method, we've added two new filters fromAddress and toAddress. This will enable our request to the Alchemy pendingTransaction API to be filtered based on the parameters that we've assigned.

Alchemy SDK

```javascript
// Installation: npm install alchemy-sdk
import { Alchemy, Network, AlchemySubscription } from "alchemy-sdk";

const settings = {
  apiKey: "<-- ALCHEMY APP API KEY -->", // Replace with your Alchemy API Key
  network: Network.ETH_MAINNET, // Replace with your network
};

const alchemy = new Alchemy(settings);

// Subscription for Alchemy's pendingTransactions API
alchemy.ws.on(
  {
    method: AlchemySubscription.PENDING_TRANSACTIONS,
    fromAddress: "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48", // Replace with address to recieve pending transactions from this address
    toAddress: "0xdAC17F958D2ee523a2206206994597C13D831ec7", // Replace with address to send  pending transactions to this address
  },
  (tx) => console.log(tx)
);
```

## How to Use Custom Webhooks for Web3 Data Ingestion

Learn How to Stream Web3 Data to Your Dapp with Custom Webhooks

[Custom Webhooks](https://www.alchemy.com/notify/custom-webhooks) are the latest additions to Alchemy’s Notify suite of web3 notification services.

Powered by Alchemy’s Supernode and GraphQL, they bring custom real-time notifications for any blockchain activity or contract-based event.

This article will cover an [introduction to Custom Webhooks](https://docs.alchemy.com/reference/custom-webhooks-quickstart#introduction), the benefits of using them, how to set up an Express.js server, and three examples of how Custom Webhooks can be used to track on-chain data.

If you’re familiar with data streaming, you can skip to any of the following tutorials:

1. [How to Stream New Block Data with Custom Webhooks](https://docs.alchemy.com/docs/how-to-use-custom-webhooks-for-web3-data-ingestion#how-to-stream-new-block-data-with-custom-webhooks)
2. [How to Implement a Custom Webhook for Created Contracts](https://docs.alchemy.com/docs/how-to-use-custom-webhooks-for-web3-data-ingestion#how-to-implement-a-custom-webhook-for-created-contracts)
3. [How to Track Block Transactions with Custom Webhooks](https://docs.alchemy.com/docs/how-to-use-custom-webhooks-for-web3-data-ingestion#how-to-track-block-transactions-with-custom-webhooks)

For more [ways to use Custom Webhooks](https://docs.alchemy.com/reference/custom-webhooks-example), explore our library of examples!

### What is web3 data ingestion?

Web3 data ingestion is the process of collecting and importing data from various sources in the web3 ecosystem into a system for storage and analysis.

#### What is the problem with web3 data ingestion?

Web3 data ingestion has several major problem areas, including large and varied data, different data formats, and lack of scalability.

These problems have arisen because of the more than 1 million smart contracts deployed on the Ethereum mainnet alone.

The amount of information that has been recorded on the blockchain has increased exponentially during recent years.

Here are three major concerns in more detail:

**1. Large and Varied Data**

Firstly, the amount of data generated by the different blockchains are often **large and varied**, making it difficult to manage and process.

**2. Different Data Formats**

Secondly, the **data may be in different formats**, making it challenging to normalize and integrate.

**3. Lack of Scalability**

Finally, the **process of web3 data ingestion needs to be efficient and scalable** to keep up with the volume and speed of incoming data.

Presently, web3 data ingestion processing is not too scalable.

All these factors make data ingestion a complex and challenging problem for web3 applications.

#### Why is data ingestion an important product for web3 developers?

Data ingestion is important for web3 because decisions in web3 are time-sensitive, and the process of data handling needs to be scalable.

**1. Web3 is Time Sensitive**

In the context of web3, proper data ingestion is crucial for making time-sensitive decisions in web3 products.

Users need to stay informed about the latest price changes, token mints, the on-chain activity of a specific wallet, the latest NFT drop or track the on-chain activity of a whale’s wallet.

**2. Web3 Products Require Scalability**

Proper data ingestion can lead to efficient and scalable web3 products that can improve with time.

However, improper data handling could result in various problems, ranging from poor performance due to information overload and scalability to critical problems like wrong data filtering.

**3. Complicated Handling of Blockchain Data**

Usually, blockchain data can be tracked and recorded with event listeners that can be encoded into the decentralized application.

However, this may get complicated and time-consuming for the developer, especially if the dapp needs to listen to multiple events from multiple smart contracts or even multiple blockchains.

This is where [webhooks](https://www.alchemy.com/overviews/what-is-a-webhook) come into play.

### What are Custom Webhooks?

Alchemy’s Custom Webhooks are the latest addition to Alchemy’s [web3 notification tool suite](https://www.alchemy.com/notify).

Alchemy’s query customization options make them superior to the existing notification solutions, which are based on a set of pre-defined triggers.

#### Why use Custom Webhooks?

Custom Webhooks provide a host of benefits for development. Here are some in more detail:

**1. Precise Information Querying**

Powered by Alchemy’s Supernode and GraphQL, Custom Webhooks allow for precise information querying so that the **application receives only the information that is required by its business logic.**

**2. Cost-Effectiveness**

Additionally, **Custom Webhooks reduce data payload and computational resources**, which makes them a more cost-effective method.

**3. Ease of Integration**

Lastly, compared to in-dapp event listeners, Custom Webhooks are way easier to integrate and provide the data in ready to digest way, which immensely contributes to the developer experience.

### How to Setup an Express.js Server for Custom Webhooks

For this tutorial, a basic Express.js server will be created.

It will have a separate endpoint for each of the webhooks.

#### Prerequisites

The following list of tools is required:

* Integrated Development Environment (IDE)
* Package Manager (NPM)
* Node.js
* Express.js for server creation
* Ngrok for exposing the localhost server
* An Alchemy Account
* An Alchemy Custom Webhook

**1. Configure the Development Environment**

Firstly, create a folder for the project.

Open the folder with VSCode, then open a new terminal inside and type the following command to initialize an **npm** project:

`npm init -y`

***

The **-y** flag prefills all the required information with its default values and speeds up the configuration process.

Then, install **express** by running:

`npm i express`

***

**Ngrok** will also be needed to expose the local Express server.

1. Sign-up for a free Ngrok account.
2. Install Ngrok using [the Ngrok guide](https://ngrok.com/docs/getting-started/). On macOS, run `brew install ngrok`
3. Connect the Ngrok account by running ngrok authtoken `YOUR_AUTH_TOKEN`

**2. Configure the Server**

The following code creates a basic Express server with separate endpoints for each of the custom hooks that will be created in the tutorial.

The information received by the webhooks is stored in three separate arrays in the memory of the server.

Create an **index.js** file in the root of the project’s directory and paste the following code inside it:

JSX

```jsx
// Import express (after running 'npm install express').
const express = require("express");
const app = express();

// Configure server. 
const PORT = 8080;

// Decode JSON data.
app.use(express.json({ limit: "25mb" }));
let blocks = [];
let createdContracts = [];
let transactionLog = [];

// Create a route for the app.
app.get("/", (req, res) => {
    res.send(
        "Welchome to Alchemy Blockchain Information Fetcher. Access the designated endpoints to retrieve the desired information."
    );
});

// Create a POST block-info route. Use as a webhook URL.
app.post("/block-info", (req, res) => {
    const blockInfo = JSON.stringify(req.body.event.data.block);
    blocks.push(blockInfo);

    // Respond with status 200.
    res.send("Success.");
})

// Create a GET block-info route.
app.get("/block-info", (req, res) => {
    res.send(blocks);
});

// Create a POST created-contracts route. Use as a webhook URL.
app.post("/created-contracts", (req, res) => {
    const createdContractsInfo = req.body.event.data.block.logs;

    // Loop over the received object of block transactions 
    // save all transactions in which a contract has been created. 
    for (let transaction of createdContractsInfo) {
        if (transaction.transaction.createdContract) {
            createdContracts.push(transaction);
        }
    }

    // Respond with status 200. 
    res.send("Success.");
});

// Create a GET created-contracts route.
app.get("/created-contracts", (req, res) => {
    res.send(createdContracts);
});

// Create a POST transactions route. Use as a webhook URL.
app.post("/transactions", (req, res) => {
    const transactions = req.body.event.data.block;
    transactionLog.push(transactions);

    // Respond with status 200.
    res.send("Success.");
})

// Create a GET transactions route.
app.get("/transactions", (req, res) => {
    res.send(transactionLog);
});

// Make the server listen to requests.
app.listen(PORT, () => {
    console.log(`Server running at: http://localhost:${PORT}/`);
})
```

**3. Start the Server**

Next, type this in the terminal:

`node index.js`

***

The server should be started at a particular port as shown below:

![](https://files.readme.io/10a0c2d-image.png)

Lastly, open another terminal and start **ngrok** with the following command:

`ngrok http PORT_NUMBER_RETURNED_BY_EXPRESS`

***

![](https://files.readme.io/8edc87e-image.png)

Now that the server and environment are configured and running, the next step is to connect Alchemy’s Custom Webhook with the server.

### How to Stream New Block Data with Custom Webhooks

Using Alchemy’s Custom Webhooks, obtaining block information is simple.

The following steps will implement a simple custom webhook with Alchemy:

#### 1. Sign Up with Alchemy

Firstly, head over to [Alchemy](https://www.alchemy.com/), create a free account, and sign in.

#### 2. Create Webhook using GraphQL

Next, navigate to the [Notify](https://dashboard.alchemy.com/notify) dashboard.

At the GraphQL section, click **+ Create Webhook.**

The page will redirect to the GraphQL query playground, where the webhook can be fine-tuned to fetch specific data from the blockchain.

> ### 📘
>
> For the simplification of the tutorial, webhook verification is not implemented. Always verify the webhooks that interact with your decentralized application. Click [here](https://docs.alchemy.com/reference/notify-api-quickstart#find-your-signing-key) to learn how to do that.

The entry point for all EVM data is the block. This is why every query starts with it.

To get the general information for every upcoming block, paste this code into the query editor:

JSX

```jsx
{
  block {
    hash,
    number,
    timestamp
  }
}
```

***

The code is a GraphQL query that asks for each produced block's hash, number, and timestamp.

#### 3. Configure the Webhook URL

Next, configure the webhook URL.

This is the address through which the webhook will submit the queried information to the server.

In the VSCode terminal, used to initialize **ngrok**, find and copy the forwarding link.

![](https://files.readme.io/1440f66-image.png)

Paste the forwarding link in the **Webhook URL** input field in the GraphQL section of the Notify dashboard.

Then, add **/block-info** at the end.

Now, the URL will point to the specific entry point in the server created for this webhook.

![](https://files.readme.io/6dcfae6-image.png)

Click on the **Create Webhook** button in the lower-left corner.

The **Chain** and **Networks** field can be configured to fetch data from other blockchains like [Arbitrum](https://www.alchemy.com/overviews/arbitrum-webhooks), Polygon, and [Optimism](https://www.alchemy.com/overviews/optimism-webhooks), but for this tutorial, the Ethereum mainnet will be used.

The webhook is now set up and will notify the server with the queried information via a POST request whenever a new block is added to the blockchain.

#### 4. Verify Retrieval of Data

Head over to the server to check the retrieved data.

In a new tab of an Internet browser, paste this URL:

`http://localhost:YOUR_PORT_NUMBER/block-info`

![](https://files.readme.io/a242c01-image.png)

The server will return information about every upcoming block.

### How to Implement a Custom Webhook for Created Contracts

Implementing a Custom Webhook is simple with Alchemy. In this guide, the webhook will fetch all the transactions included in the block.

If a contract has been created in one of them, the contract’s address is recorded and stored.

#### 1. Create a Webhook for Created Contracts using GraphQL

Assuming you already have an Alchemy account, navigate to the Notify dashboard.

At the GraphQL section, click **+ Create Webhook.**

The page will redirect to the GraphQL query playground, where the webhook can be fine-tuned to fetch specific data from the blockchain.

In the Custom Webhooks GraphQL playground, paste the following query:

JSX

```jsx
{
  block {
    logs(filter: {addresses: [], topics: []}) { 
      transaction {
        createdContract {
          address
        }
      }
    }
  }
}
```

***

The query checks every transaction in the block, checks if it is associated with the creation of a smart contract, and returns its address.

If the transaction is not associated with creating a contract, the value will be **null**.

#### 2. Configure the Webhook URL

In the webhook URL, copy the forwarding link, generated by **ngrok** and add **/created-contracts** at the end.

Click on the **Create Webhook** button in the lower-left corner.

#### 3. Verify the Created Contract Data

Next, head over to the server to check the retrieved data.

In a new tab of an Internet browser, paste this URL:

`http://localhost:YOUR_PORT_NUMBER/created-contracts`

The contracts created should be shown as below:

![](https://files.readme.io/969c00d-image.png)

If contracts are created in each upcoming block, their addresses will be recorded and returned by the server.

### How to Track Block Transactions with Custom Webhooks

This example creates a webhook that fetches all the transactions included in a block.

More specifically, it includes the number of transactions as well as an array of transaction objects.

#### 1. Create Webhook using GraphQL

Go back to the Notify dashboard, and in the GraphQL section, click **+ Create Webhook.**

The page will redirect to the GraphQL query playground, where the webhook can be fine-tuned to fetch specific data from the blockchain.

In the Custom Webhooks GraphQL playground, paste the query below:

JSX

```jsx
{
  block {
    transactionCount,
    transactions {
       hash,
        nonce,
        index,
        from {
          address
        },
        to {
					address
        },
        value,
        gasPrice,
        maxFeePerGas,
        maxPriorityFeePerGas,
        gas,
        status,
        gasUsed,
        cumulativeGasUsed,
        effectiveGasPrice,
        createdContract {
          address
        }
    }
  }
}
```

***

#### 2. Configure the Webhook URL

In the webhook URL, copy the forwarding link, generated by **ngrok** and add **/transactions** at the end.

Click on the **Create Webhook** button in the lower-left corner.

#### 3. Verify Retrieval of Data

Next, head over to the server to check the retrieved data.

In a new tab of an Internet browser, paste this URL:

`http://localhost:YOUR_PORT_NUMBER/transactions`

This will visualize the transactions in the following format:

![](https://files.readme.io/5693fdd-image.png)

Alchemy’s Custom Webhooks provide custom-tailored webhooks that supply only the information that a dapp would use. They are reliable, instant, and require only a few clicks to integrate.

With Custom Webhooks, end users can make **well-informed and time-sensitive decisions** about the latest NFT drop or track the on-chain activity of a whale’s wallet.

Read the FAQs for Custom Webhooks for more information, and [sign up](https://alchemyapi.typeform.com/to/S78PmEqB?typeform-source=www.alchemy.com) to start integrating Custom Webhooks into your dapp today, and start utilizing the power of real-time web3 data ingestion.

## How to Stream Blockchain Data to AWS with Custom Webhooks

Build a serverless web3 data ingestion API in 10 minutes using AWS, DynamoDB, and Custom Webhooks.

### Introduction

Accessing blockchain data can be computationally heavy, time-consuming, and costly. This can make it difficult for developers and applications to access real-time block information without complex infrastructures or expensive bills.

Fortunately, [Alchemy Custom Webhooks](https://www.alchemy.com/notify/custom-webhooks) can help. They allow you to track any smart contract or marketplace activity, and monitor any contract creation, or any other on-chain interaction in real-time using handy GraphQL queries. This gives developers infinite real-time and historical blockchain data access with precise filter controls.

Custom webhooks enable developers to access blockchain data in real-time using the power of GraphQL filters, Supernode, and single endpoint APIs. These webhooks process the data coming from the blocks and send it to a given API endpoint, supporting POST requests.

Developers can then store blockchain data on any database to power their applications.

In this tutorial, you’re going to learn how to build a serverless API to ingest, store and retrieve real-time blockchain data to power your blockchain dashboards, decentralized exchange, NFT marketplace, or literally any application that might need to fetch blocks data in real-time.

![](https://files.readme.io/5639bfb-small-Untitled\_Drawing-hackerdraw\_5.png)

> ### 📘Note:
>
> **Serverless**: A cloud computing model where you can focus on writing code for your applications without worrying about server management.
>
> Companies like AWS, Azure, and Google Cloud offer serverless services, such as AWS Lambda, Azure Functions, and Google Cloud Functions.
>
> These services automatically manage the server infrastructure, scaling it as needed, so that developers can focus on building and deploying applications quickly and easily. You can read more about serverless infrastructures on the red hat website.

You will learn how to use the Alchemy custom webhooks to get real-time blockchain data, AWS lambda functions to power your backend, how to store the data in a Dynamo database, and implement AWS API gateway to create the endpoint we’ll use to store and query the blocks data.

To develop our API, we’ll be using:

* **Alchemy Custom Webhooks** to retrieve real-time blockchain data
  * If you don’t already have an Alchemy account, you can [create one completely for free](https://dashboard.alchemy.com/signup/?a=aws-custom-webhooks).
* **AWS Lambda** to process blockchain data
  * You can learn how to create a new AWS account using their [creating an AWS account guide](https://docs.aws.amazon.com/accounts/latest/reference/manage-acct-creating.html).
* **AWS API Gateway** to manage our API
* **Dynamo DB** to store the received data
* **AWS Cloud formation and SAM** to create our serverless infrastructure in less than 5 minutes.
  * Refer to Amazon's guide to [install or update the latest version of the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).

Don’t worry if you don’t recognize any of these names (except Alchemy, shame on you if you don’t know about Alchemy). You’ll learn how every and each of those works in the following sections.

Before, let’s briefly take a look at the prerequisites.

### Prerequisities

To create a serverless blockchain data ingesting architecture in 5 minutes, you’ll need:

* A free Alchemy account
* An AWS Account
* AWS CLI installed and logged in

To verify that AWS has been installed correctly on your device, or if it has already been installed, run the following command in your terminal: `which was`

bash

```yaml
which aws
```

Once you have finished creating your accounts and installing the AWS CLI, you will be all set to start creating your first serverless blockchain data-ingesting and querying API in just 5 minutes.

To do this, we will use SAM, an application model by AWS that allows you to spin up serverless infrastructure quickly.

Let's see how. 👀\\

### Step 1: Install SAM (AWS Serverless Applications Model)

SAM provides a faster way to spin up serverless applications on AWS by using a Lambda-like execution environment that enables you to locally build, test, and debug your applications. Additionally, it offers templates to quickly create serverless applications.

Using SAM, you can define the application infrastructure you want and model it using YAML with just a few lines per resource. During deployment, SAM automatically transforms and expands the SAM syntax into AWS CloudFormation syntax, allowing you to build serverless applications quickly.

You can learn more about [AWS Cloud Formation on the AWS docs.](https://aws.amazon.com/cloudformation/)

First of all, navigate to [t](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html)he [SAM installation page](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html) and select the GUI Package relative to your OS.

In this tutorial, we’ll use Mac OS but everything applies to Windows and Linux as well:

To get started building SAM-based applications, you'll need to install the SAM CLI on your device.

**Follow these steps:**

* Navigate to the [SAM installation page](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html) and select the GUI package that corresponds to your operating system.
* In this tutorial, we'll use Mac OS, but the process applies to Windows and Linux as well

![AWS SAM Installation guide webpage](https://files.readme.io/9219b9a-small-Screenshot\_2023-05-09\_at\_5.36.58\_PM.png)

Once you’ll have installed the package, verify the installation by running the following command in your terminal:

bash

```jsx
$ which sam
/usr/local/bin/sam
$ sam --version
SAM CLI, version 1.66.0
```

With the SAM CLI installed, it is now time to start creating the different components that will power your serverless blockchain data ingesting and querying API.

Let’s start with setting up a new project.

### Step 2: Create a new serverless application using SAM

Thanks to SAM we can now create a new serverless application!

1. Start the process by running the following command in your terminal: `sam init`
2.  SAM will now prompt you with a number of options to select (ie, templates, serverless cloud resources needed, execution environment). Select the following options:

    1. AWS Quick Start Templates, select `1`
    2. Serverless API, select `3`
    3. nodejs18.x, select `3`
    4. Enable [X-Ray tracing](https://aws.amazon.com/xray/), select `n` as we don’t want to incur in any additional fees while following this guide
    5. Enable [Cloud Watch insights](https://aws.amazon.com/cloudwatch/), select `n`
    6. Name your application (name it `alchemy-custom-webhooks-test` to follow along easier)

    **👀 Here’s a view of what your terminal should look like as you go through the steps:**

![](https://files.readme.io/104527b-small-Screenshot\_2023-05-08\_at\_11.23.45\_AM.png)

By selecting the above options, we’re telling SAM to create a new serverless application starting from a template.

More specifically, we are using the “Serverless API” template that contains the specifications to spin up:

1. An API Gateway to access and trigger our Lambda functions
2. Lambda functions that we’ll use to process our data and store it in a DB
3. A dynamo NoSQL DB used to store the data

**Disclaimer:** In this guide, we’ll use DynamoDB, a `NoSQL DB`

After completing the process, SAM will proceed to download the necessary files in a handy project directory.

**👇 Your terminal should now show the following:**

![](https://files.readme.io/f906a5f-small-Screenshot\_2023-05-08\_at\_11.32.58\_AM\_1.png)

4. Once your terminal looks like the above screenshot, navigate to the newly-created folder by running: `cd alchemy-custom-webhooks-test` and then open the project up in an IDE of your choice, we are using VSCode for which the command is `code .`

**🏗️ This is what you should see in your code editor:**

![](https://files.readme.io/7e3ff96-small-Screenshot\_2023-05-08\_at\_12.19.42\_PM.png)

**Let’s quickly take a look at the files contained in our project folder created by SAM:**

* **`_tests_`**: contains all the tests related to the Lambda functions contained in the `src\` folder
* **`events`**: stores the events used to test the Lambda functions locally
* **`src`**: contains our Lambda functions (3 by default)
* **`buildspec.yml`**: contains the instructions used by CloudFormation to create our resources
* **`env.json`**: contains the environment variables accessible from our lambda functions
* **`package.json`:** contains our Lambda dependencies
* **`samconfig.toml`**: project-level configuration file that stores default parameters for SAM commands
* **`template.yaml`:** SAM template that represents the architecture of your serverless application

> ### 📘Note:
>
> Optional: In this tutorial we won’t test our Lambda functions locally, so you can delete the `events` folder.

**By default, the project we’ve just created using SAM and the Serverless API template, declares:**

1. Three Lambda functions to `GET` and `POST`items
2. One API gateway with three endpoints triggering the three lambda functions from above
3. One Dynamo DB instance

For our template to work as a blockchain data-ingesting serverless API, we’ll need to modify **two** main things, the `template.yaml` file as well as the Lambda functions.

Let’s do this in the next step!

### Step 3: Modify the `template.yaml` file

The `template.yaml` file is the core of our Cloud Formation instructions that specifies which resources we want to create, how we will create them, and how they will communicate with each other.

![](https://files.readme.io/ba107ad-small-Screenshot\_2023-05-10\_at\_23.27.34.png)

> ### 📘Note:
>
> For a deep dive into how the template.yaml file works as well as the properties available, [you can take a look at the official documentation](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-specification-resources-and-properties.html).

**To create our serverless blockchain data ingesting and querying API, we will need:**

* One Lambda function that can be used by **Alchemy’s Custom Webhooks** to `POST` the real-time blockchain data and store it on our Dynamo DB
* One Lambda function to `GET` all the blockchain data stored in our DB
* One Lambda function to `GET` the data contained in a specific block queried by hash

SAM has already created almost everything for us.

We’ll keep the same Dynamo DB (will only change the primary key) as well as the API Gateway (will only change the endpoint naming) and the Lambda functions, where we’ll only change some of the code to receive, store, and query the data.

To reflect these changes, we’ll need to modify a couple of things in the `template.yaml` file, let’s see how!

#### Modify the Lambda declarations

Let’s modify the Lambda declarations to reflect the changes mentioned above:

1. In the `template.yaml` file, look for the `getAllItemsFunction` and change its name to `getAllBlocksFunction`
2. Then, modify the function’s: `Handler` ,`Description`, and `API` code as showed in the following block:

YAML

```yaml
getAllBlocksFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: src/handlers/get-all-blocks.getAllBlocksHandler
      Runtime: nodejs18.x
      Architectures:
        - x86_64
      MemorySize: 128
      Timeout: 100
      Description: Http GET request to get all blocks' data stored inside the DynamoDB instance
      Policies:
        # Give Create/Read/Update/Delete Permissions to the SampleTable
        - DynamoDBCrudPolicy:
            TableName: !Ref SampleTable
      Environment:
        Variables:
          # Make table name accessible as environment variable from function code during execution
          SAMPLE_TABLE: !Ref SampleTable
      Events:
        Api:
          Type: Api
          Properties:
            Path: /get-all-blocks
            Method: GET
```

With the above changes, we are instructing CloudFormation to assign a new name to our Lambda function, use a different Lambda handler (which we will match with our Lambda filename when modifying the Lambda function in the `src` directory), and change the API endpoint name for improved semantics. 🛠️

3. Next, look for the `getByIdFunction` and change its name to `getBlockByHashFunction`
4. As we did before, update the `Handler`, `Description`, and `API Path` following the code below:

YAML

```yaml
getBlockByHashFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: src/handlers/get-block-by-hash.getBlockByHashHandler
      Runtime: nodejs18.x
      Architectures:
        - x86_64
      MemorySize: 128
      Timeout: 100
      Description: HTTP get method to retrieve block by hash from the Dynamo DB
      Policies:
        # Give Create/Read/Update/Delete Permissions to the SampleTable
        - DynamoDBCrudPolicy:
            TableName: !Ref SampleTable
      Environment:
        Variables:
          # Make table name accessible as environment variable from function code during execution
          SAMPLE_TABLE: !Ref SampleTable
      Events:
        Api:
          Type: Api
          Properties:
            Path: /get-blocks-by-hash/{hash}
            Method: GET
```

5. Now, look to modify the `putItemFunction` and change its name to `storeBlockDataFunction`
6. Then modify the `Handler`, `Description`, and `API Path` following the code below.

YAML

```yaml
storeBlockDataFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: src/handlers/store-block-data.storeBlockDataHandler
      Runtime: nodejs18.x
      Architectures:
        - x86_64
      MemorySize: 128
      Timeout: 100
      Description: HTTP post method to be called from the Alchemy custom webhooks and store a block to a DynamoDB table.
      Policies:
        # Give Create/Read/Update/Delete Permissions to the SampleTable
        - DynamoDBCrudPolicy:
            TableName: !Ref SampleTable
      Environment:
        Variables:
          # Make table name accessible as environment variable from function code during execution
          SAMPLE_TABLE: !Ref SampleTable
      Events:
        Api:
          Type: Api
          Properties:
            Path: /store-block-data
            Method: POST
```

7. Finally, let's modify our DynamoDB instance by changing the `PrimaryKey` name from `id` to `hash`, and the `ReadCapacityUnits` and `WriteCapacityUnits` from `2` to `5` (remember to save your changes):

YAML

```yaml
SampleTable:
    Type: AWS::Serverless::SimpleTable
    Properties:
      PrimaryKey:
        Name: hash
        Type: String
      ProvisionedThroughput:
        ReadCapacityUnits: 5
        WriteCapacityUnits: 5
```

> ### 📘Note:
>
> You can learn more about [ReadCapacityUnits and WriteCapacityUnits on the official AWS docs.](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadWriteCapacityMode.html)

Compare your [`template.yaml`](https://github.com/alchemyplatform/AWS-serverless-blockchain-data-ingesting-infrastructure/blob/main/template.yaml) file, with the one used in this tutorial, [visiting our Github repo](https://github.com/alchemyplatform/AWS-serverless-blockchain-data-ingesting-infrastructure/blob/main/template.yaml).

With our `template.yaml` file ready, we can modify the Lambda functions to handle blockchain data received from Alchemy’s Custom Webhooks, store it on the DynamoDB instance, and reflect the changes made in the `template.yaml` file.

### Step 4: Modify the Lambda Functions to Store & Retrieve Block Data

The Lambda functions contained in the serverless API template created by SAM already have most of the logic needed to create our blockchain data ingesting infrastructure - sweet!

We will proceed to copy-and-paste the required functions and go through the changes together (feel free to familiarize yourself with the code).

![](https://files.readme.io/44ebbbd-small-Screenshot\_2023-05-10\_at\_23.28.44.png)

1. Navigate to the `src` folder where the Lambda functions’ code is contained, and rename the files as follows to match the changes in the `Handler` properties inside the `template.yaml` file:
   1. Re-name `get-all-items.mjs` to `get-all-blocks.mjs`
   2. Re-name `get-by-id.mjs` to `get-block-by-hash.mjs`
   3. Re-name `put-item.mjs` to `store-block-data.mjs`
2. Next, open the `get-all-blocks.mjs` file and copy-paste the following code:

javascript

```jsx
// Create clients and set shared const values outside of the handler.
// Create a DocumentClient that represents the query to add an item

import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient, ScanCommand } from '@aws-sdk/lib-dynamodb';

const client = new DynamoDBClient({});
const ddbDocClient = DynamoDBDocumentClient.from(client);

// Get the DynamoDB table name from environment variables
const tableName = process.env.SAMPLE_TABLE;

/**
 * A simple example includes a HTTP get method to get all items from a DynamoDB table.
 */
export const getAllBlocksHandler = async (event) => {
    if (event.httpMethod !== 'GET') {
        throw new Error(`get all blocks only accept GET method, you tried: ${event.httpMethod}`);
    }
    // All log statements are written to CloudWatch
    console.info('received:', event);

    // get all items from the table (only first 1MB data, you can use `LastEvaluatedKey` to get the rest of data)
    // https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB/DocumentClient.html#scan-property
    // https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_Scan.html
    var params = {
        TableName : tableName
    };

    try {
        const data = await ddbDocClient.send(new ScanCommand(params));
        var items = data.Items;
    } catch (err) {
        console.log("Error", err);
    }

    const response = {
        statusCode: 200,
        body: JSON.stringify(items)
    };

    // All log statements are written to CloudWatch
    console.info(`response from: ${event.path} statusCode: ${response.statusCode} body: ${response.body}`);
    return response;
}
```

The `getAllBlocksHandler` function retrieves all the blocks saved in our DynamoDB SimpleTable.

It begins by creating a `DynamoDBClient` object, which represents the connection to the database.

Next, it creates a `DynamoDBDocumentClient` from the `DynamoDBClient`.

This provides a higher-level abstraction for working with DynamoDB, making it easier to work with the data.

Before retrieving data, the code gets the name of the DynamoDB table from an environment variable called `SAMPLE_TABLE`. This variable is exposed under each lambda declaration in the `template.yaml` file.

YAML

```yaml
   Environment:
        Variables:
          # Make table name accessible as environment variable from function code during execution
          SAMPLE_TABLE: !Ref SampleTable
```

The main function of this code is **`getAllBlocksHandler`**.

This function is executed when the code receives an HTTP GET request.

If the HTTP method is not GET, an error is thrown.

Otherwise, the code retrieves all items from the DynamoDB table using the **`ScanCommand`** object from the **`DynamoDBDocumentClient`**.

The retrieved items are stored in an array called **`items`** and returned as JSON by the function.

Pay attention that the more blocks' data we will store in our DynamoDB, the heavier will be to fetch all of their data all at once. This endpoint and lambda handler shouldn't be used, as is, in a production environment.

**Now let’s move on to the second function, the "get-block-by-hash" handler.**

Copy and paste the following code:

#### get-block-by-hash.mjs

javascript

```jsx
// Create clients and set shared const values outside of the handler.

// Create a DocumentClient that represents the query to add an item
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient, GetCommand } from "@aws-sdk/lib-dynamodb";
const client = new DynamoDBClient({});
const ddbDocClient = DynamoDBDocumentClient.from(client);

// Get the DynamoDB table name from environment variables
const tableName = process.env.SAMPLE_TABLE;

/**
 * A simple example includes a HTTP get method to get one item by id from a DynamoDB table.
 */
export const getBlockByHashHandler = async (event) => {
  if (event.httpMethod !== "GET") {
    throw new Error(
      `getMethod only accept GET method, you tried: ${event.httpMethod}`
    );
  }
  // All log statements are written to CloudWatch
  console.info("received:", event);

  // Get id from pathParameters from APIGateway because of `/{id}` at template.yaml
  const hash = event.pathParameters.hash;

  // Get the item from the table
  // https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB/DocumentClient.html#get-property
  var params = {
    TableName: tableName,
    Key: { hash: hash },
  };

  try {
    const data = await ddbDocClient.send(new GetCommand(params));
    var item = data.Item;
  } catch (err) {
    console.log("Error", err);
  }

  const response = {
    statusCode: 200,
    body: JSON.stringify(item),
  };

  // All log statements are written to CloudWatch
  console.info(
    `response from: ${event.path} statusCode: ${response.statusCode} body: ${response.body}`
  );
  return response;
};
```

The `getBlockByHashHandler` function retrieves one item based on its hash value, unlike the `getAllBlocksHandler` function which retrieves all items.

This function creates a new `DynamoClient` and gets the name of the DynamoDB table from an environment variable, as we've seen before.

It is executed when the code receives an HTTP GET request.

If the HTTP method is not GET, it throws an error.

Otherwise, the code retrieves the hash value of the item to retrieve from the `event.pathParameters` object.

The code then uses the `GetCommand` object from the `DynamoDBDocumentClient` to retrieve the item from the DynamoDB table. The retrieved item is stored in a variable called `item`, and a response is returned with a JSON-encoded body containing the item. Finally, the code logs the response and returns it to the caller.

**Now, let's get to the core.**

As we mentioned earlier, the Alchemy custom webhooks require an API endpoint that supports POST requests to send blockchain data as soon as a block is verified.

That's where our `store-block-data` function comes in.

First, copy and paste the code inside the `store-block-data.mjs` file.

Then, we'll take a closer look at what's happening inside the function.

#### store-block-data.mjs

javascript

```jsx
// Create clients and set shared const values outside of the handler.

// Create a DocumentClient that represents the query to add an item
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient, PutCommand } from "@aws-sdk/lib-dynamodb";
const client = new DynamoDBClient({});
const ddbDocClient = DynamoDBDocumentClient.from(client);

// Get the DynamoDB table name from environment variables
const tableName = process.env.SAMPLE_TABLE;

/**
 * A simple example includes a HTTP post method to add one item to a DynamoDB table.
 */
export const storeBlockDataHandler = async (event) => {
  if (event.httpMethod !== "POST") {
    throw new Error(
      `postMethod only accepts POST method, you tried: ${event.httpMethod} method.`
    );
  }
  // All log statements are written to CloudWatch
  console.info("received:", event);

  // Get hash and data from the body of the request
  const body = JSON.parse(event.body);
  const data = body.event.data.block.logs;
  const hash = body.event.data.block.hash;
  // Creates a new item, or replaces an old item with a new item
  // https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB/DocumentClient.html#put-property
  var params = {
    TableName: tableName,
    Item: { hash: hash, data: data },
  };

  try {
    const data = await ddbDocClient.send(new PutCommand(params));
    console.log("Success - block added or updated", data);
  } catch (err) {
    console.log("Error", err.stack);
  }

  const response = {
    statusCode: 200,
    message: "success"
  };

  // All log statements are written to CloudWatch
  console.info(
    `response from: ${event.path} statusCode: ${response.statusCode} body: ${response.body}`
  );
  return response;
};
```

Unlike the previous examples, this code is using an HTTP POST request to insert an item in the DynamoDB table.

A POST request enables the API endpoint to receive requests containing data.

In the previous examples, we were GETting data from our database, in this case we want to allow the Alchemy webhook to POST, send, data instead.

To start using DynamoDB, we create once again a **`DynamoDBDocumentClient`**instance and get the name of the table from the environment variable exposed inside the template.yaml file.

The main function of this code is **`storeBlockDataHandler`**.

If the HTTP method is not POST, an error is thrown.

Otherwise, we retrieve the hash of the block and its data from the body object contained in the event payload.

To get access to the “body” object, we need first to parse the event payload from string to JSON.

Once we have access to the body object, we’re able to access the block data and hash and store them in the `data` and `hash` variables.

The code then stores the item in the DynamoDB table using the **`PutCommand`** from the **`DynamoDBDocumentClient`**. The item to be stored is specified in the **`Item`** property of the **`params`** object, which contains the hash value and the data of the item.

After storing the item, the code returns a response with a JSON-encoded body containing the original request body.

Finally, the code logs the response with the data and returns a status of 200, and returns a success message to the caller, the Alchemy custom webhook, to tell the request was succesfull.

This will be the function that will be called through our API endpoint by the Alchemy custom webhook we’ll set up next. The Alchemy webhook will send a POST request to our **/store-block-data** endpoint, with the block’s data in the body. Our function will take care of destructuring the data received and storing it in our DynamoDB table.

Now that we have completed modifying our Lambda functions, it’s time to deploy our Serverless infrastructure on AWS and finally create our Custom webhook using the Alchemy dashboard.

**Let’s see how.**

### Step 5: Deploy the Cloud formation on AWS using SAM

Deploying on AWS using SAM is quite simple. It only requires you to be logged into your AWS CLI. If you haven’t already, you can learn how to set up the AWS CLI [following the official docs](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).

Once logged in, run the following command in the terminal within your project folder:

JSX

```jsx
sam deploy --guided
```

This will start the guided SAM deployment process and will prompt you to select between a number of options.

**For the sake of this tutorial, select the following options**:

* Add your stack name “alchemy-custom-webhooks-stack”
* Select the deployment region - us-x-y
* Y - Confirm the changes
* y - Allow SAM CLI IAM role creation
* n - Enable rollback
* y - Approve deploying all the lambda functions even without authorization defined
* y - Save arguments to the configuration file
* Leave the configuration file name to the default
* Leave sam configuration environment to default

Now, wait for the process to finish.

If everything went as expected, you should now be able to see the resources created in your AWS console!

Navigate to your AWS console and login if you haven't already. Then, get the POST API Endpoint to which the Alchemy Custom webhook will send the Blockchain data. Here's how.

### Step 6: Grab the webhook API endpoint

As mentioned, during deployment, SAM handles the spin-up of all the services specified in the **template.yaml** file and ensures their mutual communication.

![](https://files.readme.io/15caa0d-small-Screenshot\_2023-05-10\_at\_23.30.12.png)

Navigate to the AWS console and search for the “**API Gateway**” service.

![](https://files.readme.io/2d14ddc-small-Screenshot\_2023-04-24\_at\_14.41.27.png)

Click on the newly created **API Gateway**:

![](https://files.readme.io/faecb3c-small-Screenshot\_2023-04-24\_at\_14.41.50\_2.png)

And navigate to **stages** > **Stage** using the left-hand side menu:

![](https://files.readme.io/9051d28-small-Screenshot\_2023-04-24\_at\_14.42.59\_1.png)

At the top of the page, inside the blue box, you’ll see your API Root URL. This is the root of your API endpoints.

As with any other REST API, the different endpoints we’ve specified using SAM, will be accessible by appending the path to the root API Endpoint, for example:

* https://.execute-api..amazonaws.com/Stage/**get-all-blocks**
* https://.execute-api..amazonaws.com/Stage/**get-block-by-hash/{hash}**
* https://.execute-api..amazonaws.com/Stage/**store-block-data**

For our custom webhook, we will need to grab the **store-block-data** endpoint (last one), as it’s the POST endpoint we’ve coded to store the received data inside the Dynamo DB instance.

> ### 📘Note:
>
> Make sure to not copy-paste the example root+endpoint above. You’ll need to get your endpoint found on the API gateway stages page and append the store-block-data path to it.

With our POST endpoint copied, we’re now ready to create our Alchemy custom webhook.

### Step 7: Create a new custom webhook on the Alchemy platform

Creating a custom webhook to feed your database or application with real-time blockchain data, is surprisingly easy.

First, if you haven’t already created an Alchemy account, [make one completely for free](https://dashboard.alchemy.com/signup/?a=aws-custom-webhooks).

Then go to the notify tab and look for the “**GraphQL**” section:

![](https://files.readme.io/8ef7c1e-small-Screenshot\_2023-04-24\_at\_13.18.20.png)

Click on the “**Create webhook**” button to create a new webhook, and select the “**Blank template**”

![](https://files.readme.io/3b32f8c-small-Screenshot\_2023-04-24\_at\_14.51.30.png)

In the webhook URL input, insert the **/store-block-data** API endpoint with built in the previous section

YAML

```yaml
https://YOUR-AMAZONAWS-ENDPOINT/Stage/store-block-data
```

This configuration tells Alchemy where to send data every time a new block is validated.

To verify everything is working and that the data is being saved to the DynamoDB, click "**test webhook**".

This will send a test payload to your store-block-data endpoint and trigger the associated lambda function.

Navigate back to the AWS console, search for “**DynamoDB**,” and look for the “**tables**” section in the left-hand menu:

![](https://files.readme.io/e0bd994-small-Screenshot\_2023-04-24\_at\_14.54.20.png)

Then click on the **DynamoDB** instance we’ve created before using SAM:

![](https://files.readme.io/8d87656-small-Screenshot\_2023-04-24\_at\_13.22.32.png)

Look for the “**Items summary**” section, and click on “**Get live item count**”:

![](https://files.readme.io/cf78a2a-small-Screenshot\_2023-04-24\_at\_13.22.52.png)

Then click on the “**Start scan**” button.

If everything worked as expected you should now see the item count number incremented by 1:

![](https://files.readme.io/d5bf099-small-Screenshot\_2023-04-24\_at\_14.57.50.png)

> ### 📘Note:
>
> The item count property under the “**items summary section**” updates every 6 hours.
>
> Refer to the item count inside the “Get live item count” popup instead.

Amazing!

Now let’s go back on Alchemy.com and click on “**Create webhook**” to start our webhook:

![](https://files.readme.io/a2137b6-small-Screenshot\_2023-04-24\_at\_13.24.15.png)

Now every time a new block will be verified, a new POST request will be sent to your **store-block-data** endpoint and saved on your DynamoDB!

Amazing 🎉&#x20;

We’re almost done, one last thing before completing the tutorial!

Let’s test our API to make sure everything is working as expected.

### Step 8: Test your blockchain ingestion serverless infrastructure

Now that we have created our custom webhook and plugged it into our serverless API, let’s test the API endpoints to make sure is up and running.

As we did before, navigate to your API gateway on AWS and select our API

![](https://files.readme.io/3098001-small-Screenshot\_2023-04-24\_at\_13.25.45.png)

On the left hand side resources menu, click on the GET route under the “get-all-blocks” path.

Then click on the test button:

![](https://files.readme.io/6f2bcc4-small-Screenshot\_2023-04-24\_at\_13.26.00.png)

To test your API endpoint, on the route page, click on the blue test button.

This will trigger a call to the API, GET in this case, and populate the “response body” section on the right-hand side, with the blocks data saved on your DynamoDB instance and retrieved through the lambda function we created.

![](https://files.readme.io/01e2777-small-Screenshot\_2023-04-24\_at\_13.26.54.png)

Before leaving the page, grab the hash of one of the blocks in the response body (the value associated with the hash key in the json object), in this case:

![](https://files.readme.io/e848a41-small-Screenshot\_2023-04-24\_at\_13.29.03.png)

```
0xd6155871b482aa1c5a6b53675cb0010e9bd5614bf83fd96629276a5e756c3ef5
```

We’ll use is to test our **get-blocks-by-hash/{hash}** route.

Navigate to the GET route under **get-blocks-by-hash/{hash}** and click on the “**Test button**” again.

![](https://files.readme.io/6fe0910-small-Screenshot\_2023-04-24\_at\_13.29.57.png)

Paste the hash value copied before in the Path → {hash} input box, and click on the blue **Test button**:

![](https://files.readme.io/717aa28-small-Screenshot\_2023-04-24\_at\_13.30.22\_1.png)

You’ll see the block data related to the hash id you’ve inserted appearing on the right end side, inside the “Response body” section.

> ### 📘Note:
>
> You can also test your GET routes by directly navigating to their endpoints as we saw in section 6.

Congratulations!

The Alchemy custom webhook we set up will now send real-time blockchain data to your /store-block-data API endpoint that will then take care of processing and storing your data in the Dynamo instance we’ve created. The GET /get-all-blocks and /get-all-blocks-by-hash endpoints we've created will make your data accessible querying it from the database.

In this tutorial, we have created a serverless architecture for ingesting blockchain data using AWS and Alchemy.

We have walked through creating a DynamoDB table, creating Lambda functions to retrieve and store data in the table, and creating a webhook on the Alchemy platform to send data to our Lambda functions.

We have also explored how to use SAM to deploy our serverless infrastructure on AWS and how to test our API endpoints to ensure everything is functioning as expected.

Now that you’ve learned a ton, hopefully, of new technologies, and have a fully-fledged real-time blockchain data ingesting and querying API, what can you build with them?

Let us give you some inspiration 💡

### What’s next?

With access to so much data, the possibility to build on top of what you’ve learned in this tutorial is endless.

First, try to experiment with the Custom Webhooks GraphQL queries, and learn how to keep track of events and transactions from specific contracts, collections or wallets.

Then, definitely try to build a dashboard to keep track of a wallet movement, or NFT Collection market analytics!

Or why not, a platform that keeps track of crypto prices across multiple pools?

## Alchemy Subgraphs Overview

Learn about subgraphs, their uses cases and the Alchemy Subgraphs platform.

[Suggest Edits](https://docs.alchemy.com/edit/alchemy-subgraphs-overview)

Alchemy Subgraphs (formerly Satsuma) is a blockchain indexing platform with drop-in support for hosted subgraphs.

### Understanding Subgraphs

Alchemy Subgraphs allow developers to create specialized APIs, aka **subgraphs**, that define how to ingest, process, and store information from the blockchain, making it easier for apps to query blockchain data

### Pain Points Addressed by Alchemy Subgraphs

Alchemy Subgraphs offer a solution designed to mitigate the following pain points and streamline the process for developers:

#### **1. In-House Indexing System Limitations**

The main alternative to subgraphs is developing an in-house indexing system, which is resource-intensive, and has considerable drawbacks:

* **Engineering Time & Cost**: Engineering resources from product development are diverted in order to construct and maintain a custom indexing system.
* **Complexity & Expertise**: Building an indexing system requires specialized knowledge about blockchain intricacies, such as reorgs and data decoding. Companies often face a steep learning curve or need to hire additional, specialized staff.

#### **2. Limitations with **_**Existing**_** Subgraph Services**

Using subgraphs with other current hosted services, also comes with its challenges:

* **Performance Issues**: Slow initial sync times, reindexes, and query response can bottleneck development.
* **Reliability Concerns**: Downtime and inconsistent API performance, often due to operational inefficiencies, user abuse, or maintenance, can disrupt services.
* **Lack of Visibility**: Insufficient insights into indexing and query metrics lead to a lack of trust and additional time spent debugging.
* **Data Lag**: Delays in data updates can impede real-time functionality.
* **Customer Support**: Limited or no direct support hampers problem resolution.

Current decentralized services introduce further complications:

* **Operational Overhead**: Using this service entails complex processes such as acquiring and staking tokens, and signaling your subgraph to indexers.
* **Exposure and Privacy**: The necessity to make your subgraph publicly accessible.
* **Indexer Reliance**: No guarantee that indexers will prioritize or even process your subgraph.
* **Technical Support**: Lack of a direct line for technical support to indexers can prolong issue resolution.

#### **3. Self-Hosting Difficulties**

Self-hosting subgraphs brings its own set of issues:

* **Maintenance Burden**: Operators need to invest significant time and effort into system upkeep.
* **Performance Dependency**: Indexing speed is contingent on running personal RPC nodes.
* **Infrastructure Costs**: There are high costs associated with running the necessary graph node, database, and RPC nodes.

### Alchemy Subgraphs APIs

* [Subgraphs Quickstart](https://docs.alchemy.com/reference/subgraphs-quickstart)
* [Developing a Subgraph](https://docs.alchemy.com/reference/developing-a-subgraph)
  * [Graph CLI](https://docs.alchemy.com/reference/graph-cli)
  * [Creating a Subgraph](https://docs.alchemy.com/reference/creating-a-subgraph)
  * [Project Structure](https://docs.alchemy.com/reference/project-structure)
  * [Data Sources](https://docs.alchemy.com/reference/data-sources)
  * [Writing Mappings](https://thegraph.com/docs/en/developing/creating-a-subgraph/#writing-mappings)
* [Moving Your Subgraph to Production](https://docs.alchemy.com/reference/moving-to-production)
  * [Deploying a Subgraph](https://docs.alchemy.com/reference/deploying-a-subgraph)
  * [Subgraph Versioning](https://docs.alchemy.com/reference/subgraph-versioning)
  * [Querying a Subgraph](https://docs.alchemy.com/reference/querying-a-subgraph)
  * [Deleting a Subgraph](https://docs.alchemy.com/reference/deleting-a-subgraph)
  * [Supported Subgraph Chains](https://docs.alchemy.com/reference/supported-subgraph-chains)
  * [Direct Database Access](https://docs.alchemy.com/reference/direct-database-access)

## Introduction to Subgraphs

[Suggest Edits](https://docs.alchemy.com/edit/introduction-to-subgraphs)

Alchemy Subgraphs allow developers to create specialized APIs, aka **subgraphs**, that define how to ingest, process, and store information from the blockchain, making it easier for apps to query blockchain data.

### What is a subgraph?

**Subgraphs aggregate application-specific blockchain data for quick access to frontend developers.**

Subgraph are exposed to developers via GraphQL APIs, allowing users to query the transaction data happening on their contract in real time. Subgraphs are especially beneficial for developers of complex, custom smart contracts that need to have robust frontend interfaces.

For example, to query all transactions within a single Uniswap v3 liquidity pool over the last 24 hours, Uniswap simply needs to define their schema, index the event data to create the subgraph, and then use the generated GraphQL API to query their subgraph for flexible and efficient blockchain data.

Subgraphs allow developers to filter and sort data based on their needs, letting them extract only the information important to their dapp. Subgraphs also enable more efficient data querying by precompiling and indexing data to speed up the querying process instead of requesting data directly from full nodes or archive nodes. Let's dive in further... 🤿

### Why Do We Need Subgraphs & What Problem Do They Solve?

Before we dive deeper into subgraphs, let's first quickly analyze blockchains as data structures. It's helpful to view blockchains simply as databases that are distributed and decentralized. Blockchain databases grow by perpetually adding new blocks, each full of data.

The thing about blockchains is, if viewed as databases, they mainly optimize for:

* **Immutability**: once data is added, it is really difficult to change that data, so data integrity and historical accuracy is excellent.
* **Transparency**: all data is public, verifiable and accessible by all.
* **Decentralization**: blockchains deliver trustless and censorship-resistant server environments for all to use, given there is no central operator.
* **Security**: all transactions must be independently verified by all network participants, which makes fraudulent transactions virtually impossible.

All of the above properties are **awesome** for a database to have! They enable applications built on blockchain databases to be really powerful, as developers can lace these properties into the user experience.

_However_, while blockchains optimize for all of the above properties in great ways, they are not so great for complex data querying. When you need to find a specific piece of data in the blockchain, you need to read through _every single block ever_ and attempt to find your specific piece of data - this is obviously not very efficient! Modern databases (ie, SQL, MongoDB, Postgres DB) are a great solution for complex data querying; **subgraphs** will help us set up and maintain one such database to help us perform quick and efficient complex data queries to the blockchain.

Say, for example, you are writing a blockchain analytics app and you want to get all of the latest transfers on the [$USDC smart contract](https://docs.alchemy.com/docs/0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48). Without a subgraph, you'd need to brute force the last 10 or so blocks on the chain, each containing \~150 transactions. In each block, you'd have to check each transaction to see if there was any interaction with the relevant contract address. That's a lot of brute-force searching and that's just for 10 blocks! Here's how a subgraph can help make this query more efficient:

![](https://files.readme.io/becfea4-image\_10.png)

As seen in the diagram above, subgraphs are helpful for indexing complex blockchain data into easily queryable formats, unlocking faster speed and data reliability for end-user and developers alike. The subgraph will consume all of our wanted data for us and set it up in easy-to-query format for us to enjoy as application developers. This subgraph solution becomes even more needed when you need to calculate some aggregate data across all historical blocks, like total inflows ever for a pool.

Want to dive deeper? Check out [this comprehensive Alchemy blog post on subgraphs](https://www.alchemy.com/overviews/what-is-a-subgraph#what-is-a-subgraph)!

## How To Query a Subgraph

See examples on how to query subgraphs deployed on Alchemy Subgraphs.

[Suggest Edits](https://docs.alchemy.com/edit/how-to-query-a-subgraph)

### Introduction

In this guide, we will walk you through the process of querying an already-deployed subgraph on the Alchemy Subgraphs platform. We will include a couple of simple queries to help you get an idea on the type of data you can get using subgraphs.

There are a couple of ways you can query a subgraph, we will cover using:

1. [Alchemy Subgraphs](https://subgraphs.alchemy.com/dashboard) GraphQL Playground
2. [Postman](https://www.postman.com/)

### 1. Using the Alchemy Subgraphs GraphQL Playground

1. Go to one of your deployed subgraphs in the [Alchemy Subgraphs Dashboard](https://subgraphs.alchemy.com/dashboard) and go to the `Playground` URL at the center-top of the page
2. Check out the GraphiQL interface. This playground let's you type in GraphQL-formatted queries and send them as a data request to the loaded subgraph.

![](https://files.readme.io/a02e8b8-Screenshot\_2023-10-30\_at\_1.11.14\_PM.png)

3. Delete all of the comments so you have a clean input space to play with.
4. Copy-paste one of the following queries into the playground, then hit the Play button or press Ctrl + Enter (or Cmd + Enter on Mac) to execute the query:

### Sample Queries

If you are using the Pudgy Penguins Transfers subgraph you built in the previous guides, here are some queries you can make:

### Query to get the first 100 Pudgy Penguin transfers:

GraphQL

```Text
{
  transfers(first: 100) {
    id
    from
    to
    tokenId
    blockNumber
    blockTimestamp
    transactionHash
  }
}
```

### Query to get all transfers from a specific address

GraphQL

```Text
{
  transfers(where: { from: "0xSpecificAddress" }) {
    id
    from
    to
    tokenId
    blockNumber
    blockTimestamp
    transactionHash
  }
}
```

> You can try `0x29469395eAf6f95920E59F858042f0e28D98a20B` for the above query.

### Query to get all transfers related to a specific tokenId

GraphQL

```Text
{
  transfers(where: { tokenId: "SPECIFIC_TOKEN_ID" }) {
    id
    from
    to
    tokenId
    blockNumber
    blockTimestamp
    transactionHash
  }
}
```

> You can try tokenId `3389` for the above query.

### 2. Using Postman

Postman is a great way to quickly test your subgraph and how a simple query looks like when the subgraph responds.

1. [Download](https://www.postman.com/downloads/) and open the Postman application
2. Select 'New' and select 'HTTP' from the query options.
3. Enter your subgraph's URL endpoint into the URL field.
4. In the 'Headers' tab, set the `Key` to `Content-Type` and the `Value` to `application/json`
5. In the 'Body' tab, select the `raw` radio button
6. Finally, enter your query as a JSON object in the `query`, you can use this one, for example, to get all the transfers on Pudgy Penguin 100:

JSON

```Text
{
  "query": "query { transfers(where: { tokenId: \"100\" }) { id from to tokenId blockNumber blockTimestamp transactionHash } }"
}
```

7. Hit 'Send'

You should now see the data print in the Postman console!
