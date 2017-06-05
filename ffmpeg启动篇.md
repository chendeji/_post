---
title: ffmpeg启动篇
date: 2017-06-01 17:10:00
tags: [ffmpeg,Android]
categories: [Android]
---

## 编译ffmpeg项目
下载[NDK](https://developer.android.google.cn/ndk/downloads/index.html)
下载[ffmpeg](http://ffmpeg.org/)项目源码
还好现在谷歌在中国有服务器了，不然下载NDK就非常的慢。
目前ffmpeg最新的版本是3.3

大部分编译都是在Linux的环境下完成的，不例外，这里也是。
虚拟机->VM->Ubuntu安装
上面的过程比较多，这里不多说，下一个vmware 简称VM。然后下一个Ubuntu的系统镜像ISO。启动就可以用了。

<!-- more -->

**安装解压NDK后，配置NDK的路径**
```
export NDK_HOME=/android-ndk-rXXX //具体版本，指定一下NDK的目录
export PATH=$NDK_HOME:$PATH
```

**编辑ffmpeg的配置文件**
```
su gedit /ffmpeg-3.3/configure
```

将
```
SLIBNAME_WITH_MAJOR='$(SLIBNAME).$(LIBMAJOR)'

LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'

SLIB_INSTALL_NAME='$(SLIBNAME_WITH_VERSION)'

SLIB_INSTALL_LINKS='$(SLIBNAME_WITH_MAJOR)$(SLIBNAME)'
```

改成
```
SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'

LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'

SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'

SLIB_INSTALL_LINKS='$(SLIBNAME)'
```

上面那样改的原因是，如果不修改，最后生成的.so动态库文件，会带有数字。Android项目里用不了

## 特别注意
这里的步骤非常的重要。要特别小心，如果操作失误，会导致编译失败。

**指定临时目录**
```
export TMPDIR=/home/user/tmpdir //没有目录，自己mkdir
```

**指定NDK路径**
```
export NDK=/home/user/ndk/android-ndk-r14b
```

**使用NDK Platform版本**
```
export SYSROOT=$NDK/platforms/android-16/arch-arm/     //一般是要比目前使用的主流机型的系统版本要低
```

**指定编译工具链**
```
export TOOLCHAIN=$NDK/toolchain/arm-Linux-androideabi-4.9/prebuild/linux-x86_64
```

**指定编译后的安装目录**
```
export PREFIX=/home/user/ffmpeg/ffmpeg_install(_x86)  //后面的x86可以根据具体的平台来定，这里不是非常的重要
```

## 编写编译脚本
build_android_arm.sh
```bash
#!/bin/bash
export TMPDIR=/home/user/tmpdir
NDK=/home/user/ndk/android-ndk-r14b
SYSROOT=$NDK/platforms/android-16/arch-arm/
TOOLCHAIN=/home/user/ndk/android-ndk-r14b/toolchains/arm-linux-androideabi-4.9/prebuilt/linux-x86_64

CPU=arm
PREFIX=/home/user/ndk/ffmpeg_install/arm
ADDI_CFLAGS="-marm"

function build_one
{
./configure \
--prefix=$PREFIX \
--enable-shared \
--disable-static \
--disable-doc \
--disable-ffmpeg \
--disable-ffplay \
--disable-ffprobe \
--disable-ffserver \
--disable-doc \
--disable-symver \
--enable-small \
--cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
--target-os=linux \
--arch=arm \
--enable-cross-compile \
--sysroot=$SYSROOT \
--extra-cflags="-Os -fpic $ADDI_CFLAGS" \
--extra-ldflags="$ADDI_LDFLAGS" \
$ADDITIONAL_CONFIGURE_FLAG
make clean
make
make install
}

build_one
```

build_android_x86.sh
```bash
#!/bin/bash
export TMPDIR=/home/user/tmpdir_x86

NDK=/home/user/ndk/android-ndk-r14b
SYSROOT=$NDK/platforms/android-16/arch-x86/
TOOLCHAIN=/home/user/ndk/android-ndk-r14b/toolchains/x86-4.9/prebuilt/linux-x86_64

CPU=x86
PREFIX=/home/user/ndk/ffmpeg_install/x86

function build_one
{
./configure \
  --prefix=$PREFIX \
  --enable-shared \
  --disable-static \
  --disable-doc \
  --disable-ffmpeg \
  --disable-ffplay \
  --disable-ffprobe \
  --disable-ffserver \
  --disable-doc \
  --disable-symver \
  --enable-small \
  --disable-yasm \
  --cross-prefix=$TOOLCHAIN/bin/i686-linux-android- \
  --target-os=linux \
  --arch=x86 \
  --enable-cross-compile \
  --sysroot=$SYSROOT \
  --extra-cflags="-Os -fpic" \
$ADDITIONAL_CONFIGURE_FLAG
make clean
make
make install
}

build_one
```

**变更文件为可执行文件**
chmod +x build_android_arm.sh

## 执行编译脚本
cp build_android_arm.sh ffmpeg3.3
cd ffmpege3.3
./build_android_arm.sh

一会儿之后，你就能在编译后的安装目录中找到编译好的文件了

[参考原文链接](http://blog.csdn.net/xiaoru5127/article/details/51524795)
感谢原作者无私，解决了我编译入门问题。


