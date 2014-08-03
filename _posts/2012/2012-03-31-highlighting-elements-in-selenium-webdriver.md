---
layout: post
title: Highlighting Elements in Selenium WebDriver
---

Here's how you can highlight elements in the browser using Selenium WebDriver with Java.

![WebDriver Highlight in action]({{ site.url }}/images/webdriver-highlight.png)

If you want to skip ahead, I’ve got a working example in a [GitHub repo](https://github.com/bsnape/webdriver-highlight) that uses JBehave as the test runner. This
way, you can build the project yourself, see a “real-life” example and tinker with it.

I’d like to thank Iain for his [blog post](http://nthrbldyblg.blogspot.co.uk/2011/11/highlighting-elements-in-webdriver.html) on highlighting elements.

### Getting Started

First of all – if you are building an automation framework at your organisation – it’s sometimes a good idea to wrap certain WebDriver calls through a bespoke, lightweight common library.

* It helps enforce standards inside your organisation.
* It can reduce code.
* It increases code readability.
* Particularly difficult things to automate (e.g. hover and click, drag and drop) can be used again and again by
different project teams.
* A solid common library makes tests more stable as it is harder to introduce silly errors into your code.
* It is much much easier to update common methods with the latest API additions and best practise.

We need two methods to achieve our aim.

First, we need to create a setAttribute() method similar to the WebDriver [getAttribute()](http://selenium.googlecode.com/svn/trunk/docs/api/java/org/openqa/selenium/WebElement.html#getAttribute(java.lang.String)) method. The WebDriver API
does not have this built in because it is specifically designed to mimic the user’s interactions with the browser – and the user cannot change attribute values (through “normal” use).

The definition of this method depends on your setup. You need to cast your WebDriver instance to [JavaScriptExecutor](http://selenium.googlecode.com/svn/trunk/docs/api/java/org/openqa/selenium/JavascriptExecutor.html) for it to work.
If you use JBehave like I do then [WebDriverPage](http://jbehave.org/reference/web/stable/javadoc/web-selenium/org/jbehave/web/selenium/WebDriverPage.html) casts the WebDriver
instance to JavaScriptExecutor [for you already](http://grepcode.com/file/repo1.maven.org/maven2/org.jbehave.web/jbehave-web-selenium/3.5-beta-1/org/jbehave/web/selenium/WebDriverPage.java?av=h#112). Like so:

{% highlight java %}
public Object executeScript(String s, Object... args) {
    makeNonLazy();
    return ((JavascriptExecutor) webDriver()).executeScript(s, args);
}
{% endhighlight %}

So we can call executeScript() directly in our method without worrying about casting it first.

{% highlight java %}
  /**
   * Set an attribute in the HTML of a page.
   *
   * @param element
   *          The WebElement to modify
   * @param attributeName
   *          The attribute to modify
   * @param value
   *          The value to set
   */
  private void setAttribute(WebElement element, String attributeName, String value) {
    executeScript("arguments[0].setAttribute(arguments[1], arguments[2])", element, attributeName, value);
  }
{% endhighlight %}

If you don’t use JBehave then this should do the trick:

{% highlight java %}
  /**
   * Set an attribute in the HTML of a page.
   *
   * @param driver
   *          The WebDriver you are using
   * @param element
   *          The WebElement to modify
   * @param attributeName
   *          The attribute to modify
   * @param value
   *          The value to set
   */
  private void setAttribute(WebDriver driver, WebElement element, String attributeName, String value) {
    JavascriptExecutor js = (JavascriptExecutor) driver;
    js.executeScript("arguments[0].setAttribute(arguments[1], arguments[2])", element, attributeName, value);
  }

{% endhighlight %}

Now we just need to create a highlight method that uses setAttribute().

{% highlight java %}
  /**
   * Highlight an element in the UI.
   *
   * @param element
   *          The WebElement to highlight
   */
  private void highlight(WebElement element) {
    final int wait = 75;
    String originalStyle = element.getAttribute("style");
    try {
      setAttribute(element, "style",
          "color: yellow; border: 5px solid yellow; background-color: black;");
      Thread.sleep(wait);
      setAttribute(element, "style",
          "color: black; border: 5px solid black; background-color: yellow;");
      Thread.sleep(wait);
      setAttribute(element, "style",
          "color: yellow; border: 5px solid yellow; background-color: black;");
      Thread.sleep(wait);
      setAttribute(element, "style",
          "color: black; border: 5px solid black; background-color: yellow;");
      Thread.sleep(wait);
      setAttribute(element, "style", originalStyle);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }

{% endhighlight %}

It’s not the prettiest code but the effect is pretty cool.

You could parametrise the colours if you wanted to. I also feel that the wait time is enough to clearly notice what’s going on without slowing tests down too much.

### Using it in your code.

I have a [working example](https://github.com/bsnape/webdriver-highlight) in a GitHub repository. You just need Maven and Firefox installed.

Simply follow the running instructions to see the highlight method in action while searching for things on Amazon.
