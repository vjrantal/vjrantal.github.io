---
title: Using Azure Load Balancer to distribute UDP traffic
date: 2017-03-16T12:45:00+00:00
author: Ville Rantala
layout: post
---

# Contents
{:.no_toc}

* Will be replaced with the ToC, excluding the "Contents" header
{:toc}

# Introduction

In this blog post, I am describing steps to setup load balancing of UDP-based CoAP traffic using the [Azure Load Balancer](https://azure.microsoft.com/en-us/services/load-balancer/). The resulting architecture looks like depicted below:

![CoAP load balancing architecture]({{site.baseurl}}/images/coap-load-balancing-architecture.png)

From the short video below, you can see how I tested the end result. This was done by generating CoAP traffic from command line and expecting the response to be delivered from one of the two virtual machines that I had setup to serve the client:

<iframe width="560" height="315" src="https://www.youtube.com/embed/8qq4SrpYln0" frameborder="0" allowfullscreen></iframe>

From below you can read how to setup the virtual machines, configure the load balancer and finally details about how to test the result end to end.

# Setting up the virtual machines

I created two Ubuntu virtual machines to the same availability set. Using an availability set is [important for redundancy](https://docs.microsoft.com/en-us/azure/virtual-machines/virtual-machines-windows-create-availability-set), but more importantly for this scenario, the availability set acts as the backend pool for the load balancer (as you can read [from below](#configuring-the-load-balancer)).

After the virtual machines were provisioned, installed the software required by my CoAP server over an SSH connection. I used a Node.js project from https://github.com/mcollina/node-coap by referring to it in my `package.json` file:

```
$ cat package.json
{
  "name": "coap-demo-server",
  "version": "1.0.0",
  "main": "server.js",
  "dependencies": {
    "coap": "^0.20.0"
  }
}
```

On top of that, I implemented a simple CoAP server that has the main purpose of returning the hostname of the system so that while testing, I can identify which virtual machine served the request:

```
$ cat server.js
var coap = require('coap');
var server = coap.createServer();
var os = require('os');

var port = process.env.port || process.env.PORT || 5683;

server.on('request', function (req, res) {
  res.end('Hello from ' + os.hostname());
});

server.listen(port, function () {
  console.log('Server started on port: ' + port);
});
```

In addition to running the CoAP server, the virtual machines need respond to health probes from the load balancer (so that it can be determined if the machine in question is able to handle traffic). A UDP server can't act as health probe in itself, because UDP doesn't have acknowledgements that the load balancer could check so an additional component is needed.

A simple way to handle health probes in this test scenario is to start a TCP server on the virtual machines that just accepts TCP connections and can be used to check that the machine is up. I achieved this with the following command:

```
netcat -lk 5683
```

Above starts a TCP server in the Ubuntu virtual machine which keeps on accepting TCP connections in port `5683`.

To make the CoAP traffic and the health probe traffic flow to the virtual machines, I assigned the following network security rules:

![Network security rules]({{site.baseurl}}/images/virtual-machine-security-rules.png)

# Configuring the load balancer

After [creating the load balancer](https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-get-started-internet-portal), I configured a backend pool that looks like this:

![Backend pool]({{site.baseurl}}/images/load-balancer-backend-pool.png)

The key point is that the pool is associated to the availability set in which my virtual machines are.

The health probe that keeps connecting to the `netcat` process running in the virtual machine looks like this:

![Health probe]({{site.baseurl}}/images/load-balancer-health-probe.png)

And finally, the load balancing rule that ties everything together looks like this:

![Load balancing rule]({{site.baseurl}}/images/load-balancing-rule.png)

# Testing the result

To generate CoAP traffic, I used a [command line tool](https://github.com/eclipse/californium.tools/tree/master/cf-client) from the [Eclipse Californium project](https://eclipse.org/californium/):

```
$ java -jar cf-client-1.1.0-SNAPSHOT.jar GET coap://coap-balancer.northeurope.cloudapp.azure.com:5683
```

In above, `coap-balancer.northeurope.cloudapp.azure.com` is the domain name of my load balancer and `5683` matches with the port number I used in above configurations.

The CoAP response includes a line like below and indicates which virtual machine ended up serving the request:

```
---------------------------------------------------------------
Hello from coap-ubuntu-2
===============================================================
```

As you can see from the video in the [introduction section](#introduction), the load balancer distributes the traffic between the machines in the availability set and the machine handling the response varies.

In my case, I set everything to route IPv4 traffic, but the relevant services [also support IPv6](https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-ipv6-internet-ps) so you should be able to achieve the same for IPv6 traffic.
