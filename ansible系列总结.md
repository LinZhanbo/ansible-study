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





  [1]: ./images/1502850110093.jpg
  [2]: ./images/1502850202035.jpg