---
layout: post
title: "Using Ruby Metaprogramming to Output Timestamped Kibana Logs in Your RSpec Tests"
---

At ITV, we verify the behaviour of our Scala micro-service API using core Ruby and RSpec. In order to 
[Continuously Deliver](http://puppetlabs.com/blog/continuous-delivery-vs-continuous-deployment-whats-diff) changes to
each micro-service, we test individual commits (forming a "candidate service") in isolation against its proven stable
counterparts (from prior runs of this very process).

Consequently, test failures can usually be narrowed down to a failure in an individual micro-service or in the contract
between two or more micro-services.

As we use [Logstash](http://logstash.net/)/[Kibana](http://www.elasticsearch.org/overview/kibana/) across our
environments, we can supplement test failures with a holistic view of the system when those failures occurred.
 
Of course we only want to see the Kibana logs that encompasses the failed test (and all relevant hooks), otherwise we'll
just be dealing with noise.

#### RSpec Hooks

In order to solve our problem, we need to do the following:

1. Initialise a variable with the time before the entire test suite begins.
2. Update the variable _after_ each individual test (so that all `before` test hooks are taken into account).
3. Use the variable defined in (1) if it's the very first test that fails or (2) if it's for any test afterwards.

Unfortunately, the `before(:suite)` hook - which, as the name implies, runs once before your entire suite of tests -
does not allow instance variables defined in that scope to be used in your tests.

{% highlight ruby %}
RSpec.configure do |config|

  config.before(:suite) do
    @my_instance_variable = 'foo'
  end
  
  config.before(:all) do
    expect(@my_instance_variable).to be_truthy
  end
  
end
{% endhighlight %}

Results in:

{% highlight ruby %}
Failure/Error: Unable to find matching line from backtrace
       expected: truthy value
            got: nil
{% endhighlight %}

How about a
[class variable](http://en.wikibooks.org/wiki/Ruby_Programming/Syntax/Variables_and_Constants#Class_Variables) instead?

{% highlight ruby %}
RSpec.configure do |config|

  config.before(:suite) do
    @@my_class_variable = 'bar'
  end
  
  config.before(:all) do
    expect(@@my_class_variable).to be_truthy
  end
  
end
{% endhighlight %}

This now works but will result in Ruby runtime warnings every time the class variable is accessed:

{% highlight bash %}
/Users/bensnape/git/CPS/spec/helpers/unit_spec_helper.rb:27: warning: class variable access from toplevel
{% endhighlight %}

So not the ideal solution.

#### Enter class_variable_set

Instead, using Ruby metaprogramming and class_variable_set we can modify a variable that holds a timestamp in a class
that hasn't been created yet. 

When we do need to create an instance of this class (i.e. when we get a test failure), it will be initialised with the
timestamp from the last `after` hook (or from the intial `before(:suite)` hook if the very first test fails):

{% highlight ruby %}
class GlobalDateTime
  @@start_time = Time.now

  def self.start_time
    @@start_time
  end
end

RSpec.configure do |config|

  config.before(:suite) do
    GlobalDateTime.class_variable_set(:@@start_time, Time.now)
  end
  
  config.after(:each) do |example|
    if example.exception
      date_format = '%Y-%m-%d %H:%M:%S'
      kibana_url  = 'http://logstash.cpp.o.itv.net.uk/kibana/#/dashboard/elasticsearch/All%20Microservices%20By%20Date?'
      times       = 
        {
          from: GlobalDateTime.new.start_time.strftime(date_format),
          to:   Time.now.strftime(date_format)
        }
      kibana_url << times.map { |type, timestamp| "#{type}=#{URI::encode(timestamp)}" }.join('&')
      puts "Check the server logs at: #{kibana_url}" 
    end

    GlobalDateTime.class_variable_set(:@@start_time, Time.now)
  end

end
{% endhighlight %}
