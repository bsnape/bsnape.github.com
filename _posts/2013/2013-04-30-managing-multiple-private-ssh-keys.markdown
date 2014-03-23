---
layout: post
title: "Managing Multiple Private SSH Keys"
---

Until recently, I wasn't really aware of the concept of having multiple private/public SSH key pairs in order to access different systems. As a developer, I've obviously used SSH fairly frequently over a number of years but it's been more to <em>get things done</em>, e.g. setting up my GitHub account or connecting to some server at work once in a blue moon.

Typically, I would have a single pair of keys which I would use in all cases. So, my public key would be held by GitHub, my Mac server, my <a href="https://code.google.com/p/gerrit/" title="Gerrit" target="_blank">Gerrit</a> server, Jenkins, TeamCity and so on. If I lost my private key it wouldn't be a terrible loss - most of the systems I use are only accessible on the company intranet and are easily changed. I now know this is not the most secure setup (hey, I'm no sysadmin) but I also know that I'm not the only person doing this!

So what happens when we want to SSH onto a machine using a different key pair?

Manually managing multiple private keys
---------------------------------------

Let's assume you've <a href="https://help.github.com/articles/generating-ssh-keys" title="ssh-keygen" target="_blank">already set up new key pairs</a> in your <code>~/.ssh</code> directory. If you <code>ls</code> in that directory, you might see something like this:

{% highlight bash %}
$ ls ~/.ssh/

jenkins  jenkins.pub  github  github.pub  known_hosts
{% endhighlight %}

When SSH'ing onto a server - e.g. Jenkins - you would usually just type <code>$ ssh user@host</code> and be done with it (assuming you have the default <code>id_rsa</code> and <code>id_rsa.pub</code> keys in your <code>~/.ssh/</code> folder).

Because we now have separate keys for each server, we must specify the location of the corresponding private key (in your <code>~/.ssh/</code> directory) when we attempt to SSH onto the server with the <code>-i</code> flag:

{% highlight bash %}
$ ssh -i ~/.ssh/jenkins ben.snape@jenkins.itv.com
{% endhighlight %}

This does the trick but it's very... <strong>wordy</strong>. Surely we can shorten it?

Introducing SSH config
----------------------

Fortunately, there is a simple way to do this.

We can define a <code>config</code> in our SSH directory. The format looks like this:

{% highlight bash %}
Host           short-name
HostName       some.very.long.server.name
IdentityFile   ~/.ssh/private_ssh_key
User           username-on-remote-server
{% endhighlight %}

You can specify as many hosts as you like. Also, you may not need the <code>User</code> field depending on your use-case.

Let's fill this out with my example Jenkins login from above and test it out:

{% highlight bash %}
Host           jenkins
HostName       jenkins.itv.com
IdentityFile   ~/.ssh/jenkins
User           ben.snape
{% endhighlight %}

Now we can really easily connect by typing:

{% highlight bash %}
$ ssh jenkins
{% endhighlight %}

Much better!
