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




  [1]: ./images/1502850110093.jpg
  [2]: ./images/1502850202035.jpg
  [3]: ./images/1502850593608.jpg
  [4]: ./images/1502850641563.jpg