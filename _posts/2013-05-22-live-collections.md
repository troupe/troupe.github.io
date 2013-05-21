---
layout: post
title: Live Collections using Backbone.js, Faye and Node
---

Why LiveCollections?
---------------------

Troupe is a realtime collaborative application. Having a real-time, push-based application is essential.
At the same time, we were keen to leverage the power of Backbone.js, so we decided to extend Backbone with
real-time LiveCollection capabilities to enable realtime push in Backbone collections.

__Our aim was therefore to create a real-time drop-in replacement for Backbone collections that worked with a Node.js server infrastructure.__

We were already using Mongoose.js as a Mongo persistence framework, so it made sense for us to leverage the Mongoose middleware concept to generate CRUD notifications.

Our initial implementation used the now-defunct now.js for realtime client notifications, but later switched to Faye.

For this demo, we'll show you how to build your own LiveCollection implementation.

(delete)

Backbone is a hugely popular framework which helps tackle some of the complexity of building modern Javascript client-side applications.

A modern single-page javascript application is likely to remain open for a long period without being refreshed. With Google Mail, Twitter and Facebook, users have become accustomed to sites that remain up-to-date without having to be refreshed.

So is there a way that we can easily extend Backbone.JS to support real-time collection updates?

(delete)

Technologies
---------------------
* __[Faye](http://faye.jcoglan.com/)__
* __[Backbone.js](http://backbonejs.org/)__
* __[Mongoose.js](http://mongoosejs.com/)__
* __[Baucis](https://github.com/wprl/baucis)__: Baucis allows you to generate REST interfaces from Mongoose schemas. For the sake of brevity, we'll use it in this demo. Although rolling-your-own REST interface or using another library will work equally well.

* Down below: __Node.js__ / __Express.js__ / __MongoDB__


For the demonstration, we're using [TodoMVC's Backbone.js implementation](http://todomvc.com/architecture-examples/backbone/), with a few small changes:

* LocalStorage has been switched for a REST backend.
* The REST backend is implemented as a Node.js http server connecting to a Mongodb backend, using the technologies mentioned above:

![Technology stack 1](/images/live-collections-before.png)


Why Faye?
---------

> "Faye is a publish-subscribe messaging system based on the Bayeux protocol. It provides message servers
> for Node.js and Ruby, and clients for use on the server and in all major web browsers."
> -- [http://faye.jcoglan.com/](http://faye.jcoglan.com/)

There are many alternatives to Faye that we could've chosen - [Engine.io](https://github.com/LearnBoost/engine.io), [Socket.io](http://socket.io/), straight sockets, but here are some of the reasons we selected Faye:

* _A well documented network protocol_: Faye is built on top of the well-established and open [Bayeux](http://svn.cometd.com/trunk/bayeux/bayeux.html) protocol. Being open means ObjectiveC, Android, Ruby and Javascript implementations exist and should be able to interoperate (*theoretically!*).
* _High-level messaging interface_: Faye provides a right-level API that allows us to focus on publishing messages to channels, while Faye deals with client handshaking, reconnections, etc
* _Fallback_: in the real-world of outdated browsers, websocket implementations are notoriously flakey. Faye falls back to comet-style implementations when web sockets aren't behaving correctly.
* _Easy_: Faye is easy to understand and easy to use.

(delete)
The purpose of this article is to show how real-time live collections can be added to an existing Node.js / Backbone.js application with a minimum of effort.

(delete)


![Technology stack 2](/images/live-collections-after.png)


First take on the server
-------------------------

The node.js server is almost completely contained in index.js.

