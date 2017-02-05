---
title: Github + Hexo
date: 2016-05-22 09:31:11
tags: [hexo, github]
categories: hexo
---

May 21, 2016 9:31 AM
# Github + Hexo
### 安装

1. nodejs安装，先到nodejs.org/en网站下载nodejs
2. 安装git

### 安装Hexo

1. 在git中输入
``` bash
npm install -g hexo
```

### 部署Hexo
1. 创建一个目录 这里就创建一个hexo目录吧，然后在git cd到该目录下，并输入
```bash
hexo init
```
然后Hexo随后会自动在目标文件夹建立网站所需要的所有文件。

2. 然后我们输入如下命名
```bash
hexo g
hexo s
```
`hexo g`是用于生成网站文件如 index.html,而`hexo s`是用于启动网站，这样我们在浏览器中输入http://localhost:4000/就可以看到网站了。

### 配置GitHub
1. 登陆github后new一个Repository，该repository的名字必须为username.github.io，这是固定格式。因为github pages分两种，一种就是github用户建立的用户&组织页（站）这样的，另一种是依附于项目的pages。

### 建立hexo和github的关联
1. 在本地文件_config.yml文件最后加入
```bash
deploy:
type:git
repo:你的repository https的地址
branch:master
```

2. 然后执行命令:
```bash
npm install hexo-deployer-git --save
```

3. 最后执行部署命令：`hexo deploy`
   **这条命令很重要，不然 `hexo deploy`时会提示错误，无法解析git，对了type:一定是git,不是github。**
   **对了 repo中的地址用 SSH，不要用HTTPS，用HTTPS不知道为什么就行不行，可能是哪里配置有问题没有加密码什么的吧**
   **有问题可以看[这个网站,是hexo的官网，有文档](https://hexo.io/docs/)**

### 修改主题
  在根目录下修改_config.yml文件中的theme属性就可以了，将其设置为themes目录下的一个目录名字就行。然后执行部署。

### 总结
  其实`hexo d`之后也只是将hexo目录下public目录下的所有东西往github上推送而已，我们可以在`hexo g`之后自己用`git push`将public里的东西往github上推。
