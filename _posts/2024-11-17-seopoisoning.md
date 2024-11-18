---
title: What is SEO Poisoning? (MITRE ATT&CK TTPs)
category: MITRE-ATT&CK
---

#### Tactic: Resource Development
#### Technique: Stage Capabilities
## Purpose

To increase the number of successful [Drive-by Compromise](https://attack.mitre.org/techniques/T1189) / Watering hole attacks from a malicious site by exploiting search engine ranking mechanisms and boosting the site in search results.

## Scenario

Imagine a malicious domain "www.nomalwarehere.com". On it there is a sketchy looking landing page telling you to download some awesome application. And the application is actually a trojan, for example a cryptominer or a botnet client. While this is a trivial example, adversaries have created more convincing sites such as the forum web page used to spread the [GootLoader](https://thedfirreport.com/2022/05/09/seo-poisoning-a-gootloader-story/) malware.

## Methods

#### Link-building:

A website is boosted in search engine rankings when more websites link to that site. Link-building is a legitimate SEO technique to increase the number of links to your website from other sources.

Threat actors can engage in malicious link-building by:
- Placing the link in compromised sites
- Spamming the link in forums

This technique is difficult to accomplish at a large enough scale to really boost the site's rankings. Link-building is much more effective when the site is linked from websites with a high reputation. Higher reputation sites are, in-turn, generally more difficult to compromise.

#### Keyword Stuffing:

Another way the adversary can use SEO poisoning to increase traffic to "www.nomalwarehere.com" is using Keyword Stuffing.

Hackers often exploit people's attention on what's happening in the present moment. In our example, lets pretend it's April - tax season. A popular search during tax season may be "filed taxes late small business".

The adversary will start by running this search, then extract the text at the beginning of each search result.

![taxessearch](/images/seopoisoning/taxessearch.png)

They will then generate a bogus HTML page containing the search query and all the extracted keywords.

![bogustaxespage](/images/seopoisoning/bogustaxespage.png)

The Google bot will crawl this page when it is placed online and the SEO algorithms will boost it in the results for searches related to filing taxes/small businesses/etc.

Now, this page is not the malicious site that lures victims into downloading the malware. It needs to redirect victims to the "www.nomalwarehere.com" site. The problem with placing the keywords on the same page as the malware is that Google may detect that the page is spam and not index it at all. 

Attackers can implement the redirect two different ways:
- With cloaking:
    
    In this case, the browser will be sent a redirect response to www.nomalwarehere.com from the keyword-stuffed page if the client HTTP request has a common browser/real person User-Agent string, for example:

    `Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/118.0.0.0 Safari/537.36`

    If the User-Agent string is the GoogleBot or other search engine crawler, no redirect will be sent, avoiding detection.

    `Mozilla/5.0 AppleWebKit/537.36 (KHTML, like Gecko; compatible; Googlebot/2.1; +http://www.google.com/bot.html) Chrome/118.0.0.0 Safari/537.36` 
- Without cloaking:
    
    In this case, the keyword-stuffed page will attempt to detect if the client is a real browser by checking for mouse movement using JavaScript or detecting browser rendering of the page. Unlike browsers, crawlers do not render websites and run all the front-end code they include (CSS, JavaScript). Nor do they simulate mouse movement. 

    Setting the `background-image` CSS field to `url()` will trigger a reload of the page again.

    The second request from the client will include the `referer` HTTP header, which will be set to the keyword-stuffed site's domain.

    If this header is detected, it means the client is a browser that attempted to render the page and is likely a real user. The second response will then include the JavaScript code `window.location = "http://www.nomalwarehere.com"`, triggering a redirect to the malicious site.
    
    It's worth noting that in recent years Google's bot does run JavaScript on crawled sites but it doesn't happen immediately like in a browser. Additionally, it's unclear if the Google crawler processes CSS files.
    
## Defensive Considerations

From an enterprise perspective, the only consideration for this activity is to reinforce existing mitigations for [Drive-by Compromise](https://attack.mitre.org/techniques/T1189/) against end-users. Detection and prevention of this staging technique depends on the search engines which disseminate these malicious pages. It's possible that soon search engines will develop more advanced crawling techniques, perhaps using AI, which will be able to detect more complex SEO poisoning attempts. The demand for this is unclear since it is difficult to measure the actual extent to which this resource development technique is utilized by attackers.

## References

- [MITRE ATT&CK - Stage Capabilities: SEO Poisoning](https://attack.mitre.org/techniques/T1608/006/)
- [MalwareBytes - SEO poisoning: Is it worth it?](https://www.malwarebytes.com/blog/news/2018/05/seo-poisoning-is-it-worth-it)
- [Zscaler - Ubiquitous SEO Poisoning URLs](https://www.zscaler.com/blogs/security-research/ubiquitous-seo-poisoning-urls-0)
- [Google Search Central - Understand the JavaScript SEO Basics](https://developers.google.com/search/docs/crawling-indexing/javascript/javascript-seo-basics)