---
layout: post
title: Jenkins JScript Compilation Error Fix
---

In my Jenkins distributed build system I have a Windows slave that’s started via the recommended Java Web Start (JNLP).

Every time I restarted the Jenkins master or the Windows slave (i.e. broke the master/slave connection) I would get a couple of Microsoft JScript compilation errors:

![JScript Compilation Error]({{ site.url }}/images/jscript-compilation-error.jpeg)

This is a problem because the Jenkins slave will not connect until the errors are dismissed. Obviously this is a major problem in a headless environment.

A similar bug was raised here: http://issues.hudson-ci.org/browse/HUDSON-7819 but no resolution was given.

### The quick fix

After annoying me for a while I finally managed to suppress the errors.

To make the errors go away, go to Control Panel > Internet Options > Advanced tab > then click the two ‘disable script debugging’ checkboxes and untick ‘display a notification about every script error’:

![Disable Debugging]({{ site.url }}/images/disable-debugging.jpeg)

### The long-term solution

Instead of using JNLP I’d recommend master-slave communication using SSH.

This can be slightly tricky to set up using Windows (requires Cygwin and an SSH server), but there are plenty of guides out there.
