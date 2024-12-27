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

### Fireprox

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

{% include image.html url="/images/pwspraying/bambda_editor.png" description="Default bambda editor UI. It includes suggestions, error output, and a test editor." percentage="50" %}

Bambda rules can be for requests or responses. They must return a `requestResponse.request()` or `requestResponse.response()` object depending on the type of rule. The base rule returns an unmodified request or response object. The following request bambda will fix the issue where not all resources are loaded by prepending all requests to the API Gateway URL with `/fireprox`.

{% highlight java %}
if (requestResponse.request().headerValue("Host").equals("zrn9b63jch.execute-api.us-east-1.amazonaws.com")) {
    return requestResponse.request().withPath("/fireprox" + requestResponse.request().path());
} else {
	return requestResponse.request();
}
{% endhighlight %}

{% include image.html url="/images/pwspraying/replace_worked.png" description="'/fireprox' prepended and all resources respond with a 200 OK" percentage="50" %}

API Gateway appends some additional headers to requests before sending them to the target URL. We can inspect those additional headers by creating another proxy for Burp Collaborator and reviewing the HTTP request data. One of those headers will be problematic for password spraying if the target application records it...

{% include image.html url="/images/pwspraying/collaborator.png" description="X-Forwarded-For HTTP header reveals source IP address of the original request (I used a VPN)" percentage="50" %}

The Fireprox README contains the solution to masking this header. As of the time of this writing, adding a custom `X-My-X-Forwarded-For` header will replace the `X-Forwarded-For` header added by API Gateway. We can amend the request bambda to add the header to the HTTP request with a random, high-entropy integer to avoid repitition.

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

### Adding rules for target site

The target site may require some custom match and replace rules to be created. Login requests are often made from JavaScript and request the target site directly. This would bypass the API Gateway and defeat the purpose of creating a proxy. To demonstrate, I created another proxy for `https://doesntexist.okta.com`. Okta portals for organizations can be accessed via the okta.com subdomain. However, Okta will serve a dummy login site for invalid subdomains which we can use for testing.

Testing the login will require using a browser with the Same Origin Policy disabled. This is because cross-origin requests will be sent to the target URL from the API Gateway subdomain, which is usually blocked by browsers (unless the target site has an overly permissive CORS policy). To launch Chrome with these security protections disabled for testing, you can start it with this command: `C:\Program Files\Google\Chrome\Application\chrome.exe --disable-web-security --user-data-dir=%HOMEPATH%\.chrome-data`. Adjust as necessary for Linux or Mac.
{:.info}

