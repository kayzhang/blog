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

直接把 */usr/share/vim..../vim6.3/* 下的 `vimrc.sample` 拷到主目录下并改名 `.vimrc` 来保存配置信息。

    cp /usr/share/vim/vim63/vimrc_example.vim ~/.vimrc

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

在主目录下建立配置文件 *~/.emscs*，并添加以下内容

    ;; emacs配置信息
    ;; 取消自动备份
    (setq make-backup-files nil)
    ;; 代码折叠
    (add-hook 'c-mode-hook 'hs-minor-mode)
    (add-hook 'c++-mode-hook 'hs-minor-mode)
    (add-hook 'java-mode-hook 'hs-minor-mode)

**> Sublime Text 3**

官方指南：[Linux Package Manager Repositories](https://www.sublimetext.com/docs/3/linux_repositories.html#apt)

    wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add -
    sudo apt-get install apt-transport-https
    echo "deb https://download.sublimetext.com/ apt/dev/" | sudo tee /etc/apt/sources.list.d/sublime-text.list
    sudo apt-get update
    sudo apt-get install sublime-text

采用 Package Control 自动配置，具体参照 [https://github.com/kayzhang/sublime-config](https://github.com/kayzhang/sublime-config)

**> Atom**

官方指南：[Installing Atom on Linux](https://flight-manual.atom.io/getting-started/sections/installing-atom/#platform-linux)

    curl -L https://packagecloud.io/AtomEditor/atom/gpgkey | sudo apt-key add -
    sudo sh -c 'echo "deb [arch=amd64] https://packagecloud.io/AtomEditor/atom/any/ any main" > /etc/apt/sources.list.d/atom.list'
    sudo apt-get update
    sudo apt-get install atom-beta

Atom 配置自动同步：[Sync Settings for Atom](https://atom.io/packages/sync-settings)

GITHUB_TOKEN: d8(kai: delete this)dafa8621684fcca595b0fb423fa254b540e863

GIST_ID: 24c2d067a9e79805ece05b8a26eaf0f6

## Java
**> JDK**

官方指南：[Installation of the JDK and JRE on Linux Platforms](https://docs.oracle.com/javase/10/install/installation-jdk-and-jre-linux-platforms.htm#JSJIG-GUID-79FBE4A9-4254-461E-8EA7-A02D7979A161)

参考 Fedora 教程：[Installing Oracle JDK on Fedora](https://fedoraproject.org/wiki/JDK_on_Fedora)

1. 到 [官网](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 下载压缩包 `jdk-x.x.x_linux-x64_bin.tar`
2. 解压并将解压后的文件复制到 */usr/local/*

    sudo mkdir -p /usr/local/java
    sudo tar zxvf ./jdk-x.x.x_linux-x64_bin.tar  -C /usr/local/java

3. 修改环境变量，`vim ~/.bashrc`，添加以下内容

        # 添加 JDK 环境变量
        JAVA_HOME=/usr/local/java/jdk-x.x.x
        PATH=$PATH:$HOME/bin:$JAVA_HOME/bin
        export JAVA_HOME
        export PATH

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
**> hugo**

**> R**

官方指南：[https://www.r-project.org/](https://www.r-project.org/)

以下为针对 ubuntu artful，即 ubuntu 17.10 的安装方法：

    sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E084DAB9
    deb https://mirrors.tuna.tsinghua.edu.cn/CRAN/bin/linux/ubuntu artful/
    sudo apt-get update
    sudo apt-get install r-base
    sudo apt-get install r-base-dev
    
而现在针对 bionic 18.04 的特定源还没出，可以直接从 ubuntu 的源安装：

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

安装扩展 `User Themes` 和 `Dash to Dock`

**> 新立得软件包管理器**

    sudo apt-get install synaptic

**> indicator-keylock 大小写显示托盘**

    sudo add-apt-repository ppa:tsbarnes/indicator-keylock
    sudo apt-get update
    sudo apt-get install indicator-keylock

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
