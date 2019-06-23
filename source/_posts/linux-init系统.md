---
title: linux init系统
date: 2019-06-23 16:35:02
tags:
 - linux
 - init
 - Upstart
 - SystemD
 - SysVInit
categories: linux
---

## Linux init系统
Linux的启动从BIOS开始，bootloader载入内核，进行内核初始化。内核初始化的最后一步是启动pid为$1$的init进程。这个进程是系统的第一个进程。它负责产生其他所有用户进程。init进程以super daemon方式存在，是所有其他进程的祖先。
Init系统能够定义、管理和控制init进程的行为。它负责组织和运行许多独立的或相关的始化job(因此被称为 init 系统)，从而让计算机系统进入某种用户预订的run level。仅仅将内核运行起来是毫无实际用途的，必须由 init 系统将系统代入可操作状态。比如启动外壳 shell 后，便有了人机交互，这样就可以让计算机执行一些预订程序完成有实际意义的任务。或者启动 X 图形系统以便提供更佳的人机界面，更加高效的完成任务。这里，字符界面的 shell 或者 X 系统都是一种预设的run level。
大多数 Linux 发行版的 init 系统是和 System V 相兼容的，被称为 SysVInit。Ubuntu 和 RHEL 采用 upstart 替代了传统的 SysVInit。而 Fedora 从版本 15 开始使用 SystemD 的新 init 系统。ubuntu-16.10之后开始不再使用SysVInt管理系统，改用SystemD。SysVInit是老版的service启动方式，而SystemD是新版的。所以推荐尽量使用SystemD而不是SysVInit。这里会介绍下面三种init系统   
- SysVInit(initd)
- Upstart
- SystemD

## SysVInit(initd)
SysVInit的最大缺点是主要依赖于 Shell 脚本，启动太慢。

### runlevel
runlevel定义运行级别。SysVInit 检查/etc/inittab文件中是否含有'initdefault' 项，这告诉 init 系统是否有一个默认run level。如果没有默认的run level，那么用户将进入系统控制台，手动决定进入何种run level。SysVInit通常会有8种run-level，0到6和S或者s。不同Linux 发行版对run level的定义不太一样。但 0，1，6是公认的： 0代表关机, 1代表单用户模式, 6代表重启。
/etc/inittab 文件中还定义了各种run level的初始化范围。RedHat中定义了runlevel 3和5。level 3将系统初始化为字符界面的shell模式；level 5 将系统初始化为GUI模式。无论是命令行界面还是 GUI，run level 3和5相对于其他run level都是完整的正常运行状态，而模式1，S等往往用于系统故障之后的排错和恢复。不同的run level下系统需要初始化运行的进程和需要进行的初始化准备都是不同的。比如run level 3 不需要启动X系统。用户只需要指定需要进入哪种level，SysVInit负责执行该level所必须的初始化工作。

### SysVInit初始化顺序
SysVInit用脚本，文件命名规则和软链接来实现不同的runlevel。首先，SysVInit 需要读取/etc/inittab 文件，获得以下配置信息：
- 系统需要进入的 runlevel
- 捕获组合键的定义
- 定义电源 fail/restore 脚本
- 启动 getty 和虚拟控制台

得到配置信息后，SysVInit 顺序地执行以下初始化步骤将系统初始化为相应的runlevel X。
- /etc/rc.d/rc.sysinit
- /etc/rc.d/rc 和/etc/rc.d/rcX.d/ (X 代表运行级别 0-6)
- /etc/rc.d/rc.local
- X Display Manager（如果需要的话）

#### rc.sysinit初始化
首先运行rc.sysinit执行一些系统初始化任务。在RHEL5系统中(RHEL6 已经使用 upstart 了)，rc.sysinit 主要完成以下这些job。
- 激活 udev 和 selinux
- 设置定义在/etc/sysctl.conf 中的内核参数
- 设置系统时钟
- 加载 keymaps
- 使能交换分区
- 设置主机名(hostname)
- 根分区检查和 remount
- 激活 RAID 和 LVM 设备
- 开启磁盘配额
- 检查并挂载所有文件系统
- 清除过期的 locks 和 PID 文件

#### /etc/rd.c/rc初始化
接下来SysVInit运行/etc/rc.d/rc脚本。根据不同的runlevel，rc脚本打开对应该runlevel的rcX.d目录(X就是runlevel)，以S开头的脚本是启动时运行的脚本，S后面的数字定义了这些脚本的执行顺序。在/etc/rc.d/rcX.d 目录下的脚本其实都是一些软链接文件，真实的脚本文件存放在/etc/init.d 目录下。

#### /etc/rc.d/rc.local初始化
rc.local是自定义脚本存放目录。

### SysVInit关闭系统
关闭顺序的控制也是依靠/etc/rc.d/rcX.d/目录下所有脚本的命名规则来控制的，所有以K开头的脚本都将在关闭系统时调用，K后的数字规定了执行顺序。这些脚本负责安全地停止service或者其他的关闭job。

### SysVInit管理和控制功能
SysVInit包含了一系列启动，运行和关闭所有其他程序的工具和命令。
- halt  停止系统。
- init  SysVInit本身的init进程，pid=1，是所有用户进程的父进程。最主要的作用是在启动过程中使用/etc/inittab文件创建所有其他初始化进程。
- killall5  SystemV的killall 命令。向除自己的会话(session)进程之外的其它进程发出信号，所以不能杀死当前使用的 shell。
- last  回溯/var/log/wtmp 文件(或者-f 选项指定的文件)，显示自从这个文件建立以来，所有用户的登录情况。
- lastb 作用和 last 差不多，默认情况下使用/var/log/btmp 文件，显示所有失败登录企图。
- mesg  控制其它用户对用户终端的访问。
- pidof 找出程序的进程识别号(pid)，输出到标准输出设备。
- poweroff  等于 shutdown -h –p，或者 telinit 0。关闭系统并切断电源。
- reboot    等于 shutdown –r 或者 telinit 6。重启系统。
- runlevel  读取系统的登录记录文件(一般是/var/run/utmp)把以前和当前的run level输出到标准输出设备。
- shutdown  以一种安全的方式终止系统，所有正在登录的用户都会收到系统将要终止通知，并且不准新的登录。
- sulogin   当系统进入单用户模式时，被init调用。当接收到启动加载程序传递的-b 选项时，init 也会调用 sulogin。
- telinit   实际是init的一个连接，用来向init传送单字符参数和信号。
- utmpdump  以一种用户友好的格式向标准输出设备显示/var/run/utmp 文件的内容。
- wall  向所有有信息权限的登录用户发送消息。


## Upstart
Ubuntu使用upstart init系统，没有/etc/inittab文件。Upstart解决了热插拔以及网络共享盘的挂载问题。在/etc/fstab 中，可以指定系统自动挂载一个网络盘，比如 NFS等，SysVInit 分析/etc/fstab 挂载文件系统这个步骤是在网络启动之前。可是如果网络没有启动，NFS或者iSCSI服务都不可访问，当然也无法进行挂载操作。SysVInit采用netdev的方式来解决这个问题，即/etc/fstab发现netdev属性挂载点的时候，不尝试挂载它，在网络初始化之后，有专门的netfs service来挂载所有这些网络盘。

### Upstart 的特点
UpStart 解决了之前提到的 SysVInit 的缺点。采用event驱动模型，UpStart 可以：
- 更快地启动系统。采用event驱动机制加快了系统启动时间。SysVInit 运行时是同步阻塞的。一个脚本运行的时候，后续脚本必须等待。这意味着所有的初始化步骤都是串行执行的，而实际上很多service彼此并不相关，完全可以并行启动，从而减小系统的启动时间。
- 当新硬件被发现时动态启动相应的service，Ubuntu 开发了基于event机制的UpStart，比如 U 盘插入 USB 接口后，udev得到内核通知，发现该设备，这是一个新的event。UpStart感知到该event之后触发相应的等待任务，比如处理/etc/fstab中新的挂载点。采用这种event驱动的模式，upstart 完美地解决了即插即用设备带来的新问题。
- 硬件被拔除时动态停止相应的service

### Upstart job和event
Job是一个job unit，用来完成一件工作，比如启动一个后台service，或者运行一个配置命令。每个 Job 都等待一个或多个event，一旦event发生，upstart 就触发该job完成相应的工作。

### Job
Job是一个job的unit，一个task或者一个service。可以理解为SysVInit中的一个service脚本。有三种类型的job：
- task job，task job 代表在一定时间内会执行完毕的任务，比如删除一个文件；
- service job，service job 代表后台service进程，比如 apache httpd。这里进程一般不会退出，一旦开始运行就成为一个后台deamon，由 init 进程管理，如果这类进程退出，由 init 进程重新启动，它们只能由 init 进程发送信号停止。它们的停止一般也是由于所依赖的停止event而触发的，不过 upstart 也提供命令行工具，让管理人员手动停止某个service；
- abstract job，abstract job 仅由 upstart 内部使用，仅对理解 upstart 内部机理有所帮助。我们不用关心它。

#### system job and session job
还可以根据Upstart初始化的范围对job进行分类。系统的初始化任务叫做system job，比如挂载文件系统的任务就是一个system job；用户会话的初始化service就叫做 session job。

#### Job 生命周期
Upstart为每个job都维护一个生命周期。一般来说，job有开始，运行和结束。以及其他更精细的状态，比如开始有开始之前(pre-start)，即将开始(starting)和已经开始了(started)几种不同的状态，这样可以更加精确地描述job的当前状态。详细的状态如下表所示：
状态名|	含义
---|---
Waiting|初始状态
Starting|Job 即将开始
pre-start|执行 pre-start 段，即任务开始前应该完成的job
Spawned|准备执行 script 或者 exec 段
post-start|执行 post-start 动作
Running|interim state set after post-start section processed denoting job is running (But it may have no associated PID!)
pre-stop|执行 pre-stop 段
Stopping|interim state set after pre-stop section processed
Killed|任务即将被停止
post-stop|执行 post-stop 段

当job的状态即将发生变化的时候，init 进程会发出相应的event（event），如下图是job的状态机。其中只有四个状态Starting，Started，Stopping，Stopped会引起init进程发送相应的event，而其它的状态变化不会发出event。
![]()

### Event
Event在upstart中以通知消息的形式具体存在。一旦某个event发生了，Upstart 就向整个系统发送一个消息。Event 可以分为三类: signal，methods 或者 hooks。
- Signals，Signal event是非阻塞的，异步的。发送一个信号之后控制权立即返回。
- Methods，Methods event是阻塞的，同步的。
- Hooks，Hooks event是阻塞的，同步的。它介于 Signals 和 Methods 之间，调用发出 Hooks event的进程必须等待event完成才可以得到控制权，但不检查event是否成功。

#### Event示例
1. 系统上电启动，init 进程会发送"start"event
2. 根文件系统可写时，相应 job 会发送文件系统就绪的event
3. 一个块设备被发现并初始化完成，发送相应的event
4. 某个文件系统被挂载，发送相应的event
5. 类似 atd 和 cron，可以在某个时间点，或者周期的时间点发送event
6. 另外一个 job 开始或结束时，发送相应的event
7. 一个磁盘文件被修改时，可以发出相应的event
8. 一个网络设备被发现时，可以发出相应的event
9. 缺省路由被添加或删除时，可以发出相应的event

不同的 Linux 发行版对 upstart 有不同的定制和实现，实现和支持的event也有所不同，可以用man 7 upstart-events来查看event列表。

### Job 和 Event 的相互协作
Upstart是由event触发job运行的一个系统，每一个程序的运行都由其依赖的event发生而触发的。系统初始化时，init进程开始运行，init进程自身会发出不同的event，这些event会触发一些job运行。每个job运行过程中会释放不同的event，这些event又将触发新的job运行。如此反复，直到整个系统正常运行起来。

### job配置文件
每一个Job都是由一个job配置文件（Job Configuration File）定义的。job配置文件存放在/etc/init下面，是以.conf 作为文件后缀的文件。

#### 示例
~$:cat /etc/init/anacron.conf
``` txt
# anacron - anac(h)ronistic cron
#
# anacron executes commands at specific periods, but does not assume that
# the machine is running continuously

description	"anac(h)ronistic cron"

start on runlevel [2345]
stop on runlevel [!2345]

expect fork
normal exit 0

exec anacron -s
```

#### 常见的sec
**"expect" Stanza**
为了启动，停止，重启和查询某个系统service。Upstart 需要跟踪该service所对应的进程。部分service为了将自己变成daemon，会采用fork调用， UpStart必须采用fork出来的进程号作为service的 PID。但是，UpStart 本身无法判断service进程是否fork了，所以需要指定expect告诉 UpStart 进程是否fork了。"expect fork"表示进程只会 fork 一次；"expect daemonize"表示进程会 fork 两次。

**"exec" Stanza 和"script" Stanza**
"exec"关键字配置job需要运行的命令，"script"关键字定义需要运行的脚本。

#### 示例
``` txt
# mountall.conf
description “Mount filesystems on boot”
start on startup
stop on starting rcS
...
script
  . /etc/default/rcS
  [ -f /forcefsck ] && force_fsck=”--force-fsck”
  [ “$FSCKFIX”=”yes” ] && fsck_fix=”--fsck-fix”
    
  ...
   
  exec mountall –daemon $force_fsck $fsck_fix
end script
...
```
该job在系统启动时运行，负责挂载所有的文件系统。该job需要执行复杂的脚本，由"script"关键字定义；在脚本中，使用了 exec 来执行 mountall 命令。

**"start on" Stanza 和"stop on" Stanza**
"start on"定义了触发job的所有event。"start on"的语法很简单，如下所示：

start on EVENT [[KEY=]VALUE]... [and|or...]
EVENT 表示event的名字，可以在 start on 中指定多个event，表示该job的开始需要依赖多个event发生。多个event之间可以用 and 或者 or 组合，"表示全部都必须发生"或者"其中之一发生即可"等不同的依赖条件。除了event发生之外，job的启动还可以依赖特定的条件，因此在 start on 的 EVENT 之后，可以用 KEY=VALUE 来表示额外的条件，一般是某个环境变量(KEY)和特定值(VALUE)进行比较。如果只有一个变量，或者变量的顺序已知，则 KEY 可以省略。
"stop on"定义job在什么情况下需要停止。

#### 示例
``` txt
#dbus.conf
description     “D-Bus system message bus”
 
start on local-filesystems
stop on deconfiguring-networking
...
```
D-Bus 是一个系统消息service，上面的配置文件表明当系统发出 local-filesystems event时启动 D-Bus；当系统发出 deconfiguring-networking event时，停止 D-Bus service。

### Session Init
Session 就是一个用户会话，即用户从远程或者本地登入系统开始job，直到用户退出，整个过程构成一个会话。用户往往会为自己的会话做一个定制，如添加特定的命令别名等等。这些job都属于对特定会话的初始化操作，被称为 Session Init。在字符模式下，会话初始化相对简单。用户登录后只能启动一个 Shell，通过 shell 命令使用系统。各种 shell 程序都支持一个自动运行的启动脚本，比如~/.bashrc。用户在这些脚本中加入需要运行的定制化命令。在图形界面下，用户登录后看到的并不是一个 shell 提示符，而是一个桌面。一个完整的桌面环境包括 window manager，panel以及其它一些定义在/usr/share/gnome-session/sessions/下面的基本组件，此外还有一些辅助的应用程序，，比如 system monitors，panel applets，NetworkManager，Bluetooth，printers 等。当用户登录之后，这些组件都需要被初始化。目前启动各种图形组件和应用的job由 gnome-session 完成。过程如下：
以 Ubuntu 为例，当用户登录 Ubuntu 图形界面后，显示管理器(Display Manager)lightDM 启动 Xsession。Xsession 接着启动 gnome-session，gnome-session 负责其它的初始化job，然后就开始了一个 desktop session。如下所示，是传统的desktop session 启动过程：
``` txt
init
 |- lightdm
 |   |- Xorg
 |   |- lightdm ---session-child
 |        |- gnome-session --session=ubuntu
 |             |- compiz
 |             |- gwibber
 |             |- nautilus
 |             |- nm-applet
 |             :
 |             :
 |
 |- dbus-daemon --session
 |
 :
 :
```
但是事实上一些应用和组件并不需要在会话初始化过程中启动，而是需要在需要它们的时候才启动。比如 Network Manager，一天之内用户很少切换网络设备，所以大部分时间 Network Manager service仅仅是在浪费系统资源。UpStart的基于event的按需启动的模式就可以很好地解决这些问题。下面给出了采用UpStart之后的会话初始化过程：
``` txt
init
 |- lightdm
 |   |- Xorg
 |   |- lightdm ---session-child
 |        |- session-init # <-- upstart running as normal user
 |             |- dbus-daemon --session
 |             |- gnome-session --session=ubuntu
 |             |- compiz
 |             |- gwibber
 |             |- nautilus
 |             |- nm-applet
 |             :
 |             :
 :
 :
```

### UpStart 使用
#### Upstart 系统中的运行级别
Upstart 的运作完全是基于job和event的。Job的状态变化和运行会引起event，进而触发其它job和event。而SysVInit是基于运行级别的，因为历史的原因，Linux 上的多数软件还是采用传统的 SysVInit 脚本启动方式，所以UpStart还是必须模拟老的SysVInit的run level，以便和多数现有软件兼容。

#### 系统启动过程
下图描述了 UpStart 的启动过程。
![upstart startup](startup.png)
系统上电后运行GRUB载入内核。内核执行硬件初始化和内核自身初始化。在内核初始化的最后，内核将启动pid为1的Upstart init 进程。Upstart 进程执行一些自身的初始化job后，立即发出"startup" event。上图中用红色方框加红色箭头表示event，可以在左上方看到"startup"event。
所有依赖于"startup"event的job被触发，其中最重要的是mountall。mountall jog负责挂载系统中需要使用的文件系统，完成相应job后，mountall任务会发出以下event：local-filesystem，virtual-filesystem，all-swaps，
其中 virtual-filesystem event触发 udev 任务开始job。任务 udev 触发 upstart-udev-bridge 的job。Upstart-udev-bridge 会发出 net-device-up IFACE=lo event，表示本地回环 IP 网络已经准备就绪。同时，任务 mountall 继续执行，最终会发出 filesystem event。此时，任务 rc-sysinit 会被触发，因为 rc-sysinit 的 start on条件如下：
``` txt
start on filesystem and net-device-up IFACE=lo
```
任务rc-sysinit调用telinit。Telinit任务会发出 runlevel event，触发执行/etc/init/rc.conf。rc.conf 执行/etc/rc$.d/目录下的所有脚本，和 SysVInit 非常类似。

### Upstart注意的事项

1. fork次数，都通过fork 两次的技巧将自己变成后台service程序需要指定expect stanza。
2. fork后即可用，后台程序在完成第二次fork的时候，必须保证service已经可用。因为UpStart通过派生计数来决定service是否处于就绪状态。
3. 遵守 SIGHUP 的要求，UpStart 会给daemon发送SIGHUP信号，此时，UpStart 希望该deamon做以下这些响应job：
- 完成所有必要的重新初始化job，比如重新读取配置文件。这是因为 UpStart 的命令"initctl reload"被设计为可以让service在不重启的情况下更新配置。
- deamon必须继续使用现有的 PID，即收到 SIGHUP 时不能调用 fork。如果service必须在这里调用 fork，则等同于派生两次，参考上面的规则一的处理。这个规则保证了 UpStart 可以继续使用 PID 管理本service。

4. 收到 SIGTEM 即 shutdown。
- 当收到 SIGTERM 信号后，UpStart 希望deamon进程立即干净地退出，释放所有资源。如果一个进程在收到 SIGTERM 信号后不退出，Upstart 将对其发送 SIGKILL 信号。

5. initctl list 来查看所有job的概况，用 initctl stop 停止一个正在运行的job；用 initctl start 开始一个job；还可以用 initctl status 来查看一个job的状态；initctl restart 重启一个job；initctl reload 可以让一个正在运行的service重新载入配置文件。这些命令和传统的 service 命令十分相似。
~$:initctl list
``` txt
alsa-mixer-save stop/waiting
avahi-daemon start/running, process 690
mountall-net stop/waiting
rc stop/waiting
rsyslog start/running, process 482
screen-cleanup stop/waiting
tty4 start/running, process 859
udev start/running, process 334
upstart-udev-bridge start/running, process 304
ureadahead-other stop/waiting
```
第一列是job名，比如 rsyslog。第二列是job的目标；第三列是job的状态。
UpStart 还提供了一些快捷命令来简化 initctl。比如 reload，restart，start，stop 等等。启动一个service可以简单地调用
~$:start job
这和执行 initctl start job是一样的效果。
一些命令是为了兼容其它系统(主要是 SysVInit)，比如显示 runlevel 用/sbin/runlevel 命令：
~$:runlevel
\> N 2

在 Upstart 系统中，需要修改/etc/init/rc-sysinti.conf 中的 DEFAULT_RUNLEVEL 这个参数，以便修改默认启动运行级别。这一点和 SysVInit 的习惯有所不同，大家需要格外留意。


## SystemD
### 优势
- 同 SysVInit 和 LSB init scripts 兼容，SystemD 提供了和SysVInit以及LSB initscripts兼容的特性。系统中已经存在的service和进程无需修改。这降低了系统向SystemD迁移的成本，使得SystemD替换现有初始化系统成为可能。
- 更快的启动速度，SystemD 提供了比 UpStart 更激进的并行启动能力，采用了 socket / D-Bus activation 等技术启动service。一个显而易见的结果就是：更快的启动速度。

为了减少系统启动时间，SystemD 的目标是尽可能启动更少的进程和尽可能将更多进程并行启动，同样地，UpStart 也试图实现这两个目标。UpStart 采用event驱动机制，当service需要的时候才通过event触发启动；不相干的service可以并行启动。下面的图形演示了 UpStart 相对于 SysVInit 在并发启动这个方面的改进：
![](SystemD.png)
假设有 7 个不同的启动项目， 比如 JobA、Job B 等等。在 SysVInit 中，每一个启动项目都由一个独立的脚本负责，它们由 SysVInit 顺序地，串行地调用。因此总的启动时间为 T1+T2+T3+T4+T5+T6+T7。其中一些任务有依赖关系，比如 A,B,C,D。而Job E和F却和A,B,C,D无关。这种情况下，UpStart 能够并发地运行任务{E，F，(A,B,C,D)}，使得总的启动时间减少为 T1+T2+T3。但是在 UpStart 中，有依赖关系的service还是必须先后启动。
让我们例举一些例子， Avahi service需要 D-Bus 提供的功能，因此 Avahi 的启动依赖于 D-Bus，UpStart 中，Avahi 必须等到 D-Bus 启动就绪之后才开始启动。类似的，livirtd 和 X11 都需要 HAL service先启动，而所有这些service都需要 syslog service记录日志，因此它们都必须等待 syslog service先启动起来。然而 httpd 和他们都没有关系，因此 httpd 可以和 Avahi 等service并发启动。
SystemD 能够更进一步提高并发性，即便对于那些 UpStart 认为存在相互依赖而必须串行的service，比如 Avahi 和 D-Bus 也可以并发启动。从而实现如下图所示的并发启动过程：
![](SystemD_para.jpg)
所有的任务都同时并发执行，总的启动时间被进一步降低为 T1。
- SystemD 提供按需启动能力。SysVInit 系统初始化的时候，它会将所有可能用到的后台service进程全部启动运行。并且系统必须等待所有的service都启动就绪之后，才允许用户登录。这种做法有两个缺点：首先是启动时间过长；其次是系统资源浪费，很多service开启后可能很少用到。SystemD 可以提供按需启动的能力，只有在某个service被真正请求的时候才启动它。当该service结束，SystemD 可以关闭它，等待下次需要时再次启动它。
- SystemD采用Cgroup 特性跟踪和管理进程的生命周期。init 系统的一个重要职责就是负责跟踪和管理service进程的生命周期。它不仅可以启动一个service，也必须也能够停止service。service进程一般都会作为deamon（daemon）在后台运行，为此service程序有时候会fork(fork)两次。在 UpStart 中，需要在配置文件中正确地配置 expect 小节。这样 UpStart 通过对 fork 系统调用进行计数，从而获知真正的deamon的 PID 号。
![](find_pid.jpg)
如果 UpStart 找错了，将 p1作为service进程的 Pid，那么停止service的时候，UpStart 会试图杀死 p1进程，而真正的 p1进程则继续执行。换句话说该service就失去控制了。还有更加特殊的情况。比如，一个 CGI 程序会fork两次，从而脱离了和 Apache 的父子关系。当 Apache 进程被停止后，该 CGI 程序还在继续运行。而我们希望service停止后，所有由它所启动的相关进程也被停止。为了处理这类问题，UpStart 通过 strace 来跟踪 fork、exit 等系统调用，但是这种方法很笨拙，且缺乏可扩展性。SystemD 则利用了 Linux 内核的特性即 CGroup 来完成跟踪的任务。当停止service时，通过查询 CGroup，SystemD 可以确保找到所有的相关进程，从而干净地停止service。CGroup 已经出现了很久，它主要用来实现系统资源配额管理。CGroup 提供了类似文件系统的接口，使用方便。当进程创建子进程时，子进程会继承父进程的 CGroup。因此无论service如何启动新的子进程，所有的这些相关进程都会属于同一个 CGroup，SystemD 只需要简单地遍历指定的 CGroup 即可正确地找到所有的相关进程，将它们一一停止即可。
- 启动挂载点和自动挂载的管理。传统的 Linux 系统中，用户可以用/etc/fstab 文件来维护固定的文件系统挂载点。这些挂载点在系统启动过程中被自动挂载，一旦启动过程结束，这些挂载点就会确保存在。这些挂载点都是对系统运行至关重要的文件系统，比如 HOME 目录。和 SysVInit 一样，SystemD 管理这些挂载点，以便能够在系统启动时自动挂载它们。SystemD 还兼容/etc/fstab 文件，您可以继续使用该文件管理挂载点。有时候用户还需要U盘等动态挂载点，SysVInit依赖autofs service来实现这种功能。SystemD内建了自动挂载service，可以直接使用 SystemD提供的自动挂载管理能力来实现autofs的功能。
- 实现事务性依赖关系管理。系统启动过程是由很多的独立job共同组成的，这些job之间可能存在依赖关系，比如挂载一个 NFS 文件系统必须依赖网络能够正常job。SystemD 虽然能够最大限度地并发执行很多有依赖关系的job，但是类似"挂载 NFS"和"启动网络"这样的job还是存在天生的先后依赖关系，无法并发执行。对于这些任务，SystemD 维护一个"事务一致性"的概念，保证所有相关的service都可以正常启动而不会出现互相依赖，以至于死锁的情况。
- 能够对系统进行快照和恢复。SystemD 支持按需启动，因此系统的运行状态是动态变化的，人们无法准确地知道系统当前运行了哪些service。SystemD 快照提供了一种将当前系统运行状态保存并恢复的能力。比如系统当前正运行service A 和 B，可以用 SystemD 命令行对当前系统运行状况创建快照。然后将进程 A 停止，或者做其他的任意的对系统的改变，比如启动新的进程 C。在这些改变之后，运行 SystemD 的快照恢复命令，就可立即将系统恢复到快照时刻的状态，即只有service A，B 在运行。一个可能的应用场景是调试：比如service器出现一些异常，为了调试用户将当前状态保存为快照，然后可以进行任意的操作，比如停止service等等。等调试结束，恢复快照即可。这个快照功能目前在 SystemD 中并不完善，似乎开发人员也没有特别关注它，因此有报告指出它还存在一些使用上的问题，使用时尚需慎重。
- 日志service。SystemD 自带日志service journald，该日志service的设计初衷是克服现有的 syslog service的缺点。比如：syslog 不安全，消息的内容无法验证。每一个本地进程都可以声称自己是 Apache PID 4711，而 syslog 也就相信并保存到磁盘上。数据没有严格的格式，非常随意。自动化的日志分析器需要分析人类语言字符串来识别消息。一方面此类分析困难低效；此外日志格式的变化会导致分析代码需要更新甚至重写。SystemD Journal 用二进制格式保存所有日志信息，用户使用 journalctl 命令来查看日志信息。无需自己编写复杂脆弱的字符串分析处理程序。
SystemD Journal 的优点如下：
- 简单性：代码少，依赖少，抽象开销最小。
- 零维护：日志是除错和监控系统的核心功能，因此它自己不能再产生问题。举例说，自动管理磁盘空间，避免由于日志的不断产生而将磁盘空间耗尽。
- 移植性：日志 文件应该在所有类型的 Linux 系统上可用，无论它使用的何种 CPU 或者字节序。
- 性能：添加和浏览 日志 非常快。
- 最小资源占用：日志 数据文件需要较小。
- 统一化：各种不同的日志存储技术应该统一起来，将所有的可记录event保存在同一个数据存储中。所以日志内容的全局上下文都会被保存并且可供日后查询。例如一条固件记录后通常会跟随一条内核记录，最终还会有一条用户态记录。重要的是当保存到硬盘上时这三者之间的关系不会丢失。Syslog 将不同的信息保存到不同的文件中，分析的时候很难确定哪些条目是相关的。
- 扩展性：日志的适用范围很广，从嵌入式设备到超级计算机集群都可以满足需求。
- 安全性：日志 文件是可以验证的，让无法检测的修改不再可能。

### unit
一个service可以认为是一个uit；一个挂载点是一个unit；一个交换分区的配置是一个unit；等等。SystemD 将unit归纳为以下一些不同的类型。
- service ：代表一个后台service进程，比如 mysqld。这是最常用的一类。
- socket ：此类配置单元封装系统和互联网中的一个 套接字 。当下，SystemD 支持流式、数据报和连续包的 AF_INET、AF_INET6、AF_UNIX socket 。每一个套接字配置单元都有一个相应的service配置单元 。相应的service在第一个"连接"进入套接字时就会启动(例如：nscd.socket 在有新连接后便启动 nscd.service)。
- device ：此类配置单元封装一个存在于 Linux 设备树中的设备。每一个使用 udev 规则标记的设备都将会在 SystemD 中作为一个设备配置单元出现。
- mount ：此类配置单元封装文件系统结构层次中的一个挂载点。SystemD 将对这个挂载点进行监控和管理。比如可以在启动时自动将其挂载；可以在某些条件下自动卸载。SystemD 会将/etc/fstab 中的条目都转换为挂载点，并在开机时处理。
- automount ：此类配置单元封装系统结构层次中的一个自挂载点。每一个自挂载配置单元对应一个挂载配置单元 ，当该自动挂载点被访问时，SystemD 执行挂载点中定义的挂载行为。
- swap: 和挂载配置单元类似，交换配置单元用来管理交换分区。用户可以用交换配置单元来定义系统中的交换分区，可以让这些交换分区在启动时被激活。
- target ：此类配置单元为其他配置单元进行逻辑分组。它们本身实际上并不做什么，只是引用其他配置单元而已。这样便可以对配置单元做一个统一的控制。这样就可以实现大家都已经非常熟悉的运行级别概念。比如想让系统进入图形化模式，需要运行许多service和配置命令，这些操作都由一个个的配置单元表示，将所有这些配置单元组合为一个目标(target)，就表示需要将这些配置单元全部执行一遍以便进入目标所代表的系统运行状态。 (例如：multi-user.target 相当于在传统使用 SysV 的系统中运行级别 5)
- timer：定时器配置单元用来定时触发用户定义的操作，这类配置单元取代了 atd、crond 等传统的定时service。
- snapshot ：与 target 配置单元相似，快照是一组配置单元。它保存了系统当前的运行状态。
每个配置单元都有一个对应的配置文件，系统管理员的任务就是编写和维护这些不同的配置文件，比如一个 MySQL service对应一个 mysql.service 文件。这种配置文件的语法非常简单，用户不需要再编写和维护复杂的系统 5 脚本了。

### 依赖关系
虽然 SystemD 将大量的启动job解除了依赖，使得它们可以并发启动。但还是存在有些任务，它们之间存在天生的依赖，不能用"套接字激活"(socket activation)、D-Bus activation 和 autofs 三大方法来解除依赖（三大方法详情见后续描述）。比如：挂载必须等待挂载点在文件系统中被创建；挂载也必须等待相应的物理设备就绪。为了解决这类依赖问题，SystemD 的配置单元之间可以彼此定义依赖关系。
SystemD 用配置单元定义文件中的关键字来描述配置单元之间的依赖关系。比如：unit A 依赖 unit B，可以在 unit B 的定义中用"require A"来表示。这样 SystemD 就会保证先启动 A 再启动 B。

### SystemD 事务
SystemD 能保证事务完整性。SystemD 的事务概念和数据库中的有所不同，主要是为了保证多个依赖的配置单元之间没有环形引用。比如 unit A、B、C，假如它们的依赖关系为:
![](recurrent_dependency.jpg)
存在循环依赖，那么 SystemD 将无法启动任意一个service。此时 SystemD 将会尝试解决这个问题，因为配置单元之间的依赖关系有两种：required 是强依赖；want 则是弱依赖，SystemD 将去掉 wants 关键字指定的依赖看看是否能打破循环。如果无法修复，SystemD 会报错。
SystemD 能够自动检测和修复这类配置错误，极大地减轻了管理员的排错负担。

### Target 和运行级别
SystemD 用目标（target）替代了运行级别的概念，提供了更大的灵活性，如您可以继承一个已有的目标，并添加其它service，来创建自己的目标。下表列举了 SystemD 下的目标和常见 runlevel 的对应关系：
SysVInit 运行级别|SystemD 目标|备注
0|runlevel0.target, poweroff.target|关闭系统。
1, s, single|runlevel1.target, rescue.target|单用户模式。
2, 4|runlevel2.target, runlevel4.target, multi-user.target|用户定义/域特定运行级别。默认等同于 3。
3|runlevel3.target, multi-user.target|多用户，非图形化。用户可以通过多个控制台或网络登录。
5|runlevel5.target, graphical.target|多用户，图形化。通常为所有运行级别 3 的service外加图形化登录。
6|runlevel6.target, reboot.target|重启
emergency|emergency.target|紧急Shell

### SystemD 的并发启动原理
如前所述，在 SystemD 中，所有的service都并发启动，比如 Avahi、D-Bus、livirtd、X11、HAL 可以同时启动。乍一看，这似乎有点儿问题，比如 Avahi 需要 syslog 的service，Avahi 和 syslog 同时启动，假设 Avahi 的启动比较快，所以 syslog 还没有准备好，可是 Avahi 又需要记录日志，这岂不是会出现问题？
SystemD 的开发人员仔细研究了service之间相互依赖的本质问题，发现所谓依赖可以分为三个具体的类型，而每一个类型实际上都可以通过相应的技术解除依赖关系。
并发启动原理之一：解决 socket 依赖
绝大多数的service依赖是套接字依赖。比如service A 通过一个套接字端口 S1 提供自己的service，其他的service如果需要service A，则需要连接 S1。因此如果service A 尚未启动，S1 就不存在，其他的service就会得到启动错误。所以传统地，人们需要先启动service A，等待它进入就绪状态，再启动其他需要它的service。SystemD 认为，只要我们预先把 S1 建立好，那么其他所有的service就可以同时启动而无需等待service A 来创建 S1 了。如果service A 尚未启动，那么其他进程向 S1 发送的service请求实际上会被 Linux 操作系统缓存，其他进程会在这个请求的地方等待。一旦service A 启动就绪，就可以立即处理缓存的请求，一切都开始正常运行。
那么service如何使用由 init 进程创建的套接字呢？
Linux 操作系统有一个特性，当进程调用 fork 或者 exec 创建子进程之后，所有在父进程中被打开的文件句柄 (file descriptor) 都被子进程所继承。套接字也是一种文件句柄，进程 A 可以创建一个套接字，此后当进程 A 调用 exec 启动一个新的子进程时，只要确保该套接字的 close_on_exec 标志位被清空，那么新的子进程就可以继承这个套接字。子进程看到的套接字和父进程创建的套接字是同一个系统套接字，就仿佛这个套接字是子进程自己创建的一样，没有任何区别。
这个特性以前被一个叫做 inetd 的系统service所利用。Inetd 进程会负责监控一些常用套接字端口，比如 Telnet，当该端口有连接请求时，inetd 才启动 telnetd 进程，并把有连接的套接字传递给新的 telnetd 进程进行处理。这样，当系统没有 telnet 客户端连接时，就不需要启动 telnetd 进程。Inetd 可以代理很多的网络service，这样就可以节约很多的系统负载和内存资源，只有当有真正的连接请求时才启动相应service，并把套接字传递给相应的service进程。
和 inetd 类似，SystemD 是所有其他进程的父进程，它可以先建立所有需要的套接字，然后在调用 exec 的时候将该套接字传递给新的service进程，而新进程直接使用该套接字进行service即可。
并发启动原理之二：解决 D-Bus 依赖
D-Bus 是 desktop-bus 的简称，是一个低延迟、低开销、高可用性的进程间通信机制。它越来越多地用于应用程序之间通信，也用于应用程序和操作系统内核之间的通信。很多现代的service进程都使用D-Bus 取代套接字作为进程间通信机制，对外提供service。比如简化 Linux 网络配置的 NetworkManager service就使用 D-Bus 和其他的应用程序或者service进行交互：邮件客户端软件 evolution 可以通过 D-Bus 从 NetworkManager service获取网络状态的改变，以便做出相应的处理。
D-Bus 支持所谓"bus activation"功能。如果service A 需要使用service B 的 D-Bus service，而service B 并没有运行，则 D-Bus 可以在service A 请求service B 的 D-Bus 时自动启动service B。而service A 发出的请求会被 D-Bus 缓存，service A 会等待service B 启动就绪。利用这个特性，依赖 D-Bus 的service就可以实现并行启动。
并发启动原理之三：解决文件系统依赖
系统启动过程中，文件系统相关的活动是最耗时的，比如挂载文件系统，对文件系统进行磁盘检查（fsck），磁盘配额检查等都是非常耗时的操作。在等待这些job完成的同时，系统处于空闲状态。那些想使用文件系统的service似乎必须等待文件系统初始化完成才可以启动。但是 SystemD 发现这种依赖也是可以避免的。
SystemD 参考了 autofs 的设计思路，使得依赖文件系统的service和文件系统本身初始化两者可以并发job。autofs 可以监测到某个文件系统挂载点真正被访问到的时候才触发挂载操作，这是通过内核 automounter 模块的支持而实现的。比如一个 open()系统调用作用在"/misc/cd/file1"的时候，/misc/cd 尚未执行挂载操作，此时 open()调用被挂起等待，Linux 内核通知 autofs，autofs 执行挂载。这时候，控制权返回给 open()系统调用，并正常打开文件。
SystemD 集成了 autofs 的实现，对于系统中的挂载点，比如/home，当系统启动的时候，SystemD 为其创建一个临时的自动挂载点。在这个时刻/home 真正的挂载设备尚未启动好，真正的挂载操作还没有执行，文件系统检测也还没有完成。可是那些依赖该目录的进程已经可以并发启动，他们的 open()操作被内建在 SystemD 中的 autofs 捕获，将该 open()调用挂起（可中断睡眠状态）。然后等待真正的挂载操作完成，文件系统检测也完成后，SystemD 将该自动挂载点替换为真正的挂载点，并让 open()调用返回。由此，实现了那些依赖于文件系统的service和文件系统本身同时并发启动。
当然对于"/"根目录的依赖实际上一定还是要串行执行，因为 SystemD 自己也存放在/之下，必须等待系统根目录挂载检查好。
不过对于类似/home 等挂载点，这种并发可以提高系统的启动速度，尤其是当/home 是远程的 NFS 节点，或者是加密盘等，需要耗费较长的时间才可以准备就绪的情况下，因为并发启动，这段时间内，系统并不是完全无事可做，而是可以利用这段空余时间做更多的启动进程的事情，总的来说就缩短了系统启动时间。

### SystemD 的使用
- 后台service进程代码不需要执行两次fork来实现后台deamon，只需要实现service本身的主循环即可。
- 不要调用 setsid()，交给 SystemD 处理
- 不再需要维护 pid 文件。
- SystemD 提供了日志功能，service进程只需要输出到 stderr 即可，无需使用 syslog。
- 处理信号 SIGTERM，这个信号的唯一正确作用就是停止当前service，不要做其他的事情。
- SIGHUP 信号的作用是重启service。
- 需要套接字的service，不要自己创建套接字，让 SystemD 传入套接字。
- 使用 sd_notify()函数通知 SystemD service自己的状态改变。一般地，当service初始化结束，进入service就绪状态时，可以调用它。
- Unit 文件的编写
下面是 SSH service的配置单元文件，service配置单元文件以.service 为文件名后缀。
~$:cat /etc/system/system/sshd.service
``` txt
[Unit]
Description=OpenSSH server daemon
[Service]
EnvironmentFile=/etc/sysconfig/sshd
ExecStartPre=/usr/sbin/sshd-keygen
ExecStart=/usrsbin/sshd –D $OPTIONS
ExecReload=/bin/kill –HUP $MAINPID
KillMode=process
Restart=on-failure
RestartSec=42s
[Install]
WantedBy=multi-user.target
```
[Unit]部分仅仅有一个描述信息；Service 定义中，ExecStartPre 定义启动service之前应该运行的命令；ExecStart 定义启动service的具体命令行语法。[Install]部分，WangtedBy 表明这个service是在多用户模式下所需要的，multi-user.target 
~#:cat multi-user.target
``` txt
[Unit]
Description=Multi-User System
Documentation=man.SystemD.special(7)
Requires=basic.target
Conflicts=rescue.service rescure.target
After=basic.target rescue.service rescue.target
AllowIsolate=yes
[Install]
Alias=default.target
```
第一部分中的 Requires 定义表明 multi-user.target 启动的时候 basic.target 也必须被启动；另外 basic.target 停止的时候，multi-user.target 也必须停止。如果您接着查看 basic.target 文件，会发现它又指定了 sysinit.target 等其他的单元必须随之启动。同样 sysinit.target 也会包含其他的单元。采用这样的层层链接的结构，最终所有需要支持多用户模式的组件service都会被初始化启动好。
在[Install]小节中有 Alias 定义，即定义本单元的别名，这样在运行 systemctl 的时候就可以使用这个别名来引用本单元。这里的别名是 default.target，比 multi-user.target 要简单一些。。。
此外在/etc/SystemD/system 目录下还可以看到诸如\*.wants 的目录，放在该目录下的配置单元文件等同于在[Unit]小节中的 wants 关键字，即本单元启动时，还需要启动这些单元。比如您可以简单地把您自己写的 foo.service 文件放入 multi-user.target.wants 目录下，这样每次都会被默认启动了。
最后，让我们来看看 sys-kernel-debug.mount 文件，这个文件定义了一个文件挂载点：
~#:cat sys-kernel-debug.mount
``` txt
[Unit]
Description=Debug File Syste
DefaultDependencies=no
ConditionPathExists=/sys/kernel/debug
Before=sysinit.target
[Mount]
What=debugfs
Where=/sys/kernel/debug
Type=debugfs
```
这个配置单元文件定义了一个挂载点。挂载配置单元文件有一个[Mount]配置小节，里面配置了 What，Where 和 Type 三个数据项。这都是挂载命令所必须的，例子中的配置等同于下面这个挂载命令：
~#:mount –t debugfs /sys/kernel/debug debugfs
init 系统的管理通过service、chkconfig 以及 telinit 命令进行。SystemD 也完成同样的管理任务，只是命令工具 systemctl 的语法有所不同而已，因此用表格来对比 systemctl 和传统的系统管理命令会非常清晰。
SysVInit 命令|SystemD 命令|备注
service foo start|systemctl start foo.service	用来启动一个service (并不会重启现有的)
service foo stop|systemctl stop foo.service	用来停止一个service (并不会重启现有的)。
service foo restart|systemctl restart foo.service	用来停止并启动一个service。
service foo reload|systemctl reload foo.service	当支持时，重新装载配置文件而不中断等待操作。
service foo condrestart|systemctl condrestart foo.service	如果service正在运行那么重启它。
service foo status|systemctl status foo.service	汇报service是否正在运行。
ls /etc/rc.d/init.d/|systemctl list-unit-files --type=service	用来列出可以启动或停止的service列表。
chkconfig foo on|systemctl enable foo.service	在下次启动时或满足其他触发条件时设置service为启用
chkconfig foo off|systemctl disable foo.service	在下次启动时或满足其他触发条件时设置service为禁用
chkconfig foo|systemctl is-enabled foo.service	用来检查一个service在当前环境下被配置为启用还是禁用。
chkconfig –list|systemctl list-unit-files --type=service	输出在各个运行级别下service的启用和禁用情况
chkconfig foo –list|ls /etc/SystemD/system/\*.wants/foo.service	用来列出该service在哪些运行级别下启用和禁用。
chkconfig foo –add|systemctl daemon-reload	当您创建新service文件或者变更设置时使用。
telinit 3|systemctl isolate multi-user.target (OR systemctl isolate runlevel3.target OR telinit 3)	改变至多用户运行级别。

SystemD 电源管理命令
命令|操作
systemctl reboot|重启机器
systemctl poweroff|关机
systemctl suspend|待机
systemctl hibernate|休眠
systemctl hybrid-sleep|混合休眠模式（同时休眠到硬盘并待机）
关机不是每个登录用户在任何情况下都可以执行的，一般只有管理员才可以关机。正常情况下系统不应该允许 SSH 远程登录的用户执行关机命令。否则其他用户正在job，一个用户把系统关了就不好了。为了解决这个问题，传统的 Linux 系统使用 ConsoleKit 跟踪用户登录情况，并决定是否赋予其关机的权限。现在 ConsoleKit 已经被 SystemD 的 logind 所替代。
logind 不是 pid-1 的 init 进程。它的作用和 UpStart 的 session init 类似，但功能要丰富很多，它能够管理几乎所有用户会话(session)相关的事情。logind 不仅是 ConsoleKit 的替代，它可以：
- 维护，跟踪会话和用户登录情况。如上所述，为了决定关机命令是否可行，系统需要了解当前用户登录情况，如果用户从 SSH 登录，不允许其执行关机命令；如果普通用户从本地登录，且该用户是系统中的唯一会话，则允许其执行关机命令；这些判断都需要 logind 维护所有的用户会话和登录情况。
- Logind 也负责统计用户会话是否长时间没有操作，可以执行休眠/关机等相应操作。
- 为用户会话的所有进程创建 CGroup。这不仅方便统计所有用户会话的相关进程，也可以实现会话级别的系统资源控制。
- 负责电源管理的组合键处理，比如用户按下电源键，将系统切换至睡眠状态。
- 多席位(multi-seat) 管理。如今的电脑，即便一台笔记本电脑，也完全可以提供多人同时使用的计算能力。多席位就是一台电脑主机管理多个外设，比如两个屏幕和两个鼠标/键盘。席位一使用屏幕 1 和键盘 1；席位二使用屏幕 2 和键盘 2，但他们都共享一台主机。用户会话可以自由在多个席位之间切换。或者当插入新的键盘，屏幕等物理外设时，自动启动 gdm 用户登录界面等。所有这些都是多席位管理的内容。ConsoleKit 始终没有实现这个功能，SystemD 的 logind 能够支持多席位。

## SysVInit，Upstart，SystemD比较
SysVInit比较简单。Service开发人员只需要编写启动和停止脚本，将 service 添加/删除到某个 runlevel 时，只需要执行一些创建/删除软连接文件的基本操作。
其次，SysVInit 的另一个优点是确定的执行顺序：脚本严格按照启动数字的大小顺序执行，一个执行完毕再执行下一个，这非常有益于错误排查。UpStart 和 SystemD 支持并发启动，导致没有人可以确定地了解具体的启动顺序，排错不易。但是串行地执行脚本导致 SysVInit 运行效率较慢。
可以看到，UpStart 的设计比 SysVInit 更加先进。
SystemD 的最大特点有两个：并发启动能力强，极大地提高了系统启动速度；用 CGroup 统计跟踪子进程，干净可靠。
此外，和其前任不同的地方在于，SystemD 已经不仅仅是一个初始化系统了。SystemD 出色地替代了 SysVInit 的所有功能，但它并未就此自满。因为 init 进程是系统所有进程的父进程这样的特殊性，SystemD 非常适合提供曾经由其他service提供的功能，比如定时任务 (以前由 crond 完成)；会话管理 (以前由 ConsoleKit/PolKit 等管理) 。


## 参考文献
1.https://www.ibm.com/developerworks/cn/linux/1407_liuming_init1/index.html
2.https://www.ibm.com/developerworks/cn/linux/1407_liuming_init2/index.html?ca=drs-
3.https://www.ibm.com/developerworks/cn/linux/1407_liuming_init3/index.html?ca=drs-

