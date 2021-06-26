---
layout: post
title: Arch Linux is Cool
---


For most my life with computers I have either used Windows 7 or Linux. Windows was great but once I finally decided to try Ubuntu I never went back. I tried out all sorts of Linux distros (ie. Elementary OS, Crunchbang, Linux Mint) but none of them seemed right for me. After doing some searching around and I came to the conclusion that I had to attempt the unthinkable, Arch Linux.

# 关于我的电脑
*Hardware*: The PC I mainly use is a desktop that I built a few years back with a Intel Core i5, Nvidia GTX 650ti, and 8 GB of RAM.

某君昆仲，今隐其名，皆余昔日在中学时良友；分隔多年，消息渐阙。日前偶闻其一大病；适归故乡，迂道往访，则仅晤一人，言病者其弟也。劳君远道来视，然已早愈，赴某地候补矣。因大笑，出示日记二册，谓可见当日病状，不妨献诸旧友。持归阅一过，知所患“迫害狂”之类。语颇错杂无伦次，又多荒唐之言；亦不着月日，惟墨色字体不一，知非一时所书。间亦有略具联络者，今撮录一篇，以供医家研究。记中语误，一字不易；惟人名虽皆村人，不为世间所知，无关大体，然亦悉易去。至于书名，则本人愈后所题，不复改也。七年四月二日识

*Usage*: On my computer I mainly write code using a basic text-editor (i.e. atom or sublime text) and browse the web. I sometime play video games on Steam, which include Civ5 and Besiege.


# The Dreaded Installation
Getting Arch Linux up and running on your computer really isn't that bad. Arch has great [documentation](https://www.archlinux.org/) and the [beginner's guide](https://wiki.archlinux.org/index.php/beginners'_guide) makes everything very straight forward. It still took me a couple of tries to get a stable desktop environment (graphic drivers were tricky) but once I got all the basic stuff taken care of I was finally able to begin making the operating system I really wanted.


# So Why use Arch?
I'd be lying if I said I didn't use Arch because it makes me feel like a hacker elitist. Also seeing all the beautiful desktops on [/r/unixporn](https://www.reddit.com/r/unixporn) also influenced my decision in using Arch. But, there are some legitimate reasons on why I prefer Arch over other Linux distros or any operating system for that matter:

* The best performance
* I choose what's installed
* Always the latest software
* Installing software is easy with pacman
* Very intelligent community
* Teaches you about how Linux works


# Conclusion
I really do see myself sticking with Arch for quite some time. With its rolling releases there will be no need to ever have to install any major updates and I have everything I really need in a computer. Though some things aren't perfect (ie. Steam isn't great yet) it's worth it for the gains in performance


# What's on my PC
Here's a short list of things I have installed on my main Arch Linux computer.

* Gnome 3 with [arc-theme](https://github.com/horst3180/arc-theme) and the [Paper icons](https://snwh.org/paper/icons/)
* bspwm (if I'm feeling extra hackerish)
* [mpv](https://mpv.io/) for all forms of media (ie. music and movies)
* Google's open source browser [chromium](https://wiki.archlinux.org/index.php/chromium)
* [atom-editor](https://aur.archlinux.org/packages/atom-editor-bin/) for coding (should probably learn vim better)

<br>

*Update:* I changed some of the "What's on my PC" entries due to the input on [reddit](https://www.reddit.com/r/archlinux/comments/4c4mmh/blog_post_arch_linux_is_cool/). Thanks!



---
titile:
layout: post
title: "supervisor守护python进程"
subtitle: ""
date: 2016-03-30
author: chao
category: Jekyll
tags: python shadowosocks 
finished: true
---

shadowosocks的后台，之前我一直用screen，然而总是几天就会挂掉，当然也是各主机商的机器各有差别，Digitalocean新加坡上的隔一两天天就挂一次，vultr东京基本一个月挂一次。<br>
半年多了，每次挂掉，只有一次又一次的执行那条无比蛋疼的的命令：

```screen -S python server.py
```

最近Digitalocean上的程序经常挂掉，让我十分头疼。最终还是用了supervisor。<br>
其原理，大概相当于把你想守护的进程做成supervisor的子进程，然后supervisor管理。

## 安装

```sudo pip install supervisor
```

或者

```apt-get install  supervisor
```

## 配置

安装后默认没有配置文件，需要手动生成。

```echo_supervisord_conf > /etc/supervisord.conf
```

不过我在vps上基本都是生成失败了，不过也没有很大的关系，手动的创建/etc/supervisord.conf即可：

```[unix_http_server]
file=/tmp/supervisor.sock   ; UNIX socket 文件，supervisorctl 会使用
;chmod=0700                 ; socket 文件的 mode，默认是 0700
;chown=nobody:nogroup       ; socket 文件的 owner，格式： uid:gid

[inet_http_server]         ; HTTP 服务器，提供 web 管理界面
port=*:9001        ; Web 管理后台运行的 IP 和端口，如果开放到公网，需要注意安全性
username=user              ; 登录管理后台的用户名
password=123               ; 登录管理后台的密码

[supervisord]
logfile=/tmp/supervisord.log ; 日志文件，默认是 $CWD/supervisord.log
logfile_maxbytes=50MB        ; 日志文件大小，超出会 rotate，默认 50MB
logfile_backups=10           ; 日志文件保留备份数量默认 10
loglevel=info                ; 日志级别，默认 info，其它: debug,warn,trace
pidfile=/tmp/supervisord.pid ; pid 文件
nodaemon=false               ; 是否在前台启动，默认是 false，即以 daemon 的方式启动
minfds=1024                  ; 可以打开的文件描述符的最小值，默认 1024
minprocs=200                 ; 可以打开的进程数的最小值，默认 200

; the below section must remain in the config file for RPC
; (supervisorctl/web interface) to work, additional interfaces may be
; added by defining them in separate rpcinterface: sections
[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///tmp/supervisor.sock ; 通过 UNIX socket 连接 supervisord，路径与 unix_http_server 部分的 file 一致
;serverurl=http://127.0.0.1:9001 ; 通过 HTTP 的方式连接 supervisord

; 包含其他的配置文件
[include]
files = /etc/supervisor/*.conf   ; 可以是 *.conf 或 *.ini
```

这里的配置是将配置文件分成两部分，/etc/supervisord.conf为基本配置，有关守护的进程的配置都放到/etc/supervisor/下，该文件夹为自己手动创建。当然全写在一个文件里也是可以的。

再创建进程的配置文件/etc/supervisor/supervisor.conf


```[program:pyss]
directory = /root/manyuser/shadowosocks ; 程序的启动目录
command = python server.py ; 启动命令，可以看出与手动在命令行启动的命令是一样的
autostart = true     ; 在 supervisord 启动的时候也自动启动
startsecs = 5        ; 启动 5 秒后没有异常退出，就当作已经正常启动了
autorestart = true   ; 程序异常退出后自动重启
startretries = 3     ; 启动失败自动重试次数，默认是 3
user = leon          ; 用哪个用户启动
redirect_stderr = true  ; 把 stderr 重定向到 stdout，默认 false
stdout_logfile_maxbytes = 20MB  ; stdout 日志文件大小，默认 50MB
stdout_logfile_backups = 20     ; stdout 日志文件备份数
; stdout 日志文件，需要注意当指定目录不存在时无法正常启动，所以需要手动创建目录（supervisord 会自动创建日志文件）
stdout_logfile = /data/logs/usercenter_stdout.log

; 可以通过 environment 来添加需要的环境变量，一种常见的用法是修改 PYTHONPATH
; environment=PYTHONPATH=$PYTHONPATH:/path/to/somewhere
```

## 启动

```supervisord -c /etc/supervisord.conf
```

-c 为指定配置文件路径

查看 supervisord 是否在运行：


```ps aux | grep supervisord
```

## 维护

supervisor 是一个 C/S 模型的程序，supervisord是server端，对应的client就是supervisorctl,supervisorctl不仅可以连接到本机上的supervisord，还可以连接到远程的supervisord，当然在本机上面是通过UNIX socket连接的，远程是通过TCP socket连接的。supervisorctl和supervisord之间的通信，是通过xml_rpc完成的。<br>
使用这个命令将进入supervisorctl的交互的命令行界面：

```supervisorctl -c /etc/supervisord.conf
```

命令有：


```
> status    # 查看程序状态
> stop usercenter   # 关闭 usercenter 程序
> start usercenter  # 启动 usercenter 程序
> restart usercenter    # 重启 usercenter 程序
> reread    ＃ 读取有更新（增加）的配置文件，不会启动新添加的程序
> update    ＃ 重启配置文件修改过的程序
```

当然直接通过命令也能达到相同的效果：


```
$ supervisorctl status
$ supervisorctl stop usercenter
$ supervisorctl start usercenter
$ supervisorctl restart usercenter
$ supervisorctl reread
$ supervisorctl update

```

除了使用supervisorctl还可以用，supervisor的web界面。
---
titile:
layout: post
title: "supervisor守护python进程"
subtitle: ""
date: 2016-03-30
author: chao
category: Jekyll
tags: python shadowosocks 
finished: true
---

shadowosocks的后台，之前我一直用screen，然而总是几天就会挂掉，当然也是各主机商的机器各有差别，Digitalocean新加坡上的隔一两天天就挂一次，vultr东京基本一个月挂一次。<br>
半年多了，每次挂掉，只有一次又一次的执行那条无比蛋疼的的命令：

```screen -S python server.py
```

最近Digitalocean上的程序经常挂掉，让我十分头疼。最终还是用了supervisor。<br>
其原理，大概相当于把你想守护的进程做成supervisor的子进程，然后supervisor管理。

## 安装

```sudo pip install supervisor
```

或者

```apt-get install  supervisor
```

## 配置

安装后默认没有配置文件，需要手动生成。

```echo_supervisord_conf > /etc/supervisord.conf
```

不过我在vps上基本都是生成失败了，不过也没有很大的关系，手动的创建/etc/supervisord.conf即可：

```[unix_http_server]
file=/tmp/supervisor.sock   ; UNIX socket 文件，supervisorctl 会使用
;chmod=0700                 ; socket 文件的 mode，默认是 0700
;chown=nobody:nogroup       ; socket 文件的 owner，格式： uid:gid

[inet_http_server]         ; HTTP 服务器，提供 web 管理界面
port=*:9001        ; Web 管理后台运行的 IP 和端口，如果开放到公网，需要注意安全性
username=user              ; 登录管理后台的用户名
password=123               ; 登录管理后台的密码

[supervisord]
logfile=/tmp/supervisord.log ; 日志文件，默认是 $CWD/supervisord.log
logfile_maxbytes=50MB        ; 日志文件大小，超出会 rotate，默认 50MB
logfile_backups=10           ; 日志文件保留备份数量默认 10
loglevel=info                ; 日志级别，默认 info，其它: debug,warn,trace
pidfile=/tmp/supervisord.pid ; pid 文件
nodaemon=false               ; 是否在前台启动，默认是 false，即以 daemon 的方式启动
minfds=1024                  ; 可以打开的文件描述符的最小值，默认 1024
minprocs=200                 ; 可以打开的进程数的最小值，默认 200

; the below section must remain in the config file for RPC
; (supervisorctl/web interface) to work, additional interfaces may be
; added by defining them in separate rpcinterface: sections
[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///tmp/supervisor.sock ; 通过 UNIX socket 连接 supervisord，路径与 unix_http_server 部分的 file 一致
;serverurl=http://127.0.0.1:9001 ; 通过 HTTP 的方式连接 supervisord

; 包含其他的配置文件
[include]
files = /etc/supervisor/*.conf   ; 可以是 *.conf 或 *.ini
```

这里的配置是将配置文件分成两部分，/etc/supervisord.conf为基本配置，有关守护的进程的配置都放到/etc/supervisor/下，该文件夹为自己手动创建。当然全写在一个文件里也是可以的。

再创建进程的配置文件/etc/supervisor/supervisor.conf


```[program:pyss]
directory = /root/manyuser/shadowosocks ; 程序的启动目录
command = python server.py ; 启动命令，可以看出与手动在命令行启动的命令是一样的
autostart = true     ; 在 supervisord 启动的时候也自动启动
startsecs = 5        ; 启动 5 秒后没有异常退出，就当作已经正常启动了
autorestart = true   ; 程序异常退出后自动重启
startretries = 3     ; 启动失败自动重试次数，默认是 3
user = leon          ; 用哪个用户启动
redirect_stderr = true  ; 把 stderr 重定向到 stdout，默认 false
stdout_logfile_maxbytes = 20MB  ; stdout 日志文件大小，默认 50MB
stdout_logfile_backups = 20     ; stdout 日志文件备份数
; stdout 日志文件，需要注意当指定目录不存在时无法正常启动，所以需要手动创建目录（supervisord 会自动创建日志文件）
stdout_logfile = /data/logs/usercenter_stdout.log

; 可以通过 environment 来添加需要的环境变量，一种常见的用法是修改 PYTHONPATH
; environment=PYTHONPATH=$PYTHONPATH:/path/to/somewhere
```

## 启动

```supervisord -c /etc/supervisord.conf
```

-c 为指定配置文件路径

查看 supervisord 是否在运行：


```ps aux | grep supervisord
```

## 维护

supervisor 是一个 C/S 模型的程序，supervisord是server端，对应的client就是supervisorctl,supervisorctl不仅可以连接到本机上的supervisord，还可以连接到远程的supervisord，当然在本机上面是通过UNIX socket连接的，远程是通过TCP socket连接的。supervisorctl和supervisord之间的通信，是通过xml_rpc完成的。<br>
使用这个命令将进入supervisorctl的交互的命令行界面：

```supervisorctl -c /etc/supervisord.conf
```

命令有：


```
> status    # 查看程序状态
> stop usercenter   # 关闭 usercenter 程序
> start usercenter  # 启动 usercenter 程序
> restart usercenter    # 重启 usercenter 程序
> reread    ＃ 读取有更新（增加）的配置文件，不会启动新添加的程序
> update    ＃ 重启配置文件修改过的程序
```

当然直接通过命令也能达到相同的效果：


```
$ supervisorctl status
$ supervisorctl stop usercenter
$ supervisorctl start usercenter
$ supervisorctl restart usercenter
$ supervisorctl reread
$ supervisorctl update

```

除了使用supervisorctl还可以用，supervisor的web界面。



{% highlight%}
root@vultr:/etc# supervisord -c /etc/supervisord.conf
Traceback (most recent call last):
  File "/usr/local/bin/supervisord", line 5, in <module>
    from pkg_resources import load_entry_point
  File "/usr/lib/python2.7/dist-packages/pkg_resources.py", line 2707, in <module>
    working_set.require(__requires__)
  File "/usr/lib/python2.7/dist-packages/pkg_resources.py", line 686, in require
    needed = self.resolve(parse_requirements(requirements))
  File "/usr/lib/python2.7/dist-packages/pkg_resources.py", line 584, in resolve
    raise DistributionNotFound(req)
pkg_resources.DistributionNotFound: meld3>=0.6.5
{% endhighlight %}

