---
layout: post
title: If-Less Programming
---

While reading <a href="http://news.ycombinator.com/" title="Hacker News" target="_blank">Hacker News</a> I came across a very interesting <a href="http://alisnic.github.com/posts/ifless/" title="If-less programming" target="_blank">blog post</a> which contained a small discussion based on a <a href="http://www.youtube.com/watch?v=4F72VULWFvc&feature=channel" title="The Clean Code Talks -- Inheritance, Polymorphism, & Testing" target="_blank">Tech Talk by Google</a> about how and why you should avoid <code>if</code> statements and <code>switches</code> through polymorphism.

The basic premise is that if you are repeatedly checking if an object is a type of something or if some flag is set then you're probably violating one of the fundamental concepts of Object-Oriented Programming. Rather than introducing (complicated and/or cryptic) logic for a generic and <a href="http://en.wikipedia.org/wiki/Cohesion_(computer_science)" title="Cohesion" target="_blank">incohesive</a> class, it is advisable to implement a <a href="http://en.wikipedia.org/wiki/Open/closed_principle" title="Open/Closed Principle" target="_blank">concrete subclass of an abstract superclass</a>.

Ultimately, you should be trying to implement each <a href="http://en.wikipedia.org/wiki/Solid_(object-oriented_design)" title="SOLID" target="_blank">fundamental OOP concept (SOLID)</a> - <em>pragmatically of course</em>, as over-engineering a solution is just as poor practise.

Why Should We Do This?
----------------------

The main benefits (taken from the Tech Talk slides) are threefold:

* Functions without ifs are easier to read
* Functions without ifs are easier to test
* Polymorphic systems are easier to maintain (and extend) - again, refer to the <a href="http://en.wikipedia
.org/wiki/Open/closed_principle" title="Open/Closed Principle" target="_blank">Open/Closed Principle</a>

An Example
----------

I'm going to adapt a non-production code (i.e. test code) refactor I recently worked on.

My main responsibility is to look after ITV's (exposed) back-end systems, including <strong>Mercury</strong>, our video content playlist service. Given a production ID and the client platform (e.g. DotCom for ITVPlayer.com, Android for the ITV Android app etc.) then Mercury returns the requested piece of content. The platform part is important as Mercury handles many platform-specific things such as lower content bitrates for the mobile platforms and higher bitrates for YouView and the ITVPlayer site, for example.

So of course, it is necessary to test that platform-specific things work on the intended platforms only.

The <del>wrong way</del> fast way to complexity and technical debt
------------------------------------------------------------------

Here's a basic scenario that I've already given some background on above:

{% highlight ruby %}
  Scenario Outline: Platform bitrates
    Given I request the Mercury playlist for <platform>
    Then I get the correct bitrates for <platform>

  Examples:
    | platform |
    | DotCom   |
    | Android  |
    | Mobile   |
    | Samsung  |
    | PS3      |
    | YouView  |
{% endhighlight %}

Pretty straightforward. Now we can implement these steps quite naively:

{% highlight ruby %}
Given /^I request the Mercury playlist for (\w+)$/ do |platform|
  @savon_client = @mercury.create_client
  production = @production_helpers.get_production_from_config platform
  @response = @mercury.playlist_request(@savon_client, production, platform)
end

Then /^I get the correct bitrates for (.*)$/ do |platform|
  found_bitrates = @response.xpath('//VideoEntries/Video/MediaFiles/MediaFile').map { |node| node.attr('bitrate').to_i }

  expected_bitrates = case platform
                        when /android/i then [150000, 300000, 400000, 600000, 800000, 1200000]
                        when /samsung/i then [1200000]
                        when /youview/i then [1200000]
                        when /ps3/i then [800000]
                        when /mobile/i then [400000]
                        else [400000, 600000, 800000, 1200000]
                      end

  found_bitrates.should =~ expected_bitrates
end
{% endhighlight %}

I think it's implementations like this that give tools like Cucumber and its proponents a bad name. The step implementations are littered with conditional logic, unnecessary passing through of variables to various classes and a significant number of instance variables (compared to local variables).

Refactoring and removing the conditional logic
----------------------------------------------

A much better approach is to properly model your domain (yes, even with testing).

Platforms are objects and should be treated as such. Mercury is also an object but it should be part of platform objects in a <em>has-a</em> relationship.

Let's refactor our code from above starting with the Cucumber feature:

{% highlight ruby %}
  Scenario Outline: Platform bitrates
    Given I have a piece of <platform> catchup content
    When I request the Mercury playlist
    Then I get the correct bitrates

  Examples:
    | platform |
    | DotCom   |
    | Android  |
    | Mobile   |
    | Samsung  |
    | PS3      |
    | YouView  |
{% endhighlight %}

The plan is to have a data (platform object) instantiation pre-condition in the <em>Given</em> step, before having generic <em>When</em> and <em>Then</em> steps which will harness our new object-oriented design.

The new steps can be simply re-written (note the meta-programming in the <em>Given</em> step):

{% highlight ruby %}
Given /^I have a piece of (\w+) catchup content$/ do |platform|
  @platform = Object::const_get(platform.downcase.camelcase).new
end

When /^I request the Mercury playlist$/ do
  @platform.request_playlist
end

Then /^I get the correct bitrates$/ do
  @platform.playlist_response.bitrates.should == @platform.bitrates
end
{% endhighlight %}

Now we need our platform base class. The idea here is to define generic platform behaviour which the individual subclasses can override if required.

{% highlight ruby %}
class Platform

  attr_reader :playlist_request, :playlist_response

  attr_accessor :production

  def initialize
    ...
    @production = "#{EnvConfig['generic_production']}"
    @playlist_request = Mercury::Request.new
    @playlist_response = Mercury::Response.new
    @playlist_request.data[:request][:ProductionId] = @production
  end

  def request_playlist
    @playlist_response.response = @playlist_request.do
  end

end
{% endhighlight %}

An example platform might look something like this:

{% highlight ruby %}
class Youview < Platform

  attr_reader :bitrates

  def initialize
    super()
    @bitrates = [1200000]
    ...
  end

  def request_playlist
    @playlist_request.data[:siteInfo][:Platform] = 'YouView'
    super
  end

end
{% endhighlight %}

As a result of this new design, it is so easy to see the generic and specific behaviour for each platform. Not only that, but the test code itself is much easier to write, maintain and read. I've no doubt that any developer could come in and quickly get up and running.

I've deliberately left out the Mercury classes as they could contain some commercially sensitive information (especially the stuff around adverts). With that aside, the Mercury response class was a really important refactor as it encapsulates all the tricky xpaths and regular expression parsing of the SOAP response in one place. Again, for any platform-specific behaviour it was just a case of creating a concrete subclass of <code>Mercury::Response</code> to implement the differences.

Staying Out of Trouble
----------------------

There is always a fine line between meaningful concrete subclassing that aids understanding versus runaway subclassing and getting caught in an inheritance nightmare.

Base (abstract) classes are a Ruby anti-pattern, yet are embraced by the Java community.

Indeed, in statically typed languages there can be lots of boiler plate code which is ripe for inheritance. However, unless you're a fairly experienced developer who is deft with an IDE then it's so easy to become entangled amongst so many interfaces and implementations that you don't know which way is up (I know because I've been in that situation before).

Final Thoughts
--------------

The concept of complete if-less programming has certainly left an impression on me. Not that I didn't know that having good OO principles was desirable when designing software, I simply wasn't <em>aware</em> that there was a movement around this concept.

I think that it's easy - especially when writing non-production code - to subscribe to a 'hack away' mentality rather than properly think things through. Deadlines are often tight, testers are often lacking experience in writing clean, maintainable code and developers tasked with writing tests don't always take it up with enthusiasm.

But the fact remains that there is probably no better way of describing your system than through a solid set of BDD OO tests.
