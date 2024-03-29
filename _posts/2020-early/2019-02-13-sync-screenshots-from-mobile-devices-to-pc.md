---
title: 从手机截图报Bug扯到工具论
categories: [Tech]
tags: [tips, mobile]
date: 2019-02-13
---

我发现手机端的测试怎么截图报 bug 是个不可忽视的小问题，传统的做法真的很烦。在这里我提供一些思路给大家。

<!-- more -->

## 自动同步截图文件夹

这是一个看上去很不错的思路，现成有个工具可以达到这个目的：坚果云。

在手机端安装一个坚果云，配置好要同步的截图目录，每次截图后它会帮你自动同步。在电脑端安装客户端后直接就能看到新同步的文件，也可以直接在网页端刷新就能预览。

这个方案的问题：

1. 多台手机将登录同一个账号（实际情况必然如此，便于测试多台设备）
2. 多台手机同步的截图目录会混在一起（N 到 1）， 图片很混乱
3. 老设备很久以前的截图也会同步进来，数据量更大更混乱，你还不想删
4. 同步的 App 在后台很有可能也很容易被杀掉

所以这个方案适合新手机或者只用一台手机测试的情况，不是特别推荐。

![image](https://image.tobyqin.cn/2019-02/201901051013491325.JPG)

## 手动分享新图片

这是目前大家私人手机上很常用的方式，在测试机就不太合适了。

比如你不会在测试机登录自己的微信 QQ，就算有公共的账号，在电脑上也只能登录一个实例，你用了其他人就用不了了。N 台设备需要 N 个账号，明显不合适。

所以这个方案的关键是，找到一个合适的**可以多设备登录，分享后能在网页端预览内容的 App**，并且还不能是公开分享，我的建议还是坚果云。

在手机端登录坚果云后，截图后直接分享到坚果云的某个目录，电脑上网页端刷新一下就可以看到截图，整个流程很顺畅。很多手机还可以在分享前编辑一下图片，这时候你还能用涂鸦工具标注 bug 区域等等。

再退一步，也可以考虑用邮箱来代替这样的 App，截图，分享到邮件，填收件人，不算太爽还凑合，还有延迟可能比较高。

这个方案我用下来没有太大问题，支持安卓和苹果设备。唯一的问题是弄一个公共账号，不难。

![image](https://image.tobyqin.cn/2019-02/201901051004382311.JPG)

## 集成截图工具到被测应用

这是不少大厂的做法，比如支付宝应用截图后就会提示你是否要反馈问题，还有一些 App 在你摇一摇后就自动截屏反馈问题。

如果你们的开发团队有精力并且愿意为质量部门添砖加瓦，那么这个世界将会非常美好。这需要各部门的通力合作和强力支持，最主要还是老板和业务部门的点头，给资源和时间。

这样的集成可以做的很深入，不仅截图，还有日志，当前运行数据等等一并捕获提交到 bug 管理工具，省心啊。

![image](https://image.tobyqin.cn/2019-02/4abeb57ffe.jpg)

## DIY 一个同步工具

这是最后考虑的方案，简单来说就是自己写一个 App，比如安装后在状态栏加一个同步按钮，或者贴在屏幕边缘等等。当你需要时点击一下进行同步，同时也提供个分享接口。做这样的一个 App 不算太难，不过还是需要时间和精力的投入，如果你有兴趣我们可以聊一聊。

![image](https://image.tobyqin.cn/2019-02/201901141355323283.png)

基本功能大概是这样的：

- 监控指定文件夹（截图目录），自动上传到远程设备目录
- 提供分享接口
- 收集设备状态
- 提供 Web 管理端，方便访问截图，查看设备信息
- 不需要连接数据线

或许已经有类似的工具了，我没找到。但如果外部工具都不顺手的时候，自己做一个可能更符合需求。

![image](https://image.tobyqin.cn/bee265372c.jpg)

## 关于坚果云

坚果云是目前唯一值得推荐的云盘，不限速只限流量，每月上传 1G 下载 3G，没广告不推销，作为效率软件完全足够。每个项目组可以自己申请一个公用账号。

国内外的网盘因为各种原因都已经不值得推荐了，各种笔记类 App 也已经加上很很多限制，比如多端登录和流量限制等等。

坚果云作为一个要盈利的公司将来怎么样也不好说，而且是公网软件，敏感数据还是不要全往里面放。有朋友说可以考虑搭建私有云，我稍微对比总结了一下：

1. seafile - 国内团队开发的开源企业级云存储方案，提供全平台客户端，口碑较好。
2. owncloud - 开源的个人云解决方案，貌似吐槽比较多，占资源，不太推荐。
3. nextcloud - 据说是从 owncloud 团队分出来的，核心差不多，颜值比较高，口碑也一般。

如果将来公有云都挂了，那么我们就整一个。

![image](https://image.tobyqin.cn/201901051016009569.jpg)

## 我的工具论

有一些同学认为不能用太顺手的工具，因为会形成依赖，一旦离开就会浑身难受。

我认为这种担心是多余的，工具用得好，下班回家早。只有你工作效率提高了，才能更深入去了解业务和提高自己，如果总是忙于繁琐的事物，日复一日终将被工具替代。

古人云了，工欲善其事必先利其器。

古人还云了，磨刀不误砍柴工。

好的工具是不会消失的，但有可能收费。当你觉得值的话，就买下它吧，免费的有可能更贵。
