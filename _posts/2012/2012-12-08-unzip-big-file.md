---
layout: post
comments: true
title: "unzip引起disk full问题(旧)"
description: "unzip引起disk full问题"
categories: ["总结", "unzip"]
---

很早的记录了，以前瞎整过很多东西，还好很多资料有整理，所以看上去还有点小用的就迁移过来吧。:)

因为184解压一个oracle dmp文件的时候出错，导致数据无法同步。今天尝试解决这个问题，在日志中看到是解压的时候，
出现write error (disk full?).continue?(y/m/^C)"。发现db.dmp原大小是2.05g，压缩后400m，但是使用unzip解压到2g的时候，就出现上述错误。
但是通过df也没有看到磁盘空间不足。这就有几样可能，有可能是系统限制了或者解压工具限制了。使用ulimit发现文件大小是没有做特别限制的。
搜索一番,有可能是出现unzip的对大文件解压的限制，参考http://osde.info/HowToUnzipLargeFiles，对unzip进行重新编译，结果就可以了。
后来使用which的时候，发现oracle的bin自带了一个unzip，而这个版本也是不支持大文件解压的，但是有人却把这个路径放到path上去了。
最后嘛，我担心改动path路径对oracle造成影响，就修改了恢复脚本的unzip路径。结果总算搞定了。

简略翻译修改版:
{% highlight bash %}
# 从http://www.info-zip.org/下载源代码
cd unzip-5.52
vi unix/Makefile

# 使用:/Linux on查找到下面这段描述
# Linux on 386 platform, using the assembler replacement for crc32.c. (-O4 and
# -fno-strength-reduce have virtually no effect beyond -O3. Add "-m486
# -malign-functions=2 -malign-jumps=2 -malign-loops=2" for Pentium [Pro]
# systems.)
linux: unix_make

# 在这段的下面可以找到并进行替换
CF="-O3 -Wall -I. -DASM_CRC $(LOC)"\
CF="-O3 -Wall -I. -DASM_CRC -DLARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64 $(LOC)"\

# 保存后下面就是一些编译替换的后续工作了
make -f unix/Makefile linux
which unzip
cp unzip /usr/bin/unzip
{% endhighlight %}

### 关于gcc -D_FILE_OFFSET_BITS=64的描述(找不到原始出处了)
In a nutshell for using LFS you can choose either of the following: Compile your programs with 
"gcc -D_FILE_OFFSET_BITS=64". This forces all file access calls to use the 64 bit variants. 
Several types change also, e.g. off_t becomes off64_t. It's therefore important to always
use the correct types and to not use e.g. int instead of off_t. For portability with other
platforms you should use getconf LFS_CFLAGS which will return -D_FILE_OFFSET_BITS=64 on Linux
platforms but might return something else on e.g. Solaris. For linking, you should use the link 
flags that are reported via getconf LFS_LDFLAGS. On Linux systems, you do not need special link flags.
Define _LARGEFILE_SOURCE and _LARGEFILE64_SOURCE. With these defines you can use the LFS functions like open64 directly. 
Use the O_LARGEFILE flag with open to operate on large files. A complete documentation of the feature test macros like _FILE_OFFSET_BITS
and _LARGEFILE_SOURCE is in the glibc manual (run e.g. "info libc 'Feature Test Macros'").

