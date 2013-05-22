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

Our initial implementation used the now-defunct now.js for realtime client notifications, but we switched to Faye as our requirements became clearer.

For this demo, we'll show you how to build your own LiveCollection implementation.

Try the live demo we've depoloyed on [nodejitsu](http://faye-live-demo.jit.su/).

TL;DR
=====================

With only a few moving parts, and minimal changes to your Backbone application, it's easy to drop in a real-time push-based live collection component.


Technologies Used
---------------------

This demo uses __[Faye](http://faye.jcoglan.com/)__, __[Backbone.js](http://backbonejs.org/)__, __[Mongoose.js](http://mongoosejs.com/)__, __[Baucis](https://github.com/wprl/baucis)__, __Node.js__, __Express.js__ and __MongoDB__ (phew!)

For a front-end, we're using [TodoMVC's](http://todomvc.com/architecture-examples/backbone/) Backbone.js example, with a few changes:

* LocalStorage has been switched for a REST backend.
* The REST backend is implemented as a Node.js http server connecting to a Mongodb backend, using the technologies mentioned above.

*We'll take a fairly standard stack, like this:...*
![The basic stack that we'll be adapting for LiveCollections](/images/live-collections-before.png)

*... and extend it with additional client and server components, like this:*
![The final technology stack](/images/live-collections-after.png)

Why Faye?
=============

> "Faye is a publish-subscribe messaging system based on the Bayeux protocol. It provides message servers
> for Node.js and Ruby, and clients for use on the server and in all major web browsers."
> -- [http://faye.jcoglan.com/](http://faye.jcoglan.com/)

Alternatives to Faye include [Engine.io](https://github.com/LearnBoost/engine.io), [Socket.io](http://socket.io/), [straight sockets](http://caniuse.com/#search=websockets), but here are some reasons we feel Faye is right for us:

* _A well documented network protocol_: Faye is built on top of the well-established and open [Bayeux](http://svn.cometd.com/trunk/bayeux/bayeux.html) protocol. Being open means ObjectiveC, Android, Ruby and Javascript implementations exist and should be able to interoperate.
* _High-level messaging interface_: Faye provides a right-level API that allows us to focus on publishing messages to channels, while Faye deals with client handshaking, reconnections, etc
* _Fallback_: in the real-world of outdated browsers, websocket implementations are notoriously flakey. Faye falls back to comet-style implementations when web sockets aren't behaving correctly.
* _Easy_: Faye is easy to understand and use.

{% gist 5621243 %}


Using a REST-like scheme for Pub/Sub Channels
---------------------------------------------

Backbone Collections use REST URL endpoints and CRUD operations are performed using `POST`, `PUT` and `DELETE` operations. Faye uses channels, not URL endpoints, and passes messages. We need a way of mapping between these two domains. Luckily, it's quite easy to do:

For each LiveCollection model, we will use a separate channel. The name of the channel matches the relative REST base-url. For example, for the resource `http://server/api/todos/`, updates will be published on the channel `/api/todos/`.

The messages passed over the channel are simple JSON messages. Messages take the form:

	{
		method: 'POST/PUT/DELETE',
		body: { ... }
	}

The method attribute indicates create (POST), update (PUT) or delete (DELETE) and the body is the object associated with the event.

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

Changes to the Server to Implement Push
=======================================

Firstly, we'll need to attach a Faye server instance to our httpServer instance, like this:

{% gist 5621371 %}

The next step is to publish messages to the appropriate channel whenever the mongo collection is modified. Since we're already using Mongoose, the easiest way to generate notifications is by attaching a mongoose middleware component to the `Todo` schema.

> Middleware are functions which are passed control of flow during execution of init, validate, save and remove methods. There are two types of middleware, pre and post.

Unfortunately, in the `post('save')` middleware you can't easily tell if the save operation was for a create or an update. Therefore we use a small utility, `attachListenersToSchema()`, to help us out. Dig into the source if you're interested in how we do this.

{% gist 5621437 %}

Changes to the Client
=====================

On the client, we'll extend the `Backbone.Collection` class to listen to events from Faye. The class subscribes to it's Faye channel in it's constructor, and messages are handled in the `_fayeEvent()` method.

{% gist 5621496 %}

Then we change the base class of the Todo collection to extend the LiveCollection, like this:

{% gist 5623171 %}

And we're done!


Here Be Dragons
=====================

Obviously we've had to gloss over a few details; here's a few things to look out for:

1. _Views need to respond to non-UI-driven change events_: You may need to make changes if your view code responds to UI changes instead of collection/model changes. In the TodoMVC example, the remove model code needed to be modified in order for push deleted to show in the UI. We highly recommend [Marionette.js](http://marionettejs.com/)  to ease pain here.

1. _Event ordering_: Sometimes the client will receive Faye events before the RESTful operation that caused the event has completed, other times not. Don't make any assumptions about the order and timing of events. Read how we deal with some of these problems in the next section.

1. _Security_: Obviously this demo doesn't use security, but it's something you'll need to consider carefully.

1. _Dodgy websockets_: Websockets are awesome when they work. Faye deals with much of the pain, but not all of it. Expect weird edge-cases when dealing with bad network connections (or mobile connections), iOS, computers that have awoken from sleep and other scenarios.

1. _Testing_: creating a solid test environment when working with push technologies is hard.

Drama with Duplicates: Dealing with unexpected timing issues
------------------------------------------------------------

The LiveCollection's `_createEvent()`, `_updateEvent()` and `_removeEvent()` methods are fairly straightforward, but one issue that we'll discuss here is the problem of duplicates that occurs sometimes when saving a new object (create), and how we deal with this issue.

As mentioned, sometimes the order of events is not what you might expect. For example, take a look at the following sequence when a client saves a new model to the collection:

1. HTTP POST request is initiated with `{ id: null, title: 'Hello' }`, client (asynchronously) awaits a response.
1. On Faye, the client receives create event from server with `{ id: X, title: 'Hello' }`
1. HTTP response is received:  `{ id: X, title: 'Hello' }`

At step 2, the client does not know whether the event is related to the outstanding POST event, or is an event from another client. It's only once the HTTP POST response is received that the client is able to
tell that the event was in fact for the corresponding operation, by matching on the id. If the create was for the same object, the event can safely be ignored. If not, the created object should be inserted into the collection.

We use the following strategy to deal with this situation:
* If there are no outstanding POST operations, always insert the object immediately.
* Otherwise, wait until all the outstanding POST operations have completed.
* Once they have completed, check if the id of the event matches one of the newly sync'ed objects.
* If it does, ignore the event
* If not, insert the event's body into the collection - the create must have been from another client.

This may seem fairly complicated, but the code is actually quite simple:

{% gist 5621667 %}


In Summary
============

With a small amount of effort you can adapt your existing Backbone.js application to use Live Collections. Just watch out for gotchas. Give it a try and let us know how it goes.

