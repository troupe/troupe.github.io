---
layout: post
title: Open source communication with Gitter
date: '2014-05-12 16:11:00'
---

As we're approaching 5,000 public chat rooms on [Gitter](https://gitter.im), we thought we'd take a look at some of our more popular rooms and communities and bring you a few tips and best practices on how other people are using Gitter effectively.

####Different audiences, different rooms

The guys over at [Marionette.js](http://marionettejs.com/) have been using Gitter for some time now. They use two different chat rooms in order to separate conversations between core team members and general discussion and support about Marionette. (Mild disclaimer: we use Marionette ourselves and absolutely love it.)

Marionette have an [organisation](https://github.com/marionettejs) on GitHub. As Gitter provides any organisation a free private chat room, the team can discuss and co-ordinate the core of the project here by adding key contributors as members of the GitHub organisation. 

Using a public repository as the basis for a [public room](https://gitter.im/marionettejs/backbone.marionette) allows the wider community to participate in discussions and seek out support. 

This model isn't always possible. For instance if a corporate organisation owns the repository on GitHub, you may not wish to use your organisation room. Fear not, you can quite easily create free public channels on Gitter to achieve the same thing. You could, for example, create a public channel called ```Initech/ProjectX/core``` for your core team. Alternatively, you could easily create an organisation on GitHub and use that as the basis of the room if you'd prefer for the conversation to be private or invite only

For more information on channels, take a look at [this](/introducing-channels) post.

#### Rooms for topics

Here at Gitter, we also have a couple of public rooms. We use [gitterHQ/gitter](https://gitter.im/gitterHQ/gitter) for general product discussions and we've created [gitterHQ/developers](https://gitter.im/gitterHQ/developers) as a channel for developers with questions about our API.

You can add integrations into these rooms so that activity from your GitHub repositories will appear there and you can also easily reference GitHub issues in these chat rooms too by typing ```owner/repo#issueNumber```.

#### Office hours
We saw this great idea recently. [Inkling's](https://github.com/inkling) [Subliminal](https://github.com/inkling/Subliminal) project hosts "office hours" every Tuesday and Thursday at 3pm PST. You can see this reflected on their GitHub repository in the [contact](https://github.com/inkling/Subliminal#contact) section. They have also set their Gitter room topic to reflect this. This certainly helps both local and global audiences understand availability of the core project team for answering questions.

#### Talk code

If you ever pop into [Webpack's](https://github.com/webpack/webpack) room, you'll notice that this community is constantly sharing code. One of the very first features we built at Gitter was full markdown support in chat. By using ``` codepens you're able to get full syntax highlighting in your messages. 

Let us know the weird and wonderful ways you are using Gitter, as well as any other tools for collaboration on your open source projects and we'll update this blog post with your suggestions.

