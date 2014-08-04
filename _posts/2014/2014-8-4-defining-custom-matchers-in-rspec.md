---
layout: post
title: "Defining Custom Matchers in RSpec"
---

Defining custom RSpec matchers is really easy and a great way to increase the readability of your tests.
 
Using your own matcher is a much better option rather than trying to retrofit RSpec's built-in matchers to fit your
 individual use case.
 
#### A retro-fitted example

You may want to check that a HTTP response has a particular status code. Using RSpec's built-in matchers would look 
something like this:

{% highlight ruby %}
it 'should return a 200 status code' do
  # we need the block at the end to prevent non-200 status codes being raised as an error automatically
  response = RestClient.get('http://bensnape.com/missing-page') { |response, request, result| response } 
  expect(response.code).to eq 200 # or response.code.should == 200
end
{% endhighlight %}

But it would read much better with a custom `have_status_code` matcher.

{% highlight ruby %}
it 'should return a 200 status code' do
  response = RestClient.get('http://bensnape.com/missing-page') { |response, request, result| response } 
  expect(response).to have_status_code 200 # or response.should have_status_code 200
end
{% endhighlight %}

#### Defining a custom matcher

It's really easy to do this.

{% highlight ruby %}
require 'rspec/expectations'

RSpec::Matchers.define :have_status_code do |expected|
  match do |actual|
    actual.code == expected
  end
end
{% endhighlight %}

Ultimately, all we're really checking is that the status code of a HTTP request returns a certain value.

#### Providing a custom error message

We can improve our matcher further with a custom exception message. This is where the usefulness of writing your own
 matcher really comes out, as it provides an exact, bespoke error message rather than something generic like 
 `"expected false but got true"` which we've all experienced at some point.
 
Simply extend the matcher above with a `failure_message`:

{% highlight ruby %}
require 'rspec/expectations'

RSpec::Matchers.define :have_status_code do |expected|
  match do |actual|
    actual.code == expected
  end
  failure_message do |actual|
    "expected that #{actual} would have a status code of #{expected}, but got #{actual.code} instead"
  end
end
{% endhighlight %}

When this fails, the error looks like this:

{% highlight ruby %}
Failure/Error: expect(response).to have_status_code 200 # or response.should have_status_code 200
       expected that (entire page contents here!) would have a status code of 200, but got 404 instead
{% endhighlight %}

Which is useful as it adds more context to the test failure.

#### Extending our custom matcher further

Our custom matcher does the job but there are some potential problems with it.

Perhaps you are using more than one HTTP framework in your tests or - more likely - you are using the 
[Rack::Test](http://www.sinatrarb.com/testing.html) framework for unit-testing Sinatra apps as well as an HTTP framework
 such as `RestClient`, `Curb` or `HTTParty` for integration or acceptance tests for example.
 
In such cases it would be a good idea to use the same custom matcher defined above for all cases 
([DRY](http://en.wikipedia.org/wiki/Don't_repeat_yourself)). However, the APIs can differ e.g. to return the status 
code, `RestClient` uses `.code` whereas `Rack::Test` uses `.status`.
 
Let's harness the power of Ruby's metaprogramming using `respond_to?` to handle this.

{% highlight ruby %}
RSpec::Matchers.define :have_status_code do |expected|
  match do |actual|
    if actual.respond_to? :code
      actual.code == expected # RestClient
    else
      actual.status == expected # Rack::Test
    end
  end
end
{% endhighlight %}

Of course, the more HTTP frameworks you have the more complexity is introduced. 

It is probably a good idea to tighten up that `if` statement with an `elsif` for `Rack::Test` and a catch-all `else` 
that raises an `UnsupportedHTTPFrameworkException` or similar.

Let's finish up with our new `failure_message`.

{% highlight ruby %}
RSpec::Matchers.define :have_status_code do |expected|
  status_code = nil # define this here for visibility in the failure_message scope
  match do |actual|
    status_code = actual.respond_to?(:code) ? actual.code : actual.status
    status_code == expected
  end
  failure_message do |actual|
    "expected that #{actual} would have a status code of #{expected}, but got #{status_code} instead"
  end
end
{% endhighlight %}

To provide a single error message we needed to introduce the `status_code` variable and ensure it was at a scope that 
made it available to the `failure_message` block. This gave us the opportunity to use the much terser ternary operator 
and split out the fetching of the status code from the matcher comparison.

May your tests now be more readable...
