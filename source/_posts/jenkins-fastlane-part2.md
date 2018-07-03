---
title: 使用Fastlane和Jenkins构建自动打包系统 (Part 2)
date: 2018-07-02 10:05:14
categories: iOS
tags: iOS
---

## Fastlane match 使用已经存在的证书

经过第一部分的学习，**Fastlane**这个工具确实很好用，自动生成证书，自动打包，自动传到appstore，一切看上很完美。但是，在默认情况下，**match**是不能够使用已经存在的证书和provisiong profiles的，换句话说就是每次调用**fastlane match**的时候，如果发现git repo没有就会为你生成新的。显然，这个限制会让很多现有项目的开发者望而生畏。好在，我们有办法可以让它支持现有证，但是这个办法稍微有点麻烦，因为它不是自动的。

> 如果你还未了解什么是fastlane，请先阅读[使用Fastlane和Jenkins构建自动打包系统 (Part 1)](http://gobodigo.com/2018/06/23/fastlane-jenkins/)

### 导出证书

在我们开始使用现有的证书前，先得保证所有需要的证书和私钥都已经被导出并且保存在本地。

**首先**

打开**keychain**, 选择要将要在项目中使用的证书，然后选择导出

![](http://ot51d7lis.bkt.clouddn.com/export_cert.png)

在**File Format**选择导出为.cer文件。

![](http://ot51d7lis.bkt.clouddn.com/export_dialog.JPG)

然后，再次选择导出，在**File Format**选择导出为p12导出私钥（不要设置密码，因为如果设置了密码后，fastlane就无法导入私钥了）。

下一步就是去查找证书在apple developer portal的id了（后续需要以这个id来为证书和私钥命名），但是这个信息在管理后台是看不到的。这个时候，就需要借助**fastlane**内部直接使用的一个库**Spaceship**。

### 使用Spaceship

在命令行输入`irb`打开一个交互式的ruby shell环境。

之后，输入`require 'spaceship'`和回车，这个命令会将这个库导入到当前的shell环境中。接下来，输入下面这个命令来登录到apple developer portal：

```terminal
Spaceship.login
```

然后就是获取证书信息的命令，这里有两种方式，一种是获得全部的，另外一种是按照证书的类型来获取。

第一种，获取所有证书信息：

```bash
Spaceship.certificate.all
```

第二种，获取指定类型的证书信息：

**获取开发证书**

```bash
Spaceship.certificate.development.all
```

**获取发布证书**

```bash
Spaceship.certificate.production.all
```

**证书信息例子**：

```bash
<Spaceship::Portal::Certificate::Development 
	id="TB48U5GPM8", 
	name="iOS Development", 
	status="Issued", 
	created=2018-06-29 10:35:08 UTC, 
	expires=2019-06-29 10:25:08 UTC, 
	owner_type="teamMember", 
	owner_name="owner name", 
	owner_id="X9EZKD2K8B", 
	type_display_id="6QPB9NHCEX", 
	can_download=true>
```

这个命令会列出证书所有的信息，而且很有可能会返回多个证书。这个时候，你需要仔细将其与本地证书的时间和过期对比，一致的就是你要找的证书信息。当你找到匹配的，就以证书信息的id来命名本地保存的相应的证书和私钥。

接下来我们需要将证书push到match使用的git repo上。

### Push 证书和Provision

在命令行输入下面的命令来准备一些需要用到的变量：

```bash
irb(main):004:0> require 'match'
=> true
irb(main):005:0> git_url='git@github.com:path-to/the-encrypted-certs-repo.git'
=> "git@github.com:path-to/the-encrypted-certs-repo.git"
irb(main):006:0> shallow_clone = false
=> false
irb(main):007:0> manual_password = 'password-to-decrypt-the-repo'
=> "password-to-decrypt-the-repo"
irb(main):008:0> 
```
接下来，我们需要使用下面的命令把存放证书的git repo给clone下来并且解密

```bash
irb(main):005:0> workspace = Match::GitHelper.clone(git_url, shallow_clone, manual_password: manual_password)
[21:17:30]: Cloning remote git repo...
[21:17:31]: 🔓  Successfully decrypted certificates repo
=> "/var/folders/5s/5cgmqxkx4fx6bllwgp1y5dlh000gp/T/d2018-0411-3701-148akjh"
```
这个命令将repo clone到了`/var/folders/...`打开它。然后在这个文件夹里面，根据证书的类型来创建不同的文件夹：

- 如果创建开发证书，则放到`certs/development`里面
- 如果是发布证书，则放到`certs/distribution`里面

将证书和私钥都要对应放到上述的文件夹里面。

最后来处理**provision profiles**，首先你得从**Apple developer portal**下载所需要的**provision profiles**。下载完成后，根据不同的类型放入不同的文件夹，规则如下：

- **adhoc**: `profiles/adhoc/AdHoc_<appbundleid>.mobileprovision`
- **appstore**: `profiles/appstore/AppStore_<appbundleid>.mobileprovision`
- **development**: `profiles/development/Development_<appbundleid>.mobileprovision`

做完这些后，现在我们来把证书和provision profiles推到git repo上去吧。执行下面的命令：

```bash
irb(main):006:0> Match::GitHelper.commit_changes(workspace, "add certificate, private key and provisioning profiles", git_url)
```

大功告成。现在你应该可以通过**fastlane**命令来安装证书和provision profiles了。

```bash
fastlane match distribution --readonly
```


