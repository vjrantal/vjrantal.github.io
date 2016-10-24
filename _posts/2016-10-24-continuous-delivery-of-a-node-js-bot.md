---
title: Continuous Delivery of a Node.js bot
date: 2016-10-24T11:20:00+00:00
author: Ville Rantala
layout: post
---

# Contents
{:.no_toc}

* Will be replaced with the ToC, excluding the "Contents" header
{:toc}

# Introduction

In this blog post, I'll demonstrate a Continuous Integration (CI) and Continuous Delivery (CD) pipeline for a bot built on Node.js using the [Microsoft Bot Framework](https://dev.botframework.com/). I'll be using a simple bot as the sample case and will show an approach to automate integration tests in [Travis CI](https://travis-ci.org/) and deploy continuously to [Azure Functions](https://azure.microsoft.com/en-us/services/functions/).

The sample bot project I'll talk about can be [found from GitHub](https://github.com/vjrantal/bot-sample).

# Continuous Integration

There are these days many great options for a free (or inexpensive) CI systems that one can leverate in open source (and private) projects. I chose Travis CI to be able to test on a MacOS build worker.

The role of the CI system is to take the latest code from the GitHub project, run the automated tests and deliver the results onto the bot hosting environment.

## Automated testing of the bot

The bot is a Node.js app, so there are plenty of options to choose from to write unit and functional tests. Since this type of "lower level testing" of Node.js projects is well-documented elsewhere, in this post, I'll focus on a more bot-specific end-to-end testing scenario.

The Microsoft Bot Framework comes with [an emulator](https://docs.botframework.com/en-us/tools/bot-framework-emulator/). By using the emulator, you can test your bot and have a conversation without going through one of the actual channels (like Skype or Slack). The emulator can run on MacOS using mono and can be installed on the Travis CI worker via brew (see the [before install step](https://github.com/vjrantal/bot-sample/blob/6d973d481fac35e6137dd70e9546d2b1ae0e6911/.travis.yml#L6) for details).

The testing approach is to have a conversation with the bot and verify that it behaves in the expected way. Here is a clip from the test code that gives an idea how a test is written:

{% highlight javascript %}
    .then(function (output) {
      // The bot should greet with the name we set earlier
      t.equal(output, 'Hi Tester, say something to me and I will echo it back.');
      // Say something to the bot
      return emulatorProgram.input('Something');
    })
    .then(function (output) {
      // The bot should not send the greeting anymore, but instead,
      // the message sent should be echoed back
      t.equal(output, 'Tester, I heard: Something');
{% endhighlight %}

To see above code in the full context, have a look at [this file](https://github.com/vjrantal/bot-sample/blob/6d973d481fac35e6137dd70e9546d2b1ae0e6911/test.js).

# Continuous Delivery

For hosting the Node.js bot app, I chose Azure Functions so that I have to worry as little as possible about the hosting environemnt and can focus only on my app code.

Azure Functions can be set to automatically listen for commits in a GitHub repository, but in some cases, it is useful to have a more explicit control on what triggers new deployments. For example, you might want to deploy only after certain testing is complete, like in this case, I wanted to deploy after the CI run in master branch is successful.

The starting point for these instructions is that you already have a Functions app and have set it to deploy from a local git repository (for documentation about how to do that, see [here](https://azure.microsoft.com/en-us/documentation/articles/functions-continuous-deployment/)).

For the CI to be able to push to the git repository, we need to pass the credentials. To expose the least priviledged credentials, it is a good approach to use the credentials specific to this Azure resource. Those can be received from the publish profile, which is a `.PublishSettings` XML file that can be downloaded from the Azure portal:


![Get Publish Profile]({{site.baseurl}}/images/get-publish-profile.png)


The XML file has the relevant values in attributes named `userName` and `userPWD`. The git clone url is visible in Azure portal in the Overview section (same section from which the screenshot above is taken from). When using the encrypted environment variables in Travis, those credentials can used in a secure way from the deployment scripts (see example [here](https://github.com/vjrantal/bot-sample/blob/master/function-deploy.sh#L11)).

Due to Azure Functions expecting the individual function apps to be in subfolders of the git repository, I am [using git submodules](https://github.com/vjrantal/bot-sample/blob/master/function-deploy.sh#L11-L17) to create the right structure.
