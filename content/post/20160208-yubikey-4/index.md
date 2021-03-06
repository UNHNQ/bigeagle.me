---
title: "YubiKey 4 简介与配置"
layout: post
comments: true
date: 2016-02-08 23:48:07
slug: yubikey-4
tags: [yubikey, pam, gpg, u2f]
---

2012 年我买了自己的[第一块 YubiKey](/2012/05/yubikey/), 当时功能还很少，后来[康哥](http://scateu.me/)在参加 
[BlackHat 会议 ](https://www.blackhat.com/eu-15/briefings.html#is-your-timespace-safe-time-and-position-spoofing-opensourcely)
时，参展的 yubico 公司直接送 [Yubikey NEO](https://www.yubico.com/products/yubikey-hardware/yubikey-neo/)，于是我免费得到
一个。 

Yubikey NEO 比早前的 Yubikey 增加了 OpenPGP Smartcard 和 [U2F](https://www.yubico.com/applications/fido/) 支持，还可以通过 NFC 获得
Yubico One-Time-Password。唯一遗憾的一点是它的 OpenPGP Smartcard 支持到 2048 位 RSA，而我本人的 GPG 密钥都是 4096 位的，所以为了
使用它我只好增加了一个 2048 位的签名子密钥。

15 年 11 月 Yubico 又推出了 [YubiKey 4](https://www.yubico.com/2015/11/4th-gen-yubikey-4/)，增加了 4096 位 RSA 加密支持(貌似只有加密，没有签名)，
加上一些其他原因（后文），时间又正好赶上妹子回国，于是让妹子在 Amazon 买了帮我带了回来。

{{<img src="yubikey-4.jpg" class="center">}}

<!--more-->

首先说一下功能，YubiKey 4 可以**同时**工作在三种模式:

- 传统键盘设备模式: Yubico OTP, Challenge-Response, 静态密码, HOTP 等, 这个模式又有两个slot，对应于短按和长按操作，生成两种密码
- Smartcard 模式: OpenPGP card 和 PIV card，可以用来安全地保存 RSA 私钥
- U2F 模式: 一种两步认证协议，Google, Dropbox, Github 等网站都支持

[@BlahGeek](https://twitter.com/blahgeek) 之前入手 YubiKey 4 时写得 [blog](https://blog.blahgeek.com/yubikey-intro/) 描述得更详细一些，可以参考。


## 各功能配置

首先安装相关工具，Arch 用户直接

    sudo pacman -S yubikey-personalization yubikey-personalization-gui

然后打开所有功能模式

    ykpersonalize -m86 # 同时打开 HID+Smartcard+U2F 


### 传统键盘设备模式

这部分比较简单，使用 `yubikey-personalization-gui` 即可进行图形化配置，[前文](/2012/05/yubikey/) 也讲过 YubiKey 和 PAM 结合做认证。

由于 Yubico OTP 需要认证服务器(默认是 yubico cloud)做认证，所以如果你修改了 Yubico OTP 配置的话，还需要把密钥等信息上传到 Yubico Cloud，
[这里](https://www.yubico.com/wp-content/uploads/2013/07/YubiKey_YubiCloud_Configuration.pdf) 有一份图文并茂的官方文档。

### OpenPGP Card

如果你使用 GPG 来做加密和签名，那么 OpenPGP Card 可以进一步加强安全性和便利性。

首先你需要设置一下 YubiKey。

    gpg --card-edit  # 打开设置

首先敲 `admin` 命令打开管理模式，可以使用 `help` 命令查看帮助，然后用 `passwd` 命令设置 PIN, Admin PIN 和 Reset Code，
可以是字母和数字，PIN 的默认值是 `123456`，Admin PIN 的默认值是 `12345678`。请 **牢牢记住** 这三个密码。

- PIN 码是平时最常用的密码，如果输错三次就会被锁定，需要使用 Reset Code 来解锁，Reset Code 输错三次，只能[物理重置](https://developers.yubico.com/ykneo-openpgp/ResetApplet.html)。
- Admin PIN 是管理卡信息(如添加密钥、修改密码)使用的密码，不能短于 8 位，输错三次则管理功能被锁定，只能[物理重置](https://developers.yubico.com/ykneo-openpgp/ResetApplet.html)。
  **记住，千万不要在需要输入 Admin PIN 的时候输入短于 8 位的密码，这样会直接锁定** (我的 YubiKey NEO 就是这么悲剧的)。

你可以继续设置一些其他信息，比如持卡人姓名、公钥的URL等等，也可以下次再弄。

一个 GPG 密钥包含多个 RSA 密钥对，一对主密钥，若干对不同用途的子密钥，一对密钥包括一个公钥和一个私钥。
一般 `gpg --gen-key` 得到的，就包括一对主密钥，可以用于签名；一对子密钥用于加密。

不推荐把主密钥放进 YubiKey 里，因为主密钥还有一个重要功能就是维持社交关系，时间流逝，子密钥可能经常变化，
而别人只需要记住你的主密钥ID就可以随时更新。如果把主密钥放进 YubiKey 里，万一丢了，那么你就无法证明你是你了。

假设你已经生成好了一个 GPG 密钥组，用

    gpg --edit-key i@example.com

编辑密钥，例如我的密钥:

    gpg --edit-key i@bigeagle.me
    gpg (GnuPG) 2.1.11; Copyright (C) 2016 Free Software Foundation, Inc.
    This is free software: you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.
    
    Secret key is available.
    
    pub  rsa4096/865BAC3A
         created: 2014-12-28  expires: 2020-10-17  usage: SC  
         trust: ultimate      validity: ultimate
    ssb  rsa4096/167D42E5
         created: 2014-12-28  expires: 2016-08-16  usage: E   
         card-no: 0006 04239808
    ssb  rsa2048/1C27920A
         created: 2016-01-25  expires: 2017-01-24  usage: S   
         card-no: 0006 04239808
    [ultimate] (1). Justin Wong <i@bigeagle.me>
    [ultimate] (2)  Justin Wong <bigeagle@xdlinux.info>
    [ultimate] (3)  Justin Wong <justin.w.xd@gmail.com>
    [ultimate] (4)  Yuzhi Wang (Tsinghua) <yz-wang12@mails.tsinghua.edu.cn>
    
    gpg> 

其中 `pub` 开头的是主密钥，`865BAC3A` 就是我的主密钥 ID，功能有签名(S, sign)和证书(C, certificate)；
`ssb` 开头的是子密钥，我有两个: `167D42E5` 是我的加密(E)子密钥，密钥长度 4096bit，算法是 RSA；`1C27920A`
是签名子密钥，长度 2048bit，算法 RSA。绝大多数情况下不需要记住这些密钥 ID。

例如我们现在想把加密和签名子密钥放进 YubiKey 中，那么首先生成一个签名子密钥，用 `addkey` 命令，跟着向导走几步即可。
接下来把子密钥导入 YubiKey，使用 `key <数字>` 选择密钥对，`0`是主密钥对，`1`以上是子密钥，选中后 `ssb` 变会多一个 `*` 号，
可以多次输入`key X`命令同时选中多个密钥。然后用 `keytocard` 命令，此时会提示输入 Admin PIN，正确输入，导入完成。**注意，私钥
一旦进入YubiKey就无法取出，如果你需要备份请提前做，如果你可以取出请找 Yubico 要奖赏。**

这样以后就必须在 YubiKey 插入的时候才能完成 GPG 操作，使用时输入 PIN 码解锁即可。
另一个好处是可以方便的在别人的计算机上完成 GPG 操作，不仅免去了导入导出密钥的麻烦，更重要的更安全。在插入卡
的时候，如果你设置了公钥组的URL，使用 `fetch` 命令即可导入自己的公钥，配合卡上的私钥即可。

还需要注意的是，如果有 GPG Agent 运行，而 YubiKey 被拔出，那么之后的操作会提示找不到卡(即使重新插入)，
运行一下 `gpgconf --reload scdaemon` 即可。我配合 @xiaq 的 [udevedu](https://github.com/xiaq/udevedu)，
每次插入就 reload 一下，脚本在 [gist](https://gist.github.com/bigeagle/924814b4ffd0733db2ca) 里了。

另外，如果使用 YubiKey NEO 的话，则可以配合 [OpenKeyChains](https://www.openkeychain.org/) 通过 NFC 在 Android 手机上进行
GPG 操作。

### U2F 两次认证

现在吼多网站都提供两次认证加强安全性，比如 Google, Dropbox [等](http://www.dongleauth.info/)。
常见的两次认证是使用  Google Authenticator, Duo Mobile 等工具产生 TOTP (Time-based One-Time Password)，
每次登陆的时候都要开手机运行App，总还是麻烦了些，而 U2F 则可以通过插入 YubiKey 并轻触按钮完成认证。

目前只有 Google Chrome 浏览器完全实现了 U2F 支持，Firefox 需要安装[插件](https://addons.mozilla.org/en-US/firefox/addon/u2f-support-add-on/)
但还不太完美。我试了一下，

- GitHub 注册设备操作 Firefox 没响应，用 Chrome 注册设备后，Firefox 可正常认证。
- Google 只允许 Chrome
- Dropbox 只允许 Chrome

另外有[pam-u2f](https://github.com/Yubico/pam-u2f) 可以用于 Linux 的各项认证操作，替代传统的 Challenge-Response，用法也很简单。

Arch 用户直接安装 AUR 里的 `pam_u2f`，首先注册设备

    pamu2fcfg -u<username>

这是 YubiKey 上的灯会闪烁，轻按一下，终端里就会打出一行字，格式为

    <username>:<KeyHandle1>,<UserKey1>

把这一段内容放在 `~/.config/Yubico/u2f_keys`。

编辑需要的 PAM 服务(例如sudo)，在合适的位置加一行

    auth sufficient pam_u2f.so

这样需要认证的时候，YubiKey 灯会闪，轻按一下即可完成认证。

### 最后
还有好多没折腾的功能，比如 PIV Smartcard，后续折腾出来继续更新 #flag。


