---
title: Facebook Account Linking with Microsoft Bot Framework
date: 2016-11-15T09:45:00+00:00
author: Ville Rantala
layout: post
---

# Contents
{:.no_toc}

* Will be replaced with the ToC, excluding the "Contents" header
{:toc}

# Introduction

In chatbot scenarios, it is often required to perform some sort of user authentication and authorization. For example, the bot might be doing travel reservations on behalf of the user so it needs the proper credentials to do so.

The [Microsoft Bot Framework](https://dev.botframework.com/) has some helpers to implement user authentication, for example, the [SigninCard](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.signincard.html). There is also example [Node.js code](https://github.com/mattdot/botauth) and [C# code](https://github.com/MicrosoftDX/AuthBot) for authenticating the user. All the approaches work and can be leveraged, but are based on "external authentication flow" meaning performing the authentication outside of the chat context (for example, in a Web browser).

The Facebook Messenger offers a feature where you can perform the user authentication within the Facebook Messenger experience and this is called [Account Linking](https://developers.facebook.com/docs/messenger-platform/account-linking).

In this post, I am showing a Node.js [demo bot implementation found from GitHub](https://github.com/vjrantal/account-linking-demo-bot) that implements the Facebook Messenger Account Linking on top of the Microsoft Bot Framework.

# User Experience Demo

See below a looping GIF that shows what the user experience is on an iPhone within the Facebook Messenger app.

![account-linking-iphone](https://cloud.githubusercontent.com/assets/207474/20264373/dde1a2d8-aa73-11e6-8dfa-b2f4b4352105.gif)

# Technical Details

On a high level, the account linking flow could be depicted as below:

![Account Linking Sequence]({{site.baseurl}}/images/account-linking-sequence.png)

In the sample code, the numbered sequences are handled as follows:

1. On [this line](https://github.com/vjrantal/account-linking-demo-bot/blob/c2515c0c930aadcb5986b9e7ee4cc0eab4638130/index.js#L76) we listen when user's message is `link account` and consider that as the trigger for the flow
2. Since the button is Facebook-specific, we need to construct a Facebook-specific message to send it [like this](https://github.com/vjrantal/account-linking-demo-bot/blob/c2515c0c930aadcb5986b9e7ee4cc0eab4638130/index.js#L78-L96)
3. User clicks the button within the chat
4. The Website is served as static assets with [these code lines](https://github.com/vjrantal/account-linking-demo-bot/blob/c2515c0c930aadcb5986b9e7ee4cc0eab4638130/index.js#L39-L41) so the index page is [this one](https://github.com/vjrantal/account-linking-demo-bot/blob/c2515c0c930aadcb5986b9e7ee4cc0eab4638130/static/index.html)
5. User does authentication (in this sample, just types in a username)
6. When user clicks the continue button in the index page, we set the page source to a [URL handled in the bot code](https://github.com/vjrantal/account-linking-demo-bot/blob/c2515c0c930aadcb5986b9e7ee4cc0eab4638130/index.js#L21)
7. We pick up the username in the bot code and [redirect to the URL given by Facebook](https://github.com/vjrantal/account-linking-demo-bot/blob/c2515c0c930aadcb5986b9e7ee4cc0eab4638130/index.js#L32)
8. The Account Linking Webhook event is handled [here](https://github.com/vjrantal/account-linking-demo-bot/blob/c2515c0c930aadcb5986b9e7ee4cc0eab4638130/index.js#L47-L52)
9. We [store the username and inform the user](https://github.com/vjrantal/account-linking-demo-bot/blob/c2515c0c930aadcb5986b9e7ee4cc0eab4638130/index.js#L57-L58)
