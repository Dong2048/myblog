---
title: 博客搭建的那些事情
date: 2021-10-29 15:33:09
tags: 博客
categories : 技术
top_img: https://s4.ax1x.com/2022/02/09/H806e0.jpg
cover: https://s4.ax1x.com/2022/02/09/H806e0.jpg
---
### 1、创建hexo框架博客维护和发布文章的目录
> windows系统在桌面创建文件夹**blog**
> 
> Linux系统在Home目录下或者/usr/local目录下创建**blog**
### 2、安装hexo框架必要的环境
#### （1）安装git
> windows系统在git上下载安装程序
> linux系统 执行命令  
> sudo apt-get install git  
> 安装完成后查看版本号  
> git --version  
>windows系统在git客户端命令窗口执行
#### （2）安装nodejs和npm
> 执行安装命令  
> sudo apt-get install nodejs  
> sudo apt-get install npm  
> 安装完成后查看版本号  
> nodejs -v   
> npm -v
#### （3）安装hexo
> npm install hexo-cli -g  
> 如果报错 nodejs和npm版本低，升级nodejs和npm版本  
> 开始升级nodejs：
> 先清除npm缓存  
> npm cache clean -f
> 然后安装n模块：n模块不支持Windows，Windows官网下载最新版本。  
> 安装 nodejs 的版本管理器 n  
> npm install -g n   
>  最新版本： n lastest  
>  稳定版本： n stable  
>  安装指定版本： n 10.12.0  
> 开始升级npm：  
> npm install npm@latest -g  
>  升级完成后 在执行 npm install hexo-cli -g  
#### （4) 初始化hexo
> 查看版本  
> hexo -v  
> 初始化hexo  
> hexo init   
> 安装依赖  
> npm install  
> 完成后 hexo g hexo s  
> hexo 常用命令 ：  
> hexo clean 清缓存  
> hexo g 更新  
> hexo s 启动本地服务hexo s 是 hexo server   
> 生成网站静态文件到默认设置的 public 文件夹，hexo g 是 hexo generate 的缩写，命令效果一致  
> hexo d 自动生成网站静态文件，并部署到设定的仓库,hexo d 是 hexo deploy 的缩写，命令效果一致
### 3、创建GitHub个人仓库
> 为git 添加用户名和邮箱  
> git config --global user.name "yourname"  
> git config --global user.email "youremail"  
> 检查是否写入成功  
> git config user.name   
> git config user.email
> 创建连接git的私钥  
> ssh-keygen -t rsa -C "youremail"  
> 在home目录下找到.ssh文件夹中id_rsa是电脑的私人秘钥，id_rsa.pub是公共秘钥。  
> 登陆GitHub，在setting中，找到SSH keys的设置选项，点击New SSH key，把id_rsa.pub里面的公钥信息复制进去。  
> 检查是否生效  
> ssh -T git@github.com  
> 出现Hi Qing-Yu-1! You've successfully authenticated, but GitHub does not provide shell access. 成功。
### 4、hexo部署到github
####  (1) hexo编译后的文件上传github，github部署访问。  
> 修改配置文件_config   
> deploy:  
> type: git  
> repo: git@github.com:XXXX/XXXX.github.io.git  
> branch: master    
> 安装
####  (1) hexo源码文件上传github，通过vercel同步github源码库在vercel部署访问。
> 使用Vercel必须源码上传github，不能使用hexo d上传编译过的项目。  
> 注册Vercel 关联github，在Vercel设置中配置域名。  
> 完成博客搭建。  