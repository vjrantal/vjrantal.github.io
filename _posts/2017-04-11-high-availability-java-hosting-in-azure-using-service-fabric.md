---
title: High-availability Java hosting in Azure using Service Fabric
date: 2017-04-11T11:45:00+00:00
author: Ville Rantala
layout: post
---

# Contents
{:.no_toc}

* Will be replaced with the ToC, excluding the "Contents" header
{:toc}

# Introduction

While arguably the most carefree option to host Java apps in Azure is [using App Service](https://docs.microsoft.com/en-us/azure/app-service-web/app-service-web-get-started-java), there are sometimes situations where using that approach is not possible. An example is if you require custom ports to be opened or if you need to deal with UDP traffic.

In this post, I'll describe an approach to host a Java app as [Service Fabric guest executable](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-deploy-existing-app). This allows TCP traffic on any port and UDP. As the sample Java app, I am using [Leshan](http://www.eclipse.org/leshan/), which contains a Web server (TCP on port 8080) and CoAP endpoints (UDP on ports 5683 and 5684). The project is written in Java and uses Maven as build system. CoAP is a light-weight protocol over UDP and was designed, for example, for certain IoT scenarios.

# Setup, build and deploy

The instructions can be found from the readme at [https://github.com/vjrantal/leshan-on-service-fabric](https://github.com/vjrantal/leshan-on-service-fabric).

# Benefits of Service Fabric

For this one, I'll just point to the Azure documentation that explains what Service Fabric provides you:

[https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-deploy-existing-app](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-deploy-existing-app#benefits-of-running-a-guest-executable-in-service-fabric)

# Java runtime considerations

Because the default machines in the Service Fabric cluster doesn't have Java installed, there needs to be a mechanism to ensure the right version of Java is found when the app is run. Alternatives include:

1. Install Java on the machines after they are created
2. Include Java in the app package

The challenge with the 1. approach is that the installation is required every time new machines are provisioned in the cluster, for example, when scaling out the cluster. Manual approach isn't good because it would ruin programmatic auto-scaling. So implementing 1. in an automated fashion would require some amount of scripting.

For this demonstration, I chose 2. by using [https://github.com/libgdx/packr](https://github.com/libgdx/packr). There, the idea is that the right version of the Java runtime gets included in the app package that Service Fabric deploys to the machines in the cluster. This increases the app package size, but this should not be a big issue, because copying the app package around the machines in the cluster is relatively fast even if the size is 100+ MB. The benefit is that the app package has a self-contained executable that can be run on vanilla machines.

Another alternative would be to use containers. In that approach, you could create a Docker image that contains the right Java runtime and your app and deploy that onto the cluster.

# Workaround to allow UDP routing

Currently, the Service Fabric configuration [doesn't allow defining](https://github.com/Azure/service-fabric-issues/issues/208#issuecomment-289795833) an endpoint as UDP endpoint. For this reason, there needs to be a one-time manual configuration change after deployment of the app to allow UDP load balancing:

![Allow UDP load balancing]({{site.baseurl}}/images/allow-udp-load-balancing.png)

From the image above (especially the two red arrows), you can see how in Azure portal you can change the default TCP rule to be an UDP rule for the ports where you want to handle UDP traffic. The lower red arrow indicates that you should also switch the health probe to be something else than the UDP port, because UDP endpoint can't be used to acknowledge if server is up. In this demo, I used the TCP endpoint in port 8080 for the health probe, which is a reasonable solution and that way, the load balancer can tell that the Java app is up and running.

# Disclaimers

* This sample used an unsecure Service Fabric cluster, but if your intention is to use the cluster for real, you want to secure it so that others can't access the management interfaces.
* The sample app used was the [stand-alone Leshan server demo](https://github.com/eclipse/leshan/tree/master/leshan-server-demo), which results into independent servers in individual machines in the cluster. A much more usable system would be to deploy the [Leshan server cluster](https://github.com/eclipse/leshan/tree/master/leshan-server-cluster) together with [Azure Redis Cache](https://azure.microsoft.com/en-us/services/cache/) so that servers in the cluster would be interconnected via Redis.
* This approach is not a good one if you are looking for the very lowest cost option to host a simple Java app. A proper Service Fabric cluster is 5+ virtual machines and for that reason, you can't get into a pricing like 10 euros per month. The exact cost depends on the amount of machines in the cluster and the [size of each](https://azure.microsoft.com/en-us/pricing/details/virtual-machines/windows/).
