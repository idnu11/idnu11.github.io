---
layout: post
title:  "Cobalt Strike 简单修改证书指纹特征"
date:   2019-10-07
categories: Tools
permalink: /archivers/2019-10-07/1
description: "本文记录简单修改Cobalt Strike 缺省生成证书指纹特征及默认端口"
---

本文记录简单修改Cobalt Strike 缺省生成证书指纹特征。

<!--more-->

## 0x00 CobaltStrike 

Cobalt Strike 一款以Metasploit为基础的GUI框架式渗透测试工具，集成了端口转发、服务扫描，自动化溢出，多模式端口监听，exe、powershell木马生成，钓鱼攻击等。主要用于团队作战，能让多Attacker同时连接到teamserver，共享攻击资源与目标信息、sessions。CobaltStrike作为一款协同APT工具，做好流量隐藏这一点是非常重要，本编文章仅记录证书与端口的修改。其他流量隐藏技术见博客更新。

### 如何进行证书修改？

网上有很多cracked版本，其脚本启动文件里面的keytool生成证书并没作修改，默认也是端口是50050，这个是一个明显的特征。

此次使用的是3.14版本。

修改分两种情况： 

- 首次启动teamserver还没生成证书。

- 已经启动过teamserver生成了证书。

## 0x01修改首次启动的CobaltStrike

修改启动脚本里面的内容就可以了

```
vim ./teamserver
```

找到 

![GPbklT.png](https://s1.ax1x.com/2020/03/27/GPbklT.png)

修改成你想要定义的证书，这里实例使用了Symantec官网的证书部分信息。如下

```
keytool -keystore ./cobaltstrike.store -storepass 密码密码密码密码 -keypass 密码密码密码密码 -genkey -keyalg RSA -alias cobaltstrike -dname "CN=www.symantec.com, OU=CorpMktg&Comms-OlineExp, O=SymantecCorporation, L=MoutainView, S=California, C=Us"
```



这部分修改好了再往下翻，找到注释#start the team server的往下一行

![GPbdht.png](https://s1.ax1x.com/2020/03/27/GPbdht.png)

将其中的50050修改成想要定义的端口与<your-store-password>修改成你刚刚定义的证书存储密码即可。

## 0x02对于已经用启动脚本生成证书

- 查看已经生成的证书

- 删除原有的证书

- 使用keytool生成新的证书

  

### 查看teamserver缺省生成证书

```
keytool -list -v -keystore cobaltstrike.store -storepass 123456
```

证书内容如下图

[![GPHise.png](https://s1.ax1x.com/2020/03/27/GPHise.png)](https://imgchr.com/i/GPHise)

证书中有大量的CobaltStrike，这样会很容易被追踪到teamserver

[![GPH8ds.png](https://s1.ax1x.com/2020/03/27/GPH8ds.png)](https://imgchr.com/i/GPH8ds)

### 删除teamserver缺省证书

```
 keytool -delete  -alias cobaltstrike -keystore ./cobaltstrike.store -storepass 123456 
```

### 重新生成证书

我这里使用了Symantec官网的部分证书信息

```
keytool -genkey -keystore ./cobaltstrike.store -storepass <your-store-password > -keypass <your-keypassword> -keyalg RSA -alias symantec -dname "CN=www.symantec.com, OU=CorpMktg&Comms-OlineExp, O=SymantecCorporation, L=MoutainView, S=California, C=Us"
```

再回到teamserver启动脚本里面修改修改端口和密码就OK了。

![GPb5cT.png](https://s1.ax1x.com/2020/03/27/GPb5cT.png)

## 0x03总结

这次修改的只是端口与缺省证书指纹特征的小细节。以往CobaltStrike版本也被暴露出流量特征，如3.13版本前的HTTP响应的多余空白，这些都容易被追踪到流量信息。作为红队渗透必须要了解到软件使用的一些小细节，网络安全往往细节决定成败。



文章参考：

[教你修改cobalt strike的50050端口][1]

[1]: https://www.3hack.com/note/96.html



[keytool如何生成自签名证书？][2]

[2]: https://jingyan.baidu.com/article/6079ad0eb284ad28ff86db18.html



[java keytool导入和删除证书][3]

[3]: https://blog.csdn.net/duan19056/article/details/21025349



[CobaltStrike3.14破解][4]

[4]: https://bithack.io/forum/310



[通过存在7年之久的“空格”特征识别在野Cobalt Strike服务器组件(含规则)][5]

[5]: https://www.freebuf.com/column/196946.html


