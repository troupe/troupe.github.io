---
layout: post
title: Monitoring node.js upstart processes
---

<a href="https://twitter.com/share" class="twitter-share-button" data-text="Monitoring #node.js #upstart processes" data-via="WeAreTroupe">Tweet</a>

# Using upstart to detect node.js app crashes

We love writing node.js applications, but like any real-world application, things  occassionally go wrong. Nine times out of ten, this is down to an unhandled exception in node.js (although we're seeing a lot less of these since we started using promises extensively).

## node.js crashes

We wanted was a way of detecting process crashes and reporting the state of the server immediately after the process terminated.

Upstart's `respawn` option immediately restarts our services on termination, but it doesn't have an easy way of executing a script when the process terminated abnormally. And although the `post-stop` script executes, its not possible to tell whether the shutdown is due to a planned service shutdown or a crash.

## Upstart hacky

Upstart can execute scripts before and after a process starts and stops. It has hooks for:

 * `pre-start`
 * `post-start`
 * `pre-stop`
 * `post-stop`

When a service is shutdown correctly, the `pre-stop` and `post-stop` scripts execute. However, when a service crashes, upstart will execute the `post-stop` script, but not the `pre-stop` script.  We can use this to detect a crash.

For each of the node upstart services we add the following stanzas to the upstart configuration:


```
#
# Crash reporting
#
pre-start exec /opt/upstart-monitor/bin/pre-start
pre-stop exec /opt/upstart-monitor/bin/pre-stop
post-stop exec /opt/upstart-monitor/bin/post-stop
```

The three scripts referenced are tiny. Here's how they look:

### pr-start:

This script creates a file when the service starts.

```
#!/bin/sh

touch /var/run/$UPSTART_JOB.shutdown
```

### pre-stop:

This script removes the file on proper shutdown.

```
#!/bin/sh

rm /var/run/$UPSTART_JOB.shutdown
```


### post-stop:

This script checks to see whether proper shutdown has
been performed and if it hasn't notifies devops.

```
#!/bin/sh

SHUTDOWN_FILE=/var/run/$UPSTART_JOB.shutdown

if [ -f $SHUTDOWN_FILE ]; then
  rm -f $SHUTDOWN_FILE

  # The service did not shutdown correctly,
  # perform your diagnostics or notify your
  # administrators over here
fi
```

Do you have a better way of doing this? Pop in to [Gitter](https://gitter.im/gitterHQ/gitter) and let us know!





About the Author
================

<img alt="Andrew Newdigate" src="http://www.gravatar.com/avatar/2644d6233d2c210258362f7f0f5138c2.png" style="float:left; padding-right: 1em">

__Andrew Newdigate__ is co-founder of Troupe, where he spends most of his days building [gitter.im](https://gitter.im), writing Node.js, Javascript apps and Objective-C. He's worked as a software engineer since 1996. Loves the outdoors, travel, photography.

<a href="https://twitter.com/suprememoocow" class="twitter-follow-button" data-show-count="false" data-lang="en">Follow @suprememoocow</a>
