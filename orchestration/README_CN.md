# 简单的 Ansible 基础架构编排实例

Ansible Planybook 灵活而强大。但有时，你只需要在一组主机上运行一条命令。这时，简单的`ansible`命令就派上用场了。

就像《Ender's Game》中的设备一样，`ansible`命令允许你在一台或一百台服务器上立即运行命令或调用Ansible模块。

这个项目使用Vagrant和VirtualBox配置了三个虚拟机。`app1`、`app2`和`db`，以模拟一个小规模的真实世界基础设施（两个应用服务器和一个数据库服务器），因此您可以练习在它们之间运行`ansible`命令，并在一个灵活的Ansible清单上工作。

## 快速入门指南

### 1 - 安装依赖关系（VirtualBox, Vagrant, Ansible）。

  1. 下载并安装【VirtualBox】(https://www.virtualbox.org/wiki/Downloads)。
  2. 下载并安装 [Vagrant](http://www.vagrantup.com/downloads.html)。
  3. 仅限Mac/Linux]安装[Ansible](http://docs.ansible.com/intro_installation.html)。

Windows用户注意。*本指南假设您使用的是Mac或Linux主机。目前不支持Windows主机。

### 2 - 构建虚拟机

  1. 下载这个项目，并把它放在你想放的地方。
  2. 打开终端，cd到这个目录（包含`Vagrantfile`和这个README文件）。
  3. 输入`vagrant up`，让Vagrant发挥它的魔力。

注意： *如果在运行`vagrant up`的过程中出现任何错误，并且它将您丢回命令提示符，只需运行`vagrant provision`命令，它将继续从您报错的地方构建虚拟机。如果做了几次后仍然有错误，请在GitHub上的项目问题队列中发布一个问题并注明错误。


#### ansible.cfg

定义当前目录下执行 ansible 命令的配置参数， cfg 文件中调用了 当前目录下的 hosts.ini 清单文件。
```
[defaults]
inventory = hosts.ini
nocows = True
```

#### hosts.ini

```
# Application servers
# 应用服务器组
[app]
192.168.60.4
192.168.60.5

# Database server
# 数据库服务器组
[db]
192.168.60.6

# Group 'multi' with all servers
# 名为 multi 的嵌套组
[multi:children]
app
db

# Variables that will be applied to all servers
# 给嵌套组定义变量，应用于所有服务器
[multi:vars]
ansible_user=vagrant
ansible_ssh_private_key_file=~/.vagrant.d/insecure_private_key
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```



### 3 - 创建一个清单文件，并运行`ansible`命令。

详情请阅读【Ansible for DevOps】(https://www.ansiblefordevops.com/)第三章。

## 关于作者

这个项目是由[Jeff Geerling](https://www.jeffgeerling.com/)创建的，作为[Ansible for DevOps](https://www.ansiblefordevops.com/)的一个例子。