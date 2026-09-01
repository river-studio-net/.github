---
layout: post
title:  "Choosing a DB for Parley"
date:   2026-09-13 10:00:00 +0000
tags: parley postgresql sql design architecture
---

Choosing a DB is not an easy task when you are trying to plan for scale and many many users, right? Well, I looked quite a bit around and found that the best solution was right in front of me the whole time - just use PostgreSQL.
I'll try to explain my reasoning for this in this post and detail the research journey, but the bottomline is that this really is a problem for scale beyond my imagination at the moment and if the time comes and I need something better suited than the most used extensible open source option then I'll reopen this research and rethink my life choices, but until then PostgreSQL will do. 

## Requirements 
As always, we have a few requirements from a DB for Parley. The first and most basic one is that the tech stack of Parley is designed to be open sourced in it's entirety. While not directly disqualifying AWS RDS and other hosted solutions since they are based on open source solutions like PostgreSQL they are still closed source in their core and anyway we want to focus this discussion on the core technology and not the hosting proposition. So the question is PostgreSQL or something else and not AWS RDS or Oracle Cloud etc.. 
The open source requirements also make some other options like VoltDB and MaterializeDB unavailble for us even if they could fit the product needs well. The main competitor to PostgreSQL I have seen that is also open source is SpacetimeDB (though a strictly #FOSS look will also note that the license of SpacetimeDB is not exactly open source to the level of allowing us to use it freely without consequense, which is one of the reasons I have chosed to progress with PostgreSQL instead eventually). 

Apart from being open source, we also want the solution to be a RDBMS - a Relationel DataBase Management System - and notably a SQL DB and not a document DB. The reason for that is simple - the structure of a Commune is very relational in nature and most of the data we are going to store is text anyways so that's no concern, and the simplicity and closed characteristics of RDBMS provides the safety and structure that go well with an open source application that we expect to have many types of clients accessing. This is also the reason for choosing gRPC for the communication protocol but more on that in another post. 
This requirement filters out the solutions used over the yeas at Discord itself - ScyllaDB and CassandraDB - since they both are document DBs. The ruling out of ScyllaDB is especially note-worthy since it is (the current choice for Discord itself)[https://discord.com/blog/how-discord-stores-trillions-of-messages].

This leaves us in the world of open source RDBMSs, in which PostgreSQL is the unchallenged king and we have the new kid in town - SpacetimeDB - to check out too. 

## SpacetimeDB 
SpacetimeDB is, in its CEO's words, a DataBase operating system. In such, it encapsulates the bussiness logic and data of the application in a single place to package it in a single, all-replacing component that is meant to replace the DB as well as the backend service of the application. It has many claims for operational eliteness and throughput gains, as well as elevated user experience to back the dev process with. The throughput benchmarks are insane and promise a few orders of magnitude better performance than PostgreSQL systems, due to two main factors - the first being the bundling of the db and the backend into a single process and thus eliminating network latency between services and the second being their internal design which separates the commit log from the response API which eliminates persistency latency and decouples throughput from latency almost completely. This separation is not nessesarily complete and doesn't mean that persistency and durability of data aren't guarenteed. 
(Source)[https://www.youtube.com/watch?v=-GhNDYCchqc]
(Source)[https://www.youtube.com/watch?v=k7ZemI82Qxs]

### Concerns
There are a few reasons to avoid SpacetimeDB from my end of things. The first and probably final note too is it's (license)[https://github.com/clockworklabs/SpacetimeDB?tab=License-1-ov-file#:~:text=You,Service%2E] - it is not #FOSS per-se. Though the license is GNU based it is specifically limiting the amount of instances allowed in a deployment and when planning for scale this is unacceptable. There are also operational concerns that strengthen with this restriction such as the single-threaded-ness of the DB and its management of many concurrent readers and writers strategy, which the CEO described as just "there is a queue" and without further specification this might be a source of discomfort for a distributed application such as Parley. 
The secong technical concern is the community support and open knowledge available for the DB - while PostgreSQL is the most used RDBMS in the world and the open source community, SpacetimeDB is new and it's most recent stable version 2.0 is only 6 months old at the time of writing this post. Another concern is the lack of extensiblity of SpacetimeDB, where in PostgreSQL I just know that if there is a problem or a missing feature I have a (repo)[https://gist.github.com/joelonsql/e5aa27f8cc9bd22b8999b7de8aee9d47] of more than 1000 extensions to help me and a (stable ecosystem of plugins I can write myself)[https://github.com/pgcentralfoundation/pgrx] in Rust if the need arises. For a project this young and with a future so uncertain anything but PostgreSQL might be a complexity killing blow in terms of development knowledge and exprience. 

## Decision
(Just use PostgreSQL)[https://www.youtube.com/watch?v=3JW732GrMdg].
