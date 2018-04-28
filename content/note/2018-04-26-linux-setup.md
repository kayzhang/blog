---
title: My Linux Development Environment Setup
subtitle:
date: '2018-04-26'
slug: linux-setup
categories:
  - Setup
tags:
  - Linux
  - Ubuntu
---

本文记录我在 Ubuntu 上搭建开发环境及其他配置的具体配置信息。

# 开发环境
## 基本配置
**> 启用 root 账户**

    sudo passwd root

**> 安装 oh-my-zsh**

官方指南：[oh-my-bash](https://github.com/robbyrussell/oh-my-zsh)

1. 安装 zsh

        sudo apt install zsh

    通过 `zsh --version` 查看版本

2. 设置 zsh 为默认 shell

    首先通过 `echo $SHELL` 查看默认 shell，结果为 `/bin/bash`；`$SHELL --version` 结果为 `GNU bash, version 4.4.19(1)-release (x86_64-pc-linux-gnu)`。

    然后确认 zsh 在 shell 列表 */etc/shells* 里边，之后设置默认 shell 为 zsh：

        chsh -s $(which zsh)

    注销账户之后，查看 `echo $SHELL` 结果为 `/usr/bin/zsh`；`$SHELL --version` 结果为 `zsh 5.4.2 (x86_64-ubuntu-linux-gnu)`。

3. 安装 oh-my-zsh

    通过 curl 安装：

        sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

    或通过 wget 安装：

        sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"

**> 在设置中安装中文支持环境**

**> 安装 ibus-libpinyin 拼音输入法**

    sudo apt-get install ibus-libpinyin

**> 更新软件**

首先更改软件源，然后更新源

    sudo apt-get update

之后在软件中心进行软件更新。

**> Chrome**

官方指南：[Linux Software Repositories](https://www.google.com/linuxrepositories/)

Chrome 安装包会自动配置 repository，直接从官网下载 deb 包安装即可。

## Git & GitHub
**> Git**

官方指南：[http://www.git-scm.com/book/zh/](http://www.git-scm.com/book/zh/) & [https://help.github.com/articles/set-up-git/](https://help.github.com/articles/set-up-git/)

    sudo apt-get install git

设置用户名和邮件

    git config --global user.name "John Doe"
    git config --global user.email johndoe@example.com

查看用户名和邮件

    git config --list

**> GitHub**

官方文档：[https://help.github.com/articles/set-up-git/](https://help.github.com/articles/set-up-git/)

注：开了两步验证之后，必须用 token 作为密码 push，因此选择 store 而非 cache 会更加方便一点。

    git config --global credential.helper store

下边是之前用的 cache 方式：

Set git to use the credential memory cache:

    git config --global credential.helper cache

Change the default password cache timeout (15 minutes) to 1 hour:

    git config --global credential.helper 'cache --timeout=3600'

## 编辑器
**>	Vim**

    sudo apt-get install vim

直接把 */usr/share/vim..../vimxx/* 下的 `vimrc_example.vim` 拷到主目录下并改名 `.vimrc` 来保存配置信息。

    cp /usr/share/vim/vimxx/vimrc_example.vim ~/.vimrc

在文件结尾添加以下设置

    set shiftwidth=4   "设置缩进的空格数为4
    set nu             "打开行号

取消自动生成备份文件，找到以下代码
  
    if has("vms")
    set nobackup " do not keep a backup file, use versions instead
    else
    set backup " keep a backup file

注释掉后两行，即

    if has("vms")
    set nobackup " do not keep a backup file, use versions instead
    "else
    " set backup " keep a backup file

**> Emacs**

    sudo apt-get install emacs25

在主目录下建立配置文件 *~/.emacs*，并添加以下内容

    ;; emacs配置信息
    ;; 取消自动备份
    (setq make-backup-files nil)
    ;; 代码折叠
    (add-hook 'c-mode-hook 'hs-minor-mode)
    (add-hook 'c++-mode-hook 'hs-minor-mode)
    (add-hook 'java-mode-hook 'hs-minor-mode)

**> Sublime Text 3**

官方指南：[Linux Package Manager Repositories](https://www.sublimetext.com/docs/3/linux_repositories.html#apt)

注：https 地址貌似用不了，改称 http 即可。

    wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add -
    sudo apt-get install apt-transport-https
    echo "deb https://download.sublimetext.com/ apt/stable/" | sudo tee /etc/apt/sources.list.d/sublime-text.list
    sudo apt-get update
    sudo apt-get install sublime-text

采用 Package Control 自动配置，具体参照 [https://github.com/kayzhang/sublime-config](https://github.com/kayzhang/sublime-config)

**> Atom**

官方指南：[Installing Atom on Linux](https://flight-manual.atom.io/getting-started/sections/installing-atom/#platform-linux)

注：https 地址貌似用不了，改称 http 即可。

    curl -L https://packagecloud.io/AtomEditor/atom/gpgkey | sudo apt-key add -
    sudo sh -c 'echo "deb [arch=amd64] https://packagecloud.io/AtomEditor/atom/any/ any main" > /etc/apt/sources.list.d/atom.list'
    sudo apt-get update
    sudo apt-get install atom-beta

Atom 配置自动同步：[Sync Settings for Atom](https://atom.io/packages/sync-settings)

安装 `sync-settings`

GITHUB_TOKEN: d8(kai: delete this)dafa8621684fcca595b0fb423fa254b540e863

GIST_ID: 24c2d067a9e79805ece05b8a26eaf0f6

## Python
在 Ubuntu 18.04 中默认安装的 python3，但是有些软件需要 python2 的支持，如 Dropbox，因此就会出现 2 个 python 版本，下边是设置 python3 为默认版本的方法。

    sudo update-alternatives --install /usr/bin/python python /usr/bin/python2 1
    sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 10
    sudo update-alternatives --config python

## Java
**> JDK**

官方指南：[Installation of the JDK and JRE on Linux Platforms](https://docs.oracle.com/javase/10/install/installation-jdk-and-jre-linux-platforms.htm#JSJIG-GUID-79FBE4A9-4254-461E-8EA7-A02D7979A161)

参考 Fedora 教程：[Installing Oracle JDK on Fedora](https://fedoraproject.org/wiki/JDK_on_Fedora)

1. 到 [官网](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 下载压缩包 `jdk-x.x.x_linux-x64_bin.tar.gz`
2. 解压并将解压后的文件复制到 */usr/local/java/*

        sudo mkdir -p /usr/local/java
        sudo tar zxvf ./jdk-x.x.x_linux-x64_bin.tar.gz  -C /usr/local/java

3. 修改环境变量，`vim ~/.bashrc`，添加以下内容

        # 添加 JDK 环境变量
        JAVA_HOME=/usr/local/java/jdk-x.x.x
        PATH=$PATH:$HOME/bin:$JAVA_HOME/bin
        export JAVA_HOME
        export PATH

    注：若系统中没有预装 openjdk，那么第 4、5 步可以忽略。

4. Tell the system that the new Oracle Java version is available by the following commands:

        sudo update-alternatives --set java /usr/local/java/jdk-x.x.x/bin/java
        sudo update-alternatives --set javac /usr/local/java/jdk-x.x.x/bin/javac
        sudo update-alternatives --set javaws /usr/local/java/jdk-x.x.x/bin/javaws

5. 设置 Oracle JDK 为默认版本

        sudo update-alternatives --set java /usr/local/java/jdk-x.x.x/bin/java
        sudo update-alternatives --set javac /usr/local/java/jdk-x.x.x/bin/javac
        sudo update-alternatives --set javaws /usr/local/java/jdk-x.x.x/bin/javaws

6. 重载配置信息：`source ~/.bashrc`

7. 查看 JDK 版本：`java -version`

## 前端

**> R**

官方指南：[https://www.r-project.org/](https://www.r-project.org/)

以下为针对 ubuntu artful，即 ubuntu 17.10 的安装方法，18.04 暂时可以用这一个源：

    sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E084DAB9

在 `/etc/apt/sources.list` 结尾添加（注意，一定不要用 https）：

    deb http://mirrors.tuna.tsinghua.edu.cn/CRAN/bin/linux/ubuntu artful/

之后安装

    sudo apt-get update
    sudo apt-get install r-base
    sudo apt-get install r-base-dev

**> RStudio**

下载 deb 安装包：[https://www.rstudio.com/products/rstudio/download/](https://www.rstudio.com/products/rstudio/download/)

安装本地 deb 包安装工具

    sudo apt-get install gdebi

安装 RStudio

    sudo gdebi rstudio-xenial-1.x.xxx-amd64.deb

**> blogdown**

    ## Install from CRAN
    install.packages("blogdown")
    ## Or, install from GitHub
    if (!requireNamespace("devtools")) install.packages("devtools")
    devtools::install_github("rstudio/blogdown")

**> 其他需要的包**

    install.packages(c("processx", "later"))
    options(blogdown.generator.server = TRUE)

**> hugo**

    blogdown::install_hugo()

更新或重装 hugo

    blogdown::update_hugo()

或

    install_hugo(force = TRUE)

查看 hugo 版本，最新版本：[https://github.com/gohugoio/hugo/releases](https://github.com/gohugoio/hugo/releases)

    blogdown::hugo_version()
    
# 其他配置
**> gnome-tweak-tool**

    sudo apt-get install gnome-tweak-tool

安装 [gnome shell extensions](https://extensions.gnome.org/)

首先安装浏览器插件，然后安装本地支持

    sudo apt-get install chrome-gnome-shell

安装扩展 `User Themes` 和 `Dash to Dock` 和 `Lock Keys`

**> 新立得软件包管理器**

    sudo apt-get install synaptic

**> Ubuntu 额外的版权受限程序**

一些受限的音频和视频解码器

**> SMPlayer**

官方指南：[Install SMPlayer for Ubuntu](https://www.smplayer.info/en/downloads)

    sudo add-apt-repository ppa:rvm/smplayer
    sudo apt-get update
    sudo apt-get install smplayer smplayer-themes smplayer-skins

**> 搜狗输入法**

官方指南：[搜狗输入法 for Linux](https://pinyin.sogou.com/linux/)

注：需要用 fcitx，目前其对很多应用支持不是很好，最好妥协使用 ibus。

**> Dropbox**

官方指南：[安装 Dropbox](https://www.dropbox.com/install)

1. 首先第一步可以通过下载官网的 deb 包安装 Dropbox 前端：

        sudo dpkg -i dropbox_2015.10.28_amd64.deb

    这一步，也可以通过以下命令直接安装：

        sudo apt-get install nautilus-dropbox

    在安装的时候可能需要安装一些依赖库，按照提示操作就行。

2. 在启动器中打开 Dropbox

    此时可能会刚开始没反应，之后[报错](https://askubuntu.com/questions/562018/dropbox-and-ubuntu-software-center-doesnt-work-after-setting-python3-4-as-defau?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)，Google 之后发现是将默认 python 由 python2 设置为 python3 导致的，解决方案为：

    将文件 `/usr/bin/dropbox` 中第一行的 `#!/usr/bin/python` 改为 `#!/usr/bin/python2`。

3. 打开 Dropbox，下载后端程序 dameon

    此时发现无法下载，提示 *Trouble connecting to Dropbox servers. Maybe your internet connection is down, or you need to set your http_proxy environment variable.*

    原因在于，Dropbox 下载过程中不使用系统代理，因此选择手动安装 demeon：


        cd ~ && wget -O - "https://www.dropbox.com/download?plat=lnx.x86_64" | tar xzf -c

    接着，从新建的 *.dropbox-dist* 文件夹运行 Dropbox 守护程序

        ~/.dropbox-dist/dropboxd

    之后在弹出的页面中登陆即可。

    此时可以直接启动器启动，但是有一个问题就是终端打开 `~/.dropbox-dist/dropboxd` 可以使用系统代理，而启动器打开不能使用，只需在 Dropbox 的设置里边把代理服务器配置一下就行了。