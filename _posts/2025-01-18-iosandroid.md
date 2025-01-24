---
title: "Comparing the Attack Surfaces of iOS and Android"
tags: Learning
article_header:
  type: cover
  image:
    src: /images/iosandroid/cyberwest.jpg
cover: /images/iosandroid/header.jpg
---

## Introduction

Mobile security is a more specialized field in the world of Information Security. Where much of cybersecurity focuses on securing enterprise environments and applications, mobile security professionals focus on protecting individuals and their data from compromise, as individuals are the primary consumers of mobile technology. 

Of course, there are corporate-issued smartphones and internal applications, but these make up a relatively tiny fraction of the enterprise attack surface. The primary goal of mobile vulnerability researchers, malware reverse engineers, and application penetration testers is to secure the devices that so many people trust to store and process their most sensitive and personal information. 

Smartphones constitute arguably the greatest digital attack surface in modern society: over half the global population and most people in the developed world own one. Mobile applications are heavily relied on for everything we do, be it communication, banking, media, navigation, photos, etc. The average person spends four to five hours a day using their phone. They are a part of us. We are essentially cyborgs without a physical link between our mechanical and biological components (for now).

The two dominant platforms are Android and iOS. Combined, 99.5% of smart phones run either operating system - and there are no other legitimate choices supported by the majority of app developers. Android dominates the global market share at about 73.5%, with iOS coming in at 26%. 

{% include image.html url="/images/iosandroid/globalstats.png" description="Global market share of mobile operating systems 2024" percentage="80" %}

This article will explore the primary distinctions between the two platforms, focusing on their characteristics, architectures, and vulnerabilities. The difference between the two operating systems can be summarized as followed:

Android is like the Wild West - you have the right to do whatever you want and malware runs rampant like bandits. However, the open source nature of the platform reduces the prevalance of no-click zero days. iOS is like being in a padded cell - you have no rights and can't harm yourself by installing anything malicious. However, your protection is entirely dependent on Apple and zero-days are not uncommon.
{:.info}

## The Smartphone Market
iOS is the software powering iPhones, Apple's flagship personal computing device. For Americans, it may come as a surprise that iOS only owns about a quarter of the global smartphone market share, as previously stated. That's because in America, Apple controls a bit more than half of the market share. 

{% include image.html url="/images/iosandroid/usstats.png" description="United States market share of mobile operating systems 2024" percentage="80" %}

And because only one company produces the phones and software, it may appear that iPhones are by and large the more dominant product - an illusion created by their uniformity. That being said, iOS is the largest smartphone manufacturer in the world, but again, only controls a quarter of the total market share. Unlike iOS, the market for Android phones is a fragmented ecosystem with many device manufacturers. There are 58 companies recognized by Google as official Android device manufacturers listed on the [Android Enterprise](https://androidenterprisepartners.withgoogle.com/device-manufacturers/) website. Of those, the four leading manufacturers worldwide are Samsung, Xiaomi, Vivo, and OPPO as of January 2025. 

<div style="display:flex;flex-direction:column;align-items:center;">
<div class="appbrain-chart" style="width:100%;height:400px;background-color:white;">
  <div style="font-size: smaller; background-color:black;text-align:center;">
  <a href="https://www.appbrain.com/stats/top-manufacturers">AppBrain - Android phone manufacturer market share</a>
  </div>
</div>
</div>
<script type="module" src="https://www.appbrain.com/widgets/stats.js"></script>

In the United States, Google and its Pixel line trails Samsung. Of the five OEMs (original equipment manufacturer) mentioned, only Samsung and Google Pixel use the stock Android OS developed by Google. The other three mentioned are Chinese manufacturers that install their own customized versions of Android - specifically Xiaomi's HyperOS, Vivo's Funtouch OS, and Oppo's ColorOS.

{% include image.html url="/images/iosandroid/hyperos.png" description="Xiaomi HyperOS - AKA iOS on Android" percentage="80" %}

Some organizations build devices running Android for purposes other than personal phones, such as Honeywell, which has an entire line of ["Mobile Handheld Computers"](https://automation.honeywell.com/us/en/products/productivity-solutions/mobile-computers/handheld-computers) built for industrial use cases. This diversity in the Android device marketplace leads to the topic of software update consistency and timeliness, an important factor for evaluating the security posture of mobile operating systems.

## Security Patching

iOS updates are consistently released to the public, with many phones defaulting to automatically install the updates overnight when plugged in to charge. Since Apple controls the entire ecosystem, they can 

## References
[Time Spent Using Smartphones 2024](https://explodingtopics.com/blog/smartphone-usage-stats) \
[Mobile Operating System Market Share Worldwide](https://gs.statcounter.com/os-market-share/mobile/worldwide) \
[Mobile Operating System Market Share in the United States](https://gs.statcounter.com/os-market-share/mobile/united-states-of-america) \
[Android Statistics - Top manufacturers](https://www.appbrain.com/stats/top-manufacturers) \
