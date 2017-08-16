---
title: ansible系列总结
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---

# 1. 简介与基本安装

## 常用运维工具
- CFengine
- Chef
- Puppet
基于Ruby开发，采用C/S架构，扩展性强，基于SSL认证
- SaltStack
基于Python开发，采用C/S架构，相对于puppet更轻量级，配置语法采用YMAL，使得配置脚本更为简单
- Ansible
基于Python开发，分布式，无需客户端，轻量级，配置语言采用YAML。

现在基本都在用Puppet、SaltStack、Ansible，最主要的是Puppet、SaltStack。
Puppet是基于C/S架构，执行远程命令，一直是它的挡板，基于SSL可以保证C/S安全。
SaltStack相对Puppet来说属于新规，也采用C/S，相比puppet更轻，采用YAML语法。
Ansible更新，也更轻，ansible本身只是个工具，它使用ssh互信直接管理客户端。
当然市面用的最多的是Puppet和SaltStack。

## 为什么选择Ansible
- 相对于puppet和saltstack，ansible无需客户端，更轻量级；
- ansible甚至都不用启动服务，仅仅只是一个工具，可以很轻松的实现分布式扩展；
- 更强的远程命令执行操作，相对于Puppet来说的；
- 不输于puppet和saltstack的其他功能；
- 默认使用ssh控制各节点。

3. Ansible基本架构
![enter description here][1]

- 连接插件（Connectior Plugins）：ansible基于连接插件连接到各个主机上，虽然ansible是使用ssh连接到各个主机的，但是它还支持其他的连接方法，所以需要有连接插件；比如使用SaltStack的ZeroMQ协议；
- 核心模块（Core Modules）：Ansible是模块化的，比如你连接上具体主机后，你要做操作，比如要在远程主机创建文件，你需要file模块；你要copy一个文件到远程主机，你需要copy模块；你要检查远程主机是否存活，你需要ping模块；Ansible需要具体的模块去做具体事情；
- 扩展模块（Custom Modules）：当Ansible自带模块完成不了时候，我们可以对模块进行扩展，自定义模块；
- 插件（Plugins）：完成模块功能的补充；
- 剧本（Playbooks）：ansible的任务配置文件，将多个任务定义在剧本中，由ansible自动执行
- 主机群（Host Inventory）：定义ansible管理的主机，我们要管理哪些主机呢，就需要这样的文件来告诉Ansible。

## Ansible工作原理
![enter description here][2]
local、SSH、ZeroMq是连接插件，我们经常使用的ssh互信方式；
具体执行什么，依赖于modules的；
然后将使用到的命令都定义到playbooks中；
然后通过hosts主机清单定义好的主机来应用playbooks。

Ansible 在管理节点将 Ansible 模块通过 SSH 协议（或者 Kerberos、LDAP）推送到被管理端执行，执行完之后自动删除，可以使用 SVN 等来管理自定义模块及编排。


## Ansible安装
1. centos安装

第一步： 设置EPEL仓库
Ansible是属于Extra Packages for Enterprise Linux (EPEL)库的一部分，默认不在yum仓库中，因此要先安装EPEL。
```shell?linenums
[root@arthur ~]# yum install -y epel-release
[root@arthur ~]# yum -y repolist
```
第二步： 使用yum安装Ansible
```shell?linenums
[root@ansible ~]# yum install -y ansible
```
安装完成后，检查ansible版本：
```shell?linenums
[root@localhost ~]# clear
[root@localhost ~]# ansible --version
ansible 2.3.1.0
  config file = /etc/ansible/ansible.cfg
  configured module search path = Default w/o overrides
  python version = 2.7.5 (default, Nov  6 2016, 00:28:07) [GCC 4.8.5 20150623 (Red Hat 4.8.5-11)]
[root@localhost ~]#
```
2. Ubuntu安装

获取安装包的服务器进行替换，避免翻墙
```shell?linenums
sudo sed  -i  -re  's/\w+\.archive\.ubuntu\.com/archive.ubuntu.com/g'  /etc/apt/sources.list
```
更新安装库
```shell?linenums
sudo apt-get update
```
然后输入最后的四行命令进行安装的操作
```shell?linenums
sudo apt-get install -y software-properties-common
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update
sudo apt-get install -y ansible
```
测试ansible安装
```shell?linenums
root@b556839ea9cf:/# ansible --version
ansible 2.3.2.0
  config file = /etc/ansible/ansible.cfg
  configured module search path = Default w/o overrides
  python version = 2.7.6 (default, Oct 26 2016, 20:30:19) [GCC 4.8.4]
root@b556839ea9cf:/#
```

我们看看ansible安装时都装了哪些东西
```shell?linenums
[root@localhost ~]# cat 1.txt
/etc/ansible                        # ansible配置文件的目录
/etc/ansible/ansible.cfg            # ansible主配置文件
/etc/ansible/hosts                  # 默认的ansible管理的主机清单
/etc/ansible/roles    
/usr/bin/ansible                    # 下面全是ansible提供的命令
/usr/bin/ansible-2
/usr/bin/ansible-2.7
/usr/bin/ansible-connection
/usr/bin/ansible-console
/usr/bin/ansible-console-2
/usr/bin/ansible-console-2.7
/usr/bin/ansible-doc
/usr/bin/ansible-doc-2
/usr/bin/ansible-doc-2.7
/usr/bin/ansible-galaxy
/usr/bin/ansible-galaxy-2
/usr/bin/ansible-galaxy-2.7
/usr/bin/ansible-playbook
/usr/bin/ansible-playbook-2
/usr/bin/ansible-playbook-2.7
/usr/bin/ansible-pull
/usr/bin/ansible-pull-2
/usr/bin/ansible-pull-2.7
/usr/bin/ansible-vault
/usr/bin/ansible-vault-2
/usr/bin/ansible-vault-2.7
/usr/lib/python2.7/site-packages/ansible
/usr/lib/python2.7/site-packages/ansible-2.3.1.0-py2.7.egg-info
/usr/lib/python2.7/site-packages/ansible/module_utils/basic.pyo
/usr/lib/python2.7/site-packages/ansible/module_utils/bigswitch_utils.py
/usr/lib/python2.7/site-packages/ansible/modules/windows/win_domain.py
/usr/lib/python2.7/site-packages/ansible/modules/windows/win_domain.pyc
/usr/lib/python2.7/site-packages/ansible/modules/windows/win_domain.pyo
/usr/lib/python2.7/site-packages/ansible/release.pyo
...
/usr/lib/python2.7/site-packages/ansible/template
/usr/share/doc/ansible-2.3.1.0/COPYING
/usr/share/doc/ansible-2.3.1.0/PKG-INFO
/usr/share/doc/ansible-2.3.1.0/README.md
/usr/share/man/man1/ansible-console.1.gz
/usr/share/man/man1/ansible-doc.1.gz
/usr/share/man/man1/ansible-galaxy.1.gz
/usr/share/man/man1/ansible-playbook.1.gz
/usr/share/man/man1/ansible-pull.1.gz
/usr/share/man/man1/ansible-vault.1.gz
/usr/share/man/man1/ansible.1.gz
[root@localhost ~]#
```

首先我们先看ansible命令：
```shell?linenums
[root@localhost ~]# ansible -h
Usage: ansible <host-pattern> [options]
Options:
  -a MODULE_ARGS, --args=MODULE_ARGS
                        module arguments	指定执行模块的具体参数
  --ask-vault-pass      ask for vault password
  -B SECONDS, --background=SECONDS
                        run asynchronously, failing after X seconds
                        (default=N/A)
  -C, --check           don't make any changes; instead, try to predict some
                        of the changes that may occur
  -D, --diff            when changing (small) files and templates, show the
                        differences in those files; works great with --check
  -e EXTRA_VARS, --extra-vars=EXTRA_VARS
                        set additional variables as key=value or YAML/JSON
  -f FORKS, --forks=FORKS
                        specify number of parallel processes to use
                        (default=5)
  -h, --help            show this help message and exit
  -i INVENTORY, --inventory-file=INVENTORY
                        specify inventory host path
                        (default=/etc/ansible/hosts) or comma separated host
                        list.	指定要管理的所有远程主机
  -l SUBSET, --limit=SUBSET
                        further limit selected hosts to an additional pattern
  --list-hosts          outputs a list of matching hosts; does not execute
                        anything else
  -m MODULE_NAME, --module-name=MODULE_NAME
                        module name to execute (default=command)	指定具体操作用到的哪些模块，你要文件操作，必须file模块，你要执行命令，必须command模块。
  -M MODULE_PATH, --module-path=MODULE_PATH
                        specify path(s) to module library (default=None)
  --new-vault-password-file=NEW_VAULT_PASSWORD_FILE
                        new vault password file for rekey
  -o, --one-line        condense output
  --output=OUTPUT_FILE  output file name for encrypt or decrypt; use - for
                        stdout
  -P POLL_INTERVAL, --poll=POLL_INTERVAL
                        set the poll interval if using -B (default=15)
  --syntax-check        perform a syntax check on the playbook, but do not
                        execute it
  -t TREE, --tree=TREE  log output to this directory
  --vault-password-file=VAULT_PASSWORD_FILE
                        vault password file
  -v, --verbose         verbose mode (-vvv for more, -vvvv to enable
                        connection debugging)
  --version             show program's version number and exit
  Connection Options:
    control as whom and how to connect to hosts
    -k, --ask-pass      ask for connection password
    --private-key=PRIVATE_KEY_FILE, --key-file=PRIVATE_KEY_FILE
                        use this file to authenticate the connection
    -u REMOTE_USER, --user=REMOTE_USER
                        connect as this user (default=None)
    -c CONNECTION, --connection=CONNECTION
                        connection type to use (default=smart)
    -T TIMEOUT, --timeout=TIMEOUT
                        override the connection timeout in seconds
                        (default=10)
    --ssh-common-args=SSH_COMMON_ARGS
                        specify common arguments to pass to sftp/scp/ssh (e.g.
                        ProxyCommand)
    --sftp-extra-args=SFTP_EXTRA_ARGS
                        specify extra arguments to pass to sftp only (e.g. -f,
                        -l)
    --scp-extra-args=SCP_EXTRA_ARGS
                        specify extra arguments to pass to scp only (e.g. -l)
    --ssh-extra-args=SSH_EXTRA_ARGS
                        specify extra arguments to pass to ssh only (e.g. -R)
  Privilege Escalation Options:
    control how and which user you become as on target hosts
    -s, --sudo          run operations with sudo (nopasswd) (deprecated, use
                        become)
    -U SUDO_USER, --sudo-user=SUDO_USER
                        desired sudo user (default=root) (deprecated, use
                        become)	指定远程主机用户，默认为root
    -S, --su            run operations with su (deprecated, use become)
    -R SU_USER, --su-user=SU_USER
                        run operations with su as this user (default=root)
                        (deprecated, use become)
    -b, --become        run operations with become (does not imply password
                        prompting)
    --become-method=BECOME_METHOD
                        privilege escalation method to use (default=sudo),
                        valid choices: [ sudo | su | pbrun | pfexec | doas |
                        dzdo | ksu | runas ]
    --become-user=BECOME_USER
                        run operations as this user (default=root)
    --ask-sudo-pass     ask for sudo password (deprecated, use become)
    --ask-su-pass       ask for su password (deprecated, use become)
    -K, --ask-become-pass
                        ask for privilege escalation password
[root@localhost ~]#
```
我们现在开始做个演示。
首先修改INVENTORY：
```shell?linenums
[root@localhost ~]# vim /etc/ansible/hosts
[root@localhost ~]#
#   - Blank lines are ignored
#   - Groups of hosts are delimited by [header] elements
#   - You can enter hostnames or ip addresses
#   - A hostname/ip can be a member of multiple groups
# Ex 1: Ungrouped hosts, specify before any group headers.
## green.example.com
## blue.example.com
## 192.168.100.1
## 192.168.100.10
# Ex 2: A collection of hosts belonging to the 'webservers' group
## [webservers]
## alpha.example.org
## beta.example.org
## 192.168.1.100
## 192.168.1.110
# If you have multiple hosts following a pattern you can specify
# them like this:
## www[001:006].example.com
# Ex 3: A collection of database servers in the 'dbservers' group
## [dbservers]
##
## db01.intranet.mydomain.net
## db02.intranet.mydomain.net
## 10.25.1.56
## 10.25.1.57
# Here's another example of host ranges, this time there are no
# leading 0s:
## db-[99:101]-node.example.com
```

我的演示环境：
ansible管理节点：172.17.0.2
machine-001节点：172.17.0.3
machine-002节点：172.17.0.4
machine-003节点：172.17.0.5
我的 /etc/ansible/hosts 文件配置：
```shell?linenums
[test]
172.17.0.3
```
现在在ansible管理节点上执行一下
```shell?linenums
root@b556839ea9cf:/# ansible -i /etc/ansible/hosts test -u root -m command -a 'ls /usr' -k
SSH password:
172.17.0.3 | SUCCESS | rc=0 >>
bin
games
include
lib
local
sbin
share
src
root@b556839ea9cf:/#
```
我们这里，-i指定hosts清单文件，test为清单文件中分组；-u指定远程用户名；-m指定要用到哪个模块，-a指定调用模块传递的参数，即执行的命令；-k指定远程密码。
返回结果，SUCCESS代表成功。
![enter description here][3]
-i hosts清单文件,但哪个分组不能省掉，-u为root，-m为command都是默认的，可以省略掉。
即可以写成：
```shell?linenums
root@b556839ea9cf:/# ansible test -a 'ls /usr' -k
SSH password:
172.17.0.3 | SUCCESS | rc=0 >>
bin
games
include
lib
local
sbin
share
src
root@b556839ea9cf:/#
```
![enter description here][4]

效果是一样的。

上面就是ansible的简单使用。

# 2. 主机清单INVENTORY

Ansible 通过读取默认的主机清单配置`/etc/ansible/hosts`，可以同时连接到多个远程主机上执行任务,，默认路径可以通过修改`ansible.cfg`的`hostfile`参数指定路径。

## Hosts and Groups（主机与组）

对于/etc/ansible/hosts最简单的定义格式像下面：
### 1 简单的主机和组
```ini?linenums
mail.yanruogu.com
[webservers]
web1.yanruogu.com
web2.yanruogu.com 
[dbservers]
db1.yanruogu.com
db2.yanruogu.com
```
a、中括号中的名字代表组名，可以根据自己的需求将庞大的主机分成具有标识的组，如上面分了两个组webservers和dbservers组；
b、主机(hosts)部分可以使用域名、主机名、IP地址表示；当然使用前两者时，也需要主机能反解析到相应的IP地址，一般此类配置中多使用IP地址；
### 2 端口与别名
如果某些主机的SSH运行在自定义的端口上，ansible使用Paramiko进行ssh连接时，不会使用你SSH配置文件中列出的端口，但是如果修改ansible使用openssh进行ssh连接时将会使用：
```ini?linenums
192.168.1.1:3091
```
假如你想要为某些静态IP设置一些别名，可以这样做：
```ini?linenums
web1 ansible_ssh_port = 3333 ansible_ssh_host = 192.168.1.2
```
上面的 web1别名就指代了IP为192.168.1.2，ssh连接端口为3333的主机。

### 3 指定主机范围
```ini?linenums
[webservers]
www[01:50].yanruogu.com
[databases]
db-[a:f].yanruogu.com
```
上面指定了从web1到web50，webservers组共计50台主机；databases组有db-a到db-f共6台主机。

### 4 使用主机变量
以下是Hosts部分中经常用到的变量部分：
```shell?linenums
ansible_ssh_host     #用于指定被管理的主机的真实IP
ansible_ssh_port     #用于指定连接到被管理主机的ssh端口号，默认是22
ansible_ssh_user     #ssh连接时默认使用的用户名
ansible_ssh_pass     #ssh连接时的密码
ansible_sudo_pass     #使用sudo连接用户时的密码
ansible_sudo_exec     #如果sudo命令不在默认路径，需要指定sudo命令路径
ansible_ssh_private_key_file     #秘钥文件路径，秘钥文件如果不想使用ssh-agent管理时可以使用此选项
ansible_shell_type     #目标系统的shell的类型，默认sh
ansible_connection     #SSH 连接的类型： local , ssh , paramiko，在 ansible 1.2 之前默认是 paramiko ，后来智能选择，优先使用基于 ControlPersist 的 ssh （支持的前提）
ansible_python_interpreter     #用来指定python解释器的路径，默认为/usr/bin/python 同样可以指定ruby 、perl 的路径
ansible_*_interpreter     #其他解释器路径，用法与ansible_python_interpreter类似，这里"*"可以是ruby或才perl等其他语言
```

示例如下：
```ini?linenums
[test]
192.168.1.1 ansible_ssh_user=root ansible_ssh_pass='P@ssw0rd'
192.168.1.2 ansible_ssh_user=breeze ansible_ssh_pass='123456'
192.168.1.3 ansible_ssh_user=bernie ansible_ssh_port=3055 ansible_ssh_pass='456789'
```

上面的示例中指定了三台主机，三台主机的用密码分别是P@ssw0rd、123456、45789，指定的ssh连接的用户名分别为root、breeze、bernie，ssh 端口分别为22、22、3055 ，这样在ansible命令执行的时候就不用再指令用户和密码等了。

### 5 组内变量
变量也可以通过组名，应用到组内的所有成员：
```ini?linenums
[test]
host1
host2
[test:vars]
ntp_server=192.168.1.10
proxy=192.168.1.20
```
上面test组中包含两台主机，通过对test组指定==vars==变更，相应的host1和host2相当于相应的指定了ntp_server和proxy变量参数值 。

### 6 组的包含与组内变量
```ini?linenums
[wuhan]
web1
web2
[suizhou]
web4
web3
[hubei:children]
wuhan
suizhou
[hubei:vars]
ntp_server=192.168.1.10
zabbix_server=192.168.1.10
[china:children]
hubei
hunan
```
上面的示例中，指定了武汉组有web1、web2；随州组有web3、web4主机；又指定了一个湖北组，同时包含武汉和随州；同时为该组内的所有主机指定了2个vars变量。设定了一个组中国组，包含湖北、湖南。

注：vars变量在ansible ad-hoc部分中基本用不到，主要用在ansible-playbook中。

## Patterns（主机与组正则匹配部分）

把Patterns 直接理解为正则实际是不完全准确的，可理解成要与哪些主机进行通信。

在探讨这个问题之前我们先看下ansible的用法：
```shell?linenums
ansible <pattern_goes_here> -m <module_name> -a <arguments>
```

直接上一个示例：
```shell?linenums
ansible webservers -m service -a "name=httpd state=restarted"
```
这里是对webservers 组或主机重启httpd服务 ，其中`webservers`就是Pattern部分。
之所以上面说Pattern（模式）可以理解为正则，主要针对下面经常用到的用法而言的：
### 1 表示所有的主机可以使用`all`或`*`； 
### 2 逻辑或 和 任意
利用通配符还可以指定一组具有规则特征的主机或主机名。
`:`表示逻辑或；
`*`代表任意。

```ini?linenums
web0.yanruogu.com
web1.yanruogu.com:web2.yanruogu.com	#表示两个组中所有机器
192.168.1.1
192.168.1.*     # 表示192.168.1.x所有机器
```

当然，这里的`*`通配符也可以用在前面，如：
```ini?linenums
*.yanruogu.com
*.com    
webservers1[0]     #表示匹配 webservers1 组的第 1 个主机    webservers1[0:25]  #表示匹配 webservers1 组的第 1 个到第 25 个主机
```

### 3 逻辑非 和 逻辑与
非的表达式。如，目标主机必须在组webservers但不在phoenix组中
```ini?linenums
webserver:!phoenix
```
交集的表达式。如，目标主机必须即在组webservers中又在组staging中
```ini?linenums
webservers:&staging
```
一个更复杂的示例：
```ini?linenums
webserver:dbservers:&staging:!phoenix
```
上面这个复杂的表达式最后表示的目标主机必须满足：在webservers或者dbservers组中，必须还存在于staging组中，但是不在phoenix组中。

### 4 混合使用
```ini?linenums
*.yanruogu.com:*.org
```
还可以在开头的地方使用`”~”`，用来表示这是一个正则表达式:
```ini?linenums
~(web|db).*\.yanruogu\.com
```

ansible-playbook中经常使用Pattern的用法：
- 在ansible-playbook命令中，你可以使用变量来组成这样的表达式，但是你必须使用“-e”的选项来指定这个表达式：
```ini?linenums
ansible-playbook -e webservers:!{{excluded}}:&{{required}}
```
- 在ansible和ansible-playbook中，还可以通过一个参数”--limit”来明确指定排除某些主机或组：
```ini?linenums
ansible-playbook site.yml --limit datacenter2
```
- 从Ansible1.2开始，如果想排除一个文件中的主机可以使用"@"：
```ini?linenums
ansible-playbook site.yml --limit @retry_hosts.txt
```
# 3. ansible.cfg配置说明

Ansible默认安装好后有一个配置文件/etc/ansible/ansible.cfg，该配置文件中定义了ansible的主机的默认配置部分，如默认是否需要输入密码、是否开启sudo认证、action_plugins插件的位置、hosts主机组的位置、是否开启log功能、默认端口、key文件位置等等。
具体如下：
```ini?linenums
[defaults]
# some basic default values...
hostfile       = /etc/ansible/hosts   \\指定默认hosts配置的位置
# library_path = /usr/share/my_modules/
remote_tmp     = $HOME/.ansible/tmp
pattern        = *
forks          = 5
poll_interval  = 15
sudo_user      = root  \\远程sudo用户
#ask_sudo_pass = True  \\每次执行ansible命令是否询问ssh密码
#ask_pass      = True  \\每次执行ansible命令时是否询问sudo密码
transport      = smart
remote_port    = 22
module_lang    = C
gathering = implicit
host_key_checking = False    \\关闭第一次使用ansible连接客户端是输入命令提示
log_path    = /var/log/ansible.log \\需要时可以自行添加。chown -R root:root ansible.log
system_warnings = False    \\关闭运行ansible时系统的提示信息，一般为提示升级
# set plugin path directories here, separate with colons
action_plugins     = /usr/share/ansible_plugins/action_plugins
callback_plugins   = /usr/share/ansible_plugins/callback_plugins
connection_plugins = /usr/share/ansible_plugins/connection_plugins
lookup_plugins     = /usr/share/ansible_plugins/lookup_plugins
vars_plugins       = /usr/share/ansible_plugins/vars_plugins
filter_plugins     = /usr/share/ansible_plugins/filter_plugins
fact_caching = memory
[accelerate]
accelerate_port = 5099
accelerate_timeout = 30
accelerate_connect_timeout = 5.0
# The daemon timeout is measured in minutes. This time is measured
# from the last activity to the accelerate daemon.
accelerate_daemon_timeout = 30
```
经常用到的修改配置一：
如果在对之前未连接的主机进行连结时报错如下：
```shell?linenums
ansible test -a 'uptime'
192.168.1.1| FAILED =>Using a SSH password instead of a key is not possible because HostKey checking is enabled and sshpass does not support this.Please add this host's fingerprint to your known_hosts file to manage this host.
192.168.1.2 | FAILED => Using a SSH password instead of a key is not possible because Host Key checking is enabled and sshpass does not support this.  Please add this host's fingerprint to your known_hosts file to manage this host.
```
是由于在本机的~/.ssh/known_hosts文件中没有fingerprint key串，ssh第一次连接的时候一般会提示输入yes 进行确认，将key字符串加入到  ~/.ssh/known_hosts 文件中。
该错误解决办法有：
方法一：
在进行ssh连接时，可以使用`-o`参数将`StrictHostKeyChecking`设置为no，使用ssh连接时避免首次连接时让输入yes/no部分的提示。通过查看ansible.cfg配置文件，发现如下行：
```ini?linenums
[ssh_connection]
# ssh arguments to use
# Leaving off ControlPersist will result in poor performance, so use
# paramiko on older platforms rather than removing it
#ssh_args = -o ControlMaster=auto -o ControlPersist=60s
```
可以启用ssh_args 部分，使用下面的配置，避免上面出现的错误：
```ini?linenums
ssh_args = -o ControlMaster=auto -o ControlPersist=60s -o StrictHostKeyChecking＝no
```

方法二：
在ansible.cfg配置文件中，也会找到如下配置：
```ini?linenums
# uncomment this to disable SSH key host checking
host_key_checking = False  
```
默认host_key_checking部分是注释的，通过该行，同样也可以实现跳过ssh 首次连接提示验证部分。但在实际测试中，似乎并没有效果，建议使用方法1。

经常用到的修改配置二：
默认ansible 执行的时候，并不会输出日志到文件，不过在ansible.cfg 配置文件中有如下行：
```ini?linenums
log_path = /var/log/ansible.log
```
默认log_path这行是注释的，打开该行的注释，所有的命令执行后，都会将日志输出到/var/log/ansible.log文件。

# 4. Ad-hoc与命令执行模块

`Ad-Hoc`是指ansible下临时执行的一条命令，并且不需要保存的命令，对于复杂的命令会使用playbook。Ad-hoc的执行依赖于模块，ansible官方提供了大量的模块，如：command、raw、shell、file、cron等。
查看所有模块可以通过`ansible-doc -l`；
查看某个模块的参数可以使用`ansible-doc -s module`；
查看该模块更详细的信息可以使用`ansible-doc help module`。

## Ad-hoc使用办法
### 1 命令说明
一个ad-hoc命令的执行，需要按以下格式进行执行：
```shell?linenums
ansible 主机或组 -m 模块名 -a '模块参数'  ansible参数
```
- 主机或组，是在/etc/ansible/hosts 里进行指定的部分，当然动态Inventory 使用的是脚本从外部应用里获取的主机；
- 模块名，可以通过`ansible-doc -l`查看目前安装的模块，默认不指定时，使用的是`command`模块，具体可以查看`/etc/ansible/ansible.cfg`的`“#module_name = command ”`部分，默认模块可以在该配置文件中进行修改；
- 模块参数，可以通过`“ansible-doc -s 模块名”`查看具体的用法及后面的参数；
- ansible参数，可以通过ansible命令的帮助信息里查看到，这里有很多参数可以供选择，如是否需要输入密码、是否sudo等。

### 2 后台执行
当命令执行时间比较长时，也可以放到后台执行，使用`-B`、`-P`参数，如下：
```shell?linenums
ansible all -B 3600 -a "/usr/bin/long_running_operation --do-stuff" #后台执行命令3600s，-B 表示后台执行的时间
ansible all -m async_status -a "jid=123456789"  #检查任务的状态
ansible all -B 1800 -P 60 -a "/usr/bin/long_running_operation --do-stuff" #后台执行命令最大时间是1800s即30分钟，-P 每60s检查下状态，默认15s
```

## 命令执行模块
命令执行模块包含如下 四个模块：
- command模块：该模块通过-a跟上要执行的命令可以直接执行，不过命令里如果有带有如下字符部分则执行不成功 “  "<", ">", "|",  "&" ；
- shell 模块：用法基本和command一样，不过其是通过/bin/sh进行执行，所以shell 模块可以执行任何命令，就像在本机执行一样；
- raw模块：用法和shell 模块一样 ，其也可以执行任意命令，就像在本机执行一样；
- script模块：其是将管理端的shell在被管理主机上执行，其原理是先将shell复制到远程主机，再在远程主机上执行，原理类似于raw模块。

# 5. 常用模块

根据ansible官方的分类，将模块按功能分类为：云模块、命令模块、数据库模块、文件模块、资产模块、消息模块、监控模块、网络模块、通知模块、包管理模块、源码控制模块、系统模块、单元模块、web设施模块、windows模块 ，具体可以参看[官方页面][5]。
这里从官方分类的模块里选择最常用的一些模块进行介绍。

## ping模块
测试主机是否是通的，用法很简单，不涉及参数：
```shell?linenums
ansible test -m ping
```
## setup模块
setup模块，主要用于获取主机信息，在playbooks里经常会用到的一个参数gather_facts就与该模块相关。
setup模块下经常使用的一个参数是filter参数，具体使用示例如下：
```shell?linenums
root@b556839ea9cf:/# ansible 172.17.0.3 -m setup
172.17.0.3 | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "172.17.0.3"
        ],
        "ansible_all_ipv6_addresses": [],
        "ansible_apparmor": {
            "status": "disabled"
        },
        "ansible_architecture": "x86_64",
        "ansible_bios_date": "12/01/2006",
        "ansible_bios_version": "VirtualBox",
        "ansible_cmdline": {
            "BOOT_IMAGE": "/vmlinuz-3.10.0-514.10.2.el7.x86_64",
            "biosdevname": "0",
            "console": "ttyS0,115200",
            "crashkernel": "auto",
            "net.ifnames": "0",
            "no_timer_check": true,
            "quiet": true,
            "rd.lvm.lv": "VolGroup00/LogVol01",
            "rhgb": true,
            "ro": true,
            "root": "/dev/mapper/VolGroup00-LogVol00"
        },
        "ansible_date_time": {
            "date": "2017-08-14",
            "day": "14",
            "epoch": "1502752176",
            "hour": "23",
            "iso8601": "2017-08-14T23:09:36Z",
            "iso8601_basic": "20170814T230936076979",
            "iso8601_basic_short": "20170814T230936",
            "iso8601_micro": "2017-08-14T23:09:36.077056Z",
            "minute": "09",
            "month": "08",
            "second": "36",
            "time": "23:09:36",
            "tz": "UTC",
            "tz_offset": "+0000",
            "weekday": "Monday",
            "weekday_number": "1",
            "weeknumber": "33",
            "year": "2017"
        },
        "ansible_default_ipv4": {
            "address": "172.17.0.3",
            "alias": "eth0",
            "broadcast": "global",
            "gateway": "172.17.0.1",
            "interface": "eth0",
            "macaddress": "02:42:ac:11:00:03",
            "mtu": 1500,
            "netmask": "255.255.0.0",
            "network": "172.17.0.0",
            "type": "ether"
        },
        "ansible_default_ipv6": {},
        "ansible_devices": {
            "sda": {
                "holders": [],
                "host": "",
                "model": "VBOX HARDDISK",
                "partitions": {
                    "sda1": {
                        "holders": [],
                        "sectors": "2048",
                        "sectorsize": 512,
                        "size": "1.00 MB",
                        "start": "2048",
                        "uuid": null
                    },
                    "sda2": {
                        "holders": [],
                        "sectors": "2097152",
                        "sectorsize": 512,
                        "size": "1.00 GB",
                        "start": "4096",
                        "uuid": null
                    },
                    "sda3": {
                        "holders": [
                            "VolGroup00-LogVol00",
                            "VolGroup00-LogVol01"
                        ],
                        "sectors": "81784832",
                        "sectorsize": 512,
                        "size": "39.00 GB",
                        "start": "2101248",
                        "uuid": null
                    }
                },
                "removable": "0",
                "rotational": "1",
                "sas_address": null,
                "sas_device_handle": null,
                "scheduler_mode": "cfq",
                "sectors": "83886080",
                "sectorsize": "512",
                "size": "40.00 GB",
                "support_discard": "0",
                "vendor": "ATA"
            }
        },
        "ansible_distribution": "Ubuntu",
        "ansible_distribution_major_version": "14",
        "ansible_distribution_release": "trusty",
        "ansible_distribution_version": "14.04",
        "ansible_dns": {
            "nameservers": [
                "10.0.2.3",
                "192.168.10.1"
            ]
        },
        "ansible_domain": "",
        "ansible_effective_group_id": 0,
        "ansible_effective_user_id": 0,
        "ansible_env": {
            "HOME": "/root",
            "LOGNAME": "root",
            "MAIL": "/var/mail/root",
            "PATH": "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games",
            "PWD": "/root",
            "SHELL": "/bin/bash",
            "SHLVL": "1",
            "SSH_CLIENT": "172.17.0.2 56958 22",
            "SSH_CONNECTION": "172.17.0.2 56958 172.17.0.3 22",
            "SSH_TTY": "/dev/pts/1",
            "TERM": "xterm",
            "USER": "root",
            "_": "/bin/sh"
        },
        "ansible_eth0": {
            "active": true,
            "device": "eth0",
            "features": {},
            "ipv4": {
                "address": "172.17.0.3",
                "broadcast": "global",
                "netmask": "255.255.0.0",
                "network": "172.17.0.0"
            },
            "macaddress": "02:42:ac:11:00:03",
            "mtu": 1500,
            "promisc": false,
            "speed": 10000,
            "type": "ether"
        },
        "ansible_fips": false,
        "ansible_form_factor": "Other",
        "ansible_fqdn": "47bcd30760e2",
        "ansible_gather_subset": [
            "hardware",
            "network",
            "virtual"
        ],
        "ansible_hostname": "47bcd30760e2",
        "ansible_interfaces": [
            "lo",
            "eth0"
        ],
        "ansible_kernel": "3.10.0-514.10.2.el7.x86_64",
        "ansible_lo": {
            "active": true,
            "device": "lo",
            "features": {},
            "ipv4": {
                "address": "127.0.0.1",
                "broadcast": "host",
                "netmask": "255.0.0.0",
                "network": "127.0.0.0"
            },
            "mtu": 65536,
            "promisc": false,
            "type": "loopback"
        },
        "ansible_lsb": {
            "codename": "trusty",
            "description": "Ubuntu 14.04.5 LTS",
            "id": "Ubuntu",
            "major_release": "14",
            "release": "14.04"
        },
        "ansible_machine": "x86_64",
        "ansible_memfree_mb": 32,
        "ansible_memory_mb": {
            "nocache": {
                "free": 220,
                "used": 268
            },
            "real": {
                "free": 32,
                "total": 488,
                "used": 456
            },
            "swap": {
                "cached": 2,
                "free": 1498,
                "total": 1535,
                "used": 37
            }
        },
        "ansible_memtotal_mb": 488,
        "ansible_mounts": [
            {
                "device": "/dev/mapper/VolGroup00-LogVol00",
                "fstype": "xfs",
                "mount": "/etc/resolv.conf",
                "options": "rw,seclabel,relatime,attr2,inode64,noquota,bind",
                "size_available": 36874506240,
                "size_total": 40212119552,
                "uuid": "N/A"
            },
            {
                "device": "/dev/mapper/VolGroup00-LogVol00",
                "fstype": "xfs",
                "mount": "/etc/hostname",
                "options": "rw,seclabel,relatime,attr2,inode64,noquota,bind",
                "size_available": 36874506240,
                "size_total": 40212119552,
                "uuid": "N/A"
            },
            {
                "device": "/dev/mapper/VolGroup00-LogVol00",
                "fstype": "xfs",
                "mount": "/etc/hosts",
                "options": "rw,seclabel,relatime,attr2,inode64,noquota,bind",
                "size_available": 36874506240,
                "size_total": 40212119552,
                "uuid": "N/A"
            }
        ],
        "ansible_nodename": "47bcd30760e2",
        "ansible_os_family": "Debian",
        "ansible_pkg_mgr": "apt",
        "ansible_processor": [
            "GenuineIntel",
            "Intel(R) Core(TM) i5-5287U CPU @ 2.90GHz"
        ],
        "ansible_processor_cores": 1,
        "ansible_processor_count": 1,
        "ansible_processor_threads_per_core": 1,
        "ansible_processor_vcpus": 1,
        "ansible_product_name": "VirtualBox",
        "ansible_product_serial": "0",
        "ansible_product_uuid": "67FE42B1-3167-4F48-B89A-DC2B3627D854",
        "ansible_product_version": "1.2",
        "ansible_python": {
            "executable": "/usr/bin/python",
            "has_sslcontext": false,
            "type": "CPython",
            "version": {
                "major": 2,
                "micro": 6,
                "minor": 7,
                "releaselevel": "final",
                "serial": 0
            },
            "version_info": [
                2,
                7,
                6,
                "final",
                0
            ]
        },
        "ansible_python_version": "2.7.6",
        "ansible_real_group_id": 0,
        "ansible_real_user_id": 0,
        "ansible_selinux": false,
        "ansible_service_mgr": "upstart",
        "ansible_ssh_host_key_dsa_public": "AAAAB3NzaC1kc3MAAACBAL79cXHJanRSjNKsol5l6yq9PZ1EUjK5wL45M91dfZTuwimg2uZ6+GQ9brUODxAF8jNCNGVlObWTpD/I/trOavbwMdQxHFV5Wgkz14nRaf4NbBrQFH1XZQ88pwIke4AjS5VDwVQNgt4n+C3EzWsM0TZFMOpnYGv1htGiqf5BcoWfAAAAFQCycwjZuaaCWAIsmX2ZhrmLFaYa5QAAAIEAsR3I28y1hjmNFQ1ngVn+NsuGxkP+2OhPuQ1hRgkye+M5pY4AB8rrKUPdiIZzjh8h7aj0z8BIkJ4H95hsZmOmRr/I4dbC7p4JyjqRvXz6JsHKeAh4m4Np0BxcWD1uznbLrQJmdGnna1jHpBiHDAOkuOK6BBa655MYq/96poNHb38AAACBALoXqlLH9psbj+b9CE83Eg4GlbiQ5Ee9xyG8gmPIpSQe85lb0scAOrv1a7w2aj6qvghoQOOCOMmt0/OmoYNq/ksGIb5XbRlvAItQyWv8lv1/Y7jR5yP+aliUgprDJJceTYEcWYCRra9qEv8O2/qA+XzSH4cXReNKxOmyl3mk3BeL",
        "ansible_ssh_host_key_ecdsa_public": "AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJmP5qEhIJlL8ML8DWeq6vF1LLrq8C42b0Nslu9WC56c5H1e9TU1CzGSmrchVceYyT2HW8/XsTjz5SHGyNZWhz8=",
        "ansible_ssh_host_key_ed25519_public": "AAAAC3NzaC1lZDI1NTE5AAAAIODdRxqOihKqnUR5jfblLnTk0+etUtReRr9D75w1GZn2",
        "ansible_ssh_host_key_rsa_public": "AAAAB3NzaC1yc2EAAAADAQABAAABAQDDgSWmPaXdOgco4h961Mm2jRCNQw3nXS0q0j9B2wVscTjaf6R6opeamx7f/Czjc0uq7snsyhDl9k71xRHPLmdVLvnoe5x+ZU7BJVUkaS07a26BtcEElvK/i6ofuXxltOnCjQ3unhItxCgywaP08I4k5S/J+8gKLFMA6wWxUUEM0l0trvPW07t4JuisoQ9Lfnddcf69sHj/EFf4rMg5GezkWWzpvViOD0gyCMwK5mF5sUyDy6AFAXe6sonn4AcU2a+frBvsY2q655u73VIiC/7E5dAB7Y+iD8aVEOmdCFcTlWajFsNWYs07Ypj7ryzaHKCZhXBJYima1aYZtAwvPRSt",
        "ansible_swapfree_mb": 1498,
        "ansible_swaptotal_mb": 1535,
        "ansible_system": "Linux",
        "ansible_system_capabilities": [
            "cap_chown",
            "cap_dac_override",
            "cap_fowner",
            "cap_fsetid",
            "cap_kill",
            "cap_setgid",
            "cap_setuid",
            "cap_setpcap",
            "cap_net_bind_service",
            "cap_net_raw",
            "cap_sys_chroot",
            "cap_mknod",
            "cap_audit_write",
            "cap_setfcap+ep"
        ],
        "ansible_system_capabilities_enforced": "True",
        "ansible_system_vendor": "innotek GmbH",
        "ansible_uptime_seconds": 73045,
        "ansible_user_dir": "/root",
        "ansible_user_gecos": "root",
        "ansible_user_gid": 0,
        "ansible_user_id": "root",
        "ansible_user_shell": "/bin/bash",
        "ansible_user_uid": 0,
        "ansible_userspace_architecture": "x86_64",
        "ansible_userspace_bits": "64",
        "ansible_virtualization_role": "guest",
        "ansible_virtualization_type": "docker",
        "module_setup": true
    },
    "changed": false
}
root@b556839ea9cf:/#
```
经常使用的命令：
1. 查看主机内存信息
```shell?linenums
root@b556839ea9cf:/# ansible 172.17.0.3 -m setup -a 'filter=ansible_*_mb' -k
SSH password:
172.17.0.3 | SUCCESS => {
    "ansible_facts": {
        "ansible_memfree_mb": 31,
        "ansible_memory_mb": {
            "nocache": {
                "free": 219,
                "used": 269
            },
            "real": {
                "free": 31,
                "total": 488,
                "used": 457
            },
            "swap": {
                "cached": 2,
                "free": 1498,
                "total": 1535,
                "used": 37
            }
        },
        "ansible_memtotal_mb": 488,
        "ansible_swapfree_mb": 1498,
        "ansible_swaptotal_mb": 1535
    },
    "changed": false
}
root@b556839ea9cf:/# ansible 172.17.0.3 -a 'free -m'
172.17.0.3 | SUCCESS | rc=0 >>
             total       used       free     shared    buffers     cached
Mem:           488        447         41          7          0        188
-/+ buffers/cache:        259        229
Swap:         1535         37       1498

root@b556839ea9cf:/#
```
2. 查看接口为eth0-2的网卡信息
```shell?linenums
root@b556839ea9cf:/# ansible 172.17.0.3 -m setup -a 'filter=ansible_eth[0-2]' -k
SSH password:
172.17.0.3 | SUCCESS => {
    "ansible_facts": {
        "ansible_eth0": {
            "active": true,
            "device": "eth0",
            "features": {},
            "ipv4": {
                "address": "172.17.0.3",
                "broadcast": "global",
                "netmask": "255.255.0.0",
                "network": "172.17.0.0"
            },
            "macaddress": "02:42:ac:11:00:03",
            "mtu": 1500,
            "promisc": false,
            "speed": 10000,
            "type": "ether"
        }
    },
    "changed": false
}
root@b556839ea9cf:/# ansible 172.17.0.3 -a 'ifconfig'
172.17.0.3 | SUCCESS | rc=0 >>
eth0      Link encap:Ethernet  HWaddr 02:42:ac:11:00:03
          inet addr:172.17.0.3  Bcast:0.0.0.0  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1223 errors:0 dropped:0 overruns:0 frame:0
          TX packets:1231 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:1212530 (1.2 MB)  TX bytes:178882 (178.8 KB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

root@b556839ea9cf:/#
```

3. 将所有主机的信息输入到/tmp目录下，每台主机的信息输入到主机名文件中
```shell?linenums
root@b556839ea9cf:/# ansible all -m setup --tree /tmp -k
...
root@b556839ea9cf:/# ls /tmp -l
total 24
-rw-r--r--. 1 root root 7545 Aug 14 23:16 172.17.0.3
-rw-r--r--. 1 root root 7545 Aug 14 23:16 172.17.0.4
-rw-r--r--. 1 root root 7545 Aug 14 23:16 172.17.0.5
root@b556839ea9cf:/# less /tmp/172.17.0.4
root@b556839ea9cf:/#
...
```

## file模块

file模块主要用于远程主机上的文件操作，file模块包含如下选项： 
- force：需要在两种情况下强制创建软链接，一种是源文件不存在但之后会建立的情况下；另一种是目标软链接已存在,需要先取消之前的软链，然后创建新的软链，有两个选项：yes|no 
- group：定义文件/目录的属组 
mode：定义文件/目录的权限
owner：定义文件/目录的属主
path：必选项，定义文件/目录的路径
recurse：递归的设置文件的属性，只对目录有效
src：要被链接的源文件的路径，只应用于state=link的情况
dest：被链接到的路径，只应用于state=link的情况 
state：  
directory：如果目录不存在，创建目录
file：即使文件不存在，也不会被创建
link：创建软链接
hard：创建硬链接
touch：如果文件不存在，则会创建一个新的文件，如果文件或目录已存在，则更新其最后修改时间
absent：删除目录、文件或者取消链接文件
使用示例：
    ansible test -m file -a "src=/etc/fstab dest=/tmp/fstab state=link"
    ansible test -m file -a "path=/tmp/fstab state=absent"
    ansible test -m file -a "path=/tmp/test state=touch"
四、copy模块
复制文件到远程主机，copy模块包含如下选项：
backup：在覆盖之前将原文件备份，备份文件包含时间信息。有两个选项：yes|no 
content：用于替代"src",可以直接设定指定文件的值 
dest：必选项。要将源文件复制到的远程主机的绝对路径，如果源文件是一个目录，那么该路径也必须是个目录 
directory_mode：递归的设定目录的权限，默认为系统默认权限
force：如果目标主机包含该文件，但内容不同，如果设置为yes，则强制覆盖，如果为no，则只有当目标主机的目标位置不存在该文件时，才复制。默认为yes
others：所有的file模块里的选项都可以在这里使用
src：要复制到远程主机的文件在本地的地址，可以是绝对路径，也可以是相对路径。如果路径是一个目录，它将递归复制。在这种情况下，如果路径使用"/"来结尾，则只复制目录里的内容，如果没有使用"/"来结尾，则包含目录在内的整个内容全部复制，类似于rsync。 
validate ：The validation command to run before copying into place. The path to the file to validate is passed in via '%s' which must be present as in the visudo example below.
示例如下：
ansible test -m copy -a "src=/srv/myfiles/foo.conf dest=/etc/foo.conf owner=foo group=foo mode=0644"
    ansible test -m copy -a "src=/mine/ntp.conf dest=/etc/ntp.conf owner=root group=root mode=644 backup=yes"
    ansible test -m copy -a "src=/mine/sudoers dest=/etc/sudoers validate='visudo -cf %s'"
五、service模块
用于管理服务
该模块包含如下选项： 
arguments：给命令行提供一些选项 
enabled：是否开机启动 yes|no
name：必选项，服务名称 
pattern：定义一个模式，如果通过status指令来查看服务的状态时，没有响应，就会通过ps指令在进程中根据该模式进行查找，如果匹配到，则认为该服务依然在运行
runlevel：运行级别
sleep：如果执行了restarted，在则stop和start之间沉睡几秒钟
state：对当前服务执行启动，停止、重启、重新加载等操作（started,stopped,restarted,reloaded）
使用示例：
ansible test -m service -a "name=httpd state=started enabled=yes"
    asnible test -m service -a "name=foo pattern=/usr/bin/foo state=started"
    ansible test -m service -a "name=network state=restarted args=eth0"
六、cron模块
用于管理计划任务包含如下选项： 
backup：对远程主机上的原任务计划内容修改之前做备份 
cron_file：如果指定该选项，则用该文件替换远程主机上的cron.d目录下的用户的任务计划 
day：日（1-31，*，*/2,……） 
hour：小时（0-23，*，*/2，……）  
minute：分钟（0-59，*，*/2，……） 
month：月（1-12，*，*/2，……） 
weekday：周（0-7，*，……）
job：要执行的任务，依赖于state=present 
name：该任务的描述 
special_time：指定什么时候执行，参数：reboot,yearly,annually,monthly,weekly,daily,hourly 
state：确认该任务计划是创建还是删除 
user：以哪个用户的身份执行
示例：
ansible test -m cron -a 'name="a job for reboot" special_time=reboot job="/some/job.sh"'
    ansible test -m cron -a 'name="yum autoupdate" weekday="2" minute=0 hour=12 user="root
    ansible test -m cron  -a 'backup="True" name="test" minute="0" hour="5,2" job="ls -alh > /dev/null"'
    ansilbe test -m cron -a 'cron_file=ansible_yum-autoupdate state=absent'
七、yum模块
使用yum包管理器来管理软件包，其选项有： 
config_file：yum的配置文件 
disable_gpg_check：关闭gpg_check 
disablerepo：不启用某个源 
enablerepo：启用某个源
name：要进行操作的软件包的名字，也可以传递一个url或者一个本地的rpm包的路径 
state：状态（present，absent，latest）
示例如下：
ansible test -m yum -a 'name=httpd state=latest'
    ansible test -m yum -a 'name="@Development tools" state=present'
    ansible test -m yum -a 'name=http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm state=present'
八、user模块与group模块
user模块是请求的是useradd, userdel, usermod三个指令，goup模块请求的是groupadd, groupdel, groupmod 三个指令。
1、user模块
home：指定用户的家目录，需要与createhome配合使用
groups：指定用户的属组
uid：指定用的uid
password：指定用户的密码
name：指定用户名
createhome：是否创建家目录 yes|no
system：是否为系统用户
remove：当state=absent时，remove=yes则表示连同家目录一起删除，等价于userdel -r
state：是创建还是删除
shell：指定用户的shell环境
使用示例：
    user: name=johnd comment="John Doe" uid=1040 group=admin
    user: name=james shell=/bin/bash groups=admins,developers append=yes user: name=johnd state=absent remove=yes
    user: name=james18 shell=/bin/zsh groups=developers expires=1422403387
    user: name=test generate_ssh_key=yes ssh_key_bits=2048 ssh_key_file=.ssh/id_rsa    #生成密钥时，只会生成公钥文件和私钥文件，和直接使用ssh-keygen指令效果相同，不会生成authorized_keys文件。
注：指定password参数时，不能使用明文密码，因为后面这一串密码会被直接传送到被管理主机的/etc/shadow文件中，所以需要先将密码字符串进行加密处理。然后将得到的字符串放到password中即可。
echo "123456" | openssl passwd -1 -salt $(< /dev/urandom tr -dc '[:alnum:]' | head -c 32) -stdin
$1$4P4PlFuE$ur9ObJiT5iHNrb9QnjaIB0
#使用上面的密码创建用户
ansible all -m user -a 'name=foo password="$1$4P4PlFuE$ur9ObJiT5iHNrb9QnjaIB0"'
不同的发行版默认使用的加密方式可能会有区别，具体可以查看/etc/login.defs文件确认，centos 6.5版本使用的是SHA512加密算法。
2、group示例
ansible all -m group -a 'name=somegroup state=present'
九、synchronize模块
使用rsync同步文件，其参数如下：
archive: 归档，相当于同时开启recursive(递归)、links、perms、times、owner、group、-D选项都为yes ，默认该项为开启
checksum: 跳过检测sum值，默认关闭
compress:是否开启压缩
copy_links：复制链接文件，默认为no ，注意后面还有一个links参数
delete: 删除不存在的文件，默认no
dest：目录路径
dest_port：默认目录主机上的端口 ，默认是22，走的ssh协议
dirs：传速目录不进行递归，默认为no，即进行目录递归
rsync_opts：rsync参数部分
set_remote_user：主要用于/etc/ansible/hosts中定义或默认使用的用户与rsync使用的用户不同的情况
mode: push或pull 模块，push模的话，一般用于从本机向远程主机上传文件，pull 模式用于从远程主机上取文件
使用示例：
    src=some/relative/path dest=/some/absolute/path rsync_path="sudo rsync"
    src=some/relative/path dest=/some/absolute/path archive=no links=yes
    src=some/relative/path dest=/some/absolute/path checksum=yes times=no
    src=/tmp/helloworld dest=/var/www/helloword rsync_opts=--no-motd,--exclude=.git mode=pull
十、filesystem模块
在块设备上创建文件系统
选项： 
dev：目标块设备
force：在一个已有文件系统 的设备上强制创建
fstype：文件系统的类型
opts：传递给mkfs命令的选项
示例：
    ansible test -m filesystem -a 'fstype=ext2 dev=/dev/sdb1 force=yes'
    ansible test -m filesystem -a 'fstype=ext4 dev=/dev/sdb1 opts="-cc"'
十一、mount模块
配置挂载点
选项： 
dump
fstype：必选项，挂载文件的类型 
name：必选项，挂载点 
opts：传递给mount命令的参数
src：必选项，要挂载的文件 
state：必选项 
present：只处理fstab中的配置 
absent：删除挂载点 
mounted：自动创建挂载点并挂载之 
umounted：卸载
示例：
    name=/mnt/dvd src=/dev/sr0 fstype=iso9660 opts=ro state=present
    name=/srv/disk src='LABEL=SOME_LABEL' state=present
    name=/home src='UUID=b3e48f45-f933-4c8e-a700-22a159ec9077' opts=noatime state=present
    ansible test -a 'dd if=/dev/zero of=/disk.img bs=4k count=1024'
    ansible test -a 'losetup /dev/loop0 /disk.img'
    ansible test -m filesystem 'fstype=ext4 force=yes opts=-F dev=/dev/loop0'
    ansible test -m mount 'name=/mnt src=/dev/loop0 fstype=ext4 state=mounted opts=rw'
十二、get_url 模块
该模块主要用于从http、ftp、https服务器上下载文件（类似于wget），主要有如下选项：
sha256sum：下载完成后进行sha256 check；
timeout：下载超时时间，默认10s
url：下载的URL
url_password、url_username：主要用于需要用户名密码进行验证的情况
use_proxy：是事使用代理，代理需事先在环境变更中定义
示例：
    get_url: url=http://example.com/path/file.conf dest=/etc/foo.conf mode=0440
    get_url: url=http://example.com/path/file.conf dest=/etc/foo.conf sha256sum=b5bb9d8014a0f9b1d61e21e796d78dccdf1352f23cd32812f4850b878ae4944c
十三、unarchive模块
用于解压文件，模块包含如下选项：
copy：在解压文件之前，是否先将文件复制到远程主机，默认为yes。若为no，则要求目标主机上压缩包必须存在。
creates：指定一个文件名，当该文件存在时，则解压指令不执行
dest：远程主机上的一个路径，即文件解压的路径 
grop：解压后的目录或文件的属组
list_files：如果为yes，则会列出压缩包里的文件，默认为no，2.0版本新增的选项
mode：解决后文件的权限
src：如果copy为yes，则需要指定压缩文件的源路径 
owner：解压后文件或目录的属主
示例如下：
- unarchive: src=foo.tgz dest=/var/lib/foo
- unarchive: src=/tmp/foo.zip dest=/usr/local/bin copy=no
- unarchive: src=https://example.com/example.zip dest=/usr/local/bin copy=no










注：raw模块和comand、shell 模块不同的是其没有chdir、creates、removes参数，chdir参数的作用就是先切到chdir指定的目录后，再执行后面的命令，这在后面很多模块里都会有该参数 。
command模块包含如下选项： 
creates：一个文件名，当该文件存在，则该命令不执行 
free_form：要执行的linux指令 
chdir：在执行指令之前，先切换到该指定的目录 
removes：一个文件名，当该文件不存在，则该选项不执行
executable：切换shell来执行指令，该执行路径必须是一个绝对路径

使用chdir的示例：
ansible 192.168.1.1 -m command -a 'chdir=/tmp/test.txt touch test.file'
ansible 192.168.1.1 -m shell -a 'chdir=/tmp/test.txt touch test2.file'
ansible 192.168.1.1 -m raw -a 'chdir=/tmp/text.txt touch test3.file'
三个命令都会返回执行成功的状态。不过实际上只有前两个文件会被创建成功。使用raw模块的执行的结果文件事实上也被正常创建了，不过不是在chdir指定的目录，而是在当前执行用户的家目录。
creates与removes示例：
ansible 192.168.1.1 -a 'creates=/tmp/server.txt uptime' #当/tmp/server.txt文件存在时，则不执行uptime指令
ansible 192.168.1.1 -a 'removes=/tmp/server.txt uptime' #当/tmp/server.txt文件不存在时，则不执行uptime指令

script模块示例：
要执行的脚本文件script.sh内容如下：
#/bin/bash
ifconfig
df -hT
执行ansible指令：ansible 10.212.52.252 -m script -a 'script.sh' |egrep '>>|stdout'


  [1]: ./images/1502850110093.jpg
  [2]: ./images/1502850202035.jpg
  [3]: ./images/1502850593608.jpg
  [4]: ./images/1502850641563.jpg
  [5]: http://docs.ansible.com/list_of_all_modules.html