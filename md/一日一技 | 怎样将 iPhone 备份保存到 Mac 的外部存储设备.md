> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [sspai.com](https://sspai.com/post/76016)

> Matrix 首页推荐 Matrix 是少数派的写作社区，我们主张分享真实的产品体验，有实用价值的经验与思考。

**Matrix 首页推荐**

[Matrix](https://sspai.com/matrix) 是少数派的写作社区，我们主张分享真实的产品体验，有实用价值的经验与思考。我们会不定期挑选 Matrix 最优质的文章，展示来自用户的最真实的体验和观点。

文章代表作者个人观点，少数派仅对标题和排版略作修改。

一、问题描述
------

前段时间碰到一个问题，我 iPhone 背板碎了，需要走 AppleCare 交给苹果返厂维修，而我的 iPhone 是 512GB 的，在备份数据时候麻烦了，我笔记本本身就是 512GB 空间的，肯定不够用呀。  
而备份 iPhone 默认的存储位置是在本机的硬盘上，我有个 14T 的硬盘，那么如何将默认的备份位置改到外部存储设备上呢。

![](https://cdn.sspai.com/editor/u_kylebing/16645479872255.png)

二、如何设置备份位置到外部存储器
----------------

默认的备份位置在下面这个目录

```
~/Library/Application Support/MobileSync/Backup
```

其目录是这样的，Backup 里面就是你的每个设备每次的备份记录

![](https://cdn.sspai.com/editor/u_kylebing/16645479872274.png)

  
我们要做的就是将这个位置映射到你的移动硬盘上去。

### 1. 确定你的移动硬盘位置

你需要知道自己硬盘的完整文件路径：

1.  打开终端，
2.  输入 `cd` 然后将你的硬盘图标拖到终端中，就会看到它的路径了
3.  回车进入到移动硬盘目录下，指令 `ls -al` 能看到硬盘中的所有文件（图片中的 `ll` 是我自定义的一个指令）
4.  我的就是 `/Volumes/Kyle 14TB/` （在终端的路径需要转义空格，所以能看到终端中名字空格前面有个 `\`）

![](https://cdn.sspai.com/editor/u_kylebing/16645479872280.gif)

### 2. 移动硬盘中新建一个备份文件夹

在你的移动硬盘中新建一个备份文件夹，用于存储接下来的手机备份文件。

```
mkdir Backup
```

此时能看到目录中多出一个名为 Backup 的文件夹  
 

![](https://cdn.sspai.com/editor/u_kylebing/16645479872284.png)

进入这个文件夹并展示它的绝对路径

```
cd Backup
pwd
```

能看到我的这个文件夹的绝对路径是 `/Volumes/Kyle 14TB/Backup`，这个会在下面用到。

![](https://cdn.sspai.com/editor/u_kylebing/16645479872288.png)

### 3. 备份系统原有 Backup 文件夹

进入 `~/Library/Application Support/MobileSync` 目录，并删除或重命名 `Backup` 文件夹。  
如果你之前有已经备份的东西，可以将其重命名成其它名字，总之就是不要占用 `Backup` 这个名字就好。  
下面指令将 `Backup` 文件夹重命名成了 `Backup-old`

```
cd ~/Library/Application\ Support/MobileSync
mv Backup Backup-old
```

### 4. 建立软链接到新备份文件夹

你需要知道，iPhone 的备份目录路径是不会变的，系统备份的时候还是会去找下面这个路径

```
~/Library/Application Support/MobileSync/Backup
```

我们要做的就是建立一个连接将 `~/Library/Application Support/MobileSync/Backup` 与 `/Volumes/Kyle 14TB/Backup` 联系起来，让系统在访问原备份路径的时候就是在访问外部硬盘的路径。

上面我们已经确定了两个路径：

1.  系统的备份路径： `~/Library/Application Support/MobileSync/Backup`
2.  新建的外部备份文件夹路径： `/Volumes/Kyle 14TB/Backup`

在 `~/Library/Application Support/MobileSync` 目录下，执行下面指令建立软件链接，注意如果有空格，需要用 `\` 转义

```
ln -s /Volumes/Kyle\ 14TB/Backup  ~/Library/Application\ Support/MobileSync/Backup
```

这样，此时就在这两个文件夹之间建立了一个软件链接，访问 `~/Library/Application\ Support/MobileSync/Backup` 跟访问 `/Volumes/Kyle 14TB/Backup` 等效。

三、正常备份
------

此时再点击备份，就能正常了，并且在外部存储器的备份文件夹中也已经有了备份文件

![](https://cdn.sspai.com/editor/u_kylebing/16645479872293.png)![](https://cdn.sspai.com/editor/u_kylebing/16645479872297.png)

  
这是备份和中间过程，还没备份完成

![](https://cdn.sspai.com/editor/u_kylebing/16645479872301.png)

备份完成后就能看到的备份文件，500G 备份了 4 个小时，苹果的 USB2.0 真垃圾，万年不更新（2022-09-30）

![](https://cdn.sspai.com/editor/u_kylebing/16645479872305.png)

四、恢复数据
------

恢复数据也是个漫长的过程，用时 4-6 个小时  
 

![](https://cdn.sspai.com/editor/u_kylebing/16645479872309.png)![](https://cdn.sspai.com/editor/u_kylebing/16645479872314.png)

  
中间看《老友记》缓解一下心情 1080p 的不如分辨率小的剧情多，这个删减了太多。  
 

![](https://cdn.sspai.com/editor/u_kylebing/16645479872318.png)

恢复完成  
 

![](https://cdn.sspai.com/editor/u_kylebing/16645479872322.png)

五、完成
----

有个地方需要注意，这样操作之后，下次备份需要再连接当时的硬盘才行。  
如果你想恢复到原来的情况，只需要将那个连接文件删除即可。

```
cd ~/Library/Application\ Support/MobileSync
rm -f Backup
```

另外恢复原来备份文件目录

```
mv Backup-old Backup
```

或者新建一个新的

```
mkdir Backup
```