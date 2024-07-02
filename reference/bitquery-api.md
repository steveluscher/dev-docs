# Bitquery API

## Overview

Bitquery 's infrastructure provides you access to historical and real-time blockchain data through various interfaces such as GraphQL APIs.

### GraphQL Query API[​](https://docs.bitquery.io/docs/intro/#graphql-query-api) <a href="#graphql-query-api" id="graphql-query-api"></a>

Get started with our APIs in a minute by building [**your first query**](https://docs.bitquery.io/docs/start/first-query/).

You can query [archive](https://docs.bitquery.io/docs/graphql/dataset/archive/), [real-time](https://docs.bitquery.io/docs/graphql/dataset/realtime/) or [combined](https://docs.bitquery.io/docs/graphql/dataset/combined/) dataset based on your requirements.

After the query is built you can [save](https://docs.bitquery.io/docs/ide/private/) it and embed it in your application using [pre-cooked code snippet](https://docs.bitquery.io/docs/ide/code/) in any popular programming language.

```
query {
  EVM(dataset: archive network: bsc) {
    Transactions {
      Block {
        Date
      }
      count
    }
  }
}
```

### Integrated Development Environment (IDE)[​](https://docs.bitquery.io/docs/intro/#integrated-development-environment-ide) <a href="#integrated-development-environment-ide" id="integrated-development-environment-ide"></a>

Integrated Development Environment ([**IDE**](https://graphql.bitquery.io/ide?endpoint=https://streaming.bitquery.io/graphql)) helps you to manage your query, share them with other developers and generate a code to use the queries in your applications.

### GraphQL Subscription (WebSocket) API[​](https://docs.bitquery.io/docs/intro/#graphql-subscription-websocket-api) <a href="#graphql-subscription-websocket-api" id="graphql-subscription-websocket-api"></a>

Subscription (WebSocket) is an extension of GraphQL API. It allows to subscribe on the updates in the data in real-time and receive the new data changes using WebSocket protocol.

Protocols subscriptions-transport-ws and graphql-transport-ws are supported.

```
subscription {
  EVM(trigger_on: head) {
    Transactions {
      Block {
        Hash
        Number
        Date
      }
      count
    }
  }
}
```

### Cloud Data Storage[​](https://docs.bitquery.io/docs/intro/#cloud-data-storage) <a href="#cloud-data-storage" id="cloud-data-storage"></a>

If you build your applications in cloud or you need raw data for deep investigations or even machine learning algorithms, use the cloud data storage.

It contains optimized data for applications on different levels - from the raw data from blockchain nodes to the parsed protocols as DEX (decentralized exchanges) or NFT (non-fungible tokens).

## Solana Raydium API

In this section, we will see how to get Raydium information using Bitquery APIs.

This Solana API is part of our Early Access Program (EAP), which is intended for evaluation purposes.

This program allows you to test the data and its integration into your applications before full-scale implementation. Read more [here](https://docs.bitquery.io/docs/graphql/dataset/EAP/)

### New Liquidity Pools Created on Solana Raydium DEX (Using Websocket)[​](https://docs.bitquery.io/docs/examples/Solana/Solana-Raydium-DEX-API/#new-liquidity-pools-created-on-solana-raydium-dex-using-websocket) <a href="#new-liquidity-pools-created-on-solana-raydium-dex-using-websocket" id="new-liquidity-pools-created-on-solana-raydium-dex-using-websocket"></a>

You can subscribe to newly created Solana Raydium liquidity pools using the GraphQL subscription (WebSocket). You can try this GraphQL subscription [using this link](https://ide.bitquery.io/Latest-Radiyum-V4-pools-created\_1).

In the results, you can get pool and token details using instructions.

#### Pool Address[​](https://docs.bitquery.io/docs/examples/Solana/Solana-Raydium-DEX-API/#pool-address) <a href="#pool-address" id="pool-address"></a>

You can find the pool address using the following result: Note that the array index starts from 0. Therefore, it will be the 5th entry.

Instructions -> Instruction -> Accounts\[4] -> Address

#### Token A[​](https://docs.bitquery.io/docs/examples/Solana/Solana-Raydium-DEX-API/#token-a) <a href="#token-a" id="token-a"></a>

You can get the 1st token address using the following result: Note that the array index starts from 0. Therefore, it will be the 9th entry.

Instructions -> Instruction -> Accounts\[8] -> Address

#### Token B[​](https://docs.bitquery.io/docs/examples/Solana/Solana-Raydium-DEX-API/#token-b) <a href="#token-b" id="token-b"></a>

You can get the 2nd token address using the following result.

Instructions -> Instruction -> Accounts\[9] ->. Address

You can run the following query at [Bitquery IDE](https://ide.bitquery.io/Latest-Radiyum-V4-pools-created\_1).

```
subscription {
  Solana {
    Instructions(
      where: {
        Transaction: { Result: { Success: true } }
        Instruction: {
          Program: {
            Method: { is: "initializeUserWithNonce" }
            Address: { is: "675kPX9MHTjS2zt1qfr1NYHuzeLXfQM9H24wFSUt1Mp8" }
          }
        }
      }
    ) {
      Block {
        Time
        Date
      }
      Transaction {
        Signature
      }
      Instruction {
        AncestorIndexes
        CallerIndex
        Depth
        Data
        ExternalSeqNumber
        InternalSeqNumber
        Index
        Accounts {
          Address
          IsWritable
          Token {
            Mint
            Owner
            ProgramId
          }
        }
        CallPath
        Logs
        Program {
          AccountNames
          Method
          Json
          Name
          Arguments {
            Type
            Name
            Value {
              __typename
              ... on Solana_ABI_Integer_Value_Arg {
                integer
              }
              ... on Solana_ABI_String_Value_Arg {
                string
              }
              ... on Solana_ABI_Address_Value_Arg {
                address
              }
              ... on Solana_ABI_BigInt_Value_Arg {
                bigInteger
              }
              ... on Solana_ABI_Bytes_Value_Arg {
                hex
              }
              ... on Solana_ABI_Boolean_Value_Arg {
                bool
              }
              ... on Solana_ABI_Float_Value_Arg {
                float
              }
              ... on Solana_ABI_Json_Value_Arg {
                json
              }
            }
          }
        }
      }
    }
  }
}
```

### Latest price of a token[​](https://docs.bitquery.io/docs/examples/Solana/Solana-Raydium-DEX-API/#latest-price-of-a-token) <a href="#latest-price-of-a-token" id="latest-price-of-a-token"></a>

You can use the following query to get the latest price of a token on Raydium DEX on Solana.

You can run this query using [this link](https://ide.bitquery.io/Live-price-of-a-token).

```
  Solana {
    DEXTradeByTokens(
      limit: {count: 1}
      orderBy: {descending: Block_Time}
      where: {Trade: {Currency: {MintAddress: {is: "So11111111111111111111111111111111111111112"}}, Side: {Currency: {MintAddress: {is: "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v"}}}}}
    ) {
      Block {
        Time
      }
      Trade {
        Price
      }
    }
  }
}

```

### Latest Trades on Solana Raydium DEX[​](https://docs.bitquery.io/docs/examples/Solana/Solana-Raydium-DEX-API/#latest-trades-on-solana-raydiumdex) <a href="#latest-trades-on-solana-raydiumdex" id="latest-trades-on-solana-raydiumdex"></a>

To subscribe to the real-time trades stream for Solana Raydium DEX, [try this GraphQL subscription (WebSocket)](https://ide.bitquery.io/Updated-Real-time-trades-on-Raydium-DEX-on-Solana\_1).

```
subscription {
  Solana {
    DEXTrades(
      where: {
        Trade: {
          Dex: {
            ProgramAddress: {
              is: "675kPX9MHTjS2zt1qfr1NYHuzeLXfQM9H24wFSUt1Mp8"
            }
          }
        }
      }
    ) {
      Trade {
        Dex {
          ProgramAddress
          ProtocolFamily
          ProtocolName
        }
        Buy {
          Account {
            Address
          }
          Amount
          Currency {
            MintAddress
            Decimals
            Symbol
            ProgramAddress
            Name
          }
          PriceAgaistSellCurrency: Price
        }
        Sell {
          Account {
            Address
          }
          Amount
          Currency {
            MintAddress
            Decimals
            Symbol
            Name
          }
          PriceAgaistBuyCurrency: Price
        }
      }
      Block {
        Time
        Height
      }
      Transaction {
        Signature
        FeePayer
        Signer
      }
    }
  }
}
```

### Latest Trades for a specific currency on Solana Raydium DEX[​](https://docs.bitquery.io/docs/examples/Solana/Solana-Raydium-DEX-API/#latest-trades-for-a-specific-currency-on-solana-raydiumdex) <a href="#latest-trades-for-a-specific-currency-on-solana-raydiumdex" id="latest-trades-for-a-specific-currency-on-solana-raydiumdex"></a>

Let's say you want to receive [trades only for a specific currency on Raydium DEX](https://ide.bitquery.io/Updated-Real-time-buy-and-sell-of-specific-currency-on-Raydium-DEX-on-Solana\_1). You can use the following stream. Use currency's mint address; for example, in the following query, we are using Ray token's Mint address to get buy and sells of Ray token.

If you limit it to 1, you will get the latest price of the token because the latest trade = the Latest Price.

Run this query [using this link](https://ide.bitquery.io/Updated-Real-time-buy-and-sell-of-specific-currency-on-Raydium-DEX-on-Solana\_1).

```
subscription {
  Solana {
    Buyside: DEXTrades(
      where: {
        Trade: {
          Buy: {
            Currency: {
              MintAddress: {
                is: "4k3Dyjzvzp8eMZWUXbBCjEvwSkkk59S5iCNLY3QrkX6R"
              }
            }
          }
          Dex: {
            ProgramAddress: {
              is: "675kPX9MHTjS2zt1qfr1NYHuzeLXfQM9H24wFSUt1Mp8"
            }
          }
        }
      }
    ) {
      Trade {
        Dex {
          ProgramAddress
          ProtocolFamily
          ProtocolName
        }
        Buy {
          Account {
            Address
          }
          Amount
          Currency {
            Decimals
            Symbol
            MintAddress
            Name
          }
          PriceAgaistSellCurrency: Price
        }
        Sell {
          Account {
            Address
          }
          Amount
          Currency {
            Decimals
            Symbol
            MintAddress
            Name
          }
          PriceAgaistBuyCurrency: Price
        }
      }
      Block {
        Time
        Height
      }
      Transaction {
        Signature
        FeePayer
        Signer
      }
    }
    Sellside: DEXTrades(
      limit: { count: 10 }
      where: {
        Trade: {
          Sell: {
            Currency: {
              MintAddress: {
                is: "4k3Dyjzvzp8eMZWUXbBCjEvwSkkk59S5iCNLY3QrkX6R"
              }
            }
          }
          Dex: {
            ProgramAddress: {
              is: "675kPX9MHTjS2zt1qfr1NYHuzeLXfQM9H24wFSUt1Mp8"
            }
          }
        }
      }
    ) {
      Trade {
        Dex {
          ProgramAddress
          ProtocolFamily
          ProtocolName
        }
        Buy {
          Account {
            Address
          }
          Amount
          Currency {
            Decimals
            Symbol
            MintAddress
            Name
          }
          PriceAgaistSellCurrency: Price
        }
        Sell {
          Account {
            Address
          }
          Amount
          Currency {
            Decimals
            Symbol
            MintAddress
            Name
          }
          PriceAgaistBuyCurrency: Price
        }
      }
      Block {
        Time
        Height
      }
      Transaction {
        Signature
        FeePayer
        Signer
      }
    }
  }
}
```

### Raydium OHLC API[​](https://docs.bitquery.io/docs/examples/Solana/Solana-Raydium-DEX-API/#raydium-ohlc-api) <a href="#raydium-ohlc-api" id="raydium-ohlc-api"></a>

You can also query OHLC data in real time from Raydium DEX.

[Here](https://ide.bitquery.io/Orca-OHLC-for-all-currencies) is Websocket to get OHLC data for all currencies, however, if you want to get OHLC data for any specific currency pair, you can use [this Websocket api](https://ide.bitquery.io/Raydium-OHLC-for-specific-pair\_3).

```
subscription {
  Solana {
    DEXTradeByTokens(
      orderBy: { descendingByField: "Block_Timefield" }
      where: {
        Trade: {
          Currency: {
            MintAddress: { is: "6D7NaB2xsLd7cauWu1wKk6KBsJohJmP2qZH9GEfVi5Ui" }
          }
          Side: {
            Currency: {
              MintAddress: { is: "So11111111111111111111111111111111111111112" }
            }
          }
          Dex: {
            ProgramAddress: {
              is: "675kPX9MHTjS2zt1qfr1NYHuzeLXfQM9H24wFSUt1Mp8"
            }
          }
        }
      }
    ) {
      Block {
        Timefield: Time(interval: { in: minutes, count: 1 })
      }
      volume: sum(of: Trade_Amount)
      Trade {
        high: Price(maximum: Trade_Price)
        low: Price(minimum: Trade_Price)
        open: Price(minimum: Block_Slot)
        close: Price(maximum: Block_Slot)
      }
      count
    }
  }
}
```

### Track Latest Add Liquidity Transactions on Raydium DEX[​](https://docs.bitquery.io/docs/examples/Solana/Solana-Raydium-DEX-API/#track-latest-add-liquidity-transactions-on-raydium-dex) <a href="#track-latest-add-liquidity-transactions-on-raydium-dex" id="track-latest-add-liquidity-transactions-on-raydium-dex"></a>

You can also track Add Liquidity transactions in real time on Raydium DEX from Raydium API using instructions. Firstly, you can use this [query](https://ide.bitquery.io/Get-all-methods-of-Raydium-V4-Program) to get all the methods of Raydium V4 program to deduce which program method is triggered for add liquidity transactions. The method we want to filter for turns out to be `setPositionStopLoss`.

If you want to track latest liquidity additions in Raydium pools, you can use [this Websocket api](https://ide.bitquery.io/Track-Add-Liquidity-Transactions-on-Solana-Raydium-DEX).

In the response, mint under 7th and 8th addresses in the Accounts array gives you the Token A and Token B respectively of the pool in which liquidity is added.

```
subscription {
  Solana {
    Instructions(
      where: {
        Instruction: {
          Program: {
            Address: { is: "675kPX9MHTjS2zt1qfr1NYHuzeLXfQM9H24wFSUt1Mp8" }
            Method: { is: "setPositionStopLoss" }
          }
        }
        Transaction: { Result: { Success: true } }
      }
    ) {
      Transaction {
        Signature
      }
      Block {
        Time
      }
      Instruction {
        Accounts {
          Address
          IsWritable
          Token {
            ProgramId
            Owner
            Mint
          }
        }
        AncestorIndexes
        BalanceUpdatesCount
        CallPath
        CallerIndex
        Data
        Depth
        Logs
        InternalSeqNumber
        Index
        ExternalSeqNumber
        Program {
          Address
          AccountNames
          Method
          Arguments {
            Name
            Type
            Value {
              ... on Solana_ABI_Integer_Value_Arg {
                integer
              }
              ... on Solana_ABI_String_Value_Arg {
                string
              }
              ... on Solana_ABI_Address_Value_Arg {
                address
              }
              ... on Solana_ABI_BigInt_Value_Arg {
                bigInteger
              }
              ... on Solana_ABI_Bytes_Value_Arg {
                hex
              }
              ... on Solana_ABI_Boolean_Value_Arg {
                bool
              }
              ... on Solana_ABI_Float_Value_Arg {
                float
              }
              ... on Solana_ABI_Json_Value_Arg {
                json
              }
            }
          }
        }
      }
    }
  }
}
```

### Track Latest Remove Liquidity Transactions on Raydium DEX[​](https://docs.bitquery.io/docs/examples/Solana/Solana-Raydium-DEX-API/#track-latest-remove-liquidity-transactions-on-raydium-dex) <a href="#track-latest-remove-liquidity-transactions-on-raydium-dex" id="track-latest-remove-liquidity-transactions-on-raydium-dex"></a>

You can also track Remove Liquidity transactions in real time on Raydium DEX from Raydium API using instructions. Firstly, you can use this [query](https://ide.bitquery.io/Get-all-methods-of-Raydium-V4-Program) to get all the methods of Raydium V4 program to deduce which program method is triggered for remove liquidity transactions. The method we want to filter for turns out to be `setPositionRangeStop`.

If you want to track latest liquidity removals in Raydium pools, you can use [this Websocket api](https://ide.bitquery.io/Track-Remove-Liquidity-Transactions-on-Solana-Raydium-DEX).

In the response, mint under 7th and 8th addresses in the Accounts array gives you the Token A and Token B respectively of the pool in which liquidity is removed.

```
subscription {
  Solana {
    Instructions(
      where: {
        Instruction: {
          Program: {
            Address: { is: "675kPX9MHTjS2zt1qfr1NYHuzeLXfQM9H24wFSUt1Mp8" }
            Method: { is: "setPositionRangeStop" }
          }
        }
        Transaction: { Result: { Success: true } }
      }
    ) {
      Transaction {
        Signature
      }
      Block {
        Time
      }
      Instruction {
        Accounts {
          Address
          IsWritable
          Token {
            ProgramId
            Owner
            Mint
          }
        }
        AncestorIndexes
        BalanceUpdatesCount
        CallPath
        CallerIndex
        Data
        Depth
        Logs
        InternalSeqNumber
        Index
        ExternalSeqNumber
        Program {
          Address
          AccountNames
          Method
          Arguments {
            Name
            Type
            Value {
              ... on Solana_ABI_Integer_Value_Arg {
                integer
              }
              ... on Solana_ABI_String_Value_Arg {
                string
              }
              ... on Solana_ABI_Address_Value_Arg {
                address
              }
              ... on Solana_ABI_BigInt_Value_Arg {
                bigInteger
              }
              ... on Solana_ABI_Bytes_Value_Arg {
                hex
              }
              ... on Solana_ABI_Boolean_Value_Arg {
                bool
              }
              ... on Solana_ABI_Float_Value_Arg {
                float
              }
              ... on Solana_ABI_Json_Value_Arg {
                json
              }
            }
          }
        }
      }
    }
  }
}
```

### Track Raydium DEXTrades enabled by OpenBook Protocol[​](https://docs.bitquery.io/docs/examples/Solana/Solana-Raydium-DEX-API/#track-raydium-dextrades-enabled-by-openbook-protocol) <a href="#track-raydium-dextrades-enabled-by-openbook-protocol" id="track-raydium-dextrades-enabled-by-openbook-protocol"></a>

You can track Raydium DEXTrades which are enabled by OpenBook Protocol in real time on Raydium DEX from Raydium API using instructions. So OpenBook is an Order Book Protocol which Raydium has integrated in its constant product amm. If you want a full explaination of this API, watch our [Youtube Video](https://www.youtube.com/watch?v=aYARyvvItHA).

If you want to track latest Raydium DEXTrades enabled by OpenBook order book Protocol, you can use [this Websocket api](https://ide.bitquery.io/Raydium-dextrades-through-OpenBook-order-book).

```
subscription {
  Solana {
    DEXTrades(
      where: {
        Trade: { Dex: { ProtocolFamily: { is: "Raydium" } } }
        Instruction: {
          Accounts: {
            includes: {
              Address: { is: "srmqPvymJeFKQ4zGQed1GFppgkRHL9kaELCbyksJtPX" }
            }
          }
        }
      }
    ) {
      Trade {
        Buy {
          Account {
            Address
          }
          Amount
          AmountInUSD
          Currency {
            MintAddress
            Symbol
          }
          Price
          PriceInUSD
        }
        Market {
          MarketAddress
        }
        Dex {
          ProtocolFamily
          ProtocolName
          ProgramAddress
        }
        Sell {
          Account {
            Address
          }
          Amount
          AmountInUSD
          Currency {
            MintAddress
            Symbol
          }
          Price
          PriceInUSD
        }
      }
      Transaction {
        Signature
      }
    }
  }
}
```

## Solana Balance & Balance Updates API

In this section we will see how to monitor real-time balance changes across the Solana blockchain using our BalanceUpdates API.

Solana APIs is part of our Early Access Program (EAP), which is intended for evaluation purposes. This program allows you to test the data and its integration into your applications before full-scale implementation. Read more [here](https://docs.bitquery.io/docs/graphql/dataset/EAP/)

Note - Our [V1 APIs](https://docs.bitquery.io/v1/docs/category/examples) do support solana and you can get balances from there. However they do not have historical balance.

### Get Latest Balance Updates[​](https://docs.bitquery.io/docs/examples/Solana/solana-balance-updates/#get-latest-balance-updates) <a href="#get-latest-balance-updates" id="get-latest-balance-updates"></a>

The query will subscribe you to real-time updates for balance changes on the Solana blockchain, providing a continuous stream of data as new transactions are processed and recorded. You can find the query [here](https://ide.bitquery.io/Solana-Balance-Updates)

```
subscription {
  Solana {
    InstructionBalanceUpdates(limit: {count: 10}) {
      Transaction {
        Index
        FeePayer
        Fee
        Signature
        Result {
          Success
          ErrorMessage
        }
      }
      Instruction {
        InternalSeqNumber
        Index
        CallPath
        Program {
          Address
          Name
          Parsed
        }
      }
      Block {
        Time
        Hash
        Height
      }
      BalanceUpdate {
        Account {
          Address
        }
        Amount
        Currency {
          Decimals
          CollectionAddress
          Name
          Key
          IsMutable
          Symbol
        }
      }
    }
  }
}

```

### Get Balance Updates of a Particular Wallet[​](https://docs.bitquery.io/docs/examples/Solana/solana-balance-updates/#get-balance-updates-of-a-particular-wallet) <a href="#get-balance-updates-of-a-particular-wallet" id="get-balance-updates-of-a-particular-wallet"></a>

To focus on the balance changes of a particular Solana wallet, this query filters the data stream to include only those updates relevant to the specified address. This is especially useful for wallet owners or services tracking specific accounts.

```
subscription {
  Solana {
    InstructionBalanceUpdates(
      limit: {count: 10}
      where: {BalanceUpdate: {Account: {Address: {is: "68sMd16LafqLfyS7ejieyWhMsAwKgywVKvSEumvK1RUp"}}}}
    ) {
      Transaction {
        Index
        FeePayer
        Fee
        Signature
        Result {
          Success
          ErrorMessage
        }
      }
      Instruction {
        InternalSeqNumber
        Index
        CallPath
        Program {
          Address
          Name
          Parsed
        }
      }
      Block {
        Time
        Hash
        Height
      }
      BalanceUpdate {
        Account {
          Address
        }
        Amount
        Currency {
          Decimals
          CollectionAddress
          Name
          Key
          IsMutable
          Symbol
        }
      }
    }
  }
}

```

### Track NFT Balance Updates in Real-Time[​](https://docs.bitquery.io/docs/examples/Solana/solana-balance-updates/#track-nft-balance-updates-in-real-time) <a href="#track-nft-balance-updates-in-real-time" id="track-nft-balance-updates-in-real-time"></a>

For those interested in the NFT market, this query is tailored to track balance updates involving non-fungible tokens (NFTs) on the Solana blockchain.

You can find the query [here](https://ide.bitquery.io/Solana-NFT-Balance-Updates)

```
subscription {
  Solana {
    InstructionBalanceUpdates(
      limit: {count: 10}
      where: {BalanceUpdate: {Currency: {Fungible: false}}}
    ) {
      Transaction {
        Index
        FeePayer
        Fee
        Signature
        Result {
          Success
          ErrorMessage
        }
      }
      Instruction {
        InternalSeqNumber
        Index
        CallPath
        Program {
          Address
          Name
          Parsed
        }
      }
      Block {
        Time
        Hash
        Height
      }
      BalanceUpdate {
        Account {
          Address
          Token {
            Owner
          }
        }
        Amount
        Currency {
          Decimals
          CollectionAddress
          Name
          Key
          IsMutable
          Symbol
        }
      }
    }
  }
}

```

### Latest Balance of an Address on Solana[​](https://docs.bitquery.io/docs/examples/Solana/solana-balance-updates/#latest-balance-of-an-address-on-solana) <a href="#latest-balance-of-an-address-on-solana" id="latest-balance-of-an-address-on-solana"></a>

The query will subscribe you to real-time updates for balance changes on the Solana blockchain for the address `675kPX9MHTjS2zt1qfr1NYHuzeLXfQM9H24wFSUt1Mp8`, The `PreBalance` and `PostBalance` functions are used here to get the balance. You can find the query [here](https://ide.bitquery.io/Balance-of-the-raydium-liquidity-pool-address)

```
subscription {
  Solana {
    BalanceUpdates(
      where: {BalanceUpdate: {Account: {Address: {is: "675kPX9MHTjS2zt1qfr1NYHuzeLXfQM9H24wFSUt1Mp8"}}}}
    ) {
      BalanceUpdate {
        Account {
          Address
        }
        Amount
        Currency {
          Decimals
          CollectionAddress
          Name
          Key
          IsMutable
          Symbol
        }
        PreBalance
        PostBalance
      }
    }
  }
}

```

## Solana DEX Trades API

In this section we will see how to get Solana DEX trades information using our API.

This Solana API is part of our Early Access Program (EAP), which is intended for evaluation purposes. This program allows you to test the data and its integration into your applications before full-scale implementation. Read more [here](https://docs.bitquery.io/docs/graphql/dataset/EAP/)

### Subscribe to Latest Solana Trades[​](https://docs.bitquery.io/docs/examples/Solana/solana-dextrades/#subscribe-to-latest-solana-trades) <a href="#subscribe-to-latest-solana-trades" id="subscribe-to-latest-solana-trades"></a>

This subscription will return information about the most recent trades executed on Solana's DEX platforms. You can find the query [here](https://ide.bitquery.io/Realtime-solana-dex-trades-websocket)

```
subscription {
  Solana {
    DEXTrades {
      Trade {
        Dex {
          ProgramAddress
          ProtocolFamily
          ProtocolName
        }
        Buy {
          Amount
          Account {
            Address
          }
          Currency {
            MetadataAddress
            Key
            IsMutable
            EditionNonce
            Decimals
            CollectionAddress
            Fungible
            Symbol
            Native
            Name
          }
          Order {
            LimitPrice
            LimitAmount
            OrderId
          }
        }
        Market {
          MarketAddress
        }
        Sell {
          Account {
            Address
          }
          Currency {
            MetadataAddress
            Key
            IsMutable
            EditionNonce
            Decimals
            CollectionAddress
            Fungible
            Symbol
            Native
            Name
          }
        }
      }
      Instruction {
        Program {
          Address
          AccountNames
          Method
          Parsed
          Name
        }
      }
    }
  }
}


```

### Get all DEXes[​](https://docs.bitquery.io/docs/examples/Solana/solana-dextrades/#get-all-dexes) <a href="#get-all-dexes" id="get-all-dexes"></a>

To get the list of all DEXes operating within the Solana ecosystem, use the following query. Find the query [here](https://ide.bitquery.io/Solana-DEXs)

```
query MyQuery {
  Solana {
    DEXTrades(limitBy: {by: Trade_Dex_ProtocolFamily, count: 1}, limit: {count: 10}) {
      Trade {
        Dex {
          ProgramAddress
          ProtocolFamily
          ProtocolName
        }
      }
    }
  }
}

```

### Get Solana DEX Orders in Real-Time[​](https://docs.bitquery.io/docs/examples/Solana/solana-dextrades/#get-solana-dex-orders-in-real-time) <a href="#get-solana-dex-orders-in-real-time" id="get-solana-dex-orders-in-real-time"></a>

This query provides real-time updates on order events, including details about the DEX, market, and order specifics.

```
subscription {
  Solana {
    DEXOrders(limit: {count: 10}) {
      Instruction {
        Index
        Program {
          Address
          AccountNames
          Name
          Method
        }
      }
      OrderEvent {
        Dex {
          ProtocolName
          ProtocolFamily
          ProgramAddress
        }
        Index
        Market {
          MarketAddress
          CoinToken {
            Wrapped
            VerifiedCollection
            Uri
            UpdateAuthority
            TokenStandard
            Symbol
            TokenCreator {
              Share
              Address
            }
            Name
            Key
            Fungible
            CollectionAddress
          }
          PriceToken {
            Wrapped
            VerifiedCollection
            Uri
            TokenStandard
            Native
            Name
            MetadataAddress
            Key
            Fungible
            Decimals
            CollectionAddress
          }
        }
        Order {
          BuySide
          Account
          Payer
          OrderId
          Owner
        }
        Type
      }
    }
  }
}

```

### Get Latest Price of a Token in Real-time[​](https://docs.bitquery.io/docs/examples/Solana/solana-dextrades/#get-latest-price-of-a-token-in-real-time) <a href="#get-latest-price-of-a-token-in-real-time" id="get-latest-price-of-a-token-in-real-time"></a>

This query provides real-time updates on price, including details about the DEX, market, and order specifics. Find the query [here](https://ide.bitquery.io/Latest-price-of-Ansems-cat-token-on-Solana\_2)

```
subscription {
  Solana {
    DEXTradeByTokens(
      where: {Trade: {Currency: {MintAddress: {is: "6n7Janary9fqzxKaJVrhL9TG2F61VbAtwUMu1YZscaQS"}}, Side: {Currency: {MintAddress: {is: "So11111111111111111111111111111111111111112"}}}}}
    ) {
      Block {
        Time
      }
      Trade {
        Amount
        Price
        Currency {
          Symbol
          Name
          MintAddress
        }
        Side {
          Amount
          Currency {
            Symbol
            Name
            MintAddress
          }
        }
        Dex {
          ProgramAddress
          ProtocolFamily
          ProtocolName
        }
        Market {
          MarketAddress
        }
        Order {
          LimitAmount
          LimitPrice
          OrderId
        }
      }
    }
  }
}

```

### Latest USD Price of a Token[​](https://docs.bitquery.io/docs/examples/Solana/solana-dextrades/#latest-usd-price-of-a-token) <a href="#latest-usd-price-of-a-token" id="latest-usd-price-of-a-token"></a>

The below query retrieves the USD price of a token on Solana by setting `SmartContract: {is: "So11111111111111111111111111111111111111112"}` . Check the field `PriceInUSD` for the USD value. You can access the query [here](https://ide.bitquery.io/Get-Latest-Price-of-SOL-in--USD-Real-time).

```
subscription {
  Solana {
    DEXTradeByTokens(
      where: {Trade: {Currency: {MintAddress: {is: "So11111111111111111111111111111111111111112"}}}}
    ) {
      Transaction {
        Signature
      }
      Trade {
        AmountInUSD
        Amount
        Currency {
          MintAddress
          Name
        }
        Dex {
          ProgramAddress
          ProtocolName
        }
        Price
        PriceInUSD
        Side {
          Account {
            Address
          }
          AmountInUSD
          Amount
          Currency {
            Name
            MintAddress
          }
        }
      }
    }
  }
}


```

## Solana Instructions API

This Solana API is part of our Early Access Program (EAP), which is intended for evaluation purposes.

This program allows you to test the data and its integration into your applications before full-scale implementation. Read more [here](https://docs.bitquery.io/docs/graphql/dataset/EAP/)

### Latest Solana Instructions[​](https://docs.bitquery.io/docs/examples/Solana/solana-instructions/#latest-solana-instructions) <a href="#latest-solana-instructions" id="latest-solana-instructions"></a>

The subscription below fetches the latest instructions executed on the Solana blockchain including details like indices of preceding instructions signer, signature, balance updates, and program details

You can run the query [here](https://ide.bitquery.io/Latest-Solana-Instructions)

```
subscription {
  Solana(network: solana) {
    Instructions(limit: {count: 10}, orderBy: {descending: Block_Time}) {
      Transaction {
        Signer
        Signature
        Result {
          Success
          ErrorMessage
        }
        Index
      }
      Instruction {
        Logs
        BalanceUpdatesCount
        AncestorIndexes
        TokenBalanceUpdatesCount
        Program {
          Name
          Method
        }
      }
      Block {
        Time
        Hash
      }
    }
  }
}

```

### Latest Created Tokens on Solana[​](https://docs.bitquery.io/docs/examples/Solana/solana-instructions/#latest-created-tokens-on-solana) <a href="#latest-created-tokens-on-solana" id="latest-created-tokens-on-solana"></a>

The query below fetches the latest created tokens on the Solana blockchain including details like newly created token address which is the 1st entry in the Accounts array. We are querying Solana Token Program here with address `TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA` and filtering for latest created tokens using `Method: {in: ["initializeMint", "initializeMint2", "initializeMint3"]}`.

You can run the query [here](https://ide.bitquery.io/Get-newly-created-tokens-on-Solana\_1)

```
subscription {
  Solana {
    Instructions(
      where: {Instruction: {Program: {Address: {is: "TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA"}, Method: {in: ["initializeMint", "initializeMint2"]}}}, Transaction: {Result: {Success: true}}}
    ) {
      Instruction {
        Accounts {
          Address
          IsWritable
          Token {
            Mint
            Owner
            ProgramId
          }
        }
        Program {
          AccountNames
          Address
        }
      }
      Transaction {
        Signature
        Signer
      }
    }
  }
}



```

### Number of Latest Created Tokens on Solana[​](https://docs.bitquery.io/docs/examples/Solana/solana-instructions/#number-of-latest-created-tokens-on-solana) <a href="#number-of-latest-created-tokens-on-solana" id="number-of-latest-created-tokens-on-solana"></a>

The query below fetches the count of the latest created tokens on the Solana blockchain which were created using `initializeMint` method.

You can run the query [here](https://ide.bitquery.io/Count---Tokens-created-on-Solana)

```
query MyQuery {
  Solana(dataset: realtime, network: solana) {
    Instructions(
      where: {Instruction: {Program: {Address: {is: "TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA"}, Method: {is: "initializeMint"}}}}
      limit: {count: 10}
    ) {
      count
      Block {
        latest: Time(maximum: Block_Time)
        oldest: Time(minimum: Block_Time)
      }
    }
  }
}


```

### Get Authorities of tokens on Solana[​](https://docs.bitquery.io/docs/examples/Solana/solana-instructions/#get-authorities-of-tokens-on-solana) <a href="#get-authorities-of-tokens-on-solana" id="get-authorities-of-tokens-on-solana"></a>

The query below fetches all the authorities such as mint authority, freeze authority and update authority and also the token address on the Solana blockchain in realtime. The 1st entry in the Accounts array is the mint and the freeze authority, 2nd entry in the Accounts array is the token address, and 3rd entry in the Accounts array is the update authority.

You can run the query [here](https://ide.bitquery.io/get-authorities-of-tokens-on-solana\_2)

```
subscription{
  Solana {
    Instructions(
      where: {Instruction: {Program: {Address: {is: "metaqbxxUerdq28cj1RbAWkYQm3ybzjb6a8bt518x1s"}, Method: {is: "CreateMasterEditionV3"}}}}
    ) {
      Instruction {
        Accounts {
          Address
        }
        Program {
          Method
          AccountNames
          Address
          Name
        }
      }
    }
  }
}




```

### Track Real-time Token Burn on Solana[​](https://docs.bitquery.io/docs/examples/Solana/solana-instructions/#track-real-time-token-burn-on-solana) <a href="#track-real-time-token-burn-on-solana" id="track-real-time-token-burn-on-solana"></a>

Receive real-time updates on token burn events on the Solana blockchain. The below query applies a filter to only include instructions where Program Method includes `burn`, indicating that we filter instructions only related to token burning.

```

subscription {
  Solana {
    Instructions(
      where: {Instruction: {Program: {Method: {includes: "burn"}}}, Transaction: {Result: {Success: true}}}
    ) {
      Instruction {
        Accounts {
          Address
          IsWritable
          Token {
            Mint
            Owner
            ProgramId
          }
        }
        Program {
          AccountNames
          Address
          Name
          Method
        }
        Logs
      }
      Transaction {
        Signature
        Signer
      }
    }
  }
}
```

## Solana Transactions API

In this section we'll have a look at some examples using the Solana Transactions API.

This Solana API is part of our Early Access Program (EAP), which is intended for evaluation purposes.

This program allows you to test the data and its integration into your applications before full-scale implementation. Read more [here](https://docs.bitquery.io/docs/graphql/dataset/EAP/)

## Subscribe to Recent Transactions

The subscription query below fetches the most recent transactions on the Solana blockchain You can find the query [here](https://ide.bitquery.io/Realtime-Solana-Transactions)

```
subscription {
  Solana {
    Transactions(limit: {count: 10}) {
      Block {
        Time
        Hash
      }
      Transaction {
        BalanceUpdatesCount
        Accounts {
          Address
          IsWritable
        }
        Signer
        Signature
        Result {
          Success
          ErrorMessage
        }
        Index
        Fee
        TokenBalanceUpdatesCount
        InstructionsCount
      }
    }
  }
}

```

### Filtering Solana Transactions Based on Dynamic Criteria[​](https://docs.bitquery.io/docs/examples/Solana/solana-transactions/#filtering-solana-transactions-based-on-dynamic-criteria) <a href="#filtering-solana-transactions-based-on-dynamic-criteria" id="filtering-solana-transactions-based-on-dynamic-criteria"></a>

In this subscription query we will see how to set dynamic filters for the transactions to be retrieved based on various transaction properties. Developers can monitor specific types of transactions on the Solana network, such as high-volume or high-fee transactions.

You can run the query [here](https://ide.bitquery.io/Solana-tx-dynamic-filter)

**Variables**[**​**](https://docs.bitquery.io/docs/examples/Solana/solana-transactions/#variables)

* **`$network`**: Specifies the Solana network.
* **`$tx_filter`**: A filter object used to specify the criteria for transactions to retrieve.

```
subscription(
   $network: solana_network
   $tx_filter: Solana_Transaction_Filter
) {
  Solana(network: $network) {
    Transactions(where:$tx_filter) {
      Block {
        Time
        Hash
      }
      Transaction {
        BalanceUpdatesCount
        Accounts {
          Address
          IsWritable
        }
        Signer
        Signature
        Result {
          Success
          ErrorMessage
        }
        Index
        Fee
        TokenBalanceUpdatesCount
        InstructionsCount
      }
    }
  }
}
<!-- Parameters -->

{
  "network": "solana",
  "tx_filter":{"Transaction": {"TokenBalanceUpdatesCount": {"gt": 10}, "FeeInUSD": {"ge": "0.0010"}}}
}
```

## Solana Jupiter API

In this section, we'll show you how to access information about Jupiter data using Bitquery APIs.

This Solana API is part of our Early Access Program (EAP).

You can use this program to try out the data and see how it works with your applications before you fully integrate it. You can learn more about this [here](https://docs.bitquery.io/docs/graphql/dataset/EAP/).

### Latest Swaps on Jupiter[​](https://docs.bitquery.io/docs/examples/Solana/solana-jupiter-api/#latest-swaps-on-jupiter) <a href="#latest-swaps-on-jupiter" id="latest-swaps-on-jupiter"></a>

To retrieve the newest swaps happened on Jupiter Aggregator, we will utilize the Solana instructions API/Websocket.

We will specifically look for the latest instructions from Jupiter's program, identified by the program address `JUP6LkbZbjS1jKKwapdHNy74zcZ3tLUZoi5QNyVTaV4`.  Whenever a new swap is on Jupiter, it triggers the `sharedAccountsRoute` instructions. The tokens involved in the swap and adderesses to which tokens belong initially and the accounts involved in the route of the swap can be obtained from the Instructions API.

For instance, Index 7 and 8 represent the tokens A and B respectively involved in the swap, while Index 1 and 2 are the addresses to which the tokens B and A belong initially. Note that the indexing starts from 0.

You can run this query using this [link](https://ide.bitquery.io/Tokens-involved-in-Jupiter-swap-source-address-destination-address-DEX-involved\_2).

```
subscription {
  Solana {
    Instructions(
      where: {
        Instruction: {
          Program: {
            Address: { is: "JUP6LkbZbjS1jKKwapdHNy74zcZ3tLUZoi5QNyVTaV4" }
            Method: { is: "sharedAccountsRoute" }
          }
        }
        Transaction: { Result: { Success: true } }
      }
    ) {
      Transaction {
        Signature
      }
      Instruction {
        Program {
          Method
          AccountNames
          Address
        }
        Accounts {
          Address
          IsWritable
          Token {
            Mint
            Owner
            ProgramId
          }
        }
      }
    }
  }
}
```

### Track Latest Created Limit Orders on Jupiter[​](https://docs.bitquery.io/docs/examples/Solana/solana-jupiter-api/#track-latest-created-limit-orders-on-jupiter) <a href="#track-latest-created-limit-orders-on-jupiter" id="track-latest-created-limit-orders-on-jupiter"></a>

To track the latest created limit orders happened on Jupiter Limit Order Program, we will utilize the Solana instructions API/Websocket.

To know which instruction method is used for creating limit orders, [we use this API](https://ide.bitquery.io/Get-all-methods-of-Jupiter-Limit-Order-Program). We will specifically look for the latest instructions from Jupiter's Limit Order program, identified by the program address `jupoNjAxXgZ4rjzxzPMP4oxduvQsQtZzyknqvzYNrNu`.  Whenever a limit order is created on Jupiter, it triggers the `initializeOrder` instructions. The input mint token address, output mint token address, base address, maker address, reserve address, maker input account and makeroutput address involved in the transaction can be obtained from the Instructions API.

The above mentioned addresses can be seen in the response in Program Account Names and the addresses to these ordered names maps directly to addresses in the Accounts array. You can run this query using this [link](https://ide.bitquery.io/Latest-created-Limit-Order-on-Jupiter-in-realtime).

```
subscription {
  Solana {
    Instructions(
      where: {
        Instruction: {
          Program: {
            Method: { is: "initializeOrder" }
            Address: { is: "jupoNjAxXgZ4rjzxzPMP4oxduvQsQtZzyknqvzYNrNu" }
          }
        }
        Transaction: { Result: { Success: true } }
      }
    ) {
      Transaction {
        Signature
      }
      Instruction {
        Accounts {
          Address
          IsWritable
          Token {
            Mint
            Owner
            ProgramId
          }
        }
        AncestorIndexes
        BalanceUpdatesCount
        CallPath
        CallerIndex
        Data
        Depth
        ExternalSeqNumber
        InternalSeqNumber
        Index
        Logs
        Program {
          AccountNames
          Arguments {
            Name
            Type
            Value {
              ... on Solana_ABI_Integer_Value_Arg {
                integer
              }
              ... on Solana_ABI_String_Value_Arg {
                string
              }
              ... on Solana_ABI_Address_Value_Arg {
                address
              }
              ... on Solana_ABI_BigInt_Value_Arg {
                bigInteger
              }
              ... on Solana_ABI_Bytes_Value_Arg {
                hex
              }
              ... on Solana_ABI_Boolean_Value_Arg {
                bool
              }
              ... on Solana_ABI_Float_Value_Arg {
                float
              }
              ... on Solana_ABI_Json_Value_Arg {
                json
              }
            }
          }
        }
      }
      Block {
        Time
      }
    }
  }
}
```

### Track Latest Cancel Limit Order Transactions on Jupiter[​](https://docs.bitquery.io/docs/examples/Solana/solana-jupiter-api/#track-latest-cancel-limit-order-transactions-on-jupiter) <a href="#track-latest-cancel-limit-order-transactions-on-jupiter" id="track-latest-cancel-limit-order-transactions-on-jupiter"></a>

To track the latest cancel limit order transactions happened on Jupiter Limit Order Program, we will utilize the Solana instructions API/Websocket.

To know which instruction method is used for canceling limit orders, [we use this API](https://ide.bitquery.io/Get-all-methods-of-Jupiter-Limit-Order-Program). We will specifically look for the latest instructions from Jupiter's Limit Order program, identified by the program address `jupoNjAxXgZ4rjzxzPMP4oxduvQsQtZzyknqvzYNrNu`.  Whenever a limit order is canceled on Jupiter, it triggers the `cancelOrder` instructions. The input mint token address, maker address, reserve address and maker input account involved in the transaction can be obtained from the Instructions API.

The above mentioned addresses can be seen in the response in Program Account Names and the addresses to these ordered names maps directly to addresses in the Accounts array. You can run this query using this [link](https://ide.bitquery.io/Latest-Cancel-Limit-Order-Transactions-on-Jupiter-in-realtime).

```
subscription {
  Solana {
    Instructions(
      where: {
        Instruction: {
          Program: {
            Method: { is: "cancelOrder" }
            Address: { is: "jupoNjAxXgZ4rjzxzPMP4oxduvQsQtZzyknqvzYNrNu" }
          }
        }
        Transaction: { Result: { Success: true } }
      }
    ) {
      Transaction {
        Signature
      }
      Instruction {
        Accounts {
          Address
          IsWritable
          Token {
            Mint
            Owner
            ProgramId
          }
        }
        AncestorIndexes
        BalanceUpdatesCount
        CallPath
        CallerIndex
        Data
        Depth
        ExternalSeqNumber
        InternalSeqNumber
        Index
        Logs
        Program {
          AccountNames
          Arguments {
            Name
            Type
            Value {
              ... on Solana_ABI_Integer_Value_Arg {
                integer
              }
              ... on Solana_ABI_String_Value_Arg {
                string
              }
              ... on Solana_ABI_Address_Value_Arg {
                address
              }
              ... on Solana_ABI_BigInt_Value_Arg {
                bigInteger
              }
              ... on Solana_ABI_Bytes_Value_Arg {
                hex
              }
              ... on Solana_ABI_Boolean_Value_Arg {
                bool
              }
              ... on Solana_ABI_Float_Value_Arg {
                float
              }
              ... on Solana_ABI_Json_Value_Arg {
                json
              }
            }
          }
        }
      }
      Block {
        Time
      }
    }
  }
}
```

### Track Latest Cancel Expired Limit Order Transactions on Jupiter[​](https://docs.bitquery.io/docs/examples/Solana/solana-jupiter-api/#track-latest-cancel-expired-limit-order-transactions-on-jupiter) <a href="#track-latest-cancel-expired-limit-order-transactions-on-jupiter" id="track-latest-cancel-expired-limit-order-transactions-on-jupiter"></a>

To track the latest cancel expired limit order transactions happened on Jupiter Limit Order Program, we will utilize the Solana instructions API/Websocket.

To know which instruction method is used for canceling expired limit orders, [we use this API](https://ide.bitquery.io/Get-all-methods-of-Jupiter-Limit-Order-Program). We will specifically look for the latest instructions from Jupiter's Limit Order program, identified by the program address `jupoNjAxXgZ4rjzxzPMP4oxduvQsQtZzyknqvzYNrNu`.  Whenever a expired limit order is canceled on Jupiter, it triggers the `cancelExpiredOrder` instructions. You can run this query using this [link](https://ide.bitquery.io/Latest-Cancel-Expired-Order-Transactions-on-Jupiter-in-realtime\_1).

```
subscription {
  Solana {
    Instructions(
      where: {
        Instruction: {
          Program: {
            Method: { is: "cancelExpiredOrder" }
            Address: { is: "jupoNjAxXgZ4rjzxzPMP4oxduvQsQtZzyknqvzYNrNu" }
          }
        }
        Transaction: { Result: { Success: true } }
      }
    ) {
      Transaction {
        Signature
      }
      Instruction {
        Accounts {
          Address
          IsWritable
          Token {
            Mint
            Owner
            ProgramId
          }
        }
        AncestorIndexes
        BalanceUpdatesCount
        CallPath
        CallerIndex
        Data
        Depth
        ExternalSeqNumber
        InternalSeqNumber
        Index
        Logs
        Program {
          AccountNames
          Arguments {
            Name
            Type
            Value {
              ... on Solana_ABI_Integer_Value_Arg {
                integer
              }
              ... on Solana_ABI_String_Value_Arg {
                string
              }
              ... on Solana_ABI_Address_Value_Arg {
                address
              }
              ... on Solana_ABI_BigInt_Value_Arg {
                bigInteger
              }
              ... on Solana_ABI_Bytes_Value_Arg {
                hex
              }
              ... on Solana_ABI_Boolean_Value_Arg {
                bool
              }
              ... on Solana_ABI_Float_Value_Arg {
                float
              }
              ... on Solana_ABI_Json_Value_Arg {
                json
              }
            }
          }
        }
      }
      Block {
        Time
      }
    }
  }
}
```

## Pump Fun API

### Pump Fun Trades in Real-Time[​](https://docs.bitquery.io/docs/examples/Solana/Pump-Fun-API/#pump-fun-trades-in-real-time) <a href="#pump-fun-trades-in-real-time" id="pump-fun-trades-in-real-time"></a>

The below query gets real-time information whenever there's a new trade on the Pump.fun including program method called , buy and sell details, details of the currencies involved, and the transaction specifics like signature. You can run the query [here](https://ide.bitquery.io/Pumpfun-DEX-Trades\_1)

```
subscription MyQuery {
  Solana {
    DEXTrades(
      where: {
        Trade: { Dex: { ProtocolName: { is: "pump" } } }
        Transaction: { Result: { Success: true } }
      }
    ) {
      Instruction {
        Program {
          Method
        }
      }
      Trade {
        Dex {
          ProtocolFamily
          ProtocolName
        }
        Buy {
          Amount
          Account {
            Address
          }
          Currency {
            Name
            Symbol
            MintAddress
            Decimals
            Fungible
            Uri
          }
        }
        Sell {
          Amount
          Account {
            Address
          }
          Currency {
            Name
            Symbol
            MintAddress
            Decimals
            Fungible
            Uri
          }
        }
      }
      Transaction {
        Signature
      }
    }
  }
}
```

### Track New Token Creation on Pump Fun[​](https://docs.bitquery.io/docs/examples/Solana/Pump-Fun-API/#track-new-token-creation-on-pump-fun) <a href="#track-new-token-creation-on-pump-fun" id="track-new-token-creation-on-pump-fun"></a>

Get notified about the newly created token and its information on Pump Fun DEX for getting early bird benefits.

#### Get Pump Program Address and Method[​](https://docs.bitquery.io/docs/examples/Solana/Pump-Fun-API/#get-pump-program-address-and-method) <a href="#get-pump-program-address-and-method" id="get-pump-program-address-and-method"></a>

To run our desired query we need two parameters, that are Program Address and Method Name. To get these parameters, we can run the [following query](https://ide.bitquery.io/Get-Pump-Address-and-Method-Name?\_gl=1\*ygo5vg\*\_ga\*MTQwMzE3MzI0My4xNzEyNjUzNzMw\*\_ga\_ZWB80TDH9J\*MTcxODg4MTYwOS4xMzMuMS4xNzE4ODgxNzg3LjAuMC4w).

```
query MyQuery {
  Solana {
    Instructions(
      where: { Instruction: { Program: { Name: { is: "pump" } } } }
    ) {
      Instruction {
        Program {
          Method
          Address
        }
      }
      count
    }
  }
}
```

[Here](https://ide.bitquery.io/Track-new-token-launches-on-Pump-Fun-in-realtime) is the subscription to get the notification of new token creation event on Pump Fun.

```
query MyQuery {
  Solana {
    Instructions(
      where: { Instruction: { Program: { Name: { is: "pump" } } } }
    ) {
      Instruction {
        Program {
          Method
          Address
        }
      }
      count
    }
  }
}
```

### Get OHLC Data of a Token on Pump Fun[​](https://docs.bitquery.io/docs/examples/Solana/Pump-Fun-API/#get-ohlc-data-of-a-token-on-pump-fun) <a href="#get-ohlc-data-of-a-token-on-pump-fun" id="get-ohlc-data-of-a-token-on-pump-fun"></a>

The below query gets OHLC data of the specified Token `66VR6bjEV5DPSDhYSQyPAxNsY3dgmH6Lwgi5cyf2pump` for 1 minute time interval for last 10 minutes on Pump Fun DEX. You can run the query [here](https://ide.bitquery.io/OHLC-for-a-token-on-Pump-Fun\_3)

Note - You can only use this API using `query` keyword, using this API as `subscription` will give wrong results because aggregation and interval don't work correctly together in `subscription`.

```
query {
  Solana {
    DEXTradeByTokens(
      limit: { count: 10 }
      orderBy: { descendingByField: "Block_Timefield" }
      where: {
        Trade: {
          Currency: {
            MintAddress: { is: "66VR6bjEV5DPSDhYSQyPAxNsY3dgmH6Lwgi5cyf2pump" }
          }
          Dex: {
            ProgramAddress: {
              is: "6EF8rrecthR5Dkzon8Nwu78hRvfCKubJ14M5uBEwF6P"
            }
          }
        }
      }
    ) {
      Block {
        Timefield: Time(interval: { in: minutes, count: 1 })
      }
      volume: sum(of: Trade_Amount)
      Trade {
        high: Price(maximum: Trade_Price)
        low: Price(minimum: Trade_Price)
        open: Price(minimum: Block_Slot)
        close: Price(maximum: Block_Slot)
      }
      count
    }
  }
}
```

### Track Price of a Token in Realtime on Pump Fun[​](https://docs.bitquery.io/docs/examples/Solana/Pump-Fun-API/#track-price-of-a-token-in-realtime-on-pump-fun) <a href="#track-price-of-a-token-in-realtime-on-pump-fun" id="track-price-of-a-token-in-realtime-on-pump-fun"></a>

The below query gets real-time price of the specified Token `JDm62GykLKugcWiW8wKNEXurhyGk3meE8hM8v7Ytpump` on the Pump Fun DEX. You can run the query [here](https://ide.bitquery.io/Realtime-Price-of-a-token)

```
subscription {
  Solana {
    DEXTrades(
      where: {
        Trade: {
          Dex: {
            ProgramAddress: {
              is: "6EF8rrecthR5Dkzon8Nwu78hRvfCKubJ14M5uBEwF6P"
            }
          }
          Buy: {
            Currency: {
              MintAddress: {
                is: "JDm62GykLKugcWiW8wKNEXurhyGk3meE8hM8v7Ytpump"
              }
            }
          }
        }
      }
    ) {
      Block {
        Time
      }
      Trade {
        Buy {
          Account {
            Address
          }
          Amount
          AmountInUSD
          Currency {
            MintAddress
            Name
            Symbol
          }
          Price
          PriceInUSD
        }
        Dex {
          ProtocolName
          ProtocolFamily
          ProgramAddress
        }
        Market {
          MarketAddress
        }
        Sell {
          Account {
            Address
          }
          Amount
          AmountInUSD
          Currency {
            MintAddress
            Name
            Symbol
          }
          Price
          PriceInUSD
        }
      }
      Transaction {
        Signature
      }
    }
  }
}
```

### Get the Token Holders of a specific Token[​](https://docs.bitquery.io/docs/examples/Solana/Pump-Fun-API/#get-the-token-holders-of-a-specific-token) <a href="#get-the-token-holders-of-a-specific-token" id="get-the-token-holders-of-a-specific-token"></a>

The below query gets top 10 token holders of the specified Token `HeGMgxcuASNEgGH8pTUBEfb3K4KjgaXwaMK3bs68pump` on the Pump Fun DEX. Keep in mind you can use this API only as a query and not a subscription websocket because aggregates don't work with subscription and you will end up getting wrong results. You can run the query [here](https://ide.bitquery.io/top-10-holders-for-a-pump-fun-token\_3)

```
query MyQuery {
  Solana {
    BalanceUpdates(
      limit: { count: 10 }
      orderBy: { descendingByField: "TotalHolding" }
      where: {
        BalanceUpdate: {
          Currency: {
            MintAddress: { is: "HeGMgxcuASNEgGH8pTUBEfb3K4KjgaXwaMK3bs68pump" }
          }
        }
      }
    ) {
      BalanceUpdate {
        Currency {
          Name
          MintAddress
          Symbol
        }
        Account {
          Address
        }
      }
      TotalHolding: sum(of: BalanceUpdate_Amount, selectWhere: { gt: "0" })
    }
  }
}
```

### Get the Trading Volume of a specific Token on Pump Fun DEX[​](https://docs.bitquery.io/docs/examples/Solana/Pump-Fun-API/#get-the-trading-volume-of-a-specific-token-on-pump-fun-dex) <a href="#get-the-trading-volume-of-a-specific-token-on-pump-fun-dex" id="get-the-trading-volume-of-a-specific-token-on-pump-fun-dex"></a>

The below query gets the Trading volume of the specified Token `HeGMgxcuASNEgGH8pTUBEfb3K4KjgaXwaMK3bs68pump` on the Pump Fun DEX in the past 1 hour. You will have to change the time in this `Block: { Time: { since: "2024-06-27T06:46:00Z" } }` when you try the query yourself. Keep in mind you can use this API only as a query and not a subscription websocket because aggregates don't work with subscription and you will end up getting wrong results. You can run the query [here](https://ide.bitquery.io/Realtime-Price-of-a-token)

```
query MyQuery {
  Solana {
    DEXTradeByTokens(
      where: {
        Trade: {
          Currency: {
            MintAddress: { is: "HeGMgxcuASNEgGH8pTUBEfb3K4KjgaXwaMK3bs68pump" }
          }
          Dex: { ProtocolName: { is: "pump" } }
        }
        Block: { Time: { since: "2024-06-27T06:46:00Z" } }
      }
    ) {
      Trade {
        Currency {
          Name
          Symbol
          MintAddress
        }
        Dex {
          ProtocolName
          ProtocolFamily
        }
      }
      TradeVolume: sum(of: Trade_Amount)
    }
  }
}
```

## Jito Bundle API

In this section, we will show you how to access information about Jito Bundles using Bitquery APIs.

This Solana API is part of our Early Access Program (EAP).

You can use this program to try out the data and see how it works with your applications before you fully integrate it. You can learn more about this [here](https://docs.bitquery.io/docs/graphql/dataset/EAP/).

### Transfers to the Tip Payment Accounts[​](https://docs.bitquery.io/docs/examples/Solana/Solana-Jito-Bundle-api/#transfers-to-the-tip-payment-accounts) <a href="#transfers-to-the-tip-payment-accounts" id="transfers-to-the-tip-payment-accounts"></a>

Jito foundation has Tip Payment Program that allows users to transfer tips to a set of static public keys (compared to signing the transaction with the next N leaders) and ensure that the incentives are distributed to the correct block leader, while enabling bundles to execute in upto 8 parallel threads.

The subscription that provides you the transfer data of one of these addresses is [writen below](https://ide.bitquery.io/Transfers-of-Tip-Payment-Accounts-on-Solana\_1). To get the data of all addresses you can use the [this](https://ide.bitquery.io/Transfers-of-All-Tip-Payment-Accounts-on-Solana) query.

```
subscription {
  Solana {
    Recieved: Transfers(
      where: {Transfer: {Receiver: {Address: {is: "HFqU5x63VTqvQss8hp11i4wVV8bD44PvwucfZ2bU7gRe"}}}}
    ) {
      Transfer {
        Currency {
          Name
          Symbol
          MintAddress
        }
        AmountInUSD
        Sender {
          Address
        }
      }
      Transaction {
        Signature
      }
      Block {
        Time
        Slot
      }
    }
  }
}
```

### List of methods for Jito Merkle Upload Authority[​](https://docs.bitquery.io/docs/examples/Solana/Solana-Jito-Bundle-api/#list-of-methods-for-jito-merkle-upload-authority) <a href="#list-of-methods-for-jito-merkle-upload-authority" id="list-of-methods-for-jito-merkle-upload-authority"></a>

Jito Merkle Upload Autjority account is the block builder that uploads the root of Merkle tree created by processing the MEV data in an offchain setting.

The [below query](https://ide.bitquery.io/All-Methods-for-Jito-Bundles-on-Solana\_1) returns all the methods that are accessible to this account.

```
{
  Solana {
    Instructions(
      where: {Transaction: {Signer: {is: "GZctHpWXmsZC1YHACTGGcHhYxjdRqQvTpYkb9LMvxDib"}}}
    ) {
      Instruction {
        Program {
          Address
          Method
          Name
        }
      }
      count
    }
  }
}

```

### More information on Transfer Method[​](https://docs.bitquery.io/docs/examples/Solana/Solana-Jito-Bundle-api/#more-information-on-transfer-method) <a href="#more-information-on-transfer-method" id="more-information-on-transfer-method"></a>

From the above query, we'll get a list of methods that the account is signing. One of those method is "Transfer".

To get more info on the "Tranfer" method we can run the query given [below](https://ide.bitquery.io/Transfer-Function-Call-Event-Alert-for-Jito-Bundles-on-Solana\_1).

```
{
  Solana {
    Instructions(
      limit: {count:10},
      where: {Transaction: {Signer: {is: "GZctHpWXmsZC1YHACTGGcHhYxjdRqQvTpYkb9LMvxDib"}, Result: {Success: true}}, Instruction: {Program: {Method: {is: "Transfer"}}}}
    ) {
      Instruction {
        Program {
          Address
          Method
          Name
          Arguments {
            Name
            Type
            Value {
              ... on Solana_ABI_Integer_Value_Arg {
                integer
              }
              ... on Solana_ABI_String_Value_Arg {
                string
              }
              ... on Solana_ABI_Address_Value_Arg {
                address
              }
              ... on Solana_ABI_Json_Value_Arg {
                json
              }
              ... on Solana_ABI_Float_Value_Arg {
                float
              }
              ... on Solana_ABI_Boolean_Value_Arg {
                bool
              }
              ... on Solana_ABI_Bytes_Value_Arg {
                hex
              }
              ... on Solana_ABI_BigInt_Value_Arg {
                bigInteger
              }
            }
          }
        }
      }
      Block {
        Slot,
        Time
      }
      Transaction {
        Signature
      }
    }
  }
}
```

## Meteora DEX API

### Meteora Trades in Real-Time[​](https://docs.bitquery.io/docs/examples/Solana/Solana-Meteora-api/#meteora-trades-in-real-time) <a href="#meteora-trades-in-real-time" id="meteora-trades-in-real-time"></a>

The below query gets real-time information whenever there's a new trade on the Meteora DEX including detailed information about the trade, including the buy and sell details, the block information, and the transaction specifics. You can run the query [here](https://ide.bitquery.io/Real-time-trades-on-Meteora-DEX-on-Solana\_1)

```
subscription {
  Solana {
    DEXTrades(
      where: {Trade: {Dex: {ProtocolFamily: {is: "Meteora"}}}}
    ) {
      Trade {
        Dex {
          ProgramAddress
          ProtocolFamily
          ProtocolName
        }
        Buy {
          Currency {
            Name
            Symbol
            MintAddress
          }
          Amount
          Account {
            Address
          }
          PriceAgainstSellCurrency:Price
        }
        Sell {
          Account {
            Address
          }
          Amount
          Currency {
            Name
            Symbol
            MintAddress
          }
          PriceAgainstBuyCurrency: Price
        }
      }
      Block {
        Time
      }
    }
  }
}

```

### Volatility of a Pair on Meteora[​](https://docs.bitquery.io/docs/examples/Solana/Solana-Meteora-api/#volatility-of-a-pair-on-meteora) <a href="#volatility-of-a-pair-on-meteora" id="volatility-of-a-pair-on-meteora"></a>

Volatility is an important factor in trading world as it determines the fluctuation in price that implies the possibility of profit and risk of loss. Lesser volatility denotes that the pair is stable.

[Here](https://ide.bitquery.io/Volatility-of-WSOL-USDC-Pair-on-Meteora-Dex-on-Solana\_3) is the query to get the volatility for a selected pair in the last 24 hours.

```
query VolatilityonMeteora {
  Solana {
    DEXTrades(
      where: {
        Trade: {
          Dex: {ProtocolFamily: {is: "Meteora"}},
          Buy: {Currency: {MintAddress: {is: "So11111111111111111111111111111111111111112"}}},
          Sell: {Currency: {MintAddress: {is: "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v"}}}
        },
        Block: {
          Time: {
            after: "2024-06-04T00:00:00Z", 
            before: "2024-06-05T00:00:00Z"
          }
        }
      }
    ) {
      volatility:standard_deviation(of: Trade_Buy_Price)
    }
  }
}
```

## Solana Logs API

Solana Logs API helps you filter program instruction logs using regular expressions.

This Solana API is part of our Early Access Program (EAP), which is intended for evaluation purposes.

This program allows you to test the data and its integration into your applications before full-scale implementation. Read more [here](https://docs.bitquery.io/docs/graphql/dataset/EAP/)

## Finding Instructions with Log Matching

The Solana Logs API allows you to search for specific instructions based on log content that matches exact phrases. For example, to find logs related to 'AnchorError' with a specific error code and message You can find the query [here](https://ide.bitquery.io/SlippageToleranceExceeded)

```
{
  Solana {
    Instructions(
      limit: {count: 1}
      where: {Instruction: {Logs: {includes: {is: "Program log: AnchorError occurred. Error Code: SlippageToleranceExceeded. Error Number: 6001. Error Message: Slippage tolerance exceeded."}}}}
    ) {
      Instruction {
        Accounts {
          Address
        }
        Data
        Index
        Logs
        ExternalSeqNumber
        Program {
          Json
          AccountNames
          Method
          Name
          Arguments {
            Name
            Type
          }
        }
        Index
      }
      Transaction {
        Result {
          ErrorMessage
        }
      }
    }
  }
}

```

### Filtering Instructions using Not Like Filter[​](https://docs.bitquery.io/docs/examples/Solana/solana-logs/#filtering-instructions-using-not-like-filter) <a href="#filtering-instructions-using-not-like-filter" id="filtering-instructions-using-not-like-filter"></a>

To exclude instructions containing specific log phrases such as 'AnchorError' you can use the `notLike` filter.

You can find the query [here](https://ide.bitquery.io/Not-Anchor-Error-Solana-Logs)

```
{
  Solana {
    Instructions(
      limit: {count: 1}
      where: {Instruction: {Logs: {includes: {notLike: "Program log: AnchorError occurred."}}}}
    ) {
      Instruction {
        Accounts {
          Address
        }
        Data
        Index
        Logs
        ExternalSeqNumber
        Program {
          Json
          AccountNames
          Method
          Name
          Arguments {
            Name
            Type
          }
        }
        Index
      }
      Transaction {
        Result {
          ErrorMessage
        }
      }
    }
  }
}

```

### Finding Instructions using Like Filter[​](https://docs.bitquery.io/docs/examples/Solana/solana-logs/#finding-instructions-using-like-filter) <a href="#finding-instructions-using-like-filter" id="finding-instructions-using-like-filter"></a>

To find instructions based on logs that contain specific patterns or keywords, such as an invoke log, you can use the `like` filter.

```
{
  Solana {
    Instructions(
      limit: {count: 1}
      where: {Instruction: {Logs: {includes: {like: "Program Vote111111111111111111111111111111111111111 invoke [1]"}}}}
    ) {
      Instruction {
        Accounts {
          Address
        }
        Data
        Index
        Logs
        ExternalSeqNumber
        Program {
          Json
          AccountNames
          Method
          Name
          Arguments {
            Name
            Type
          }
        }
        Index
      }
      Transaction {
        Result {
          ErrorMessage
        }
      }
    }
  }
}

```

### Searching Logs using a Particular Keyword[​](https://docs.bitquery.io/docs/examples/Solana/solana-logs/#searching-logs-using-a-particular-keyword) <a href="#searching-logs-using-a-particular-keyword" id="searching-logs-using-a-particular-keyword"></a>

To find instructions based on logs that contain specific patterns or keywords, such as an ZETA market log, you can use the `includes` filter which searches for the presence of the keyword as a whole in the log. You can run the query [here](https://ide.bitquery.io/Solana-Zeta-Market-logs)

```
{
  Solana {
    Instructions(
      where: {Instruction: {Logs: {includes: {includes: "ZETA"}}}}
      limit: {count: 10}
    ) {
      Transaction {
        Signature
      }
      Instruction {
        Logs
      }
    }
  }
}
```

## Solana Transfers API

In this section we'll have a look at some examples using the Solana Transfers API.

This Solana API is part of our Early Access Program (EAP), which is intended for evaluation purposes.

This program allows you to test the data and its integration into your applications before full-scale implementation. Read more [here](https://docs.bitquery.io/docs/graphql/dataset/EAP/)

### Subscribe to the latest NFT token transfers on Solana[​](https://docs.bitquery.io/docs/examples/Solana/solana-transfers/#subscribe-to-the-latest-nft-token-transfers-on-solana) <a href="#subscribe-to-the-latest-nft-token-transfers-on-solana" id="subscribe-to-the-latest-nft-token-transfers-on-solana"></a>

Let's see an example of NFT token transfers using GraphQL Subscription (Webhook). In the following API, we will be subscribing to all NFT token transfers. You can run the query [here](https://ide.bitquery.io/Subscribe-to-the-latest-NFT-transfers-on-Solana)

```
subscription {
  Solana {
    Transfers(where: {Transfer: {Currency: {Fungible: false}}}) {
      Transfer {
        Amount
        AmountInUSD
        Currency {
          Name
          MintAddress
          Fungible
          Symbol
          Uri
        }
        Receiver {
          Address
        }
        Sender {
          Address
        }
      }
      Transaction {
        Signature
      }
    }
  }
}


```

### SPL Token Transfers API | Token transfers of a particular token on Solana[​](https://docs.bitquery.io/docs/examples/Solana/solana-transfers/#spl-token-transfers-api--token-transfers-of-a-particular-token-on-solana) <a href="#spl-token-transfers-api--token-transfers-of-a-particular-token-on-solana" id="spl-token-transfers-api--token-transfers-of-a-particular-token-on-solana"></a>

One of the most common types of transfers on Solana are SPL token transfers. Let's see an example to get the latest SPL token transfers using our API. Today we are taking an example of JUPITER token transfers. The contract address for the JUPITER token is `JUPyiwrYJFskUPiHa7hkeR8VUtAeFoSYbKedZNsDvCN`. You can find the query [here](https://ide.bitquery.io/SPL-transfers-websocket\_1)

```
subscription {
  Solana {
    Transfers(
      where: {Transfer: {Currency: {MintAddress: {is: "JUPyiwrYJFskUPiHa7hkeR8VUtAeFoSYbKedZNsDvCN"}}}}
    ) {
      Transfer {
        Currency {
          MintAddress
          Symbol
          Name
          Fungible
          Native
        }
        Receiver {
          Address
        }
        Sender {
          Address
        }
        Amount
        AmountInUSD
      }
    }
  }
}



```

### Transfers sent by specific address[​](https://docs.bitquery.io/docs/examples/Solana/solana-transfers/#transfers-sent-by-specific-address) <a href="#transfers-sent-by-specific-address" id="transfers-sent-by-specific-address"></a>

This websocket retrieves transfers where the sender is a particular address `2g9NLWUM6bPm9xq2FBsb3MT3F3G5HDraGqZQEVzcCWTc`. For this subscription query we use `where` keyword and in that we specify `{Transfer: {Sender: {Address: {is: "2g9NLWUM6bPm9xq2FBsb3MT3F3G5HDraGqZQEVzcCWTc"}}}}` to get the desired data. You can find the query [here](https://ide.bitquery.io/transfers-where-sender-is-the-specified-address\_1)

```
subscription {
  Solana {
    Transfers(
      where: {Transfer: {Sender: {Address: {is: "2g9NLWUM6bPm9xq2FBsb3MT3F3G5HDraGqZQEVzcCWTc"}}}}
    ) {
      Transaction {
        Signature
      }
      Transfer {
        Amount
        AmountInUSD
        Sender {
          Address
        }
        Receiver {
          Address
        }
        Currency {
          Name
          Symbol
          MintAddress
        }
      }
    }
  }
}

```

### Monitor multiple solana wallets transfers in real time using Websocket[​](https://docs.bitquery.io/docs/examples/Solana/solana-transfers/#monitor-multiple-solana-wallets-transfers-in-real-time-using-websocket) <a href="#monitor-multiple-solana-wallets-transfers-in-real-time-using-websocket" id="monitor-multiple-solana-wallets-transfers-in-real-time-using-websocket"></a>

You can also monitor multiple wallet addresses using Bitquery's GraphQL subscription via WebSocket. The following query listens to real-time transfers sent or received by the specified wallet addresses.

You can include around 100 addresses in a single subscription, potentially more, as we typically do not throttle this on our end.

However, if you need to track thousands of addresses, you can subscribe to all transfers and filter them on your side.

Websockets are priced based on their running time, not the amount of data delivered.

Run query using [this link](https://ide.bitquery.io/Solana-Websocket---Subscribe-to-all-transfers-of-specific-addresses-in-realtime)

```
subscription {
  Solana {
    Transfers(
      where: {any: [{Transfer: {Sender: {Address: {in: ["7Ppgch9d4XRAygVNJP4bDkc7V6htYXGfghX4zzG9r4cH", "G6xptnrkj4bxg9H9ZyPzmAnNsGghSxZ7oBCL1KNKJUza"]}}}}, {Transfer: {Receiver: {Address: {in: ["7Ppgch9d4XRAygVNJP4bDkc7V6htYXGfghX4zzG9r4cH", "G6xptnrkj4bxg9H9ZyPzmAnNsGghSxZ7oBCL1KNKJUza"]}}}}]}
    ) {
      Transfer {
        Amount
        AmountInUSD
        Authority {
          Address
        }
        Currency {
          Decimals
          CollectionAddress
          Fungible
          MetadataAddress
          MintAddress
          Name
          Native
          Symbol
        }
        Receiver {
          Address
        }
        Sender {
          Address
        }
      }
    }
  }
}
```

## Difference between Bitquery GraphQL V1 and V2 APIs

Today we will explain the difference between our V1 and V2 APIs. Let’s start with naming; we call our V2 APIs as Streaming APIs. Additionally, there is also a difference in the endpoint.

If you have any questions please comment on [this community post](https://community.bitquery.io/t/difference-between-bitquery-graphql-v1-and-v2-apis/1561).

### Endpoints[​](https://docs.bitquery.io/v1/docs/graphql-ide/v1-and-v2#endpoints) <a href="#endpoints" id="endpoints"></a>

V1 — [https://graphql.bitquery.io/](https://graphql.bitquery.io/) V2 — [https://streaming.bitquery.io/graphql](https://streaming.bitquery.io/graphql)

If you want to access V1 on IDE, please use [this link](https://ide.bitquery.io/), and for V2, please use [this link](https://streaming.bitquery.io/).

Also, Here are [examples of V1](https://ide.bitquery.io/explore/ethereum) and [getting a starter guide](https://community.bitquery.io/t/how-to-get-started-with-bitquerys-blockchain-graphql-apis/13). Additionally, check out [examples of V2](https://ide.bitquery.io/explore/EVM) and [V2 docs](https://docs.bitquery.io/).

Now, let’s talk about a few features which we added in new Streaming APIs (V2).

### 1. Real-time APIs[​](https://docs.bitquery.io/v1/docs/graphql-ide/v1-and-v2#1-real-time-apis) <a href="#id-1-real-time-apis" id="id-1-real-time-apis"></a>

As the name suggests, V2 APIs are designed to provide [real-time blockchain data without any delay](https://bitquery.io/blog/analysis-of-blockchain-availabilitybased-on-block-lag).

It combines both real-time and historical data. Therefore, provides a seamless view of querying blockchains with real-time updates.

### 2. WebSocket support[​](https://docs.bitquery.io/v1/docs/graphql-ide/v1-and-v2#2-websocket-support) <a href="#id-2-websocket-support" id="id-2-websocket-support"></a>

Streaming API support WebSockets through [GraphQL Subscriptions](https://graphql.org/blog/subscriptions-in-graphql-and-relay/), it’s an [event-based data subscription](https://docs.bitquery.io/docs/ide/subscription/) where you can write a query, and whenever a query has new data, it will be pushed to you. Because of this, you can now access real-time blockchain updates.

GraphQL subscriptions are very useful for building notification services.

### 3. New blockchains[​](https://docs.bitquery.io/v1/docs/graphql-ide/v1-and-v2#3-new-blockchains) <a href="#id-3-new-blockchains" id="id-3-new-blockchains"></a>

V1 APIs support [40+ blockchains](https://account.bitquery.io/user/system\_status). However, Streaming APIs currently have Ethereum, Binance Smart Chain, and **Arbitrum.**

We are adding Optimism and slowly migrating more chains from V1 API in which Real timeliness becomes critical for a better user experience.

### 4. Token holder API[​](https://docs.bitquery.io/v1/docs/graphql-ide/v1-and-v2#4-token-holder-api) <a href="#id-4-token-holder-api" id="id-4-token-holder-api"></a>

Streaming APIs have [BalanceUpdates](https://docs.bitquery.io/docs/evm/balances/) APIs which allow you to aggregate data to get token balances or holders.

You can get them for any time in the past for NFTs and fungible tokens. Additionally, you can get the holders with over a specific balance.

Here is an example of [token holder API](https://docs.bitquery.io/docs/examples/balances/tokenHolders-api/).

### 5. NFT support[​](https://docs.bitquery.io/v1/docs/graphql-ide/v1-and-v2#5-nft-support) <a href="#id-5-nft-support" id="id-5-nft-support"></a>

One of the major improvements we brought in V2 APIs is [NFT APIs](https://bitquery.io/products/nft-apis).

Now you can query all sorts of NFT data, such as Ownership, Metadata, Transfers, trades, origin & history, using our APIs.

Check out our [Opensea](https://bitquery.io/blog/opensea-nft-api) and [Blur APIs](https://bitquery.io/blog/blur-nft-marketplace-api).

### 6. Call data and Trace APIs[​](https://docs.bitquery.io/v1/docs/graphql-ide/v1-and-v2#6-call-data-and-trace-apis) <a href="#id-6-call-data-and-trace-apis" id="id-6-call-data-and-trace-apis"></a>

Many times our users look for [trace data](https://community.bitquery.io/t/bitquery-trace-api/1556) with raw transaction data for various reasons, including data verifiability.

We now support much richer call data with path and argument details, opcodes, and raw data in our V2 APIs.

See examples of our [Trace APIs](https://ide.bitquery.io/Transaction-Call-Trace-v2\_1).

### 7. Argument filtering[​](https://docs.bitquery.io/v1/docs/graphql-ide/v1-and-v2#7-argument-filtering) <a href="#id-7-argument-filtering" id="id-7-argument-filtering"></a>

This is our game-changing feature in which you can query the Arguments of a smart contract like a Mysql table.

You can check data for specific argument values, aggregate argument values, etc.

Check our [Blur marketplace API](https://bitquery.io/blog/blur-nft-marketplace-api) article to see the usage of this feature.

### What’s Next?[​](https://docs.bitquery.io/v1/docs/graphql-ide/v1-and-v2#whats-next) <a href="#whats-next" id="whats-next"></a>

We are very excited to see enable more features in our Streaming APIs. Here are a few things which we will enable in the upcoming weeks.

* **Mempool data** — We already have Mempool data, and we are going to enable it soon in our V2 APIs.
* **Cross-chain queries** — We are going to add Cross chain queries functionality where you can query multiple blockchains using 1 query.
* **More blockchains** — We are adding Optimism and going to add more blockchains with high throughput.
* [**Data in the cloud** ](https://bitquery.io/products/streaming)— This is our new product, which uses streaming internally, using which we will allow complete historical and Real-time data through AWS, Snowflake, Google, and other Cloud vendors.

## Address

The Solana address API gives you information of the balance of a wallet in SOL.

```
query MyQuery {
  solana(network: solana) {
    address(address: {is: "address of the wallet"}) {
      address
      balance
      annotation
    }
  }
}


```

<details>

<summary>Filtering Address</summary>

`address`: Specify the address of the wallet. The `is` keyword is used to specify that the address must match the value that is provided.

</details>

### Fields[​](https://docs.bitquery.io/v1/docs/Schema/solana/address#fields) <a href="#fields" id="fields"></a>

`address`

The address field specifies the address of the account that you want to get the balance for.

`annotation`

Any label on the account.

`balance`

The balance of the account in SOL.

The balance of the account.

## Coinpath

The Solana Coinpath API allows you to get the money flow for an address on the Solana blockchain. You can track any levels of fund movement with this API. This is a very useful API for crypto investigations.

```
query ($network: SolanaNetwork!, $address: String!, $from: ISO8601DateTime, $till: ISO8601DateTime) {
  solana(network: $network) {
    inbound: coinpath(
      initialAddress: {is: $address}
      depth: {lteq: 1}
      options: {direction: inbound, asc: "depth", desc: "amount", limitBy: {each: "depth", limit: 10}}
      date: {since: $from, till: $till}
    ) {
      sender {
        address
        annotation
      }
      receiver {
        address
        annotation
      }
      amount
      currency {
        symbol
        name
        address
        decimals
        tokenId
        tokenType
      }
      depth
      count
      signature {
        value
        hash
      }
    }
    outbound: coinpath(
      initialAddress: {is: $address}
      depth: {lteq: 1}
      options: {asc: "depth", desc: "amount", limitBy: {each: "depth", limit: 10}}
      date: {since: $from, till: $till}
    ) {
      sender {
        address
        annotation
      }
      receiver {
        address
        annotation
      }
      amount
      currency {
        symbol
        name
        address
        decimals
        tokenId
        tokenType
      }
      depth
      count
      signature {
        value
        hash
      }
    }
  }
}

<!-- Parameters -->

{
  "network": "solana",
  "address": "aGhLX8kiZ2sWbcdpqwbGMHL1PLBY8Xe3srCFv8aoFbv",
  "from": "2023-07-31",
  "till": "2023-08-07T23:59:59",
  "dateFormat": "%Y-%m-%d"
}

```

<details>

<summary>Filtering Coinpath</summary>

*   **initialAddress**

    The address of the account whose coinpath you want to get. The filter can be used to specify the initial address of the coinpath.
*   **depth**

    The maximum depth of the coinpath. The filter can be used to specify the maximum depth of the fund flow either inbound and outbound.
*   **options**

    A set of options that can be used to filter the results. The following options are supported:

    * **direction**
      * The direction of the coinpath. The supported directions are `inbound` and `outbound`.
    * **asc**
      * The order of the coinpath. The supported orders are `asc` and `desc`.
    * **limitBy**
      * The limit of the coinpath. The limit is the maximum number of accounts that will be included in the coinpath.
    * **minimumTxAmount**
      * The minimum amount of funds that must be transferred in a transaction.
    * **maximumAddressTxCount**
      * The maximum number of transactions that can be included for a single address.
    * **maximumTotalTxCount**
      * The maximum number of transactions that can be included in the coinpath.
    * **offset**
      * The offset of the coinpath. The offset is the number of accounts that will be skipped at the beginning of the coinpath.
    * **seed**
      * The seed of the coinpath. The seed is a random number that can be used to randomize the order of the coinpath.
*   **date**

    The date range of the coinpath. The filter can be used to specify the start date and end date of the coinpath.
*   **receiver**

    The address of the receiver of the transfer. The filter can be used to specify the receiver address of the coinpath.
*   **time**

    The timestamp of the transfer. The filter can be used to specify the timestamp of the coinpath.
*   **sender**

    The address of the sender of the transfer. The filter can be used to specify the sender address of the coinpath.
*   **currency**

    The currency of the transfer. The filter can be used to specify the currency of the coinpath.
*   **initialTime**

    The timestamp of the initial transaction. The filter can be used to specify the timestamp of the initial transaction.
*   **initialDate**

    The date of the initial transaction. The filter can be used to specify the date of the initial transaction.

</details>

### Fields[​](https://docs.bitquery.io/v1/docs/Schema/solana/coinpath#fields) <a href="#fields" id="fields"></a>

`sender`

The address of the sender of the transfer.

`receiver`

The address of the receiver of the transfer.

`amount`

The amount of funds that was transferred.

`currency`

The currency that was transferred.

`depth`

The depth of the connection between the two addresses.

`signature`

The signature of the transfer.

## InstructionAccounts

This API returns information about the accounts involved in an instruction. An instruction is the smallest execution logic on Solana. One transaction can have 1 or more instructions. Below are the fields in the API:

```
query ($network: SolanaNetwork!, $signature: String!) {
  solana(network: $network) {
    instructionAccounts(signature: {is: $signature}) {
      instruction {
        action {
          name
        }
        program {
          parsedName
        }
      }
      account {
        index
        name
        owner
        type
      }
      block {
        height
        hash
        previousBlockHash
        timestamp {
          time
        }
      }
      transaction {
        feePayer
        signature
        success
        transactionIndex
      }
    }
  }
}

<!-- Parameters -->

{
  "signature": "19d4fXU8iyMfSkzMadyxzRb9iAkiULA7QYyKPA8rkJP3KPLnhLQLengqj8GpSHVrN4FbCJuFFCgLJc4ifuJjQCs",
  "network": "solana"
}
```

<details>

<summary>Filtering instructionAccounts</summary>

`account`

This field filters the results by the account that was affected by the instruction. You can filter by the account index, name, owner, or type.

`transactionIndex`

This field filters the results by the transaction index of the instruction.

`time`

This field filters the results by the timestamp of the block in which the instruction was included.

`success`

This field filters the results by the success of the transaction that included the instruction.

`signature`

This field filters the results by the signature of the transaction that included the instruction.

`programId`

This field filters the results by the program ID of the program that was used to execute the instruction.

`previousBlockHash`

This field filters the results by the hash of the previous block.

`parsedType`

This field filters the results by the parsed type of the account that was affected by the instruction.

`parsedProgramName`

This field filters the results by the parsed program name of the program that was used to execute the instruction.

`parsedActionName`

This field filters the results by the parsed action name of the action that was performed by the instruction.

`parsed`

This field filters the results by the parsed instruction object.

`options`

Filter returned data by ordering, limiting, and constraining it. Available fields: `asc`, `ascByInteger`, `desc`, `descByInteger`, `limit`, `limitBy`, `offset`.

`height`

This field filters the results by the block height of the block in which the instruction was included.

`feePayer`

This field filters the results by the address of the account that paid the transaction fee.

`fee`

This field filters the results by the transaction fee.

`external`

This field filters the results by whether the instruction was external.

`date`

This field filters the results by the date of the block in which the instruction was included.

`callPath`

This field filters the results by the call path of the instruction.

`blockHash`

This field filters the results by the hash of the block in which the instruction was included.

`any`

A catch-all filter (OR Logic) that can be used to filter the results by any of the other fields. group: A filter that groups the results by a specific field.

`accountType`

This field filters the results by the type of the account that was affected by the instruction.

`accountOwner`

This field filters the results by the owner of the account that was affected by the instruction.

`accountIndex`

This field filters the results by the index of the account that was affected by the instruction.

</details>

### Fields[​](https://docs.bitquery.io/v1/docs/Schema/solana/instructionAccounts#fields) <a href="#fields" id="fields"></a>

*   **instruction**

    This object contains information about the instruction itself.

    *   **action**

        This field contains the name of the action that was performed by the instruction.
    *   **program**

        This field contains the name of the program that was used to execute the instruction.
*   **account**

    This object contains information about the account that was affected by the instruction.

    *   **index**

        This field contains the unique identifier for the account.
    *   **name**

        This field contains the human-readable name of the account.
    *   **owner**

        This field contains the address of the account owner.
    *   **type**

        This field contains the type of the account, such as `tokenAccount` or `programAccount`.
*   **block**

    This object contains information about the block in which the instruction was included.

    *   **height**

        This field contains the number of blocks that have been processed since the genesis block.
    *   **hash**

        This field contains the unique identifier for the block.
    *   **previousBlockHash**

        This field contains the hash of the previous block.
    *   **timestamp**

        This field contains the Unix timestamp of the block.
*   **transaction**

    This object contains information about the transaction that included the instruction.

    *   **feePayer**

        This field contains the address of the account that paid the transaction fee.
    *   **signature**

        This field contains the cryptographic signature of the transaction.
    *   **success**

        This field indicates whether the transaction was successful.
    *   **transactionIndex**

        This field is a unique identifier for the transaction within the block.

## Instructions

The Solana instructions API allows you to query information about instructions that have been executed on the Solana blockchain. This information includes the program that executed the instruction, the accounts that were affected by the instruction, the data that was passed to the instruction, and the log message that was produced by the instruction. The schema includes the following fields:

```
query ($network: SolanaNetwork!, $signature: String!) {
  solana(network: $network) {
    instructions(signature: {is: $signature}) {
      program {
        id
        name
        parsedName
        parsed
      }
      accountsCount
      data {
        hex
      }
      log {
        consumed
        logs
        result
        totalGas
        instruction
      }
      action {
        name
        type
      }
      externalAction {
        type
        name
      }
      externalProgram {
        id
        name
        parsed
        parsedName
      }
      transaction {
        transactionIndex
        success
        signature
        feePayer
      }
    }
  }
}
<!-- Parameters -->
{
  "signature": "126VTE4UyQJU7FR68aeaFno11WMYUYX2SRn1dVVT83beQnP3sY2hpHyP2uRgni4u3a2jR2kRnGqs2br2K3V8BkE6",
  "network": "solana"
}
```

## Transactions

The Solana transactions API allows you to query for transactions on the Solana blockchain. You can use this API to get information about specific transactions, such as the signature, block, transaction fee, success, fee payer, inner instructions count, instructions count, signer, and transaction index.

```
query ($network: SolanaNetwork!, $date: ISO8601DateTime, $height: Int) {
  solana(network: $network) {
    transactions(options: {limit: 10}, date: {is: $date}, height: {is: $height}) {
      signature
      block {
        timestamp {
          time
        }
        height
        parentSlot
        hash
        previousBlockHash
      }
      transactionFee
      success
      feePayer
      innerInstructionsCount
      instructionsCount
      signer
      transactionIndex
    }
  }
}

{
  "network": "solana",
  "height": 208986228,
  "from": "2023-07-26",
  "till": "2023-08-02T23:59:59",
  "dateFormat": "%Y-%m-%d"
}
```

<details>

<summary>Filtering Transactions</summary>

`transactionIndex`: This field allows you to filter transactions by their index in the block.

`transactionFee`: This field allows you to filter transactions by their fee.

`success`: This field allows you to filter transactions by whether or not they were successful.

`signer`: This field allows you to filter transactions by the account that signed the transaction.

`signature`: This field allows you to filter transactions by their signature.

`recentBlockHash`: This field allows you to filter transactions by the hash of the most recent block they were included in.

`previousBlockHash`: This field allows you to filter transactions by the hash of the block that they were included in.

`parentSlot`: This field allows you to filter transactions by the parent slot of the block that they were included in.

`options`: This field allows you to filter returned data by ordering, limiting, and constraining it.

`instructionsCount`: This field allows you to filter transactions by the number of instructions they contain.

`innerInstructionsCount`: This field allows you to filter transactions by the number of inner instructions they contain.

`height`: This field allows you to filter transactions by their height.

`feePayer`: This field allows you to filter transactions by the account that paid the transaction fee.

`fee`: This field allows you to filter transactions by their fee.

`date`: This field allows you to filter transactions by their date.

blockHash: This field allows you to filter transactions by their block hash.

any: This field allows you to filter transactions by any of the other fields in OR condition.

accountsCount: This field allows you to filter transactions by the number of accounts they interact with.

</details>

### Fields[​](https://docs.bitquery.io/v1/docs/Schema/solana/transactions#fields) <a href="#fields" id="fields"></a>

`signature`: The signature of the transaction.

`block`: The block that the transaction was included in.

`transactionFee`: The transaction fee.

`success`: Whether or not the transaction was successful.

`feePayer`: The account that paid the transaction fee.

`innerInstructionsCount`: The number of inner instructions in the transaction.

`instructionsCount`: The total number of instructions in the transaction.

`signer`: The account that signed the transaction.

`transactionIndex`: The index of the transaction in the block.

## Transfers

The transfers API allows you to get a information on transfers that have occurred on the Solana blockchain.Below are the fields in the schema:

```
query ($network: SolanaNetwork!, $date: ISO8601DateTime, $height: Int) {
  solana(network: $network) {
    transfers(options: {limit: 10}, date: {is: $date}, height: {is: $height}) {
      block {
        height
        timestamp {
          time
        }
        hash
        previousBlockHash
      }
      transaction {
        signature
        accountsCount
        error
        fee
        feePayer
        innerInstructionsCount
        instructionsCount
        recentBlockHash
        success
        signer
      }
      sender {
        address
        mintAccount
        type
      }
      receiver {
        address
        mintAccount
        type
      }
      currency {
        symbol
        address
        tokenType
        tokenId
        name
        decimals
      }
      amount
      transferType
      instruction {
        callPath
        action {
          name
          type
        }
        externalAction {
          name
          type
        }
        external
        externalProgram {
          id
          name
          parsed
          parsedName
        }
        program {
          id
          name
          parsed
          parsedName
        }
      }
    }
  }
}
<!-- Parameters -->
{
  "network": "solana",
  "height": 209945815,
  "from": "2023-07-31",
  "till": "2023-08-07T23:59:59",
  "dateFormat": "%Y-%m-%d"
}
```

<details>

<summary>Filtering Transfers</summary>

`amount`

The amount of currency that was transferred.

`transferType`

The type of transfer. Some of the supported transfer types are create\_account and transfer.

`transactionIndex`

The index of the transaction in the block.

`tokenAccount`

The token account that was transferred.

`time`

The timestamp of the transfer.

`success`

Whether the transfer was successful.

`signer`

The address of the signer of the transaction.

`signature`

The signature of the transaction.

`senderType`

The type of the sender account. The supported sender types are program, user, and unknown.

`senderMintAddress`

The mint address of the sender account.

`senderAddress`

The address of the sender account.

`recentBlockHash`

The hash of the most recent block that the transfer was included in.

`receiverType`

The type of the receiver account.

`receiverMintAddress`

The mint address of the receiver account.

`receiverAddress`

The address of the receiver account.

`programId`

The program ID of the program that was used to perform the transfer.

`previousBlockHash`

The hash of the previous block.

`parsedType`

The parsed type of the transfer.

`parsedActionName`

The parsed name of the action that was used to perform the transfer.

`parsedProgramName`

The parsed name of the program that was used to perform the transfer.

`parsed`

The parsed transfer object.

`options`

A set of options that can be used to filter the results.

`limit` The maximum number of transfers to return.

`date` The date of the transfers.

`height`

The height of the transfers.

`feePayer`

The address of the fee payer.

`externalProgramId`

The external program ID of the program that was used to perform the transfer.

`externalParsedType`

The parsed type of the external transfer.

`externalParsedProgramName`

The parsed name of the external program that was used to perform the transfer.

`externalParsedActionName`

The parsed name of the external action that was used to perform the transfer.

`external`

Whether the transfer was an external transfer.

`date`

The date of the transfers. The date should be in the format YYYY-MM-DD.

`currency`

The currency that was transferred.

`callPath`

The call path of the transfer.

`externalParsed`

The parsed external transfer object.

`blockHash`

The hash of the block that the transfer was included in.

`any`

A catch-all filter (OR Logic) that can be used to filter the results by any of the other fields.

</details>

### Fields[​](https://docs.bitquery.io/v1/docs/Schema/solana/transfers#fields) <a href="#fields" id="fields"></a>

*   **block**

    The block that the transfer was included in.

    * **height**
      * The height of the block.
    * **timestamp**
      * The timestamp of the block.
    * **hash**
      * The hash of the block.
    * **previousBlockHash**
      * The hash of the previous block.
*   **transaction**

    The transaction that the transfer was part of.

    * **signature**
      * The signature of the transaction.
    * **accountsCount**
      * The number of accounts that were involved in the transaction.
    * **error**
      * Whether the transaction failed.
    * **fee**
      * The fee that was paid for the transaction.
    * **feePayer**
      * The address of the account that paid the fee.
    * **innerInstructionsCount**
      * The number of instructions that were included in the transaction.
    * **instructionsCount**
      * The total number of instructions that were included in the transaction, including the inner instructions.
    * **recentBlockHash**
      * The hash of the most recent block that the transaction was included in.
    * **success**
      * Whether the transaction was successful.
    * **signer**
      * The address of the signer of the transaction.
*   **sender**

    The address of the sender of the transfer.

    * **address**
      * The address of the sender account.
    * **mintAccount**
      * Whether the sender account is a mint account.
    * **type**
      * The type of the sender account.
*   **receiver**

    The address of the receiver of the transfer.

    * **address**
      * The address of the receiver account.
    * **mintAccount**
      * Whether the receiver account is a mint account.
    * **type**
      * The type of the receiver account.
*   **currency**

    The currency that was transferred.

    * **symbol**
      * The symbol of the currency.
    * **address**
      * The address of the token account that was transferred.
    * **tokenType**
      * The type of the token. The supported token types are `SPL`, `Native`, and `Unknown`.
    * **tokenId**
      * The token ID of the token that was transferred.
    * **name**
      * The name of the token.
    * **decimals**
      * The number of decimal places for the token.
*   **amount**

    The amount of currency that was transferred.
*   **transferType**

    The type of transfer. Some of the supported transfer types are `mint`, `burn`, and `transfer`.
*   **instruction**

    The instruction that was used to perform the transfer.

    * **callPath**
      * The call path of the instruction.
    * **action**
      * The action that was called.
    * **externalAction**
      * The external action that was called.
    * **external**
      * Whether the transfer was an external transfer.
    * **externalProgram**
      * The external program ID of the program that was used to perform the transfer.
    * **name**
      * The name of the action.
    * **type**
      * The type of the action.
    * **program**
      * The program ID of the program that was used to perform the transfer.

## Coinpath® API

Coinpath® api is targeted to help build compliance solutions by providing money tracking capabilities. This API is supported for [all blockchains we support](https://account.bitquery.io/admin/accounts) and tokens built on them.

### What is depth?[​](https://docs.bitquery.io/v1/docs/Examples/coinpath/money-flow-api#what-is-depth) <a href="#what-is-depth" id="what-is-depth"></a>

Please check following image to understand the depth (hop).

![coinpath](https://docs.bitquery.io/v1/assets/images/depth\_coinpath-c3dd51c63ab512f714c1ccad1ae41794.png)

### What is direction?[​](https://docs.bitquery.io/v1/docs/Examples/coinpath/money-flow-api#what-is-direction) <a href="#what-is-direction" id="what-is-direction"></a>

It;s direction of fund flow, inbound (Incoming) or outbound (Outgoing).

### minimumTxAmount[​](https://docs.bitquery.io/v1/docs/Examples/coinpath/money-flow-api#minimumtxamount) <a href="#minimumtxamount" id="minimumtxamount"></a>

It's a `parameter` available inside `Options`, which allow you to filter transaction based on amount.

### maximumAddressTxCount[​](https://docs.bitquery.io/v1/docs/Examples/coinpath/money-flow-api#maximumaddresstxcount) <a href="#maximumaddresstxcount" id="maximumaddresstxcount"></a>

If defined > 0, then it will not try to expand an addresses for the next depth, having more that this count of transactions. Use to stop on exchange-type addresses and not expand them

### maximumTotalTxCount[​](https://docs.bitquery.io/v1/docs/Examples/coinpath/money-flow-api#maximumtotaltxcount) <a href="#maximumtotaltxcount" id="maximumtotaltxcount"></a>

Do not extend the next depth in case total tx count on prev hop exceed this metric. Used to prevent hanging on the calculatomg for a long time

### complexityLimit[​](https://docs.bitquery.io/v1/docs/Examples/coinpath/money-flow-api#complexitylimit) <a href="#complexitylimit" id="complexitylimit"></a>

If the initial count of transactions for the address under coinpath is exceeding this value, do not proceed and return an error. Works for the same reason as the previous parameter.

### seed[​](https://docs.bitquery.io/v1/docs/Examples/coinpath/money-flow-api#seed) <a href="#seed" id="seed"></a>

A random number which can be used to prevent caching of results. Needed omly if blockchain data expected to be modified during coipath calculations

### Destination of funds from an address[​](https://docs.bitquery.io/v1/docs/Examples/coinpath/money-flow-api#destination-of-funds-from-an-address) <a href="#destination-of-funds-from-an-address" id="destination-of-funds-from-an-address"></a>

In 2019, Upbit exchange was hacked and tweeted out the [hackers address](https://explorer.bitquery.io/ethereum/address/0xa09871aeadf4994ca12f5c0b6056bbd1d343c029/graph?from=2018-03-01\&till=2021-01-31), using the following API you can track destination of fund over multiple hops (depth).

[Open this query on IDE](https://ide.bitquery.io/destination-of-funds-for-upbit-hackers)

```
{
  ethereum(network: ethereum) {
    outbound: coinpath(
      initialAddress: { is: "0xa09871aeadf4994ca12f5c0b6056bbd1d343c029" }
      currency: { is: "ETH" }
      depth: { lteq: 2 }
      options: {
        asc: "depth"
        desc: "amount"
        limitBy: { each: "depth", limit: 10 }
      }
      date: { since: "2018-03-01", till: "2021-01-31" }
    ) {
      sender {
        address
        annotation
        smartContract {
          contractType
          currency {
            symbol
            name
          }
        }
      }
      receiver {
        address
        annotation
        smartContract {
          contractType
          currency {
            symbol
            name
          }
        }
      }
      amount
      currency {
        symbol
        name
      }
      transaction {
        hash
        value
      }
      block {
        height
        timestamp {
          time(format: "%y-%d-%m")
        }
      }
      depth
      count
    }
  }
}
```

### Source of funds from an address[​](https://docs.bitquery.io/v1/docs/Examples/coinpath/money-flow-api#source-of-funds-from-an-address) <a href="#source-of-funds-from-an-address" id="source-of-funds-from-an-address"></a>

To check the source of funds, you can use the following API. You can increase depth based on your requirements.

[Open this query on IDE](https://ide.bitquery.io/All-inbound-transactions-to-Upbit-hacker-address)

```
{
  ethereum(network: ethereum) {
    inbound: coinpath(
      initialAddress: { is: "0xa09871aeadf4994ca12f5c0b6056bbd1d343c029" }
      currency: { is: "ETH" }
      depth: { lteq: 2 }
      options: {
        direction: inbound
        asc: "depth"
        desc: "amount"
        limitBy: { each: "depth", limit: 10 }
      }
      date: { since: "2018-03-01", till: "2021-01-31" }
    ) {
      sender {
        address
        annotation
        smartContract {
          contractType
          currency {
            symbol
            name
          }
        }
      }
      receiver {
        address
        annotation
        smartContract {
          contractType
          currency {
            symbol
            name
          }
        }
      }
      amount
      currency {
        symbol
        name
      }
      transaction {
        hash
        value
      }
      block {
        height
        timestamp {
          time(format: "%y-%m-%d")
        }
      }
      depth
      count
    }
  }
}
```

### Relation between two addresses[​](https://docs.bitquery.io/v1/docs/Examples/coinpath/money-flow-api#relation-between-two-addresses) <a href="#relation-between-two-addresses" id="relation-between-two-addresses"></a>

Using combination of above two queries you can check if two address ever transacted in the past.

[Open this query on IDE](https://ide.bitquery.io/Relation-between-two-ethereum-addresses)

```
{
  ethereum(network: ethereum) {
    inbound: coinpath(
      initialAddress: { is: "0xa09871aeadf4994ca12f5c0b6056bbd1d343c029" }
      sender: { is: "0xb3a9b79f4d5dc2cdcdc00da22869502cbf65a0a5" }
      currency: { is: "ETH" }
      depth: { lteq: 1 }
      options: {
        direction: inbound
        asc: "depth"
        desc: "amount"
        limitBy: { each: "depth", limit: 10 }
      }
      date: { since: "2019-11-20", till: "2019-11-30" }
    ) {
      sender {
        address
        annotation
        smartContract {
          contractType
          currency {
            symbol
            name
          }
        }
      }
      receiver {
        address
        annotation
        smartContract {
          contractType
          currency {
            symbol
            name
          }
        }
      }
      amount
      currency {
        symbol
      }
      transaction {
        value
        hash
      }
      block {
        height
        timestamp {
          time(format: "%y-%m-%d")
        }
      }
      depth
      count
    }
    outbound: coinpath(
      initialAddress: { is: "0xa09871aeadf4994ca12f5c0b6056bbd1d343c029" }
      receiver: { is: "0xb3a9b79f4d5dc2cdcdc00da22869502cbf65a0a5" }
      currency: { is: "ETH" }
      depth: { lteq: 2 }
      options: {
        asc: "depth"
        desc: "amount"
        limitBy: { each: "depth", limit: 10 }
      }
      date: { since: "2019-11-20", till: "2019-11-30" }
    ) {
      sender {
        address
        annotation
        smartContract {
          contractType
          currency {
            symbol
            name
          }
        }
      }
      receiver {
        address
        annotation
        smartContract {
          contractType
          currency {
            symbol
            name
          }
        }
      }
      amount
      currency {
        symbol
      }
      depth
      count
    }
  }
}
```

### Tracing SOL movement between two addresses on Solana[​](https://docs.bitquery.io/v1/docs/Examples/coinpath/money-flow-api#tracing-sol-movement-between-two-addresses-on-solana) <a href="#tracing-sol-movement-between-two-addresses-on-solana" id="tracing-sol-movement-between-two-addresses-on-solana"></a>

The query below will trace the movement of SOL between two addresses on Solana The query will return two objects: `inbound` and `outbound`. The `inbound` object will contain information about all inbound transactions to the specified address, and the `outbound` object will contain information about all outbound transactions from the specified address.

[Open this query on IDE](https://ide.bitquery.io/solana-coinpath-example)

```
query ($network: SolanaNetwork!, $address: String!, $inboundDepth: Int!,
        $outboundDepth: Int!, $limit: Int!, $from: ISO8601DateTime, $till: ISO8601DateTime,
        $currency: String!
       ) {
        solana(network: $network) {
          inbound: coinpath(
            initialAddress: {is: $address}
            depth: {lteq: $inboundDepth}
            options: {direction: inbound, asc: "depth", desc: "amount", limitBy: {each: "depth", limit: $limit}}
            date: {since: $from, till: $till}
            currency: { is: $currency }
          ) {
            sender {
              address
              annotation
            }
            receiver {
              address
              annotation
            }
            amount
            currency {
              symbol
              name
            }
            depth
            count
          }
          outbound: coinpath(
            initialAddress: {is: $address}
            depth: {lteq: $outboundDepth}
            options: {asc: "depth", desc: "amount", limitBy: {each: "depth", limit: $limit}}
            date: {since: $from, till: $till}
            currency: { is: $currency }
          ) {
            sender {
              address
              annotation
            }
            receiver {
              address
              annotation
            }
            amount
            currency {
              symbol
              name
            }
            depth
            count
          }
        }
      }

<!-- Parameters -->

      {
  "inboundDepth": 1,
  "outboundDepth": 1,
  "limit": 10,
  "offset": 0,
  "network": "solana",
  "address": "4yWr7H2p8rt11QnXb2yxQF3zxSdcToReu5qSndWFEJw",
  "currency": "Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB",
  "from": "2022-09-19",
  "till": "2022-09-26T23:59:59",
  "dateFormat": "%Y-%m-%d"
}
```

### Getting inflows ( fund moving in) to a Cardano Wallet /Address[​](https://docs.bitquery.io/v1/docs/Examples/coinpath/money-flow-api#getting-inflows--fund-moving-in-to-a-cardano-wallet-address) <a href="#getting-inflows--fund-moving-in-to-a-cardano-wallet-address" id="getting-inflows--fund-moving-in-to-a-cardano-wallet-address"></a>

We get the details of tokens moving in to a wallet using the using the `outputs` function and setting the `outputAddress:` as the address of the wallet to which funds are moving in. Below is the sample query that gets token movements into a cardano wallet between two dates. Here's the [query on IDE](https://ide.bitquery.io/All-inflows-into-Cardano-wallet)

```
query ($network: CardanoNetwork!, $address: String!, $from: ISO8601DateTime, $till: ISO8601DateTime) {
  cardano(network: $network) {
    outputs(
      date: {since: $from, till: $till}
      outputAddress: {is: $address}
      options: {desc: ["block.height", "outputIndex"], limit:10}
    ) {
      block {
        height
        timestamp {
          time(format: "%Y-%m-%d %H:%M:%S")
        }
      }
      transaction {
        hash
      }
      outputIndex
      outputDirection
      value
      value_usd: value(in: USD)
      currency {
        symbol
      }
    }
  }
}

<!-- Parameters -->

{
  "address": "addr1qxz3ve4caaywwg6q82ax9l5xknyc7juvwwsw20cpugyz5gv9zent3m6guu35qw46vtlgddxf3a9ccuaqu5lsrcsg9gss69fhxw",
  "network": "cardano",
  "from": "2022-10-19",
  "till": "2022-10-26T23:59:59",
  "dateFormat": "%Y-%m-%d"
}

```

### Get Transactions from Multiple Addresses to a Final Destination Address[​](https://docs.bitquery.io/v1/docs/Examples/coinpath/money-flow-api#get-transactions-from-multiple-addresses-to-a-final-destination-address) <a href="#get-transactions-from-multiple-addresses-to-a-final-destination-address" id="get-transactions-from-multiple-addresses-to-a-final-destination-address"></a>

You can get the trasactions to a final destination addresses from an initial addresses no matter how many hops happen. Below is an example on how to use it. You can access the query [here](https://ide.bitquery.io/Ethereum-inbound-coinpath-from-one-address-to-another)

```
query ($network: EthereumNetwork!, $address: String!, $inboundDepth: Int!, $limit: Int!, $currency: String!, $from: ISO8601DateTime, $till: ISO8601DateTime) {
  ethereum(network: $network) {
    inbound: coinpath(
      initialAddress: {is: $address}
      currency: {is: $currency}
      depth: {lteq: $inboundDepth}
      options: {direction: inbound, asc: "depth", desc: "amount", limitBy: {each: "depth", limit: $limit}}
      date: {since: $from, till: $till}
      finalAddress: {is: "0xa910f92acdaf488fa6ef02174fb86208ad7722ba"}
    ) {
      sender {
        address
        annotation
        smartContract {
          contractType
          currency {
            symbol
            name
          }
        }
      }
      receiver {
        address
        annotation
        smartContract {
          contractType
          currency {
            symbol
            name
          }
        }
      }
      amount
      currency {
        symbol
      }
      depth
      count
    }
  }
}
{
  "inboundDepth": 3,
  "limit": 10,
  "offset": 0,
  "network": "ethereum",
  "address": "0xbfba2df39ae248e3dfdefa7a92ac3df9be260bf7",
  "currency": "ETH",
  "from": "2018-10-01",
  "till": "2023-10-11T23:59:59",
  "dateFormat": "%Y-%m"
}
```

This field can be used to filter the results of the query to only include coinpaths that end at a specific address. For example, the query in the example above will only return coinpaths that end at the address `0xa910f92acdaf488fa6ef02174fb86208ad7722ba`.

The `finalAddress` field can also be used to calculate the total amount of funds that were sent to a specific address. To do this, you can use the count field to count the number of coinpaths that end at the address, and the amount field to sum the amount of funds that were sent in each coinpath.
