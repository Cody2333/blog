---
title: Git工作流的最佳实践
tags: [
    "git",
]
date: 2017-04-18
categories: [
    "git"
]
---

## 简介

今天来总结一下在实际项目开发中使用Git工作流的最佳实践，项目使用gitlab作为版本控制。比较适合小型或者中型的多人合作，快速迭代开发的项目。

## 分支

首先来介绍一下分支的设计。一般分为这么几类分支:

- master
- develop
- feature分支
- bug分支

master为主分支，为protected分支，只允许其他分支的merge，不允许直接push代码。
develop为开发的主分支，feature和bug分支提交merge request到develop。

在开发中，针对每个功能开一个feature分支进行开发，命名可以为 [feature/xxx]，开发完成后提交merge request，合并到develop分支。

针对开发中遇到的bug，新开一个bug分支，修复后，也提交merge request到develop分支。

## 开发流程

注意点：

- 规范的commit message格式
- 拉取develop分支代码的时候使用rebase而不是merge
- 使用rebase整理个人feature分支的commit
- 开merge request进行code review

首先应该制定一个commit的message的规范和格式，要求每次提交的commit message清晰。具体的格式可以根据团队的习惯制定，只需要能表述清晰即可。

对于个人的feature分支，在准备提交MR合并进develop之前，应该用git rebase来整理自己的commit，使得逻辑清楚。同时可以避免产生[merge branch develop ***]这样的commit。

具体操作如下：
```
// 在feature/xxx分支上进行开发，并进行了一些提交，此时要合并到develop分支

// 首先用rebase的方式拉取develop分支代码
git pull origin develop --rebase

// 如果有conflict，修复conflict，然后
git rebase --continue

// 如果有多个conflict，不同于直接merge，使用rebase可能需要多次解决conflict，并多次执行git rebase --continue

// 如果有需要使用
git rebase -i
// 来整理自己的commit，可以用来将一些小的commit合并，以及修改优化自己的commit message

// 然后push到feature/xxx分支，由于rebase，可能会导致本地与远端的history不一致，使用force强制重写远端分支
git push --force

// 最后在gitlab上开一个merge request到develop分支

// 团队其他成员review这个MR，并提出修改意见，然后根据修改意见，优化原来的代码，把存在问题的地方改掉，知道最后没有问题，将feature/xxx分支合并到develop
```
## 部署

master分支永远是稳定的版本，在master分支上，可以使用CI工具进行持续集成和自动部署。develop分支每次完成部分功能就可以merge到master，使得项目可以快速高效的迭代开发。

## rebase or merge？

在个人分支上，推荐只使用rebase而不是merge，因为rebase可以让你的提交看起来意义明确，并且没有无意义的merge其他分支的commit。

而在主分支上， 只允许merge操作，从而保证主分支的commit历史的完整性。在多人开发的主分支上进行rebase操作是一件很危险的行为，可能会导致其他成员的history与主分支不一致。

## 总结

个人认为，现在这样的git工作流可以很好的适应小团队的项目快速迭代开发，使得项目网络清晰明了。但是当项目变得复杂的时候，可能需要更复杂的控制，这部分没有什么实践的经验就不谈了。如果有不严谨的地方请指出。
