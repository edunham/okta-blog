---
layout: blog_post
title: "Elevate Access Token Security by Demonstrating Proof-of-Possession"
author: alisa-duncan
by: advocate
communities: [security]
description: "Protect your OAuth 2.0 access token with sender constraints. Learn about possession proof tokens using DPoP."
tags: [oauth2, security, okta, authorization, dpop, spa]
image: blog/dpop-oauth/social.jpg
type: awareness
---

We use access tokens to request data and perform actions within our software systems. The client application sends a bearer token to the resource server. The resource server checks the validity of the access token before acting upon the HTTP request. What happens if the requesting party is malicious, steals your token, and makes a fraudulent API call? Would the resource server honor the HTTP request? If you use a bearer token, the answer is "yes." 

My teammate wrote that an access token is like a hotel room keycard. If you have a valid keycard, anyone can use it to access the room. If you have a valid access token, anyone can use it to access a resource server.

{% excerpt /blog/2019/06/05/seven-ways-an-oauth-access-token-is-like-a-hotel-key-card %}

Bearer tokens (and static API keys) mean [whoever presents the valid token to the resource server has access](/blog/2023/09/25/oauth-api-tokens), which makes the token powerful and vulnerable. We can look at high-profile [token thefts](https://www.canonic.security/blog/top-6-most-notorious-oauth-attacks) to see how prevalent and disastrous token theft is, so we want to ensure our applications aren't vulnerable to similar attacks.

To protect tokens, we incorporate secure coding techniques into our apps, configure a quick expiration time on the token, and ensure only requests sent to allowed origins include the access token. Still, token attacks pose a risk to highly sensitive resources. What more can we do to secure requests?

This post describes a new OAuth 2.0 spec supported by Okta that makes access tokens less prone to misuse and helps mitigate security risks. If you want to refresh your OAuth knowledge, check out [What the heck is OAuth](/blog/2017/06/21/what-the-heck-is-oauth).

**Table of Contents**{: .hide }
* Table of Contents
{:toc}

## Bind OAuth 2.0 access tokens to client applications

If we go back to the hotel keycard analogy, we want a hotel keycard that only you can use and that links you as the rightful user of the hotel keycard. 

In the OAuth world, ideally, we want to link the authorization server, the client, and the access token and limit token use to the client. In OAuth terminology, the sender and client application are the same entity. By linking these entities, external parties can't misuse the access token.

OAuth 2.0 defines a few methods to bind access tokens. 

<table>
  <tr>
    <td style="font-size: 3rem;">🤐</td>
    <td markdown="span">
      [**Client secret**](https://oauth.net/2/client-authentication/)<br/>
      Confidential clients are applications running in a protected environment where user authentication and token storage occur within backend servers, such as traditional server-rendered web applications. Confidential clients can use a secret value known to the requestor (the client application requesting the tokens) and the authorization server as part of HTTP requests. The client secret is a long-lived value generated by the authorization server. However, malicious parties who steal the secret can use it.
    </td>
  </tr>
  <tr>
    <td style="font-size: 3rem;">🌐</td>
    <td markdown="span">
      [**Mutual TLS Client Authentication and Certificate-Bound Access Tokens (mTLS)**](https://oauth.net/2/client-authentication/)<br/>
      Mutual authentication means parties at the ends of the network connection identify themselves using a combination of [asymmetric encryption](https://www.okta.com/identity-101/asymmetric-encryption/) and TLS certificate as part of the HTTP request. mTLS is a highly secure method for confidential clients but can be complex to implement and maintain.
    </td>
  </tr>
  <tr>
    <td style="font-size: 3rem;">🔒</td>
    <td markdown="span">
      [**Private key JSON Web Token (JWT)**](https://oauth.net/private-key-jwt/)<br/>
      Machine-to-machine HTTP requests don't have user context. The requesting service often uses a combination of an ID and secret using the Basic authorization scheme when making HTTP calls, but doing so isn't secure. Private key JWTs offer a more secure approach. The requesting service uses asymmetric encryption to sign any JWTs it creates.
    </td>
  </tr>
</table>

These methods apply only to confidential clients that can maintain secrets, not to public clients.

Public clients are apps that run authentication code within the user's hardware, such as in Single-Page Applications (SPA) and mobile clients. Software applications use public client architecture but contain avenues for token security exploits without careful protection. Is there an alternative that works for confidential and public clients without incurring costly implementation and maintenance?

## Demonstrate proof of possession (DPoP) using JWTs

There's now a solution for all client types calling sensitive resources! The IETF published a new extension to OAuth 2.0: [Demonstrating Proof of Possession (DPoP)](https://www.rfc-editor.org/rfc/rfc9449), targeted primarily for public client use. You may have heard of this idea before, as the concept has been around for a while. With a published spec, it's now official, standardized, and supported!

The client and authorization server work together to generate tokens with proof of possession.
  1. The client creates non-repudiable proof of ownership using [asymmetric encryption](/blog/2019/09/12/why-public-key-cryptography-matters) 
  1. The authorization server uses this proof when generating the token 

How is this different from earlier methods that bind the caller to the access token? The big difference is this method happens at runtime across any client type. Confidential clients have cryptographic libraries supporting public/private key encryption, but a gap exists for public clients. Thanks to enhanced browser API capabilities such as the [Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API) and [SubtleCrypto](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto), modern browser-based JavaScript apps can also use DPoP. 

🚨 **You must protect the client from Cross-Site Scripting (XSS) and Remote File Inclusion (RFI) attacks to prevent exfiltration or unauthorized use of the keyset.** 🚨 

Store the keys in a storage format that someone can't export and guard the app against attacks where an attacker's code can run in the user's context. Use up-to-date secure SPA frameworks, [employ defensive coding practices](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html), and add appropriate [Content Security Policies (CSP)](https://web.dev/articles/csp) to protect the client. Apply [secure header best practices](/blog/2021/10/18/security-headers-best-practices) and consider using the [Trusted Types API](https://developer.mozilla.org/en-US/docs/Web/API/Trusted_Types_API) if you can limit end-user browser usage to browsers that support it.

> ⚠️ **Note**
> 
> We will investigate DPoP proofs and inspect how the client constructs them. However, despite this knowledge, you should always use Okta SDKs or a vetted, well-maintained library with built-in DPoP support when making requests using DPoP.

## Incorporating DPoP into OAuth 2.0 token requests

When using DPoP, the client creates a "proof" using asymmetric encryption. The proof is a JWT, which includes the URI, the HTTP method of the request, and the public key. The client application requests tokens from the authorization server and includes the proof as part of the request. The authorization server binds a public key hash and the HTTP request information from the proof within the access token it returns to the client. This means the access token is only valid for the specific HTTP request.

A sequence diagram for the OAuth 2.0 Authorization Code flow with DPoP looks like this:

{% img blog/dpop-oauth/token-request.svg alt:"Sequence diagram where client redirects to authorization server for user challenge. The authorization server redirects back to the client with the authorization code. The client generates a public/private key and creates the DPoP proof. The client sends the proof in the token request. The authorization server returns an access token bound to the proof." width:"800" %}

{% comment %}
Tweak the diagram on https://mermaid.live/ with the following content
sequenceDiagram
    participant C as Client
    participant A as Authorization Server
    C-->>+A: Redirect to sign-in page for user challenge
    A-->>-C: Redirect to client with authorization code
    C->C: Generate public/private key and DPoP proof 
    C->>+A: Request token sending DPoP proof and code 
    A->>-C: Token bound to proof
{% endcomment %}

The proof contains metadata proving the sender and ways to limit unauthorized use by limiting the HTTP request, the validity window, and reuse. If you inspect a decoded DPoP proof JWT, you'll see the header contains information proving the sender:
 * The `typ` claim set to `dpop+jwt`
 * The public/private key encryption algorithm
 * The public key in [JSON Web Key (JWK)](https://www.rfc-editor.org/rfc/rfc7517) format

Inspecting the decoded proof's payload shows claims that limit unauthorized use, such as:
 * HTTP request info including the URI and HTTP method (such as `https://{yourOktaDomain}/oauth2/v1/token` and `POST`)
 * Issue time to limit the validity window for the proof
 * An identifier that's unique within the validity window to mitigate replay attacks

Let's inspect the `/token` request a little further. When making the request, the client adds the proof in the header. The rest of the request, including the grant type and the code itself, remains the same for the Authorization Code flow.

```http
POST https://{yourOktaDomain}/oauth2/v1/token HTTP/1.1
DPoP: eyJ0eXAiOiJkcG9w.....H8-u9gaK2-oIj8ipg
Accept: application/json
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
code=XGa_U6toXP0Rvc.....SnHO6bxX0ikK1ss-nA
```

The authorization server decodes the proof and incorporates properties from the JWT into the access token. The authorization server responds to the `/token` request with the token and explicitly sets the response header to state the token type as `DPoP`.

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
"access_token":"eyJhbG1NiIsPOk.....6yJV_adQssw5c",
"token_type":"DPoP",
"expires_in":3600,
"refresh_token":"5PybPBQRBKy2cwbPtko0aqiX"
}
```

You now have a DPoP type access token with a possession proof. What changes when requesting resources?

## Use DPoP-bound access tokens in HTTP requests

DPoP tokens are no longer bearer tokens; the token is now "sender-constrained." The sender, the client application calling the resource server, must have both the access token and a valid proof, which requires the private key held by the client. This means malicious sorts need both pieces of information to impersonate calls into the server. The spec builds in constraints even if a malicious sort steals the token and the proof. The proof limits the call to a unique request for the URI and method within a validity window. Plus, your application system still has the defensive web security measures applicable to all web apps, preventing the leaking of sensitive data such as tokens and keysets. 

The client generates a new proof for each HTTP request and adds a new property, a hash of the access token. The hash further binds the proof to the access token itself, adding another layer of sender constraint. The proof's payload now includes:
 * HTTP request info including the URI and HTTP method (such as `https://{yourResourceServer}/resource` and `GET`)
 * Issue time to limit the validity window for the proof
 * An identifier that's unique within the validity window to mitigate replay attacks
 * Hash of the access token

Clients request resources by sending the access token in the `Authorization` header, along with proof demonstrating they're the legitimate holders of the access token to resource servers using a new scheme, `DPoP`. HTTP requests to the resource server change to

```http
GET https://{yourResourceServer}/resource HTTP/1.1
Accept: application/json
Authorization: DPop eyJhbG1NiIsPOk.....6yJV_adQssw5c
DPoP: eyJhbGciOiJIUzI1.....-DZQ1NI8V-OG4g
```

The resource server verifies the validity of the access token and the proof before responding with the requested resource. 

## Extend the DPoP flow with an enhanced security handshake

DPoP optionally defines an enhanced handshake mechanism for calls requiring extra security measures. The client _could_ sneakily create proofs for future use by setting the issued time in advance, but the authorization and resource servers can wield their weapon, the nonce. The nonce is an opaque value the server creates to limit the request's lifetime. If the client makes a high-security request, the authorization or resource server may issue a nonce that the client incorporates within the proof. Doing so binds the specific request and time of the request to the server.

An example of a highly secure request is when making the initial token request. Okta follows this pattern. Different industries may apply guidance and rules for the types of resource server requests requiring a nonce. Since the enhancement requires an extra HTTP request, use it minimally.

When the authorization server's `/token` request requires a nonce, the server rejects the request and returns an error. The response includes a new header type, `DPoP-Nonce`, with the nonce value, and a new standard error message, `use_dpop_nonce`. The flow for requesting tokens now looks like this:

{% img blog/dpop-oauth/token-request-with-nonce.svg alt:"Sequence diagram where client redirects to authorization server for user challenge. The authorization server redirects back to the client with the authorization code. The client generates a public/private key and creates the proof. The client sends the proof in the token request. The authorization server rejects the request and returns a nonce. The client regenerates the proof with nonce incorporated and re-requests the tokens. The authorization server returns an access token bound to the proof." width:"800" %}

{% comment %}
Tweak the diagram on https://mermaid.live/ with the following content
sequenceDiagram
    participant C as Client
    participant A as Authorization Server
    C-->>+A: Redirect to sign-in page for user challenge
    A-->>-C: Redirect to client with authorization code
    C->C: Generate public/private key and DPoP proof 
    C->>+A: Request token sending DPoP proof and code 
    A->>-C: Error! Here's a nonce for you
    C->C: Generate DPoP proof with nonce
    C->>+A: Request token sending DPoP proof with nonce and code 
    A->>-C: Token bound to proof
{% endcomment %}

Let's look at the HTTP response from the authorization and resource servers requiring a nonce. The authorization server responds to the initial token request with a `400 Bad Request` and the needed nonce and error information.

```http
HTTP/1.1 400 Bad Request
DPoP-Nonce: server-generated-nonce-value

{
  "error": "use_dpop_nonce",
  "error_description": "Authorization server requires nonce in DPoP proof"
}
```

When the resource server requires a nonce, the response changes. The resource server returns a `401 Unauthorized` with the `DPoP-Nonce` header and a `WWW-Authenticate` header containing the `use_dpop_nonce` error message.

```http
HTTP/1.1 401 Unauthorized
DPoP-Nonce: server-generated-nonce-value
WWW-Authenticate: error="use_dpop_nonce", error_description="Resource server requires nonce in DPoP proof"
```

We want that resource, so it's time for a new proof! The client reacts to the error and generates a new proof with the following info in the payload:
 * HTTP request info including the URI and HTTP method (such as `https://{yourResourceServer}/resource` and `GET`)
 * Issue time to limit the validity window for the proof
 * An identifier that's unique within the validity window to mitigate replay attacks
 * The server-provided nonce value
 * Hash of the access token

With this new proof, the client can remake the request.

## Validate DPoP requests in the resource server

Okta's API resources support DPoP-enabled requests. If you want to add DPoP support to your own resource server, you must validate the request. You'll decode the proof to verify the properties in the header and payload sections of the JWT. You'll also need to verify properties within the access token. OAuth 2.0 access tokens can be opaque, so use your authorization server's `/introspect` endpoint to get token properties. Okta's API security guide, [Configure OAuth 2.0 Demonstrating Proof-of-Possession](https://developer.okta.com/docs/guides/dpop/nonoktaresourceserver/main/#make-a-request-to-a-non-okta-resource) has a step-by-step guide on validating DPoP tokens, but you should use a well-maintained and vetted OAuth 2.0 library to do this for you instead. Finally, enforce any application-defined access control measures before returning a response.

## Learn more about OAuth 2.0, Demonstrating Proof-of-Possession, and secure token practices

I hope this intro to sender-constrained tokens is helpful and inspires you to use DPoP to elevate token security! Watch for more content about DPoP, including hands-on experimentation and code projects. If you found this post interesting, you may also like these resources:

* [Secure OAuth 2.0 Access Tokens with Proofs of Possession](/blog/2024/09/10/angular-dpop-jwt)
* [Why You Should Migrate to OAuth 2.0 From Static API Tokens](/blog/2023/09/25/oauth-api-tokens)
* [How to Secure the SaaS Apps of the Future](https://sec.okta.com/appsofthefuture)
* [Step-up Authentication in Modern Application](/blog/2023/03/08/step-up-auth)
* [OAuth 2.0 Security Enhancements](https://auth0.com/blog/oauth2-security-enhancements/)
* [Add Step-up Authentication Using Angular and NestJS](/blog/2024/03/12/stepup-authentication)

Remember to follow us on [Twitter](https://twitter.com/oktadev) and subscribe to our [YouTube channel](https://www.youtube.com/c/OktaDev/) for more exciting content. We also want to hear from you about topics you want to see and questions you may have. Leave us a comment below!