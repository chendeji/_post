---
title: Android源码下载与编译
date: 2015-06-23 16:20:42
tags: [aosp]
categories: [Android]
---

# 下载Android源码

参考：[官方指导](https://source.android.com/source/downloading.html)
其中由于被墙，需要对repo文件中进行修改的地方暂时先不讲，先按照以下步骤来走。

**配置**
环境：VMware 虚拟机
系统：Ubuntu 12
java环境：jdk1.6.0_45

**创建存放下载脚本目录**
```
mkdir ~/bin
PATH=~/bin:$PATH
```

**下载repo脚本**
```
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```

<!-- more -->

**创建存放Android源码目录**
```
mkdir android_source
cd android_source
```

# repo脚本需要修改的地方

上面的下载repo的脚本地址是国外的网址，可能会非常的慢或者下载不下来，所以还是要采用国内的网址。
```
git clone https://aosp.tuna.tsinghua.edu.cn/android/git-repo.git/
cp git-repo/repo ~/bin/
```

**修改repo脚本内容**
```
REPO_URL = 'https://gerrit.googlesource.com/git-repo'
改为
REPO_URL = 'https://gerrit-google.tuna.tsinghua.edu.cn/git-repo'
```

这里采用清华大学的镜像[AOSP](https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/)

**同步源码**
```
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest 
repo sync -j4
```

上面的步骤执行完，目前4.3版本的首次同步代码会有大概27G好像，最好使用以下的方法来下载。
```
wget -c -t 0 https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar 
tar -xf aosp-latest.tar
cd aosp 
repo sync
```

解压过程要大概十几分钟。取决于电脑配置


# 选择同步的版本
```
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-4.3.1_r1
repo sync -j4
```

**查看所有分支**
```
cd .repo/manifests
git branch -a
```

![](/css/images/android_tags.png)


# 编译源码
本人下载的源码是4.3版本的最后一个版本android-4.3.1_r1，在Ubuntu64位的系统上进行编译，会有一些问题，需要注意

```
source build/envsetup.sh
```

**选择编译的目标**
```
lunch full-eng
```
上面的意思是针对所有的移动设备来进行编译

**注意**
编译之前要注意安装必要的依赖
```
sudo apt-get install xsltproc flex bison gperf libxml2-utils libswitch-perl gcc-multilib lib32z1
```


**make版本的注意事项**
make版本需要是3.8.1或者3.8.2的，所以要去下载
[make](ftp://ftp.gnu.org/gnu/make/)
```
tar -zxvf make3.8.2.tar.gz
cd make-3.82

./configure
make
make install
```
完成make安装


# 开始编译

```
make -j4
```
编译后如果看到这一句输出，意味着完成了编译
![](/css/images/aosp_make_finished.png)


# 启动模拟器
```
emulator &
```

![](/css/images/emulator.png)



# Android 内核源码 国内镜像地址

common 
[google](https://android.googlesource.com/kernel/common.git) [清华](https://aosp.tuna.tsinghua.edu.cn/kernel/common.git)

***

exynos 
[google](https://android.googlesource.com/kernel/exynos.git) [清华](https://aosp.tuna.tsinghua.edu.cn/kernel/exynos.git)

***

goldfish 
[google](https://android.googlesource.com/kernel/goldfish.git) [清华](https://aosp.tuna.tsinghua.edu.cn/kernel/goldfish.git)

***

hikey-linaro 
[google](https://android.googlesource.com/kernel/hikey-linaro.git) [清华](https://aosp.tuna.tsinghua.edu.cn/kernel/hikey-linaro.git)

***

lk 
[清华](https://aosp.tuna.tsinghua.edu.cn/kernel/lk.git)

***

msm 
[google](https://android.googlesource.com/kernel/msm.git) [清华](https://aosp.tuna.tsinghua.edu.cn/kernel/msm.git)

***

omap 
[google](https://android.googlesource.com/kernel/omap.git) [清华](https://aosp.tuna.tsinghua.edu.cn/kernel/omap.git)

***

samsung 
[google](https://android.googlesource.com/kernel/samsung.git) [清华](https://aosp.tuna.tsinghua.edu.cn/kernel/samsung.git)

***

tegra 
[google](https://android.googlesource.com/kernel/tegra.git) [清华](https://aosp.tuna.tsinghua.edu.cn/kernel/tegra.git)

***

x86_64 
[google](https://android.googlesource.com/kernel/x86_64.git) [清华](https://aosp.tuna.tsinghua.edu.cn/kernel/x86_64.git)
