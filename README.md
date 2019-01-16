shadowsocks
===========

[![PyPI version]][PyPI]
[![Build Status]][Travis CI]
[![Coverage Status]][Coverage]

A fast tunnel proxy that helps you bypass firewalls.

**CENTOS7上部署魔改版后端**

**1.更新yum源**<BR/>
`yum update`<BR/>
**2.添加epel源**<BR/>
`yum install -y epel-release`<BR/>
**3.安装git等**<BR/>
`yum install -y git`<BR/>
`yum install -y wget`<BR/>
`yum install python-setuptools && easy_install pip`<BR/>
**4.安装 libsodium _备注：如果不适用chacha20 方式可以省略此步骤**<BR/>

yum -y groupinstall "Development Tools"
wget https://github.com/jedisct1/libsodium/releases/download/1.0.13/libsodium-1.0.13.tar.gz
tar xf libsodium-1.0.13.tar.gz && cd libsodium-1.0.13
./configure && make -j2 && make install
echo /usr/local/lib > /etc/ld.so.conf.d/usr_local_lib.conf
ldconfig

**下载程序源代码并安装依赖**<BR/>

git clone  https://github.com/wggwcn/ssr_manyu.git
cd ssr_manyu<BR/>
yum -y install python-devel<BR/>
yum -y install libffi-devel<BR/>
yum -y install openssl-devel<BR/>
pip install -r requirements.txt<BR/>

**主要编辑 userapiconfig.py ,来解释下里面各项配置的意思**<BR/>

 >  # Config<BR/>
>   #节点ID<BR/>
  NODE_ID = 1<BR/>
   #自动化测速，为0不测试，此处以小时为单位，要和 ss-panel               > 设置的小时数一致<BR/>
  SPEEDTEST = 6<BR/>
  #云安全，自动上报与下载封禁IP，1为开启，0为关闭<BR/>
CLOUDSAFE = 1<BR/>
  #自动封禁SS密码和加密方式错误的 IP，1为开启，0为关闭<BR/>
  ANTISSATTACK = 0<BR/>
  #是否接受上级下发的命令，如果你要用这个命令，请参考我之前写的东西，公钥   放在目录下的 ssshell.asc<BR/>
AUTOEXEC = 1<BR/>
  多端口单用户设置，看重大更新说明。<BR/>
  MU_SUFFIX = 'zhaoj.in'<BR/>
  多端口单用户设置，看重大更新说明。<BR/>
  MU_REGEX = '%5m%id.%suffix'<BR/>
  #不明觉厉<BR/>
  SERVER_PUB_ADDR = '127.0.0.1' # mujson_mgr need this to generate ssr     link<BR/>
  #访问面板方式<BR/>
  `API_INTERFACE = 'glzjinmod' #glzjinmod (数据库方式连接)，modwebapi (http api)<BR/>
  #mudb，不要管<BR/>
MUDB_FILE = 'mudb.json'<BR/>
>   # HTTP API 的相关信息，看重大更新说明。<BR/>
  WEBAPI_URL = 'https://zhaoj.in'<BR/>
  WEBAPI_TOKEN = 'glzjin'<BR/>
>   # Mysql 数据库连接信息<BR/>
  MYSQL_HOST = '127.0.0.1'<BR/>
MYSQL_PORT = 3306<BR/>
MYSQL_USER = 'ss'<BR/>
MYSQL_PASS = 'ss'<BR/>
MYSQL_DB = 'shadowsocks'<BR/>
>   # 是否启用SSL连接，0为关，1为开<BR/>
MYSQL_SSL_ENABLE = 0<BR/>
>   # 客户端证书目录，请看   https://github.com/glzjin/shadowsocks/wiki/Mysql-SSL%E9%85%8D%E7%BD%AE<BR/>
 MYSQL_SSL_CERT = '/root/shadowsocks/client-cert.pem'<BR/>
MYSQL_SSL_KEY = '/root/shadowsocks/client-key.pem'<BR/>
MYSQL_SSL_CA = '/root/shadowsocks/ca.pem'<BR/>
>   # API，不用管<BR/>
API_HOST = '127.0.0.1' <BR/>
API_PORT = 80<BR/>
API_PATH = '/mu/v2/'<BR/>
API_TOKEN = 'abcdef'<BR/>
API_UPDATE_TIME = 60<BR/>
>   # Manager 不用管<BR/>
MANAGE_PASS = 'ss233333333'<BR/>
>   #if you want manage in other server you should set this value to global ip<BR/>
MANAGE_BIND_IP = '127.0.0.1'<BR/>
>   #make sure this port is idle<BR/>
MANAGE_PORT = 23333<BR/>
>   #安全设置，限制在线 IP 数所需，下面这个参数随机设置，并且所有节点需要保持一致。<BR/>
  IP_MD5_SALT = 'randomforsafety'<BR/>
  
运行的话，有几种方式。<BR/>

 - python server.py 用于调错的<BR/>
 - /run.sh 无日志后台运行<BR/>
 - /logrun.sh 有日志后台运行<BR/>
 - supervisord<BR/>

**优化**<BR/>

wget  https://raw.githubusercontent.com/wggwcn/ssr_manyu/master/centos7/limits.conf -O /etc/security/limits.conf && ulimit -n 51200<BR/>

wget  https://raw.githubusercontent.com/wggwcn/ssr_manyu/master/centos7/sysctl.conf -O /etc/sysctl.conf  && sysctl -p<BR/>

wget https://raw.githubusercontent.com/wggwcn/ssr_manyu/master/centos7/systemd-supervisor.service -O /usr/lib/systemd/system/systemd-supervisor.service<BR/>

**编辑supervisord配置文件**<BR/>
>编辑 /etc/supervisord.conf 最后一段改成如下的，以 /root/shadowsocks/ 为例<BR/>
[program:mu]<BR/>
command=python /root/shadowsocks/server.py<BR/>
directory=/root/shadowsocks<BR/>
autorestart=true<BR/>
startsecs=10<BR/>
startretries=36<BR/>
redirect_stderr=true<BR/>
user=root ; setuid to this UNIX account to run the program<BR/>
log_stdout=true ; if true, log program stdout (default true)<BR/>
log_stderr=true ; if true, log program stderr (def false)
logfile=/var/log/mu.log ; child log path, use NONE for none; default AUTO<BR/>
;logfile_maxbytes=1MB ; max # logfile bytes b4 rotation (default 50MB)<BR/>
;logfile_backups=10 ; # of logfile backups (default 10)<BR/>
保存<BR/>

如果想直接下载该文件supervisor的配置文件覆盖<BR/>
wget https://raw.githubusercontent.com/wggwcn/ssr_manyu/master/centos7/supervisord.conf -O -O /etc/supervisord.conf<BR/>


**supervisor 启动，停止，自开机命令**<BR/>

systemctl enable systemd-supervisor设置开机自启<BR/>
systemctl start systemd-supervisor启动supervisord<BR/>
systemctl status systemd-supervisor查看supervisord运行情况<BR/>




 
Client
------

* [Windows] / [OS X]
* [Android] / [iOS]
* [OpenWRT]

Use GUI clients on your local PC/phones. Check the README of your client
for more information.

Documentation
-------------

You can find all the documentation in the [Wiki].

License
-------

Copyright 2015 clowwindy

Licensed under the Apache License, Version 2.0 (the "License"); you may
not use this file except in compliance with the License. You may obtain
a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
License for the specific language governing permissions and limitations
under the License.

Bugs and Issues
----------------

* [Troubleshooting]
* [Issue Tracker]
* [Mailing list]



[Android]:           https://github.com/shadowsocks/shadowsocks-android
[Build Status]:      https://travis-ci.org/falseen/shadowsocks.svg?branch=manyuser-travis
[Configuration]:     https://github.com/shadowsocks/shadowsocks/wiki/Configuration-via-Config-File
[Coverage Status]:   https://jenkins.shadowvpn.org/result/shadowsocks
[Coverage]:          https://jenkins.shadowvpn.org/job/Shadowsocks/ws/PYENV/py34/label/linux/htmlcov/index.html
[Debian sid]:        https://packages.debian.org/unstable/python/shadowsocks
[iOS]:               https://github.com/shadowsocks/shadowsocks-iOS/wiki/Help
[Issue Tracker]:     https://github.com/shadowsocks/shadowsocks/issues?state=open
[Install Server on Windows]: https://github.com/shadowsocks/shadowsocks/wiki/Install-Shadowsocks-Server-on-Windows
[Mailing list]:      https://groups.google.com/group/shadowsocks
[OpenWRT]:           https://github.com/shadowsocks/openwrt-shadowsocks
[OS X]:              https://github.com/shadowsocks/shadowsocks-iOS/wiki/Shadowsocks-for-OSX-Help
[PyPI]:              https://pypi.python.org/pypi/shadowsocks
[PyPI version]:      https://img.shields.io/pypi/v/shadowsocks.svg?style=flat
[Travis CI]:         https://travis-ci.org/falseen/shadowsocks
[Troubleshooting]:   https://github.com/shadowsocks/shadowsocks/wiki/Troubleshooting
[Wiki]:              https://github.com/shadowsocks/shadowsocks/wiki
[Windows]:           https://github.com/shadowsocks/shadowsocks-csharp
