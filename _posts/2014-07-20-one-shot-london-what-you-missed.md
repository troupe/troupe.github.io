---
layout: post
title: 'One Shot London: What you missed'
date: '2014-07-20 20:11:36'
---

Ghost written by [Andy Trevorah](https://twitter.com/andytrevorah).

# Using Node.js for Everything
![](https://avatars1.githubusercontent.com/u/354440?s=250)
<br>
[Charlie Key](https://twitter.com/zwigby) burns through his jetlag to give a talk on how in 2010 he and 2 other cofounders started messing with node. They created a full javascript multiplayer game (tic-tac-together) in node 0.6.8 which ended up having an uptime of 2 years (!). The ability to quickly develop apps and services led to the creation of [Modulus](https://modulus.io) (one of the sponsors of the event). Their growth taught them some valuable lessons:

* Let a process crash (but use `pm2` or `forever`)
* Find issues with `node-inspector`
* not everyone uses semantic versioning, you may need to be stricter with your dependencies
* keep your apps stateless
* but if you need state, use redis
* keep it simple, use npm, and write tests

# Success in Open Source
![](https://avatars0.githubusercontent.com/u/169364?s=250)
<br>
[Owen Barnes](https://twitter.com/temporalwave) starts his journey with an idea he had in 2010 - single page apps and websockets and the future, so why not combine them? He ended up creating [SocketStream](http://socketstream.com/). This was his first big opensource offering, and people loved it! Stars, pull-requests and tweets came flooding in, but something started to go wrong. When the project started, it was opinionated (stylus for styles, redis for state), but people used other tools, and so they sent pull requests adding support for them. Who could say no to that? Sadly the vision became diluted, the project became bloated, and it was no longer fun. Sure people liked it, but noone *loved* it anymore. Owen explained his 5 big lessons:

1. Know what you're building. Is it a tool or experience? Tools should do one thing well, experiences needs to be opinionated with one way to do everything. 
2. Know who you're building for.
3. Know why you're building it.
4. Know your strengths.
5. Enforce the vision!

[Paul Jensen](https://twitter.com/paulbjensen) helped to lend a hand in saying no to people and bringing the love back, but they currently have a slot open for a new visionary leader.

# Future Node
![](https://avatars0.githubusercontent.com/u/579?s=250)
<br>
[Mikeal Rogers](https://twitter.com/mikeal) (the master of ceremonies) gives and explanation of the changes that node has gone through, but from the eyes of a http proxy library. From using using the `sys` library (remember that?) all the way to a simple one liner with pipes, Mikeal shows how node has evolved to fit its users. With eyes towards the future, he then shows how promises are being drowned out by the sheer number of new libraries, and how generators are the future. With libraries like `thunkify` and `co`, we can start using these new standards while staying compatible with our past.

# "Just turn it into a node module," and other mantras Edna taught me.
![](https://avatars1.githubusercontent.com/u/5609?s=250)
<br>
[Soledad Penadés](https://twitter.com/supersole) shares her guilt that accumulates with bad projects that get reused. With help from [Edna](https://twitter.com/dnapiranha), jetlag and cats she takes us through the journey of learning the philosophy and best practices of node. She actually explains the mental blocks that most of us have seen but forgotten to tell anyone about. Things like why do we create new modules? Why do we use code from other people? Why don't we just copy and paste? These are questions that we now just shoo away, but are real stumbling points for people who are picking up node for the first time. She ends things with 5 major points:

1. Node Modularity is awesome
2. Find a mentor
3. Or hang out on the proper channels
4. Experiment!
5. Publish your code!

# Building offline data sync
![](https://avatars1.githubusercontent.com/u/1140553?s=250)
<br>
[Miroslav Bajtoš](https://twitter.com/bajtos) starts by asking the audience "Who here has built a mobile webapp?". Almost everyone raises there hands. "Who managed to make it work offline?". Everyone put their hands down. He then introduces [LoopBack](http://loopback.io), an api framework powered by node. He then shows how easy it is to create a new api, and then goes into the problems and the solutions that result in having one api that can be used online and offline.

His slides are also available [here](http://www.slideshare.net/Bajtos/2014-07offline-data-sync-in-loop-back).

# Silo free realtime applications!
![](https://avatars3.githubusercontent.com/u/271622?s=250)
<br>
[Lloyd Watkin](https://twitter.com/lloydwatkin) starts by quoting Tim Berners-Lee on open standards: "When they succeed they dramatically lower the cost of creating something". Lloyd shows the cost of relying on these silos in terms of data, freedom, and signing ourselves away. He then gives reignites passions for XMPP, the 15 year old standard. By using the [node-xmpp-*](https://github.com/node-xmpp/node-xmpp) libraries, we shows how easy we can start using XMPP, and not have to touch XML!

His slides are also available [here](http://talks.evilprofessor.co.uk/nodeconf-london-oneshot).

# Large (mostly scientific) datasets and JavaScript
![](https://avatars1.githubusercontent.com/u/39759?s=250)
<br>
[Max Ogden](https://twitter.com/maxogden) shows that despite node's lack of 64bit integer support, its native streams are great for managing large datasets. With [dat](http://dat-data.com), Max plans on making data easier to share and use. Dat's ultimate goal is to enable a vibrant ecosystem of reusable data modules by using the unix philosophy along with its small core. After some neat demos and some inspiration to help work on projects like bionode, genome.js, bio.js and torrent-mount, he announces that dat will see its first stable release soon.

# Next Level Chat Robots
![](https://avatars1.githubusercontent.com/u/584259?s=250)
<br>
[Ole Michaelis](https://twitter.com/CodeStars) talks about something close to our hearts - how awesome chat is! After giving his opinion on why chat rooms are so useful for software projects (asynchronous communication etc), he laments on how isolated some web tools are. However, by using [hubot](https://hubot.github.com), you can start bringing the web to your chat room. This helps teams share, preserve instruction history and collaborate in ways not possible before.


# Using Node-RED to help build the Internet of Things
![](https://avatars0.githubusercontent.com/u/5375409?s=250)
<br>
[Dave Conway-Jones](https://twitter.com/ceejay) finishes up the talks by showing off [node-red](http://nodered.org), a ~~tool~~ experience to make working with the internet of things easier. He demonstrates the large number of "nodes" that allow for plug and play experimentation and data manipulation. Its being used in the real world and allows for easy tweaking and makes everything easier for people who are just getting into software via the hardware scene.
