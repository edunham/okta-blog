---
layout: blog_post
title: "The End of Third-Party Cookies"
author: edunham
by: advocate
communities: [devops,security,mobile,.net,java,javascript,go,php,python,ruby]
description: ""
tags: []
tweets:
- ""
- ""
- ""
image: blog/3pc/social.jpg
type: awareness
---

## What are third-party cookies?

Cookies are as old as the internet. Historically, cookies were among the only options for personalizing a user's online experience and carrying their preferences from page to page. First-party cookies are issued by the web site where they're used, and third-party cookies come from other domains. 

Third-party cookies allow user behavior to be tracked across different sites. These cookies are now widely abused to collect and share uesrs' data. For the legitimate use cases which used to require third-party cookies, like federated logins and multi-brand identity providers, more secure options are actively being developed. 

Today, the drawbacks to users' security and privacy from third-party cookie implementations outweigh their benefits so much that all major browsers are phasing them out. Safari has blocked thrd-party cookies for years, and Firefox retricts third-party cookies associated with trackers. Chrome is now [phasing out third-party cookies](https://developers.google.com/privacy-sandbox/3pcd) in 2024. 

If your code uses Okta features that rely on third-party cookies, this means that you'll need to make some changes to keep the identity experience working as intended. 


## Does your application use third-party cookies? 

Okta's core features do not rely on third-party cookies. However, third-party cookies are used in several areas to enhance the login experience.  Here are the design patterns in which Okta uses third-party cookies. If your application is in one of these categories, please test its behavior with third-party cookie deprecation. 

Okta uses cookies to let applications introspect and extend user sessions. Cookies aren't required for basic login functionality. 

### Third-party cookie deprecation affects web applications that rely on the Okta session for user context

If an application hosted on your domain (`mycompanyapp.com`) redirects to your Okta subdomain (`mycompany.okta.com`) for login and then returns users to your own domain, third-party cookie restrictions will limit how your app can introspect or extend the Okta session. 


### Third-party cookie deprecation affects customer-hosted Okta Sign-In Widget and customer-built login applications

If you're hosting your own sign-in experience on a separate top level domain from your main app, you may be using third-party cookies. You might be hosting your own sign-in experience by cloning the Okta Sign-In Widget from GitHub or installing it from NPM to embed in your application, or you might have built your own custom sign-in using Okta's APIs. 

If your sign-in experience is hosted on the same top-level domain as your application, third-party cookie deprecation won't affect its behavior. 

If the sign-in experience and app are on different top-level domains, third-party cookie deprecation will break its ability to introspect and extend sessions, because these features use cookies. Authentication will still be possible, and tokens will still be returned, because these features do not rely on cookies. 

### Third-party cookie deprecation affects "remember me" features

"Remember Last Used Factor" (RLUF), for automatically selecting the user's preferred factor, uses third-party cookies. The "keep me signed in" feature of Okta Identity Engine and "Remember me" feature of Okta Classic rely on third-party cookies when the login application is on a different domain from the main app.  

## When will this affect you? 

Google has made an exemption for Okta's third-party cookies until the end of 2024. However, you can set Chrome's flags to simulate how the browser will treat Okta's third-party cookies after that exemption ends. 

### Test your application today!

To simulate how Chrome will treat Okta's third-party cookies in 2025 and beyond, follow [the Okta help center's testing guide](https://support.okta.com/help/s/article/deprecation-of-3rd-party-cookies-in-google-chrome?language=en_US). 

## What's next? 

Here on the Okta Developer Blog, we'll keep you updated about how to mitigate each type of third-party cookie impact.

* Learn more about [how blocking third-party cookies affects Okta environments](https://support.okta.com/help/s/article/FAQ-How-Blocking-Third-Party-Cookies-Can-Potentially-Impact-Your-Okta-Environment?language=en_US).
* See the [Okta session cookies guide](https://developer.okta.com/docs/guides/session-cookie/main/) for more on how cookies are used.
* [Use Chrome's feature flags](https://support.okta.com/help/s/article/deprecation-of-3rd-party-cookies-in-google-chrome?language=en_US) to test your login experience with third-party cookies disabled.  