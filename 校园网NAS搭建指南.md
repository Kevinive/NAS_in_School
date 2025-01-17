# 校园网NAS搭建指南

尝试在校园网内搭建一个NAS服务器，并利用IPV6的特性使其在整个校园内可以提供文件服务和媒体串流服务。

<u>关键词：NAS; 群晖; 锐捷网络认证; 阿里云DNS; IPV6; PLEX;</u>

## 0.前言

从很早开始，就梦想自己拥有一台NAS。在国内，针对个人和家庭用户的NAS产品中，群晖应该是做的最好的。我也想拥有一台群晖NAS，但一方面因为没有实际使用过带完整功能的NAS，不清楚NAS能对我的生活产生多大影响；另一方面，群晖系列的产品价格确实超出了学生党的承受范围（人话：没钱）。因此我迟迟没有下手。

而2019年暑假，因为种种机缘巧合，我同时找到了我使用NAS的需求和可以接受的解决方案，于是毫不犹豫地开始了探索之路，并把过程记录下来，供有需要的人（主要是在校大学生）参考。



## 1.系统配置和功能

### 1.1配置

**硬件**：(二手) 蜗牛星际NAS四盘位，J1900四核CPU，4G内存，自带16G固态硬盘，一块(二手)2T机械硬盘，两块1T机械硬盘。

**软件**：商家自带的黑裙晖6.2.2 系统

前段时间矿难，正好捡了个矿渣，机箱+一块2T硬盘花费600+，学生党表示负担得起，很满足。由于笔者宿舍要交电费，挥霍不起，因此才专门买一个低功耗的NAS主机来担当服务器的作用。

*（一般的NAS主机运行功耗<30W。因为NAS的性质，基本上要7*24小时工作，算下来也将是一笔不小的电费，因此还是建议有能力的朋友上专门的主机）。*

另外两块1T硬盘是笔者自带的闲置硬盘。

以上配置仅供参考，理论上**任何配置的主机只要有群晖系统就能够实现这个项目的全部功能，因此手头富裕的同学也可以直接上正版群晖。**

*（黑裙晖稳定性不佳，需要费时间折腾，因此笔者以后有经济实力了也会选择正版群晖）*

### 1.2实现功能

- 自动登录校园网（以武汉大学的锐捷网络认证为例）

- 基于IPV4的内网穿透

- 基于IPV6的NAS直连

- 文件服务器、流媒体服务器




## 2.服务器设置与文件管理

服务器搭起来，首先肯定要设置。

由于NAS的特殊性，为了方便设置，我强烈建议大家**前期使用路由器**，将NAS隐藏在内网内，这样可以在不用实现 <u>4.校园网登录功能</u> 的情况下为NAS提供完整的网络访问环境，方便搭建。

群晖的搭建网上有很多教程了，各位同学可以自行摸索和百度。

在这里只要求：进入控制面板->终端&SNMP->打开ssh服务 和 控制面板->用户->高级（选项卡）->（最下面）启用home文件夹



其实看到这里，将群晖设置起来后，你已经有了一个能够在自己路由器后面提供服务的NAS服务器。它已经能做许多事情了：FTP文件服务，本地媒体串流，BT下载，主动备份等功能。

但你仍有许多事情是不能做的：**你完全不能在寝室之外（或者说路由器wifi覆盖范围之外）访问到它，它真真正正就是一台本地服务器。**

而接下来我们要做的，就是让更多的地方能够访问到它。

过程不算简单，可能会遇到很多麻烦。但最后搭建完成时的喜悦与爽感一定也是非同寻常的。

好的我们继续往下做。



## 3.基于云服务器的内网穿透

> 为什么要使用内网穿透：一般而言，校园内的网络是一个很大的局域网。局域网的特性决定了在其之外的计算机无法直接访问到局域网内到计算机，而内网计算机访问外网服务器则没有问题。
>

我们现在将服务器隐藏在自己的内网里，也就是说**内网之外没人能访问到它**，获得它的服务。

内网穿透则是**利用公共的云服务器做中介**（任何人都能访问得到）提供一种转发服务：处于校园网内网深处的服务器在启动时主动寻找在外网的云服务器，建立数据链路；当有计算机想要访问在校园网深处的服务器时，它无法直接访问内网服务器，因此它也连接云服务器，建立数据链路，由云服务器将数据转发给内网内的服务器。云服务器在此就如同一个透明中介，只起到转发作用。这样设置完之后，用户就可以在任何有网络的地方访问到处于内网深处的服务器了。

提供内网穿透的服务商有很多，如Oray公司的花生壳。由于笔者在腾讯云上有一台学生服务器，因此果断选择使用自己的服务器做内网穿透。使用花生壳做内网穿透的步骤就不赘述。

> PS：腾讯云与阿里云对高校学生申请服务器都有较大折扣，笔者使用的服务器是腾讯云上的2核2G内存1Mbps带宽的学生服务器，租金每月12元，还送一个公网IP（玩服务器的同学都知道在IPV4年代一个公网IP有多么珍贵）。因此建议喜欢折腾的同学可以考虑在云上养一台服务器 ：）
>

内网透传方案我选择的是frp（全称 Fast Reverse Proxy），是一个高性能的内网透传解决方案。

------

### 具体步骤：

##### 3.1复制文件

打开 frp 文件夹，编辑frps.ini，更改 bind_port 和 bind_udp_port 至你喜欢的端口（两个不要重复了，或者不改也可以），然后进入云运营商的控制面板，设置安全组，放行此端口（如果出现连接不上的错误大概率就是云端有防火墙或者没设置开放端口）；

##### 3.2 编辑fprc.ini，更改以下设置

[common]内，server_addr 更改为你的服务器地址；server_port更改为你frps.ini中的bind_port；

[ssh] 以及之下的相关配置内，仅更改 remote_port 即可（这个属性决定了了云服务器会将哪个端口映射到你内网的 local_port 上。

例如：remote_port = 50022；local_port = 22 那么当其他计算机访问：（你云服务器IP）:50022 时，就等于访问你内网服务器的22号端口，也就是提供ssh服务的端口。

PS:可以根据你具体要开放哪些服务，来设置相应的端口。

##### 3.3 上传 

上传frps 和 frps.ini 至云端服务器内；上传 frpc 和 frpc.ini 和 frpc_start.sh 至群晖内

##### 3.4 在云端后台运行 frps

笔者使用 screen 执行后台运行的任务（ubuntu下可以执行 sudo apt-get install screen 来安装）

命令行（伪代码）：

```bash
screen -S frpServer

(进入 screen 内后）

cd （你存放frps的位置）

sudo ./frps -c ./frps.ini
```

观察输出，看frps是否正常启动

如果要退出，按ctrl+a, d即可退出screen，此时程序仍然在后台运行

PS 如果出现无法运行的情况，可能是因为你服务器的系统架构和frp可执行文件支持的不太一样。如果遇到这种情况，自行查找对应架构的frp文件即可。

##### 3.5 在NAS运行frpc

通过 file station 找到你存放frpc等文件的位置，右键其中一个文件，查看属性，复制文件位置信息。

通过文本编辑器打开frpc_start.sh，将第一行cd后面的路径删掉，粘贴刚刚复制的路径（记得删除路径最后的文件名）。

通过ssh登录NAS，找到存放frpc的文件夹，输入

```bash
sudo ./frpc -c ./frpc.ini
```

观察运行情况，查看是否有报错。（如果遇到报错可以百度一下，我没遇到过报错所以我也不知道会报什么错）

测试成功后按ctrl+c结束任务

##### 3.6 设置开机启动

以管理员登录群晖主页面，控制面板->定时任务->新建，名字自己取，执行用户选root，在第二章选项卡中的下面填入

```bash
bash （你的frpc完整路径）/frpc_start.sh
```

（什么？你不记得路径了？回去看看吧）

保存，退出，重启群晖

##### 3.7 确认是否正常工作

可以直接尝试用你的手机访问你云服务器的ip和你设置的端口（比如用NAS的5000访问管理页面），查看其是否能正常工作。

如果出现错误，可以回云服务器的screen看是否出现错误信息。

------

至此，设置内网穿透的过程大致完成。

理想情况下，你应该能够在**任何有网络的地方**通过云服务器IP+端口的方式访问你校园网内服务器的相关服务了。

如果再骚一点，可以买个域名，做个域名解析（需要实名认证），就可以很酷地访问你自己的NAS了。

当然，内网透传做到这一步还是有些问题的。之前已经介绍过了内网透传的原理，所有的流量都要流经我们的云服务器。虽然我们的云服务器不是按流量来收费的，但服务器带宽成为了我们最大的瓶颈：**NAS本身就是用来存储数据的，大量的数据需要传输，如果连接速度过慢（1Mbps的学生机网速撑死100KB/s），就算这个服务器可以在全世界被访问，它也失去了存在的意义**。

不过看官您莫着急，咱先把网络登录解决了，之后还有搞头呢 ：）



## 4.自动登录校园网

在校园网内搭建NAS服务器可能会遇到的第二个问题就是联网。以武汉大学为例，校园网内使用了锐捷网络认证，每台设备在获得上网权限之前要先“登录”。而群晖系统没有桌面界面，一切设置靠网络页面来完成。如果你将NAS直接接入校园网服务器，那么学校的DHCP服务器会为它随机分配一个IP地址。又因为我们没法通过显示器来获取它的IP地址（或者说很麻烦），所以一旦我们将其接入学校网段，它便有可能淹没在学校的几千个其他设备中。

比较可行的解决方法是：**NAS开机后自行登录校园网，然后和云服务器握手建立连接，然后我们通过云服务器来访问到NAS，并获取到其当前IP。当获取到其当前IP后，我们便可以（在同一网段内）使用IP地址进行直连。**

因此，自主登录问题必须解决。

------

*（以下设置以武汉大学校园网环境为例）*

### 所需软件：

##### mentohust：

项目介绍地址：[https://wiki.ubuntu.org.cn/锐捷、赛尔认证MentoHUST](https://wiki.ubuntu.org.cn/锐捷、赛尔认证MentoHUST)

mentohust 是一个由华中科技大学（**感谢邻校的兄弟**！）的同学开发的支持Windows、Linux、Mac OS下锐捷认证的程序（附带支持赛尔认证）

##### 锐捷客户端(for Windows)：

下载页面：http://nic.whu.edu.cn/info/1023/1121.htm

这个客户端是用来测试登录和提取属性文件给mentohust用的

### 具体步骤：

##### 4.1 下载

下载mentohust_related 文件夹到本地

##### 4.2 上传

将 mentohust_related 文件夹上传至群晖中的home目录下（以下假设登录用户为admin，若非admin则以下步骤中出现admin的部分用自己的用户名替换即可）

##### 4.3 解压

进入mentohust_related 文件夹，解压 mentohust-0.3.1-2-x86_64.pkg.tar.gz 包至根目录，命令行：

```bash
cd ~/mentohust_related

sudo tar -xzvf mentohust-0.3.1-2-x86_64.pkg.tar.gz -C /
```

输入密码之后即可成功解压

##### 4.4 移动配置文件

将 8021x.exe、data.mpf、SuConfig.dat、W64N55.dll 文件移动至 /etc/mentohust 目录下（该目录需要新建）

命令行：

```bash
cd ~/mentohust_related

sudo mkdir /etc/mentohust

sudo cp 8021x.exe /etc/mentohust/8021.exe

sudo cp data.mpf /etc/mentohust/data.mpf

sudo cp SuConfig.dat /etc/mentohust/SuConfig.dat

sudo cp W64N55.dll /etc/mentohust/W64N55.dll
```



##### 4.5 更改 /etc/mentohust.conf 文件

命令行：

```bash
sudo vi /etc/mentohust.conf
```

检查其中的设置，具体如下：

```go
;组播地址类型选0

StartMode=0

;DHCP方式选 1（二次认证）

DhcpMode=1

;禁止后台运行，调试时候可查看登录情况

DaemonMode=0

;客户端版本号：选择你考入/etc/mentohust中8021x.exe的对应windows版本号

Version=5.28

;认证数据文件填写为数据文件所在文件夹，本文为/etc/mentohust/

DataFile=/etc/mentohust/
```



> #### vi 编辑器基本操作:
>
> i---进入编辑模式和切换插入/替换模式
>
> esc---退出编辑模式
>
> :wq---在非编辑模式下输入可以保存并退出
>
> :q!---在非编辑模式下输入可以强制退出（不保存）
>

##### 4.6 运行mentohust

命令行：

```bash
sudo mentohust -f /etc/mentohust/data.mpf
```

按照提示输入基本信息如：选择网卡 等

然后观察其是否登录成功

注意：主机必须直接暴露在校园网下才可能在此步骤登录成功。如果主机连接到校园网内的内网（比如你自己的路由器LAN端），由于和认证服务器不在一个网段，因此无法登录成功。在这种情况下，无法登录成功也没关系，继续之后的步骤。

##### 4.7 更改 /etc/mentohust.conf 文件，使其在后台运行

命令行：

```bash
sudo vi /etc/mentohust.conf
```

更改如下

```go
;启用后台运行并输出log文件

DaemonMode=3
```



##### 4.8 设置定时任务

登入群晖的管理页面，选择控制面板->定时任务->新建任务。第一个选项卡中，任务名：自己取名，选择使用root用户运行，在启动时运行。在第二个选项卡下面 运行命令 中填写

```
mentohust -f /etc/mentohust/data.mpf
```

然后保存。

同时，我们编辑一下IPV4内网穿透的定时任务，选择编辑，在 前序任务（大概这个名字）的选项卡中选择登录的任务，这样只有在登录之后才会尝试连接服务器的内网穿透服务。

##### 4.9 接入校园网

现在，同学可以尝试将NAS直接接入校园网的网口了。

重启路由器，如果一切正常的话，应该5分钟内能够通过第三步中实现的内网穿透服务重新访问到你的NAS。

------

### 可能遇到的问题：

- **登录不成功**：请参照[https://wiki.ubuntu.org.cn/锐捷、赛尔认证MentoHUST](https://wiki.ubuntu.org.cn/锐捷、赛尔认证MentoHUST)中给出的各所大学的解决方案进行修改
- **无法选择其他运营商**：这个问题暂时没解决，现在还只能用默认校园网的服务登录，不能像网页版一样选择电信或者移动，期待其他高手提出解决方案。

------

这一段的实现是比较艰辛的，可能会遇到各种问题，大家百度一下并沉着冷静应对吧。

如果你成功连接到你的NAS了，那么恭喜你，你的NAS已经学会了如何自己登录校园网，自己和云服务器握手开通内网穿透服务。

不仅如此，你服务器的服务范围和质量也进一步提升了：**只要是和你在同一网段下（网段的概念暂时不作解释，一般来说大概一栋宿舍楼的设备都在同一网段下）的设备，就能通过你NAS的IP地址与你进行直连。直连的好处在于：数据传输速度基本就是你的设备到NAS之间网线的带宽（通常为100Mbps，也就是10MB/s+)：这个速率足以支持在线播放大多数的全高清视频了。**

现在你可以和你隔壁宿舍的小伙伴分享你的NAS啦！可以分享学习资料，可以躺在床上抱着pad用媒体串流看下载的电影，可以分享互相下载的电影

怎么样？舒服吗？快乐吗？

**如果满足于此，那你可以开心地享受NAS带来的便利了～（真的，做到这样对大部分人就很够了）**

------

### 但是！

啥？**你想和你隔壁宿舍楼的哥们分享电影可你们不在一个网段下**？

啥？**你想在图书馆串流你NAS上的电影可是你们不在一个网段下**？（为什么会有人来图书馆看电影）

啥？**这个范围还不够大？还想再大一点？**



方法也不是没有，你看啊，frp还提供了点对点内网穿透服务，也就是说，通过走UDP协议，不用经过服务器，也可以点对点传输，

https://www.geek-share.com/detail/2765886877.html

这篇文章就告诉我们，**只要我们学校的NAT类型不是Symmetic，就可以用点对点传输**，

> 让我们来测一下
>
> ......
>
> ......
>
> ......
>
> 还真是Symmetic啊......T^T
>
> 那凉了呀，外面访问就只能通过云服务器了呀......
>
> 抱歉，我真的没办法了......
>



正当我躺在床上万念俱灰之际，突然一道从窗外照了进来，

它的名字是：

IPv6



## 5.基于IPV6的NAS直连

近几年，国内高校和三大运营商陆陆续续都开始部署IPv6，武汉大学也不例外。IPv6的出现主要是为了解决网络地址不足的问题。前面我也说到，在IPv4时代拥有一个公网IP是多么宝贵的一件事，因为现在地球上所有的上网设备加起来数量已经远远超过了IPv4地址数量。（因此我们不得不用内网的形式来节省IP）但到了IPv6时代，它们说能够为地球上每一粒沙子都分配一个IP。（沙子：我要IP有何用？）也就是说，进入了IPv6时代，内网的概念就将成为历史。而只要知道了对方设备的IPv6地址，就能通过这个地址直接访问到这个设备。

**好巧，我们校内网已经通了IPv6了。**

**好巧，我们的群晖也能自动获取IPv6地址。**

**好巧，我们的手机也已经是IPv6的地址了。**



如果你现在拿手机，接入4G网络，**输入你NAS上面的IPv6地址**，很大概率是能直接访问到的。

*（在浏览器内输入"[完整的ipv6地址]:5000"即可访问，建议使用Chrome浏览器，其他浏览器不知可否成功）*

这意味着什么？

这意味着只要你有IPv6的设备，你就能直接访问到你的NAS。

这意味着你的NAS服务范围瞬间从一个小网段扩展到了整个IPv6覆盖的网络内。

整个校园网都是支持IPv6的，你的手机也是支持IPv6的

**这个范围**

**够大了吧？**



那么现在的问题就是，你的IPv6地址很大程度也是会变的，也是动态分配的。而IPv6地址又贼难记

**所以你想到了DDNS。**

> DDNS 也就是 动态域名解析，能够将域名动态地指向你当前服务器的地址。

之前不使用这个服务的原因是因为**内网**，DDNS能帮助你将你的域名解析到公网进校园网的第一个路由器那，但由于整个校园里的设备对外都只有一个IP（可能不止一个），实际上外部的计算机并不能访问到你的NAS。

**但IPv6让你的NAS有名有姓了，通过唯一的IP，外部计算机可以找到唯一的你，并与你交换数据。**

因此我选择了阿里云的域名解析服务，网络上也有现成的教程，讲解如何设置IPV6的动态域名解析，在此也不再赘述

教程地址：https://nnboli.com/939.html

*(教程我随便贴的，大概思路就是：用阿里DNS设置解析到路由器当前地址->周期运行脚本查看解析是否正确，如果不正确就重新设置解析）*



至此，整个IPv6网络都能访问到你的NAS了

*（不管你爽不爽，反正我是很爽）*



## 6.BT种子

## 7.PLEX流媒体服务器搭建