# 引言

有一篇很出名的文章：[It's the future](https://circleci.com/blog/its-the-future/)，文章大意是一个boss想要部署一个简单的CRUL rails项目，他咨询一位专家，专家极力反对他将项目部署在heroku上，并给出的建议是将他的简单的app拆分成12个微服务\(microservices\)，运行在docker container中，彼此通过api调用。然后将这些微服务部署在一个由8台机器组成的kubernetes集群中。

初看这篇文章的时候，我并没有理解这种部署的优势。但随着之后课程格子在部署方式的改变，我重新回顾了这篇文章，发现当业务和服务复杂起来时，使用kubernetes集群部署的优势所在。

#### 当时课程格子的后端组织

* 一个巨大rails项目，承载了app端的所有版本的api、很多h5页面、以及运营同学的运营工具
* 电商运营后台
* 教务爬虫系统
* 后端存储服务，如MySQL，PG，Redis，ElasticSearch

项目以ruby为主，部署使用capstrano。

#### 项目的水平扩展

* 在这台机器上添加负责部署的工程师public key
* 为项目添加这台机器的public key作为deploy key
* 配置ruby环境
* 拷贝项目的production配置文件
* 配置nginx作为反向代理，serve静态文件
* 针对机器硬件设置web服务器（puma）的进程数、线程数
* 讲机器端口加入Load Balance

这个过程很枯燥也很机械，一个熟练的后端工程师，能在半小时内搞定这些事情。在项目少，机器数量不多的时候，这种方式可以接受。

#### 随着机器增多，项目增多，以及工程师增多，问题出现：

* 部署一个项目时(多环境：sandbox, staging, production)，耗费很多时间，由于项目的增多，这个问题愈发严重
* 服务器的权限管理混乱
* 因为手动分配的计算资源不合理，导致集群部分机器闲的要死，部分机器忙的要命
* 需要编写脚本监视管理进程（god）
* 自动化程度低
* 开始使用更多的语言和框架，部署方式分散，对工程师要求越来越高

#### 开始探索新的部署方案，期望能达到：

* 自动化部署
* 简单的多环境部署 \(sandbox，staging，production\)
* 进程监控、出错重启
* 水平扩展简单(scaling)
* 部署期间服务可用(rolling update)
* 部署出现问题时，能快速回滚(rollback)
* 统一多种语言、框架的部署方式
* 最小程度开放服务器权限

#### 最终采取的方案

gitlab + docker + kubernetes

#### 目前的部署流程

目前格子的开发到部署到上线的流程如下

1. 新建branch开发需求
2. commit
3. push branch
  - gitlab ci 构建此branch的image
  - gitlab ci 将构建好的image提交到registry
  - gitlab ci 在这个image的基础上运行测试
  - gitlab ci 将这个image发布到sandbox环境中
4. 创建merge request
5. Code Review
6. Merge Branch To Master
  - gitlab ci 构建master的image
  - gitlab ci 将构建好的image提交到registry
  - gitlab ci 在这个image的基础上运行测试
  - gitlab ci 将这个image发布到sandbox环境
  - gitlab ci 将这个image发布到staging环境
7. 确认staging环境代码工作正常
8. 手动点击gitlab pipeline中的按钮，一键部署到production

接下来的部分，我会向大家来介绍如何实现以上的部署方式
