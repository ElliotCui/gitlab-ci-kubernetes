# 引言

有一篇很出名的文章：[https://circleci.com/blog/its-the-future/](https://circleci.com/blog/its-the-future/ "asdfasdf")，文章大意是一个boss想要部署一个简单的CRUL rails项目，他咨询一位专家，专家极力反对他将项目部署在heroku上，并给出的建议是将他的简单的app拆分成12个微服务\(microservices\)，运行在docker container中，彼此通过api调用。然后将这些微服务部署在一个由8台机器组成的kubernetes集群中。

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

* 第一次部署一个项目时，耗费很多时间，由于项目的增多，这个问题愈发严重
* 服务器的权限管理混乱
* 因为手动分配的计算资源不合理，导致集群部分机器闲的要死，部分机器忙的要命
* 需要编写脚本监视管理进程（god）
* 自动化程度低
* 开始使用更多的语言和框架，部署方式分散，对工程师要求越来越高

#### 开始探索新的部署方案，期望能达到：

* 自动化部署
* 简单的多环境部署 \(sandbox，staging，production\)
* 进程监控、出错重启
* 简单的水平扩展
* 部署出现问题，快速rollback
* 统一多种语言、框架的部署方式



