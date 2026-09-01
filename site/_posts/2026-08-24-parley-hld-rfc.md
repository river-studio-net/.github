---
layout: post
title:  "Parley High Level Design RFC"
date:   2026-08-24 10:00:00 +0000
tags: rust design parley rfc parley-server architecture
---

## Parley
Parley is the name of the future e2ee messaging app I am hoping to build over the next few months. It is intended to be a discord replacement - as opposed to messaging apps where the first class citizen of the system is a user and therefore one-to-one conversations, the first class citizen of Parley is going to be a channel, so the communties channels create are the main organizing feature of the system - like in discord. 

## RFC 
This document is intended as an actual request for comments. I hope that a few past collegues will be able to review it and inspire me with their expertise about system design and large scale planning.

## Context and Scope
At this stage, Parley is a greenfield project. There are no existing components to integrate and no existing infrastructure to rely on. This document is intended to propose a high level design for the application layer of the system, i.e. there will be no talk of infrastructure and hosting solutions yet. These will be considered as implmenetation details that will be considered in a later stage of the development, after there is an actual system to host.

Parley is intended to be decentralized in such a way that the frontend client will not depend on a specific server and will be able to connect to multiple servers to access channels and communities hosted on many servers and participate in conversations happening on many such hosts. 
The scope of this document is to design a single server structure. The responsibility on managing connection and syncing the client to many a host will be on the frontend client application, so no server syncing behavior is needed. Moreover, no talk about future plans of the projcet (such as connection to [ActivityPub](https://activitypub.rocks/)) will be included here.

### Terminology
* Parley: The name of the system including the hosted service that clients connect to and the clients that connect to it. Also including other hosted servers and any client with capabilities of conncting to the Parley network. 
* Parley Server: The focus of this document. The main component of the Parley network/system - serving as an entry point to access channels, communities and other users information and connection details.
* Parley Client: A user operated application (mobile, desktop, TUI, etc.) that connects to Parley Servers. The user account should reside in a single server host but the client app may and should connect to multiple servers to access the full network. 
* User: Analogous to a single client connecting to Parley. A human is expected to be behind a user/client. 
* Channel: a one-to-one or many-to-many conversation between users. Icludes both "direct messages" channels which are disconnected from a higher level entity, multi-DMs analogous to a group direcet messaging channel, and channels that are part of a lerger entity - a Category and a Commune.
* Category: A group of channels roofed by a user under as single title. A way to organize channels into groups inside of a Commune. 
* Commune: A group of categories roofed by a user under a singe title, serving as a single entity where users join, leave and participate in conversations that happend in the Commune's channels. 
* E2EE: End-to-end-encryption. The methods of encrypting Channel's content. The metadata and structure of the channels are not encripted, but the text used inside (including channel, category, and commune names) is.

### Goals and Non-Goals
#### Goals
* A server that enables the following user flow via API calls:
  * User management
  * Commune management 
  * Channel and Category management 
* Documented API for the above actions.

#### Non-Goals
* E2EE should be discussed in a separate scope and context. The E2EE features of Parley will be enabled on top of a working system of messaging and we will treat the cyphertexts as raw text in the meantime. 
* Live features like streaming and voice chat.
* User client application.
* Decentralized servers syncing functionallity.

## Design
### System-Context
#### Principles
With the goal of developing a Parley server in mind, we are going to have the server be our most meaningful component. It's designed as a modular monolith to enable expansion and future break into sub systems if needed and meant to be a single repository written in Rust for safety and performance. See tradeoffs 1 and 2. 
As such, when mentioning a service that the server provides, I mean that this is part of the server's functionallity and not as a separate microservice.

#### Diagram
![[HLD Architecture Diagram v2.png]]
Notes: 
- E2EE service and live service design are out of scope for this document.
- DB type choice is out of scope for this document.
- User management system choice is out of scope for this document. 
- Client design is out of scope for this document. 

### APIs
#### Auth Service
- User authentication
- User setup
- User details change and management
- User directory search
- CRUD blocklist
- CRUD friends list

#### Commune Service
- CRUD Commune + Roles + Permissions
- CRUD Category + Permissions
- CRUD Channel + Permissions
- CRUD Message + Permissions
- Get Channel/User Messages

### Tradeoffs
1. Modular monolith vs. microservices: Being a team of one gal, I believe in the thumb rule of one system to maintain per teammate, and therefore a modular monolith makes much more sense. Keeping it modular makes sure that if the time comes and the team grows and the need occurs, different parts of the system can be split and deployed to different targets.
2. Rust: This project started as my learning rust project. That being said, it is also true that rust is optimized for this kind of systems programming, and it's safety and performance promises will benefit the project immensely. 

## Cross Cutting Concerns
I am mostly concerned about the future efforts in this project and how they integrate into the system I am designing.
### E2EE
I am still worried about implementing E2EE over the system. The server will need to support metadata and many strings handling without ever relying upon understanding them or reading them ecxept for sending them to the users. This is no small effort and keeping honest to it will require a lot of work and discipline. 
### Live Services
I expect Parley to support live voice and video chat, as well as have streaming capabilities. The plan is to implement the different services the server will handle almost like plugins, so that adding new functionallity is easy and fast.
