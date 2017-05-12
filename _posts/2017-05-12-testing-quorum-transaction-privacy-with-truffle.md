---
title: Testing Quorum transaction privacy with Truffle
date: 2017-05-12T11:45:00+00:00
author: Ville Rantala
layout: post
---

# Contents
{:.no_toc}

* Will be replaced with the ToC, excluding the "Contents" header
{:toc}

# Introduction

In this post, I'll describe an approach to use the [Truffle framework](http://truffleframework.com) to test the privacy features offered by [Quorum](https://github.com/jpmorganchase/quorum).

The source code can be found from [https://github.com/vjrantal/telemetry-contract](https://github.com/vjrantal/telemetry-contract) and it acts also as a sample of how to deploy a private contract onto Quorum network and how to define privacy when making transactions.

# Running the tests

## Getting the code

```
git clone https://github.com/vjrantal/telemetry-contract.git
```

## Dependencies

Install the Quorum 7nodes example network as instructed at https://github.com/jpmorganchase/quorum-examples.

Currently, the code in this repository assumes using the default configurations of the 7nodes network, but if
that is not the case, the node addresses and public keys need to be updated.

```
npm install -g truffle
```

## Running

```
truffle test --network quorum
```

# Code walkthrough

If you are not familiar with testing with Truffle, I suggest to first look at the [overall documentation](http://truffleframework.com/docs/getting_started/testing). In this walkthrough, I will focus only on the aspects that are unique when targeting Quorum.

## Network definition

In `truffle.js` at the root of the project, the target networks are defined. In my case, I added the following to have a network named quorum available:

```
    quorum: {
      host: 'localhost',
      port: 22001,
      network_id: '*'
    }
```

The target network can be passed to truffle commands with `--network quorum`.

## Contract deployment

The code to deploy can be found from file `migrations/2_deploy_contracts.js`. The privacy of the contract can be defined with a `privateFor` property that contains a list of public keys. These are the public keys of the nodes that can access the contract. In code, this looks as follows:

```
  deployer.deploy(Telemetry, {
    privateFor: ['ROAZBWtSacxXQrOe3FGAqJDyJjFePR5ce4TSIzmJ0Bc=']
  });
```

## Privacy during calls and transactions

When making transactions and calls towards the deployed contract, the privacy is again defined with the `privateFor` property. This can be seen in file `test/telemetry.js` and below an example of a transaction with the privacy flag:

```
      instance.sendTelemetry.sendTransaction(JSON.stringify(sampleTelemetry), {
        from: fromAccount,
        privateFor: [nodes.node7.publicKey]
      })
```

## Connecting to multiple nodes during tests

Because the privacy is defined at the node level, we must interact with the network via multiple nodes to be able to check what data is accessible to each node. I could not find an official API within Truffle to accomplish this, but as workaround, I am setting the provider for Web3 dynamically as follows:

```
const setProvider = function (instance, url) {
  const newProvider = new web3.providers.HttpProvider(url);
  web3.setProvider(newProvider);
  instance.contract._eth._requestManager.provider = newProvider;
};
```

The result of above is that Web3 dynamically changes the JSON RPC endpoint to which it connects to and allows me to make calls and transactions to the deployed contract via various nodes.

## Dealing with synchronization delays

When connecting via multiple nodes, it must be taken into account that when a transaction is made on one node, the result is not immediately readable on the other. It takes a while when the nodes synchronize with each other. In my tests, I have taken this into account by polling the expected result on another node until the response I expect is received. In addition, the test could include appropriate timeout to deal with situations where the expected result never occurs. In JavaScript code, this can be achieved with `setTimeout`:

```
setTimeout(function () {
  instance.getLatestTelemetry.call(fromAccount).then(resultChecker);
}, 1000);
```

# Conclusion

Using the approach described in this post, you can use the popular Truffle network also when you are targeting Quorum and leveraging the privacy-features it offers. The Quorum project has a testing sample at https://github.com/jpmorganchase/quorum-examples/blob/master/examples/7nodes/README.md#testing-privacy, but the benefit of the approach described here is that the testing is automated and doesn't require as many manual steps.
