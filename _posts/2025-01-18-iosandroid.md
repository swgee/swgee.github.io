---
title: "Risk Analysis of the Major Mobile Platforms"
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

Smartphones constitute arguably the greatest digital attack surface in modern society: over half the global population and most people in the developed world own one. Mobile applications are heavily relied on for everything we do, be it communication, banking, media, navigation, photos, etc. The average person spends four to five hours a day using their phone<sup>1</sup>. They are a part of us. We are essentially cyborgs without a physical link between our mechanical and biological components (for now).

The two dominant platforms are Android and iOS. Combined, 99.5% of smart phones run either operating system - there are no other legitimate choices supported by the majority of app developers. Android dominates the global market share at about 73.5%, with iOS coming in at 26%. 

{% include image.html url="/images/iosandroid/globalstats.png" description="Global market share of mobile operating systems 2024<sup>2</sup>" percentage="80" %}

This article will explore the primary distinctions between the two platforms, and compare their attack surfaces from the perspective of an every day user. The difference between the two operating systems can be summarized as followed:

Android is like the Wild West - you have the right to do whatever you want and malware runs rampant like bandits. However, the open nature of the platform may reduce the possibility of being targeted with advanced no-click exploits. iOS is like being in a padded cell - you can't harm yourself by installing anything malicious, but you also have no rights. Also, your protection is entirely dependent on Apple (the prison guards).
{:.info}

## The Smartphone Market
iOS is the software powering iPhones, Apple's flagship personal computing device. For Americans, it may come as a surprise that iOS only owns about a quarter of the global smartphone market share, as previously stated. That's because in America, Apple controls a bit more than half of the market share. 

{% include image.html url="/images/iosandroid/usstats.png" description="United States market share of mobile operating systems 2024<sup>3</sup>" percentage="80" %}

And because only one company produces the phones and software, it may appear that iPhones are by and large the more dominant product - an illusion created by their uniformity. That being said, iOS is the largest smartphone manufacturer in the world, but again, only controls a quarter of the total market share. Unlike iOS, the market for Android phones is a fragmented ecosystem with many device manufacturers. As of January 2025, there are 58 companies recognized by Google as official Android device manufacturers listed on the [Android Enterprise](https://androidenterprisepartners.withgoogle.com/device-manufacturers/) website. Of those, the four leading manufacturers worldwide are Samsung, Xiaomi, Vivo, and OPPO as of January 2025. 

<div style="display:flex;flex-direction:column;align-items:center;">
<div class="appbrain-chart" style="width:100%;height:400px;background-color:white;">
  <div style="font-size: smaller; background-color:black;text-align:center;">
  <a href="https://www.appbrain.com/stats/top-manufacturers">AppBrain - Android phone manufacturer market share</a>
  </div>
</div>
</div>
<script type="module" src="https://www.appbrain.com/widgets/stats.js"></script>

In the United States, Samsung, followed by Google Pixel, lead the market. Google Pixel is the only device that uses the stock Android OS since Google also maintains the Android Open Source Project (AOSP). Most other OEMs add additional features and modifications to the Android OS. Some manufacturers change Android so much they are marketed as brand new - like Xiaomi's HyperOS, Vivo's Funtouch OS, and Oppo's ColorOS.

{% include image.html url="/images/iosandroid/hyperos.png" description="Xiaomi HyperOS - AKA iOS on Android" percentage="80" %}

Some organizations build devices running Android for purposes other than personal phones, such as Honeywell, which has an entire line of ["Mobile Handheld Computers"](https://automation.honeywell.com/us/en/products/productivity-solutions/mobile-computers/handheld-computers) built for industrial use cases. The diversity in the Android device marketplace leads into the topic of software update consistency and timeliness, an important factor to consider when evaluating the security posture of mobile operating systems.

## Security Patching

iOS is a monolithic architecture, meaning every major component is built by Apple across both the software and hardware. The operating system is composed of the following layers starting from the lowest level:

1. Core OS: Kernel, bluetooth/accessory integration, cryptographic operations, CPU optimization
2. Core Services: Includes additional basic functionality like cloud data transfer, location services, file system access
3. Media: Handles graphics and audio rendering and processing
4. Cocoa Touch: API to create native iOS apps by providing access to hardware and software features like user interface elements or the camera

Because Apple controls the entire technology stack of their products, their update process is very streamlined. Since iOS 16.4.1, Apple also implemented [Rapid Security Responses](https://support.apple.com/en-us/102657) to quietly push critical bug fixes in between standard iOS updates. The full list of security updates for all Apple products can be seen on a single web page on the Apple website. 

{% include image.html url="/images/iosandroid/iosupdates.png" description="Beginning of the Apple security update list at time of writing<sup>7</sup>" percentage="80" %}

The security update process for Android is not as simple. The platform is composed of several layers, each managed by different entities.

{% include image.html url="/images/iosandroid/android-stack.png" description="Android platform architecture<sup>9</sup>" percentage="80" %}

Android is built on the [Linux kernel](https://github.com/torvalds/linux), which provides core system services like memory, process, and driver management. The AOSP builds on the Linux kernel to form the Android Common Kernel (ACK), which is extended to be hardware-agnostic with the additiona of Generic Kernel Image (GKI) modules. Hardware vendors can implent device drivers using the Hardware Abstraction Layer (HAL) Interface Definition Language (HIDL) so chips, cameras, microphones, etc. can be controlled by the operating system. 

{% include image.html url="/images/iosandroid/gki-arch.png" description="How the AOSP maintained kernel interops with the vendor modules implemented using the HAL<sup>10</sup>" percentage="80" %}

Applications run in the Android Runtime (ART) virtual machine, which compiles Java bytecode to native machine code and provides a layer of separation between the application and the operating system. The Android Framework defines higher-level Java APIs that allow applications to access OS features like the UI (View class), Location Services, and inter-application communication (Content Providers). 

As previously mentioned, OEMs often customize every layer of the Android stack to add their own proprietary features. For example, Samsung has developed some additional capabilities into the Android OS that they release with their devices, such as:

* One UI: A custom user interface that builds on the Android Framework that provides more modification and accessibility controls.
* Integration with Samsung devices like the S-Pen - requiring additional HAL drivers and Android Framework components.
* Samsung Knox: A security platform that secures every layer of the Android stack, similar to endpoint detection and response.
* System level apps: Samsung installs its own default apps like the Samsung Internet browser, Notes, Pay, Health, and the Galaxy Store.

Because OEMs customize stock Android so much, patch consistency is highly variable. Some OEMs release updates on a consistent basis, while others do not. Additionally, when a vulnerability is discovered in core Android components, it may take months for the vulnerability to be patched, as the update to the upstream AOSP must be modified to fit the specific vendor's implementation. The [Mainline project](https://source.android.com/docs/core/ota/modular-system) aims to address this issue. Starting with Project Treble, components of the Android framework were divided into modules that could be updated outside the normal Android releases. Mainline extended this to include critical components like Bluetooth, DNS Resolver, Media Codecs, Network Stack, PermissionController, and Wi-Fi, that are often the target of the most severe vulnerabilities. The complete list of supported Mainline modules can be seen on the Mainline page linked above.

{% include image.html url="/images/iosandroid/mainline.png" description="Mainline architecture" percentage="80" %}

In order for devices to receive Mainline updates, they must have Google Mobile Services (GMS) enabled. Updates are distributed through the Google Play Store, which requires GMS to be enabled. GMS is a requirement for Android devices to be certified under the Android Enterprise Partner Program. The program helps inform enterprise customers about the quality and reliability of their mobile fleet. The question then is, are most Android devices certified under the Android Enterprise Partner Program? Well it depends. The most popular Android device manufacturers are listed as trusted partners on the Android Enterprise website, however the requirements for device manufacturers do not appear to be as stringent as individual devices. The link to the partner program requirements for OEMs and devices is broken on the [Google developer](https://developers.google.com/android/enterprise) site as of January 2025.

{% include image.html url="/images/iosandroid/developer-site.png" description="Trying to find the Android Enterprise Partner Program requirements on the Google developer site" percentage="80" %}

On the Android Enterprise website, the Glossary page has a large list of technical requirements for devices to be certified, including the requirement for GMS.

{% include image.html url="/images/iosandroid/device-requirements.png" description="Device requirements page<sup>12</sup>" percentage="80" %}

However, the device manufacturers page is bare.

{% include image.html url="/images/iosandroid/oem-requirements.png" description="Where are the requirements?<sup>13</sup>" percentage="80" %}

My suspicion for this lack of transparency is that Google services are banned in China, so devices sold in that market cannot be certified. However, some Chinese manufacturers still want to participate in other markets, so the devices they sell in other countries do have GMS enabled and receive critical updates. As a result, manufacturers that produce devices that pass the certification process are considered an enterprise partners.

In general, most Android devices do receive timely critical patches thanks to the Mainline project, however devices sold in certain countries that restrict access to Google services are entirely dependent on the OEM to release updates (and are likely subject to backdoors placed by the OEM required by those governments).

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

[1. Time Spent Using Smartphones 2024](https://explodingtopics.com/blog/smartphone-usage-stats) \
[2. Mobile Operating System Market Share Worldwide](https://gs.statcounter.com/os-market-share/mobile/worldwide) \
[3. Mobile Operating System Market Share in the United States](https://gs.statcounter.com/os-market-share/mobile/united-states-of-america) \
[4. Android Statistics - Top manufacturers](https://www.appbrain.com/stats/top-manufacturers) \
[6. Architecture of iOS Operating System](https://www.geeksforgeeks.org/architecture-of-ios-operating-system/) \
[7. Architecture of the iOS](https://www.oreilly.com/library/view/beginning-ios-5/9781118144251/ch001-sec009.html) \
[5. Apple security releases](https://support.apple.com/en-us/100100) \
[8. Android Architecture Overview](https://source.android.com/docs/core/architecture) \
[9. Android Platform Architecture](https://developer.android.com/guide/platform) \
[10. Android Kernel Overview](https://source.android.com/docs/core/architecture/kernel) \
[11. Everything you need to know about Android's Project Mainline](https://r1.community.samsung.com/t5/others/everything-you-need-to-know-about-android-s-project-mainline/td-p/7187163) \
[12. Android Enterprise Device Glossary](https://androidenterprisepartners.withgoogle.com/glossary/device/) \
[13. Android Enterprise OEM Glossary](https://androidenterprisepartners.withgoogle.com/glossary/device-manufactures/) \
[14. GMS certification: A complete guide from requirements to submission](https://emteria.com/learn/google-mobile-services) \
[15. Is Android Enterprise supported in China?](https://bayton.org/android/android-enterprise-faq/is-android-enterprise-supported-in-china/) \