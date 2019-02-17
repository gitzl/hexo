---
title: Linux
---
> 查看操作系统信息
``` bash
uname -a
cat /proc/version
cat /etc/issue
lsb_release -a
```
> centos crontab操作
``` bash
cat /ect/crontab
crontab -e
* * * * * date >> /home/date1.txt　　测试例子
crontab -l 查看
crontab -r 终止
/sbin/service crond start
/sbin/service crond stop
/sbin/service crond restart
/sbin/service crond reload
```
> vi/vim 常用命令
```bash
dd 删除当前行
u  撤销，恢复到上一次操作
home/end 一行首和尾
/   查找
```
> 操作文件

``` bash
df -h     查看磁盘使用状态       
df -a -lh 查看全部磁盘使用情况  
ls -lh    查看文件大小             
du -a     列出所有文件              
fdisk -l  列出磁盘信息         
du -sh *  查看目录下全部文件的大小  

```
> yum 命令
```bash
yum update  xx
yum install xx
yum search  xx
yum remove  xx
```
> Linux分屏工具tmux
```bash
ctrl+b 松开  % 面板分屏
ctrl+b  o 切换
```
> web开发命令

```bash

curl -I  http://www.baidu.com      
tar -xf  xxx.tar
tar -xzf xxx.tar.gz  解压          
top 监控        
touch 创建文件              
mkdir 创建目录        
rm -rf 删除        
cp -r xxx/xx/ .      
mv xxx/xx  yyy/yy 移动       
cat xxx.sh 显示       

```

