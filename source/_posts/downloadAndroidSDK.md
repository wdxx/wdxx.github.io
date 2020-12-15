---
title: Andorid SDK 下载
date: 2020-07-29 21:56:54
tags:
---
## 下载commandlinetools
下载地址：https://developer.android.com/studio 点击进去可以查看`Command line tools only`小节

## 解压安装commandlinetools
```bash
cp commandlinetools-linux-6609375_latest.zip ~/
mkdir -p ~/Android/platform-tools
unzip  ~/Android/commandlinetools-linux-6609375_latest.zip -d ~/Android/platform-tools/
export ANDROID_HOME=~/Android
export PATH=$PATH$:$ANDROID_HOME/cmdline-tools/tools/bin
```

##安装android sdk
安装sdk请参考官方文档：`https://developer.android.com/studio/command-line/sdkmanager`
```bash
sdkmanager "platform-tools" "platforms;android-28"
```