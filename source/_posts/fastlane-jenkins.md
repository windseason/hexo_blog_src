---
title: 使用Fastlane和Jenkins构建自动打包系统 (Part 1)
date: 2018-06-23 22:30:11
categories: iOS
tags: iOS
---

> 发这篇博文的目的主要是总结一些在使用fastlane过程遇到的坑，顺便吐槽一下Fastlane官方网站：写的好飘逸，对于新手来说不看几遍不试几次，完全不知到整个体系是如何的。。。最后我希望能够帮助看到这篇文章的你:]

## Fastlane

![](https://docs.fastlane.tools/img/fastlane_text.png)

**Fastlane**是一种能够将iOS和android平台beta测试或者正式发布完全自动化的工具！It's amazing dude! 它能够做的事情有：

- 生成应用商店需要的所有截图 (本地化的截图！可以根据需要为各种语言生成）
- 处理打包时候的签名
- 发布应用到商店

我们接下来以iOS为例讲解如何使用fastlane打包iOS app

### fastlane初始化配置

详细的setup步骤这里不赘述，大家可以到[Getting started with fastlane for iOS](https://docs.fastlane.tools/getting-started/ios/setup/)查看。

根据文档，安装完fastlane后，配置工程的第一个命令是(这里不建议使用swift版本的init，一个是目前是beta版本，另外一个是当前能查到的资料都是以[ruby DSL](https://www.martinfowler.com/articles/rake.html)来说明的):

```termianl
fastlane init
```

运行这个命令之后，会看到命令行出现交互式提问，第一个问题经常是问你想用fastlane 来干嘛？

```terminal
1. 📸  Automate screenshots (自动化appstore 本地化截图)
2. 👩‍✈️  Automate beta distribution to TestFlight (自动化发布beta测试app到test flight)
3. 🚀  Automate App Store distribution (appstore 发布自动化)
4. 🛠  Manual setup - manually setup your project to automate your tasks (手动配置自己的工程)
```

如果您的app有多个target或者没有找到跟workspace名字一样的scheme，fastlane会提示输入bundle id，然后会提示输入apple id及其密码，如果apple id在多个team还会提示选择team等等，总之问题问完，你的初始配置也就完成了。

配置完后，会在您的工程下面生成一个名为**fastlane**的文件夹，这个文件夹就是所有fastlane的配置所在了。文件夹会包含两个文件:

- Appfile: 用来存储在使用fastlane过程中所需要的有用信息，例如apple id，bundle id等
- Fastfile: fastlane编译以及发布的核心文件，所有需要自动编译配置的代码都放到这个文件里面

### Fastfile

现在重点讲一下如何配置**Fastfile**。初始化完后，fastlane会自动为工程生成如下代码在**fastfile**里面(我在初始化的时候选择的是第3项--appstore发布自动化，如果选择别的选项可能会与下面的例子有稍微不同)。

```ruby
platform :ios do
  desc "Push a new release build to the App Store"
  lane :release do
    build_app(workspace: "Spacename.xcworkspace", scheme: "SchemeName")
    upload_to_app_store(skip_metadata: true, skip_screenshots: true)
  end
end
```

- platform: 告诉fastlane是哪个平台。
- desc: 注解。一般放到每个lane之前一行用于解释这个lane的作用
- lane: 简单来讲就是一个任务的配置，比如发布到test flight算一个任务，打内部测试Adhoc包等。lane 后面跟着的是冒号加任务名称，如上代码： :release, 这样你就可以在命令行下面输入如下命令就可以执行这个任务了：

```terminal
fastlane release
```
- lane的body里面能够使用的命令都叫做action，能调用的[actions](https://docs.fastlane.tools/actions)官方已经帮我们列出来了，而且每个action都有详细的文档。例子中的任务一眼就能看出来是做什么：编译名为**Spacename.xcworkspace**，scheme为**SchemeName**的工程，编译完后上传到app store（跳过上传meta data和screenshots）

> 读者可以亲手试一试，讲workspace和scheme改为自己工程的名字，然后在命令行运行fastlane release，之后如果编译没有失败并且itunes connect连接和验证成功的话，您会看到ipa包已经成功的传到了itunes connect的后台了。怎么样？很厉害吧，就2行代码就搞定了编译，打包，上传到appstore这种平时繁杂的任务！

等等，我们好像漏了什么。对，最重要的程序签名！

### Fastlane match

什么是Fastlane match? Fastlane match不仅仅是在配置fastlane自动打包时候需要用到，而且极大地简化了在一个开发团队中共享证书和provisioning profiles的流程！

- 项目组中每来一个新成员，是不是都要花时间为新成员配置证书，并且把他加入到相应项目的provisioning profiles里面？如果项目众多，是不是管理的一场噩梦？想要简化吗？快使用**Fastlane match**!
- 连接到苹果服务器很慢，下载provisioning profile很慢！想每次都下载快吗？快使用**Fastlane match**!
- 想在团队中方便的使用个人开发者证书，而不用拷来拷去吗？快使用**Fastlane match**!
- 项目组有新设备后，快使用**Fastlane match**! 一键搞定！

说了那么多好处，咱们来看看怎么用！首先在工程所在文件夹运行如下命令（确保在运行之前先运行过fastlane init命令）

```terminal
fastlane match init
```

运行完后，fastlane会问用来存储工程的证书和provisioning profiles的Git repo的URL是什么？所以，马上去Github或者Gitlab上创建一个去吧！

**注意！！！**

- 所有传到指定git repo的证书和provisioning profiles将会被fastlane使用OpenSSL来加密存储的。
- 你必须对曾git repo有完全的控制权力。最好保证你所使用的git repo是私有的并且使用安全的私钥。
- 即使你的证书泄露了，在不知道你的itunes connect账号的情况下并不能对你造成伤害
- 如果你使用Git hub或者Bitbuckt，建议你使用开启两步验证的itunes connect账号来作为match连接时用的账号

跟随着所有提示做完后会在fastlane文件夹下生成一个叫做**Matchfile**的文件，初始的文件内容与下面类似：

```ruby
git_url("git@xxx/certificates.git")

type("development") # The default type, can be: appstore, adhoc, enterprise or development

app_identifier(["bundle identifier 1", "bundle identifier 2"])
#username("xxx@xxx.com") # Your Apple Developer Portal username

# For all available options run `fastlane match --help`
# Remove the # in the beginning of the line to enable the other options

#match(app_identifier: ["bundle identifier 1", "bundle identifier 2"], type: "adhoc")
```

初始化完后，就到为工程生成证书和provisioning profiles了 ，（如果不是新项目，想利用已经存在的证书怎么办？这部分也有解决方案，我打算放到part 2的时候再讲）运行如下命令，就会为工程生成development证书和相应的provisiong profile：

```terminal
fastlane match development
```

第一次运行这个命令会提示请输入passphase，这个passphase就是用来加密证书和provisioning profiles用的。

**注意**: 运行以上命令,如果发现没有相应的证书和provisiong profiles就会生成新的证书和provisioning profiles，所以请慎用！

接下来，如果想在一台新的机器上安装该工程的证书就可以运行下面的命令：

```terminal
fastlane match development --readonly
```

加上**--readonly**就能保证不会生成新的，而仅仅只是执行安装而已。除了development,我们还能指定adhoc, appstore, enterprise等参数来生成不同场景的证书。

生成的证书和provisioning profiles都会被加密放到初始化match时指定的git repo中，这样当使用**fastlane match development**或者**fastlane match development --readonly**的时候，fastlane会第一时间从指定的git repo中去寻找并安装，所以不需要经过苹果服务器。如果git repo是自己公司服务器的，那就更快了:]。

如果想一口气装上所需要的证书和provisioning profiles, 那么可以写一个脚本来完成：

```shell
#!/bin/sh
#install appstore development profiles and cert
fastlane match development --readonly

#uncomment below lines if you need install other types of certs
#fastlane match adhoc --readonly
#fastlane match appstore --readonly
```

将脚本传到工程git上，这样只要有新机器，就运行一下这个脚本就全部装上了。

#### 在Fastfile中使用match

配置好match后，如何在fastfile中使用呢？其实很简单，如下：

```ruby
platform :ios do
  desc "Push a new release build to the App Store"
  lane :release do
  	sync_code_signing
    build_app(workspace: "Spacename.xcworkspace", scheme: "SchemeName")
    upload_to_app_store(skip_metadata: true, skip_screenshots: true)
  end
end
```

只需要在编译之前调用sync_code_signing即可！