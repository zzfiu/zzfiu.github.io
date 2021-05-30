---
title: 基于CloudFlare的DDNS解析 进阶版
abbrlink: 21ac
date: 2021-05-22 22:54:45
tags:
---

### 前提条件

要自建DDNS服务，首先必须要有自己的域名且域名已经接入 Cloudflare （即DNS为CF提供的地址），另外就是使用阿里云解析、DNSPOD云解析、Cloudflare云解析等服务，本次教程以 Cloudflare 为例。

#### 1. 获取CFKEY

打开网页：https://dash.cloudflare.com/profile

在页面下方找到【Global API Key】，点击右侧的View查看Key，并保存下来

![img](https://cdn.jsdelivr.net/gh/zzfiu/pic/img/blog/cf-1.png)

#### 2. 设置用于 DDNS 解析的二级域名

在 Cloudflare 中新建一个A记录，如：ddns.yourdomain.com，指向 1.1.1.1
（可随意指定，如123.123.123.123等等，主要用于后续查看 DDNS 是否生效）

![img](https://cdn.jsdelivr.net/gh/zzfiu/pic/img/blog/cf-2.png)

#### 3. 下载 DDNS 脚本

```shell
yum install -y wget && wget  -N --no-check-certificate https://raw.githubusercontent.com/yulewang/cloudflare-api-v4-ddns/master/cf-v4-ddns.sh
```

#### 4. 修改 DDNS 脚本并填写相关信息

您可在线使用 nano/vi/vim 等工具进行修改，也可以下载到本地进行修改再上传覆盖！
可以参考下面命令使用vi进行编辑

```shell
vi cf-v4-ddns.sh
```

然后按小写字母 i 进入编辑模式

```shell
# API key, see https://www.cloudflare.com/a/account/my-account,
# 这里填写上一步获取的CFKEY
CFKEY=

#输入你需要解析用来DDNS解析的根域名 eg: example.com，比如我的域名是123.com，那么此处填写123.com
CFZONE=

# 登陆CF的Username, eg: user@example.com(即CF的登录邮箱)
CFUSER=

# 填写用来DDNS解析的二级域名，与上面设置的要一致, eg: ddns.yourdomain.com（例 ddns.123.com）
CFHOST=
```

全部填写完毕后按左上角的Esc退出编辑模式，然后输入 :wq 它会自动保存并退出

#### 5. 脚本授权并执行

```shell
chmod +x cf-v4-ddns.sh
./cf-v4-ddns.sh
```

> 如果脚本相关信息填写正确，输出内容会显示当前母鸡IP，登录 Cloudflare **DNS选项** 查看之前设置的 1.1.1.1 已变为母鸡IP

#### 6. 设置定时任务

```shell
输入 crontab -e  然后会弹出 vi 编辑界面，按小写字母 i 进入编辑模式，在文件里面添加一行：

*/2 * * * * /root/cf-v4-ddns.sh >/dev/null 2>&1
```

**如果您需要日志文件，上述代码请替换成下面代码**

```shell
\#如果您需要日志文件，输入下面命令
*/2 * * * * /root/cf-v4-ddns.sh >> /var/log/cf-ddns.log 2>&1
```

至此，教程结束！

转载至 https://blog.natcloud.net/cf-ddns.html
