# Section One - Building a Subgraph

The Graph is a powerful tool that allows developers to focus on building their dApps, instead of focusing on how to retrieve data from the blockchain. Prior the advent of The Graph, dApp developers needed to setup and maintain their own centralized servers, and building softwares on top to go through every single transaction that happened on-chain.

In this tutorial, you will learn how to set up a Subgraph from scratch for your favorite token, and how to write your Subgraph's Manifest.

### Table of Contents
- Subgraph's components
- Setting up an account
- Initializing a Subgraph
- Explaining the Manifest

### Subgraph's components
Every Subgraph has three important files:

#### 1. Manifest - `subgraph.yaml`
In this file, yyou specify which blockchain you intend to index, what contract(s) to listen to, which events and functions should trigger the subgraph to update or store information.

#### 2. Schema - `schema.graphql`
Here you define which data you want to be able to query after indexing your Subgraph using GraphQL. This is actually similar to a model for an API, where the model defines the structure of a request body.

#### 3. AssemblyScript Mappings - `mapping.ts`
This is the logic that determines how data should be retreived and stored when someone interacts with the contracts you listen to. The data gets translated and is stored based off the schema you have listed. The [Graph CLI](https://github.com/graphprotocol/graph-cli) also generates AssemblyScript types using a combination of the subgraph's schema along with a smart contract's ABIs.
AssemblyScript is a TypeScript-based programming language that is optimized for, and statically compiled to WebAssembly. 

In this tutorial, we are going to build a Subgraph to retrieve some info about your favorite ERC20 token. I will be building it for $GRT. 

> Note: if your token is not on ETH mainnet, you just need to remember to change the network when you initialize your Subgraph.

Jump to the Discord #section-one(link to channel) channel and let us know the token you are going to build the Subgraph for!

### Setting up an account
You need to create an account to be able to access the hosted service. Go ahead and [sign up](https://thegraph.com/hosted-service/) using your GitHub account.

Now, navigate to your dashboard and click on `Add Subgraph`. Fill in the required information, then click on `Create a Subgraph`. You are ready to go!

### Initializing a Subgraph
Okay, now let's roll up our sleeves and start writing some code. 

The first thing we need to do is initialize a Subgraph using this command in your terminal:

```bash
graph init subgraph-name
```

> In my case, `subgraph-name` is `grt-token`.

The CLI will ask you to choose a protocol, Ethereum or Near. Click on Ethereum. Then choose the hosted-service.

After that, it will ask for the name of your Subgraph. It should be `your-GitHub-account/your-subgraph-name`. 

Make sure your Subgraph's name matches the one you have set up in the last step. Also, to replace the spaces with `-` and the capital letters with small letters. For example, if your Subgraph on your dashboard is "Grt Token", you should enter "grt-token" in your terminal.

> Note: if you don't follow the rule above, you will get this error; Failed to install dependencies: Command failed: yarn error package.json: Name contains illegal characters.

After clicking enter, it will prompt the available networks, choose mainnet.

Now we need to provide our Subgraph with the smart contract we want to index. You can inspect your token on [EtherScan](https://etherscan.io/). The GRT token contract is: 
`0xc944e90c64b2c07662a292be6244bdf05cda44a7`

You can leave the default contract name; Contract.

The CLI should create your Subgraph now. If the process fails for any reason, please use the #section-one(link to channel) channel on Discord and share a screenshot with us.

> Note: If you are already using `yarn` by default, then you may encounter an error `× Failed to install dependencies: Command failed: yarn`. Simply open a new CMD in your Subgraph folder and run `npm install`. 

Congrats, you Initialized your first Subgraph! 

### Explaining the Manifest 
Open the `subgraph.yaml` file and replace your code with: 

```bash
specVersion: 0.0.1
schema:
  file: ./schema.graphql
dataSources:
  - kind: ethereum/contract
    name: Contract
    network: mainnet
    source:
      address: "0xc944E90C64B2c07662A292be6244BDf05Cda44a7"
      abi: Contract
      startBlock: 11446769
    mapping: 
      kind: ethereum/events
      apiVersion: 0.0.4
      language: wasm/assemblyscript
      entities:
        - Transfer
      abis:
        - name: Contract
          file: ./abis/Contract.json
      eventHandlers:
        - event: Transfer(indexed address,indexed address,uint256)
          handler: handleTransfer
      file: ./src/mapping.ts
```

Don't worry, we are going to explain the `.yaml` file line by line.

```bash
schema:
  file: ./schema.graphql
```

Here you are just defining the path of your GraphQL schema. It is automatically generated when you initialized your Subgraph.

Under `dataSources` you define the sources of your data. It is possible to index multiple contracts at the same time, but they have to be on the same chain.

```bash
  - kind: ethereum/contract
    name: Contract
    network: mainnet
    source:
      address: "0xc944E90C64B2c07662A292be6244BDf05Cda44a7"
```

If your token is on a different chain (not ETH mainnet), then your network should refer to it. Also, make sure you fill in the address with the contract of your token.

```bash
      abi: Contract
      startBlock: 11446769
```

Above, you are just defining the name of your ABI (this is like the table of contents for a smart contract). We will talk about this later. And the `startBlock` is telling the indexer from which block to start indexing data. It is recommended to fill in the startBlock with the block that the contract was created. Otherwise you will start syncing from the genesis block and end up waiting long periods of time before you encounter any relevant data.

It is easy to find that block, just head to [EtherScan](https://etherscan.io/), and click on the creator txn hash. 

-- Insert Section-1-1.PNG here --

-- Insert Section-1-2.PNG here --


`mapping.entities` - The entities are defined in the GraphQL schema file, and they are actually defining the structure of the stored data.

The ABI refers to Application Binary Interface, and it is basically how you call functions in a contract and get data back. When you initialize a Subgraph, it is automatically grabbing the ABIs (you can check under the ABIs folder), but in some cases, you will need to add them manually.

If you are the owner of the contact, you will have the latest ABIs. If you are not the owner, go to [EtherScan](https://etherscan.io/), click on `Contract`, then scroll down, copy the `Contract ABI`, and add it manually to your Subgraph.

-- Insert Section-1-3.PNG here --

-- Insert Section-1-4.PNG here --

> Note: Make sure to provide the new ABIs to your Subgraph and update it whenever you update your smart contracts.

> Note: Sometimes, you may not be able to find the ABIs on EtherScan, then you have to contact the smart contract deployer.

```bash
      eventHandlers:
        - event: Transfer(indexed address,indexed address,uint256)
          handler: handleTransfer
```

Now the cool part, in our Subgraph we are only going to index the `transfer` event.

Remember the `Contract` page? Just scroll down and you will find the source code of your contract. Click CTRL+F and search for `event`. You will find all the events that the contract is emitting and their parameters. All ERC20 contracts have the same events.

The event we are currently setting up our Subgraph to be listening to is: 

```bash
event Transfer(address indexed from, address indexed to, uint256 value);
```

As you can see we now have access to the parameters of what address the transfer is sent from, who it's sent to, and how much is sent.

```bash
handler: handleTransfer
```

Here we are just telling our Subgraph, hey, whenever the `transfer` event is emitted, run our handler `handleTransfer`. We are going to write our handlers in our mapping file, so don't worry about it right now.

We are only going to use `eventHandlers` in our tutorial, but The Graph also supports `callHandlers` and `BlockHandlers`.

If you are using `blockHandlers` without a filter, the handler will be triggered whenever a new block is created on the chain.

Also, `callHandlers` and `blockHandlers` are not supported on every chain. All Geth chains do not support tracing, therefore, you can only use `eventHandlers` in this case.

Wow! You are now a Manifest hero and know how to set up a Subgraph properly.
Let's test the knowledge you have gained in this tutorial.

1. If you are building a Subgraph to index the CHR token on ETH mainnet, the StartBlock should be:
- 12988298
- 11988298
- 10988298 (correct answer)
- 10788298

-- If the reader chooses a wrong answer - Hint: make sure to check on EtherScan. --

2. The part of your subgraph that contains states which contract you want to listen to:
- The Subgraph Manifest - subgraph.yaml (correct answer)
- The Mappings - mappings.ts
- The ABI 
- The Body - body.graphql

3. When you update your smart contract, you need to add the new ABIs to your Subgraph and redeploy it:
- Yes (correct answer)
- No 

-- If the reader chooses a wrong answer - Hint: If you don't update your ABIs, your Subgraph will fail while indexing new data. --

4. You can deploy a Subgraph to index BSC on Mainnet:
- Yes
- No (correct answer)

-- If the reader chooses a wrong answer - Hint: Mainnet currently only Supports ETH mainnet. --








