---
title: "Windows 系统媒体服务器搭建"
subtitle: "使用 Jellyfin, KODI 和 tinyMediaManager"
date: 2022-07-29T19:04:13+08:00
lastmod: 2022-07-29T19:04:13+08:00

tags: [
  "Jellyfin",
  "KODI",
  "tinyMediaManager",
  "Powershell"
]
categories: [
  "娱乐"
]
---

很早就想搞个自己的媒体服务器，可由于 PC 硬件市场巨变，之前配来当 NAS + HTPC（~~交学费~~）的机器只好刷个 Windows 10 当主力机用，拖了不少时间。想了半天，一时半会儿不太可能搞个新机器了，于是就在主力机上开个媒体服务器，优化看片体验。

主力机硬件配置 `i3-8100 (核显 UHD630), 16G DDR4`，前几天还塞了个 `HL-DT-ST BU40N`，可以硬解 HEVC、也能读蓝光碟，完全能胜任。

不多说，直接开搞。

## 安装 Jellyfin

鉴于 EMBY 之流都要收费，我就直接选了开源的 Jellyfin。

下载地址 [Jellyfin Stable Windows Archives](https://repo.jellyfin.org/releases/server/windows/stable/)，如果速率太低，可以挂个代理。

打开安装程序后一路 Next 就行。你也可以选择把 Jellyfin 安装成服务，只是我这个电脑还要当主力机用，希望能随时开关服务器。

安装结束后右键托盘图标，点击 `Open Jellyfin` 即可打开 Web 页面。创建完管理员账号后登录。

其实这个时候已经可以添加目录，并设置元数据查找（刮削），然后开始看了。但是 Jellyfin 的刮削功能并不好，因此我们使用另一个工具 tinyMediaManager 来做。

## 配置 tinyMediaManager 并刮削元数据

这个工具 v3 免费，v4 收费，v3 可以在 [https://www.tinymediamanager.org/download/release-v3/](https://www.tinymediamanager.org/download/release-v3/) 下载。注意，v3 运行需要 JRE 1.8 或更新版本，我这里为了玩 Minecraft 装了 OpenJDK 18，于是就没管了。

安装后运行，电影和剧集的数据源都选 `themoviedb.org`，它的数据是很全的，并且中文数据覆盖广。

添加媒体文件所在的目录，更新源（可能需要较长时间，泡杯茶听会儿歌），然后搜索并刮削（选自动匹配那个）。这期间要保证网络通畅，建议挂个代理。然后你会发现相关的介绍文字、图片等等都出现了，就像这样：

![](/images/media-server/tmm-1.png)

不过，毕竟目前的计算机不具有真正的智能，因此大概率不能一次变成上面那样，你需要手动输入片名搜索，搜索时只需要保留片名，其他东西应当去掉。

当然，有些冷门片没有图片或者某些文字描述，比如这个：

![](/images/media-server/tmm-2.png)

这时候就靠自己发挥了 :D。

剧集和电影类似，只是有时（~~大部分时候~~）tmm 不能正确识别分集，这时需要右键编辑资料手动指定，做好之后大概是这个样子：

![](/images/media-server/tmm-3.png)

刮削结束后，建议执行重命名，方便 Jellyfin 识别。

现在打开 Jellyfin，禁用所有的元数据下载，直接添加媒体库，扫描后你将得到一个完美的海报墙：

![](/images/media-server/jellyfin-1.png)

## KODI

似乎事情到了上一步就结束了，然而 Jellyfin Web 端的奇葩解码水平不断触发转码，让 `ffmpeg.exe` 吞满了 GPU 和 CPU，实在接受不了。我们换个客户端：KODI。

KODI 除了 iOS 那边比较麻烦，其他平台全覆盖，且解码完善，不会触发转码。要让 KODI 添加 Jellyfin 里的媒体，需要先添加插件源 [repository.jellyfin.kodi.zip](https://repo.jellyfin.org/releases/client/kodi/repository.jellyfin.kodi.zip)，然后安装 Jellyfin 插件。安装结束后会给一个对话框要求连接服务器，按指示操作即可。

登陆后会自动添加媒体，然后可以享受了！

## 与 PT 共存

其实还有个问题：对于片源质量较高的 PTers，是不能轻易重命名下载的媒体文件的。复制一份确实可以，但我的硬盘空间承受不了，于是我选择为原文件创建硬链接（NTFS 支持硬链接，硬链接可以看作给原来存储文件的数据块增加了一个指针，重命名不影响原来的文件）。

文件太多，只能写脚本了，既然是 Windows，那就用 PowerShell：

```powershell
$path = "D:\Media"
$targetDir = "D:\MediaDisplay"
$files = Get-ChildItem $path -Recurse

foreach ($file in $files)
{
    $newPath = $targetDir + $file.FullName.Substring($path.Length)
    $newPath = $newPath.Substring(0, $newPath.Length - $file.Name.Length)
    $newPath = $newPath.Replace("[", "``[").Replace("]", "``]")

    if ($file.GetType().FullName -eq [System.IO.FileInfo])
    {
        $clearFullName = $file.FullName.Replace("[", "``[").Replace("]", "``]")
        New-Item -ItemType HardLink -Path $newPath -Name $file.Name -Target $clearFullName
    }
    else
    {
        New-Item -ItemType Directory -Path $newPath -Name $file.Name
    }
}
```

上面的脚本可以解决阴间的方括号被解析为正则表达式的问题。

**然而，tmm 仍可能修改原有的 nfo 文件，不过我这里权限卡得比较死，tmm 不给管理员是改不了的。**

不管怎么说，现在可以在任意位置用任意设备看片了！