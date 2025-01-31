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

Smartphones constitute arguably the greatest digital attack surface in modern society: over half the global population and most people in the developed world own one. Mobile applications are heavily relied on for everything we do, be it communication, banking, media, navigation, photos, etc. The average person spends four to five hours a day using their phone<sup>1</sup>. They are a part of us. We are essentially cyborgs without a physical link between our mechanical and biological components (for now).

The two dominant platforms are Android and iOS. Combined, 99.5% of smart phones run either operating system - there are no other legitimate choices supported by the majority of app developers. Android dominates the global market share at about 73.5%, with iOS coming in at 26%. 

{% include image.html url="/images/iosandroid/globalstats.png" description="Global market share of mobile operating systems 2024<sup>2</sup>" percentage="80" %}

This article will explore the primary distinctions between the two platforms, and compare their native attack surfaces. Vulnerabilities in applications developed for each platform will not be discussed, as this is a separate topic that has been thoroughly covered by the [OWASP Mobile Application Security Verification Standard (MASVS)](https://mas.owasp.org/MASVS/) and other resources. 

The difference between the two operating systems can be summarized as followed:

Android is like the Wild West - you have the right to do whatever you want and malware runs rampant like bandits. However, the open nature of the platform may reduce the possibility of being targeted with advanced no-click exploits. iOS is like being in a padded cell - you can't harm yourself by installing anything malicious, but you also have no rights. Also, your protection is entirely dependent on Apple (the prison guards).
{:.info}

## The Smartphone Market
iOS is the software powering iPhones, Apple's flagship personal computing device. For Americans, it may come as a surprise that iOS only owns about a quarter of the global smartphone market share, as previously stated. In America, Apple controls a bit more than half of the market share. 

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

In the United States, Samsung, followed by Google Pixel, lead the market. Google Pixel is the only device that uses stock Android OS since Google also maintains the Android Open Source Project (AOSP). Most other OEMs add additional features and modifications to the Android OS. Some manufacturers change Android so much they are marketed as brand new - like Xiaomi's HyperOS, Vivo's Funtouch OS, and Oppo's ColorOS.

{% include image.html url="/images/iosandroid/hyperos.png" description="Xiaomi HyperOS - AKA iOS on Android" percentage="80" %}

Some organizations build devices running Android for purposes other than personal phones, such as Honeywell, which has an entire line of ["Mobile Handheld Computers"](https://automation.honeywell.com/us/en/products/productivity-solutions/mobile-computers/handheld-computers) built for industrial use cases. The diversity in the Android device marketplace leads into the topic of software update consistency and timeliness, an important factor to consider when evaluating the security posture of mobile operating systems.

## Security Patching

#### iOS Architecture

iOS is a monolithic architecture, meaning every major component is built by Apple across both the software and hardware. The operating system is composed of the following layers starting from the lowest level:

1. Core OS: Kernel, bluetooth/accessory integration, cryptographic operations, CPU optimization
2. Core Services: Includes additional basic functionality like cloud data transfer, location services, file system access
3. Media: Handles graphics and audio rendering and processing
4. Cocoa Touch: API to create native iOS apps by providing access to hardware and software features like user interface elements or the camera

Because Apple controls the entire technology stack of their products, their update process is very streamlined. Since iOS 16.4.1, Apple also implemented [Rapid Security Responses](https://support.apple.com/en-us/102657) to quietly push critical bug fixes in between standard iOS updates. The full list of security updates for all Apple products can be seen on a single web page on the Apple website.

{% include image.html url="/images/iosandroid/iosupdates.png" description="Beginning of the Apple security update list at time of writing<sup>7</sup>" percentage="80" %}

#### Android Architecture

The security update process for Android is not as simple. The platform is composed of several layers, each managed by different entities.

{% include image.html url="/images/iosandroid/android-stack.png" description="Android platform architecture<sup>9</sup>" percentage="80" %}

Android is built on the [Linux kernel](https://github.com/torvalds/linux), which provides core system services like memory, process, and driver management. The AOSP builds on the Linux kernel to form the Android Common Kernel (ACK), which is extended to be hardware-agnostic with the additiona of Generic Kernel Image (GKI) modules. Hardware vendors can implent device drivers using the Hardware Abstraction Layer (HAL) Interface Definition Language (HIDL) so chips, cameras, microphones, etc. can be controlled by the operating system. 

{% include image.html url="/images/iosandroid/gki-arch.png" description="How the AOSP maintained kernel interops with the vendor modules implemented using the HAL<sup>10</sup>" percentage="80" %}

Applications run in the Android Runtime (ART) virtual machine, which compiles Java bytecode to native machine code and provides a layer of separation between the application and the operating system. The Android Framework defines higher-level Java APIs that allow applications to access OS features like the UI (View class), Location Services, and inter-application communication (Content Providers). 

#### OEM Customization

As previously mentioned, OEMs often customize every layer of the Android stack to add their own proprietary features. Tweaks may be added to ensure only a specific cellular service can be used for devices sold through the carrier, like restricting the SIM card or the bootloader to prevent users from flashing custom ROMs. The operating system is also often modified to gain a competitive advantage over other manufacturers. For example, Samsung has developed some additional capabilities into the Android OS that they release with their devices, such as:

* One UI: A custom user interface that builds on the Android Framework that provides more modification and accessibility controls.
* Integration with Samsung devices like the S-Pen - requiring additional HAL drivers and Android Framework components.
* Samsung Knox: A security platform that secures every layer of the Android stack, similar to endpoint detection and response.
* System level apps: Samsung installs its own default apps like the Samsung Internet browser, Notes, Pay, Health, and the Galaxy Store.

Because OEMs customize stock Android so much, patch consistency is highly variable. Some OEMs release updates on a predictable basis, while others do not. A 2024 [research paper on Android Security Updates](https://research.google/pubs/50-shades-of-support-a-device-centric-analysis-of-android-security-updates/) published by Google (so take it with a grain of salt) found that Samsung releases monthly updates for newer models but then begins to decrease in frequency over time. Xiaomi and Oppo release security patches quarterly more often and have shorter device support periods. And Google Pixel releases security patches every month throughout their device lifespans regardless of region.

When a vulnerability is discovered and fixed in the AOSP, it may take months for the fix to be implemented by OEMs, as it must be modified to fit the specific vendor's implementation. Cellular carriers may also need to test the updates to ensure they are compatible with their networks, contributing to the delay in releases. 

#### Project Mainline

The [Mainline project](https://source.android.com/docs/core/ota/modular-system) aims to address this issue. Starting with Project Treble, components of the Android framework were divided into modules that could be updated outside the normal Android releases. Mainline extended this to include critical components like Bluetooth, DNS Resolver, Media Codecs, Network Stack, PermissionController, and Wi-Fi, that are often the target of the most severe vulnerabilities. The complete list of supported Mainline modules can be seen on the Mainline page linked above.

{% include image.html url="/images/iosandroid/mainline.png" description="Mainline architecture" percentage="80" %}

In order for devices to receive Mainline updates, they must have Google Mobile Services (GMS) enabled. Updates are distributed through the Google Play Store, which requires GMS to be enabled. GMS is a requirement for Android devices to be certified under the Android Enterprise Partner Program. The program informs enterprise customers about the quality and reliability of devices to include in their mobile inventory. The question then is, are most Android devices sold to consumers certified under the Android Enterprise Partner Program? Well, it depends. The most popular Android device manufacturers are listed as trusted partners on the Android Enterprise website, however, the requirements for device manufacturers do not appear to be as stringent as individual devices. The link to the partner program requirements for OEMs and devices is broken on the [Google developer](https://developers.google.com/android/enterprise) site as of January 2025.

{% include image.html url="/images/iosandroid/developer-site.png" description="Trying to find the Android Enterprise Partner Program requirements on the Google developer site" percentage="80" %}

On the Android Enterprise website, the Glossary page has a large list of technical requirements for devices to be certified, including the requirement for GMS.

{% include image.html url="/images/iosandroid/device-requirements.png" description="Device requirements page<sup>12</sup>" percentage="80" %}

However, the device manufacturers page is bare.

{% include image.html url="/images/iosandroid/oem-requirements.png" description="Where are the requirements?<sup>13</sup>" percentage="80" %}

My suspicion for this lack of transparency is that Google services are banned in China, so devices sold in that market cannot be certified. However, some Chinese manufacturers still want to participate in other markets, so the devices they sell in other countries do have GMS enabled and receive critical updates. As a result, manufacturers that produce any devices that pass the certification process are considered an enterprise partner. It's also possible some companies produce devices for specialized industries that cannot satisfy the device requirements of the Android Enterprise Partner Program, but are given partner status as manufacturers because of their reputation.

In general, most Android devices purchased by standard consumers do receive timely security patches for certain critical Android framework components thanks to Project Mainline. However, devices sold in countries that restrict access to Google services are entirely dependent on the OEM to release updates (but are likely subject to backdoors anyway, required by those governments to be placed by the OEMs). Also, according to the [Android Security Update Research Paper](https://www.ndss-symposium.org/wp-content/uploads/2024-175-paper.pdf) previously mentioned, the researchers found that Mainline updates only include the "partial security patch level (SPL)" published in the monthly Android Security Bulletins. The complete SPL includes CVEs identified in proprietary components at the hardware level of many Android phones like Qualcomm and MediaTek chips (page 3). The researchers found that only Google Pixel releases the complete SPL each month.

However, Project Mainline is not a comprehensive solution, and as we will see, many of the security bugs discovered in Android affect system components are not covered by Google Play system updates (Project Mainline).

## Vulnerability Research

So, for the most part, both platforms have an efficient method of delivering critical security patches to end-users, with some variation between manufacturers. However, the question remains - which platform is more likely to be exploited? Specifically, by the kind of advanced attacks that require limited interaction from the user or physical access to the device - the kind you can do nothing to prevent? To determine this, we can only look at historical data, and make an educated guess.

One thing is certain - both platforms, iOS and Android, are riddled with security bugs. This has nothing to do with the quality of the engineers building the platforms, they are obviously world-class. It's simply a result of the size and rate of change of the codebases. Both are extremely mature platforms that are constantly being improved, modified, and built on. Even if a product has a small vulnerability density (number of vulnerabilities per lines of code), as the codebase grows, so too does the number of bugs.

#### Research Accessibility

Apple products are completely closed source. No one has access to the iOS source code except for Apple employees. Apple does have a bug bounty program, like most major organizations at this point, and pays big bounties for flaws found in their products. They also implemented the [Security Research Device Program](https://security.apple.com/research-device/) to provide researchers with a modified device that allows access to operating system internals and allows bypasses to certain restrictions. However, this program is very restricted and has limited application windows.

Jailbreaking iOS devices is very difficult, and the latest versions rarely ever have a working jailbreak available to the public anymore. Although a jailbreak is in itself a major security flaw, it also makes it easier for researchers to find other issues. [Corellium](https://www.corellium.com/) is a virtual mobile device platform that can be used for vulnerability research on the latest iOS, but the emulators are still not the ideal environment. Corellium devices are reverse engineered versions of iOS and some software components (like iMessage and phone calls) do not work since they are technically in violation of Apple terms of use. Regardless, no one but Apple has source code access.

Conversely, Android is completely open source, save individual OEM device drivers and Google Mobile Services. The source code of the platform can be browsed on [Android Code Search](https://cs.android.com/android), and includes all levels of the OS from the kernel to the Android Runtime. Many off-the-shelf Android phones can also easily be rooted, allowing for more visibility into the internals of the device. This makes vulnerability research very accessible to anyone with the skills and drive to find bugs in Android. That being said, Android is a very mature platform so finding major bugs is very difficult and requires talented researchers. But anyone who has done product security testing with and without source code understands the significance of having source code and how much faster vulnerabilities can be identified.

{% include image.html url="/images/iosandroid/art-code.png" description="Android Runtime initialization methods<sup>16</sup>" percentage="80" %}

#### Security Bulletins

Comparing recent security bulletins from Apple and AOSP, we may be able to draw some conclusions. However, this is anecdotal evidence and by no means a comprehensive analysis of the thoroughness of vulnerability discovery across the two platforms.

The Apple security update issued on January 27th, 2025 for iOS 18.3 included 27 CVEs across different iOS components like AirPlay, WebKit, CoreAudio, and others. The previous security update for iOS was released on December 11th, 2024 for iOS 18.2 which included 36 CVEs affecting Apple software and follows the update before released on November 19th, 2024, together making up about 2 months of updates.

The January 2025 Android security bulletin includes vulnerabilities discovered during December 2024 and includes 26 CVEs affecting the most recent AOSP versions (12 through 15) specifically and not third party products. The majority of these vulnerabilities affect core Android system and framework components. Only two of them are Google Play system updates covered by Project Mainline modules, specifically the Documents UI and Permission Controller. The December 2024 bulletin includes only 6 vulnerabilities in the AOSP.

{% include image.html url="/images/iosandroid/december-bulletin.png" description="December Android security bulletin<sup>20</sup>" percentage="80" %}

Granted, two months of data is nothing compared to the entire history of these platforms. Retrieving the metrics of all vulnerabilities found in the platforms would be a tedious task (that AI agents would be useful for). However, we can learn about the general nature of the security updates. The number of disclosed CVEs does not necessarily indicate a more vulnerable platform or robust vulnerability discovery program. I would predict iOS to have far more CVEs than Android since Apple built the operating system from scratch along with many additional features to allow iPhones to more easily interoperate with other Apple products (and make it more difficult to switch to Android).

What is noticeable about the advisories are the severity of the bugs fixed in the updates. The Android security bulletins include at a minimum high severity bugs, mainly relating to privilege escalation and remote code execution. The January 2025 bulletin includes five "critical" severity issues according to the AOSP, with CVSS scores mainly in the high eights since most of the bugs are exploitable over the local IP network as opposed to wide area network delivery methods like SMS and phone calls.

{% include image.html url="/images/iosandroid/cve-2024-49747.png" description="CVE-2024-49747 - remote code execution in Android caused by an out-of-bounds write<sup>21</sup>" percentage="80" %}

The iOS security updates include bugs that are generally not as severe. The majority of them are related to unexpected app termination or denial-of-service with many requiring user interaction or physical device access, equating to mainly medium level CVSS scores. There are a few arbitrary code execution bugs, but these also require user interaction, diminishing their severity.

{% include image.html url="/images/iosandroid/cve-2024-54499.png" description="CVE-2024-54499 - arbitrary code execution in iOS using a maliciously crafted image exploiting a use-after-free bug<sup>21</sup>" percentage="80" %}

CVSS scores alone don't paint the full picture. A vulnerability may be critical according to its CVSS score, but may only be exploitable under very specific conditions and with limited reliability, reducing its actual risk. However, attacks carried out by APTs and other sophisticated threat actors often chain together several zero-day exploits to gain full control of a device. It is possible that the general severity of issues that are regularly fixed in Android compared to iOS reduces the likelihood of logical attack vectors being available to attackers targeting Android with zero or one-click exploits. Without being an APT and having an arsenal of exploits at our disposal, we can only look at previous attacks on mobile devices to ascertain which operating system is more susceptible to advanced threats. 

## Exploit History

As we will see, the Android malware ecosystem is far more virulent than iOS since there are no restrictions on the type of programs users can install on their devices, similar to PCs. However, which platform has been the subject of more advanced zero-click exploits, requiring no interaction from the user or installation of malicious applications? Both platforms have undergone significant overhaul since their inception with many new security protections and bug fixes being implemented, so going back five years is more than enough when researching these events.



One thing to note is that Android devices may also be subject to supply chain exploits from their hardware components, which vary between OEMs and increases the overall attack surface of Android device. Apple is trying to build more of the hardware components of the iPhone in-house and become increasingly self-reliant, and is planning on ditching Broadcom which provides its Bluetooth and Wi-Fi chips in 2025. This may or may not make iPhones more secure, but should be considered if someone trusts Apple's reputation more than common chipset manufacturers.

## Common Malware

__ talk about code signing requirements on ios vs android __
__ talk about different types of malware on android __

## Conclusion


## References
The inspiration for this blog post and some of its information comes from [SANS SEC575](https://www.sans.org/cyber-security-courses/ios-android-application-security-analysis-penetration-testing/): iOS and Android Application Security Analysis and Penetration Testing, taught by Jeroen Beckers.

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
[16. AndroidRuntime.cpp](https://cs.android.com/android/platform/superproject/main/+/main:frameworks/base/core/jni/AndroidRuntime.cpp) \
[17. About the security content of iOS 18.3 and iPadOS 18.3](https://support.apple.com/en-us/122066) \
[18. About the security content of iOS 18.2 and iPadOS 18.2](https://support.apple.com/en-us/121837) \
[19. Android Security Bulletin January 2025](https://source.android.com/docs/security/bulletin/2025-01-01) \
[20. Android Security Bulletin December 2024](https://source.android.com/docs/security/bulletin/2024-12-01) \
[21. NIST NVD: CVE-2024-49747 Detail](https://nvd.nist.gov/vuln/detail/CVE-2024-49747) \
[22. NIST NVD: CVE-2024-54499 Detail](https://nvd.nist.gov/vuln/detail/CVE-2024-54499) \
[23. Apple nears switch to in-house Bluetooth and Wi-Fi chip for iPhone, smart home, Bloomberg reports](https://www.reuters.com/technology/apple-nears-switch-in-house-bluetooth-wi-fi-chip-iphone-smart-home-bloomberg-2024-12-12/) \
[. Operation Triangulation: The last (hardware) mystery](https://securelist.com/operation-triangulation-the-last-hardware-mystery/111669/)