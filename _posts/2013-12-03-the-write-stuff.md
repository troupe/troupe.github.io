---
layout: post
title: The write stuff
date: '2013-12-03 17:48:49'
---

![](http://rstvideo.com/trailer/files/2011/10/the-right-stuff1.jpg)

**Update** (17 December 2013)

We now ask for **write** access on your profile, this is GitHub's *user* OAuth scope and it allows us to get a list of your organisations and verify if you belong to an organisation when trying to join an org chat room. 

Rest assured we **do not** actually write anything to your profile. This is all an unfortunate limitation of GitHub's OAuth scoping. For more information, feel free to read the blog post below which we originally posted when we used to ask for write access to just about everything.

<hr>

There's been a lot of talk on Twitter about us asking for **write** access on GitHub.

Firstly, what are we actually doing?

We want Gitter to be a great solution for all kinds of use cases. Not only does this include chats based on public repositories, but also chats for organisations and for people to discuss private repositories too.

Behind the scenes, technically we're pulling a list of all of your organisations and repositories and giving you the ability to create or join chat rooms based on those objects. There's nothing sinister behind this, just a good user experience of being able to get chatting in any org or repo room in one click.

Strictly speaking, we don't need write access to do this. Unfortunately, GitHub doesn't have such a granular level of [OAuth scope](http://developer.github.com/v3/oauth/#scopes). The only level of scope that allows you to get a list of private organisations and repositories is *repo* which asks for write access at the same time. 

We've spent much of today today changing this so that we initially don't ask for any scope. This somewhat ruins the user experience as publishing organsiation membership is off by default. 

What this now means is that most users will sign in for the first time and not see all, or even any, of the organisations they belong to. If chat rooms exist for private repositories, they won't see these either because we don't know what repositories you belong to. 

We will now ask them either publicize your orgs or increase GitHub scope. This isn't really an ideal solution from a user experience perspective but at least we don't appear to be asking for too much info when you intially sign up.

We'll soon deploy these changes and let you know on [Twitter](http://twitter.com/gitchat) when we do.

Would much appreciate your thoughts on the matter, feel free to let us know in the comments. 










