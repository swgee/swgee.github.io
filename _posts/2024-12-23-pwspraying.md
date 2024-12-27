---
title: Password Spraying with Selenium and Fireprox
tags: Tutorials
article_header:
  type: cover
  image:
    src: /images/pwspraying/header.jpg
cover: /images/pwspraying/octopus.webp
---

Password spraying is an attack technique that brute-forces a small set of predictable passwords across a set of likely valid user accounts. Many applications employ account lockouts to mitigate the threat of dictionary attacks on their authentication portals. If an attacker obtains a list of likely valid user accounts but cannot try many passwords, they may resort to password spraying, and choose a few passwords that have the highest chance of being valid across the target usernames. Staying under the lockout limit also maintains stealth. The larger the list of targets, the higher chance a user is using a weak password.

### Building the wordlists

The username list can be built from a variety of sources. If the login portal has a username enumeration vulnerability, the attacker can brute-force a large list of potential usernames to determine which are valid. Social media sites like LinkedIn can be scraped to retrieve a list of people that work for an organization, and those names applied to the company email format. Data breach dumps may also contain a list of valid usernames, email addresses, or passwords.

The attempted passwords should conform to the password policy used by the organization or application. Common passwords, keyboard patterns, popular phrases, or names are often a good choice, but it depends on the target. If a certain keyword, such as the company's name, appears in many data breaches, that may also be a solid choice for the password list as it could be reused across the organization. Capital letter requirements are usually used for the first character, 1 is usually used for the number requirements, and "!" is the most popular special character. If the authentication system does not employ a denylist of weak passwords, "Password1!" would be good choice if the site requires a minimum of 10 characters.

### Setting up Fireprox

IP-based lockout is another protection mechanism used to defend against brute-force attacks. If too many failed login attempts originate from the same IP address, that address will be blocked. Attackers would need to send their traffic from many different IP addresses to be able to conduct password spraying. One popular open source tool for IP rotation is [Fireprox](https://github.com/ustayready/fireprox). From the project's README: "FireProx leverages the AWS API Gateway to create pass-through proxies that rotate the source IP address with every request." So traffic would appear to come from many different AWS IP addresses.

To create a proxy, install Fireprox and create an AWS account. An IAM user should be created with the `AmazonAPIGatewayAdministrator` policy attached, and access keys should be generated for the user. A proxy can be created using the following command:

{% highlight bash %}
$ python3 fire.py --secret_access_key [SECRET_ACCESS_KEY] --access_key [ACCESS_KEY] --region us-east-1 
    --command create --url https://benkofman.com

Creating => https://benkofman.com...
[2024-12-24 07:28:51-05:00] (zrn9b63jch) fireprox_benkofman => 
    https://zrn9b63jch.execute-api.us-east-1.amazonaws.com/fireprox/ (https://benkofman.com)
{% endhighlight %}

Now any request sent to the proxy URL will be forwarded to the target URL, and each request will come from a different IP address.

{% include image.html url="/images/pwspraying/proxy_example.png" description="My website's HTML returned" percentage="50" %}

The `us-east-1` region's AWS API Gateway Network Border Group contains `13,312` according to this [range tracker](https://github.com/joetek/aws-ip-ranges-json/blob/master/ip-ranges-api-gateway.json). So for very large targets, multiple regions may be necessary as some IP addresses within the range may be reused.

### Burp Suite proxy rules

From the screenshot above, you can see that some of the site's static resources were not loaded. Inspecting the developer console shows 403 Forbidden errors for the CSS file and images.

{% include image.html url="/images/pwspraying/wrong_path.png" description="403 responses for any paths on the API gateway subdomain that's not '/fireprox'" percentage="50" %}

Since these resources' locations are relative to the base URL, the browser is requesting them from the API gateway subdomain used to proxy traffic to the target site. However, no proxy has been created for each of those paths, only the "/fireprox" path. While proxying traffic through Burp Suite, we can use match and replace rules to modify each request to ensure all traffic sent to the proxy reaches the correct location on the target.

Most of the following modifications can be done using standard match and replace rules, but we will implement it using Bambda match and replace rules, which were introduced as of December 19th 2024. They offer more flexibility for modifying requests in flight using the [Montoya API](https://portswigger.github.io/burp-extensions-montoya-api/javadoc/index.html), the Java interface that implements all Burp functionality that can be leveraged to build extensions.

{% include image.html url="/images/pwspraying/bambda_editor.png" description="Default bambda editor UI. It includes suggestions, error output, and a test editor." percentage="80" %}

Bambda rules can be for requests or responses. They must return a `requestResponse.request()` or `requestResponse.response()` object depending on the type of rule. The base rule returns an unmodified request or response object. The following request bambda will fix the issue where not all resources are loaded by prepending all requests to the API Gateway URL with `/fireprox`.

{% highlight java %}
if (requestResponse.request().headerValue("Host").equals("zrn9b63jch.execute-api.us-east-1.amazonaws.com")) {
    return requestResponse.request().withPath("/fireprox" + requestResponse.request().path());
} else {
	return requestResponse.request();
}
{% endhighlight %}

{% include image.html url="/images/pwspraying/replace_worked.png" description="'/fireprox' prepended and all resources respond with a 200 OK" percentage="80" %}

API Gateway appends some additional headers to requests before sending them to the target URL. We can inspect those additional headers by creating another proxy for Burp Collaborator and reviewing the HTTP request data. One of those headers will be problematic for password spraying if the target application records it...

{% include image.html url="/images/pwspraying/collaborator.png" description="X-Forwarded-For HTTP header reveals source IP address of the original request (I used a VPN)" percentage="80" %}

The Fireprox README contains the solution to masking this header. As of the time of this writing, adding a custom `X-My-X-Forwarded-For` header will replace the `X-Forwarded-For` header added by API Gateway. We can amend the request bambda to add the header to the HTTP request with a random, high-entropy integer to avoid repetition.

{% highlight java %}
if (requestResponse.request().headerValue("Host").equals("zrn9b63jch.execute-api.us-east-1.amazonaws.com")) {
    Random random = new Random();
    String random_number = String.valueOf(random.nextInt(1000000000) + 1);
    return requestResponse.request().withPath("/fireprox" + requestResponse.request().path())
        .withAddedHeader("X-My-X-Forwarded-For",random_number);
} else {
	return requestResponse.request();
}
{% endhighlight %}

### Custom rules for the target site

The target site may require additional match and replace rules to successfully proxy all requests made by the login page. A few possible scenarios that would require additional modification are:

- The target site issues a Content-Security-Policy header that the API Gateway subdomain violates
- The target site issues a redirect to another path on the site, leaving the proxy site

To demonstrate, I created another proxy for `https://doesntexist.okta.com`. Okta portals for organizations are generally accessed via their okta.com subdomain. However, Okta will serve a dummy login site for invalid subdomains which we can use for testing.

Server CORS policies will likely be violated since the proxy site is not the same origin as the target site. Although response headers can be modified to bypass CORS, we can do all testing in a Chrome instance with CORS disabled using the command `chrome --disable-web-security --user-data-dir=/tmp/chrome_testing_data`. Adjust for Windows as required.
{:.info}

After swapping the API Gateway subdomain in the request bambda, I browsed to the new proxy and attempted a test login. While the page loaded without issues, the site's Content-Security-Policy caused Chrome to block the login request.

{% include image.html url="/images/pwspraying/csp.png" description="CSP violated when doesntexist.okta.com is requested from the API Gateway URL" percentage="80" %}

To bypass this, we can create a response bambda to remove the Content-Security-Policy header from responses to requests to the API Gateway URL.

{% highlight java %}
if (requestResponse.request().headerValue("Host").equals("gg9j5bfyxd.execute-api.us-east-1.amazonaws.com")) {
	return requestResponse.response().withRemovedHeader("Content-Security-Policy");
} else {
	return requestResponse.response();
}
{% endhighlight %}

The login request is now made, and no CSP errors are thrown. However, the login request is made by JavaScript and sent directly to the target URL, bypassing the proxy site.

{% include image.html url="/images/pwspraying/unauthorized.png" description="The browser is happy but the proxy is pointless now" percentage="80" %}

We need to add another request bambda to send requests made directly to the target to the proxy instead. The below bambda creates a new HTTP request to the proxy URL instead of the target URL, copying the original request headers, body, and method. An `X-My-X-Forwarded-For` header is added and the Host header is updated.

{% highlight java %}
if (requestResponse.request().headerValue("Host").equals("doesntexist.okta.com")) {
    String new_url = "https://gg9j5bfyxd.execute-api.us-east-1.amazonaws.com/fireprox" + requestResponse.request().path();
    Random random = new Random();
    String random_number = String.valueOf(random.nextInt(1000000000) + 1;);
    return HttpRequest.httpRequestFromUrl(new_url).withUpdatedHeaders(requestResponse.request().headers())
        .withBody(requestResponse.request().body())
        .withAddedHeader("X-My-X-Forwarded-For",random_number)
        .withMethod(requestResponse.request().method())
        .withUpdatedHeader("Host","gg9j5bfyxd.execute-api.us-east-1.amazonaws.com");
} else {
    return requestResponse.request();
}
{% endhighlight %}

Now, all requests are sent through the proxy and the login request is made successfully.

{% include image.html url="/images/pwspraying/login_attempt_successful.png" description="Proxy history after all the custom rules are applied" percentage="80" %}

### Automating logins with Selenium

Once the proxy is set up and the match and replace rules are created, the login attempts can now be automated. This could be possible using somethign simple like Python Requests if the login form is basic and only the HTML needs to be parsed. However, many login forms, like single-sign-on portals such as Okta, are more complex, have multiple steps, and require JavaScript to be executed. To automate this process, many browser automation tools exist which control a browser process (called a "driver") to test website functionality at flexibly and at scale.

The ]Selenium](https://github.com/SeleniumHQ/selenium) project is one of the most popular of these tools. It is an browser automation ecosystem with support for a variety of languages. The easiest way to get up and running with Selenium is to use the [Selenium IDE](https://www.selenium.dev/selenium-ide/), a Chrome and Firefox extension that can record browser actions and output the scripts. Below is a recording of using the Selenium IDE to record the login on the Okta site and export to a Python script.

{% include image.html url="/images/pwspraying/selenium.gif" description="Easy as pie" percentage="80" %}

The unaltered, exported Python script is below:

{% highlight python %}
# Generated by Selenium IDE
import pytest
import time
import json
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.action_chains import ActionChains
from selenium.webdriver.support import expected_conditions
from selenium.webdriver.support.wait import WebDriverWait
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.desired_capabilities import DesiredCapabilities

class TestTest():
  def setup_method(self, method):
    self.driver = webdriver.Chrome()
    self.vars = {}
  
  def teardown_method(self, method):
    self.driver.quit()
  
  def test_test(self):
    self.driver.get("https://gg9j5bfyxd.execute-api.us-east-1.amazonaws.com/")
    self.driver.set_window_size(1200, 1021)
    self.driver.find_element(By.ID, "okta-signin-username").send_keys("test")
    self.driver.find_element(By.ID, "okta-signin-password").send_keys("test")
    self.driver.find_element(By.ID, "okta-signin-submit").click()
{% endhighlight %}
