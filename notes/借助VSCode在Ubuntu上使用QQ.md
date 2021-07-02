---
date: 2021-6-29
categories: Daily_Life
tags:
  - Ubuntu
  - QQ
comments: true
title: 借助VSCode在Ubuntu上使用QQ
---


<fancybox>
<img src="https://cdn.jsdelivr.net/gh/TianZonglin/tuchuang/ubuntu/2021-07-02_23-59.png"/>
</fancybox>

更新于2021年6月。Ubuntu上使用QQ一直是个痛点，而且令人不解的是腾讯除了在19年把远古版本LinuxQQ拿出来修补一下之外，没有任何迹象表明其会推出官方的Linux新版QQ,所以通常Linux之上使用QQ就几个方式：① Wine版QQ，② 通过协议自定义封装的仿QQ的IM工具。本文使用的就是后者，而且作者借助Vscode以插件形式运行。

<!--more-->

直接上地址： https://github.com/takayama-lily/vscode-qq，**此项目非常新鲜，三个月前才被创建，项目并非专为Linux设计，但完全可以在Ubuntu上使用**，目前使用上发送信息有些小bug，但可以稳定的接受个人和群聊消息。

#### **为什么不用Wine版QQ**

第一，Wine的环境需要额外的安装，整体上有些复杂。本质上Wine类似于JVM虚拟机，可以让Windows应用跨平台的运行在Linux之上，但问题是QQ的适配并不完美，甚至可以说是**十分糟糕**，由于WindowsQQ的版本迭代更新太快，所以导致Wine上的适配工作往往并不及时，而且由于现在QQ的软件本身体量巨大，有些Wine版本的QQ直接放弃维护了，早期的比如：

https://github.com/askme765cs/Wine-QQ-TIM （这在2020年时尚可使用，但目前已无法登陆。）

第二，字体适配不佳。Wine环境本身有一套渲染字体，但由于其并非官方QQ的字体，而且仅有几种简单的字体以供选择，这直接导致即便你可以使用WineQQ，但整个界面都显得十分古怪，甚至使用体验不及官方的水货版LinuxQQ。


可以看到，对于使用QQ而言，WineQQ并不是十分友好的选择，虽然目前有新的针对Ubuntu20的WineQQ版本，但那些版本都基本无法在18、16、14版ubuntu上使用。而且对于我、或者使用Ubuntu作为实验编程平台的用户来说，更换、升级Ubuntu版本是个十分头疼的事情，因为涉及显卡驱动等等事项，非常容易因为升级了内核导致驱动无效而黑屏无法进入系统的情况。综上，在有替代品的前提下，为什么还要用wineQQ呢？！

#### **为什么用Vscode-QQ**

第一，轻量。以Vscode插件的形式，借助QQ协议库，直接重写一套UI，这是个绝佳的想法，没有冗余的功能需求，直接砍掉大量和聊天无关的功能，只保留核心的收发消息、收发表情图片。（文件好像还不支持！）

第二，界面友好。同样，在Vscode内，整个聊天界面直接由文件页面代替，也就是说打开某个人、群组进行聊天，就像打开一个文件那样操作，这在Vscode内是十分舒服的，可以无缝的和你编程工作混合在一起。

第三，安装方便。直接在Vscode插件栏搜索安装即可，安装之后需要用Vscode的命令搜索栏登陆一下账号，之后就可以愉快的聊天了，没有任何多余步骤，真的一分钟就可以在Ubuntu上使用到QQ！

<fancybox>
<img src="https://cdn.jsdelivr.net/gh/TianZonglin/tuchuang/ubuntu/2021-07-02_23-55.png"/>
</fancybox>

#### **结语**

就像开头说的，目前此项目还处在开始阶段，连issue也才十几个，使用上的问题也存在，但不影响整体收发信息的功能，可以说很好的解决了QQ在Ubuntu的使用问题，作为一个Ubuntu使用者，非常真诚的向大家推荐这个工具！

<fancybox>
<img src="https://cdn.jsdelivr.net/gh/TianZonglin/tuchuang/ubuntu/2021-07-02_15-57.png"/>
</fancybox>
