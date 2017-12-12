---
title: Measuring Event Hubs throughput with Application Insights
date: 2017-12-11T07:13:26+00:00
author: Ville Rantala
layout: post
---
# Contents
{:.no_toc}

* Will be replaced with the ToC, excluding the "Contents" header
{:toc}

# Introduction

This post demonstrates an approach to measure data ingestion throughput via Azure Event Hubs using Application Insights. It also includes some results while comparing the implementation of the so-called [Event Processor Host](https://blogs.msdn.microsoft.com/servicebus/2015/01/16/event-processor-host-best-practices-part-1/) (EPH) in three different programming languages (.NET Core, Python and Java) and includes [link to the source code of the experiments](https://github.com/vjrantal/event-hubs-benchmarks).

This work and the measurements were done to find optimal virtual machine sizes and the right amount of EPH instances to achieve certain throughput levels.

For similar benchmarking when using Azure Functions, I recommend checking the [great post by Paul Batum](https://blogs.msdn.microsoft.com/appserviceteam/2017/09/19/processing-100000-events-per-second-on-azure-functions/).

# Test setup

The test was performed by first loading an Event Hub with messages using a [tool developed for that purpose](https://github.com/vjrantal/event-hub-loader). The payload was a static JSON with size 12B in one test and 2KB in others.

The EPH was implemented to checkpoint every 10 seconds, which was considered to be a reasonable tradeoff between performance and potentially having to deal with some amount of duplicate messages.

After the Event Hub was loaded with messages, the EPH implementations were deployed on a Kubernetes cluster in the same region as the Event Hub. The deployments were done one after another so that only one language implementation is processing at any given time.

While running, the implementations sent telemetry about the current throughput to Application Insights - see [here](https://github.com/vjrantal/event-hubs-benchmarks/blob/6dae4385c99033e442e781b12af1b19fb8b67b7e/dotnet/SimpleEventProcessor.cs#L73) for an example how it was done in the .NET Core implementation.

# Results

Once the data is in Application Insights, you can use custom queries to get insights into what was going on during the test.

For example, the following query:

```
customMetrics
| where timestamp > datetime("2017-11-20T15:19:43")
| summarize sum_by_name=sum(value) by name, bin(timestamp, 10s)
| summarize avg(sum_by_name) by name, bin(timestamp, 30s)
| render timechart
```

Would result into a graph like this:

![Insights]({{site.baseurl}}/images/dotnet-100-partitions.png)

In above, I was using 2KB payload in an Event Hub with 100 partitions and reading the data with 20 EPH instances. Those instances were scheduled by Kubernetes according to the [resource definition](https://github.com/vjrantal/event-hubs-benchmarks/blob/6dae4385c99033e442e781b12af1b19fb8b67b7e/dotnet/deployment.yaml#L17), which effectively meant that two instances fit a single `Standard_D2_v2` virtual machine (since that has 2 vCPUs).

In below table, I collected a summary of the main tests that I run. The header is in format `partitions / EPH instances / payload size`. In all tests, the number of Event Hubs Throughput Units was 100. The result is a peak throughput in MB/s.


| Language | 4 / 4 / 2KB | 100 / 20 / 2KB | 2 / 2 / 12B |
| ----------- | ----------- | ----------- | ----------- |
| Java | 29.7 MB/s | 269.0 MB/s | 0.44 MB/s |
| .NET Core | 12.2 MB/s | 246.6 MB/s | 0.71 MB/s |
| Python | 12.7 MB/s | 39.5 MB/s | 0.06 MB/s |


It should be noted that above does not tell any absolute truth about the speed of those different languages, but should be considered a snapshot of how the current versions of the implementation behaved with the given parameters. For example, changing the payload size and the size of the virtual machines might give different results.

There is also built-in metrics in Event Hubs that can be queried with the [azure-cli](https://github.com/Azure/azure-cli) command `az monitor metrics list`. During testing, I used a command like:

```
az monitor metrics list --resource "<full-resource-id>" --metric-names OutgoingBytes --time-grain PT1M
```

To get the in-built metrics partly to compare them to the results I saw via Application Insights to get confidence that there were no mistakes in the way I measured.