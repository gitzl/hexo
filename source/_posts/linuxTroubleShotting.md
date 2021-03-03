---
title: Linux-Online-Troubleshooting
---


### 日志

```
less catalina.out

j    下一行
k    上一行
f    向下滚动一屏幕
b    向上滚动一屏幕
g    定位到文档头部
G    定位到文档最尾部
q    退出less模式

/keyword　　向下查找
n    向下匹配下一处匹配文本
N    向上匹配下一处匹配文本

?keyword　　向上查找
n    向上匹配下一处匹配文本
N    向下匹配下一处匹配文本


head  -10 catalina.out  前10行的数据
tail -8   catalina.out  后10行数据

```


### 磁盘文件

```

df -h 查看整体磁盘目录占用情况
du -sh * 查看当前目录下文件大小
du -sh  xxx 指定文件名查看文件大小
find / -name xxx 查询文件


fdisk -l  显示分区列表
mount /dev/hda1     /mnt 目录挂载
parted 磁盘分区

```

### 网络

```

## 抓包
 tcpdump ip host 210.27.48.1 and 210.27.48.2 -w test.cap  ## 抓特定主机之间包
 tcpdump -i any port 5161 -w auth.cap   #抓包 使用：Wireshark分析
 tcpdump -s 0 -v  -i any -w tet.pcap

## 查看网卡信息
 ifconfig 查看网卡信息  
 ethtool ens1f0 （网卡名）  | grep Speed（网卡名）  ##查看千兆还是万兆网卡 ：1000M/s-表示千兆

iftop   ## 查看本机流量走向

sudo lsof -i:32100 ##查看端口

```

### 监控

```
top #整体查看cpu 内存使用情况
nmon #较详细查看：CPU Memory   Disk NetWork等使用情况

```

### JVM

```
jps 查看java进程: pid
jmap -dump:live,format=b,file=auth.hprof  pid  #生成堆栈文件、使用MAT进下分析

```