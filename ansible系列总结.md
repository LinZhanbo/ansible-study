---
title: ansible系列总结
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---

# 1. 简介与基本安装

Ansible是一个综合的强大的管理工具,他可以对多台主机安装操作系统,并为这些主机安装不同的应用程序,也可以通知指挥这些主机完成不同的任务.查看多台主机的各种信息的状态等,ansible都可以通过模块的方式来完成。
1、Ansible特性
No agents：不需要再被管理节点上安装客户端，只要有sshd即可
No server：在服务端不需要启动任何服务，只需要执行命令就行
No additional PKI：由于不基于ssl，所以也不基于PKI工作
Modules in any language：基于模块工作，ansible拥有众多的模块
YAML：支持YAML语法
SSH by default：默认使用ssh控制各节点

