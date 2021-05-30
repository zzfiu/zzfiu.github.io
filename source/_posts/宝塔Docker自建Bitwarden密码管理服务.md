---
title: 宝塔Docker自建Bitwarden密码管理服务
abbrlink: 376a
date: 2021-05-23 01:02:54
tags:
---

[Bitwarden](https://bitwarden.com)是一个跨平台的密码管理软件，类似于 1Password、EnPass、LastPass 等。Bitwarden 是免费开源的，可以将服务端部署在自己的服务器上，比如群晖，并且支持 Docker 部署。但官方的镜像要求至少 2G 以上内存，要求比较高。有人用 Rust 实现了 Bitwarden 服务器，项目叫 [bitwarden_rs](https://github.com/dani-garcia/bitwarden_rs)，并且提供了 Docker 镜像，这个实现更进一步降低了对机器配置的要求，并且 Docker 镜像体积很小，部署非常方便。另外可以用腾讯云 [CDN](https://console.cloud.tencent.com/cdn/access)服务隐藏我们的服务器真实IP以及实现访问加速。

### 1. BitWarden好处都有啥？

- 开源（好像是大部分开源）
- 可以自建
- 不要钱（指自建就不需要额外花钱买服务）
- 客户端比LastPass好看+好用
- 可以同步TOTP密钥
- 官方服务器中需要付费订阅的一些功能，在自建中是免费的

### 2. 宝塔安装Docker

进入到宝塔面板的软件商店，搜索docker安装即可

![](https://img.zzf.red//20200801011631.png#vwid=1247&vhei=489)

### 3. 获取镜像

打开docker管理器，点击获取镜像，输入Bitwarden_rs的官方镜像`bitwardenrs/server`后点击获取镜像。

![c14a8df9528d3](https://img.zzf.red//20200801011742.png#vwid=850&vhei=596)

### 4.创建容器

1.点击创建容器按钮
2.填写端口映射中的  `容器端口`：`80`,`服务端口`：`6666`6666可以自定义
3.填写目录映射中的`服务器目录`：`/www/wwwroot/xxx.zzf.red`(该目录可以自定义，这里用网址作为目录)，`容器目录`：`/data`
 4.填写内存配额，根据自己服务器的配置按需填写
5.提交创建容器
6.点击容器名称，修改容器名称为`Bitwarden`以方便辨认
![Snipaste_2020-08-01_01-20-05](https://img.zzf.red//20200801012025.png#vwid=631&vhei=719)

### 5.添加站点

在宝塔里面添加一个站点，FTP、数据库均不用创建，PHP版本选择纯静态。目录选择刚才创建的服务器目录


网站添加完成后设置SSL，自己准备证书，或者用免费的Let's Encrypt，设置完毕后开启强制Https。

最后添加一个反向代理，名称随意填，目标URL为`http://127.0.0.1:6666`，端口号和上面`创建容器`时`服务端口`保持一致。![Snipaste_2020-08-01_01-34-09](https://img.zzf.red//20200801013443.png#vwid=806&vhei=569)

**上述步骤都做完后别忘了在宝塔面板安全-防火墙中放行端口`6666`。**

### 6. 启用CDN

**CDN配置步骤如下：**

#### 1. 激活服务
打开腾讯云CDN管理界面，确认已经激活服务：[https://console.cloud.tencent.com/cdn/access](https://console.cloud.tencent.com/cdn/access)
#### 2. 添加域名
点击添加域名：`xxx.zzf.red` 如图：
- 源站地址填写服务器IP+上文bitwarden容器的`服务端口`6666
- 回源协议选择 HTTP


#### 3. 缓存配置
- 将全部类型-所有文件的刷新时间改为0天，即不缓存
- 新增一个文件类型，内容为：.jpg;.png;.css;.woff;.woff2;.svg，刷新时间设置为30天或更长，即静态文件缓存到CDN
- 调整下优先级，确保需要缓存的规则放到最前，全部-所有文件这个兜底类型放到最后。
#### 4. 域名解析
在基本配置界面找到CDN给分配的CNAME地址，然后前往你的DNS管理添加 `CNAME`记录 名字为你的网站前缀，记录值为DN给分配的CNAME地址

**做完以上配置，我们就可以正常访问Bitwarden的后台管理了**

### 7. 关闭注册

最后，因为我们部署Bitwarden是私人使用场景，因此需要修改下Bitwarden的容器启动脚本，将前面的SIGNUPS_ALLOWED=true改为SIGNUPS_ALLOWED=false，也就是禁止用户注册。具体步骤如下：

#### 1. 删除容器
在docker管理器中点击刚才创建的容器的状态绿色图标，停止容器运行，然后删除容器（删除容器后不会删除数据）

#### 2. 重新运行容器
 在VPS中运行下面的命令重新运行容器，其中`Bitwarden`为容器的名字，`SIGNUPS_ALLOWED=false`代表禁止注册，`/www/wwwroot/xxx.zzf.red`为上面创建容器时所写的`服务器目录`,`/data`为容器目录，`6666:80`代表上面创建容器时的`服务端口:容器端口`

```
docker run -d --name Bitwarden \
  -e SIGNUPS_ALLOWED=false \
  -v /www/wwwroot/xxx.zzf.red/:/data/ \
  -p 6666:80 \
  bitwardenrs/server:latest
```

运行完成后在容器列表里就又能看到了。
然后再去试下创建账号就会出现一个不能创建账号的错误提示。
