---
layout: post
title: "Coloured Output in Jenkins"
---

Everyone knows that looking at the Jenkins logs to figure out why a build failed can be tedious work.

You're usually presented with a wall of text which you need to wade through.

![Horrible Jenkins log]({{ site.url }}/images/jenkins-coloured-output/1.png)

Builds that look like this are difficult for members in a cross-functional team to investigate,
which ultimately slows everyone down.

Fortunately there are number of improvements we can make for both Ruby and other languages.

### Language-agnostic colours

The first step is to install the [Jenkins AnsiColor plugin](https://wiki.jenkins-ci
.org/display/JENKINS/AnsiColor+Plugin).

Once you've done that, make sure the xterm colours are present in the Jenkins configuration screen.

![AnsiColor Setup]({{ site.url }}/images/jenkins-coloured-output/2.png)

You have to enable colours for each job that you want to see colours for.

![AnsiColor job]({{ site.url }}/images/jenkins-coloured-output/3.png)

You should now be running with basic colour output. Great.

### RSpec colours

To get RSpec colours working in Jenkins you have to specify the following in the RSpec configuration block:

{% highlight ruby %}
RSpec.configure do |config|
  config.tty = true
end
{% endhighlight %}

This is because the Jenkins shell is a pseudo TTY.

Looking good.

![Rspec Colours]({{ site.url }}/images/jenkins-coloured-output/4.png)

### Ruby colours (colorize gem)

If you want to get some really sweet colour action going on like this:

![Ruby colours]({{ site.url }}/images/jenkins-coloured-output/5.png)

You have to do a couple of things.

First, add the `colorize` gem to your Gemfile.

{% highlight ruby %}
source 'https://rubygems.org'

gem 'colorize'
{% endhighlight %}

Next, you need to monkey-patch the Ruby `IO` class (yeah, I know). There's some discussion on this
[here](https://github.com/dblock/jenkins-ansicolor-plugin/issues/5).

{% highlight ruby %}
class IO

  def isatty
    true
  end

end
{% endhighlight %}

Now you just need to use `colorize` in your Ruby code...

{% highlight ruby %}
puts "hello there".colorize(:red)
{% endhighlight %}

### Picking the colours to use

Here's a really handy tip if you want to experiment with which colours work best for you.

Create a Jenkins job with the following:

{% highlight bash %}
ruby -r rubygems -e "class IO; def isatty; true; end; end;" -e "require 'colorize'" -e "String.color_matrix('FOO')"
{% endhighlight %}

Now run it...et voila!

![Colorize matrix]({{ site.url }}/images/jenkins-coloured-output/6.png)

Finally, if you're feeling really helpful you can provide your team members with a glossary of what each colour means
in your build:

![Colour glossary]({{ site.url }}/images/jenkins-coloured-output/7.png)
