---
layout: post
title: Manual Ruby Gem Version Management in Non-Ruby Projects
---

Another quick post from something that I worked on today.

I needed to create a Bundler-esque solution for a non-Ruby project. Often, such pragmatic decisions do not offer the cleanest approach but I required a fairly robust long-term solution.

Introduction
------------

To give some context, I am currently migrating many year's worth of builds from <a href="http://www.jetbrains.com/teamcity/" target="_blank">TeamCity</a> to <a href="http://jenkins-ci.org/" target="_blank">Jenkins</a>. I'm not going to go into details of why I recommend Jenkins but it's been one of my favourite tools for a long time (and Hudson before that).

A set of jobs that I'm migrating are non-Ruby but utilise a bespoke Ruby gem via a Rakefile. The problem is that a large number of jobs have a dependency on this gem but as they are not Ruby projects there are no gemspecs, hence no Bundler support.

Possible Approaches
-------------------

I think there are two main approaches here:

1. Introduce Bundler support (i.e. gemspecs) for each project.
2. Introduce a more defensive approach as a pre-build step in Jenkins.

The first approach is difficult due to the large number of projects across various source control systems (Git, Mercurial). Furthermore, the front-end developers who the projects belong to would prefer not to be burdened with extra local build steps (i.e. complexity with a tool outside of their core skills).

Therefore the second approach was necessary. I had a few requirements:

* Builds had to be fast and reliable in my distributed Jenkins build system.
* The builds had to work on any *nix master or slave, installing any required dependencies itself.

Implementing a solution
-----------------------

So I began with a simple Ruby class that lives at the root directory of the bespoke gem. First, I needed to know the current version directly from the gem source.

{% highlight ruby %}
def get_source_gem_version
  Bloomfedtasks.const_get(:VERSION).to_s
end
{% endhighlight %}

Where <code>Bloomfedtasks</code> is the name of my Ruby gem (and therefore the Ruby gem module).

I then needed an easy way to figure out what version of the gem was currently installed on the system (if at all). I wasn't previously aware of the <a href="http://docs.rubygems.org/read/chapter/10" target="_blank">wealth of possible RubyGems commands</a> there were but I easily found what I was looking for. I tested the command in a terminal window first:

{% highlight bash %}
$ gem query --name-matches bloomfedtasks


*** LOCAL GEMS ***

bloomfedtasks (0.3.3)
{% endhighlight %}

Great - all I needed now was a regular expression to capture the version and return it if found, otherwise return 0.0.0 (in other words, not installed).

{% highlight ruby %}
def get_installed_gem_version
  version = (`gem query --name-matches bloomfedtasks`.match /(\d+\.\d+\.\d+|\d+\.\d+)/).to_s
  version == '' ? '0.0.0' : version
end
{% endhighlight %}

A straightforward method that decides whether to update is up next:

{% highlight ruby %}
def update?
  get_source_gem_version > get_installed_gem_version ? true : false
end
{% endhighlight %}

And that's it. The complete class looks like this:

{% highlight ruby %}
class GemVersion

  require_relative 'lib/bloomfedtasks/version'

  class << self

    def get_source_gem_version
      Bloomfedtasks.const_get(:VERSION).to_s
    end

    def get_installed_gem_version
      version = (`gem query --name-matches bloomfedtasks`.match /(\d+\.\d+\.\d+|\d+\.\d+)/).to_s
      version == '' ? '0.0.0' : version
    end

    def update?
      get_source_gem_version > get_installed_gem_version ? true : false
    end

  end

  # true if it needs updating
  puts update?

end
{% endhighlight %}

Going further with a shell script
---------------------------------

The code above is useful in so far that it tells you if your installed version of the gem is the latest. It still requires you to acknowledge that your gem is out of date before needing you to manually build and install it. A simple bash script can do this for you:

{% highlight bash %}
#!/bin/bash

if [ `ruby gem_version.rb` = true ]; then
  echo "Your bloomfedtasks gem needs updating. Installing now..."
  gem build bloomfedtasks.gemspec
  gem install bloomfedtasks
else
  echo "Your bloomfedtasks gem is up to date."
fi
{% endhighlight %}

It simply executes the Ruby script we wrote earlier and builds and installs the gem if it is out of date.

Conclusion
----------

It was interesting to see how easy this was to put together. It has also sped up a large number of Jenkins builds by ~80% which is nice to see. Of course the biggest pitfall is forgetting to update the gem after making a change...but I'm willing to take that risk as it is not widely used or business critical.
