---
layout: post
title: "Infrastructure Testing with Sensu and RSpec"
---

#### The Divide

Despite agile, cross-functional product teams there is often still a clear separation between Engineering (development and testing functions) and Operations.
Assuming a Continuous Delivery pipeline, there is strong collaboration between the three areas in local (e.g. Vagrant), test and stage environments.
However, when it comes to the production environment, Operations typically have the keys to the castle (complete with moat and drawbridge).

This isn't necessarily a bad thing - roles and responsibilities are clearly defined for good reason.
But it does help reinforce the Engineering-Operations divide of "we build it, you run it".

![The Engineering-Operations Divide]({{ site.url }}/images/infrastructure-testing-with-sensu-and-rspec/the_divide.png)


#### Shared Responsibility

A tangible way to help bridge this divide is with a set of behaviour-driven infrastructure tests.

The idea is to run a set of tests at regular intervals across all environments (especially Production) to monitor your application from a _behavioural_ perspective.
When tests fail, they raise very specific errors regarding your application. The benefits here are twofold:

1. It enables Operations' to understand the application behaviour.
2. It improves Engineering's understanding of how the environment effects the application.

Over time, this leads to increased collaboration between two teams that tend to be deeply entrenched.

Operations will be exposed to very specific alerts when tests fail which they can feed back to the Engineering team (e.g. service X has gone unhealthy because it depends on service Y which has lost database connectivity).
They can also suggest - and contribute - new tests for missed or problem areas.

The Engineering team can understand how their application works at scale in a complex environment (e.g. with load-balancing, caching, multiple instances).
This can aid troubleshooting complicated edge-cases such as message queuing, race conditions and cache invalidation which may be difficult to simulate locally.

The goals of universal logging, graphing, trending and alerting are strong drivers for both teams, helping to expose, debug and fix issues.

![Shared Responsibility]({{ site.url }}/images/infrastructure-testing-with-sensu-and-rspec/shared_responsibility.png)


#### Introducing Sensu

Let's take a look at how you can run your infrastructure tests and raise alerts for failures.
The tests don't have to be complex - you can even re-use your existing BDD tests from your Continuous Delivery pipeline!

[Sensu](http://sensuapp.org/docs/0.16/overview) is an open-source monitoring framework designed with the cloud in mind. Taken straight from the documentation:

> Sensu takes the results of “check” scripts run across many systems, and if certain conditions are met; passes their information to one or more “handlers”. Checks are used, for example, to determine if a service like Apache is up or down. Checks can also be used to collect data, such as MySQL query statistics or Rails application metrics. Handlers take actions, using result information, such as sending an email, messaging a chat room, or adding a data point to a graph.

Sensu provides an [open-source repository](https://github.com/sensu/sensu-community-plugins) which contains a large number of "checks" and "metrics" which you can use. Sensu also supports any existing [Nagios plugins](http://www.nagios.org/) that you might have.

All we need now is a Sensu "check" to run our infrastructure-tests. Luckily, [here's one I made earlier](https://github.com/sensu/sensu-community-plugins/blob/master/plugins/rspec/check-rspec.rb) for running RSpec tests.


#### Running Infrastructure Tests
I have a [working example](https://github.com/bsnape/sensu-rspec-integration) where you can see Sensu running some failing RSpec tests and consequently raising alerts.

Once you have the Vagrant VMs up and running you can see the (deprecated) Sensu dashboard with the alerts:

![Sensu Dashboard]({{ site.url }}/images/infrastructure-testing-with-sensu-and-rspec/sensu_dashboard.png)


##### Credits

Thanks to Rob Taylor for the first two images.
