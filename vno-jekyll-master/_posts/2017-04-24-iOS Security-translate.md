---
layout: post
title: iOS Security 安全白皮书（一）
date: 2017-04-24
---
[TOC]

![](https://raw.githubusercontent.com/pengxuyuan/quzhibo/master/iOS%20Security%20/4DACC0BE-094F-44BE-AE94-6923AACDFC39.png)

## 前言

iOS 安全白皮书是苹果官方提供的，里面有苹果对于安全设计的一些细节介绍，阅读可以更佳理解苹果系统的构造，虽然不能说是看了能很深的明白其设计思路，但是可以增加知识，为以后做安全方面打下基础

[官方原文地址]([**https://www.apple.com/business/docs/iOS_Security_Guide.pdf****?****from=timeline&isappinstalled=0**](https://www.apple.com/business/docs/iOS_Security_Guide.pdf?from=timeline&isappinstalled=0))  这个是官方的文档，英文好的同学强烈建议看原文档，个人经验来说，看英文跟看中文完全是两种不同的体验，但是看英文会更加有趣更加深刻

[17年白皮书翻译]([**http://www.jianshu.com/p/3a151735fa89#**](http://www.jianshu.com/p/3a151735fa89#))，我是看到巧神推荐才知道有iOS 安全白皮书的，以前对于白皮书的概念还是高中的练习辅导呢，那时候上网找了下有没中文文档，很遗憾没有找到，然后过了半个月这篇翻译就出来了，很快，很强，但是原文都是机翻，然后没有润色，有点难读懂，理解能力强的同学可以直接看，全文都已经翻译好了

[16年白皮书](https://pan.baidu.com/share/link?shareid=3622394083&uk=809426888) 这个无敌，强烈建议大家去看，翻译的堪称完美，至今不知道作者是如何做到的，一句话：无敌，是真的无敌

在我翻译第一章的时候找到这些优秀的资料，一开始是放弃翻译的了。后来自我反思，以前Blog 我也看了很多，但是印象一直不深，但是这次翻译完第一章給我自己的感觉挺好的，认识了很多新知识点，感觉印象很深刻，所以还是想自己慢慢的翻译下，可以慢慢成长。这次打算自己翻译17年的白皮书和阅读[16年白皮书](https://pan.baidu.com/share/link?shareid=3622394083&uk=809426888)，一起讲iOS 安全这方面的知识学习下

以下是翻译

***

### Introduction

### System Security  

#### Secure boot chain  

#### System Software Authorization  

#### Secure Enclave  

#### Touch ID

### Encryption and Data Protection

#### Hardware security features  

#### File Data Protection  Passcodes  

#### Data Protection classes  

#### Keychain Data Protection  

#### Access to Safari saved passwords  

#### Keybags  

#### Security certifications and programs  

### App Security

#### App code signing  

#### Runtime process security  

#### Extensions  

#### App Groups  

#### Data Protection in apps 

#### Accessories  

#### HomeKit  

#### SiriKit  

#### HealthKit  

#### ReplayKit  

#### Secure Notes  

#### Apple Watch  

### Network Security

#### TLS  

#### VPN  

#### Wi-Fi  

#### Bluetooth  

#### Single Sign-on

#### AirDrop security

### Apple Pay

#### Apple Pay components  

#### How Apple Pay uses the Secure Element

#### How Apple Pay uses the NFC controller

#### Credit, debit, and prepaid card provisioning

#### Payment authorization  Transaction-specific dynamic security code

#### Contactless payments with Apple Pay

#### Paying with Apple Pay within apps  

#### Paying with Apple Pay on the web

#### Rewards cards  Suspending, removing, and erasing cards

### Internet Services

#### Apple ID 

#### iMessage 

#### FaceTime 

#### iCloud  

#### iCloud Keychain 

#### Siri  

#### Continuity  Safari Suggestions, Spotlight Suggestions, Lookup, #images, and News

#### Widget in Non-News Countries  

### Device Controls

#### Passcode protection  

#### iOS pairing model  

#### Configuration enforcement  

#### Mobile device management (MDM) 

#### Shared iPad  

#### Apple School Manager  

#### Device Enrollment  

#### Apple Configurator 2  

#### Supervision  

#### Restrictions  

#### Remote Wipe  

#### Lost Mode  

#### Activation Lock  

### Privacy Controls

####  Location Services  

#### Access to personal data  

#### Privacy policy  

### Apple Security Bounty

### Conclusion

#### A commitment to security

### Glossary

### Document Revision History

***


##Introduction
苹果设计的 iOS 平台以安全为核心。当我们着手打造最好的移动平台时，我们借鉴了几十年的经验，打造了全新的架构。我们考虑了桌面环境的安全隐患，并在 iOS 的设计中建立了新的安全方法。我们开发并纳入许多创新功能，来加强手机安全和保护整个系统。因此，iOS 在移动设备安全方面有大的飞跃

每一个 iOS 设备结合软件、硬件和服务一起工作，为了是提供更加具有安全性和透明性的用户体验。iOS 不仅是保护本地的设备和它的数据，而是扩展到整个生态系统，包括用户在本地做的一切，还有在网络上的各种互联网服务

iOS 和 iOS 设备提供先进的安全功能，并且它们也很容易使用。许多功能是默认启用的，因此IT 部门不需要去大量的配置。但是关键的安全功能，如设备加密是不可配置的，所以用户不会因为错误的操作而禁止设备加密。其他功能，例如指纹识别，在提高用户体验方面，可以更简单，更直观的保护设备

本文档提供了如何在 iOS 平台上实现安全技术和功能的详细细节。它也将有助于开发者将 iOS 平台安全技术和功能与自己的政策和程序相结合，以满足其特定的安全需求。

这个文档主要是讨论以下几个话题:  
* 系统安全：为 iPhone、iPad、iPod touch 平台集成安全的软件和硬件  
* 加密和数据保护：架构和设计来保护用户数据，在设备丢失或被盗，或未经授权的人试图使用或修改它的情况下  
* 应用程序安全：使应用程序安全运行，且不影响平台完整性  
* 网络安全：使用行业标准的网络协议，提供安全的认证和加密的数据传输  
* 苹果支付：关于安全支付的实现  
* 互联网服务：苹果为通讯，同步，和备份做的网络基础设施
* 设备控制：允许管理 iOS 设备，防止未经授权的使用，如果设备丢失或被盗可以远程清除数据。  
* 隐私控制：可以控制是否访问位置服务和用户数据。  


![](https://raw.githubusercontent.com/pengxuyuan/quzhibo/master/0B32958D-5334-4746-A1B2-F0D005BE1EE4.png)


##System Security   
系统安全设计，软件和硬件在所有的iOS 设备中的核心组件都是安全的，这包括启动过程、软件更新、安全区域。这种架构在iOS中是安全的核心，不会有另外一种方式让可以运行设备

在iOS 设备中，硬件和软件紧密的相结合，确保系统的每一个组件都是可信任的，并且系统是作为一个有效的整体。从最初的启动到iOS 软件更新到第三方应用程序,每一个步骤进行了分析和审查,以确保硬件和软件以最佳性能结合,并且正确合理的使用资源

进入设备固件升级模式 （DFU Device Firmware Upgrade）

确定只有未修改的Apple 签名代码，才可以恢复设备进入DFU 模式后，将其恢复到已知的良好状态。DFU 模式可以手动的进入：首先用USB 电缆将设备连接到电脑上，然后同时按住Home 键和电源键，然后过了8秒，放开电源键继续按住Home 键 提示：当设备进入DFU 模式屏幕上是不会出现任何东西的，如果苹果Logo 出现了，就是电源键被按住的时间太久了  

###Secure boot chain 安全启动链  
启动过程的每一步都包含了苹果密码签名的组件，来确保完整性，并且只有信任链验证后才能进入。这包括引导装载器、内核,内核扩展,基带固件。安全启动链确保这些最底层的软件不会被篡改

当启动iOS 设备，它的应用处理器会立刻执行只读存储器（ROM）中的代码，这里称之为Boot ROM。这些不可改变的代码，被称之为硬件信任根(hardware root of trust)，在芯片制造的时候就烧进去了，而且是默认信任的

引导ROM 代码包含了苹果根CA 的公钥，在允许进入加载之前用来验证iBoot 引导装载器是否是苹果签名的，这个是在信任链中的第一步，来确保接下来的每一步都是苹果签名认证的，当iBoot 完成它的任务时，它会验证和运行iOS 内核。对于搭载S1，A9或则A系列处理器的设备，Boot ROM 验证一个额外的低级引导装载器（Low-Level Bootloader  LLB），依次加载和验证iBoot

如果引导过程这一步无法加载或者无法验证下一步的程序，启动会被停止，这个时候设备屏幕会显示 “Connect to iTunes” ，这个就是恢复模式。如果引导Boot ROM 无法加载或则验证低级引导装载器，这个时候会进入到DFU 模式（设备固件升级模式）。对于这两种情况，必须将设备用USB 线缆连接到电脑的itunes 恢复到工厂默认设置。对于手动进入恢复模式更多信息可以查看链接：[https://support.apple.com/kb/HT1808]()

对于蜂窝移动数据接入的设备，基带子系统也在相似的安全启动过程中，利用基带处理器来签名软件和密钥验证

设备有安全区域（Secure Enclave），这个安全区域处理器也会利用一个安全启动的启动过程来确保它由苹果来独立的软件验证和签名

###System Software Authorization  系统软件授权  
苹果公司定期发布软件更新来解决新兴的安全问题，并且提供新功能，同时这些支持更新到所有的iOS 设备，用户在设备上接受到iOS 更新提示，可以通过iTunes 和无线来更新，并且鼓励大家快速采用最新的安全补丁

上面所述的启动过程有助于确保在设备上只能安装苹果签名的代码。防止设备被降级到旧版本以致缺乏最新的安全保护，iOS 使用这一过程被称之为系统软件授权。如果降级是可能的，攻击者可以在设备安装一个旧版本的iOS 系统来利用漏洞，但是这个漏洞已经在新版本被修复了

在设备上有安全区域（Secure Enclave），安全区域处理器也会利用系统软件授权来确保软件的完整性，并且防止降级设备。具体可以看接下来的安全区域章节

iOS 软件更新可以使用iTunes 更新，也可以在设备上使用空中下载技术（over the air OTA）。利用iTunes 更新，会下载一份完整的iOS 系统并安装，用OTA 更新只会下载必要的组件进行更新，提高网络效率，并不会下载整个操作系统。另外，软件更新可以被缓存在macOS 服务器运行在本地网络服务，所以iOS 设备不需要访问苹果服务器获取必要的更新数据

在iOS 升级过程中，iTunes 更新（或者利用设备的OTA 更新），会连接到苹果安装授权服务器，然后发送一系列的加密测量结果（cryptographic measurements ），里面有每个安装包需要更新的信息（例如：iBoot、内核、OS 映像），还有一个临时随机的不重复的（anti-replay）值，设备的唯一ID（ECID）

授权服务器会检查这个加密的测量结果，查看是否允许安装，如果匹配成功，会将ECID 加到这个测量结果中并且标记这个结果。在升级过程中，服务器将一组完整的签名数据传递给设备。添加一个处理过的ECID 到请求设备，只对加密的测量结果授权和认证，服务器确保只会由苹果提供更新

启动时信任链会评估验证这个签名是从苹果来的，和从磁盘中加载这个测量结果，结合设备的ECID，匹配所要更新的签名

这些步骤确保为特定的设备授权，和一个设备的旧版系统的不能被复制到另外一个设备上面去，可以防止攻击者保存服务器的响应，并且利用它来篡改设备或修改系统软件

###Secure Enclave 安全区域
安全区域是一个协助处理器，是在Apple S2，Apple A7 以及后面的A 系列处理器中出现的。它使用加密内存，包括硬件随机数发生器。安全区域提供了所有的数据保护密钥管理的加密操作，并且维护数据的完整性，即使内核已受到破坏。安全区域跟应用处理器通讯被隔离在一个中断驱动信箱和一个共享内存数据缓冲器中

安全区域运行一个苹果定制的L4 微核，安全区域利用自己的安全启动去做一个个性化的软件更新，这里也是跟应用处理器分离的。在A9 及以后的A 系列的处理器中，芯片会安全的生成一个UID（唯一的ID）。这个UID 连苹果和系统的其他部分都不知道的

当设备启动的时候，会创建一个临时密钥，并跟UID 混淆在一起，这个用来对设备内存模块中的安全区域进行加密。除了在苹果A7 处理器上，安全区域的内存也用这个临时密钥做了加密

另外，数据用一个随机值和UID 混淆后加密存储在安全区域的文件系统中

安全区域负责处理指纹识别传感器的指纹数据，判断指纹是否有效，然后允许访问设备或者通过用户的购买行为，处理器跟指纹识别传感器是在串行外设接口上通讯的，处理器将数据传输到安全区域但是处理器并不能读取数据内容，这个数据用指纹识别传感器跟安全区域的一个设备共享密钥来进行加密和验证，这个密钥用AES 方法来加密，两边都会提供一个随机密钥，并且用AES-CCM 来传输数据

###Touch ID 指纹识别传感器
Touch ID 是指纹识别传感器系统，可以更快、更简单、更安全的解锁手机。该技术会从任何角度读取指纹数据，并随着时间的推移更多地了解用户的指纹，随着每次使用的其他重叠节点被识别，传感器继续扩展指纹图

Touch ID 可以让用户使用更长、更复杂、更实用的密码，因为用户现在不用频繁地输入密码。Touch ID 克服了用密码解锁的不便，但不是通过替换它来解决，是在一定的时间和场景内可以使用 Touch ID 来安全的访问设备

###Touch ID and passcodes 指纹识别传感器和密码

要使用Touch ID，用户必须要为手机设置一个解锁密码，当Touch ID 扫描并识别到一个已经注册过的指纹时，设备可以在不需要密码的情况下解锁。Touch ID 通常可以代替密码，但是在以下情况仍然需要使用密码：

* 设备刚刚启动或者重启
* 设备48小时内没有被解锁
* 在过去的156小时（六天半天）中，密码未被用于解锁设备，Touch ID在过去4小时内未解锁设备
* 设备已收到远程锁定命令
* 指纹识别失败5次
* 使用Touch ID设置或注册新手指时

当启用Touch ID时，按下睡眠/唤醒按钮时，设备立即锁定。很多用户会设置一个解锁的宽限期，以避免每次使用该装置输入密码。使用Touch ID，设备会在每次进入睡眠状态时锁定，并在每次唤醒时都需要指纹或者输入密码

Touch ID 可以最多设置5根不同的手指，随着一根手指的录入，跟别人误匹配的机率是50万分之一。在Touch ID 指纹识别失败了五次，用户才需要输入密码以获取访问权限

###Other uses for Touch ID
Touch ID 也可以用于Apple Pay（苹果提供的安全支付），有关更多Touch ID 的信息，请参阅本文档的Apple Pay 部分

另外，第三方应用程序可以使用系统提供的API 来让用户使用Touch ID 或密码进行身份验证。应用程序仅被通知身份验证是否成功; 它无法访问Touch ID 或与注册指纹相关联的数据

Keychain 也一样可以使用Touch ID 来解锁，只有通过指纹识别或者输入密码才能被访问。应用程序开发人员还有API 来验证用户是否设置了密码，因此可以使用Touch ID对钥匙串项进行身份验证或解锁

在iOS 9 及以后，开发者可以做以下一些事情：

* Require that Touch ID API operations don’t fall back to an application password or the device passcode. Along with the ability to retrieve a representation of the state of enrolled fingers, this allows Touch ID to be used as a second factor in security sensitive apps.
* Generate and use ECC keys inside Secure Enclave. These keys can be protected byTouch ID. Operations with these keys are always done inside Secure Enclave afterSecure Enclave authorizes the use. Apps can access these keys using Keychainthrough SecKey. SecKeys are just references to the Secure Enclave keys and the  keys never leave Secure Enclave.

Touch ID 也可以直接解锁 iTunes Store、App Store、iBooks Store 的购买项目，用户从而不用输入Apple ID 的密码。当用户授权购买的时候，设备和商店之间交换认证令牌。令牌和加密随机数保存在安全区域（Security Enclave）中，这个随机数是被所有设备和iTunes Store 共享的安全区域密钥进行签名的。在iOS 10 中，Touch ID 保护安全区域中的ECC 密钥，这个密钥是用于通过认证购买的请求

### Touch ID security 

仅当Home按钮的电容钢环检测到手指的触摸时，指纹传感器才会起作用，这将触发高级成像阵列扫描手指并将扫描发送到安全区域

光栅扫描临时存储在安全存储器内的加密存储空间中，同时被矢量化分析，然后被丢弃。这个分析是使用subdermal ridge flow angle mapping 技术，这是丢弃重建用户实际指纹所需的细节数据的有损过程。生成的结果是没有任何身份信息相关的数据，并且加密存储在安全区域，只能有安全区域访问，而且永远不会发送給苹果或者备份在iCloud、iTunes 上面

### How Touch ID unlocks an iOS device

如果Touch ID关闭，设备锁定时，保留在安全区域中的Data Protection class Complete的密钥将被丢弃。在用户通过输入密码解锁设备之前，该类中的文件和钥匙串项目是无法访问的。

当Touch ID 开启的时候，这个密钥不会在设备锁定的时候被丢弃。相反，这个密钥会被加到在安全区域内的Touch ID 子系统的密钥中。当用户尝试解锁设备时，如果Touch ID识别用户的指纹，则它会提供用于一个解密	Data Protection keys 的密钥，并且设备被解锁。这个解锁的过程需要Data Protection 和 Touch ID 子系统共同协作提供额外的保护

如果设备重新启动，并且在48小时没有解锁或五次失败的Touch ID 识别尝试后，安全区域会丢弃Touch ID 解锁设备所需的密钥