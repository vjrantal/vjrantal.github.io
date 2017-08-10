---
title: Load testing with Azure Container Instances and wrk
date: 2017-08-10T09:13:46+00:00
author: Ville Rantala
layout: post
---
# Contents
{:.no_toc}

* Will be replaced with the ToC, excluding the "Contents" header
{:toc}

# Introduction

While there exists good services for cloud-based load testing of Web sites (like in [Visual Studio Team Services](https://www.visualstudio.com/en-us/docs/test/performance-testing/getting-started/getting-started-with-performance-testing)), sometimes there is a need for more lightweight and flexible solution.

In my case, I wanted to create load towards an Azure Event Hub to benchmark an IoT solution where the Event Hub handles the data ingestion to the cloud. Additionally, I wanted to generate the load with varying payload so that I could track in the downstream services how individual messages got handled.

For this purpose, there is an HTTP benchmarking tool called [wrk](https://github.com/wg/wrk) that supports executing a LuaJIT script to modify the requests.

In this post, I'll describe how I used wrk within a docker container and how I run it in Azure in a scalable and cost-efficient way.

# Single instance locally

The sources of the image I used can be found [here](https://github.com/vjrantal/wrk-with-online-script). Basic usage locally is:

```
docker pull vjrantal/wrk-with-online-script
docker run -e TARGET_URL=https://google.com vjrantal/wrk-with-online-script
```

As the name of the image indicate, in addition to running wrk, there is support for pointing wrk to a script hosted online. An example of such run below:

```
docker run -e SCRIPT_URL=https://gist.githubusercontent.com/vjrantal/113fa910444130d2d6431cdc84e6f80e/raw/0f67559a620647d6842c579b362a139a6b338cb1/script.lua -e TARGET_URL=https://google.com vjrantal/wrk-with-online-script
```

The benefit of pulling the script online versus passing it to the container as a file is that with the online approach, there is no need to mount any storage to the containers.

# Scaling it up in Azure

There are several options to run containers in the cloud, but for this purpose, [Azure Container Instances (ACI)](https://azure.microsoft.com/en-us/services/container-instances/) works really well.

Running the container can be done using the [azure-cli](https://github.com/Azure/azure-cli) with a command like:

```
az container create -g MyResourceGroup --name wrk-with-online-script --image vjrantal/wrk-with-online-script -e TARGET_URL=https://google.com
```

Logs can be seen with:

```
az container logs -g MyResourceGroup --name wrk-with-online-script
```

And the container can be deleted with:

```
az container delete -g MyResourceGroup --name wrk-with-online-script
```

With a simple Python/Bash script, one can create multiple containers in wanted regions. The benefit over, for example, virtual machines is that the containers start in seconds and due to the per-second billing, you end up paying only for how long your load generator runs.

# End to end example

As mentioned in the introduction, I used the described approach to generate load to an Event Hub. You can find the necessary steps and scripts from [https://github.com/vjrantal/event-hub-loader](https://github.com/vjrantal/event-hub-loader) and the repository also contains a Travis CI configuration that runs the load generator using a configurable amount of containers. Every CI run generates more than million messages with a running counter as the JSON payload. This all happens within minutes and can be considered a relatively cheap approach to generate load.
