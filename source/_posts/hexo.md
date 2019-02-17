---
title: hexo+github搭建博客以及主题更换
---

> step1：安装node.js

* 下载node ：https://nodejs.org/zh-cn/
* 配置path

> step2: 安装Hexo

* 创建一个目录为：hexo
* cd ~/hexo 进入 ，执行如下命令
* sudo npm install -g hexo
* hexo init
* hexo generate（hexo g也可以，生成静态页面）
* hexo server (启动)
* http://localhost:4000 本地可以访问了

> step3: Git配置

* 初始化Git,提交SSH key到gitHub
* 建立GitHub repository ，仓库名[ your_git_name].github.io，后面用这个域名直接访问。
* 配置_config.yml文件，找到deploy节点，修改如下:
		deploy:

			type: git

			repo: git@github.com:[ your_git_name]/[ your_git_name].github.io.git

			branch: master

> step3: 发布到GitHub

* 安装Hexo-delpoyer-git
* npm install hexo-deployer-git --save
* hexo deploy

> step4: 访问博客地址

* [ your_git_name].github.io

> 主题更换

* hexo主题地址：https://hexo.io/themes，我选择的主题是：hexo-theme-next

* install with git 
```bash
 cd hexo
 git clone https://github.com/iissnan/hexo-theme-next themes/next
```
* update  _config.yml
```bash
theme: next 注意:theme和next之间有个空格
```
* run 
```bash
hexo clean 清理缓存数据
hexo s --debug 
hexo g 编译
hexo d 发布
```