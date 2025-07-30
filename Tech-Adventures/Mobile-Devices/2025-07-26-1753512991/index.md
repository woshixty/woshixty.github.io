转载自: [介绍一个跳过开屏广告的软件：gkd](https://zhuanlan.zhihu.com/p/1891142892980003672)

# 介绍一个跳过开屏广告的软件：gkd

![](assets/1753512991-2250f3372ff768483fff2dc4879eb5ba.jpg)

### **来给大家介绍一个跳过开屏广告的神器gkd**

一些应用是真恶心啊，开屏广告和应用内弹窗广告越来越多，开屏广告就算了，还有那种开屏广告+手机摇一摇跳转的，严重影响用户体验。

为了解决这一问题，各种广告拦截和自动化工具应运而生，其中[GKD](https://zhida.zhihu.com/search?content_id=255958874&content_type=Article&match_order=1&q=GKD&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTM2ODU3NjcsInEiOiJHS0QiLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoyNTU5NTg4NzQsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.DkYi-Iv0WBN49XMqtI6L9UTqdKoC1qPMYO8r6k7v7C0&zhida_source=entity)（搞快点）作为一款开源的自定义屏幕点击应用，因其强大的功能和灵活性而受到广泛关注。

### **应用介绍**

GKD（搞快点）是一款基于[无障碍服务](https://zhida.zhihu.com/search?content_id=255958874&content_type=Article&match_order=1&q=%E6%97%A0%E9%9A%9C%E7%A2%8D%E6%9C%8D%E5%8A%A1&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTM2ODU3NjcsInEiOiLml6Dpmpznoo3mnI3liqEiLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoyNTU5NTg4NzQsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.SdHaUUW8e_u2uf8PGLD5WufhlQTOYEK8n6luTYM-EJ0&zhida_source=entity)、[高级选择器](https://zhida.zhihu.com/search?content_id=255958874&content_type=Article&match_order=1&q=%E9%AB%98%E7%BA%A7%E9%80%89%E6%8B%A9%E5%99%A8&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTM2ODU3NjcsInEiOiLpq5jnuqfpgInmi6nlmagiLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoyNTU5NTg4NzQsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.fie8puxnKdV-Z_cnu51DYadXiKe7kN8xqKuWOrOiZEY&zhida_source=entity)和[订阅规则](https://zhida.zhihu.com/search?content_id=255958874&content_type=Article&match_order=1&q=%E8%AE%A2%E9%98%85%E8%A7%84%E5%88%99&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTM2ODU3NjcsInEiOiLorqLpmIXop4TliJkiLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoyNTU5NTg4NzQsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.1pc1MCkwCsrFh950SMOY271pkxL9L0d4ikslx440QYo&zhida_source=entity)的自定义屏幕点击Android应用。它的核心功能是自动识别并跳过开屏广告、关闭应用内弹窗广告，以及实现一些常见的自动化操作，如微信自动登录确认和红包领取等。

![](assets/1753512991-05644638ba62acc58b09fe265fc544f7.jpg)

作为一个完全开源的项目，GKD在GitHub上拥有超过25.7K的星标和1.3K的分支，显示出其在开发者和用户社区中的广泛认可。该应用采用[GPL-3.0](https://zhida.zhihu.com/search?content_id=255958874&content_type=Article&match_order=1&q=GPL-3.0&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTM2ODU3NjcsInEiOiJHUEwtMy4wIiwiemhpZGFfc291cmNlIjoiZW50aXR5IiwiY29udGVudF9pZCI6MjU1OTU4ODc0LCJjb250ZW50X3R5cGUiOiJBcnRpY2xlIiwibWF0Y2hfb3JkZXIiOjEsInpkX3Rva2VuIjpudWxsfQ.0t79SnsXBgRlV3wn6F2_3D__W1ocmOriumVu7YD0duo&zhida_source=entity)开源协议，允许用户自由使用、修改和分发，体现了开源精神和社区协作的价值。

### **广告拦截与跳过**

GKD的最主要功能是自动识别并处理各类广告内容。与传统的网络层广告拦截不同，GKD通过分析屏幕元素来识别广告，因此能够处理更广泛的广告类型，包括原生应用中的广告。

开屏广告跳过是GKD的基础功能，也是用户最常用的功能之一。当用户打开一个应用时，GKD能够迅速识别出开屏广告，并自动执行点击"跳过"按钮的操作，让用户无需等待广告结束就能直接进入应用的主界面。这不仅节省了用户的时间，也避免了广告内容对用户的干扰。

除了开屏广告，GKD还能处理应用内的各种弹窗广告。例如，在百度贴吧中的帖子广告卡片、知乎回答底部的推荐广告卡片等，GKD都能自动识别并关闭。这些应用内广告往往更加隐蔽，也更难处理，但GKD通过其高级选择器和订阅规则系统，能够精准地定位并处理这些广告元素。

值得一提的是，GKD的广告处理是非侵入式的，它不会修改应用本身的代码或拦截网络请求，而是模拟用户的点击操作来跳过广告。这种方式虽然不能完全阻止广告的加载，但能够最大限度地减少广告对用户体验的影响，同时也降低了与应用本身发生冲突的可能性。

## **如何下载和使用**

首先来到官网指导页面进行下载：[https://gkd.li/guide/](https://link.zhihu.com/?target=https%3A//gkd.li/guide/)

![](assets/1753512991-6cb241fa252c52db8fccf0a4af778e6d.jpg)

或者进入gkd的github仓库：[https://github.com/gkd-kit/gkd/releases](https://link.zhihu.com/?target=https%3A//github.com/gkd-kit/gkd/releases)

找到apk文件点击下载，不建议beta版，可以找稳定版：

![](assets/1753512991-ce7181596cb7791d241e9058ea2e60e8.jpg)

下载后自己安装到手机上。

![](assets/1753512991-b33720306e9afecf19ccea7679a8e112.jpg)

然后打开gkd，配置无障碍授权，具体步骤可以参考指导：[https://gkd.li/guide/](https://link.zhihu.com/?target=https%3A//gkd.li/guide/)

之后我们要配置订阅链接，我们打开这个订阅规则列表链接：[https://github.com/topics/gkd-subscription](https://link.zhihu.com/?target=https%3A//github.com/topics/gkd-subscription)

![](assets/1753512991-288126df89e0dd446202c4d8b1e104c0.jpg)

随便找一个或者多个订阅链接点进去，然后找到订阅链接的复制按扭点击复制：

![](assets/1753512991-52d2c86f95d2ef47dc37c8ad4f120cf8.jpg)

然后回到你的gkd，配置订阅链接，先选择下边的订阅按钮，然后再点击＋号：

![](assets/1753512991-35f0b33b2944ef6c944fc0b79dd94981.jpg)

在弹出来的这个输入框里面，粘贴你刚才复制的订阅链接：

![](assets/1753512991-0ffe2685aa3afcbe56e0c01639341e57.jpg)

粘贴完点击确定，然后记得打开规则匹配和让它常驻后台，这样就可以跳过大量广告啦。

## **GKD 这款“广告跳过神器”，用户们到底买不买账？**

手机广告，尤其是那烦人的开屏“牛皮癣”，简直是当代数字生活的一大痛点。这时候，像 GKD 这样的“救星”就应运而生了。那么，这款号称能干掉广告的 App，在真正用它的人手里，口碑究竟如何？咱们一起来扒一扒真实用户的反馈和评价。

### **总的来说：口碑相当不错，甚至有点“炸裂”**

翻翻各大应用分享平台和技术论坛，你会发现 GKD 的风评普遍挺好。大家普遍觉得这家伙功能给力，干活（跳广告）也稳。不少人甚至把它奉为“李跳跳”的最佳接班人，有些地方甚至青出于蓝。

看看 GitHub 上那 **25.7K 的星标**，这可不是刷出来的。这背后代表着一大票开发者和技术爱好者对 GKD 的认可，不仅是功能好用，更是对它开源理念和技术实力的点赞。

### **核心功能：跳广告，它到底行不行？**

这必须是重头戏！用户反馈最多的就是 GKD 对付**开屏广告**确实有一手，识别快、成功率高，广告刚露头就被“咔嚓”点掉，等待时间大大缩短，爽！

就像有用户在果核剥壳上说的：“很不错，无广告，功能还算可以，继续支持！” 简单直接，表达了满意。当然，也有用户提到小插曲，比如 CSDN 上有人说：“这个好用，就是更新了版本，自动关闭代理就不会自动开了。” 虽然有点小瑕疵，但整体还是认可的。

至于**应用内广告**，大家的感受就有点差异了。有的 App 里，GKD 表现神勇；有的 App 里，就显得有点力不从心。这很大程度上取决于“规则”的更新维护给不给力。那些社区活跃、规则更新勤快的应用，广告自然就更容易被拿下。

### **稳定性和性能：是不是个“省心省电”的主？**

在这方面，GKD 收获了大量好评。很多用户都提到它运行起来\*\*“稳如老狗”\*\*，很少闹脾气（崩溃或卡死）。就算后台挂着，也不会感觉手机变慢或者电量哗哗掉。

知乎上有位用户的评价堪称经典总结：“【稳定】运行极其稳定，无需反复开启无障碍权限。【安全】无需联网，从源头上杜绝了隐私外泄的可能。【速度快】能秒跳绝大多数广告。【能省电】每跳过一次广告，相当于变相地帮你节省 2~3 秒的电量。” 这几点，精准戳中了用户的需求，难怪受欢迎。

### **上手难度：对小白友好吗？**

答案是：相当友好！GKD 的界面设计得挺简洁，功能布局也清楚，不搞那些花里胡哨的。哪怕是对手机不太懂的用户，跟着指引也能很快搞定。

芊芊精典网站上的教程就强调了它的\*\*“傻瓜式操作”\*\*：下好、安装、进设置、找到“自动跳过开屏广告”打开，完事！之后打开别的 App，GKD 就在后台默默干活了，不用你再操心。这种简单直接，无疑大大降低了使用门槛。

### **江湖地位：和其他“广告杀手”比，GKD 排老几？**

聊 GKD，就绕不开它的前辈——**李跳跳**。曾经的王者，可惜英雄迟暮，停止了更新。GKD 很自然地被视为接过了大旗。

用户普遍觉得，GKD 在核心的跳广告功能上不输李跳跳，甚至在某些方面更胜一筹。比如，GKD 的**规则订阅系统**更灵活，你可以自己添加第三方规则源，火力更猛，这是李跳跳做不到的。而且，GKD 的选择器（定位广告元素的技术）也更高级，定位更准。更别提那个方便规则开发者调试的**快照审查**功能了。

最关键的一点：**GKD 是开源的！** 这意味着代码公开透明，更安全，而且就算原开发者不干了，社区也能接力续命。这一点，在李跳跳停更后，显得尤为珍贵，也让大批用户转投 GKD 怀抱。看看西瓜视频上那个“广告杀手GKD，比李跳跳更强的跳广告软件，附几千条跳广告规则”的标题，就知道大家对它的期望和认可度了。

当然，市面上不止 GKD 和李跳跳。还有像 **[AdGuard](https://zhida.zhihu.com/search?content_id=255958874&content_type=Article&match_order=1&q=AdGuard&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTM2ODU3NjcsInEiOiJBZEd1YXJkIiwiemhpZGFfc291cmNlIjoiZW50aXR5IiwiY29udGVudF9pZCI6MjU1OTU4ODc0LCJjb250ZW50X3R5cGUiOiJBcnRpY2xlIiwibWF0Y2hfb3JkZXIiOjEsInpkX3Rva2VuIjpudWxsfQ.x6-zY6Qg5Asx2xiscNrUxtlqK9x_5fY7bRjssjqG1WM&zhida_source=entity) 这类网络广告拦截器**，它们主要通过 VPN 或代理过滤网络请求来挡广告。GKD 跟它们比，优势在于能处理 App 内部的原生广告（很多广告是内置的，网络过滤无效），而且不用搞复杂的 VPN 设置，不抢占 VPN 通道。缺点是，GKD 是“事后处理”（看到广告再点掉），不能阻止广告加载，省不了流量，而且需要无障碍权限，有些人可能会担心隐私。聪明点的用法，可能是 GKD 配上网路拦截器，双管齐下。

还有像 **[Tasker](https://zhida.zhihu.com/search?content_id=255958874&content_type=Article&match_order=1&q=Tasker&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTM2ODU3NjcsInEiOiJUYXNrZXIiLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoyNTU5NTg4NzQsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.trLwbOADQBydlJpB9moCCOF60nRL6F9lLnm7pyu93xc&zhida_source=entity) 这种通用自动化工具**，理论上也能写脚本跳广告。但 GKD 的优势在于**专注和简单**，开箱即用，有现成的社区规则库，不用自己折腾。Tasker 更适合喜欢 DIY、追求极致自动化的玩家。

最后，还有些**系统级广告拦截**（通常要 Root 或特定系统）。这种方式效果可能更彻底，连广告加载都省了。但 GKD 的好处是**门槛低**，不用 Root，绝大多数安卓手机都能用，更新维护也方便。

**总而言之**，GKD 凭借其易用性、强大的跳广告能力（尤其是在规则加持下）、开源特性和活跃社区，在李跳跳之后成功站稳了脚跟。它不是万能的，有自己的适用场景和局限，但对于绝大多数想摆脱烦人广告的普通安卓用户来说，GKD 无疑是一个非常值得尝试的选择。

### **用户的碎碎念：还能更好吗？**

虽然好评如潮，但用户们总有点“小期待”。比如，希望有更多自定义选项，能针对不同 App 设置不同策略；或者规则编辑功能再友好点，让普通人也能轻松 DIY；也有少数用户反映在低端机上偶尔会卡，希望性能再优化优化。甚至有人畅想，GKD 能不能再多干点活，比如自动填表、自动签到啥的？

这些声音，恰恰说明了大家对 GKD 的喜爱和高期待，也给它未来的发展指明了方向。路漫漫其修远兮，期待 GKD 继续进化吧！

## 参考资料

1.  GKD官方GitHub仓库：[https://github.com/gkd-kit/gkd](https://link.zhihu.com/?target=https%3A//github.com/gkd-kit/gkd)
2.  知乎文章：《软件神器丨GKD，一款安静的安卓应用开屏广告跳过软件》，[https://zhuanlan.zhihu.com/p/689786215](https://zhuanlan.zhihu.com/p/689786215)
3.  果核剥壳网站：《Android GKD(自定义屏幕点击) v1.10.0-beta.3》，[https://www.ghxi.com/gkd.html](https://link.zhihu.com/?target=https%3A//www.ghxi.com/gkd.html)
4.  芊芊精典：《GKD v1.10.0-beta.3正式版跳过开屏广告/广告拦截APP必备》，[https://myqqjd.com/67511.html](https://link.zhihu.com/?target=https%3A//myqqjd.com/67511.html)
5.  哔哩哔哩：《自定义屏幕自动点击器 Android GKD v1.8.0激活版》，[https://www.bilibili.com/read/cv36944726](https://link.zhihu.com/?target=https%3A//www.bilibili.com/read/cv36944726)
6.  CSDN博客：《Github 星标9.3K，又一款超实用的开源手机自动跳过广告软件》，[https://blog.csdn.net/easylife206/article/details/135492840](https://link.zhihu.com/?target=https%3A//blog.csdn.net/easylife206/article/details/135492840)
7.  不死鸟：《GKD搞快点：安卓广告跳过应用》，[https://iui.su/2891/](https://link.zhihu.com/?target=https%3A//iui.su/2891/)
