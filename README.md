# Building a Twitter-like app on Flow 

## Introduction

In this guide, we’ll be building a Web3 version of Twitter. By the end of it, you should have an understanding of what building “on Web3” means, how to write smart contracts that go beyond the typical JPEG NFT example, and hopefully walk away with inspiration and ideas on what _you_ can build next. 

You know that inspirational quote: "It's not the destination, it's the journey"?  While the finished product is a *[reductio ad absurdum]()* of a real Twitter app, the process along the way is what I want you to focus on. That is, building on-chain logic on a blockchain using smart contracts is relatively simple, and is a general purpose technology. My hope is that it helps cut through the hype and speculation, and shows you how the components work, so you can re-apply them to new use-cases.

### Is this tutorial for me?

I had two choices when writing this: either start from a finished demo and explain how it works, or bring you along as I create the app from scratch and iterate over it and add complexity bit by bit. I've opted for the latter. 

In other words, this is not a quick tutorial ([here's one though](https://developers.flow.com/tools/fcl-js/tutorials/flow-app-quickstart)), it's a somewhat comprehensive step-by-step guide meant to be followed in an afternoon or weekend.

Does that sound good to you? Great, let's dive in!

### Wait...What _is_ web3?

Let's start with the elephant in the room. *Web3*. What is it? You’ll likely get a different answer from everyone you ask. What follows is _a_ definition, which is by no means exhaustive or prescriptive. 

> Web3 is a way to build experiences in a way that they benefit from composability and decentralization. 

Composability means anyone can build on top of any existing experience. In “web2”, the classic way to offer composability is via APIs, and while APIs are great, they need to be built, maintained and are a conscious choice. In Web3, the smart contracts are open to all, and they themselves _are the APIs_. Even more importantly: every app built using smart contracts is effectively a “module” that you can import into your own app. 

> In traditional tech stacks, the libraries and packages are composable (think npm modules) - in web3, the _applications_ are composable (think Twitter). That's the paradigm shift.

Practically, it means that when we deploy our Twitter clone to mainnet (the "live" production chain), anyone else can build experiences on top of it – not just UIs, but higher-order experiences such as games and extended functionality (e.g. Rate-my-tweet or Tweet tipping, etc)

Likewise, Web3’s decentralized nature means we can rest easy knowing no single entity (or billionaire) has the power to override our platform, both at the infrastructure level (the servers that run the code), as well as the content itself.


## Why Flow?

When building a web3 application, the “tech stack” is usually composed of a traditional tech stack (e.g. servers, frontend frameworks, databases, etc), as well as a web3-specific stack that allows us to interact with a blockchain (cli's, client libraries, wallets).

Like all tech stacks, there’s no right or wrong tool for the job - just different ones, and each come with their unique characteristics. It really depends on what _you_ believe are the most important factors. I recommend you follow this guide regardless of what blockchain you think is best, and then try to implement this same guide on your favorite blockchain as a challenge!

In this guide, we’ll be building a Twitter clone using the Flow blockchain. The main reasons behind choosing Flow are: 

1. Flow is used by some of the most mainstream web3 applications, like NBA Top Shot, Instagram and YouTube.
2. When it comes to smart contracts, Cadence (Flow’s native smart contract programming language) is a safer, more readable language than Solidity or Rust, and features a resource-oriented paradigm that makes working with digital assets a lot more intuitive (i.e. you can’t lose or duplicate things accidentally like you can on other chains)
3. Flow has a very intuitive account model whereby digital assets are stored in the respective accounts that own them - not centralized in a key/value map in the smart contract like Ethereum. This distributed ownership and improves performance and security.
4. Flow has extremely low fees, which are usually subsidized by the wallets, meaning it’s effectively free to build and transact. Perfect for creating a bustling social network!
5. Thanks to its scalability, storage is really cheap on Flow, meaning we can build it entirely on-chain. More on that later!
6. Most importantly: Flow was built and designed from the ground up for composability and writing complex applications powered by smart contracts.

Read more about Flow [here](https://flow.com/primer). 

## Getting Started


### What we’re going to build

In this guide we’ll build a simplified version of Twitter. We'll start basic, and progressively enhance it. This will minimize the cognitive load, and help focus on novel concepts before we introduce additional abstractions/complexity.

#### What we'll cover

- Part 1 - Smart Contracts
    - Learn Cadence basics by writing a smart contract (3 times!)
    - Interact with the smart contract using scripts and transactions (i.e. reading and mutating the chain)
    - Learn how to use the Flow Playground
- Part 2 - Building an web app & Tooling
    - Authentication
    - Flow Client Library
    - Connect the smart contract to a simple web app in React/NextJS
    - Develop locally using the Flow Emulator
    - Learn about the Flow CLI and Dev Wallet
    - Learn how to run scripts & transactions on the Flow CLI
    - Deploying to testnet


Let's dive in, shall we?

## Smart Contract v0 - A *Very* Basic Example: A single Tweet.

Given that most devs are familiar with a typical web stack, we'll start with the new interesting bit first - the smart contract. Writing smart contracts usually involves interacting with the chain via commands and CLIs. We'll definitely do that, but first, we'll start our journey in the [Flow Playground](https://play.flow.com), where the chain is emulated and abstracted away for the most part, so we can focus mainly on the smart contract logic. 


### Our first smart contract in Cadence

> 📖 What is Cadence? [Cadence](https://developers.flow.com/cadence) is Flow’s native smart contract programming language. As mentioned earlier, Cadence is a safe, powerful language, and features a resource-oriented paradigm that makes working with digital assets a lot more intuitive (i.e. you can’t lose or duplicate things accidentally like you could on other chains). You might think - "Oh no, a new language I need to learn". Don't worry, it's actually quite easy to grasp, and if you're at all used to TypeScript or Swift, you'll find it's very familiar.

This very first iteration of the contract only allows to create and store a single tweet. Not exciting, I know, but it gives me a chance to explain some fundamentals before we start introducing more complexity.

Before explaining what it does, try to read the code below line by line and see if you can make sense of it with out ANY context at all. I'll wait. (And yes, you are allowed to read the comments)

```swift!=
// Twitter-v0.cdc
//

pub contract Twitter {

    // Declare the Tweet resource type
    pub resource Tweet {
        // The unique ID that differentiates each Tweet
        pub let id: UInt64

        // String mapping to hold metadata
        pub var metadata: {String: String}

        // Initialize both fields in the init function
        init(message: String) {
            self.id = self.uuid
            self.metadata = {
                "message": message
            }
        }
    }

    // Function to create a new Tweet
    pub fun createTweet(_ message: String): @Tweet {
        return <-create Tweet(message: message)
    }

    // Create a single new Tweet and save it to account storage
    init() {
        self.account.save<@Tweet>(<- self.createTweet("My first tweet!"), to: /storage/TwitterPath)
    }
}

```

Ok, let's dissect it bit by bit.

#### Contract

```swift
pub contract Twitter {
```

With this line, we created a contract. We gave it the name "Twitter" because a contract is essentially a program that lives on the blockchain. Within a contract, functions and properties can exist. We can also import other contracts and use *their* functions and properties within our own contract, making them extremely composable. In our case, our Twitter contract will be the main contract of our application.

You'll see towards the end of the contract that there's a `init()` function. When the contract is deployed (i.e. saved on the chain somewhere), it will be initialized and the init function will be called once.

#### Resources

```swift
pub resource Tweet {
```

Most other blockchains have what's called a central ledger. Basically a `{key: value}` map property within the smart contract that keeps track of all the state. This means if anyone (malicious or not) has access to the contract, they can change that map however they like and modify the global state. Cadence has something called resources, which are special objects that are stored in users' own accounts. Anything can be a resource (e.g. NFTs), and resources benefit from the resource ownership rules that are enforced by the type system, that is: resources can only have a single owner, they cannot be duplicated, and they cannot be lost due to accidental or malicious programming errors. These protections ensure that unique digital assets, whether Tweets or NFTs, or whatever else, can be safely represented on Flow.


```swift=
// The unique ID that differentiates each Tweet
pub let id: UInt64

// String mapping to hold metadata
pub var metadata: {String: String}

// Initialize both fields in the init function
init(message: String) {
    self.id = self.uuid
    self.metadata = {
        "message": message
    }
}
```

A resource is similar to a contract in that it can contain properties, functions, including the lifecycle `init` function, which will be called when the resource is created.

> 📖 All composite types like contracts, resources, and structs can have an optional init() function that only runs when the object is initially created. Cadence requires that all fields must be explicitly initialized, so if the object has any fields, this function has to be used to initialize them. [Source](https://developers.flow.com/cadence/tutorial/03-resources)

In our case, the Tweet resource type has 2 properties: `id` and `metadata`. These are *completely arbitrary*. It just makes sense to give a unique resource an id, and the metadata object is a blank object that we can fill with whatever we need. 

On Flow, all resources have a unique id automatically assigned to them, i.e. [a `uuid` property](https://developers.flow.com/cadence/language/resources#resource-identifier), that in this case we are using to set the "official" ID of our Tweet. This ensures uniqueness. We could also create our ID from scratch, or use an incrementing counter, but `uuid` is generally recommended (at least IMHO!). 

When we create the resource we're going to pass in an argument called message, which we will use to populate the metadata object with a key called message. Here's this exact process in action:

```swift=
// Function to create a new Tweet
pub fun createTweet(_ message: String): @Tweet {
    return <-create Tweet(message: message)
}
```

In this snippet, which belongs within the Twitter contract, is a function that creates a new Tweet. The Twitter contract is so basic that we don't even need it, as we only call it once in the `init`, but it shows you how to create a function. What's more interesting is what happens inside of it. You see we `create` a new Tweet resource (resources have to be explicityly `create`d or `destroy`ed with the respective keywords), or moved (read on!)

There's also bunch of weird symbols `_`, `@`, `<-`. Let's see what they do in order:

- `_`: just means it's the default argument, so we don't need to add a "message" label when we create the Tweet.
- `@`: what comes after the colon in the function definition is the expected return type. In this case the @ symbol means it's a resource.
- `<-`: looks like an arrow right? It is! The move operator in Cadence, through which resources are passed around the execution state. Before the transaction ends the resource must be stored somewhere or destroyed otherwise Cadence will throw an error. More on that later.

> 👍 If you find any more weird symbols, just look them up in the **[Glossary](https://developers.flow.com/cadence/language/glossary)** (Fun fact: I contributed the first version of that Glossary when I started building on Flow because I also was confused by all the symbols!).

```swift
// Create a single new Tweet and save it to account storage
init() {
    self.account.save<@Tweet>(<- self.createTweet("My first tweet!"), to: /storage/TwitterPath)
}
```

Finally, we get to the contract's own `init` method. Within it, the contract creates a Tweet upon initialization, containing the message "My first tweet!" and stores it within the account in a custom path called `TwitterPath`. Think of account storage a bit like directories, we need to create our custom "folder" within which to save our resource. These names should be unique. Read more about Cadence's [Account Storage](https://developers.flow.com/cadence/language/accounts#account-storage) and [storing data](https://developers.flow.com/learn/concepts/storage) on Flow.

> 📖 If you want to dive a little deeper into Cadence and Flow's account-based ownership read these posts:
> - https://flow.com/post/resources-programming-ownership
> - https://flow.com/post/flow-blockchain-cadence-programming-language-resources-assets


### Flow Playground - Deploying and interacting with our contract 
Looking at code is cool, but running it is even cooler. Let's now interact with the contract. To do that, we'll use the Flow Playground. Click on the image or the link below to go to the live Playground example. 

[![](https://i.imgur.com/bM1BkaJ.png)](https://play.flow.com/c1ab866b-c71d-41af-a832-29d89434cc50?type=account&id=d8b423b1-2051-47f3-964c-61a5634a7667&storage=0x01)


#### [👉 Click me to view in Playground](https://play.flow.com/c1ab866b-c71d-41af-a832-29d89434cc50?type=account&id=d8b423b1-2051-47f3-964c-61a5634a7667&storage=0x01)

----


1. Click on `0x01` and `Deploy` the contract in the top right.

![](https://i.imgur.com/fngvsho.png)

2. Once you deploy the contract you should see a confirmation like the following. 

![](https://i.imgur.com/xdOrnDv.png)


What did we do? We deployed the contract to the emulated chain to the `0x01` account. This means the contract is now "running" on that account and can be called from other accounts or imported into other smart contracts.

**IMPORTANT**: the playground is an **emulator**, this means it's self-contained. If we wanted other real users to interact with it, we would need to deploy this to testnet or mainnet. We'll get to that, don't worry!

Now let's "see" the resource we created. Click on the database icon on the right of `0x01`:

![](https://i.imgur.com/eYEo883.png)

A popup should appear like the following:

![](https://i.imgur.com/K90R9ny.png)

Here's the full JSON:

```
{
  "Fields": [
    38,
    38,
    {
      "DictionaryType": {
        "ElementType": {},
        "KeyType": {}
      },
      "Pairs": [
        {
          "Key": "message",
          "Value": "My first tweet!"
        }
      ]
    }
  ],
  "ResourceType": {
    "Fields": [
      {
        "Identifier": "uuid",
        "Type": {}
      },
      {
        "Identifier": "id",
        "Type": {}
      },
      {
        "Identifier": "metadata",
        "Type": {
          "ElementType": {},
          "KeyType": {}
        }
      }
    ],
    "Initializers": null,
    "Location": {
      "Address": "0x0000000000000005",
      "Name": "Twitter",
      "Type": "AddressLocation"
    },
    "QualifiedIdentifier": "Twitter.Tweet"
  }
}
```

Nothing too crazy right? It's just a very verbose JSON object, containing the properties we expect of our resource. 

#### Running a Transactions

If you click on the first Transaction "Check if Tweet exists"
![](https://i.imgur.com/Eqn8g35.png)

In general, Transactions are signed and they **mutate** the chain, whereas Scripts only **read** the chain, meaning they can have no side-effect on the chain itself.

> Think of **transactions** like authorized `POST` requests, and **scripts** like unauthorized `GET` requests to a traditional backend.

![](https://i.imgur.com/xREtLB0.png)

In our case, we will run a transaction to look within our own contract and check that the Tweet resource exists. 

Click on "Send". 

![](https://i.imgur.com/eS0GUH9.png)

You should see "Tweet exists" in the transaction results.


#### Tweet again or die tryin'

Let's try to create a new Tweet, just to show you why it's not possible with our current contract.

![](https://i.imgur.com/TsMl0oy.png)

There's a second transaction called "Create new Tweet". Here's the code for it:

```swift=
// (try to) Create new tweet

import Twitter from 0x01

// This transaction checks if a tweet exists in the storage of the given account
// by trying to borrow from it. If the borrow succeeds (returns a non-nil value), the token exists!
transaction {
    prepare(acct: AuthAccount) {
        acct.save<@Twitter.Tweet>(<-Twitter.createTweet("This is my second tweet"), to: /storage/TwitterPath)
    }
}
```

If you try to run this transaction, which is more or less identical to the code in the contract's `init` function, we'll get an error:

![](https://i.imgur.com/7Wrr2MN.png)

As you can infer from the error, when we initialized the contract, we stored our first Tweet resource in a location (`TwitterPath`), and now that location cannot be re-used for another Tweet, as it would overwrite the other Tweet. Dang! How do we fix this? We'll cover this in the next section :) 

### Summary

Let's recap: in this section we created a super basic contract to showcase some initial key features of Cadence, and we interacted with the code in the Flow Playground, an emulated environment that abstracts away a lot of the complexity like wallets. 

Despite having started with a very basic example, it's a LOT to cover. So don't worry if you still don't get all of it. I've purposely created this tutorial in phases, so that at every phase, we will cover old ground and will help you *get* it.

## Smart Contract v1 - Basic Example: Multiple Tweets

Alright, we've covered the *super* basic example, now let's start going a bit deeper in the Cadence rabbit hole and introduce some new concepts. 

First off, let's fix one of the biggest, glaring issues with our first iteration of the Twitter contract: it only saves 1 tweet 😅

Something cool about resources is that **resources can contain other resources**. On Flow resources that are made to contain the same type of something are most commonly called **Collections**. Collections are none other than a resource that we create that will be responsible for storing other resources (our Tweets). That way, rather than storing a single Tweet in our Twitter storage path, we will create and store a collection resource, and the collection will expose  dedicated methods to write/read/remove resources (Tweets) contained within. 

> 👉 Fun fact: on Flow, NFT projects use collections to allow accounts to store multiple NFTs of the same project within their accounts. A completely on-chain [NFT Catalogue](https://www.flow-nft-catalog.com/) helps ensure other dapps like Marketplaces or wallets know what collection (and location) to search for when trying to load a specific projects' NFTs. 


### Adding a Collection Resource

In this section we will do the *bare-minimum* to get a collection working. We'll significantly enhance it later.

```swift=
// Twitter-v1.cdc
// Still basic, but with multiple tweets!

pub contract Twitter {

    // Declare a Path constant so we don't need to harcode in tx
    pub let TweetCollectionStoragePath: StoragePath

    // Declare the Tweet resource type - nothing changed here!
    pub resource Tweet {
        // The unique ID that differentiates each Tweet
        pub let id: UInt64

        // String mapping to hold metadata
        pub var metadata: {String: String}

        // Initialize both fields in the init function
        init(message: String) {
            self.id = self.uuid
            self.metadata = {
                "message": message
            }
        }
    }

    // Function to create a new Tweet
    pub fun createTweet(_ message: String): @Tweet {
        return <-create Tweet(message: message)
    }

    // NEW! 
    // Declare a Collection resource that contains Tweets.
    // it does so via `saveTweet()`, 
    // and stores them in `self.tweets`
    pub resource Collection {
        // an object containing the tweets
        pub var tweets: @{UInt64: Tweet}

        // a method to save a tweet in the collection
        pub fun saveTweet(tweet: @Tweet) {
            // add the new tweet to the dictionary with 
            // a force assignment (check glossary!)
            // If there were to be a value at that key, 
            // it would fail/revert. 
            self.tweets[tweet.id] <-! tweet
        }

        // get all the id's of the tweets in the collection
        pub fun getIDs(): [UInt64] {
            return self.tweets.keys
        }

        init() {
            self.tweets <- {}
        }

        destroy() {
            // when the Colletion resource is destroyed, 
            // we need to explicitly destroy the tweets too.
            destroy self.tweets
        }
    }

    // create a new collection
    pub fun createEmptyCollection(): @Collection {
        return <- create Collection()
    }

    init() {
        // assign the storage path to /storage/TweetCollection
        self.TweetCollectionStoragePath = /storage/TweetCollection
        // save the empty collection to the storage path
        self.account.save(<-self.createEmptyCollection(), to: self.TweetCollectionStoragePath)
    }
}
```

Let's break down what changed. Here's the diff to make it easier to spot the differences. 

![](https://i.imgur.com/KhXwBod.png)


What you'll notice is the `Tweet` resource and `createTweet` function have not changed. Here are the changes:
- a Path constant (line 7) - by defining it, we won't have to hard code it in our transactions and scripts.
- a new `Collection` resource (lines 26-56) along with a `createEmptyCollection()` function to create a new (empty) collection.
- changed the `init()`, to assign the newly created Path constant, and then make use of it in the next line where we create a new collection and save it in the account.

There's a couple new things on the Cadence side here:

##### Collection 

```swift 
pub resource Collection {
    // an object containing the tweets
    pub var tweets: @{UInt64: Tweet}
```

The Collection resource contains a [dictionary of Resources](https://developers.flow.com/cadence/language/resources#resources-in-arrays-and-dictionaries) called `tweets` (identified by the `@{...}`). In our case, the keys are the Tweet IDs, and the values are the Tweet resources. 

Now let's how we populate that dictionary.

##### Modifying a resource dictionary

```swift=
// a method to save a tweet in the collection
pub fun saveTweet(tweet: @Tweet) {
    // add the new tweet to the dictionary with 
    // a force assignment (check glossary!)
    // If there were to be a value at that key, 
    // it would fail/revert. 
    self.tweets[tweet.id] <-! tweet
}
```

The function called `saveTweet`, expects a Tweet resource and saves it in the resource dictionary. You'll notice a weird exclamation mark (!) next to the move (<-) operator. This is called a [force-assignment move operator](https://developers.flow.com/cadence/language/glossary#--lower-than-hyphen-exclamation-mark-force-assignment-move-operator) which assigns a resource to an [optional variable](https://developers.flow.com/cadence/language/values-and-types#optionals). The result of a dictionary read is optional, as the given key might not exist in the dictionary. In this case, we actually *need* it to be nil, otherwise the force-assignment will fail.

> 📖 Don't worry, you'll get a hand for optionals as we go through the course. Also, when you see question marks in transaction code, you're likely looking at [optional chaining](https://developers.flow.com/cadence/language/composite-types#accessing-fields-and-functions-of-composite-types-using-optional-chaining).

Let's move to the second half of the Collection resource:

```swift!
...
    // get all the id's of the tweets in the collection
    pub fun getIDs(): [UInt64] {
        return self.tweets.keys
    }

    init() {
        self.tweets <- {}
    }

    destroy() {
        // when the Colletion resource is destroyed, 
        // we need to explicitly destroy the tweets too.
        destroy self.tweets
    }
}
```

`getIDs()` uses a built-in property of dictionaries/maps in Cadence to return all the keys. You guessed it: it's `.keys`. We'll use this function later to check how many tweets are saved in the account when we interact with the contract.

In the `init` method we simply initialize tweets to be an empty resource dictionary.

The `destroy` method is new. If the Collection resource is destroyed with the destroy command, it needs to know what to do with the resources it stores in the dictionary. This is why resources that store other resources have to include a destroy function that runs when destroy is called on it. This destroy function has to either explicitly destroy the contained resources or move them somewhere else. In this example, we destroy them.


#### Updated Contract init function

As mentioned above, in the init function we assign and make use of our new Path constant. Rather than creating a tweet right off the bat, we create a new collection, and we'll create Tweets using transactions in just a second!

```swift
init() {
    // assign the storage path to /storage/TweetCollection
    self.TweetCollectionStoragePath = /storage/TweetCollection
    // save the empty collection to the storage path
    self.account.save(<-self.createEmptyCollection(), to: self.TweetCollectionStoragePath)
}
```

Alright, with the contract explained, let's play!


----



### Flow Playground - Deploying and interacting with the new contract 
Let's interact with the new and improved contract. We'll of course jump back to the Flow playground, but with a new link. Click on the image or the link below to go to the updated live playground example. 

[![](https://i.imgur.com/0hmYfzz.png)](https://play.flow.com/95ea4027-2d75-4204-999d-c4de30083af3?type=account&id=67bbd568-3be9-407b-a1e1-ce03d1a705b7&storage=0x01)


#### [👉 Click me to go to the Playground Example](https://play.flow.com/95ea4027-2d75-4204-999d-c4de30083af3?type=account&id=67bbd568-3be9-407b-a1e1-ce03d1a705b7&storage=0x01)

#### Creating Tweets

Rather than creating a Tweet automatically in the contract initialization step, now we'll purely create them through a transaction. What's more, is that our transaction will accept an argument with will be the message of the Tweet. 

You can view the transaction code by clickin gon "Create new Tweet" in the Transaction templates of the Playground.

![](https://i.imgur.com/MrWgvUx.png)

Here's the code if you're lazy:

```swift 
// Create new Tweet

import Twitter from 0x01

// This transaction creates a new tweet with an argument
transaction (message: String) {
    // Let's check that the account has a collection
    prepare(acct: AuthAccount) {
        if acct.borrow<&Twitter.Collection>(from: Twitter.TweetCollectionStoragePath) != nil {
            log("Collection exists!")
        } else {
            // let's create the collection if it doesn't exist
            acct.save<@Twitter.Collection>(<-Twitter.createEmptyCollection(), to: Twitter.TweetCollectionStoragePath)
        }

        // borrow the collection
        let collection = acct.borrow<&Twitter.Collection>(from: Twitter.TweetCollectionStoragePath)

        // call the collection's saveTweet method and pass in a Tweet resource
        collection?.saveTweet(tweet: <-Twitter.createTweet(message))
        log("Tweet created successfully, with message ".concat(message))
    }
}
```

Right off the bat, differently from our previous transaction where we attempted to create a second tweet (which failed), this new transaction accepts an argument, called `message`. We'll pass that argument to the `createTweet` function. 

You notice we *only* interact with the Collection resource now, and the collection resource will be responsible for managing the tweets contained within it. 

If we were to break up the transaction into rough logic chunks:
- Check if collection exists in account running transaction
  - If collection exists, log "collection exists" (just fyi)
  - If collection does NOT exist, create one in that account
- Borrow the collection to call saveTweet, and pass in a newly created Tweet using the createTweet function. 

You may be asking yourself - wait - why are we checking for a collection if we automatically create one when we initalize the contract? Well, because that only happens on the account where the contract is deployed to. *Other* accounts can interact with the contract and create their own tweets. So if *any other* account wants to create a Tweet, they first need a collection. So we must always check for a collection to be present whenever we are creating/saving a Tweet inside someones account. 

Let's create our first Tweet:

![](https://i.imgur.com/xXAtwWw.png)

You should see: 

![](https://i.imgur.com/ZWsdmdv.png)


Let's create our second one:

![](https://i.imgur.com/B7EFAYj.png)

You should see:

![](https://i.imgur.com/gfkG8d7.png)


#### Counting Tweets in an Account

Now to double check, let's run a second transaction. You'll find it right under the first one in the Playground template. `Get tweets` which calls the Collection's `getIDs` method. It will the length of the array that is returned by `getIDs` all the Tweet IDs contained in the collection. 


```swift 
// Get tweets

import Twitter from 0x01

// This transaction retrieves all Tweets
transaction {
    // Let's check that the account has a collection
    prepare(acct: AuthAccount) {
        let collection = acct.borrow<&Twitter.Collection>(from: Twitter.TweetCollectionStoragePath)
        if collection != nil {
           log (collection?.getIDs()?.length)
        } else {
           log ("Collection does not exist")
        }
    }
}
```

If you run it, you should get the following output: 

![](https://i.imgur.com/T18PQG3.png)

*Note: the number may vary if you created more than 2 tweets!*

#### Creating tweets with another account

Now want to see something cool?

Rather than running the `Create new Tweet` with `0x01`, try selecting any other account (e.g. `0x02`) as the signer. You'll see it doesn't log "Collection exists", as it will create the collection and then create the tweet successfully. Now `0x02` has a Tweet Collection! Yay.

![](https://i.imgur.com/JSpoWWM.png)

### Summary & Limitations with our current implementation

Great, our contracts now supports a collection and we can now create multiple tweets. **However**, other accounts can't read tweets from any other accounts but themselves. This is because we are working with storage directly, which only the owning account has access to. 

How do we get access to other accounts' tweets? The bad news is we need to upgrade our contract (AGAIN). The good news is we get to learn some cool new Cadence features! What we need to do is to start creating [public capabilities](https://developers.flow.com/cadence/language/capability-based-access-control). Capability-based access control is one of Cadence's most powerful features and will regulate access rights outside of the contract and owning account. In our case, we want to be able to read tweets from other accounts, but not create tweets on behalf of other accounts. 

This will also be a great opportunity to introduce the concept of interfaces. Let's get to it!


## Smart Contract v2 - Less Basic: Letting other people view our tweets

### Interfaces and Capabilities

To allow other accounts (or scripts, which are essentially un-authenticated requests) to access our tweets, we need to do two things:
1. Create an public-facing **[interface](https://developers.flow.com/cadence/language/interfaces)** that our Collection resource will conform to
2. Create a **[capability](https://developers.flow.com/cadence/language/capability-based-access-control)** to access the Collection through the public-facing interface

Let's tackle these in order. Here's a formal definition from the docs.

> 📖 **[Interfaces](https://developers.flow.com/cadence/language/interfaces)**: An interface is an abstract type that specifies the behavior of types that implement the interface. Interfaces declare the required functions and fields, the access control for those declarations, and preconditions and postconditions that implementing types need to provide. 
> There are three kinds of interfaces:
> - Structure interfaces: implemented by structures
> - Resource interfaces: implemented by resources
> - Contract interfaces: implemented by contracts

> 📖 **[Capabilities](https://developers.flow.com/cadence/language/capability-based-access-control)**: if an account wants to be able to access another account's stored objects, it must have a valid capability to that object. Capabilities are identified by a path and link to a target path, not directly to an object. Capabilities are either public (any user can get access), or private (access to/from the authorized user is necessary). Public capabilities are created using public paths, i.e. they have the domain public. After creation they can be obtained from both authorized accounts (AuthAccount) and public accounts (PublicAccount).

It's easier to explain by showing you the code, so if it's still confusing, keep on reading!

### Updating the contract with interfaces and capabilities

Here's the new contract code. Scroll down for a diff and we'll cover each change bit by bit!

```swift=
// Twitter-v2.cdc
// Still basic, but with multiple tweets (and capabilities!)

pub contract Twitter {

    // Declare a Path constant so we don't need to harcode in tx
    pub let TweetCollectionStoragePath: StoragePath
    pub let TweetCollectionPublicPath: PublicPath

    // Declare the Tweet resource type - nothing changed here!
    pub resource Tweet {
        // The unique ID that differentiates each Tweet
        pub let id: UInt64

        // String mapping to hold metadata
        pub var metadata: {String: String}

        // Initialize both fields in the init function
        init(message: String) {
            self.id = self.uuid
            self.metadata = {
                "message": message
            }
        }
    }

    // Function to create a new Tweet
    pub fun createTweet(_ message: String): @Tweet {
        return <-create Tweet(message: message)
    }

    pub resource interface CollectionPublic {
        pub fun getIDs(): [UInt64]
        pub fun borrowTweet(id: UInt64): &Tweet? 
    }

    // NEW! 
    // Declare a Collection resource that contains Tweets.
    // it does so via `saveTweet()`, 
    // and stores them in `self.tweets`
    pub resource Collection: CollectionPublic {
        // an object containing the tweets
        pub var tweets: @{UInt64: Tweet}

        // a method to save a tweet in the collection
        pub fun saveTweet(tweet: @Tweet) {
            // add the new tweet to the dictionary with 
            // a force assignment (check glossary!)
            // If there were to be a value at that key, 
            // it would fail/revert. 
            self.tweets[tweet.id] <-! tweet
        }

        // get all the id's of the tweets in the collection
        pub fun getIDs(): [UInt64] {
            return self.tweets.keys
        }

        pub fun borrowTweet(id: UInt64): &Tweet? {
            if self.tweets[id] != nil {
                let ref = (&self.tweets[id] as &Twitter.Tweet?)!
                return ref
            }
            return nil
        }

        init() {
            self.tweets <- {}
        }

        destroy() {
            // when the Colletion resource is destroyed, 
            // we need to explicitly destroy the tweets too.
            destroy self.tweets
        }
    }

    // create a new collection
    pub fun createEmptyCollection(): @Collection {
        return <- create Collection()
    }

    init() {
        // assign the storage path to /storage/TweetCollection
        self.TweetCollectionStoragePath = /storage/TweetCollection
        self.TweetCollectionPublicPath = /public/TweetCollection
        // save the empty collection to the storage path
        self.account.save(<-self.createEmptyCollection(), to: self.TweetCollectionStoragePath)
        // publish a reference to the Collection in storage
        self.account.link<&{CollectionPublic}>(self.TweetCollectionPublicPath, target: self.TweetCollectionStoragePath)
    }
}
```

#### Diff

![](https://i.imgur.com/3J17bVb.png)

You'll notice, not too much has changed. Here's what I added:

- Added a new Path constant called `TweetCollectionPublicPath`, that we will reference later to expose the Collection publically.
- Added a new resource interface `Collection Public`, which I then assign to the `Collection` resource.
- Created a new method `borrowTweet` in the Collection that returns a reference to the Tweet based on a Tweet ID. 
- Updated the contract `init()` function to 
  -  initialize the new Path constant above
  -  save a capability to access the collection within the scope of the Collection Public interface.

#### Public Collection Interface

Let's start with the `Collection Public`.

```swift=
pub resource interface CollectionPublic {
    pub fun getIDs(): [UInt64]
    pub fun borrowTweet(id: UInt64): &Tweet? 
}
```

What we're doing here is describing a restricted version of the Collection that is for public consumption. As you can see, the only two methods included in this interface are `getIDs` and `borrowTweet` (a new method which we still need to look at - soon!).

How do we ensure that our Collection conforms to this interface? By adding it to the Collection declaration similar to how we add types to variables.

`pub resource Collection: CollectionPublic {`

Resources can implement multiple interfaces. You should see interfaces in two ways:
1) ensure a minimum set of functionality is implemented, ensuring compatibility
2) ensure *only* a subset of the original functionality is within scope, ensuring that other methods or properties will not be included.

In our specific case, we are using interfaces mainly for reason #2.

So how do we use this interface to grant the public a restricted access to our collection?

Look at the last line (90) of the contract, within the `init()`:

```swift
self.account.link<&{CollectionPublic}>(self.TweetCollectionPublicPath, target: self.TweetCollectionStoragePath)
```

The line above creates a public capability that allows access to the stored Collection resource but only through the type `{CollectionPublic}`, i.e. only the functionality outlined in the interface. You can think of it as `Collection` being "stripped" of anything other than what is strictly defined in `CollectionPublic` before it is made available.

You can see how this can create powerful and flexible access controls. We can create interfaces and capabilities for various actors/counterparts within our application and Cadence will do the hard work of enforcing control.

> ⚠️ It's not terribly important in this example, but in the code above, `&{CollectionPublic}` is the equivalent of writing `&AnyResource{CollectionPublic}`. That is, any resource that conforms to the `CollectionPublic` interface will be valid. If we want to scope it to *only* accept *the* collection resource that we created, we would have to re-write it as `&Collection{CollectionPublic}`. This would guarantee that not only does the resource have to conform to the interface, but it needs to be the specific underlying resource. 

#### Borrowing a resource

So we've created a capability that allows us access to two methods of the collection: `getIDs` and `borrowTweet`. The latter is new, so let's take a look at it. 

```swift 
pub fun borrowTweet(id: UInt64): &Tweet? {
    if self.tweets[id] != nil {
        let ref = (&self.tweets[id] as &Twitter.Tweet?)!
        return ref
    }
    return nil
}
```

🚨 New symbol alert! What the heck is `&`? The ampersand (&) represents a [`reference`](https://developers.flow.com/cadence/language/references) to a resource. Rather than loading the resource from storage and passing it back and forth, in Cadence we can create references to any object (both resources or structures). A reference can be used to access fields and call functions on the referenced object as a value type.

References are created by using the `&` operator, followed by the object, the `as` keyword, and the type through which they should be accessed. The given type must be a supertype of the referenced object's type. 

If we zero in on the line `let ref = (&self.tweets[id] as &Twitter.Tweet?)!`, this is exactly what we do: we create a reference of the Tweet resource at a certain ID, and cast it as a Tweet type. Since the ID could not exist, the dictionary returns an optional, so we use the exclamation mark at the end to force unwrap it. 


### Flow Playground - Deploying and interacting with the new contract 
Let's interact with the new and improved contract. Click on the image or the link below to go to the updated live playground example. 

[![](https://i.imgur.com/fxoVGCV.png)
](https://play.flow.com/02aeaca5-3516-45f8-ae79-9f10d5db3cb5?type=account&id=d80cd930-fab4-4c51-a3c3-824deec36c96&storage=none)


#### [👉 Click me to go to the Playground Example](https://play.flow.com/02aeaca5-3516-45f8-ae79-9f10d5db3cb5?type=account&id=d80cd930-fab4-4c51-a3c3-824deec36c96&storage=none)


First things first deploy the contract! 

#### Updating the transaction

We still have the same transactions as last time, although you'll notice the `Create new Tweet` transaction has an extra line: 

```swift
// publish a reference to the Collection in storage
acct.link<&{Twitter.CollectionPublic}>(Twitter.TweetCollectionPublicPath, target: Twitter.TweetCollectionStoragePath)
```

This is the same line that we added to the init of the contract. This will ensure all other accounts that interact with the Twitter contract will also link the public capability.

Create a couple random Tweets from `0x01` and `0x02` in the same way we did in the previous section.

### Structs, and running our first script

Scripts *query* the chain without affecting its state. Since there is no risk of side-effects, scripts can be executed by anyone, similar to an unauthorized GET request. Similar to how in a normal server you have to expose a port and serve specific data, on Flow we used the public capability to expose our collection, which the script below will make use of.

```swift=
import Twitter from 0x01

pub struct TweetMetadata {
    pub let id: UInt64
    pub let message: String 

    init(id: UInt64, message: String) {
        self.id = id
        self.message = message
    }
}

// Get tweets owned by an account
pub fun main(account: Address): [TweetMetadata] {
    // Get the public account object for account
    let tweetOwner = getAccount(account)

    // Find the public capability for their Collection
    let capability = tweetOwner.getCapability<&{Twitter.CollectionPublic}>(Twitter.TweetCollectionPublicPath)

    // borrow a reference from the capability
    let publicRef = capability.borrow()
            ?? panic("Could not borrow public reference")

    // get list of tweet IDs
    let tweetIDs = publicRef.getIDs()

    let tweets: [TweetMetadata] = []

    for tweetID in tweetIDs {
        let tweet = publicRef.borrowTweet(id: tweetID) ?? panic("this tweet does not exist")
        let metadata = TweetMetadata(id: tweet.id, message: tweet.metadata["message"]!)
        tweets.append(metadata)
    }

    return tweets
}
```

A very basic script only needs to implement a `pub fun main()` function. In our script you'll notice we also define a `struct` called `TweetMetadata`. This is not really required, but I just wanted to show you what structs are like. 
Similar to resources, [structs](https://developers.flow.com/cadence/language/composite-types#structures) are [composite types](https://developers.flow.com/cadence/language/composite-types#structures), in that they can implement interfaces and also can have an init function. The main difference is structs are regular value types meaning they don't need to be created/moved/destroyed like a resource would. 

In our case, we are using this struct to create an intermediate abstraction of what the contents of a Tweet should be. This means that if our contract changes, we just need to change our TweetMetadata struct and anything downstream that relies on it will just work.

> Note: in future sections we'll continue to iterate on this contract and make the metadata even more elegant by using what's called the Metadata Standard called [MetadataViews](https://github.com/onflow/flow/blob/master/flips/20210916-nft-metadata.md)


The rest of the script is relatively straightforward:

- Get the public account object using the built-in `getAccount` method, using the account passed in as the argument of the main function.
- From that account, we borrow the `CollectionPublic` capability (if it doesn't exist, we panic, meaning the script will abort)
- If it exists, we call the `getIDs` the same way we call it in our transaction
- using the Cadence [looping function](https://developers.flow.com/cadence/language/control-flow#for-in-statement) `for ... in ... {}` , we iterate over every ID
- In each iteration of the loop, we use the Collection's `borrowTweet` method to populate a new TweetMetadata struct and append it to  a `tweets` array which will be our return value of the script. 


Try running the script! Just select it in the Script Templates.

![](https://i.imgur.com/CIF7zgM.png)

Similar to the transaction send window, you'll see a script execution window where we can specify any arguments. Since our script 

![](https://i.imgur.com/QvujAnY.png)


If you've created at least 1 tweet with 0x01, you should see a script result like the following:

![](https://i.imgur.com/QqI6JD8.png)


Here it is pretty-formated:

![](https://i.imgur.com/f2lbIEA.png)

It seems like a complicated object, but the good news is "in real life" when we use this script in our app or via the Flow CLI, the object is "flattened" and types are inferred, so we'll work with a regular object containing the contents we expect (id and message).


### Summary

In this third iteration of the contract, we added public capabilities to our collection, allowing anyone to access a public version, so that when we build our first POC app, we can view tweets from other people!

I think we've spent enough time on the Playground for now. Let's start building an app so we can see how all this fits together.


## Building a first app with the v2 contract

### Preface

In this section, we'll build an app and see how to interact with the chain. First, I just want to make something clear that the whole point of this tutorial is *not* to show you how much better building on the blockchain is compared to using traditional servers. It's to help you understand blockchain concepts applied to a use-case that we're all used to, regardless of whether you're in the web3 space or not. 

Much like we iterated on the contract several times in the previous sections, in this section we will iterate back and forth both on the contract and web app as we add more and more features. I really hope you enjoy this "progressive enhancement" type of learning. It's more verbose than jumping right to the final version, but I think makes for a gentler introduction and helps you absorb concepts a bit more gradually.

Here's a sneak peek of the app:

![](https://user-images.githubusercontent.com/27052451/202925591-1053d059-57a5-419c-9884-339ef7ed1001.png)

### Our app's tech stack

Let's pick what web stack to use. 

#### Web Framework - NextJS, an "obvious" choice

To develop our app, we'll use a web framework. [React/NextJS](https://nextjs.org/) is the industry standard, so we'll use that.

> 😅 I'm a [Svelte](https://svelte.dev) fan - React has weird quirks like JSX, that mixes HTML and CSS in the Javascript..who thought that was a good idea?! I'll probably build a Svelte version of this as well in case anyone reading prefers that, and any contributions in other

#### Minimal CSS styling

I opted to use [`pico.css`](https://picocss.com/) for the styling. It's a drop-in classless CSS framework, meaning it styles the base HTML tags, so we can use good ol' semantic HTML and have styles for free. Cool! 


#### Flow Client Library - the perfect choice

Regardless of what web framework you use, one choice is already made for us. The JS [Flow Client Library](https://developers.flow.com/tools/fcl-js/index). 

The Flow Client Library (FCL) JS is a package used to interact with user wallets and the Flow blockchain. When using FCL for authentication, apps are able to support all FCL-compatible wallets on Flow and their users without any custom integrations or changes needed to their code. Sweet!

FCL was created to make developing applications that connect to the Flow blockchain super easy and secure. It defines a standardized set of communication patterns between wallets, applications, and users that is used to perform a wide variety of actions for your dapp. 

> Note: While FCL itself is a concept and standard, FCL JS is the javascript implementation of FCL and can be used in both browser and server environments. All functionality for connecting and communicating with wallet providers is restricted to the browser. We also have FCL Swift implementation for iOS, see [FCL Swift](https://github.com/zed-io/fcl-swift), contributed by @lmcmz.


#### Flow CLI - the one stop shop for all things Flow

Lastly, one other important tool we're going to use is the [Flow CLI](https://developers.flow.com/tools/flow-cli) - a command-line interface that provides useful utilities for building Flow applications. It includes several commands to interact with Flow networks, such as querying account information, deploying contracts, sending transactions and executing scripts. It also includes the Flow Emulator and Dev Wallet (which is a development version Flow wallet used when developing locally on the emulator).

### Setting up our dev environment

#### Prerequisites

Before moving to the next step, ensure you have the following tools installed:

##### NodeJS

Ensure you have NodeJS [installed](https://nodejs.org/en/). Version **16+** is what I'm using currently.

```
> node -v

v16.x.x
```

> **📣 Tip**: Need to manage different versions of NodeJS? Try using [NVM](https://github.com/nvm-sh/nvm).

##### Flow CLI

Ensure you have the [Flow CLI](https://docs.onflow.org/flow-cli/install/) installed too. At the time of this writing, I am using version **0.41.3**.

```
> flow version

Version: v0.41.3
```

### Initializing our project

Rather that have you copy and paste by hand a ton of stuff, I'm going to show you a pre-made app and explain how it works. Then we'll expand on it together!


With your favorite terminal, create and navigate to a new directory like `/twitter3`. Then clone the starter repo: 

```
git clone https://github.com/muttoni/twitter3.git .
```

Let's take a look at the project directory:

![](https://i.imgur.com/GDMRLy0.png)

#### Folder Structure

You can structure your folders however you want, the app shows a common way to do it. Here they are explained order:

##### File Glossary

- 🆕 `/cadence`: contains all Cadence related files, like contracts, transactions and scripts. It contains respective subfolders for each one. If you open the subfolders, you'll see the exact same contract, transaction and script we created in the [Playground example](https://play.flow.com/02aeaca5-3516-45f8-ae79-9f10d5db3cb5?type=account&id=d80cd930-fab4-4c51-a3c3-824deec36c96&storage=none) in the last section, with only 1 small change: the imports.
- `/components`: this is standard NextJS folder to store React components
- 🆕 `/constants`: just a folder to save some handy constants
- 🆕 `/flow`: a folder dedicated to Flow-specific stuff. Currently this folder only contains a config file for the Flow Client Library, but in the future we could move Flow-logic to this folder.
- `/hooks`: standard NextJS Hooks folder
- `/pages`: standard NextJS Pages folder
- `/public`: standard NextJS public assets folder
- `/styles`: standard NextJS styles folder 
- 🆕 `emulatore.private.json`: a config snippet containing sensitive information, which we don't want added to the public flow.json.
- 🆕 `flow.json`: the standard Flow config file that instructs the Flow CLI what contracts to deploy
- ...standard NextJS files

If you've used NextJS in the past, this folder structure looks very familiar, and only the 🆕 folders should be surprising.

#### Installing dependencies
Next let's install dependencies and run our app in dev! 

```
npm install
```

### It's alive!

Make sure to have 3 terminal windows open:
1. App terminal
2. Flow Emulator terminal
3. Flow Dev Wallet terminal


In terminal 2, run

```
flow emulator start
```

In terminal 3, run

```
flow dev-wallet
```

Now in terminal 1, let's start the app!

```
npm run dev:local:deploy
```

If you're curious as to what the command above does, check the `package.json`. In short, it deploys/updates the contracts to the Flow Emulator as defined in our `flow.json`, which is the standard Flow CLI config file, and then runs `npm run dev` for our NextJS app. 


If you ran everything correctly you should see:

![](https://i.imgur.com/8CUovx1.png)

Click on "**Log In With Wallet**"

You should see popup window like the following:

![](https://i.imgur.com/EBqzW4K.png)

This is the FCL discovery service, that exposes available wallets a user may want have/want to use. What's great about FCL is that the wallet is completely abstracted from the app, and the app is completely abstracted from the wallet, so any user can use any wallet and the code and app don't need to care (usually).

Since we're on the Emulator and not on Testnet or Mainnet yet, you'll want to select Dev Wallet. 

You'll then be presented with a Dev Wallet splash screen, where you can select which account to use:

![](https://i.imgur.com/mSkn2wB.png)

This is because in the Dev Wallet you can easily create and switch between different accounts, and quickly fund accounts with imaginary FLOW. The Service account is the "base" account on Flow, and you can create as many other "normal" accounts as you want. Click on Service Account or Account A. 

Alright, now you're authenticated and the UI should reflect that.

![](https://i.imgur.com/aDAQdVi.png)

Feel free to play around with the app first, then we'll start dissecting how it works. 


### Understanding the Flow

Before we look at the app code, I want to show you how to use the Flow CLI by running the exact same transaction and script we ran in our Playground example, only this time "for real" on the emulator.

As mentioned in the glossary, the `/cadence` folder contains all Cadence related files, like our Twitter contract, and the transaction and script we'll need. It contains respective subfolders for each type.

![](https://i.imgur.com/PY9ZmuK.png)


If you open the subfolders, you'll see the exact same contract, transaction and script we created in the [Playground example](https://play.flow.com/02aeaca5-3516-45f8-ae79-9f10d5db3cb5?type=account&id=d80cd930-fab4-4c51-a3c3-824deec36c96&storage=none) in the last section, with only 1 small change: the imports. You'll notice the transaction and script doesn't import `0x01` anymore, but a longer address. This is the address on the emulator that we are deploying the contracts to. You'll see how in just a second!

Let's imagine we don't have an app yet, and we just have these three folders setup. How would we interact with the emulator? The answer is the Flow CLI!

### Deploying a contract to the emulator

In our app, the `npm run dev:local:deploy` command does this automatically, but the underlying command it uses to deploy the contracts (or update them if they've changed) is just as simple:

```
flow project deploy
```

How does it know _what_ to deploy? The answer is in the `flow.json`

### Flow Configuration - flow.json

How does the Flow CLI know what contract to use and what account to find it in? The answer is the [`flow.json`](https://developers.flow.com/tools/flow-cli/configuration) - a project-specific configuration file. Here's what ours looks like:

```json 
{
  "networks": {
    "emulator": "127.0.0.1:3569",
    "mainnet": "access.mainnet.nodes.onflow.org:9000",
    "testnet": "access.devnet.nodes.onflow.org:9000"
  },
  "contracts": {
    "Twitter": {
      "source": "./cadence/contracts/Twitter.cdc",
      "aliases": {
        "emulator": "0xf8d6e0586b0a20c7"
      }
    }
  },
  "accounts": {
    "emulator-account": {
      "fromFile": "./emulator.private.json"
    }
  },
  "deployments": {
    "emulator": {
      "emulator-account": ["Twitter"]
    }
  }
}
```

This configuration outlines what network addresses the CLI should use, what contracts are included in our application and therefore, should be deployed, and to which address. 

> 📖 Read more about the flow.json [here](https://developers.flow.com/tools/flow-cli/initialize-configuration)
> 
### Sending a transaction with the CLI

> 🚨 Check your transaction and scripts in this section and makes ure they are importing the Twitter contract from `0xf8d6e0586b0a20c7` and not `0xTwitter`. The latter is specific to FCL which we will look at later.

Let's start by creating a Tweet. For this we'll use the Flow CLI's `flow transaction send` command. As a first argument it expects the path or string of the transaction, and any arguments after that will be passed as arguments to the transaction. With the command below we're creating a tweet with text "Hello" using the default account and network (emulator) as defined in our `flow.json` 

Try running this in your terminal (with the emulator running):
```
flow transactions send ./cadence/transactions/CreateNewTweet.cdc Hello
```

You should see an output similar to this:

![](https://i.imgur.com/CMskUgJ.png)

You'll notice it used the account `f8d6e0586b0a20c7` which is the main emulator service account we saw earlier (i.e. the service account). Fun fact, the `0x` is completely optional on Flow.

The status is shown as ✅ SEALED, which means the transaction has been fully confirmed and it is committed to the blockchain. There are transaction codes for different intermediate phases (and we'll see how to use them to provide transaction feedback when we deploy the app to testnet). 



| Code    | Value     | Description                                                                                                                               |
| --- | --------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
|  0   | UNKNOWN   | The transaction status is not known.                                                                                                      |
|  1   | PENDING   | The transaction has been received by a collector but not yet finalized in a block.                                                        |
|  2   | FINALIZED | The consensus nodes have finalized the block that the transaction is included in                                                          |
|   3  | EXECUTED  | The execution nodes have produced a result for the transaction                                                                            |
|  4   | SEALED    | The verification nodes have verified the transaction (the block in which the transaction is) and the seal is included in the latest block |
|  5   | EXPIRED   | The transaction was submitted past its expiration block height.                                                                           |


[Source](https://developers.flow.com/nodes/access-api#transaction-status)


> 📖 The `flow transactions send` command accepts other flags, such as what account to sign it as. [See here for more info](https://developers.flow.com/tools/flow-cli/send-transactions#flags)


### Executing a script with the CLI

> 🚨 Check your transaction and scripts in this section and makes ure they are importing the Twitter contract from `0xf8d6e0586b0a20c7` and not `0xTwitter`. The latter is specific to FCL which we will look at later.

To query an account's Tweets, we can use `flow scripts execute`, which is structured very similarly to the transaction command: first argument is the script to execute, and any subsequent arguments will be passed to the script. 

```
flow scripts execute ./cadence/scripts/GetTweetsByAccount.cdc f8d6e0586b0a20c7
```

Let's run it:

![](https://i.imgur.com/QLhei5x.png)

> 📖 The `flow scripts execute` command accepts several flags. [See here for more info](https://developers.flow.com/tools/flow-cli/execute-scripts)


### Flow configuration - config.js

The other important Flow configuration is within the app, and configures the behavior of the Flow Client Library. 

![](https://i.imgur.com/58jy6HV.png)

You'll find this in `/flow/config.js`. This contains standard info like network, access node, as well as more app specific configuration data, like app name and icon. 


### FCL

Alright, so you now know how to interact with the contracts, transactions and scripts via the CLI. Let's start taking a look at how these are connected to the web app via the Flow Client Library (FCL).

Navigate to `/flow/config.js`. You'll see a configuration file.

```js
import { config } from '@onflow/fcl'
import { ACCESS_NODE_URLS } from '../constants'
import flowJSON from '../flow.json'

const flowNetwork = process.env.NEXT_PUBLIC_FLOW_NETWORK

console.log('Dapp running on network:', flowNetwork)

config({
  'flow.network': flowNetwork,
  'accessNode.api': ACCESS_NODE_URLS[flowNetwork],
  'discovery.wallet': `https://fcl-discovery.onflow.org/${flowNetwork}/authn`,
  'app.detail.icon': 'https://avatars.githubusercontent.com/u/50278?s=200&v=4',
  'app.detail.title': 'Twitter3'
}).load({ flowJSON })
```

In this file we are telling FCL to:

#### `flow.network`

`flow.network` is used to specific network (emulator, testnet, mainnet), as defined in the `flowNetwork` variable populated by the .env file. In our case it's going to be the emulator. You'll notice we don't have a .env file anywhere in our project. That's because you have to look in the package.json. See below!

```json 
  ...
  "scripts": {
    "dev": "next dev",
    "dev:local": "NEXT_PUBLIC_FLOW_NETWORK=local npm run dev",
    "dev:local:deploy": "flow project deploy --network=emulator --update && NEXT_PUBLIC_FLOW_NETWORK=local npm run dev",
    "dev:testnet": "NEXT_PUBLIC_FLOW_NETWORK=testnet npm run dev",
    "dev:mainnet": "NEXT_PUBLIC_FLOW_NETWORK=mainnet npm run dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
...
```
#### `accessNode.api`

`accessNode.api` is where we set what Access Node to connect to. This URL will change based on our Flow network. For the emulator, that is `localhost:8888`. You can see the various API urls defined in `constants/index.js`:

```js 
export const ACCESS_NODE_URLS = {
  'local': 'http://localhost:8888',
  'testnet': 'https://rest-testnet.onflow.org',
  'mainnet': 'https://rest-mainnet.onflow.org'
}
```

#### `discovery.wallet`

FCL Discovery is one of the most *powerful* features of FCL. It allows any wallet on Flow to self-announce itself, meaning it will show up automatically in the list of available wallets when a user wants to login to a Flow app, without developers needing to do a *single thing*! Isn't that nifty?

#### `app.detail.icon` and `app.detail.title`

These are purely cosmetic (an icon for your app, and a title), and these attributes will be used to populate the wallet selection popup generated by FCL. In our case we're using the Twitter logo and the title Twitter3. It looks like this:

![](https://i.imgur.com/EBqzW4K.png)


### Hooks

React hooks are a way to set some state that can be used throughout the app. If you go to `/hooks/useConfig.js` and `/hooks/useCurrentUser.js` we'll see two important hooks that also are linked with FCL.


#### `useConfig.js`

In this hook, we are setting state to load the current Flow network. We use `fcl.config.get` to get a specific setting that we set previously.

```js 
import * as fcl from '@onflow/fcl'
import { useEffect, useState } from 'react'

export default function useConfig() {
  const [network, setNetwork] = useState()

  useEffect(() => {
    async function getConfig() {
      const flowNetwork = await fcl.config.get('flow.network')
      setNetwork(flowNetwork)
    }
    getConfig()
  }, [])

  return { network }
}
```

#### `useCurrentUser.js`

In this hook, we're creating a `user` object and populating it based on the state of `fcl.currentUser`. This is a function that can be subscribed to and returns the state of the authenticated (or not) user. This gives us a user object we can query and create conditional logic around throughout the app (e.g. showing a Connect Wallet button if not logged in for example).

```js 
import * as fcl from '@onflow/fcl'
import { useEffect, useState } from 'react'

export default function useCurrentUser() {
  const [user, setUser] = useState({ loggedIn: null })

  useEffect(() => {
    fcl.currentUser.subscribe(setUser)
  }, [])

  return user
}
```

Here's what the `fcl.currentUser` object contains. `loggedIn` and `addr` are usually the most important values:

#### `fcl.currentUser object`

| Key         | Value Type          | Default   | Description                                                                                                                                                                                                                                                                                    |
| ----------- | ------------------- | --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `addr`      | [Address](#address) | `null`    | The public address of the current user                                                                                                                                                                                                                                                         |
| `cid`       | string              | `null`    | Allows wallets to specify a [content identifier](https://docs.ipfs.io/concepts/content-addressing/) for user metadata.                                                                                                                                                                         |
| `expiresAt` | number              | `null`    | Allows wallets to specify a time-frame for a valid session.                                                                                                                                                                                                                                    |
| `f_type`    | string              | `'USER'`  | A type identifier used internally by FCL.                                                                                                                                                                                                                                                      |
| `f_vsn`     | string              | `'1.0.0'` | FCL protocol version.                                                                                                                                                                                                                                                                          |
| `loggedIn`  | boolean             | `null`    | If the user is logged in.                                                                                                                                                                                                                                                                      |
| `services`  | [ServiceObject]     | `[]`      | A list of trusted services that express ways of interacting with the current user's identity, including means to further discovery, [authentication, authorization](https://gist.github.com/orodio/a74293f65e83145ec8b968294808cf35#you-know-who-the-user-is), or other kinds of interactions. |

> 📖 Read more about the currentUser object and FCL [here](https://developers.flow.com/tools/fcl-js/reference/api#currentuserobject).


### Components & Pages

Now that we covered the basic foundations of FCL, let's look at how these are applied in the actual app by looking at its components and pages.

#### `pages/index.js`

This is the main entry point into the app. Here we load the Flow config and use the `useCurrentUser` hook to conditionally render the main container if the user is logged in.

```jsx 
import Head from 'next/head'
import Navbar from '../components/Navbar'
import '../flow/config.js'

import Container from '../components/Container'
import useCurrentUser from '../hooks/useCurrentUser'

export default function Home() {
  const { loggedIn } = useCurrentUser()

  return (
    <div>


      <Head>
        <title>Twitter 3</title>
        <meta name="description" content="Twitter3 - thoughts onchain'ed" />
        <link rel="icon" href="/favicon.ico" />
      </Head>

      <main className="container">
        <Navbar />
        {loggedIn && <Container />}
      </main>
    </div>
  )
}
```

#### `components/Container.js`

This is the other "big" one to look through. In this component we render most of the UI and more or less this is where everything comes together: FCL, the transactions, the scripts and our app functionality. This is the MOTHERLOAD!

```jsx 
import * as fcl from "@onflow/fcl"
import { useEffect, useState } from "react"

import GetTweetsByAccount from '../cadence/scripts/GetTweetsByAccount.cdc'
import CreateNewTweet from '../cadence/transactions/CreateNewTweet.cdc'
import styles from '../styles/Container.module.css'
import useConfig from "../hooks/useConfig"
import useCurrentUser from '../hooks/useCurrentUser'

import { BLOCK_EXPLORER_URLS } from "../constants"

import Tweet from './Tweet'

export default function Container() {
  const [tweetList, setTweetList] = useState([])
  const [tweetText, setTweetText] = useState('')
  const [lastTransactionId, setLastTransactionId] = useState()
  const [transactionStatus, setTransactionStatus] = useState('N/A')
  const { network } = useConfig()
  const user = useCurrentUser()

  const isEmulator = network => network !== 'mainnet' && network !== 'testnet'
  const isSealed = statusCode => statusCode === 4 // 4: 'SEALED'

  useEffect(() => {
    if (lastTransactionId) {
      console.log('Last Transaction ID: ', lastTransactionId)

      fcl.tx(lastTransactionId).subscribe(res => {
        setTransactionStatus(res.statusString)
  
        // Query for new chain string again if status is sealed
        if (isSealed(res.status)) {
          getTweets()
        }
      })
    }
  }, [lastTransactionId])

  const getTweets = async (account) => {
    account = user?.addr;

    let res;

    try {
      res = await fcl.query({
        cadence: GetTweetsByAccount,
        args: (arg, t) => [arg(account, t.Address)]
      })
    } catch(e) {
      res = []
    }

    console.log(res)

    setTweetList(res.sort((a, b) => b.id - a.id))
  }

  const createTweet = async (event) => {
    event.preventDefault()

    if (!tweetText.length) {
      throw new Error('Please add a new greeting string.')
    }

    const transactionId = await fcl.mutate({
      cadence: CreateNewTweet,
      args: (arg, t) => [arg(tweetText, t.String)]
    })

    setLastTransactionId(transactionId)
  }
  
  const openExplorerLink = (transactionId, network) => window.open(`${BLOCK_EXPLORER_URLS[network]}/transaction/${transactionId}`, '_blank')


  return (
    <div className={styles.container}>
      <div>
        <form onSubmit={createTweet}>
        <label for="tweet">Create a new Tweet</label>
        <textarea 
          id="tweetContents"               
          placeholder="I feel..."
          value={tweetText}
          onChange={e => setTweetText(e.target.value)}
          name="tweetContents" required></textarea>
          <small>Share your thoughts with the world.</small>
          <input type="submit" value="Tweet" />
        </form>
      </div>
      <hr />
      <h3>Your Tweets
      <button onClick={getTweets} className={styles.refresh}>
      <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="bi bi-arrow-clockwise" viewBox="0 0 16 16">
        <path fill-rule="evenodd" d="M8 3a5 5 0 1 0 4.546 2.914.5.5 0 0 1 .908-.417A6 6 0 1 1 8 2v1z"/>
        <path d="M8 4.466V.534a.25.25 0 0 1 .41-.192l2.36 1.966c.12.1.12.284 0 .384L8.41 4.658A.25.25 0 0 1 8 4.466z"/>
      </svg>
      </button>
      </h3>
        {
        tweetList.length > 0 ?
          tweetList.map((tweet) => {
            return <Tweet address={user?.addr} message={tweet.message} key={tweet.id} />
          }) : 'Your tweets will show up here!'
        } 

    </div>
  )
}
```

#### Structure

At a high level, there are 3 interacting parts to this code:
- **Imports**: We load transactions and scripts as variables.
- **States**: We set states to keep track of data
- **Actions**: We define functions like `getTweets` that run the imported code on-chain, then populate the states.

##### Imports

```js 
import GetTweetsByAccount from '../cadence/scripts/GetTweetsByAccount.cdc'
import CreateNewTweet from '../cadence/transactions/CreateNewTweet.cdc'
```

We are able to import Cadence (.cdc) files thanks to the loader defined in the webpack settings defined in `/next.config.js`.


##### States

We create several states:

```js
const [tweetList, setTweetList] = useState([])
const [tweetText, setTweetText] = useState('')
const [lastTransactionId, setLastTransactionId] = useState()
const [transactionStatus, setTransactionStatus] = useState('N/A')
```

- `tweetList` is used to keep track of the array of tweets on chain.
- `tweetText` is used to keep track of the tweet text that we are creating and will then publish
- `lastTransactionId` is used to keep track of the transaction ID so that we can link out to the chain explorers like Flowscan or Flow-View-Source. We don't really need this when using the emulator because transactions are instant and we get feedback in the console.
- `transactionStatus` is what contains the state of the transaction. Remember our section around possible transaction states above? No? Well here it is again: 

| Code    | Value     | Description                                                                                                                               |
| --- | --------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
|  0   | UNKNOWN   | The transaction status is not known.                                                                                                      |
|  1   | PENDING   | The transaction has been received by a collector but not yet finalized in a block.                                                        |
|  2   | FINALIZED | The consensus nodes have finalized the block that the transaction is included in                                                          |
|   3  | EXECUTED  | The execution nodes have produced a result for the transaction                                                                            |
|  4   | SEALED    | The verification nodes have verified the transaction (the block in which the transaction is) and the seal is included in the latest block |
|  5   | EXPIRED   | The transaction was submitted past its expiration block height.                                                                           |


[Source](https://developers.flow.com/nodes/access-api#transaction-status)

#### Actions

#### Running a script via FCL

How do we get the list of tweets from an account? It's super simple. We just use `fcl.query` and pass in the Cadence code we want to run, and the arguments. Here's the annotated source:

```jsx
const getTweets = async (account) => {
    // read addr from user object
    account = user?.addr;

    let res;

    try {
      // run fcl.query which runs a script, 
      // using the addr above as argument
      res = await fcl.query({
        cadence: GetTweetsByAccount,
        args: (arg, t) => [arg(account, t.Address)]
      })
    } catch(e) {
      res = []
    }

    // set the state as the resulting array, after sorting.
    setTweetList(res.sort((a, b) => b.id - a.id))
}
```


#### Running a transaction via FCL

Similarly to the script, running a transaction via FCL is also incredibly straightforward. Rather than `fcl.query`, we use `fcl.mutate`. We the pass in the Cadence transaction code imported previously and the arguments, like we do in the script. FCL will take care of triggering an authorization popup when publishing a tweet.

```jsx!
  const createTweet = async (event) => {
    event.preventDefault()

    if (!tweetText.length) {
      throw new Error('Please add a new greeting string.')
    }

    const transactionId = await fcl.mutate({
      cadence: CreateNewTweet,
      args: (arg, t) => [arg(tweetText, t.String)]
    })

    setLastTransactionId(transactionId)
  }
```

The authorization popup looks like this: 

![](https://i.imgur.com/5Zp2jJb.png)

Note: this layout is what the Dev Wallet shows, other wallets on Flow will show something similar, but it won't be exactly the same. The point is, there's an authorization step, and to **mutate** the chain you need to be authenticated as an account and sign the transaction. So any time you run a transaction, a popup will show up, asking the user to authorize the transaction.

## Deploying to Testnet

Last thing to try is to deploy our wonderful app to testnet. This will give us a chance to see it in action on a *real* live chain.

### Creating a testnet account

First things first, to deploy to testnet, given that it's a real live chain, we need a testnet account. To create one, all we need to do is run the following Flow CLI command:

```
flow accounts create
```

Here's what the process looks like:

#### 1. Name your Account

Name your new account `hero` and hit <kbd>Enter</kbd>. Follow the rest of the instructions on screen.

```
Enter an account name: hero
```

> 💡 You can pick any name, we are trying to keep the instructions in line with your experience. If you would decide to name your account differently, please use that name everywhere we refer to `hero` account and address.


#### 2. Set your network to Flow Testnet

Scroll down once to select Flow Testnet, then hit <kbd>Enter</kbd>
```
Use the arrow keys to navigate: ↓ ↑ → ← 
? Choose a network: 
    Local Emulator
  ▸ Flow Testnet
    Flow Mainnet
```

#### 3. Save Account Info

You'll then get presented with a confirmation step. Type <kbd>y</kbd> and hit <kbd>Enter</kbd>.
```
✔ Flow Testnet

❗ This command will perform the following:
 - Generate a new ECDSA P-256 public and private key pair.
 - Save the private key to hero.private.json and add it to .gitignore.
 - Create a new account on Flow Testnet paired with the public key.
 - Save the newly-created account to flow.json.


? Do you want to continue? [y/N] y
```

#### 4. Fund your Testnet Account

```
Please complete the following steps in a web browser:
 1. Complete the captcha challenge.
 2. Click the 'Create Account' button.
 3. Return to this window.

✔ Press <ENTER> to open in your browser...: █

```
Once you press <kbd>Enter</kbd>, your browser will be automatically directed to the [Flow Testnet Faucet](https://testnet-faucet.onflow.org/) with your account information **pre-populated**. 

The only actions that is required are: 

```

Please complete the following steps in a web browser:
 1. Complete the captcha challenge.
 2. Click the 'Create Account' button.
 3. Return to this window.

You can also navigate to the link manually: https://testnet-faucet.onflow.org/?key=<key_that_is_pre_populated>

Waiting for your account to be created, please finish all the steps in the browser...

```

![Funding your testnet account from Flow faucet](https://i.imgur.com/P6hyGlk.gif)

#### 5. You're all set!

```
🎉 New account created with address 0xebeb17c521a0d375 and name hero.

Here’s a summary of all the actions that were taken:
 - Added the new account to flow.json.
 - Saved the private key to hero.private.json.
 - Added hero.private.json to .gitignore.
```

After you finish all the steps, you will notice that 2 new files are now present in the directory:

1. `flow.json`
2. `hero.private.json` 

The Flow CLI automatically created a second file called `hero.private.json`. This file contains our private key from our newly created testnet account. This file is automatically added to the `.gitignore` so you don't accidentally leak any credentials! 

If you inspect the files, you should see the address and private key for your freshly minted account 👍!

#### flow.json 

You'll also notice the flow.json changed automatically, and now lists "hero" as an account, linking to the json.

```json 
...
"accounts": {
    "emulator-account": {
        "fromFile": "./emulator.private.json"
    },
    "hero": {
        "fromFile": "hero.private.json"
    }
},
...
```

We need to now create an account alias for the contract so FCL will know which account to substitute the imports with when on testnet on our transactions and scripts.

```diff 
  "contracts": {
    "Twitter": {
      "source": "./cadence/contracts/Twitter.cdc",
      "aliases": {
        "emulator": "0xf8d6e0586b0a20c7",
+       "testnet": "0x3c1f94baa070dd43"
      }
    }
  },
```

And we also need to add a deployment specification for testnet in the flow.json:

```json 
...
"testnet": {
    "hero": [
        "Twitter"
    ]
}
```

Our new and improved flow.json looks like this:

```diff 
{
  "contracts": {
    "Twitter": {
      "source": "./cadence/contracts/Twitter.cdc",
      "aliases": {
        "emulator": "0xf8d6e0586b0a20c7",
+       "testnet": "0x3c1f94baa070dd43"
      }
    }
  },
  "networks": {
    "emulator": "127.0.0.1:3569",
    "mainnet": "access.mainnet.nodes.onflow.org:9000",
    "testnet": "access.devnet.nodes.onflow.org:9000"
  },
  "accounts": {
    "emulator-account": {
      "fromFile": "./emulator.private.json"
    },
+   "hero": {
+     "fromFile": "hero.private.json"
+   }
  },
  "deployments": {
    "emulator": {
      "emulator-account": [
        "Twitter"
      ]
    },
+   "testnet": {
+     "hero": [
+       "Twitter"
+     ]
+   }
  }
}
```

#### Deploying

> 🚨 Check your transaction and scripts before starting this section and makes ure you change the imports from `import Twitter from 0xf8d6e0586b0a20c7` to `import Twitter from 0xTwitter`. The latter is specific to FCL and will allow seamlessly switching between the emulator and testnet/mainnet *in our app*. If you ever want to use the CLI again directly, you'll need to switch back.

Alright, let's get to deploying! 

Let's run the same command we ran in the emulator, but specifying testnet as our network. The Flow CLI will read the configuration file (flow.json) we updated and know which contract to deploy.

```
flow project deploy --network=testnet
```

You should see:

```
Deploying 1 contracts for accounts: hero

Twitter deploying...⠴
```

It might take 30 seconds or so. Then you should see:

![](https://i.imgur.com/6JqK3jy.png)

🎉 Yay!

#### Running app on testnet

> 🚨 Check your transaction and scripts before starting this section and makes ure you change the imports from `import Twitter from 0xf8d6e0586b0a20c7` to `import Twitter from 0xTwitter`. The latter is specific to FCL and will allow seamlessly switching between the emulator and testnet/mainnet *in our app*. If you ever want to use the CLI again directly, you'll need to switch back.

Now let's use the built-in `npm run dev:testnet`, which will run the app and set the `NEXT_PUBLIC_FLOW_NETWORK` to `testnet`. Our app should update automagically!

```
npm run dev:testnet
```

> ✅ Remember, the npm commands are just conveniences, you can always look through the package.json to see what the commands do. You'll realize they are very thin wrappers around standard and simple Flow CLI commands + NextJS ones.


You should see the same login page as before, and if you click "Connect Wallet" you should now see:

![](https://i.imgur.com/9dcjn0x.png)

These are actual, real, live, awesome, supercharged Flow wallets!

Let's pick Blocto. All you need is an email. Pretty sweet right? Wallets on Flow are super user-friendly. 

![](https://i.imgur.com/WpjswRy.png)

> 👉 You'll also notice the `DEV` next to Blocto. This is because we are running on testnet, so the wallet is *also* on testnet. Just put in your email (or a temp one) and click Sign in / Register.

![](https://i.imgur.com/6PH5CF2.png)

Check your email and put in the code and click Login!

We're live, on testnet!

![](https://i.imgur.com/XDAqZr0.png)

Let's try writing a tweet. Then click Tweet.

![](https://i.imgur.com/TFluecz.png)

A popup similar to the dev wallet will show up, asking you to approve the transaction. 

![](https://i.imgur.com/7gaTPqE.png)

You can also toggle the "Script" section and look at the code, similar to how we saw it in the Dev Wallet.

![](https://i.imgur.com/JH7H1Qi.png)

Once you click approve, you see nothing really happens, but if you check the console, you'll see a transaction ID:

![](https://i.imgur.com/U3whURi.png)

We can use that to visit a blockchain explorer like [Flowscan](flowscan.org) which is more user-friendly for non-devs or [Flow View Source](https://flow-view-source.com) which is more "in the weeds". 

Here's Flowscan:

![](https://i.imgur.com/Usew3fc.png)

Here's Flow View Source:

![](https://i.imgur.com/rNIzabU.png)

If everything worked correctly, you should now have a sweet, amazing, awesome tweet on Testnet!

![](https://i.imgur.com/qqxhSFD.png)


## Homework
Ok, now you have all the pieces. Now it's your turn to make this app better!

### Better transaction feedback
For example, you see we have a transaction ID, and a transaction state. Try to create a simple transaction feedback component (or in the Container directly) to give people some feedback around what's happening with their transaction. Sometimes transactions can take 10-30 seconds, so it's important to give feedback.

### Extending the contract
Currently, the only metadata we have is the actual contents of the tweet. Wouldn't it be cool if tweets had additional stuff like: date, number of likes, etc? Some additional metadata will require a pretty significant overhaul of the contract, incuding adding brand new transactions and scripts...are you up for it? :) 








