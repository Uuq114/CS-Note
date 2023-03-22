[toc]

## Introduction

* <u>Goal</u>: simple IT automation

* manage cloud instances
* perform arbitrary IT orchestration(update, hotfix...)



## Concept

* control node: 安装ansible的机器，控制节点
* inventory: `ini`文件。记录了管理的机器的信息
* playbook: `yaml`文件。记录了一系列需要执行的操作
* task: 定义一个要执行的操作。例如，安装一个包
* module: module抽象了一些系统命令，比如创建文件、修改文件。ansible有一些built-in module
* role: 一些相关联的playbook、template等文件的集合，为了可以重用将它们组合起来
  * 例如，项目中经常需要配置的有：httpd、php服务器、mysql服务器，那么为了模块化调用，就可以定义roles为websvr、phpappsrv、dbsrv。多个角色可以独立重复调用
* facts: 全局变量，记录了系统的有关信息，例如网络接口等
* handlers: 用来触发服务状态的变化，比如restart、reload

