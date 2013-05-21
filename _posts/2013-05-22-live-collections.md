---
layout: post
title: Live Collections using Backbone.js, Faye and Node
---

Why LiveCollections?
=====================

Troupe is a realtime collaborative application. Having a real-time, push-based application is essential.
At the same time, we were keen to leverage the power of Backbone.js, so we decided to extend Backbone with
real-time LiveCollection capabilities to enable realtime push in Backbone collections.

__Our aim: create a real-time drop-in replacement for Backbone collections that plays nicely with Node.js__

We were already using Mongoose.js for Mongo persistence, so it made sense for us to leverage the Mongoose middleware concept to generate CRUD notifications.

Our initial implementation used the now-defunct now.js for realtime client notifications, but switched to Faye as our requirements became clearer.

For this demo, we'll show you how to build your own LiveCollection implementation.

Try the live demo on [nodejitsu](http://faye-live-demo.jit.su/)

TL;DR
=====================

With only a few moving parts, and minimal changes to your Backbone application, it's easy to drop in a real-time push-based live collection component.


Technologies Used
---------------------
* __[Faye](http://faye.jcoglan.com/)__
* __[Backbone.js](http://backbonejs.org/)__
* __[Mongoose.js](http://mongoosejs.com/)__
* __[Baucis](https://github.com/wprl/baucis)__: Baucis allows you to generate REST interfaces from Mongoose schemas. For the sake of brevity, we'll use it in this demo. Although rolling-your-own REST interface or using another library will work equally well.

* Down below: __Node.js__ / __Express.js__ / __MongoDB__


For the demonstration, we're using [TodoMVC's](http://todomvc.com/architecture-examples/backbone/) Backbone.js, with a few small changes:

* LocalStorage has been switched for a REST backend.
* The REST backend is implemented as a Node.js http server connecting to a Mongodb backend, using the technologies mentioned above:

*We'll take a fairly standard stack, like this:*
![The basic stack that we'll be adapting for LiveCollections](/images/live-collections-before.png)

*And extend it with additional client and server components, like this:*
![The final technology stack](/images/live-collections-after.png)

Why Faye?
=============


> "Faye is a publish-subscribe messaging system based on the Bayeux protocol. It provides message servers
> for Node.js and Ruby, and clients for use on the server and in all major web browsers."
> -- [http://faye.jcoglan.com/](http://faye.jcoglan.com/)

There are many alternatives to Faye that we could've chosen - [Engine.io](https://github.com/LearnBoost/engine.io), [Socket.io](http://socket.io/), [straight sockets](http://caniuse.com/#search=websockets), but here are some reasons we selected Faye:

* _A well documented network protocol_: Faye is built on top of the well-established and open [Bayeux](http://svn.cometd.com/trunk/bayeux/bayeux.html) protocol. Being open means ObjectiveC, Android, Ruby and Javascript implementations exist and should be able to interoperate (*theoretically!*).
* _High-level messaging interface_: Faye provides a right-level API that allows us to focus on publishing messages to channels, while Faye deals with client handshaking, reconnections, etc
* _Fallback_: in the real-world of outdated browsers, websocket implementations are notoriously flakey. Faye falls back to comet-style implementations when web sockets aren't behaving correctly.
* _Easy_: Faye is super easy to understand and a pleasure to use!

{% gist 5621243 %}


Using a REST-like scheme for Pub/Sub Channels
---------------------------------------------

Backbone Collections use REST URL endpoints and CRUD operations are performed using `POST`, `PUT` and `DELETE` operations. Faye uses channels, not URL endpoints, and passes messages. We need a way of mapping between these two domains. Luckily, it's quite easy to do:

For each LiveCollection model, we will use a separate channel. The name of the channel matches the relative REST base-url. For example, for the resource `http://server/api/todos/`, updates will be published on the channel `/api/todos/`.

The messages passed over the channel are simple JSON messages. All messages have the form:

	{
		method: 'POST/PUT/DELETE',
		body: { ... }
	}

Where the method relates to whether the operation is a create, update or delete and the body is the object associated with the event.

<table>
	<thead>
		<tr>
			<th>Operation</th>
			<th>REST (client to server)</th>
			<th>Faye (broadcast, server to clients)</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td></td>
			<td><code>GET /api/todos/</code></td>
			<td><code>client.subscribe(‘/api/todos/’, ...)</code></td>
		</tr>

		<tr>
			<td>Create</td>
			<td><code>POST /api/todos/</code></td>
			<td><code>{ method:“POST”, body: {...} }</code></td>
		</tr>

		<tr>
			<td>Update</td>
			<td><code>PUT /api/todos/id</code></td>
			<td><code>{ method:“PUT”, body: {...} }</code></td>
		</tr>

		<tr>
			<td>Delete</td>
			<td><code>DELETE /api/todos/id</code></td>
			<td><code>{ method:“DELETE”, body: {...} }</code></td>
		</tr>

	</tbody>
</table>

Changes to the Server
=====================

Firstly, we'll need to attach a Faye server instance to our httpServer instance, like this:

{% gist 5621371 %}

The next step is to publish messages to the appropriate channel whenever a model is created, updated or deleted. Since we're already using Mongoose, the easiest way to generate notifications is by attaching a mongoose middleware component to the `Todo` schema.

> Middleware are functions which are passed control of flow during execution of init, validate, save and remove methods. There are two types of middleware, pre and post.

Unfortunately, in the `post('save')` middleware you can't easily tell if the save operation was for a create or an update. Therefore we use a small utility, `attachListenersToSchema()`, to help us out. Dig into the source if you're interested in how we do this.

{% gist 5621437 %}

Changes to the Client
=====================

On the client, we'll extend the `Backbone.Collection` class to listen to events from Faye. The class subscribes to it's Faye channel in it's constructor, and messages are handled in the `_fayeEvent()` method.

{% gist 5621496 %}

Dealing with unexpected timing issues
-------------------------------------

The `_createEvent()`, `_updateEvent()` and `_removeEvent()` methods are fairly straightforward, but one issue that we'll discuss here is the problem of duplicates that occurs sometimes when saving a new object (create), and how we deal with this issue.

Sometimes, the order of events is not what you might expect. For example, take a look at the following sequence when a client saves a new model to the collection:

1. HTTP POST request is initiated with `{ id: null, title: 'Hello' }`, client (asynchronously) awaits a response.
1. Client receives create event from server with `{ id: X, title: 'Hello' }`
1. HTTP response is received:  `{ id: X, title: 'Hello' }`

At step 2, the client does not know whether the event is related to the outstanding POST event, or is an event from another client. It's only once the HTTP POST response is received that the client is able to
tell that the event was in fact for the corresponding operation, by matching the on the id. If the create was for the same object, the event can safely be ignored. If not, the created object should be inserted into the collection.

We use the following strategy to deal with this situation:
* If there are no outstanding POST operations, always insert the object immediately.
* Otherwise, wait until all the outstanding POST operations have completed.
* Once they have completed, check if the id of the event matches one of the newly sync'ed objects.
* If it does, ignore the event
* If not, insert the event's body into the collection (the create must have been from another client)

This may seem fairly complicated, but the code is actually quite simple:

{% gist 5621667 %}

