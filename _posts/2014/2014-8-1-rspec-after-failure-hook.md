---
layout: post
title: "RSpec After Failure Hook"
---

Inevitably, tests fail. 

For tests that run in complex applications and/or complex environments, the test failure alone often does not provide 
enough information to conclusively diagnose the underlying issue.

Therefore, it's handy to be able to intercept failed tests and provide extra debug information.

There are a couple of ways you can configure the after hook, depending on your use-case.

### Generic after failure hook

This is an easy way to provide default debug output for every test failure (e.g. a snapshot of the current state of the 
application/environment, timestamps etc.).

#### RSpec 3:

{% highlight ruby %}
Rspec.configure do |config|

  config.after(:each) do |example|
    if example.exception
      # do something
    end
  end
    
end
{% endhighlight %}

#### RSpec 2:

{% highlight ruby %}
Rspec.configure do |config|

  config.after(:each) do
    if @example.exception
      # do something
    end
  end
    
end
{% endhighlight %}


### Specific after failure hook

There are bound to be certain special-cases in your tests which are especially tricky to debug. In situations like this 
you can simply specify an example-level `after` hook in your `spec` file:

#### RSpec 3:

{% highlight ruby %}
after(:each) do |example|
  if example.exception
    # do something
  end
end
{% endhighlight %}

#### RSpec 2:

{% highlight ruby %}
after(:each) do
  if @example.exception
    # do something
  end
end
{% endhighlight %}

Now go break things!
