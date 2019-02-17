---
title: 与ansible的第一次对话
---

>ansible 是个神马东东?

1.  基于python的自动化运维工具
2.  可以实现批量执行命令、批量系统配置、批量部署等自动化功能

> 通信协议

1. ansible默认通过SSH协议实现主机之间的通信，管理主机

>敲下第一个命令

```
ansible --version #获取版本,确定安装成功
ansible all -a "/bin/echo hello" 
```
tips：安装ansible，请参考 http://www.ansible.com.cn/docs/intro_installation.html

>ansible hosts

hosts路径：cat etc/ansible/hosts
hosts可注册目标主机的地址，如下:

```
#   - You can enter hostnames or ip addresses
#   - A hostname/ip can be a member of multiple groups

# Ex 1: Ungrouped hosts, specify before any group headers.

139.159.228.59
192.168.1.188
## green.example.com
## blue.example.com
## 192.168.100.1
## 192.168.100.10

# Ex 2: A collection of hosts belonging to the 'webservers' group

## [webservers]
## alpha.example.org
## beta.example.org
## 192.168.1.100
## 192.168.1.110

```
>SSH认证

ansible基于SSH协议进行通信，因此主机之间通信需要取得SSH认证

```
cd ~/.ssh
ls -a 
# authorized_keys  id_rsa  id_rsa.pub
# authorized_keys  假设当前主机为:A，如果另一台主机B要通过SSH和A通信，将A主机下的公钥添加到B主机下的authorized_keys
#id_rsa.pub 公钥 
#id_rsa 私钥
```
> 模拟一个场景：A主机下执行playbook 远程给主机B创建一个名为:hello的目录

step1：

```
将A主机下的ssh目录下的公钥(id_rsa.pub)添加到B主机ssh目录下的authorized_keys
```
step2：

```
在A主机下的/etc/ansible/hosts里面添加B主机的IP地址
```
step3：ansible-playbook helloPlayBook.yml -i hosts #执行第一个playbook
###### helloPlayBook.yml：
```
- hosts: 139.159.228.59
  tasks:
    - name: creat a folder
      command: mkdir hello
# 如果 /etc/ansible/hosts注册了 139.159.228.59 ，执行：ansible-playbook helloPlayBook.yml
# 如果 /etc/ansible/host未注册 139.159.228.59，执行：ansible-playbook helloPlayBook.yml -i 139.159.228.59,
# 多个目标主机地址用逗号分隔
```
###### tips ：更多内容可以参考: http://ansible.com.cn/docs/