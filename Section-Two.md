# Section Two - Building a Subgraph - part 2

In the last section, you have learned how to set up a Subgraph and how to write your `.yaml` file. In this tutorial, you will learn how to write your mappings to translate the data your Subgraph is listening to to the structured entities that you will define in your schema.

### Table of Contents
- Defining the Schema
- Exploring the mappings
- Deploying the Subgraph
- FAQs and Common Errors

### Defining the Schema
Let's begin by defining our Schema. Since our Subgraph is only indexing the transfer event, our schema will be simple and have only 1 entity.

Open your `schema.graphql` file and replace the code with:

```
type TransferEvent @entity {
  id: ID!

  " Quantity of tokens transferred "
  amount: BigDecimal!

  " Transaction sender address "
  sender: Bytes!

  " Address of destination account "
  destination: Bytes!

  " Block number "
  block: BigInt!

  " Event timestamp "
  timestamp: BigInt!

  " Transaction hash "
  transaction: Bytes!
}
```

We have seven fields:

- `id` - It has the type `ID`. This must be unique. Someimes, it is easier to set it to the address of the sender. But in our case, an address can send multiple times, so we have to come up with a way to set a unique id across all the transfer events we are going to store.
 
- `amount` - It has the type `BigDecimal`,  high precision decimals represented as a significand and an exponent with a range of [−6143,+6144]. Rounded to 34 significant digits. This is going to be the amount of tokens sent when the `transfer` event is emitted.

- `sender` - It has the type `Bytes`. It is recommended to use `Bytes` with addresses and hashes. It is represented as a hexadecimal string.

- `destination` - The address of the receiver.

- `block` - The block number in which the transaction was placed.

- `timestamp` - It identifies when the `transfer` event occurred.

- `transaction` - The hash of the transaction.

The ! sign means that field can't be null.

> Note: It is important to add comments to your schema as we did above, because it is going to be added to the playground of your Subgraph when you deploy it.

### Exploring the mappings
As we have explained in the introduction, you need to have some experience with Type Script. But don't worry about it, the function we are going to write should be easy to understand.

Open you `mapping.ts` file, delete the auto generated code, and let's start by adding:

```bash
import { Transfer } from "../generated/Contract/Contract"
import { TransferEvent } from "../generated/schema"
```

This will throw some errors but it is only because we didn't generate our code yet. Don't worry about it, we will fix it soon.

Above, we are just telling our mapping where to find our event and the structure of our data.

Now, let's define our function, and pass our event as a parameter.

```bash
export function handleHolder(event: Transfer): void {}
```

As we have explained earlier, whenever our event is emitted, this handler is going to be called.

Next, add this inside our function:

```bash
    let transferEvent = new TransferEvent(event.transaction.hash.toHex())
    
    let amount = (event.params.value.toBigDecimal())
    transferEvent.amount = amount
    
    transferEvent.sender = event.params.from
    transferEvent.destination = event.params.to

    transferEvent.block = event.block.number
    transferEvent.timestamp = event.block.timestamp
    transferEvent.transaction = event.transaction.hash

    transferEvent.save()
```

In the first line, you can imagine it as we are initializing a new place to hold the data of the emitted event. 
Remember when we said that our id should be unique? Here we are initializing a new id and it is the hash of the transaction.

The `amount` field is going to be the `value` parameter of our event, but we are changing its type to `BigDecimal` before storing it. This is because we have already set it up to be `BigDecimal` in our schema. So you have always to be sure that the types here are the same as your schema, or you will get errors.

The `sender` and `destination` are the addresses of the sender and receiver, respectively. 

The `block` holds the number of the block that the event was emitted in.

The `transaction` holds the hash of the tx.

The `timestamp` is a unique serial that determines the exact moment in which the block has been mined and validated by the blockchain. Each block on a blockchain has its own timestamp.

And finally, we are saving the data into our database.

Simple, right? Now, our Subgraph is ready to be deployed.

### Deploying the Subgraph
After building your first Subgraph, you need to deploy it to the network. 

The steps are easy to follow. let's start by authenticating our account. Open a new terminal and run:

```bash
graph auth
```

Choose the `Hosted Service`. Now, it will ask you to enter the `Deploy Key`. Head to your [dashboard](https://thegraph.com/hosted-service/dashboard), copy your `Access Token`, and paste it.

Then, we need to run this command to generate our code:

```bash
npm run codegen
```

Now your Subgraph is ready to be deployed! Run the following command:

```bash
npm run deploy
```

Wait for it to compile your Subgraph and upload it to [IPFS](https://ipfs.io/). 

If you head to your [dashboard](https://thegraph.com/hosted-service/dashboard), you will see that your Subgraph is deployed and syncing.

If your Subgraph fails for any reason, please head to the #section-two channel on Discord and post a screenshot and a link to your Subgraph.

### FAQs and Common Errors

There are various errors you may encounter while building/deploying your Subgraph. We will try to cover as much as possible but please feel free to leverage our Discord if you get any other errors or have any questions.

If you get the following error when you run `npm run deploy`:

```bash
The current version of graph-cli can't be used with mappings on apiVersion less than '0.0.x'
```

That indicates that the graph-cli version and apiVersion are not compatible, you just need to make sure that you have the latest version of the graph-cli by updating it.

```bash
Failed to deploy to Graph node. Invalid account name or access token. 
```

If you encounter the above error, double-check that you entered the name of your account and your access token correctly.

One of the most common errors is that developers use `blockHandlers` and `callHandlers` with chains that do not support traces. Always make sure that the chain you are indexing support traces or you will have to use `eventHandlers` only.

You have built and deployed your first Subgraph!
Let's test the knowledge you have gained in this tutorial.

1. If you need to store an address, the data type you will choose is:
- Bytes (correct answer)
- ID
- String
- BigInt

-- If the reader chooses a wrong answer - Hint: The data type Bytes is represented as a hexadecimal string. --

2. The ! sign means that the field is not required to be filled:
- Yes
- No (correct answer)

3. If your auth token is compromised, you can change it:
- Yes (correct answer)
- No 

-- If the reader chooses a wrong answer - Hint: You can head to your dashboard and click on re-generate. --

4. The id of your Subgraph doesn't need to be unique across all the events emitted:
- Yes
- No (correct answer)

-- If the reader chooses a wrong answer - Hint: Your id can't be null and should hold a unique value across all events stored. --












