---
layout: blog_post
title: "Manage your Okta Terraform App from Terraform"
author: edunham
by: advocate
communities: [devops,security]
description: ""
tags: [terraform, workshop]
tweets:
- ""
- ""
- ""
image: blog/terraform-workshop/tf-workshop-social-image.jpg
type: awareness
---

## Intro
In the [Terraform Workshop](), you set up Terraform to manage infrastructure in your Okta Developer Organization. 

One goal of using Terraform is spending less time in the admin console, and more time in your code. When your interactions with any cloud provider are mediated through infrastructure configuration code, your team can use that code's test suite to externalize their memory and knowledge of best practices, and never make the same mistake twice. 

That's a lofty goal, but it's hard to get there when you still have to use the admin console any time you need to change the configuration of the Terraform provider's own Okta integration. 

But Terraform can help! In this workshop, you'll use Terraform to import and manage the Terraform integration that your Terraform code uses to manage infrastructure in your Okta account. 

**Wait, what?**

Let's break this down into smaller pieces. 

**Terraform** runs on your laptop when you're experimenting with it, but might run in your automation later. Terraform communicates with Okta using credentials specified in the Okta Terraform Provider in your Terraform Project. 

**Okta** is a software product that helps its customers manage identity in the products they build and use. This kind of product is also known as an Identity Provider, or IdP. The word "provider" showing up in both "identity provider" and "Terraform provider" is just a coincidence. 

When Terraform talks to Okta to create resources for you, it uses secrets that you provide. This is good, because you wouldn't want just anyone taking the kinds of actions in Okta that your Terraform is allowed to! This connection between Terraform and Okta has two ends, whose secrets need to match. On the Okta end, you have an object called an App Integration, or just an Okta App. The app is where Okta keeps track of what kinds of actions Terraform is allowed to take, using Scopes, and other settings. On Terraform's end of this connection is the provider configuration block, which contains the Client ID of the app and an RSA key created from the PKCS-1 key in the app. 

**Connection**

In the Terraform Workshop, you set up a connection between your Terraform code and an app within your Okta account. 

The Okta side of this connection, the app integration, is just another Okta resource that Terraform can manage! 

Sometimes you'll want to change the Okta app settings for Terraform. Wouldn't it be nice if you could do this directly from your Terraform code? 

### grant scopes on okta app

There's only one change necessary in the Okta app to do this: in your Terraform application's "Okta API Scopes" tab, grant `okta.apps.manage`. 

### grant scopes in the Okta Terraform Provider

in main.tf in your Terraform project, add the `okta.apps.manage` scope. 

### Get the app ID to prepare for import

When you imported resources to Terraform in the introductory workshop, you looked up their ID by extracting it from the URL on the admin console. Now that you're getting more comfortable with Terraform, you can retrieve the ID another way: By using a data source. 

This will look like: 

```tf
data "okta_app_oauth" "tf" {
  label = "Terraform Workshop"
}

output "workshop_oauth_app_id" {
  description = "Issuer URL for org's default auth server"
  value       = data.okta_app_oauth.tf.id
}

```
Note that the app lookup is slow, like 30 seconds slow, because [under the hood](https://developer.okta.com/docs/reference/api/apps/#list-applications) it's actually doing a whole search for all the apps with matching IDs. 

But hey, it saves copying and pasting from the web browser!

### Import the resource

```tf

import {
  to = okta_app_oauth.tf
  id = data.okta_app_oauth.tf.id
}
```