---
title: 折腾一下小米路由器
categories: [Life]
tags: [Router]
date: 2022-05-07
---

折腾一下小米路由器，让家里的网络更通畅一点。

> 路由器型号：小米 AX1800

## 降级并打开 ssh

小米 AX1800 可通过降级固件版本至 `1.0.378` 版本后开启 SSH。

> 固件地址： http://cdn.cnbj1.fds.api.mi-img.com/xiaoqiang/rom/rm1800/miwifi_rm1800_firmware_ed621_1.0.378.bin

固件下载后在管理界面找到系统升级，选择本地文件升级即可。注意，最好要清除用户配置，否则后面可能会有很多坑，只能重头再来。比如我就遇到了下面这些错误，重新恢复出厂后再拿 ssh 和 root 就自动修复的。

```
遇到过的错误：

cp: can't stat '/etc/dnsmasq.d/*': No such file or directory

/etc/mixbox/apps/kms/scripts/kms.sh: line 1: can't create /tmp/etc/dnsmasq.d/kms.conf: nonexistent directory

# clash
-ash: clash: not found
```

降级成功后建议在初始化引导里把固件更新关掉，避免功能失效。

登录小米路由管理页面，地址栏 url 里面找到 `stok` 后面字符串替换掉下面 url 里面的 `stok`。

```
a. 获取 SSH 权限

http://192.168.31.1/cgi-bin/luci/;stok=<STOK>/api/misystem/set_config_iotdev?bssid=Xiaomi&user_id=longdike&ssid=-h%3B%20nvram%20set%20ssh_en%3D1%3B%20nvram%20commit%3B%20sed%20-i%20's%2Fchannel%3D.*%2Fchannel%3D%5C%22debug%5C%22%2Fg'%20%2Fetc%2Finit.d%2Fdropbear%3B%20%2Fetc%2Finit.d%2Fdropbear%20start%3B

b. 修改 root 用户密码为 admin

http://192.168.31.1/cgi-bin/luci/;stok=<STOK>/api/misystem/set_config_iotdev?bssid=Xiaomi&user_id=longdike&ssid=-h%3B%20echo%20-e%20'admin%5Cnadmin'%20%7C%20passwd%20root%3B

需要修改为其他密码自行替换 url 中 admin 部分。
```

复制上面编辑好的 URL 到浏览器地址栏中，然后回车确认，看到以下提示已经成功了。

```json
{ "code": 0 }
```

好了，已经获取了 SSH 权限，并且修改了 ROOT 用户的登录密码，默认是 admin。通过 ssh 连接路由器即可。

```
$ ssh root@192.168.31.1
root@192.168.31.1's password:


BusyBox v1.25.1 (2020-11-02 11:05:47 UTC) built-in shell (ash)

 -----------------------------------------------------
       Welcome to XiaoQiang!
 -----------------------------------------------------
  $$$$$$\  $$$$$$$\  $$$$$$$$\      $$\      $$\        $$$$$$\  $$\   $$\
 $$  __$$\ $$  __$$\ $$  _____|     $$ |     $$ |      $$  __$$\ $$ | $$  |
 $$ /  $$ |$$ |  $$ |$$ |           $$ |     $$ |      $$ /  $$ |$$ |$$  /
 $$$$$$$$ |$$$$$$$  |$$$$$\         $$ |     $$ |      $$ |  $$ |$$$$$  /
 $$  __$$ |$$  __$$< $$  __|        $$ |     $$ |      $$ |  $$ |$$  $$<
 $$ |  $$ |$$ |  $$ |$$ |           $$ |     $$ |      $$ |  $$ |$$ |\$$\
 $$ |  $$ |$$ |  $$ |$$$$$$$$\       $$$$$$$$$  |       $$$$$$  |$$ | \$$\
 \__|  \__|\__|  \__|\________|      \_________/        \______/ \__|  \__|

```

## 安装配置 ShellClash

官方地址：https://github.com/juewuy/ShellClash

在路由器的 Shell 界面下输入：

```bash
#fastgit.org加速
export url='https://raw.fastgit.org/juewuy/ShellClash/master' && sh -c "$(curl -kfsSl $url/install.sh)" && source /etc/profile &> /dev/null
#GitHub源
export url='https://raw.githubusercontent.com/juewuy/ShellClash/master' && sh -c "$(curl -kfsSl $url/install.sh)" && source /etc/profile &> /dev/null
#jsDelivrCDN源
export url='https://cdn.jsdelivr.net/gh/juewuy/ShellClash@master' && sh -c "$(curl -kfsSl $url/install.sh)" && source /etc/profile &> /dev/null
```

如果安装失败不是因为网络原因，大概就是因为路由器有了奇奇怪怪的问题，建议直接恢复出厂，再来一遍，不需要多次时间。

安装完毕后就，就可以配置，里面有个小坑爬了很久。

```bash
 1 启动/重启clash服务
 2 clash功能设置
 3 停止clash服务
 4 禁用clash开机启动
 5 设置定时任务
 6 导入配置文件
 7 clash进阶设置
 8 其他工具
 9 更新/卸载
```

选择 `6 导入配置文件`，进入下面菜单。

```
 1 在线生成Clash配置文件
 2 导入Clash配置文件链接
 3 还原配置文件
 4 更新配置文件
 5 设置自动更新
```

选择 `1 在线生成Clash配置文件`，进入下面菜单。

```
 1 开始生成配置文件（原文件将被备份）
 2 设置节点过滤关键字
 3 设置节点筛选关键字
 4 选取在线配置规则模版
 5 选取在线生成服务器
 0 撤销输入并返回上级菜单
```

注意，在这里不能直接选`菜单 1`，必须要先输入一个链接，再按`菜单 1`，这样才能生成配置文件。比如这样：

```
...
-----------------------------------------------
请直接输入第1个链接或对应数字选项 > vmess://xxxx
```

我用的是 v2ray 的 vmess 协议，配置文件生成后服务就可以正常启动。如果你的路线没问题，服务启动后连接到路由器的任意设备就可以开始畅通无阻了。

### 小贴士

1. 在高级菜单还可以配置定时重启，自己摸索一下就好。

2. ShellClash 可以安装网页管理面板，管理地址： http://192.168.31.1/clash/

3. 在管理面板里可以找到配置菜单，开启局域网 HTTP PROXY 和 SOCKS5 PROXY。（好像有 bug，重启后配置丢失）

4. 在管理面板里可以根据服务类型修改代理规则，比如微软全家桶都走代理，这样 OneNote 和 OneDrive 就好用了。

5. 在管理面板里可以监控连接，可以实时看到访问的服务到底走的是代理还是直连。

clash 功能比较复杂，找到一个官网，但也没太看懂是怎么工作的，相关资料不是太多。

1. https://lancellc.gitbook.io/clash/
2. https://github.com/Dreamacro/clash

## 安装配置 mixbox

官方地址：https://github.com/monlor/MIXBOX-ARCHIVE

安装命令如下。

```bash
# ghproxy 源一键安装命令【NEW】
export MB_URL=https://ghproxy.com/https://raw.githubusercontent.com/monlor/mbfiles/master && sh -c "$(curl -kfsSl ${MB_URL}/install.sh)" && source /etc/profile &> /dev/null

# github 源一键安装命令
export MB_URL=https://raw.githubusercontent.com/monlor/mbfiles/master && sh -c "$(curl -kfsSl ${MB_URL}/install.sh)" && source /etc/profile &> /dev/null

# jsdelivr 源一键安装命令
export MB_URL=https://cdn.jsdelivr.net/gh/monlor/mbfiles && sh -c "$(curl -kfsSl ${MB_URL}/install.sh)" && source /etc/profile &> /dev/null
```

注意，确定安装目录前最好确认一下路由器的磁盘空间，如果安装到 /etc 目录很容出现磁盘满的问题。

```bash
# df -h
Filesystem                Size      Used Available Use% Mounted on
mtd:ubi_rootfs           19.3M     19.3M         0 100% /rom
tmpfs                    95.0M         0     95.0M   0% /sys/fs/cgroup
tmpfs                    95.0M      2.1M     92.8M   2% /tmp
/dev/ubi0_2              14.2M    700.0K     12.8M   5% /overlay
overlayfs:/overlay       14.2M    700.0K     12.8M   5% /
ubi1_0                   15.7M      4.3M     10.6M  29% /data
ubi1_0                   15.7M      4.3M     10.6M  29% /userdisk
overlayfs:/overlay       14.2M    700.0K     12.8M   5% /userdisk/data
ubi1_0                   15.7M      4.3M     10.6M  29% /etc
tmpfs                   512.0K         0    512.0K   0% /dev
overlayfs:/overlay       14.2M    700.0K     12.8M   5% /userdisk/appdata/chroot_file/lib
overlayfs:/overlay       14.2M    700.0K     12.8M   5% /userdisk/appdata/chroot_file/usr/lib
```

通过上面的命令结果，我们知道 /tmp 目录空间最大，但是也不能选，因为它属于 tmpfs， 这是一块虚拟内存，重启后数据就会丢失。AX1800 只有 16M 内存，只能放在 /etc 目录里随便玩玩。

```
***************************************
     *****   MIXBOX 工具箱   *****
***************************************
当前版本：[0.1.9.13]	最新版本：[0.1.9.13]
设备型号：[RM1800]  	核心温度：[53°C]
***************************************
00. 返回主菜单
01. aliddns[动态将你的路由器IP绑定到域名] [未安装]
02. aliyundrivefuse[阿里云盘 FUSE 磁盘挂载] [未安装]
03. aria2[Linux下一款高效的下载工具] [未安装]
04. baidupcs[第三方百度网盘web客户端，基于Go语言] [未安装]
05. dms[dms是一款DLNA数字媒体服务器] [未安装]
06. dropbear[移植小米的SSH功能到工具箱] [未安装]
07. easyexplorer[一款跨设备的P2P文件同步工具] [未安装]
08. entware[一款开源且强大的包管理工具，许多功能都通过它来实现] [未安装]
09. fastdick[迅雷快鸟，宽带提速工具] [未安装]
10. filebrowser[Web文件浏览器] [未安装]
11. firewall[防火墙端口开放插件] [未安装]
12. frpc[内网穿透工具，相对于ngrok资源占用较多] [未安装]
13. frps[内网穿透工具Frp服务端] [未安装]
14. httpfile[搭建简单的web文件浏览页面] [未安装]
15. jetbrains[快速搭建JetBrains激活服务器] [未安装]
16. kms[快速搭建Windows、Office激活服务器] [未安装]
17. kodexplorer[可道云，在线文档管理器，需要entware环境] [未安装]
18. koolproxy[简单，快速屏蔽网页或视频广告，TG：https://t.me/joinchat/AAAAAD-tO7GPvfOU131_vg] [未安装]
19. lingmaxdns[DNS优化插件，类似SmartDNS] [未安装]
20. miwifi[小米路由器系统管理，修改samba禁用系统更新等] [未安装]
21. ngrok[轻量级的内网穿透工具] [未安装]
22. npc[一款轻量级、高性能、功能强大的内网穿透代理服务器] [未安装]
23. pptpd[简单但并不安全的VPN服务器] [未安装]
24. qiandao[koolshare merlin 自动签到程序] [未安装]
25. shadowsocks[最好的翻墙工具，没有之一，还可以加速国内外游戏] [未安装]
26. smartdns[DNS加速工具，从多个上游DNS服务器查询，避免DS污染] [未安装]
27. ssserver[快速搭建ss服务端程序] [未安装]
28. tinyproxy[轻量级的Http代理工具] [未安装]
29. transmission[一款BT下载神器] [未安装]
30. ttyd[网页ssh工具，可在网页上执行 shell] [未安装]
31. verysync[基于p2p的文件同步工具，局域网同步速度快] [未安装]
32. vsftpd[快速搭建Ftp服务器，局域网文件共享] [未安装]
33. webd[一款只有40k大小的mini个人网盘] [未安装]
34. webshell[网页ssh工具，可在网页上管理路由器] [未安装]
35. zerotier[一款非常简单易用的内网穿透工具] [未安装]
```

看上去能玩的东西挺多，暂时没爬坑，后续再说。

顺便提一下，mixbox 里集成的 shadowsocks 其实并不好用：

1. 需要占用不少空间，etc 目录安装不下。
2. v2ray 支持不友好，我尝试好多方法还是没成功。
3. 启动不成功，github 还类似的问题不少。
