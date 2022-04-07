# Section Three - Querying a Subgraph
In this tutorial, you will learn how to query your Subgraph, and at the same time how to leverage other Subgraphs instead of building your own from scratch.

### Table of Contents
- How to query a Subgraph
- Leveraging deployed Subgraphs

### How to query a Subgraph
As we have mentioned in the introduction of this course, you need to have some experience with GraphQL. Don't worry if you are a beginner, the queries we are going to write should be easy to grasp!

First of all, head to the playground of your Subgraph, it should be similar to:

-- Insert Section-3-1.PNG here --

If you click on `TransferEvent`, you should see the fields of your Schema with the comments we have written.

Now, let's execute the following query:

```
{
  transferEvents(first: 5) {
    amount
    sender
    destination
    block
    timestamp
    transaction
  }
}
```

This should return the first 5 transactions with their information. Copy one of the transaction hashes you see and inspect it on EtherScan to make sure that you received the correct info!

Now, let's query a specific transaction, we are going to ask for the addresses of the sender and receiver and the amount sent.

```
{
  transferEvent(id:"0x0000000a2ea080459faf59bed4cdb63f3f9692ad0569e4cbf4b45a28b208f3b9") {
    amount
    sender
    destination
  }
}
```

> Note: if your Subgraph is indexing a different token than GRT, make sure to query a correct id.

If your Subgraph is indexing GRT, the output should be:
```
{
  "data": {
    "transferEvent": {
      "amount": "210700000000000000000000",
      "sender": "0x21a31ee1afc51d94c2efccaa2092ad1028285549",
      "destination": "0x891f3e471698f3355b06d3b56f93bc5ed289381e"
    }
  }
}
```

This is the beauty of GraphQL! It gives you the power to ask for exactly what you need and nothing more.

### Leveraging deployed Subgraphs

I've seen a lot of developers building Subgraphs from scratch to index DEXes or NFTs. And most of them are already deployed and you can integrate them immediately. Some examples are:
- [SushiSwap Exchange](https://thegraph.com/explorer/subgraph?id=D7azkFFPFT5H8i32ApXLr34UQyBfxDAfKoCEK4M832M6&view=Overview)
- [ENS domains](https://thegraph.com/hosted-service/subgraph/ensdomains/ens)
- [EIP721](https://thegraph.com/explorer/subgraph?id=AVZ1dGwmRGKsbDAbwvxNmXzeEkD48voB3LfGqj5w7FUS&view=Playground) - Indexes all ERC721 NFTs.

So make sure to search on the [Graph Explorer](https://thegraph.com/explorer) and the [Hosted Service](https://thegraph.com/hosted-service) before building your Subgraph.

Now, let's learn how to pull out some data about your favorite NFTs collection!

First of all, head to [EIP721 Subgraph](https://thegraph.com/explorer/subgraph?id=AVZ1dGwmRGKsbDAbwvxNmXzeEkD48voB3LfGqj5w7FUS&view=Playground), as we have mentioned earlier, this Subgraph is indexing all the ERC721 NFTs.

My favorite collection is [Doodles](https://doodles.app/), but feel free to grab your favorite collection's contract on [EtherScan](https://etherscan.io/)!

The smart contract of Doodles is: `0x8a90cab2b38dba80c64b7734e58ee1db38b8992e`

Enter this query in the Playground:
```
{
  erc721Contract(id: "0x8a90cab2b38dba80c64b7734e58ee1db38b8992e") {
    id
    name
    symbol
  }
}
```

The output should be:

```
{
  "data": {
    "erc721Contract": {
      "id": "0x8a90cab2b38dba80c64b7734e58ee1db38b8992e",
      "name": "Doodles",
      "symbol": "DOODLE"
    }
  }
}
```

We got the name of the collection associated with its token's symbol. How cool is that!

Now, let's track a specific NFTs transaction. I choose [this one](https://etherscan.io/tx/0x4cb52b25d8a1a1c04d0eded8fb7787b81953f5be6d0b26a71df104cfbad612f0) randomly, where the owner transferred the Doodle with an id [3126](https://etherscan.io/token/0x8a90cab2b38dba80c64b7734e58ee1db38b8992e?a=3126) to a different wallet.

Enter this query in the Playground:
```
{
  transaction(id:"0x4cb52b25d8a1a1c04d0eded8fb7787b81953f5be6d0b26a71df104cfbad612f0"){
    timestamp
    blockNumber
  }
}
```

The output should be:
```
{
  "data": {
    "transaction": {
      "blockNumber": "14466900",
      "timestamp": "1648366416"
    }
  }
}
```

We got the block number and the timestamp of that tx, you can verify that on [EtherScan](https://etherscan.io/tx/0x4cb52b25d8a1a1c04d0eded8fb7787b81953f5be6d0b26a71df104cfbad612f0).

Let's test the knowledge you have gained in this tutorial!
1. Adding `last: 5` to your query should:
- Grab the first 5 events saved
- Grab the first 5 events ordered by timestamp
- Grab the last 5 events saved (correct answer)
- Grab the last 5 ordered by the block number

2. The timestamp of the block is the time at which a miner mined the block:
- Yes (correct answer)
- No

3. The timestamp of the block is represented as:
- Minutes
- Seconds (correct answer)
- Milliseconds
- Hours

4. If we want to query the [SushiSwap Subgraph](https://thegraph.com/explorer/subgraph?id=D7azkFFPFT5H8i32ApXLr34UQyBfxDAfKoCEK4M832M6&view=Playground) to get daily volume about a token, the correct query structure is:
-```{
 token(id:"0x0000a1c00009a619684135b824ba02f7fbf3a572"){
  dayData {
    id
    volumeETH
  }
}
}``` (correct answer)
-```{
 token(id:"0x0000a1c00009a619684135b824ba02f7fbf3a572"){
  id {
    dayData
    volumeETH
  }
}
}``` 
-```{
 token(id:"0x0000a1c00009a619684135b824ba02f7fbf3a572"){
  dayData {
    id(volumeETH)
  }
}
}```
-```{
 token(){
  dayData(id: "id:"0x0000a1c00009a619684135b824ba02f7fbf3a572"") {
    id
    volumeETH
  }
}
}```


### What's next?

That's all! You now should have a full understanding of how to build custom Subgraphs, and how to query them.

If you have any questions, please feel free to leverage our [Discord](https://discord.gg/vzru5qWA). We will answer any questions you may have.

Some useful links to go through after finishing this course:
- [The Graph official docs](https://thegraph.com/docs/en/developer/quick-start/).
- [Figment Learn](https://learn.figment.io/protocols/thegraph) - Figment is a core dev team at The Graph, and they have a lot of great tutorials.
- [Building GraphQL APIs on Ethereum](https://dev.to/edge-and-node/building-graphql-apis-on-ethereum-4poa) by [Nader Dabit](https://twitter.com/dabit3).












