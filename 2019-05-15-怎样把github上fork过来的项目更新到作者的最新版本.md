---
title: 怎样把github上fork过来的项目更新到作者的最新版本
description: 只需要在网页上操作下，不用命令行
#date: 2019-11-22 
#updated: 2018-11-22
categories:
- github
tags:
- github
---

&emsp;&emsp;github的pull request的功能是如果你fork了某人的项目，并对原作者的代码进行了一些修改，想要合并到他的项目中，那你就可以通过pull request来请求合并。如果你只是想备份一个仓库，而原作者对自己的代码有了更新，你想要同步到自己的github上，该如何解决呢？我们可以将pull request反过来用，将我们fork过来的项目作为被合并的项目，而原作者的项目作为请求合并的项目。这样通过pull request，我们就可以把原作者最新的项目同步到我们的项目上。

1.点击New pull request
![new_pull_request](2019-05-15-怎样把github上fork过来的项目更新到作者的最新版本.assets/190515-new_pull_request.png)

2.点击switching the base，将左边的base换成自己的项目，右边的comapre换成原作者的项目
![switching_the_base](2019-05-15-怎样把github上fork过来的项目更新到作者的最新版本.assets/190515-switching_the_base.png)

3.点击Create pull request
![create_pull_request](2019-05-15-怎样把github上fork过来的项目更新到作者的最新版本.assets/190515-create_pull_request.png)

4.取个名字写下说明，点击Create pull request
![取个名字](2019-05-15-怎样把github上fork过来的项目更新到作者的最新版本.assets/190515-取个名字.png)

5.这一步选第二个，压缩提交，相当于把之前所有的改动合并，简单点，顺便节省点空间
![压缩提交](2019-05-15-怎样把github上fork过来的项目更新到作者的最新版本.assets/190515-压缩提交.png)

6.最后确认
![确认提交](2019-05-15-怎样把github上fork过来的项目更新到作者的最新版本.assets/190515-确认提交.png)