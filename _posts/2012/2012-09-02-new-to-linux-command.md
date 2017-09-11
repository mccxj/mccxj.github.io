---
layout: post
comments: true
title: "Linux命令入门(旧)"
description: "09年写的傻瓜文章"
categories: ["linux", "命令", "总结"]
---

__这是09年写的一篇入门文章，内容简单得有点傻瓜了。__
以后再写些实用点的内容，这里只是留个纪念。

## 正文
今天是2009年4月11日，在家听范玮琪的歌，番禺之行再次搁浅。心血来潮，随便写写关于linux的东西。
想想linux的博大精深，对于自己是否找到门还不敢保证，
这次就不自量力，写点简单的，抛砖引玉。方式嘛，还是流水形式，我个人比较懒，对于段落组织太费力，不做也罢。
本来按照惯例应该像颁奖典礼一样弄堆名字，可是我就省略了，只是特别祝愿我们的梓恩细妹快高长大，健康快乐。

先扯扯自己神往linux的经历，在学校的时候，没好好学东西，第一次去实验室，对着linux系统，
那叫哭呀，弄个黑乎乎的界面，仿佛我回到了初中时对着那个DOS的年代。
回去总得自己装个系统吧，redhat9？不料我x环境没弄好，结果又到了黑乎乎的界面，
看别人弄才知道可以用startx可以进去界面的，可惜我的就是会出错，同学说我的x没装好。
我也不知道yum，rpm，就傻傻的重装系统好了。经过一番折腾，我终于找到组织，看到那个"windows"，
可是相当不好用，想想那vi简直是个低级工具呀，rar也不知到哪去解压，没兴趣又跑回winxp去了。
后来硬盘不够了\(60g\)，决定把redhat去掉，结果把硬盘格式了，因为这个事情，有次在面试的时候，说到linux的时候，给人笑话了一番。

到06年底，linux的东西写到简历上都觉得汗颜，有次面试的时候，cat，echo是干嘛的也说不出，现在觉得做人还是要谦虚点。
到了公司，我第一次听到个fedora的系统，那个时候好像是6还是7吧，觉得挺不错的，玩了一下，借助鸟哥的教程还有网上的资料，
学了一些命令，shell，一些服务配置等等，说为什么弄这个，只是那个时候给人扔到一边凉快去了，自己还有点兴趣，就这么摸索起来了。

07年底的样子，学了点java，学了点ruby，学了点linux，算是有点交代，虽然那些对着教程有些自己的操作，知道的linux还是比较杂乱的，
那个时候对现在一直用的centos也是没有概念的。转眼就到08年了，在春节回家前，我到书店去搬了一本linux系统管理技术手册第二版回去，
那个硬皮英文版，花了我100大元，在春节那段时间，我磕磕绊绊的看了一大半，这段时间自己对于linux有个知识的梳理，感觉还是很充实的。
08年开始了，自从小七的到来，偶开始接触centos，也开始了对redhat系列情有独钟，虽然工具是工具，系统也是个工具，可是我就是不想去弄多一套系统，
例如很流行的ubuntu，而且现在感觉centos做桌面版也是挺顺手的。所以还是有点体会的，每个发行版都有长处，但不是最重要的地方，
选择一个资料比较多的发行版，然后坚持下去就好了。至于我选择centos，那是因为redhat是相当流行的系列，服务器上最多人用的，
centos几乎是100%兼容redhat企业版的，所以资料也很多，系统相当稳定有效率，所以就这么看上了。
那ubuntu是最流行的桌面版linux系统，用户体验相当好\(听说的\),资料也很多，选择这个也是不错的。
我定位是更靠近服务器而不是图更方便\(winxp是最方便的系统，哈哈\)，所以我选择centos，现在也觉得相当合适我。

unix的哲学是很有趣的，对linux也是合适的\(这方面可以参考unix编程艺术\)。
小而精，这方面在linux命令表现得淋漓尽致，由很多小命令组合起来的威力有些出人意料，让我们见识到可扩展性需要复杂性支持并不是必然的。
说到这里，极力推荐这本unix编程艺术，这是本通用读物，里边没有教我们具体是怎么编程的，这是win迷的“洗脑”必备工具。

想写的东西没有具体规划，也有可能最后沦落为克隆版，呵呵，不过没关系，一来我没想这是什么大作，二来想着只是作为自己的笔记，来一番复习。
毕竟教程不是最重要的秘籍，是信心，耐心和兴趣，才能让人无畏惧，不断前进。
我参考的资料大体是google到的，还有网上很流行的linux常用命令全集。好了，现在入正题。

有个很重要的概念，就是文件。跟windows里边的文件的概念很不一样，linux里边什么东西都可以看成文件，就算硬件，io，进程这些东西，
也是用一系列文件来表示的。首先我们使用ctrl+alt+f1～f6来切换到文本模式界面，
而用ctrl+alt+f7的话可以切换到xwindow界面去。输入用户名，密码登陆后会出现

    [caixj@localhost ~]$

如果是用root用户登录的话，就是

    [root@localhost ~]#

有个明显的区别就是后面管理员帐号是带\#而普通用户是用$。
这里要强调的是一定要警惕诱惑，不要随便使用root用户登录，这涉及安全问题，所以服务器一般都会禁止root远程ssh登录，
有些桌面版本(ubuntu)会禁止root登录桌面。这里的一个原则就是最小化用户权限。
如果用__ssh__来登录的话，大概就是这样的(可以使用__putty__或者__F-Secure SSH Client_，
后面这个工具不支持utf8等编码格式，所以显示不了中文)：

    [caixj@localhost opt]$ ssh -p22 caixj@192.168.1.46
    caixj@192.168.1.46's password:
    Last login: Sat Apr 11 14:03:42 2009 from 192.168.1.184
    [caixj@localhost ~]$

这里说明的就是p是指端口，默认的ssh端口是22来的，默认会使用所在机器的ssh端口，
例如这个184的机器用的就不是默认的22端口。后面跟着的是用户名和主机ip。
关于ssh服务的配置请看配置/etc/ssh/sshd_config。

登录系统之后，可以使用whoami\(成龙有个电影叫"我是谁"看过么？\)，或者who/id来查看登录信息

    [caixj@localhost hello]$ whoami
    caixj
    [caixj@localhost hello]$ who
    caixj    pts/1        2009-04-11 14:08 (192.168.1.184)
    [caixj@localhost hello]$ id
    uid=500(caixj) gid=500(caixj) groups=500(caixj)
    [caixj@localhost hello]$ id root
    uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel)还可以查看一下系统的一些版本信息，例如使用uname
    [caixj@localhost ~]$ uname -a
    Linux localhost.localdomain 2.6.18-92.1.18.el5 #1 SMP Wed Nov 12 09:30:27 EST 2008 i686 i686 i386 GNU/Linux

如果有需要使用更高级别的用户，可以使用su带用户名来切换到另一个用户：

    [caixj@localhost ~]$ su root
    口令：
    [root@localhost caixj]#

不带用户名的话，是指切换到root，另外，root可以不输入密码切换到任何用户，
要退出的时候，使用exit就可以了\(记得那个win95的dos，我当时就不知是怎么退出的\)。
这里还是提示一下，尽量不使用root用户，这个问题再强调也不过分。这里可以举例说明一下，
大家都使用root登录，那还能区分谁是谁么？你一不小心把重要的东西删除了，怎么办？
我就试过rm -fr /bin就这样系统基本报销了。还有很多服务器如httpd，tomcat，mysql都不能随便使用root用户运行，
一旦服务给入侵了，就等于有了root的权限，这也是安全隐患。

现在我们想创建一个文件夹，可以使用mkdir\(make directory\)

    [caixj@localhost ~]$ mkdir testdir
    [caixj@localhost ~]$ mkdir -p testdir/subdir/subdir2

p参数是如果父文件夹不存在的时候，会自动创建，这对于要建立一个深层次的文件路径的时候非常有用。
如果你要删除某个文件夹，可以使用rmdir(remove directory)

    [caixj@localhost ~]$ rmdir testdir/subdir/subdir2
    rmdir: testdir/subdir/subdir2: 目录非空
    [caixj@localhost ~]$ rm testdir/subdir/subdir2/hello
    [caixj@localhost ~]$ rmdir testdir/subdir/subdir2
    [caixj@localhost ~]$ rm -i testdir/file1
    rm：是否删除 一般文件 “testdir/file1”? n

这里出错是因为rmdir在目录下面为空的情况才会成功，同样可以使用p参数递归删除文件夹。
如果文件夹不为空，可以上面提到使用rm命令。rm命令相当危险，比较常用的命令有f和r，f是force，强制删除的意思，r是
recursive，递归的意思，所以rm -fr /就等于把系统挂掉了。这个命令要慎用！
还有那个i的参数是interactive的意思，交互的意思，就是会提示你一下。
这里说说其他的，我们怎么知道一个命令怎么用，有哪些参数？我们可以求助几个工具，--help,info,man，help是一般的工具自带的帮助，
而info和man则像是通用的帮助系统(info一般是man的补充或者包含更新版本的工具介绍)，使用很简单：

    [caixj@localhost ~]$ man rm
    [caixj@localhost ~]$ info rm
    [caixj@localhost ~]$ rm --help再回到上面的rm，或许你会发现你的系统上rm不带i也会出现提示，那你可以查看一下alias
    [caixj@localhost ~]$ alias
    alias l.='ls -d .* --color=tty'
    alias ll='ls -l --color=tty'
    alias ls='ls --color=tty'
    alias vi='vim'
    alias which='alias | /usr/bin/which --tty-only --read-alias --show-dot --show-tilde'

看看是否存在alias rm='rm -i'的字眼，
alias是个别名机制，用得好或许可以省点事。例如我这样自己手动加一个rm别名，这样的效果就和加i的是一样的了：

    [caixj@localhost ~]$ alias rm='rm -i'
    [caixj@localhost ~]$ rm testdir/file1
    rm：是否删除 一般文件 “testdir/file1”? n
    [caixj@localhost ~]$ unalias rm

同样，可以使用unalias来取消这个别名。这里有个安全建议(出自“linux黑客大曝光”),就是对于alias，只使用无负面影响的别名，
或者不要覆盖原来的命令名，如上面的rm命令，安全的做法是alias del='rm -i'，因为如果覆盖了原来的rm，
那么你转移到一个无别名的机器上的时候，使用rm的时候对系统没有给出提示而感到诧异，
我第一次就是对centos上的rm不提示而感到奇怪。这里在友情推销一下这本linux黑客大曝光，安全方面的经典著作，值得收藏。

貌似现在屏幕已经满了，眼花了，可以使用clear命令来清除屏幕。

    [caixj@localhost testdir]$ clear

现在我们进去一个文件夹看看，可以使用cd(change directory)

    [caixj@localhost ~]$ cd testdir/
    [caixj@localhost testdir]$ cd ..
    [caixj@localhost ~]$ cd -
    /home/caixj/testdir
    [caixj@localhost testdir]$

这里的..是表示进入上一层目录，-是表示返回到上一次的目录。要知道身处哪个目录，可以使用pwd：

    [caixj@localhost ~]$ pwd
    /home/caixj/testdir

现在我们在用ls(list)看看目录下有什么内容

    [caixj@localhost testdir]$ ls -al
    总计 24
    drwxrwxr-x  3 caixj caixj  4096 04-11 15:20 .
    drwx------ 91 caixj caixj 12288 04-11 14:43 ..
    -rw-rw-r--  1 caixj caixj     0 04-11 15:20 file1
    drwxrwxr-x  2 caixj caixj  4096 04-11 14:35 subdir

a和l是非常常用的参数，会列出详细信息并每行只有一个文件或文件夹。
我们要新建一个文件怎么办？

    [caixj@localhost testdir]$ touch file2
    [caixj@localhost testdir]$ ls
    file1  file2  subdir
    [caixj@localhost testdir]$ cat \> file3
    hello,world!
    
    [caixj@localhost testdir]$ more file3
    hello,world
    [caixj@localhost testdir]$ cat file1 \>\> file3
    [caixj@localhost testdir]$ more file3
    hello,world!
    file1's content

touch可以新建一个空文件，但是可以用来更新一个文件的时间戳(以前用来伪造一些数据的时候用过)，
cat也可以用来写文本(>)，或者添加文本\(\>\>\)。更加有效的工具可以求助vi或者gedit这样的可视化工具。
再走一下题：>,>>的操作和|一样是属于管道的概念，把输入输出看成管道里边的水流，|就是把前面命令的输出当成后面命令的输入，
具体的说法，请参考其他资料。

上面也看到了，more是用来查看一个文件内容的工具，而且是一次只显示一部分，适合长文本查看，
类似有less命令，不同的地方就是它支持回滚方式。还有其他几个常用的cat(查看全部),
head（查看前面的）,tail（查看后面的），更多参数找man去吧。有个不错的用法是tail -f filename 
会随着文件内容增加而输出，例如在tomcat启动的时候用来观察一下启动日志。

接下来，ctrl+c,ctrl+v,ctrl+x应该是文件操作的必杀技了，可以使用cp（copy）和mv（move）

    [caixj@localhost testdir]$ ls
    file1  file2  file3  subdir
    [caixj@localhost testdir]$ mv file3 subdir/
    [caixj@localhost testdir]$ ls
    file1  file2  subdir
    [caixj@localhost testdir]$ cp file2 subdir/
    [caixj@localhost testdir]$ ls subdir/
    file2  file1
    [caixj@localhost testdir]$ cp -R subdir hello
    [caixj@localhost testdir]$ ls
    file1  file2  hello  subdir
    [caixj@localhost testdir]$

注意的是，mv可以用来重命名一个文件名，意思就是移动到同个目录的另个文件名去就是了。
还可以移动整个文件夹，cp可以拷贝整个文件夹，不过就要加上R的参数，不然是不会成功的。
另外，如果你是拷贝文件到远程机器，或者从远程机器拷贝文件的话，可以求助scp(Secure Copy)命令

    [caixj@localhost hello]$ scp -P2281 file.tgz caixj@192.168.1.184:/home/caixj
    caixj@192.168.1.184's password:
    file.tgz                                                                                    100%  198     0.2KB/s   00:00
    [caixj@localhost hello]$ scp -P2281 caixj@192.168.1.184:/home/caixj/file.tgz ./
    caixj@192.168.1.184's password:
    file.tgz                                                                                    100%  198     0.2KB/s   00:00
    [caixj@localhost hello]$

其实和ssh的命令一整套的，有个比较特殊的地方是端口用的是大写的P，
小写的p有其他用途了。不过这也不是非常方便，推荐使用gftp这样的图形化工具，
而且这个工具也集成了http/https，ftp/ftps,ssh2等各类协议，使用非常方便。好了，继续回到地球。

批量改后缀名的也不错吧，可以使用rename:

    [caixj@localhost hello]$ ls
    file2.rb  file3.rb  subdir
    [caixj@localhost hello]$ rename .rb .java *
    [caixj@localhost hello]$ ls
    file2.java  file3.java  subdir

不过这个方法我很少用到的，大概是因为linux没有后缀名这个说法，
至于你经常会看到的.zip,.tar.tgz,.log的东西在linux里边只是一种常用约定，而不是用后缀名来决定文件属性的。
这个跟win系列是不一样的。

既然说到这几个后缀名，再说说文件的打包压缩解压操作。.rar的文件是蛮受到歧视的，
因为一般的机器默认不带这个相应的解压工具，当然你可以使用rarlinux来处理，具体做法就不说了。

    [caixj@localhost hello]$ ls
    file2.java  file3.java  subdir
    [caixj@localhost hello]$ tar cvf file.tar file2.java file3.java
    file2.java
    file3.java
    [caixj@localhost hello]$ tar zcvf file.tgz file2.java file3.java
    file2.java
    file3.java
    [caixj@localhost hello]$ tar jcvf file.bz2 file2.java file3.java
    file2.java
    file3.java
    [caixj@localhost hello]$ zip file.zip file2.java file3.java
      adding: file2.java (deflated 99%)
      adding: file3.java (stored 0%)
    [caixj@localhost hello]$ ls
    file2.java  file3.java  file.bz2  file.tar  file.tgz  file.zip  subdir
    [caixj@localhost hello]$ tar tvf file.tgz
    -rw-rw-r-- caixj/caixj   10240 2009-04-11 16:25:56 file2.java
    -rw-rw-r-- caixj/caixj      29 2009-04-11 16:00:50 file3.java

zip和unzip是一对的，并且有个zipinfo来查看zip包的信息。tar就是集大成者，可以不压缩的简单打包，
也可以加上z(gzip)或者j(bzip2)进行压缩处理，解压处理只要把对应的c(create)改成x(extract)就可以了。
现在我们在试试file这个命令

    file2.java: POSIX tar archive
    file3.java: ASCII text
    file.bz2:   bzip2 compressed data, block size = 900k
    file.tar:   POSIX tar archive
    file.tgz:   gzip compressed data, from Unix, last modified: Sat Apr 11 16:26:44 2009
    file.zip:   Zip archive data, at least v2.0 to extract
    subdir:     directory
    subfile:    symbolic link to `subdir/file2

这个命令可以用来识别文件类型，如上面看到的ascii文本，bzip2，tar，文件夹，链接文件等等，
从file2.java也可以看出linux不是靠后缀名来识别文件类型的。

文件这个东西涉及非常多的知识点，我们在下载centos的dvd的时候，
有没有见过网站上还顺便提供了一个md5的值，这就是涉及到文件校验，
如何确保这个文件是你要的那个，而不是给修改过了？这里有很多工具，
如sum,cksum(基于crc),不过这两个都太简单了点，工业级的应该是md5sum(最常用的),
sha1sum(sha有很多版本)，这2个算来的那串值是很难重复的，所以内容具有很难调包。
使用方法很简单也很类似，也可以使用openssl这个工具来做这个：

    [caixj@localhost hello]$ md5sum file.tgz
    28f8084aad424b4187ec404dead6fb95  file.tgz
    [caixj@localhost hello]$ openssl sha1 file.tgz
    SHA1(file.tgz)= 9bafce8ad310aaf8a9384a43613176ad442e6e4f
    [caixj@localhost hello]$ openssl md5 file.tgz
    MD5(file.tgz)= 28f8084aad424b4187ec404dead6fb95

这样我们用这个值和网上公布的值做一下对比就可以了。再走一下话题，
说到这个md5sum虽然挺安全的了，调包内容而保持sum一样几乎是不可能的，
不过也有可能黑客把网页上的sum给调包了。呵呵，扯远了。在回到上面说的file命令，
那个链接文件是什么来的？ln(link)命令的链接文件包括两种，一种就是软链接，一种叫硬链接，如下所示

    [caixj@localhost hello]$ ln -s subdir/file2 subfile
    [caixj@localhost hello]$ ln subdir/file2 subfile2
    [caixj@localhost hello]$ ls -al subfile subfile2
    lrwxrwxrwx 1 caixj caixj 12 04-11 17:15 subfile -> subdir/file2
    -rw-rw-r-- 2 caixj caixj 50 04-11 17:25 subfile2

软链接(s)类似于win里边的快捷方式，而硬链接跟像是一个别名，
效果其实也差不多，就是软链接可以跨档案系统，而硬链接不能。
一般我是使用软链接，硬链接没用过。删除一个链接也很简单，
像文件一样rm掉就可以了，非常方便。

在提到ls -al的时候，出现很多信息，其中包括文件的读写属性，所有者，
所有者使用的组等等。linux的一个文件必须属于某个用户，一个用户可以属于多个用户组，
一个用户组可以包括多个用户。操作一个用户的过程可以使用useradd/adduser,userdel,usermod,
操作用户组的是使用groupadd,roupmod,groupdel(需要root)

    [root@localhost hello]# /usr/sbin/useradd testme
    [root@localhost hello]# passwd testme
    Changing password for user testme.
    New UNIX password:
    BAD PASSWORD: it does not contain enough DIFFERENT characters
    Retype new UNIX password:
    passwd: all authentication tokens updated successfully.
    [root@localhost hello]# /usr/sbin/userdel testme
    [root@localhost hello]# /usr/sbin/groupdel testme

上面的操作就是新建一个testme的用户，并通过passwd更改密码。
最后删除用户，并删除用户组testme。linux增加个用户的时候，
默认会弄个相同名字的用户组，并把这个用户加到这个用户组去，
而删除用户组的时候，只有在用户组下面没有用户的情况下，操作才会成功。
跟用户相关的文件有/etc/passwd,/etc/groups,/etc/shadow。
这里再扯扯一些资料和书籍，比较实用的除了鸟哥的，还有linux新手管理员手册，
Linux系统管理员指南，Linux网络管理员指南。因为没什么资料可以一本万利，
就算linux系统管理技术手册这样的，也得有所侧重，不能面面俱到。

再回过头来说那个文件的属性/所有者/组等等关系。可以使用chgrp(change group)/chown(change owner)，
来修改所有者和用户组。不过，只有文件所有者和root才能更改文件所属的用户组，
并且并且只能用自己所属的用户组，改文件所有者只有root才可以做到。
考虑一下安全问题就可以理解这样的设计方式了。

    [caixj@localhost hello]$ chgrp root file3.java
    chgrp: 正在更改 “file3.java” 的所属组: 不允许的操作
    [caixj@localhost hello]$ chown root file3.java
    chown: 正在更改 “file3.java” 的所有者: 不允许的操作

接下来再看看文件的权限属性-rw-rw-r--，一共有10位，第一位是文件属性，
常见的有-(普通文件),d(文件夹),s(socket),l(链接文件)等等，
后面的每位分成一组，分别代表所有者，所属组，其他用户的权限，
每组从左到右是r(read)w(write)x(execute)的权限。有一些关于文件夹的规则，
例如你有目录的wx权限，你就可以删除文件夹里边的文件，即使你对这个文件没什么权限。
如果你没有目录的x权限，你就不能cd，不能修改增加和删除里边的文件，如果没有w权限，
不能增加和删除里边的文件，但可以修改里边的文件。所以文件的增加和删除是受到文件夹的wx权限限制，
修改文件除了受到文件的w权限限制之外，还受到文件夹的x权限影响。可以使用chmod来修改权限：

    [caixj@localhost testdir]$ chmod u=rwx hello
    [caixj@localhost testdir]$ chmod u-x hello
    [caixj@localhost testdir]$ chmod u+x hello
    [caixj@localhost testdir]$ chmod 775 hello

chmod是非常灵活的，用u(user)，g(group)，o(other)，a(all)来代表各个组，
可以使用+-=来灵活定制，支持使用3位的数字来设定。r，w，x分别代表4，2，1，
权限就几个数字相加就是了。另外,chgrp/chown/chmod也支持-R的递归参数，
不过这个参数也是要谨慎，一不小心可能造成极其严重的后果。

我们使用touch或者mkdir的时候，产生的文件/文件夹有个默认的权限，
这个默认权限可以通过umask命令来设定，也可以用这个命令来查看默认的权限：

    [caixj@localhost testdir]$ umask
    0002
    [caixj@localhost testdir]$ touch tmpfile
    [caixj@localhost testdir]$ mkdir tmpdir
    [caixj@localhost testdir]$ ls -l
    总计 4
    drwxrwxr-x 2 caixj caixj 4096 04-11 20:43 tmpdir
    -rw-rw-r-- 1 caixj caixj    0 04-11 20:43 tmpfile

umask和chmod的操作是相反的效果，umask意思就是没有那些权限的意思，
根据777(文件夹),666(文件)去减去umask就是默认的权限了。
linux默认的配置是比较合理的，像root的umask就是0022，一般也无需做调整。
不过linux文件还有一些特殊的属性可以设置,以加强管理功能。要进行这些操作的话，
可以使用chattr来修改(有些属性是需要root的)，用lsattr来查看这些属性。例如常用的i参数用于防止文件的修改删除：

    [root@localhost testdir]# chattr +i tmpfile
    [root@localhost testdir]# lsattr tmpfile
    ----i-------- tmpfile
    [root@localhost testdir]# rm tmpfile
    rm：是否删除有写保护的 一般空文件 “tmpfile”? y
    rm: 无法删除 “tmpfile”: 不允许的操作

还有a(append)属性只允许增加内容：

    [root@localhost testdir]# chattr -i +a tmpfile
    [root@localhost testdir]# lsattr tmpfile
    -----a------- tmpfile
    [root@localhost testdir]# vi tmpfile
    [root@localhost testdir]# cat > tmpfile
    bash: tmpfile: 不允许的操作
    [root@localhost testdir]# cat >> tmpfile
    xxxxxxxxx
    
    [root@localhost testdir]# more tmpfile
    xxxxxxxxx

这些属性也不是经常会用到，反正我是觉得挺难记得的，每次还是要man一下才行。

再回到上面提到的useradd/userdel/usermod的命令，出现过这么一行

    [root@localhost hello]# /usr/sbin/useradd testme

但是前面很多文件都没有用上全路径的形式，这里就涉及到$PATH这个东西，我们可以使用echo命令来查看

    [caixj@localhost testdir]$ echo $PATH
    /usr/kerberos/sbin:/usr/lib/qt-3.3/bin:/usr/kerberos/bin:/usr/lib/oracle/10.2.0.3/client/lib:/opt/CollabNet_Subversion/bin:/usr/local/grails-1.1/bin:/usr/local/groovy-1.6.0/bin:/usr/local/mysql-5.1.32/bin:/usr/local/wine-1.1.16/bin:/usr/local/scala-2.7.3.final/bin:/usr/local/apache-ant-1.7.1/bin:/usr/java/default/bin:/usr/local/python-2.6.1/bin:/usr/local/ruby-1.8.7-p72/bin:/usr/local/jruby-1.2.0/bin:/usr/local/bin:/bin:/usr/bin:/home/caixj/bin

你甚至可以使用set命令来查看所有的变量。

    [caixj@localhost ~]$ set
    .......
    PATH=/usr/lib/oracle/10.2.0.3/client/lib:/opt/CollabNet_Subversion/bin:/usr/local/grails-1.1/bin:/usr/local/groovy-1.6.0/bin:/usr/local/mysql-5.1.32/bin:/usr/local/wine-1.1.16/bin:/usr/local/scala-2.7.3.final/bin:/usr/local/apache-ant-1.7.1/bin:/usr/java/default/bin:/usr/local/python-2.6.1/bin:/usr/local/ruby-1.8.7-p72/bin:/usr/local/jruby-1.2.0/bin:/usr/lib/qt-3.3/bin:/usr/kerberos/bin:/usr/lib/oracle/10.2.0.3/client/lib:/opt/CollabNet_Subversion/bin:/usr/local/grails-1.1/bin:/usr/local/groovy-1.6.0/bin:/usr/local/mysql-5.1.32/bin:/usr/local/wine-1.1.16/bin:/usr/local/scala-2.7.3.final/bin:/usr/local/apache-ant-1.7.1/bin:/usr/java/default/bin:/usr/local/python-2.6.1/bin:/usr/local/ruby-1.8.7-p72/bin:/usr/local/jruby-1.2.0/bin:/usr/local/bin:/bin:/usr/bin:/home/caixj/bin
    .......

如果不带路径名的时候，就会在$PATH里边寻找这个某个可执行文件，我这个路径没有牵涉到/usr/sbin所以就要自己加上去了。

关于PATH或者其他变量可以在登录的时候设定好，例如在/etc/profile,~/.bash_profile,~/.bashrc里边写好,例如/etc/profile的某个片段：

    RUBY_HOME=/usr/local/ruby-1.8.7-p72
    PATH=$RUBY_HOME/bin:$PATH
    .......
    export RUBY_HOME JAVA_HOME
    export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE INPUTRC

我们这样设置好之后,可以直接使用source命令读取文件命令并在当前shell环境中执行，这样就可以了。

    [caixj@localhost ~]$ source /etc/profile

或者我们直接使用export的命令更新一个变量,或者unset来取消一个变量的定义

    [caixj@localhost ~]$ export xx=333
    [caixj@localhost ~]$ echo $xx
    333
    [caixj@localhost ~]$ export xx=$xx+3
    [caixj@localhost ~]$ echo $xx
    333+3
    [caixj@localhost ~]$ unset xx
    [caixj@localhost ~]$ echo $xx

上面看到的set命令输出是蛮多行的，我们如何迅速定位到PATH呢?这里可以配合管道|和grep命令来找到所有符合某个模式的行，例如

    [caixj@localhost ~]$ set | grep PATH
    LD_LIBRARY_PATH=/usr/lib/oracle/10.2.0.3/client/lib
    PATH=/usr/lib/oracle/10.2.0.3/client/lib:/opt/CollabNet_Subversion/bin:/usr/local/grails-1.1/bin:/usr/local/groovy-1.6.0/bin:/usr/local/mysql-5.1.32/bin:/usr/local/wine-1.1.16/bin:/usr/local/scala-2.7.3.final/bin:/usr/local/apache-ant-1.7.1/bin:/usr/java/default/bin:/usr/local/python-2.6.1/bin:/usr/local/ruby-1.8.7-p72/bin:/usr/local/jruby-1.2.0/bin:/usr/lib/qt-3.3/bin:/usr/kerberos/bin:/usr/lib/oracle/10.2.0.3/client/lib:/opt/CollabNet_Subversion/bin:/usr/local/grails-1.1/bin:/usr/local/groovy-1.6.0/bin:/usr/local/mysql-5.1.32/bin:/usr/local/wine-1.1.16/bin:/usr/local/scala-2.7.3.final/bin:/usr/local/apache-ant-1.7.1/bin:/usr/java/default/bin:/usr/local/python-2.6.1/bin:/usr/local/ruby-1.8.7-p72/bin:/usr/local/jruby-1.2.0/bin:/usr/local/bin:/bin:/usr/bin:/home/caixj/bin
    [caixj@localhost ~]$ set | grep P.*
    GROUPS=()
    HOSTTYPE=i686
    INPUTRC=/etc/inputrc
    KDE_IS_PRELINKED=1
    KDE_NO_IPV6=1
    LD_LIBRARY_PATH=/usr/lib/oracle/10.2.0.3/client/lib
    LESSOPEN='|/usr/bin/lesspipe.sh %s'
    MACHTYPE=i686-redhat-linux-gnu
    .......

现在，我们就可以把一些自己需要的命令加到PATH上去了。
这里又有个常见的安全隐患需要介绍一下，就是在PATH路径添加.(当前目录)，
这样一来./hello，就可以用hello来代替了。看起来的确是不错，可是这不是安全的做法，
例如别有用心的用户或黑客在/tmp目录上弄个hack过的ls的脚本，
root不小心在/tmp目录下执行个ls就可能出现安全问题了。所以推荐不添加.来节省一点点的方便而留个安全隐患。

好了，现在的问题变成，我们怎么知道useradd命令在/usr/sbin目录里边呢?
linux也提供了一些工具便于查找，常用的有which,whereis,locate，
当然有更加强大的find(以后在介绍)，前面几个就很够用了。

    [caixj@localhost ~]$ whereis ruby
    ruby: /usr/bin/ruby /usr/lib/ruby /usr/share/man/man1/ruby.1.gz
    [caixj@localhost ~]$ which ruby
    /usr/local/ruby-1.8.7-p72/bin/ruby
    [caixj@localhost ~]$ locate useradd
    /etc/default/useradd
    /mnt/d/fisheye-1.3.5/content/web-inf/classes/org/apache/jsp/WEB_002dINF/jsp/admin/useradd_jsp.class
    /mnt/d/fisheye-1.3.5/content/web-inf/jsp/admin/useradd.jsp
    /usr/sbin/luseradd
    /usr/sbin/useradd
    /usr/share/man/fr/man8/useradd.8.gz
    /usr/share/man/id/man8/useradd.8.gz
    /usr/share/man/it/man8/useradd.8.gz
    /usr/share/man/ja/man8/useradd.8.gz
    /usr/share/man/man1/luseradd.1.gz
    /usr/share/man/man8/useradd.8.gz
    /usr/share/man/pl/man8/useradd.8.gz
    /usr/share/man/ru/man8/useradd.8.gz
    /usr/share/man/tr/man8/useradd.8.gz
    /usr/share/man/zh_CN/man8/useradd.8.gz
    /usr/share/man/zh_TW/man8/useradd.8.g

有一些区别，which是在PATH里边找的，所以对前面的需求来说，是没什么用处的。
whereis是可以设定在一定目录上找，是我用得最多的工具。
而locate是基于预先建立的一个包括系统内所有档案名称及路径的数据库，
一般是通过cron任务来定时更新这个数据库，可以搜索到非常详细的内容，不过不是实时的。

快12点了，今天到此为止，关机睡觉。说起关机呢，对于linux来说，并不是很常用，
服务器几年都不关机一次那是家常便饭。不过这里还是要讲讲，关机的命令也不少，
shutdown,halt,reboot是我用得最多的。shutdown是包括很多选项，包含了reboot和halt的功能。
这些命令都需要root权限，具体是怎么操作就不演示了，不然我的文档没法写了。

下课了，待续。

2008年4月12日，一大早外边就在吵，等我爬起来又不吵了，做人真不厚道。洗衣服后刚刚开机，忘了现在是几点了怎么办？
可以用date来查看(其实看手机也挺方便:))

    [caixj@localhost ~]$ date

2009年 04月 12日 星期日 07:57:31 CSTdate同样也可以用来调整时间，服务器时间也是很重要的因素，
特别要保证集群服务器的时间同步，要不同一时间操作的2条数据就可能时间差好远。另外，也可以看看日历，
虽然这个cal命令很少用到(看来我也没什么时间概念)：

    [caixj@localhost ~]$ cal
         四月 2009  	
    日 一 二 三 四 五 六
       1    2   3   4
     5    6    7   8   9  10  11
    12  13  14 15  16  17 18
    19  20  21 22  23 24 25
    26  27 28 29  30

那怎么知道最近有那些人登录过呢？可以使用last来看看，它会记录系统开机到现在的登录信息：

    [caixj@localhost testdir]$ last|head
    caixj    pts/1        192.168.1.184    Sun Apr 12 07:57   still logged in   
    caixj    pts/1        192.168.1.184    Sat Apr 11 21:36 - 23:56  (02:20)	
    caixj    pts/1        192.168.1.184    Sat Apr 11 14:08 - 21:35  (07:26)	
    caixj    pts/1        192.168.1.184    Sat Apr 11 14:03 - 14:08  (00:04)	
    root     pts/1        192.168.1.184    Sat Apr 11 14:01 - 14:03  (00:01)	
    caixj    pts/1        192.168.1.184    Sat Apr 11 13:52 - 14:01  (00:08)	
    caixj    pts/1        192.168.1.184    Sat Apr 11 12:45 - 12:45  (00:00)	
    caixj    pts/1        :0.0             Fri Apr 10 16:29 - 16:32  (00:02)	
    caixj    pts/1        :0.0             Fri Apr 10 16:26 - 16:28  (00:02)	
    caixj    pts/1        :0.0             Fri Apr 10 16:07 - 16:08  (00:01)   

这也是为什么不提倡使用root的原因，不然清一色的root，看得过来么？从左到右分别是登录用户，终端，登录地址，登录时间段，登录时间。

有些人是软件狂，什么软件都想试试，那么总得知道linux下面怎么安装软件吧？
像很多发行版都自带了自己的一套软件包管理器，例如rpm(RedHat Package Manager),deb。
对于redhat系列，还有个基于rpm的前端软件包管理工具，叫yum(Yellow dog Updater, Modified)，是相当的方便，可以自动包依赖关系。

    [root@localhost testdir]# yum install ruby*
    Loaded plugins: fastestmirror
    Loading mirror speeds from cached hostfile
     * epel: mirror.yandex.ru
    base                                                                                                   | 1.1 kB     00:00 	
    updates                                                                                                |  951 B     00:00 	
    primary.xml.gz                                                                                         |  99 kB     00:00 	
    updates                                                        150/150
    addons                                                                                                 |  951 B     00:00 	
    extras                                                                                                 | 1.1 kB     00:00 	
    Setting up Install Process
    Parsing package install arguments
    Package ruby-1.8.5-5.el5_2.6.i386 already installed and latest version
    Package ruby-libs-1.8.5-5.el5_2.6.i386 already installed and latest version
    Resolving Dependencies
    --> Running transaction check
    ---> Package rubygem-rake.noarch 0:0.8.3-1.el5 set to be updated
    ---> Package rubygem-mongrel.i386 0:1.0.1-6.el5 set to be updated
    .......
    
    [root@localhost testdir]# yum update
    Loaded plugins: fastestmirror
    Loading mirror speeds from cached hostfile
     * epel: ftp.yz.yamagata-u.ac.jp
    Setting up Update Process
    Resolving Dependencies
    --> Running transaction check
    ---> Package mono-core.i386 0:1.2.6-6.1.el5 set to be updated
    ---> Package NetworkManager.i386 1:0.7.0-4.el5_3 set to be updated
    .......
    
    [root@localhost caixj]# yum erase ruby*
    Loaded plugins: fastestmirror
    Setting up Remove Process
    Resolving Dependencies
    --> Running transaction check
    ---> Package ruby.i386 0:1.8.5-5.el5_2.6 set to be erased
    --> Processing Dependency: ruby >= 1.8 for package: kdebindings
    ---> Package ruby-libs.i386 0:1.8.5-5.el5_2.6 set to be erased

yum还带有其他很多参数，
我基本就靠上面几个过活，另外clean参数也是比较常用的。直接用rpm安装也是很方便的。

    [root@localhost caixj]# rpm -ivh xxx.rpm
    error: open of xxx.rpm failed: 没有那个文件或目录
    [root@localhost caixj]# rpm -Uvh xxx.rpm
    error: open of xxx.rpm failed: 没有那个文件或目录
    [root@localhost caixj]# rpm -qa | grep openssl
    openssl-0.9.8e-7.el5
    openssl-devel-0.9.8e-7.el5

当然最好是使用yum来安装，省事又省心，具体的yum源的配置参考/etc/yum.repos.d/\*.repo并参考其他资料。

话说回来，有时候没有现成的rpm包，或者rpm很旧(看上面centos下的ruby才1.8.5)，
又或者存在潜在的性能问题(前段日子报道的ubuntu的ruby的性能问题)。
当然也有可能是工作学习需要(需要测试多个软件版本)，或者压根就是个软件狂。
在linux下是免不了要用到源码安装的，跟windows下安装软件相比，安装多个版本简单得多，而且不容易发生冲突。

首先，我们要下载源码包,一般的源码包是上面提到那几个压缩格式。可以使用wget(web get)来下载,

    [caixj@localhost testdir]$ export http_proxy=http://user:paswd@192.168.1.184
    [caixj@localhost testdir]$ wget http://www.lighttpd.net/download/lighttpd-1.4.22.tar.gz
    .......
    [caixj@localhost testdir]$ tar zxvf lighttpd-1.4.22.tar.gz
    .......
    [caixj@localhost testdir]$ cd lighttpd-1.4.22

上面第一行，是因为网络有限制，设置个代理而已:).接下去的工作基本是标准流程(有些不一样的就要自己参考文档了)：

    [root@localhost caixj]# .configure --prefix=/usr/local/lighttpd-1.4.22
    [root@localhost caixj]# make
    [root@localhost caixj]# make install

基本上都是这样，configure推荐带上prefix的参数，不然版本不好控制，
卸载也不容易。make和make install，很多时候可以写成make && make install。

最后当你安装(yum或源码)之后，会不会觉得每次启动都要手动一把很无聊呢？
例如我的mysql每天都要用，像windows那样当成服务启动就好了。
linux也有开机启动这样的功能，具体怎么把一个程序弄成服务自动启动请自行google,redhat系列的基本流程是写个包括start/stop/restart的一定格式的脚本,
然后扔到/etc/init.d(这是/etc/rc.d/init.d的一个soft link)，最后再使用chkconfig来配置.例如我们已经弄了个httpd的脚本

    [root@localhost ~]# ls -al /etc/init.d/ | grep httpd
    -rwxr-xr-x  1 root root  3200 01-22 11:05 httpd
    [root@localhost ~]# /sbin/chkconfig --add httpd
    [root@localhost ~]# /sbin/chkconfig --list httpd
    httpd           0:关闭  1:关闭  2:关闭  3:关闭  4:关闭  5:关闭  6:关闭
    [root@localhost ~]# /sbin/chkconfig --level 3 httpd on
    [root@localhost ~]# /sbin/chkconfig --list httpd
    httpd           0:关闭  1:关闭  2:关闭  3:启用  4:关闭  5:关闭  6:关闭

0－6是指不同的运行状态,具体参考/etc/inittab，关闭/启动表示在某个运行状态下该服务是否自动启动。
假如你想调整某些运行状态的服务，用chkconfig一个个调整不是很方便，这时可以求助ntsysv:

     [root@localhost ~]# /usr/sbin/ntsysv
     [root@localhost ~]# /usr/sbin/ntsysv --level 35

不加level参数就表示当前运行状态。
会出现一个图形化的界面，只需设置并保存就可以了。
最后，要让某个关闭的服务启动或者正在运行的服务关闭,可以使用service方法,例如

     [root@localhost ~]# /sbin/service httpd start
     启动 httpd：                                               [确定]
     [root@localhost ~]# /sbin/service httpd stop
     停止 httpd：                                               [确定]

程序总算启动了，可是一切还没完。我们怎么能看到这些程序的进程呢？
或者突然你机器慢得不行了，想知道那个程序不知好歹？
你会不会怀念windows那个ctrl+alt+del的任务管理器呢？还是忘了它吧，
当真的很卡的时候，任务管理器都自身难保了！再来看看linux的进程管理工具ps和top

    [caixj@localhost ~]$ ps aux
    USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
    root         1  0.0  0.0   2068   592 ?        Ss    2008   1:19 init [3]     	
    root         2  0.0  0.0      0     0 ?        S<    2008   0:00 [migration/0]
    root         3  0.0  0.0      0     0 ?        SN    2008   0:00 [ksoftirqd/0]
    root         4  0.0  0.0      0     0 ?        S<    2008   0:00 [watchdog/0]
    root         5  0.0  0.0      0     0 ?        S<    2008   0:00 [migration/1]
    root         6  0.0  0.0      0     0 ?        SN    2008   0:00 [ksoftirqd/1]
    root         7  0.0  0.0      0     0 ?        S<    2008   0:00 [watchdog/1]
    root         8  0.0  0.0      0     0 ?        S<    2008   0:00 [events/0]
    root         9  0.0  0.0      0     0 ?        S<    2008   0:00 [events/1]
    root        10  0.0  0.0      0     0 ?        S<    2008   0:00 [khelper]

昨天下午开始就没干活了，今天是4月13号，刚刚看完倾城之恋，继续开工！

ps可以用来获取进程的瞬时状态，也有很多选项可供选择。如上面所示，USER表示运行该进程的用户，
PID是process id就是进程id，1一般就是init，这是系统启动的时候第一个运行的程序。%CPU，
就是占用的CPU时间，是很重要的数据项，当你的系统反应缓慢的时候，或许就是有个进程占用了大部分的CPU。
%MEM就是占用的物理内存使用率，VSZ是占用的虚拟内存大小，RSS是占用的内存大小，TTY是终端号，
STAT就是运行状态，特别需要注意的是Z这样的僵尸进程(已经运行完了却没有消除)，
START和TIME分别是运行开始时间和运行的时间，COMMAND就是执行的命令。

一般都会有很多行，所以经常配合grep进行过滤，例如查找java相关的,ps aux | grep java，
还有经常用的就是查找某个用户执行的进程状态,ps ux -U caixj,更多的选项还是要用man查阅，
不过这也基本够用了。再看看下面是用lax参数的情况：

    [caixj@localhost ~]$ ps lax
    F   UID   PID  PPID PRI  NI    VSZ   RSS WCHAN  STAT TTY        TIME COMMAND
    4     0     1     0  15   0   2068   592 -      Ss   ?          1:19 init [3]     	
    1     0     2     1 -100  -      0     0 -      S<   ?          0:00 [migration/0]
    1     0     3     1  34  19      0     0 -      SN   ?          0:00 [ksoftirqd/0]
    5     0     4     1 -100  -      0     0 -      S<   ?          0:00 [watchdog/0]
    1     0     5     1 -100  -      0     0 -      S<   ?          0:00 [migration/1]
    1     0     6     1  39  19      0     0 -      SN   ?          0:00 [ksoftirqd/1]
    5     0     7     1 -100  -      0     0 -      S<   ?          0:00 [watchdog/1]
    1     0     8     1  10  -5      0     0 -      S<   ?          0:00 [events/0]
    1     0     9     1  10  -5      0     0 -      S<   ?          0:00 [events/1]
    1     0    10     1  10  -5      0     0 -      S<   ?          0:00 [khelper]

这个有几列比较重要的数据是PPID，parent process id，父进程ID，PRI是priority优先级，
NI是nice优先权。这几个选项还是有一定用处的。不过先来看看top命令是怎样的？

    [caixj@localhost ~]$ top
    top - 00:23:13 up 117 days,  7:42,  1 user,  load average: 0.01, 0.03, 0.01
    Tasks: 128 total,   1 running, 127 sleeping,   0 stopped,   0 zombie
    Cpu(s): 33.3%us, 33.3%sy,  0.0%ni, 33.3%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
    Mem:   3115980k total,  3016584k used,    99396k free,    83396k buffers
    Swap:  5164856k total,    65684k used,  5099172k free,  2022860k cached
    
      PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
    15346 caixj     15   0  2196 1008  812 R  0.3  0.1   0:00.01 top
        1 root      15   0  2064  580  536 S  0.0  0.1   0:00.35 init
        2 root      RT  -5     0    0    0 S  0.0  0.0   0:00.00 migration/0
        3 root      34  19     0    0    0 S  0.0  0.0   0:00.00 ksoftirqd/0
        4 root      RT  -5     0    0    0 S  0.0  0.0   0:00.00 watchdog/0
        5 root      10  -5     0    0    0 S  0.0  0.0   0:00.44 events/0
        6 root      10  -5     0    0    0 S  0.0  0.0   0:00.00 khelper
        7 root      20  -5     0    0    0 S  0.0  0.0   0:00.00 kthread
       10 root      10  -5     0    0    0 S  0.0  0.0   0:02.27 kblockd/0 

top是一个实时的进程监控工具，默认是10秒更新一次状态。可以看到输出的信息非常丰富。简单解析一下，第一行和uptime命令的内容大同小异：

    [caixj@localhost ~]$ uptime
     23:54:11 up 3 days,  7:48,  1 user,  load average: 0.00, 0.01, 0.00

除了列出当前时间，系统运行的时间，用户数，还有很有价值的系统负载参数，分别是最近1分钟，5分钟，15分钟的系统负载平均值。
以某个值(如3)为基准，当值过高的时候，就需要研究对策加以处理了。
第二行是任务统计，还是特别强调一下zombie，就是上面提到的僵尸进程，其实是不占用什么进程的。
第三行描述CPU周期是如何使用的。从左到右，us(user)用户模式，sy(system)内核模式,ni(nice)用户进程空间内改变过优先级的进程，
id(idle)处于空闲的，wa(iowait)IO等待，hi(hard irq)硬件中断,si(soft irq)软件中断，st(???不清楚,谁来告诉一下)。
特别注意的是wa，当系统运行缓慢的时候，是否是IO瓶颈?
第四行显示物理内存的使用情况，包括总的可以使用的内存、已用内存、空闲内存、缓冲区占用的内存。
第五行显示交换分区使用情况，包括总的交换分区、使用的、空闲的和用于高速缓存的大小。
和windows的不一样，物理内存剩余很少是正常的，因为linux会最大化利用内存，并不代表物理内存不足了。
后面的很多和ps是差不多的，只是说说其中几个，VIRT是使用的虚拟内存的总量，VIRT = SWAP + RES.SWAP是被交换出去的部分，
RES是进程占用的物理内存值，RES = CODE + DATA，CODE是运行的代码，DATA是运行的代码使用的数据和堆栈空间大小，SHR是进程使用的共享内存值。

假如某个进程失控了，占用了大量的CPU和内存，我们想把它“结束任务”，可以使用kill，只要找到相应的PID就可以了，例如

    [caixj@localhost ~]$ kill -9 3227

其中的9是信号值，就是KILL，这个信号不能被拦截，
能够保证进程被杀死。不过对于上面提到的僵尸进程是没什么效果的。要去掉它，
可以使用ps lax找到进程的PPID，先把父进程杀掉才行，如果父进程是1，就是init，那么对不起，只能等重启了。

还有，那个top命令涉及到了内存这个东西，我们可以使用free来查看系统内存状况

    [caixj@localhost ~]$ free -m
                                total       used       free     shared    buffers     cached
    Mem:                       978        332        646          0         23        204
    -/+ buffers/cache:     103        874
    Swap:                    2047         13       2034

带m参数是为了用MB为单位而已。上面提到，在linux中认为内存不用白不用，
因此它尽可能的cache和buffer一些数据，所以空闲内存=free+buffers+cached。
如果还要更加详细的内容，可以使用下面的命令(linux设备也是文件)

    [caixj@localhost ~]$ cat /proc/meminfo
    MemTotal:      1001956 kB
    MemFree:        660660 kB
    Buffers:         25032 kB
    Cached:         209904 kB
    SwapCached:       2544 kB
    Active:         126664 kB
    Inactive:       133652 kB
    .......

那就再多说几句，对于某个进程PID，我们可以通过查看文件的方式来了解它的内存使用情况，例如
显示进程所占用的虚拟地址(pid为进程号) 

    #cat /proc/pid/maps

显示进程所占用的内存情况(pid为进程号) 

    #cat /proc/pid/status

我们的结论就是，/proc就是内存的映射,就是是top，也是来这里找内存信息的。

很晚了，明天还要上班，睡觉好了。不过，先写个倾城之恋的影评先。

周二了，影评还没写，继续弄这个。

关于内存，还有一个命令vmstat，是用来实时查看内存使用情况的，不带参数的话，显示的是当前的情况。
可以在后面加个时间间隔来采集数据，例如每3秒采集一次

    [caixj@localhost updates]$ vmstat 3
    procs -------memory--------swap---io----system-- -----cpu------
     r  b      swpd   free   buff  cache     si   so    bi    bo   in   cs us        sy id wa st
     0  0      70860  16048   3688  98676    0    4    68    60     1167 1096 14     2 82  2  0
     1  0      70860  16048   3688  98680    0    0     0      5      1202 2248 25    2 73  0  0
     0  0      70860  16080   3696  98672    0    0     0      16     1167 1309 41     2 57  0  0
     2  0      70860  16112   3696  98680    0    0     0      44    1170 1360 44      1 54  0  0
     0  0      70860  16112   3704  98672    0    0     0       5     1159 1178 22      1 77  0  0
     2  0      70860  16020   3704  98680    0    0     0      0     1156 1189 30      1 68  0  0
     5  0      70860  16020   3716  98668    0    0     0      71     1171 1190 31      1 68  0  0
     0  0      70860  16020   3716  98680    0    0     0       0    1163 1143 22       1 77  0  0

担心解释不好各列的意思，还是直接上英文吧：

    Procs
        r: The number of processes waiting for run time.
        b: The number of processes in uninterruptible sleep.
    Memory
        swpd: the amount of virtual memory used.
        free: the amount of idle memory.
        buff: the amount of memory used as buffers.
        cache: the amount of memory used as cache.
        inact: the amount of inactive memory. (-a option)
        active: the amount of active memory. (-a option)
    Swap
        si: Amount of memory swapped in from disk (/s).
        so: Amount of memory swapped to disk (/s).
    IO
        bi: Blocks received from a block device (blocks/s).
        bo: Blocks sent to a block device (blocks/s).
    System
        in: The number of interrupts per second, including the clock.
        cs: The number of context switches per second.
    CPU
        These are percentages of total CPU time.
        us: Time spent running non-kernel code. (user time, including nice time)
        sy: Time spent running kernel code. (system time)
        id: Time spent idle. Prior to Linux 2.5.41, this includes IO-wait time.
        wa: Time spent waiting for IO. Prior to Linux 2.5.41, included in idle.
        st: Time stolen from a virtual machine. Prior to Linux 2.6.11, unknown.

上次提到内存，现在就再来看看硬盘，先使用命令fdisk来看看系统的分区表信息，如下(需要root)

    [root@localhost caixj]# fdisk -l
    
    Disk /dev/sda: 36.4 GB, 36401479680 bytes
    255 heads, 63 sectors/track, 4425 cylinders
    Units = cylinders of 16065 * 512 = 8225280 bytes
    
       Device Boot      Start         End      Blocks   Id  System
    /dev/sda1   *           1          13      104391   83  Linux
    /dev/sda2              14        1543    12289725   83  Linux
    /dev/sda3            1544        2574     8281507+  83  Linux
    /dev/sda4            2575        4425    14868157+   f  W95 Ext'd (LBA)
    /dev/sda5            2575        3217     5164866   82  Linux swap / Solaris
    /dev/sda6            3218        4425     9703228+  83  Linux

/dev/hd*是IDE硬盘，而/dev/sd* 是SCSI硬盘，a,b,c...这些是指第几块硬盘。
1,2,3就是硬盘的分区。和vmstat类似的磁盘吞吐量工具有iostat，并可以使用间隔和计数参数，如

    [caixj@localhost ~]$ iostat 5 3
    Linux 2.6.18-128.1.6.el5 (localhost.localdomain)        2009年04月14日
    
    avg-cpu:  %user   %nice %system %iowait  %steal   %idle
              15.27    0.00    1.91    2.43    0.00   80.40
    
    Device:            tps   Blk_read/s   Blk_wrtn/s   Blk_read   Blk_wrtn
    hda              10.25       158.19       143.35    5941103    5383713
    hda1              0.00         0.02         0.00        858          0
    hda2              0.00         0.00         0.00         26          0
    hda5              0.00         0.02         0.00        675          0
    hda6              0.00         0.02         0.00        712          0
    hda7              0.01         0.16         0.00       6036         33
    hda8              2.82        59.62        55.55    2239234    2086152
    hda9              7.41        98.32        87.80    3692594    3297528

其中的Blk列是512字节块的意思，avg-cpu的信息基本上和top一样的字段，这里就不做解释了。

接下去用df(disk file)来看看系统的系统硬盘空间的使用情况。

    [caixj@localhost ~]$ df
    文件系统               1K-块        已用     可用 已用% 挂载点
    /dev/sda3              8022104   6296016   1312016  83% /
    /dev/sda1               101086     21729     74138  23% /boot
    tmpfs                  1557988         0   1557988   0% /dev/shm
    /dev/sda2             12096756  11042992    439280  97% /opt
    /dev/sda6              9549556   3213036   5948392  36% /sdb

硬盘的某个分区可以挂载到某个目录上去，其中的tmpfs是交换分区来的。
可以通过可用空间，空间使用率来监控硬盘空间是否足够。一般来说，如果你是双系统的话，
windows分区在Linux系统中默认是不会自动挂载的(有些发行版已经是自动识别的了)。
你可以用fdisk命令查看system列为W95 FAT32的分区，然后所以mount命令来挂载一个文件系统，如

    /dev/hda7            6689        9964    26314438+   b  W95 FAT32
    ........
    [root@localhost caixj]# mount -t vfat /dev/hda7 /mnt/f

mount支持很多文件类型，
如vfat,ramfs,iso9660(光驱),ext2/3之类，更多信息参考man。所以例如移动硬盘，U盘，
光盘这些都是要mount进去才能使用的。你也可以把配置写到/etc/fstab里边去，如

    [root@localhost caixj]# more /etc/fstab
    .......
    /dev/hda5               /mnt/d                  vfat    defaults        0 0
    /dev/hda6               /mnt/e                  vfat    defaults        0 0
    /dev/hda7               /mnt/f                  vfat    defaults        0 0

相反，要卸载一个文件系统，只是简单的使用umount就可以了

    [root@localhost caixj]# umount /mnt/f

需要注意的是，像光盘之类，需要先umount再取出光盘，而不会像windows那样弹出就自动消失了。

如果df之后发现某个文件系统空间突然紧张了，这个时候你需要知道究竟是什么东西占用了。
这个时候进入某个文件系统后，我们可以用tree来查看整个目录结构

    [caixj@localhost ~]$ tree updates/
    updates/
    |-- datasheets
    |   |-- edit.rhtml
    |   |-- index.rhtml
    |   |-- new.rhtml
    |   `-- show.rhtml
    |-- datasheets_controller.rb
    |-- field.rb
    |-- fields
    |   |-- edit.rhtml
    |   |-- index.rhtml
    |   |-- new.rhtml
    |   `-- show.rhtml
    `-- fields_controller.rb
    
    2 directories, 11 files

不过这个工具用处不大，一般文件结构太复杂，而且也没有显示占用空间。
这个时候应该使用du(disk usage)

    [caixj@localhost updates]$ du -h .
    20K     ./datasheets
    20K     ./fields
    64K     .

这里的h参数是human，人性化格式的意思。但是如果文件层次很多，还是会出现很多行，并且统计起来还是很久。
其实我们大部分时候只是想要类似文件夹属性看到文件夹大小的功能，
这个时候可以使用--max-depth=n的参数，n是指目录层次，一般1或者2就够了。

有时候我们想在某个文件系统中查找某个文件，虽然有locate，但是它不是实时的，这个时候或许可以求助功能强大的find命令，如

    [caixj@localhost ~]$ find . -name style.xml -type f
    ./style.xml

其中的name是文件名，type是查找的文件类型，还有其他很多参数，关于这个可以参考man，
这里有一个文章也是写的很好，可以参考：http://www.oracle.com/technology/global/cn/pub/articles/calish-find.html

到这里，已经出现了不少命令了，虽然命令是需要多用用，才能熟悉的。
如果万一对以前用过的命令记不起来了，或者不想再敲打一遍，这个时候你需要history来提供点帮助

    [caixj@localhost ~]$ history
    .......
      994  su
      995  exit
      996  ssh -p2281 caixj@192.168.1.184

你可以结合grep进行过滤。例如上面的ssh那个命令，我想执行，可以直接使用!996。
另外，你也可以使用c(clear)参数进行清理，这样可以避免给偷窥到你敲打过的命令。

    [caixj@localhost ~]$ history -c

没想到一放下就一个星期了，又是周末了。加紧继续干活，假如现在有个文本：

    [caixj@localhost ~]$ more book
    mysql
    spring
    linux
    linux
    spring
    hibernate
    oracle
    lighttpd
    mysql
    mysql

可以用wc(word count)来统计一下文本行数，词数，字符数等等

    [caixj@localhost ~]$ more book | wc -l
    10
    [caixj@localhost ~]$ more book | wc
         10      10      70
    [caixj@localhost ~]$ more book | wc -lw
         10      10
    [caixj@localhost ~]$ more book | wc -lwc
         10      10      70

可是10行里边有重复了，能不能知道不同的有多少个?试试uniq(unique)

    [caixj@localhost ~]$ cat book | uniq
    mysql
    spring
    linux
    spring
    hibernate
    oracle
    lighttpd

mysql可是还是有重复的，uniq不能跨行识别重复的，这个时候需要先sort一下

    [caixj@localhost ~]$ cat book | sort | uniq -c
          1 hibernate
          1 lighttpd
          2 linux
          3 mysql
          1 oracle
          2 spring

来个稍稍复杂点，统计一下你最常用的10个命令

    [root@localhost caixj]# history | awk '{print $2}' | sort | uniq -c | sort -nr | head -10
         24 history
         12 ls
         11 exit
          8 cd
          6 ssh
          6 mount
          5 cat
          4 umount
          3 ps
          3 man

上面有些参数也是man过来，大概比较怪异的就那个awk命令了。
awk是一个相当复杂的文本编辑命令，功能强大，说实话，我只好一点最简单的，
再复杂我会用其他脚本语言来处理。awk可以对某些行的各列进行处理，
这里的列是指特定的分隔符来区分的，用F来指定，例如查看passwd文件来看有多少用户，
还可以用NR来指定对某些行有效

    [root@localhost caixj]# awk -F: '{print $1}' /etc/passwd
    root
    bin
    daemon
    adm
    lp
    sync
    shutdown
    halt
    mail
    [root@localhost caixj]# awk -F: 'NR==3,NR==5 {print $1}' /etc/passwd
    daemon
    adm
    lp

总体来说，这个命令还是不是很好学习的。有兴趣的话，多参考其他资料吧。现在我们来看看下面这2个命令

    [root@localhost caixj]# ps axu | grep mongrel | awk '{print "kill -9",$2}' | sh
    [root@localhost caixj]# lsof -i :80 | grep -v "PID" | awk '{print "kill -9",$2}'| sh

其中sh就是用来执行一些命令的子shell。第一句是用来杀死所有的mongrel进程的，
不过需要确认是不是所有的进程，别把不是的也杀掉了，特别用来那些进程已经无法控制，
要强制杀死而且数量还不少的时候，例如做mongrel cluster的时候，手动一个个杀掉不是个什么好办法。
第二句是类似的，是用在端口给占用的时候，例如上面的是apache httpd占用的80端口，
就是占用80端口的那些进程要给杀掉。两个手段其实是差不多的效果。

现在来看看这个lsof(list open files)，一般有以下用法：
找出正在使用某个文件的进程

    [root@localhost caixj]# lsof /var/log/messages

用来解除阻塞

    [root@localhost caixj]# lsof /mnt/f

搜索打开的网络

    [root@localhost caixj]# lsof -i@192.168.1.184

找出程序打开的所有文件

    [root@localhost caixj]# lsof -p 6800

说起打开的所有文件，一个进程所能打开的文件是有限制的。这个可以通过命令ulimit来查看和设置

    [root@localhost ~]# ulimit -a
    core file size          (blocks, -c) 0 # 设定core文件的最大值，单位为区块
    data seg size           (kbytes, -d) unlimited # 程序数据节区的最大值，单位为KB
    scheduling priority             (-e) 0  # 最大的任务优先值(nice)
    file size               (blocks, -f) unlimited # # shell所能建立的最大文件，单位为区块
    pending signals                 (-i) 32255 # 未结束的信号的最大数量
    max locked memory       (kbytes, -l) 32 # 可锁定在内存中的最大值，单位为KB
    max memory size         (kbytes, -m) unlimited # 可使用内存的上限，单位为KB
    open files                      (-n) 8192 # 同一时间最多可开启的文件数
    pipe size            (512 bytes, -p) 8 # 管道缓冲区的大小，单位512字节
    POSIX message queues     (bytes, -q) 819200 # POSIX的消息队列的最大字节
    real-time priority              (-r) 0 # 实时任务优先级的数值
    stack size              (kbytes, -s) 10240 # 栈的上限，单位为KB
    cpu time               (seconds, -t) unlimited # CPU使用时间的上限，单位为秒
    max user processes              (-u) 2047 # 用户最多可开启的程序数目
    virtual memory          (kbytes, -v) unlimited # 可使用的虚拟内存上限，单位为KB
    file locks                      (-x) unlimited #文件锁的数量

再回过头来看看网络相关的命令，首先查看一下网络配置，类似ipconfig的命令是ifconfig

    [root@localhost caixj]# ifconfig eth0
    eth0      Link encap:Ethernet  HWaddr 00:D2:55:DF:07:73  
              inet addr:192.168.1.184  Bcast:192.168.1.255  Mask:255.255.255.0
              inet6 addr: fe80::202:55ef:fedf:773/64 Scope:Link
              UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
              RX packets:136087020 errors:0 dropped:0 overruns:0 frame:0
              TX packets:60190414 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:1000
              RX bytes:2454973828 (2.2 GiB)  TX bytes:2764993762 (2.5 GiB)
              Interrupt:185ifconfig

也可以用来设置网络，例如

    [root@localhost ~]# ifconfig eth0 down
    [root@localhost ~]# ifconfig eth0 192.168.1.99 broadcast 192.168.1.255 netmask 255.255.255.0
    [root@localhost ~]# ifconfig eth0 up
    [root@localhost ~]# ifconfig eth0

如何查看或检测网络状态呢？最普通的ping可以用来试试，如

    [root@localhost caixj]# ping 192.168.1.63
    PING 192.168.1.63 (192.168.1.63) 56(84) bytes of data.
    64 bytes from 192.168.1.63: icmp_seq=1 ttl=64 time=1.27 ms
    64 bytes from 192.168.1.63: icmp_seq=2 ttl=64 time=0.314 ms
    64 bytes from 192.168.1.63: icmp_seq=3 ttl=64 time=0.294 ms

再复杂的命令有netstat,如检测正在监听的端口，包括t(tcp),u(udp)

    [root@localhost caixj]# netstat -ntul
    Active Internet connections (only servers)
    Proto Recv-Q Send-Q Local Address               Foreign Address             State  	
    tcp        0      0 0.0.0.0:3938                0.0.0.0:*                   LISTEN  	
    tcp        0      0 0.0.0.0:1158                0.0.0.0:*                   LISTEN  	
    tcp        0      0 0.0.0.0:742                 0.0.0.0:*                   LISTEN  	
    tcp        0      0 0.0.0.0:3306                0.0.0.0:*                   LISTEN  	
    tcp        0      0 0.0.0.0:29355               0.0.0.0:*                   LISTEN  	
    tcp        0      0 192.168.1.184:11211         0.0.0.0:*                   LISTEN  	
    tcp        0      0 0.0.0.0:3468                0.0.0.0:*                   LISTEN  	
    tcp        0      0 0.0.0.0:111                 0.0.0.0:*                   LISTEN  	
    tcp        0      0 0.0.0.0:8080                0.0.0.0:*                   LISTEN  	
    tcp        0      0 0.0.0.0:5520                0.0.0.0:*                   LISTEN  	
    tcp        0      0 0.0.0.0:1521                0.0.0.0:*                   LISTEN  	
    tcp        0      0 ::ffff:127.0.0.1:8005       :::*                        LISTEN  	
    tcp        0      0 :::2281                     :::*                        LISTEN  	
    udp        0      0 127.0.0.1:9603              0.0.0.0:*                           	
    udp        0      0 127.0.0.1:32534             0.0.0.0:*                           	
    udp        0      0 127.0.0.1:34463             0.0.0.0:*                           	
    udp        0      0 0.0.0.0:61638               0.0.0.0:*                           	
    udp        0      0 0.0.0.0:12360               0.0.0.0:*                           	
    udp        0      0 192.168.1.184:11211         0.0.0.0:*                           	
    udp        0      0 127.0.0.1:63571             0.0.0.0:*                           	
    udp        0      0 0.0.0.0:736                 0.0.0.0:*                           	
    udp        0      0 127.0.0.1:3809              0.0.0.0:*                           	
    udp        0      0 0.0.0.0:739                 0.0.0.0:*                           	
    udp        0      0 0.0.0.0:111                 0.0.0.0:*               

就暂时到此为止好了，越发觉得不好掌控。很多东西自己也不是很清楚。无论学什么做什么都好，
还是要在学习中实践，在实践中学习，是一个交互的过程，这样才能朝着广度和深度的方向不断进步。
这里只是简单介绍了一些常用命令，可能会不是很全，我水平有限，精力有限，再更深的学习，
还是要靠自己努力。就这样了，以后或许会继续补充，并且在版本更新中注明，谢谢大家支持。