---
layout: post
comments: true
title: "linux文件目录结构简介(旧)"
description: "linux文件目录结构简介两年前的东西"
categories: ["linux", "配置", "总结"]
---

__这是09~10年的时候弄的一份总结文档，一直存在在google docs里边。这次把它弄过来，做个见证，留个纪念。__

## 正文

### 总体的，不好归类的
{% highlight bash %}
/media 用来挂载usb存储设备，DVD, CD-ROM等
/mnt 用来临时挂载文件系统，可插拔的设备应该挂载到/media上去
/home 除root之外的用户目录的默认所在地
/root root用户目录
/bin 最常用的命令
/sbin 系统管理员使用的命令(sbin=system bin)
/usr/local 使用源码安装的话，一般把prefix目录指定到这里，如/usr/local/ruby
/usr/share/applications desktop文件是桌面的菜单项
~/.gnome*,~/.gconf* gnome面板的个人配置信息，当gnome面板乱了，可以尝试删除这些文件来恢复默认面板
{% endhighlight %}

### /boot目录，kernel相关部分
{% highlight bash %}
/boot/symvers-%{KRELEASE}.gz 保存着内核中所有符号的crc值
/boot/System.map-%{KRELEASE} 给kernel使用的符号表(symbol table)
/boot/vmlinuz-%{KRELEASE} 可引导的、压缩的内核
/boot/initrd-%{KRELEASE}.img 包含了支持 Linux 系统两阶段引导过程所需要的必要可执行程序和系统文件
/boot/config-%{KRELEASE} 包括kernel的make config
/boot/message cpio格式的打包文件，存放Grub的配置信息，里面包括了图片，文字说明等内容
{% endhighlight %}

### /boot目录，grub配置
{% highlight bash %}
/boot/grub/menu.lst 一个链接文件，真实文件是grub.conf
/boot/grub/grub.conf grub的配置文件
/boot/grub/device.map 设备的映射文件
/boot/grub/splash.xpm.gz grub开机画面的gzip压缩包
/boot/grub/stageN 一般有stage1和stage2，是grub的核心，受限于mbr512字节的大小限制，所以切开成几个，stage1是用来加载stage2的
/boot/grub/XXX_stage1_5 stage2文件较大，一般存放于文件系统中，需要XXX_stage1_5来识别各种各样的文件系统
{% endhighlight %}

### /etc目录，系统用户/用户组
{% highlight bash %}
/etc/passwd 存放所有系统用户及相关信息
/etc/shadow 存放所有系统用户的密码信息
/etc/group 存放所有系统用户组及相关信息
/etc/gshadow 存放所有系统用户组的密码信息
{% endhighlight %}

### /etc目录，系统启动流程相关
{% highlight bash %}
/etc/issue 发行版信息
/etc/redhat-release redhat版本信息
/etc/inittab 系统初始化配置
/etc/init.d 存放服务脚本的地方
/etc/rc[0-6S].d 每个运行级别对应的服务，里边的脚本都是链接到/etc/init.d目录
/etc/rc rc启动脚本
/etc/rc.local 在所有init脚本结束后调用
/etc/rc.sysinit 在系统启动时运行一次
/etc/profile 环境变量配置
/etc/profile.d 保存一些脚本，可在/etc/profile中调用
~/.bash_profile  针对某个用户的配置，会调用.bash_rc
~/.bashrc 针对某个用户的配置，会调用/etc/bashrc
/etc/bashrc 使用bash时，可设置全局环境配置
~/.bash_history  命令的历史记录
~/.bash_logout   用户退出时执行
/etc/xinetd.conf xinetd的配置文件
/etc/xinetd.d 存放xinetd服务的地方
/etc目录，基本应用配置相关
/etc/skel 存放用户文件的“骨架”，当一个用户创建的时候，里边的文件就会拷贝到相应的home目录

/etc/X11 存放X Window的系统配置文件，例如xorg.conf

/etc/DIR_COLORS ls的时候，文件/文件夹显示的颜色
/etc/mtab 记录目前挂载的文件系统信息
/etc/fastboot 由shutdown -f 所产生的 ,在重启之后, 系统会去检查这个文件是否存在以决定是否要执行fsck
/etc/nologin 系统关闭的时候自动产生，里边放着shutdown message。在这个时候如果有用户企图登录，就会打印出这个文件存放的message，然后阻止你登录

/etc/fstab 默认的文件系统挂载情况

/etc/virc vi的配置
/etc/vimrc vim的配置
/etc/wgetrc wget的配置
/etc/yum.conf yum的配置
/etc/yum.repos.d yum源的存放位置

/etc/kdump.conf kdump内核的配置文件
/etc/my.cnf mysql的配置文件
/etc/ssh ssh的配置文件目录，重要的有sshd_config
/etc/syslog.conf syslog的配置文件
/etc/updatedb.conf updatedb的配置文件
/etc/mtools.conf mtools配置，用于在*UNIX系统中直接访问dos/win文件系统
/etc/sysctl.conf sysctl预加载的配置文件
/etc/moprobe.conf modprobe的配置文件
/etc/ld.so.conf 加载动态链接库的配置文件，默认会加载ld.so.conf.d里边的配置
/etc/ld.so.conf.d 存放动态链接库的配置文件
/etc/ld.so.cache 动态链接库的缓存，二进制文件，可以通过ldconfig --print-cache查看

/etc/services 网络服务列表(服务名,端口，协议等)
/etc目录，域名解析，主机访问控制
/etc/host.conf  定义DNS客户端主机发出域名解析的处理顺序，默认是先查看/etc/hosts文件，再发送远程请求
/etc/hosts 自定义ip-域名解析
/etc/resolv.conf DNS服务器地址
/etc/hosts.allow 和hosts.deny一起用来作为tcpd服务器的配置文件，tcpd服务器可以控制外部IP对本机服务的访问。hosts.allow控制可以访问本机的IP地址
/etc/hosts.deny  控制禁止访问本机的IP。如果和hosts.allow的配置有冲突，以hosts.deny为准

/etc目录，定时任务控制
/etc/crontab cron任务的配置文件，一般在里边配置有cron.hourly，cron.daily，cron.weekly和cron.monthly
/etc/cron.d 如果你要在特殊的时间使用crontab，可以把配置放到文件夹里边，配置的格式和/etc/crontab一样
/etc /cron.daily 每天定时任务
/etc/cron.hourly 每小时定时任务
/etc/cron.monthly 每月定时任务
/etc/cron.weekly 每星期定时任务
/etc/cron.allow 指定那些用户可以使用crontab
/etc/cron.deny 指定哪些用户禁止使用crontab，如果文件存在且为空，所有人都可以使用，如果文件不存在，那么只有root可以使用
/etc/at.allow 指定那些用户可以使用at
/etc/at.deny 指定哪些用户禁止使用at，如果文件存在且为空，所有人都可以使用，如果文件不存在，那么只有root可以使用
{% endhighlight %}

### /dev目录 硬件设备信息
{% highlight bash %}
/dev/hd[a-z]  第几个IDE硬盘
/dev/tty[0-9] 第几个虚拟控制台
/dev/sd[a-z]  第几个SCSI或SATA硬盘
/dev/zero 一个无穷尽地提 供0(NULL)的设备，可以用来初始化文件
/dev/null 一个空设备，可以向它输出任何数据，而任何写入它的输出都会被抛弃。如果不想让消息以标准输出显示或写入文件，那么可以将消息重定向到位桶
/dev/stderr  链接文件，指向/proc/self/fd/2(标准错误)

/dev/stdin  链接文件，指向/proc/self/fd/0(标准输入)
/dev/stdout 链接文件，指向/proc/self/fd/1(标准输出)
/dev/console 系统控制台，也就是直接和系统连接的监视器。如果你用cat查看该设备，并敲入一些内容，可以看到在屏幕上回显
/dev/fd[0-9] 第几个软驱设备
/dev/st SCSI磁带驱动器
/dev/pty 提供远程登陆伪终端支持。在进行Telnet登录时就要用到该设备
/dev/ttys 计算机串行接口，对于DOS来说就是com1口
/dev/cua 计算机串行接口，与调制解调器一起使用的设备
{% endhighlight %}

###/proc目录 虚拟文件系统
{% highlight bash %}
/proc/apm Advanced Power Management(APM)系统信息，与apm命令相关
/proc/buddyinfo 每个内存区中的每个order有多少块可用,和内存碎片问题有关
/proc/cmdline 启动时传递给kernel的参数信息
/proc/cpuinfo cpu的信息
/proc/crypto 内核使用的所有已安装的加密密码及细节
/proc/devices 已经加载的设备并分类
/proc/dma 已注册使用的ISA DMA频道列表
/proc/execdomains Linux内核当前支持的execution domains
/proc/fb 帧缓冲设备列表，包括数量和控制它的驱动
/proc/filesystems 内核当前支持的文件系统类型
/proc/interrupts x86架构中的每个IRQ中断数
/proc/iomem 每个物理设备当前在系统内存中的映射
/proc/ioports 一个设备的输入输出所使用的注册端口范围
/proc/kcore 代表系统的物理内存，存储为核心文件格式，里边显示的是字节数，等于RAM大小加上4kb
/proc/kmsg 记录内核生成的信息，可以通过/sbin/klogd或/bin/dmesg来处理
/proc/loadavg 根据过去一段时间内CPU和IO的状态得出的负载状态，与uptime命令有关
/proc/locks 内核锁住的文件列表
/proc/mdstat 多硬盘，RAID配置信息(md=multiple disks)
/proc/meminfo RAM使用的相关信息
/proc/misc 其他的主要设备(设备号为10)上注册的驱动
/proc/modules 所有加载到内核的模块列表
/proc/mounts 系统中使用的所有挂载
/proc/mtrr 系统使用的Memory Type Range Registers (MTRRs)
/proc/partitions 分区中的块分配信息
/proc/pci 系统中的PCI设备列表

/proc/slabinfo 系统中所有活动的 slab 缓存信息
/proc/stat 所有的CPU活动信息
/proc/sysrq-trigger 使用echo命令来写这个文件的时候，远程root用户可以执行大多数的系统请求关键命令，就好像在本地终端执行一样。要写入这个文件，需要把/proc/sys/kernel/sysrq不能设置为0。这个文件对root也是不可读的
/proc/uptime 系统已经运行了多久
/proc/swaps 交换空间的使用情况
/proc/version Linux内核版本和gcc版本
/proc/bus 系统总线(Bus)信息，例如pci/usb等
/proc/driver 驱动信息
/proc/fs 文件系统信息
/proc/ide ide设备信息
/proc/irq 中断请求设备信息
/proc/net 网卡设备信息
/proc/scsi scsi设备信息
/proc/tty tty设备信息
/proc/net/dev 显示网络适配器及统计信息
/proc/vmstat 虚拟内存统计信息
/proc/vmcore 内核panic时的内存映像
/proc/diskstats 取得磁盘信息
/proc/schedstat kernel调度器的统计信息
/proc/zoneinfo 显示内存空间的统计信息，对分析虚拟内存行为很有用
/proc目录， 进程N的信息
/proc/N pid为N的进程信息
/proc/N/cmdline 进程启动命令
/proc/N/cwd 链接到进程当前工作目录
/proc/N/environ 进程环境变量列表
/proc/N/exe 链接到进程的执行命令文件
/proc/N/fd 包含进程相关的所有的文件描述符
/proc/N/maps 与进程相关的内存映射信息
/proc/N/mem 指代进程持有的内存，不可读
/proc/N/root 链接到进程的根目录
/proc/N/stat 进程的状态
/proc/N/statm 进程使用的内存的状态
/proc/N/status 进程状态信息，比stat/statm更具可读性
/proc/self 链接到当前正在运行的进程
{% endhighlight %}

### /var目录 存放经常变化数据的地方
{% highlight bash %}
/var/lib/rpm 存放大多数rpm相关的文件
/var/cache/yum yum升级时下载的rpm文件的临时存放地，还包括系统中rpm包的头信息
/var/spool/cron/$username 每个用户自定义的cron任务，可以使用crontab或vi来操作

/var/lock 一般用来存放文件锁
/var/log 一般用来存放日志文件
/var/run 一般用来存放pid文件
/var/crash 一般是存放系统崩溃时产生的信息
/var/cache 一般用来存放缓存信息,例如yum package的缓存
{% endhighlight %}

### /etc/sysconfig目录 系统基本配置
{% highlight bash %}
/etc/sysconfig/amd 为amd提供操作参数，用来自动mount/unmount文件系统
/etc/sysconfig/apmd   由apmd使用来配置电源设置
/etc/sysconfig/arpwatch 在启动的时候传递给arpwatch守护进程的参数
/etc/sysconfig/authconfig 设置主机使用的验证方式
/etc/sysconfig/autofs 自动挂载设备的自定义选项
/etc/sysconfig/clock 系统硬件时钟的设置
/etc/sysconfig/desktop 设置新用户的桌面和进入运行级别5所使用的显示管理器
/etc/sysconfig/dhcpd 在启动的时候传递给dhcpd守护进程的参数
/etc/sysconfig/gpm 在启动的时候传递给gpm守护进程的参数
/etc/sysconfig/hwconf 列出kudzu检测到的所有硬件
/etc/sysconfig/i18n 默认系统语言，系统支持的所有语言，默认系统字体
/etc/sysconfig/init 系统启动时的显示方式
/etc/sysconfig/ip6tables-config  在系统启动或者ip6tables服务启动时，内核用来设置IPv6包过滤
/etc/sysconfig/iptables-config 在系统启动或者iptables服务启动时，内核用来设置包过滤
/etc/sysconfig/keyboard 控制键盘的行为
/etc/sysconfig/kudzu 在启动的时候通过kudzu触发一次安全的系统硬件探查
/etc/sysconfig/named 在启动的时候传递给named守护进程的参数
/etc/sysconfig/netdump netdump服务的配置文件
/etc/sysconfig/network 网络的配置信息
/etc/sysconfig/ntpd 在启动的时候传递给ntpd守护进程的参数
/etc/sysconfig/radvd 在启动的时候传递给radvd守护进程的参数
/etc/sysconfig/samba 在启动的时候传递给smbd/nmbd守护进程的参数
/etc/sysconfig/selinux selinux的基本控制选项
/etc/sysconfig/spamassassin 在启动的时候传递给spamd守护进程的参数
/etc/sysconfig/squid 在启动的时候传递给squid守护进程的参数
/etc/sysconfig/vncservers 配置vnc服务启动的方式
/etc/sysconfig/xinetd 在启动的时候传递给xinetd守护进程的参数
{% endhighlight %}

### /proc/sys目录 系统重要配置参数，涉及众多内核参数
{% highlight bash %}
/proc/sys/fs/file-max 可以分配的文件句柄的最大数目
/proc/sys/fs/file-nr 已分配文件句柄的数目、已使用文件句柄的数目、文件句柄的最大数目
/proc/sys/fs/inode-* 任何以名称“inode”开头的文件所执行的操作与上面那些以名称“file”开头的文件所执行的操作一样，但所执行的操作与索引节点有关，而与文件句柄无关
/proc/sys/fs/overflowuid 和 /proc/sys/fs/overflowgid 这两个文件分别保存那些支持 16 位用户标识和组标识的任何文件系统的用户标识（UID）和组标识（GID）
/proc/sys/fs/super-max 该文件指定超级块处理程序的最大数目。挂装的任何文件系统需要使用超级块，所以如果挂装了大量文件系统，则可能会用尽超级块处理程序
/proc/sys/fs/super-nr　显示当前已分配超级块的数目
/proc/sys/kernel/acct 该文件有三个可配置值，根据包含日志的文件系统上可用空间的数量（以百分比表示），这些值控制何时开始进行进程记帐：如果可用空间低于这个百分比值，则停止进程记帐/如果可用空间高于这个百分比值，则开始进程记帐/检查上面两个值的频率（以秒为单位）
/proc/sys/kernel/ctrl-alt-del 该值控制系统在接收到 ctrl+alt+delete 按键组合时如何反应
/proc/sys/kernel/domainname 配置网络域名
/proc/sys/kernel/hostname 主机名
/proc/sys/kernel/msgmax 指定了从一个进程发送到另一个进程的消息的最大长度
/proc/sys/kernel/msgmnb  指定在一个消息队列中最大的字节数
/proc/sys/kernel/msgmni 指定消息队列标识的最大数目
/proc/sys/kernel/panic 如果发生“内核严重错误（kernel panic）”，内核在重新引导之前等待的时间
/proc/sys/kernel/printk 该文件有四个数字值，它们根据日志记录消息的重要性，定义将其发送到何处
/proc/sys/kernel/shmall 在任何给定时刻系统上可以使用的共享内存的总量（以字节为单位）
/proc/sys/kernel/shmax 内核所允许的最大共享内存段的大小（以字节为单位）
/proc/sys/kernel/shmmni 用于整个系统共享内存段的最大数目
/proc/sys/kernel/sysrq  如果该文件指定的值为非零，则激活 System Request Key
/proc/sys/kernel/threads-max 内核所能使用的线程的最大数目
/proc/sys/net/core/message_burst 写新的警告消息所需的时间（以 1/10 秒为单位）；在这个时间内所接收到的其它警告消息会被丢弃。这用于防止某些企图用消息“淹没”您系统的人所使用的拒绝服务攻击
/proc/sys/net/core/message_cost 存有与每个警告消息相关的成本值。该值越大，越有可能忽略警告消息
/proc/sys/net/core/netdev_max_backlog  在接口接收数据包的速率比内核处理这些包的速率快时，允许送到队列的数据包的最大数目
/proc/sys/net/core/optmem_max 每个套接字所允许的最大缓冲区的大小
/proc/sys/net/core/rmem_default 接收套接字缓冲区大小的缺省值（以字节为单位）
/proc/sys/net/core/rmem_max 接收套接字缓冲区大小的最大值（以字节为单位）。
/proc/sys/net/core/wmem_default 发送套接字缓冲区大小的缺省值（以字节为单位）。
/proc/sys/net/core/wmem_max 发送套接字缓冲区大小的最大值（以字节为单位）

/proc/sys/net/ipv4/ip_forward ip转发是否生效
/proc/sys/net/ipv4/tcp_retrans_collapse 控制TCP双方窗口协商出现错误的时候的一些重传的行为。但是在老的2.6的核 (<2.6.18)里头，这个重传会导致kernel oops，kernel panic，所以如果出现有 tcp_retrans_*样子的kernel panic，可以把这个参数给设置成0
/proc/sys/vm/buffermem 控制用于缓冲区内存的整个系统内存的数量（以百分比表示）。它有三个值，通过把用空格相隔的一串数字写入该文件来设置这三个值。用于缓冲区的内存的最低百分比/如果发生所剩系统内存不多，而且系统内存正在减少这种情况，系统将试图维护缓冲区内存的数量/用于缓冲区的内存的最高百分比
/proc/sys/vm/freepages 控制系统如何应对各种级别的可用内存。它有三个值，通过把用空格相隔的一串数字写入该文件来设置这三个值。如果系统中可用页面的数目达到了最低限制，则只允许内核分配一些内存/如果系统中可用页面的数目低于这一限制，则内核将以较积极的方式启动交换，以释放内存，从而维持系统性能/内核将试图保持这个数量的系统内存可用。低于这个值将启动内核交换
/proc/sys/vm/kswapd 控制允许内核如何交换内存。它有三个值，通过把用空格相隔的一串数字写入该文件来设置这三个值：内核试图一次释放的最大页面数目。如果想增加内存交换过程中的带宽，则需要增加该值/内核在每次交换中试图释放页面的最少次数/内核在一次交换中所写页面的数目。这对系统性能影响最大。这个值越大，交换的数据越多，花在磁盘寻道上的时间越少。然而，这个值太大会因“淹没”请求队列而反过来影响系统性能
/proc/sys/vm/pagecache 该文件与/proc/sys/vm/buffermem 的工作内容一样，但它是针对文件的内存映射和一般高速缓存
/proc/sys/vm/dirty_background_ratio 记录当所有被更改页面总大小占工作内存超过某个限制时，pdflush 会开始写回工作,默认是10%
/proc/sys/vm/dirty_ratio 控制文件系统的文件系统写缓冲区的大小，单位是百分比，表示系统内存的百分比，表示当写缓冲使用到系统内存多少的时候，开始向磁盘写出数据。默认是40%
/proc/sys/vm/dirty_writeback_centisecs 记录pdflush进程把page cache里边的内容写入磁盘的时间周期，默认是5秒
/proc/sys/vm/dirty_expire_centisecs 控制一个更改过的页面经过多长时间后被认为是过期的、必须被写回的页面，默认是30秒
/proc/sys/vm/laptop_mode 是否使用笔记本模式，在kernel2.6之后支持
{% endhighlight %}
