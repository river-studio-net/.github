---
layout: post
title:  "Choosing an IAM Provider for Parley"
date:   2026-08-27 10:00:00 +0000
tags: rust design parley parley-server
---

## Choosing An Identity Provider For Parley
Parley is the project I am working on these days. It is an E2EE messaging application that is focused around a community rather than a single user, much like discord. This post is another post in the series of explainers of the architectural decisions involved in the making of a project this size and with this profile. I am writing everything I do by hand - no AI involved, including the contents of my posts. 
This particular post will explore the world of Identity Providers, IAMs (Identity Access Management) for short, and the decision of which one is going to be used for Parley. 

I will assume some knowledge about app development and programming here, but much as I learn this myself I will try and make the post introductory to the world of IAM for application development. 

### Requirements
Ideally, what I expect of whatever system I end up choosing for the project is that it will solve most if not all handling of authentication handling, including user registration, login, progile management, deletion of users, and validation of user identity from the backend of the application. I expect some user-app flow of:
* User tries to perform a restricted operation from the frontend application, calling the backend service
* Backend application redirects user to the IAM solution
* User does user management things and logs in
* User get's session token
* User uses session token to authenticate to the backend service, and the backend service can authorize the session token with the IAM system
* User session is now valid for X hours and then repeat

Another important aspect is that the solution will be open source and free to self host, since this is one of the promises of Parley. 
And hopefully there is an SDK in rust to make the development easier for me.
Socials login is also very important, and hopefully multiple forms of them to ease onboarding for the users. 

So to sum up we need a solution that provides:
- Open source self hosting support
- IAM and Identity Management features
- No need for frontend access directly but unauthenticated access is a must
- Rust SDK
- If there is a solution for gRPC integration that would be amazing, more on that in a future post

## Landscape
### Closed Source Alternatives
There are a bunch of well known and trusted IAM providers that we can rule out from the beginning for being closed source - Auth0, AWS Cognito and Okta just to name a few. This helps us reduce the search space but also reduces the available polished solutions we can explore. 

### Open Source Alternatives
The main names I strated seeing when exploring the world of FOSS IAM solutions are Authenik, Zitadel, Authelia, and Keycloak. Other names that popped up are oauth2proxy, Pomerium, Kandim and Ory Kratos. Let's take them apart. 
Disclaimer here - I did not use any of the services we are exploring, and not saying they are bad or wrong. The only observation I am trying to make is whether they are suitable for my usecase or not - and that's it. These are my technical thoughts and opinions, nothing more. 

#### Comparison
Since there are so many tools, I summed up the results of my research into this table.

| **_App_** 	| (**Keycloak**)[https://github.com/keycloak/keycloak] 	| (**Authentik**)[https://github.com/goauthentik/authentik] 	| (**Zitadel**)[https://github.com/zitadel/zitadel] 	| (**Authelia**)[https://github.com/authelia/authelia] 	| (**OAuth2Proxy**)[https://github.com/oauth2-proxy/oauth2-proxy] 	| (**Pomerium**)[https://github.com/pomerium/pomerium] 	| (**Ory**)[https://github.com/ory/kratos] 	| (**Kanidm**)[https://github.com/kanidm/kanidm] 	|
|---	|:---:	|:---:	|:---:	|:---:	|:---:	|:---:	|:---:	|:---:	|
| _Self Hosting_ 	| V 	| V 	| V 	| V 	| V 	| V 	| V 	| V 	|
| _GH Stars_ 	| 36k 	| 25k 	| 15k 	| 28k 	| 15k 	| 5k 	| 14k 	| 5k 	|
| _OAuth2/OICD_ 	| V 	| V 	| V 	| V 	| V 	| V 	| V 	| V 	|
| _Socials Login_ 	| V 	| V 	| V 	| ? 	| V 	| ? 	| V 	| ? 	|
| _Multiple Social Logins_ 	| V 	| V 	| V 	| ? 	| X 	| ? 	| V 	| ? 	|
| _gRPC Support_ 	| X 	| ? 	| V 	| X 	| X 	| X 	| In Premium Only 	| X 	|
| _Rust SDK_ 	| V 	| V 	| V 	| X 	| X 	| X 	| V 	| V 	|
| _Resources Class_ 	| Heavy 	| Heavy 	| Lightweight 	| Lightweight 	| Lightweight 	| ? 	| Medium 	| Lightweight 	|

As you can see, it looks like Zitadel is perfect for my usecase! To be honest I am surprised, as as I was going along the research it looked like Ory is going to be the final contendant, but I am as happy to proceed further into development with Zitadel. 
With a community guage of 15k stars on Github and a support discord server, it's promising a solid support system, and there are even quickstart guides for Rust which is vert appealing.
Overall it feels like a solid choice I am confident about.
