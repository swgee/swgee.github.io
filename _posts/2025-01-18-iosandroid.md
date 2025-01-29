---
title: "Attack Surface Analysis of the Leading Mobile Platforms"
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

The two dominant platforms are Android and iOS. Combined, 99.5% of smart phones run either operating system - there are no other legitimate choices supported by the majority of app developers. Android dominates the global market share at about 73.5%, with iOS coming in at 26%. 

{% include image.html url="/images/iosandroid/globalstats.png" description="Global market share of mobile operating systems 2024" percentage="80" %}

This article will explore the primary distinctions between the two platforms, and compare their attack surfaces from the perspective of an every day user. The difference between the two operating systems can be summarized as followed:

Android is like the Wild West - you have the right to do whatever you want and malware runs rampant like bandits. However, the open nature of the platform may reduce the possibility of being targeted with advanced no-click exploits. iOS is like being in a padded cell - you can't harm yourself by installing anything malicious, but you also have no rights. Also, your protection is entirely dependent on Apple.
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

iOS is a monolithic architecture, meaning every major component is built by Apple across both the software and hardware. The operating system is composed of the following layers starting from the lowest level:

1. Core OS: Kernel, bluetooth/accessory integration, cryptographic operations, CPU optimization
2. Core Services: Includes additional basic functionality like cloud data transfer, location services, file system access
3. Media: Handles graphics and audio rendering and processing
4. Cocoa Touch: API to create native iOS apps by providing access to hardware and software features like user interface elements or the camera

Because Apple controls the entire technology stack of their products, their update process is very streamlined. Since iOS 16.4.1, Apple implemented [Rapid Security Responses](https://support.apple.com/en-us/102657) to push critical bug fixes in between standard iOS updates. The full list of security updates for all Apple products can be seen on a single web page on the Apple website. 

{% include image.html url="/images/iosandroid/iosupdates.png" description="Top of the Apple security update list at time of writing" percentage="80" %}

The security update process for Android is not as simple. The Android platform is composed of several layers, each managed by different entities.

___ list android architecture and who manages them, talk about different release schedules for oems ___

## Vulnerability Research

___ compare ios security updates to android___
___https://support.apple.com/en-us/121837___
___https://source.android.com/docs/security/bulletin/2025-01-01___
___ mention ios research program ___
___ discuss severity and size of security bulletins ___
___ discuss how the more vulnerabilities found the more secure the platform is ___
___ discuss how exploits chain together vulnerabilities ___

## Exploit History

___ get examples of big exploits on ios and android like pegasus, operation triangulation ___
___ mention zerodium ___

## Malware Prevalance

__ talk about code signing requirements on ios vs android __
__ talk about different types of malware on android __

## Conclusion


## References
The inspiration for this blog post and much of its information came from [SANS SEC575](https://www.sans.org/cyber-security-courses/ios-android-application-security-analysis-penetration-testing/): iOS and Android Application Security Analysis and Penetration Testing, taught by Jeroen Beckers.

[Time Spent Using Smartphones 2024](https://explodingtopics.com/blog/smartphone-usage-stats) \
[Mobile Operating System Market Share Worldwide](https://gs.statcounter.com/os-market-share/mobile/worldwide) \
[Mobile Operating System Market Share in the United States](https://gs.statcounter.com/os-market-share/mobile/united-states-of-america) \
[Android Statistics - Top manufacturers](https://www.appbrain.com/stats/top-manufacturers) \
[Apple security releases](https://support.apple.com/en-us/100100) \
[Architecture of iOS Operating System](https://www.geeksforgeeks.org/architecture-of-ios-operating-system/) \
[Architecture of the iOS](https://www.oreilly.com/library/view/beginning-ios-5/9781118144251/ch001-sec009.html) \
[Android Platform Architecture](https://developer.android.com/guide/platform) \