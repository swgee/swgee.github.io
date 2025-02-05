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

Smartphones constitute arguably the greatest digital attack surface in modern society - over half the global population and most people in the developed world own one. Mobile applications are heavily relied on for everything we do, be it communication, banking, media, navigation, photos, etc. The average person spends four to five hours a day using their phone. They are a part of us. We are essentially cyborgs without a physical link between our mechanical and biological components (for now).

The two dominant platforms are Android and iOS. Combined, 99.5% of smartphones run either operating system - there are simply no other legitimate choices supported by the majority of app developers. Android dominates the global market share at about 73.5%, with iOS coming in at 26%. 

This article will explore the primary distinctions between the two platforms, and compare their native attack surfaces. Mobile security is a broad field, and subjects such as cellular network security and other adjacent systems used by smartphones will not be discussed. We will evaluate whether the device itself running either operating system is more likely to be compromised depending on the user's risk profile. Vulnerabilities in mobile applications will also not be covered, as this is a separate topic that has been thoroughly covered by the [OWASP Mobile Application Security Verification Standard (MASVS)](https://mas.owasp.org/MASVS/) and other resources, and is mainly a responsibility of application developers.

## The Smartphone Market
For Americans, it may come as a surprise that iOS only makes up about a quarter of the global smartphone market share, as previously stated. In America, Apple controls a bit more than half of the market share. And because only one company produces the phones and software, it may appear that iPhones are by and large the more dominant product. That being said, iOS is the largest smartphone manufacturer in the world, but again, only controls a quarter of the total market share. 

Unlike iOS, the market for Android phones is a fragmented ecosystem with many device manufacturers. As of January 2025, there are 58 companies recognized by Google as official Android device manufacturers listed on the [Android Enterprise](https://androidenterprisepartners.withgoogle.com/device-manufacturers/) website. Of those, the four leading manufacturers worldwide are Samsung, Xiaomi, Vivo, and Oppo as of January 2025. 

<div style="display:flex;flex-direction:column;align-items:center;">
<div class="appbrain-chart" style="width:100%;height:400px;background-color:white;">
  <div style="font-size: smaller; background-color:black;text-align:center;">
  <a href="https://www.appbrain.com/stats/top-manufacturers">AppBrain - Android phone manufacturer market share</a>
  </div>
</div>
</div>
<script type="module" src="https://www.appbrain.com/widgets/stats.js"></script>

In the United States Samsung and Google Pixel lead the Android device market. Google Pixel is the only device that uses the stock Android OS since Google also maintains the Android Open Source Project (AOSP). Most other OEMs add additional features and modifications to the Android OS. Some manufacturers change Android so much they are marketed as a brand new operating system - like Xiaomi's HyperOS, Vivo's Funtouch OS, and Oppo's ColorOS.

{% include image.html url="/images/iosandroid/hyperos.png" description="Xiaomi HyperOS - AKA iOS on Android" percentage="80" %}

Some organizations build devices running Android for purposes other than personal phones, such as Honeywell, which has an entire line of ["Mobile Handheld Computers"](https://automation.honeywell.com/us/en/products/productivity-solutions/mobile-computers/handheld-computers) made for industrial use cases. The diversity in the Android device marketplace leads into an important the risk factor of mobile operating systems - software update consistency and timeliness.

## Security Patching

#### iOS Architecture

iOS has a monolithic architecture, and most major components are built by Apple across both software and hardware. The operating system is composed of the following layers starting from the lowest level:

1. Core OS: Kernel, chip drivers, cryptographic operations
2. Core Services: Includes basic functionality like cloud data transfer, location services, file system access
3. Media: Handles graphics and audio rendering and processing
4. Cocoa Touch: API providing access to hardware and software features like user interface elements or the camera

Because Apple controls the entire tech stack of their products, their update process is very streamlined. Since iOS 16.4.1, Apple also implemented [Rapid Security Responses](https://support.apple.com/en-us/102657) to quietly push critical bug fixes in between standard iOS updates. The full list of security updates for all Apple products can be viewed on a single page on the Apple website.

{% include image.html url="/images/iosandroid/iosupdates.png" description="Beginning of the Apple security update list at time of writing" percentage="80" %}

#### Android Architecture

The security update process for Android is not as simple. The platform is composed of several layers, each managed by different entities.

{% include image.html url="/images/iosandroid/android-stack.png" description="Android platform architecture" percentage="80" %}

The AOSP builds on the Linux kernel to form the Android Common Kernel (ACK), which is extended by Generic Kernel Image (GKI) modules to be hardware-agnostic. Hardware vendors implement device drivers using the Hardware Abstraction Layer (HAL) Interface Definition Language (HIDL) so chips, cameras, microphones, etc. can be controlled by the operating system.

{% include image.html url="/images/iosandroid/gki-arch.png" description="How the AOSP maintained kernel interops with the vendor modules implemented using the HAL" percentage="80" %}

Applications run in the Android Runtime (ART) virtual machine, which compiles Java byte code to native machine code and acts as a wall between the application and operating system. The Android Framework defines higher-level Java APIs like the UI (View class), Location Services, and inter-application communication (Content Providers). 

#### OEM Customization

OEMs often customize every layer of the Android stack to add their own proprietary features. Tweaks may be added to ensure only a specific cellular carrier can be used for devices sold through the carrier, like restricting the SIM card or the bootloader to prevent users from flashing custom ROMs. The operating system is also often modified to gain a competitive advantage over other manufacturers. For example, Samsung has developed some additional capabilities into the Android OS that they release with their devices, such as:

* One UI: A custom user interface that builds on the Android Framework that provides more modification and accessibility controls.
* Integration with Samsung devices like the S-Pen - requiring additional HAL drivers and Android Framework components.
* Samsung Knox: A security platform that secures every layer of the Android stack, similar to endpoint detection and response.
* System level apps: Samsung installs its own default apps like the Samsung Internet browser, Notes, Pay, Health, and the Galaxy Store.

When a vulnerability is discovered and fixed in the AOSP, it may take months for the fix to be implemented by OEMs, as it must be modified to fit the specific vendor's implementation. Cellular carriers may also need to test the updates to ensure they are compatible with their networks, contributing to the delay in releases.

Because manufacturers customize the stock AOSP so much, patch consistency is highly variable among OEMs. A 2024 [research paper on Android Security Updates](https://research.google/pubs/50-shades-of-support-a-device-centric-analysis-of-android-security-updates/) published by Google (so take it with a grain of salt) analyzed patch consistency among the top Android device vendors. They found that Xiaomi and Oppo generally release patches quarterly rather than monthly and have shorter support periods. Samsung releases monthly updates for newer models but slows over time. And Google Pixel releases security patches every month throughout their device lifespans, regardless of region, and has the longest support periods.

#### Project Mainline

The [Mainline project](https://source.android.com/docs/core/ota/modular-system) aims to address this issue. Starting with Project Treble, components of the Android framework were divided into modules that could be updated outside normal Android updates. Mainline extended this to include critical components like Bluetooth, Media Codecs, Network Stack, Permission Controller, and Wi-Fi, which are often the target of network-based exploits. 

{% include image.html url="/images/iosandroid/mainline.png" description="Mainline architecture" percentage="80" %}

In order for devices to receive Mainline updates, they must have Google Mobile Services (GMS) enabled as updates are released through the Google Play Store. GMS allows Android devices to use Google applications like the Play Store, Gmail, YouTube, Chrome, etc. However, GMS is banned in China. Android devices sold in China are completely reliant on OEM updates to be patched... but most likely have built in backdoors anyway.

Project Mainline is not a comprehensive solution, and many of the security bugs discovered in Android affect system components are not covered by Google Play system updates. Also, the Mainline updates only include the "partial security patch level (SPL)" published in the monthly Android Security Bulletins. The complete SPL includes CVEs identified in proprietary components in hardware like Qualcomm and MediaTek chips used in many Android devices, and Google Pixel is the only vendor that releases the complete SPL each month.

## Vulnerability Research

So, iOS seems generally more reliable when it comes to timely security patches. But that doesn't answer the main question - which platform is more likely to be exploited? Specifically, by the kind of advanced attacks that require limited interaction from the user or physical access to the device - the kind a security conscious user can do nothing to prevent? To determine this, we can only look at historical data, and make an educated guess.

One thing is certain - both platforms, iOS and Android, are riddled with security bugs. This has nothing to do with the quality of the engineers building the platforms, they are obviously world-class. It's simply a result of the size and rate of change of the codebases. Both are extremely mature platforms that are constantly being improved. Even if a product has a small defect density (number of bugs per lines of code), as the codebase grows, so does the number of bugs.

#### Research Accessibility

iOS has a very high barrier to entry for security research. Apple does have a bug bounty program, like most major organizations, and pays big bounties for flaws found in their products. They have also implemented the [Security Research Device Program](https://security.apple.com/research-device/), which provides researchers with a modified device that allows privileged access to the OS. However, this program is very restricted and has limited application windows.

Jailbreaking iOS devices is very difficult, and the latest versions rarely ever have a working jailbreak available to the public. Although a jailbreak is in itself a major security flaw, it also makes it easier for researchers to find other issues. [Corellium](https://www.corellium.com/) is a virtual mobile device platform that can be used for vulnerability research on a jailbroken version of the latest iOS, but the emulators are still not the ideal environment. Corellium devices are reverse engineered versions of iOS and some features like iMessage and phone calls do not work. Finally, iOS is totally proprietary - no one has access to the iOS source code except for Apple employees.

Conversely, Android is completely open source. The only software on an Android device that isn't open source is OEM drivers (and other OEM modifications), Google Mobile Services, and applications. The source code of the platform can be browsed on [Android Code Search](https://cs.android.com/android), and includes all levels of the OS from the kernel to the Android Runtime. Many off-the-shelf Android phones can easily be rooted, allowing for more visibility into the internals of the device. This makes vulnerability research very accessible to anyone with the skills and drive to find bugs in Android. That being said, finding major bugs in Android is still very difficult. But anyone who has done product security testing with and without source code understands the significance of having source code and how much faster security issues can be identified.

{% include image.html url="/images/iosandroid/art-code.png" description="Viewing the Android Runtime initialization methods on cs.android.com" percentage="80" %}

#### Security Bulletins

Comparing recent security bulletins from Apple and AOSP, we may be able to draw some conclusions. The Apple security update issued on January 27th, 2025 for iOS 18.3 included 27 CVEs across different iOS components like AirPlay, WebKit, CoreAudio, and others. The previous security update for iOS was released on December 11th, 2024 for iOS 18.2 which included 36 CVEs affecting Apple software and follows the previous update released on November 19th, 2024, together making up about 2 months of updates.

The January 2025 Android security bulletin includes vulnerabilities discovered during December 2024 and includes 26 CVEs affecting the most recent AOSP versions (12 through 15). The majority of these affect core Android system and framework components. Only two vulnerabilities are for Project Mainline modules, specifically the Documents UI and Permission Controller. The December 2024 bulletin includes only 6 CVEs in the AOSP.

{% include image.html url="/images/iosandroid/december-bulletin.png" description="December Android security bulletin" percentage="80" %}

Granted, two months of data is nothing compared to the entire history of these platforms. However, we can learn about the general nature of the security updates. The number of disclosed CVEs does not necessarily indicate a more vulnerable platform or robust vulnerability discovery program. I would predict iOS to have far more CVEs than Android since Apple built the operating system from scratch along with many additional features to allow iPhones to more easily interoperate with other Apple products (and make it more difficult to switch to Android).

What is noticeable about the advisories are the severity of the bugs fixed in the updates. The Android security bulletins include at a minimum high severity bugs, mainly relating to privilege escalation and remote code execution. The January 2025 bulletin includes five "critical" severity issues according to the AOSP, with CVSS scores mainly in the high eights since most of the bugs are exploitable over the local IP network as opposed to wide area network delivery methods like SMS and phone calls.

{% include image.html url="/images/iosandroid/cve-2024-49747.png" description="CVE-2024-49747 - remote code execution in Android caused by an out-of-bounds write" percentage="80" %}

The iOS security updates include bugs that are generally not as severe. The majority of them are related to unexpected app termination or denial-of-service with many requiring user interaction or physical device access, equating to mainly medium level CVSS scores. There are a few arbitrary code execution bugs, but these also require user interaction, diminishing their severity.

{% include image.html url="/images/iosandroid/cve-2024-54499.png" description="CVE-2024-54499 - arbitrary code execution in iOS using a maliciously crafted image exploiting a use-after-free bug" percentage="80" %}

CVSS scores alone don't paint the full picture. A vulnerability may be critical according to its CVSS score, but may only be exploitable under very specific conditions and with limited reliability, reducing its actual risk. However, attacks carried out by APTs and other sophisticated threat actors often chain together several zero-days to gain full control of a device. It is possible that the average severity of the issues regularly fixed in Android compared to iOS reduces the likelihood of logical attack vectors being available to attackers targeting Android with zero or one-click exploits. Without being an APT and having an arsenal of zero-days at our disposal, we can only look at previous attacks on mobile devices to ascertain which operating system is more susceptible to advanced threats. 

## Exploit History

As we will see, the Android malware ecosystem is far more virulent than iOS since there are fewer limitations on which applications can be installed, similar to a PC. However, which platform has been the subject of more advanced zero-click attacks?

It's fairly well known that iOS has long been the target of Pegasus spyware developed by NSO Group. Exploits such as KISMET, FORCEDENTRY, and BLASTPASS have been giving NSO Group customers access to their targets iPhones for several years by exploiting attack vectors like iMessage, Safari links, Find My, and other native iOS features. There are countless articles about these attacks, and they don't seem to be slowing down anytime soon. [Citizen Lab](https://citizenlab.ca/) has been tracking Pegasus for years and regularly releases reports on Pegasus targets and technical details.

In 2023, Kaspersky researchers discovered a sophisticated iOS hacking campaign which they dubbed "[Operation Triangulation](https://securelist.com/operation-triangulation-the-last-hardware-mystery/111669/)" targeting mainly Russian citizens and officials. The malware, called "TriangleDB", was (again) delivered via malicious iMessage attachments, and included a series of four zero-days to achieve full device compromise.

{% include image.html url="/images/iosandroid/triangulation.png" description="Operation Triangulation attack chain diagram created by Kaspersky" percentage="80" %}

Apple has created additional security measures around iMessage such as [BlastDoor](https://support.apple.com/guide/security/blastdoor-for-messages-and-ids-secd3c881cee/web), which is vaguely described as additional sandboxing and memory protections, but time will tell if iOS continues to fall victim to more spyware strains. iOS full device compromises have also historically been cheaper than Android on the zero-day black market. While zero-day prices are obviously not as accurately measured as the S&P 500, zero-day brokers like Zerodium (which closed down in 2024) have advertised higher bounties for Android Full Compromise with Persistence (FCP) packages than iOS. 

{% include image.html url="/images/iosandroid/zerodium.webp" description="Zerodium payout table" percentage="80" %}

Android zero-click exploits appear less common in the wild than iOS, but they still exist. As we have seen, plenty of remote code execution bugs are regularly found in the AOSP, such as [CVE-2024-49415](https://thehackernews.com/2025/01/google-project-zero-researcher-uncovers.html), which can be exploited via a crafted RCS message sent on Google Messages. Occasionally, Android security bulletins will also advise whether some CVEs are actually being exploited in the wild, such as in the [August 2024 bulletin](https://source.android.com/docs/security/bulletin/2024-08-01) (although few additional details are provided).

{% include image.html url="/images/iosandroid/august.png" description="Google warning regarding CVE-2024-36971" percentage="80" %}

WhatsApp vulnerabilities like [CVE-2019-3568](https://nvd.nist.gov/vuln/detail/CVE-2019-3568) have been used to install spyware on iOS and Android. The vulnerability is a buffer overflow affecting native code in the VOIP stack of WhatsApp. It's unclear how reliable these types of application-level exploits are against Android. They most likely need pairing with other attacks to escape the Android Runtime sandbox such as type confusion, object corruption, or insecure deserialization.

One thing to note is that Android devices may also be affected by hardware supply chain vulnerabilities. Recently, Serbian activists' phones were hacked using the [Cellebrite Universal Forensic Extraction Device (UFED)](https://cellebrite.com/en/ufed/), another Israeli-developed mobile exploitation tool marketed for law enforcement and government agencies (that the company deems ethical). The UEFD gives attackers root access without needing the passcode, but requires physical access to the device. 

A Serbian journalist that was pulled over by police for allegedly driving under the influence suspected his phone was tampered with while detained and provided it to Amnesty International, a human rights NGO that also performs security research, to analyze the device. Amnesty International in coordination with Google Project Zero and Threat Analysis Group determined Cellebrite UEFD was exploiting vulnerabilities in a Qualcomm chip driver used in the journalist's Xiaomi device. The researchers found NoviSpy malware on the device, likely installed after gaining root access using Cellebrite UEFD. The Serbian government needing physical access to the activists' devices could indicate the lack of availability of remote zero-click exploit packages targeting Android. Amnesty International has since released a [guide](https://securitylab.amnesty.org/latest/2024/12/tech-guide-detecting-novispy-spyware-with-androidqf-and-the-mobile-verification-toolkit-mvt/) for detecting NoviSpy on Android devices.

## Common Malware

Nation-states likely have a harder time gaining covert remote access to a fully patched Android device than iPhones. But what about less reliable attacks like malicious applications delivered via phishing or app stores? Compared to iOS, there's no doubt Android is like the Wild West. Run-of-the-mill malware strains run rampant and if users are not carefully, their devices can easily be infected. 

#### Android Malware

For an application to be installed on Android, it must be self-signed by the developer. That's it. This is only to prevent existing apps with the same package name from being overwritten, like during application updates, not to verify the identity of the developer. For this reason, many fake versions of popular applications exist that share the same name as the legitimate version. As long as the legitimate version is not already installed, the fake one can be installed, and a user's credentials for that application may be stolen when they log in. Devices purchased from sketchy sources may also be preloaded with malware and fake versions of popular applications.

Android applications are bundled into APK files, which can be sent over any communication medium such as email, text, instant messaging, websites, etc. The APK can be installed directly from downloads. Third-party app stores also exist which do little to nothing to filter out malware and scams. There are many well-known Android malware sprains that propagate using these methods and engage in all sorts of nastiness such as infostealing, cryptomining, and banking theft. However, in addition to having to be first installed by the user, the application often needs to request additional permissions to perform its tasks. Below are the three major permissions categories along with some examples:

* **Normal**: Granted automatically and don't have any major security concern
    * Internet Access, Flashlight, Vibrate, Modify Audio Settings, etc.
* **Dangerous**: User is prompted to allow or deny
    * Divided into 15 groups such as Camera access, Microphone, Location, Contacts, etc.
* **Special**: Must be manually granted in device settings
    * BIND_DEVICE_ADMIN: Set passcode, lock device, wipe device
    * SYSTEM_ALERT_WINDOW: Draw on user's screen (automatically granted to Play Store apps)
    * BIND_ACCESSIBILITY_ADMIN: Automate keystrokes, retrieve window and text content

Google has implemented some protections to mitigate the threat of Android malware. They regularly audit applications submitted to the Play Store for malicious behavior, however often fall short. One application, iRecorder, was uploaded to the Play Store in September 2021 without any malicious functionality. The developers added the AhMyth remote access trojan (RAT) in August 2022, which went undetected until May 2023 after the App was downloaded more than 50,000 times.

Google Play Protect is a feature of Google Mobile Services that essentially acts as EDR for Android phones. It performs the malware screening for Play Store apps and can remotely scan sideloaded applications. It also helps locate lost devices and remote wipe if needed. However, like any EDR/antivirus, it is often bypassed, especially by less common payloads or new versions of existing malware and obfuscation techniques that Google has yet to create detections for.

#### iOS Malware

Unlike Android, the standard iOS malware ecosystem is very limited. This is mainly due to iOS code signing requirements. All code executed on non-jailbroken iPhones must be signed by Apple, including the operating system itself. The BootROM in the CPU contains Apple's public key, which is used to validate the signature of the bootloader, iBoot engine, and the kernel. A vulnerability in the BootROM on Apple chips before A12 allowed this signature validation to be bypassed, paving the way for unpatchable, hardware-level jailbreaks on those devices. 

{% include image.html url="/images/iosandroid/codesigning.png" description="iOS chain of trust" percentage="80" %}

That same public key is used to verify the signature of every application downloaded from the Apple App store, which is signed by Apple's private key. Apple performs very strict screening on applications submitted to the App store, and unlike Google Play, malware is short lived if it begins behaving maliciously after being accepted. Unlike Android apps, iOS apps cannot remotely download additional libraries and execute them since they will not be signed by Apple. 


## Conclusion

__ ios is better for grandma __
__ the country you live in and the people that would target you matters a lot, for example if you are a chinese citizen you can't even get access to the most secure systems, and if FVEY wants to spy on you they will be able to by working with companies to install backdoors in software like GMS __
__ no system is 100% secure and private __
__ if you don't want to use carrier pigeons, your best bet is android since it's likely more secure from advanced exploits, specifically google pixel. ideally sms, phone calls, bluetooth should be disabled so the phone is less of a server __
__security vendor cve issue __

## References
The inspiration for this blog post comes from [SANS SEC575](https://www.sans.org/cyber-security-courses/ios-android-application-security-analysis-penetration-testing/): iOS and Android Application Security Analysis and Penetration Testing, taught by Jeroen Beckers.

* [Time Spent Using Smartphones 2024](https://explodingtopics.com/blog/smartphone-usage-stats)
* [Mobile Operating System Market Share Worldwide](https://gs.statcounter.com/os-market-share/mobile/worldwide)
* [Mobile Operating System Market Share in the United States](https://gs.statcounter.com/os-market-share/mobile/united-states-of-america)
* [Android Statistics - Top manufacturers](https://www.appbrain.com/stats/top-manufacturers)
* [Architecture of iOS Operating System](https://www.geeksforgeeks.org/architecture-of-ios-operating-system/)
* [Architecture of the iOS](https://www.oreilly.com/library/view/beginning-ios-5/9781118144251/ch001-sec009.html)
* [Apple security releases](https://support.apple.com/en-us/100100)
* [Android Architecture Overview](https://source.android.com/docs/core/architecture)
* [Android Platform Architecture](https://developer.android.com/guide/platform)
* [Android Kernel Overview](https://source.android.com/docs/core/architecture/kernel)
* [Everything you need to know about Android's Project Mainline](https://r1.community.samsung.com/t5/others/everything-you-need-to-know-about-android-s-project-mainline/td-p/7187163)
* [Android Enterprise Device Glossary](https://androidenterprisepartners.withgoogle.com/glossary/device/)
* [Android Enterprise OEM Glossary](https://androidenterprisepartners.withgoogle.com/glossary/device-manufactures/)
* [Google Mobile Services](https://www.android.com/gms/)
* [GMS certification: A complete guide from requirements to submission](https://emteria.com/learn/google-mobile-services)
* [Is Android Enterprise supported in China?](https://bayton.org/android/android-enterprise-faq/is-android-enterprise-supported-in-china/)
* [AndroidRuntime.cpp](https://cs.android.com/android/platform/superproject/main/+/main:frameworks/base/core/jni/AndroidRuntime.cpp)
* [About the security content of iOS 18.3 and iPadOS 18.3](https://support.apple.com/en-us/122066)
* [About the security content of iOS 18.2 and iPadOS 18.2](https://support.apple.com/en-us/121837)
* [Android Security Bulletin January 2025](https://source.android.com/docs/security/bulletin/2025-01-01)
* [Android Security Bulletin December 2024](https://source.android.com/docs/security/bulletin/2024-12-01)
* [NIST NVD: CVE-2024-49747 Detail](https://nvd.nist.gov/vuln/detail/CVE-2024-49747)
* [NIST NVD: CVE-2024-54499 Detail](https://nvd.nist.gov/vuln/detail/CVE-2024-54499)
* [Apple nears switch to in-house Bluetooth and Wi-Fi chip for iPhone, smart home, Bloomberg reports](https://www.reuters.com/technology/apple-nears-switch-in-house-bluetooth-wi-fi-chip-iphone-smart-home-bloomberg-2024-12-12/)
* [Operation Triangulation: The last (hardware) mystery](https://securelist.com/operation-triangulation-the-last-hardware-mystery/111669/)
* [BLASTPASS - NSO Group iPhone Zero-Click, Zero-Day Exploit Captured in the Wild](http://citizenlab.ca/2023/09/blastpass-nso-group-iphone-zero-click-zero-day-exploit-captured-in-the-wild/)
* [Zero-Day Marketplace Explained: How Zerodium, BugTraq, and Fear contributed to the Rise of the Zero-Day Vulnerability Black Market](https://lab.wallarm.com/zero-day-marketplace-explained-how-zerodium-bugtraq-and-fear-contributed-to-the-rise-of-the-zero-day-vulnerability-black-market/)
* [Serbia: Authorities using spyware and Cellebrite forensic extraction tools to hack journalists and activists](https://www.amnesty.org/en/latest/news/2024/12/serbia-authorities-using-spyware-and-cellebrite-forensic-extraction-tools-to-hack-journalists-and-activists/)
* [The Qualcomm DSP Driver - Unexpectedly Excavating an Exploit](https://googleprojectzero.blogspot.com/2024/12/qualcomm-dsp-driver-unexpectedly-excavating-exploit.html)
* [Permissions on Android](https://developer.android.com/guide/topics/permissions/overview)
* [Google Play malware clocks up more than 600 million downloads in 2023](https://usa.kaspersky.com/blog/malware-in-google-play-2023/29356/)
* [Google Play Protect: On-device Protections](https://developers.google.com/android/play-protect/client-protections)
* [Analysis and Reproduction of iPhone BootROM Exploit: Checkm8](https://ritcsec.wordpress.com/2020/04/28/analysis-and-reproduction-of-iphone-bootrom-exploit-checkm8/)
* [App code signing process in iOS, iPadOS, tvOS, watchOS, and visionOS](https://support.apple.com/guide/security/app-code-signing-process-sec7c917bf14/web)
* [Face Off: Group-IB identifies first iOS trojan stealing facial recognition data](https://www.group-ib.com/blog/goldfactory-ios-trojan/)
