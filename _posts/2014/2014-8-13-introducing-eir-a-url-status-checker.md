---
layout: post
title: "Introducing Eir - a URL status checker"
---

In software development, I often find that the simplest tools are the most valuable. If you've worked with RabbitMQ
before you'll know what I mean (joke, Rabbit lovers).

I've been working on implementing a [microservice-based](http://martinfowler.com/articles/microservices.html) API at
[ITV](http://www.itv.com) for around 18 months now. It's been an interesting departure from the typically monolithic
systems I've worked with in the past. However, despite numerous benefits, microservices offer their own unique
challenges when it comes to testing the system as a whole.

Interactions and dependencies between each individual - or group of - microservices can be tricky to negotiate. Even
with thorough knowledge of the underlying domain, system-wide acceptance test failures could be due to any number of
reasons and can be tiresome to debug.

In such cases, the best approach is to methodically debug the system, gradually becoming more hands-on. For example,
tailing application logs, reading exceptions (if any) and inspecting access logs should come after more fundamental
checks such as are all the expected services even running and healthy?

With this in mind, I wanted a simple, no-frills, graphical URL status checker.

It turned out that no such thing existed(at least I couldn't find anything) so I created one.

#### Initial Development

[Eir](http://en.wikipedia.org/wiki/Eir), in Norse mythology is a goddess of health which I thought was appropriate.

I sat on my idea of an easy-to-use status checker for some time as I wasn't quite sure how I was going to architect it.

I've used [Sinatra](http://www.sinatrarb.com/) extensively for creating standalone system mocks and eventually I
realised that the answer had been staring me in the face all along.

I found a great `Sinatra` and `AJAX` [screencast](http://screencasts.org/episodes/ajax-website-with-sinatra-jquery)
which helped point me in the right direction.

Thus, [Eir](http://github.com/bsnape/eir) was born.

![Eir Dashboard](https://raw.githubusercontent.com/bsnape/eir/master/dashboard.png "Eir Dashboard")

#### Using it

Using it couldn't get any simpler.

If you have a Ruby project, add `gem 'eir'` to your `Gemfile`. Otherwise, run `gem install eir` in your terminal.

Create a file called `uris.yaml` with a list of endpoints and their display/short names like so:

{% highlight yaml %}
- http://www.google.co.uk : Google
- http://www.yahoo.co.uk : Yahoo
- http://www.itv.com : ITV
{% endhighlight %}

Then simply type `eir` in your terminal and navigate to `http://localhost:8700`.

That's it, but there are more detailed instructions in [GitHub](https://github.com/bsnape/eir).

#### Looking Ahead

I've had great feedback from my team so far but [there's loads more I could do](https://github.com/bsnape/eir/issues/1).

I want to hear from more people about how they'd like to use Eir, so feel free to create feature requests.
