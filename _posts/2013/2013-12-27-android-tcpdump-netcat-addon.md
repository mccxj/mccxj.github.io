---
layout: post
comments: true
title: "对UC文章《实时监控Android设备网络包》的补充"
categories: ["uc", "android", "tcpdump"]
---

补充的内容，主要是一些细节的问题，备忘.

## 编译netcat

android上自己好像带了一个，不过也可以自己编译一个。 我这里使用cygwin来编译的，首先去下载源码。

```bash
# cygwin
cd /cygdrive/d/
mkdir -p netcat/toolchain

export NDK=/cygdrive/d/android-ndk-r8e
/cygdrive/d/android-ndk-r8e/build/tools/make-standalone-toolchain.sh --platform=android-8 --install-dir=netcat/toolchain
export PATH='pwd'/netcat/toolchain/bin:$PATH
export CC=arm-linux-androideabi-gcc
export RANLIB=arm-linux-androideabi-ranlib
export AR=arm-linux-and roideabi-ar
export LD=arm-linux-androideabi-ld

# 开始编译源码
cd netcat-0.7.1/
./configure —host=arm-linux
make

# 用file进行检测一下
file src/netcat
src/netcat: ELF 32-bit LSB executable, ARM, version1 (SYSV), ...

# 发到android上去
adb push src/netcat /data/local/netcat
adb shell chmod 777 /data/local/netcat
```
## tcpdump的使用

如果只是监听所有的包，可以用下面的：
```bash
adb shell "tcpdump -n -s 0 -w - | nc -I -p 11233"
```

如果先监听端口的话，又想转发的话，写法有点特别。 
```bash
adb shell "tcpdump -X -n -s 0 -w - port 5000 | nc -l -p 11233"
```

另外，如果像我这样，有cygwin的话，就已经有nc命令了，可以像下面一样进行转发。
```bash
adb forward tcp:11333 tcp:11233 && nc -v 127.0.0.1 | /cygdriver/d/Wireshark/Wireshark.exe -k -S -i -
```