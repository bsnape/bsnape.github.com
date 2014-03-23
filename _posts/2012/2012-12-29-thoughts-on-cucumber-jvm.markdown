---
layout: post
title: Thoughts on Cucumber-JVM
---

As a long-term(ish) user of Java but a recent adopter of Ruby and Cucumber, I've been aware of Cucumber-JVM -- a rewrite of Cucumber in Java for the uninitiated -- for some time now.

But first, some background. I reacquainted myself -- and got more serious with -- Java a couple of years ago (after allowing myself to get far too rusty with it after my degree finished). Getting <a href="https://www.google.co.uk/url?sa=t&rct=j&q=&esrc=s&source=web&cd=2&cad=rja&ved=0CE0QFjAB&url=http%3A%2F%2Feducation.oracle.com%2Fpls%2Fweb_prod-plq-dad%2Fdb_pages.getpage%3Fpage_id%3D320&ei=tl7fUNHSEOrJ0QXxmoGICA&usg=AFQjCNHx4NeQYVoQIkCP1HboHjUyRI3qiA" title="OCPJCP" target="_blank">certified</a> re-taught me a lot of the fundamentals as well as properly uncovering the deep, dark depths of the language for the first time. At the time I was using Java with the most popular BDD framework at the time - <a href="http://jbehave.org/" title="JBehave" target="_blank">JBehave</a>.

With hindsight, JBehave was unwieldy, unfriendly and simply tried to do <em>too</em> much. I didn't realise this at the time as verbosity and unwieldiness are often part-and-parcel of Java (and other similar languages e.g. C#). This isn't a criticism of the language itself -- it's just how it is.

Eventually, I moved on to another company where I was exclusively using Ruby and Cucumber for the first time. Initially it was a struggle but now I fully appreciate just how powerful and flexible the language is and how strongly I identify with Ruby's mantra and raison d'être.

Which brings us to the whole point of this post.

Ruby and Cucumber
-----------------

What I love about Ruby and Cucumber is that the simplicity of each perfectly compliments the other.

There is no <a href="http://jbehave.org/reference/stable/configuration.html" title="JBehave Configuration" target="_blank">mandatory boilerplate</a>.

There aren't numerous ways to <a href="http://jbehave.org/reference/stable/dependency-injection.html" title="Dependency Injection" target="_blank">configure the execution</a>.

It just works.

My personal transition from Java to Ruby has given me food for thought. In the same way that I learnt from my own journey, perhaps Java also can learn from the leanness of Cucumber?

Java vs. Ruby
-------------

I decided to put the idea into practise.

Today I looked at implementing a basic example from the <a href="http://www.itv.com/" title="ITV" target="_blank">ITV</a> back-end test suite in Java and Cucumber-JVM. Take a look on my <a href="https://github.com/bsnape/Cucumber-JVM-Example" title="Cucumber-JVM Example" target="_blank">GitHub account</a> for the complete example.

It's difficult to directly compare the two implementations as they do differ very slightly in their aim and use of helper methods but you cannot deny how similar the two snippets look:

Java:

{% highlight java %}
    @Given("^I request the (\\w+) Most Watched Feed for (\\w+)$")
    public void I_request_the_type_Most_Watched_Feed_for_platform(String type, String platform) throws Throwable {
        mercuryRequest = new MercuryRequest();
        response = mercuryRequest.requestMostWatchedFeed(type, platform);
    }

    @Then("^I get a successful (\\w+) response with the correct (\\w+)$")
    public void I_get_a_successful_type_response_with_the_correct_platform(String type, String platform) throws Throwable {
        String text;

        switch (RequestType.valueOf(type.toUpperCase())) {
            case JSON:
                JSONObject json = mercuryRequest.stringToJson(response);
                text = JsonPath.read(json, "$.Parameters.Platform");
                assertEquals(platform, text);
                break;
            case XML:
                Document xml = mercuryRequest.stringToXml(response);
                XPathFactory xpf = XPathFactory.newInstance();
                XPath xpath = xpf.newXPath();
                text = xpath.evaluate("//Params/Param[2]/Value", xml.getDocumentElement());
                assertEquals(platform, text);
                break;
        }
    }
{% endhighlight %}

Ruby:

{% highlight ruby %}
Given /^I request the (\w+) (\w+) (.*) api$/ do |type, platform, uri|
  @uri = "#{EnvConfig['mercury_url']}/api/#{type}/#{platform}/#{uri}"
  @response = @mercury_api.get_response_from_url @uri
end

Then /^I get a successful (\w+) response with the correct (\w+)$/ do |type, platform|
  case type
    when 'xml'
      xml = @mercury_api.get_xml_from_response @response
      unless @mercury_api.value_exists_in_xml_node?(xml, "Value", platform)
        raise "could not find the correct platform value: #{platform} in the response for uri: #@uri"
      end
    when 'json'
      json = @mercury_api.parse_json_response @response
      @mercury_api.find_value_in_hash("Platform", json).should == platform
  end
end
{% endhighlight %}

Looking Ahead - The Killer Questions
------------------------------------

I'm not trying to draw any concrete conclusions but I think the next year or two could be an interesting time for BDD development with regard to Java and Ruby (and possibly other languages).

* Will Java learn from Cucumber (and Ruby's) example in terms of verbosity, clarity and maintainability?
* Will the emergence of Cucumber-JVM together with the open-source movement encourage developers/testers to focus more
 strongly on a common language -- particularly in light of Ruby's comparatively poor performance for enterprise applications versus Java?
* Will Ruby truly be able to throw off the performance shackles it wears <a href="http://news.ycombinator
.com/item?id=4857773" title="Ruby 2.0 Discussion" target="_blank">with the release of Ruby 2.0</a> and become more than a niche startup, prototyping and test automation language?
