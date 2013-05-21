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

Technologies
---------------------
* __[Faye](http://faye.jcoglan.com/)__
* __[Backbone.js](http://backbonejs.org/)__
* __[Mongoose.js](http://mongoosejs.com/)__
* __[Baucis](https://github.com/wprl/baucis)__: Baucis allows you to generate REST interfaces from Mongoose schemas. For the sake of brevity, we'll use it in this demo. Although rolling-your-own REST interface or using another library will work equally well.

* Down below: __Node.js__ / __Express.js__ / __MongoDB__


For the demonstration, we're using [TodoMVC's](http://todomvc.com/architecture-examples/backbone/) Backbone.js, with a few small changes:

* LocalStorage has been switched for a REST backend.
* The REST backend is implemented as a Node.js http server connecting to a Mongodb backend, using the technologies mentioned above:

![Technology stack 1](/images/live-collections-before.png)


![Technology stack 2](/images/live-collections-after.png)

Why Faye?
---------

> "Faye is a publish-subscribe messaging system based on the Bayeux protocol. It provides message servers
> for Node.js and Ruby, and clients for use on the server and in all major web browsers."
> -- [http://faye.jcoglan.com/](http://faye.jcoglan.com/)

There are many alternatives to Faye that we could've chosen - [Engine.io](https://github.com/LearnBoost/engine.io), [Socket.io](http://socket.io/), straight sockets, but here are some of the reasons we selected Faye:

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





