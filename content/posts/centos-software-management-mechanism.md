---
title: CentOS 软件管理机制
date: 2019-06-17T13:21:55+08:00
draft: false
tags: ["Linux", "CentOS"]
---


> 在 CentOS 上安装的软件都是以 rpm 格式的软件包进行安装的，并且每个软件包在安装时，都提供依赖检查，检查是否缺少必要的依赖。
rpm 和 yum 是 CentOS 提供的两个软件管理工具，yum 是架构在 rpm 上面发展起来的，那么简单了解下 rpm，着重了解 yum 。

### 简单了解 rpm 使用，`man rpm` 查看更多详情
```
# 安装 xxx
rpm -i xxx.rpm

# 查看 xxx 是否已安装
rpm -q xxx
```

### yum 工作原理
当执行 `yum install xxx` 或 `yum search xxx` 时，会去 /etc/yum.repos.d 路径下取 *.repo 文件里获取软件仓库的链接，下载 yum server 中的 rpm 清单至 /etc/yum.conf 中 cachedir 指定的位置(eg: /var/cache/yum)。若存在 /var/cache/yum 匹配，如果匹配成功，然后去 yum server 进行下载。


> 如果自建的 yum server 有更新，yum server 更新位置需要执行 `createrepo .`，重新生成下 rpm 清单，客户端需要执行 `yum clean all` 清理 cache。

#### yum 配置文件
示例如下：

```
[base] # 软件包所属于的位置，必须是 []
name=CentOS-$releasever - Base # name 而已
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra # 列出该 base 可以使用的镜像点
#baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/ # 实际地址, 支持 httpd, ftp, file 等格式
gpgcheck=1 # 指定需要查阅 RPM 文件内的数字证书 
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7 # 数字证书公钥文件所在处
```
`$releasever` 表示 CentOS 的大版本，我的是7.5，`$releasever` = 7

`$basearch` 表示操作硬件平台，我的是 64 位，`$basearch` = x86_64

#### 常用的 yum 命令
```
# 升级 xxx 软件
yum -y update xxx

# 查看源里是否存在 xxx
yum list xxx

# 查看 xxx 的依赖包
yum deplist xxx

# 仅下载 xxx 软件包及依赖, 并不安装
yumdowloader --reslove xxx
```
