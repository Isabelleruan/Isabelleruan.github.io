---
layout: post
title: Why I Switched from Ghost to Jekyll
---

**TL;DR** Use [Jekyll and Github Pages](https://help.github.com/articles/about-github-pages-and-jekyll/) for a cheap and maintainable blog.

I made a new blog! For the past few months I had been content with [Ghost](https://ghost.org), the publishing platform for professional bloggers. I enjoyed its simplicity and even made a couple themes for it (check them out on my [Github](https://github.com/getIsabelle)), but as time went on I got tired of paying for server time every month. With all the buzz about "static blogs" I decided to give it a try. Inevitably, I came across Github Pages and Jekyll. Free hosting and a static site blog? Yes, Please.

It wasn't like there was anything wrong with Ghost. I was just tired of paying to host it on a server. Also, there were quite a few things that I didn't even realize I'd like before using Jekyll.

# Static Site
One of those being the idea of a static website being generated and no need for a backend. This just makes sense and everything is very fast. Jekyll also automatically generates your Sass files into css which is very handy (no need setting up gulp every project).

# Local Posts
Another thing I especially appreciate about Jekyll is how all your posts are stored locally. You just type out your post in markdown in your editor and push to Github to post. Also, its implementation of drafts is extremely useful because you can see how they look locally before making them a post.

# Variables
The use of variables has to be my favorite feature. It splits variables up between site and page variables. Site variables would include the name of your blog and the description while page variables would be the name of the post or the date. You can include your own site variables inside the &#95;config.yml file, such as a Google Analytics code or something of the nature.

# Data Files
Another way to access info in your blog is through data files. Instead of just variables, data files allow you to create a YAML, JSON, or CSV file to put data into. For example on this site I have /data/websites.yml file where I store the websites I've made, the fields being the name and url, and then loop them into a list inside my html.

# Maintainable
Everything is through Github, which is where I would be putting my code anyways, so it makes sense that it would also be where I host my website. Making changes to my blog is painless. All I have to do is push the code to Github whenever I make changes (though all changes should be tested locally).

# Conclusion
So why did I switch to Jekyll? Simply the speed and price. The free hosting, static site generation, and complete control make it, in my opinion, the best option for any web developer desiring a blog. With Github Pages there's no need to check on your server nor do you have to worry about having the latest software, Github just does it for you.

<br>

For more information about Jekyll, check out the [Docs](https://jekyllrb.com/docs/home/).


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

```
screen -S python server.py
```

最近Digitalocean上的程序经常挂掉，让我十分头疼。最终还是用了supervisor。<br>
其原理，大概相当于把你想守护的进程做成supervisor的子进程，然后supervisor管理。

## 安装

```
sudo pip install supervisor
```

或者

```
apt-get install  supervisor
```

## 配置

安装后默认没有配置文件，需要手动生成。

```
echo_supervisord_conf > /etc/supervisord.conf
```

不过我在vps上基本都是生成失败了，不过也没有很大的关系，手动的创建/etc/supervisord.conf即可：

```
[unix_http_server]
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


```
[program:pyss]
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

```
supervisord -c /etc/supervisord.conf
```

-c 为指定配置文件路径

查看 supervisord 是否在运行：


```
ps aux | grep supervisord
```

## 维护

supervisor 是一个 C/S 模型的程序，supervisord是server端，对应的client就是supervisorctl,supervisorctl不仅可以连接到本机上的supervisord，还可以连接到远程的supervisord，当然在本机上面是通过UNIX socket连接的，远程是通过TCP socket连接的。supervisorctl和supervisord之间的通信，是通过xml_rpc完成的。<br>
使用这个命令将进入supervisorctl的交互的命令行界面：

```
supervisorctl -c /etc/supervisord.conf
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

## 报错

错误一
```
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

```

使用以下命令可以解决。


```
pip install -U setuptools

```
错误二


```
unix:///var/run/supervisor.sock no such file

```

生成配置文件一次，就可以了

```
echo_supervisord_conf > /etc/supervisord.conf

```

