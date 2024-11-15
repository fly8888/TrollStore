# TrollStore

TrollStore 是一个永久签名的越狱应用程序,可以永久安装任何你打开的 IPA 文件。

它之所以能工作是因为 AMFI/CoreTrust 的一个漏洞,iOS 在验证包含多个签名者的二进制文件的代码签名时存在问题。

支持版本: 14.0 beta 2 - 16.6.1, 16.7 RC (20H18), 17.0

## 安装 TrollStore

关于安装 TrollStore,请参考 [ios.cfw.guide](https://ios.cfw.guide/installing-trollstore) 上的指南

16.7.x (不包括 16.7 RC) 和 17.0.1+ 将永远不会被支持(除非发现第三个 CoreTrust 漏洞,但这种可能性很小)。

## 更新 TrollStore

当有新的 TrollStore 更新可用时,在 TrollStore 设置顶部会出现一个安装按钮。点击按钮后,TrollStore 将自动下载更新,安装并重新启动。

或者(如果出现任何问题),你可以在 Releases 下载 TrollStore.tar 文件并在 TrollStore 中打开,TrollStore 将安装更新并重新启动。

## 卸载应用

通过 TrollStore 安装的应用只能从 TrollStore 本身卸载,在"应用"选项卡中点击应用或向左滑动即可删除。

## 持久性助手

TrollStore 使用的 CoreTrust 漏洞只足以安装"系统"应用,这是因为 FrontBoard 在每次启动用户应用之前都有额外的安全检查(它调用 libmis)。不幸的是,无法安装在图标缓存重新加载后仍然保持的新"系统"应用。因此,当 iOS 重新加载图标缓存时,所有 TrollStore 安装的应用(包括 TrollStore 本身)都会恢复为"用户"状态,无法再启动。

解决这个问题的唯一方法是在系统应用中安装持久性助手,然后可以使用这个助手将 TrollStore 及其安装的应用重新注册为"系统"应用,使其可以再次启动,TrollStore 设置中提供了这个选项。

在已越狱的 iOS 14 上,当使用 TrollHelper 进行安装时,它位于 /Applications 中,并且会在图标缓存重新加载后作为"系统"应用保持,因此在 iOS 14 上使用 TrollHelper 作为持久性助手。

## URL 方案

从 1.3 版本开始,TrollStore 替换了系统 URL 方案 "apple-magnifier"(这样做是为了防止"越狱"检测像检测独特 URL 方案那样检测到 TrollStore)。这个 URL 方案可以用于直接从浏览器安装应用,或者从应用本身启用 JIT(仅 2.0.12 及以上版本),格式如下:

- `apple-magnifier://install?url=<IPA文件的URL>`
- `apple-magnifier://enable-jit?bundle-id=<包ID>`

在没有安装 TrollStore (1.3+) 的设备上,这只会打开放大镜应用。

## 功能

IPA 内的二进制文件可以有任意授权,使用 ldid 和所需授权进行假签名(`ldid -S<授权文件路径> <二进制文件路径>`),TrollStore 在安装时使用假根证书重新签名时会保留这些授权。这给你提供了很多可能性,下面解释了其中一些。

### 禁用的授权

在 A12+ 的 iOS 15 上,以下三个与运行未签名代码相关的授权被禁用,没有 PPL 绕过是不可能获得的,使用这些授权签名的应用在启动时会崩溃。

`com.apple.private.cs.debugger` (苹果私有调试器)

`dynamic-codesigning` (动态代码签名)

`com.apple.private.skip-library-validation` (跳过库验证)

### 解除沙盒限制

你的应用可以使用以下任一授权来运行在沙盒外:

```xml
<key>com.apple.private.security.container-required</key>
<false/>
```

```xml
<key>com.apple.private.security.no-container</key>
<true/>
```

```xml
<key>com.apple.private.security.no-sandbox</key>
<true/>
```

如果你仍然想要为应用保留沙盒容器,推荐使用第三个选项。

你可能还需要平台应用授权才能使这些正常工作:

```xml
<key>platform-application</key>
<true/>
```

请注意,平台应用授权会导致一些副作用,比如沙盒的某些部分变得更加严格,所以你可能需要额外的私有授权来绕过这些限制。(例如,之后你需要为每个想要访问的 IOKit 用户客户端类添加例外授权)。

为了让带有 `com.apple.private.security.no-sandbox` 和 `platform-application` 的应用能够访问其自己的数据容器,你可能需要以下额外授权:

```xml
<key>com.apple.private.security.storage.AppDataContainers</key>
<true/>
```

### Root 助手

当你的应用不在沙盒中时,你可以使用 posix_spawn 生成其他二进制文件,你还可以使用以下授权以 root 身份生成二进制文件:

```xml
<key>com.apple.private.persona-mgmt</key>
<true/>
```

你也可以在应用包中添加自己的二进制文件。

之后你可以使用 [TSUtil.m 中的 spawnRoot 函数](./Shared/TSUtil.m#L79) 以 root 身份生成二进制文件。

### 使用 TrollStore 无法实现的功能

- 获取正确的平台化权限 (`TF_PLATFORM` / `CS_PLATFORMIZED`)
- 生成启动守护程序 (需要 `CS_PLATFORMIZED`)
- 向系统进程注入插件 (需要 `TF_PLATFORM`, 用户层 PAC 绕过和 PMAP 信任级别绕过)

### 编译

要编译 TrollStore,请确保已安装 [theos](https://theos.dev/docs/installation)。另外确保已安装 [brew](https://brew.sh/),并从 brew 安装 [libarchive](https://formulae.brew.sh/formula/libarchive)。

## 致谢与延伸阅读

[@alfiecg_dev](https://twitter.com/alfiecg_dev/) - 通过补丁比对发现了使 TrollStore 工作的 CoreTrust 漏洞,并致力于自动化绕过。

Google 威胁分析组 - 作为野外间谍软件链的一部分发现了 CoreTrust 漏洞并报告给苹果。

[@LinusHenze](https://twitter.com/LinusHenze) - 发现了在 iOS 14-15.6.1 上通过 TrollHelperOTA 安装 TrollStore 的 installd 绕过,以及 TrollStore 1.0 中使用的原始 CoreTrust 漏洞。

[Fugu15 演示](https://youtu.be/rPTifU1lG7Q)

[关于第一个 CoreTrust 漏洞的详细说明](https://worthdoingbadly.com/coretrust/)