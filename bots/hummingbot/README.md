# Hummingbot

Hummingbot is a local software client that helps you run trading strategies that automate the execution of orders and trades on various cryptocurrency exchanges and protocols. The Hummingbot architecture features modular components that can be maintained and extended by individual community members.

### Installation

We recommend installing Hummingbot using Docker if you want the simplest, easiest installation method and don't need to modify the Hummingbot codebase. Check out Install via Docker for the basic process.

### Strategies

A Hummingbot strategy loads market data directly from centralized and decentralized exchanges, adaptable to the unique features of each trading venue's WebSocket/REST APIs and nodes, and executes logic defined by the strategy creator.

To run a strategy, a user selects a script or strategy template, defines its input parameters in a Config File, and starts it with the start command in the Hummingbot client or via the command line.

Sample Scripts: Examples of basic and advanced strategy templates in the official codebase

Walkthrough: Learn how to run a simple directional strategy using Hummingbot

The new Hummingbot StrategyV2 framework allows you to mix and match components, offering a modular approach to strategy creation and making the development process faster and more efficient. See StrategyV2 Architecture to learn how V2 components like Executors and Controllers work together.
