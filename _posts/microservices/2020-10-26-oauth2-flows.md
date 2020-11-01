---
title: OAuth2.0 Flows and OAuth 2.1
categories:
- Microservices
layout: post
aside: true
excerpt: 
---
In my [last post](/jwt-intro) we had looked at how JWTs can be used in OAuth 2.0. Here we will look at the flows in OAuth 2.0 and the the proposed changes in Oauth 2.1 draft.
<!--more-->

> Similar to the previous post, this post is also targeted at complete beginners. I wanted to just introduce the concepts to people who are not familiar with Authentication / Authorization flows. I have glossed over certain topics and linked to detailed resources to avoid duplication. I recommend reading [OAuth 2.0 Simplified by Aaron Parecki](https://aaronparecki.com/oauth-2-simplified/) once you have read this post for more details. 

### The OAuth2 Flows
When OAuth2 was introduced, it proposed 4 recommended flows as part of the specification.
* The Authorization Code Flow
* The Implicit Flow
* Resource Owner Password Credentials Flow, or simply, the Password Flow
* Client Credentials Flow

Each flow is a small variation of the Abstract Protocol Flow (as proposed in [RFC 6749](https://tools.ietf.org/html/rfc6749#section-1.2)) shown below:

```
     +--------+                               +---------------+
     |        |--(A)- Authorization Request ->|   Resource    |
     |        |                               |     Owner     |
     |        |<-(B)-- Authorization Grant ---|               |
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(C)-- Authorization Grant -->| Authorization |
     | Client |                               |     Server    |
     |        |<-(D)----- Access Token -------|               |
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(E)----- Access Token ------>|    Resource   |
     |        |                               |     Server    |
     |        |<-(F)--- Protected Resource ---|               |
     +--------+                               +---------------+
```

#### Login with _X_
Many websites today, support logging in with third party services like Google, Facebook, Github etc. A very common way in which this is enabled is via the [OAuth2 Authorization Code Flow](https://tools.ietf.org/html/rfc6749#section-4.1). 
The RFC provides even more details. At a high level, this is what happens:
* The client clicks on Login with _X_ at the Resource Owner Website
* The client is redirected to the corresponding Authorization server along with certain metadata such as:
  - where to redirect the user on success and failure
  - requested scope
* The client is authenticated by the server and redirected correspondingly along with an Authorization Code
* The client then requests for the access token using this Authorization code
* An access token is shared on successful authentication


### OAuth2.1
OAuth 2.0 has been around for about a decade now. Over the years, the original proposal has been modified and extended based on live implementations and security concerns. More so because, the technological landscape has shifted dramatically. The new world is primarily mobile-first. Single Page Applications have become very popular. All this pretty much meant that OAuth2 required an upgrade. 

But also because of these modifications, a lot of factors had to be taken into consideration before implementing OAuth2.0 securely. There were a lot of documents to read and practices to follow. Some practices could have been deprecated in future RFCs. OAuth2 was intended to be a simpler alternative to the original OAuth. But ironically, these modifications sort of took it the other way. It isn't too complicated, but it can get confusing.

[OAuth 2.1](https://oauth.net/2.1/) was proposed late last year to overcome some of these challenges. OAuth2.1 is more restrictive in it's approach. OAuth2 was pretty flexible. That is what made it successful to begin with but later on caused issues. OAuth2 is still in it's draft stage but I believe you can adopt it, if required. OAuth 2.1 basically collates some of the OAuth 2.0 best practices over the years.

#### The Major Changes
The most important change is that the password and implicit flows must not be used. This is accordance with the Security Best Current Practices (BCP) published earlier this year as well. You could visit [this page](https://oauth.net/2/oauth-best-practice/), for the latest information.

The other major change is the mandated use of Proof Key for Code Exchange (PKCE) flow with all OAuth2 flows. This is an interesting mandate. PKCE was originally designed for mobile apps, but due to it's ability to prevent Authorization Code injection attacks, it is suitable for all flows. The other changes all involve adopting the latest BCP updates. 

##### PKCE Flow
The PKCE flow is very similar to the Authorization Code flow. The only difference is that, while initially requesting the Authorization code, the client generates two secret strings: a `code_verifier` and a `code_challenge` from the verifier. 

* Initially, the `code_challenge` is used to get an authorization code from the server.
* Then, when the client requests for an access token it also sends the  `code_verifier`. 
* The `code_challenge` will contain the details on how to verify the `code_verifier`.
* On successful verification the access token is returned.

Also, you can check out the [official page](https://oauth.net/2/pkce/) for more resources on this topic.

### Which flow should I use?
As I said, adopting OAuth2.1 basically means adopting the latest security best practices. As to which flow should be used, it totally depends. A useful recommendation can be found [here.](https://auth0.com/docs/authorization/which-oauth-2-0-flow-should-i-use) The community is considering a proposal for an [OAuth 3 standard](https://oauth.net/3/) as well. OAuth 2 is due for an upgrade. Currently we have many more use cases for Authentication, than what could have been covered all those years ago! But until then OAuth 2.1 seems to be a good recommendation for OAuth implementations.