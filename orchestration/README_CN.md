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

被管理服务器定义清单文件 inventory 。



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


### 运行 ad-hoc 命令

运行多次下面的命令，了解多线程并发的特性

ansible multi -a "hostname"
ansible multi -a "hostname"
ansible multi -a "hostname"

ansible multi -a "hostname" -f 1
ansible multi -a "hostname" -f 1
ansible multi -a "hostname" -f 1

环境状况检查

ansible multi -a "df -h"

ansible multi -a "free -m"

ansible multi -a "date"

使用 Ansible 模块做变更

ansible multi -b -m yum -a "name=chrony state=present"

ansible multi -b -m service -a "name=chronyd state=started enabled=yes"

ansible multi -b -a "chronyc tracking"

ansible multi -a "date"

配置服务器组和单一服务器

配置应用服务器：安装 python3 和 django

ansible app -b -m yum -a "name=python3-pip state=present"

ansible app -b -m pip -a "name=django<4 state=present"


ansible app -a "python -m django --version"

配置数据库服务器

ansible db -b -m yum -a "name=mariadb-server state=present"

ansible db -b -m service -a "name=mariadb state=started enabled=yes"

ansible db -b -m yum -a "name=firewalld state=present"

ansible db -b -m service -a "name=firewalld state=started enabled=yes"

ansible db -b -m firewalld -a "zone=database state=present permanent=yes"

ansible db -b -m firewalld -a "source=192.168.60.0/24 zone=database state=enabled permanent=yes"

ansible db -b -m firewalld -a "port=3306/tcp zone=database state=enabled permanent=yes"

ansible db -b -m yum -a "name=python3-PyMySQL state=present"

ansible db -b -m mysql_user -a "name=django host=% password=12345 priv=*.*:ALL state=present"

对服务器组中的某一个执行命令

ansible app -b -a "service chronyd restart" --limit "192.168.60.4"

用星号匹配

ansible app -b -a "service ntpd restart" --limit "*.4"

用正则表达式匹配

ansible app -b -a "service ntpd restart" --limit ~".*\.4"


管理用户和组

ansible app -b -m group -a "name=admin state=present"

ansible app -b -m user -a "name=johndoe group=admin createhome=yes"

ansible app -b -m user -a "name=johndoe state=absent remove=yes"

管理软件包

ansible app -b -m package -a "name=git state=present"

管理文件和目录

查看文件属性

ansible multi -m stat -a "path=/etc/environment"

从本地复制文件到服务器

ansible multi -m copy -a "src=/etc/hosts dest=/tmp/hosts"

从服务器上下载文件

ansible multi -b -m fetch -a "src=/etc/hosts dest=/tmp"

创建目录和文件

ansible multi -m file -a "dest=/tmp/test mode=644 state=directory"

ansible multi -m file -a "src=/src/file dest=/dest/symlink state=link"

删除目录和文件

ansible multi -m file -a "dest=/tmp/test state=absent"

用异步作业异步的更新服务器

ansible multi -b -B 3600 -P 0 -a "yum -y update"

ansible multi -b -m async_status -a "jid=169825235950.3572"

查看日志的方法

ansible multi -b -a "tail /var/log/messages"

ansible multi -b -m shell -a "tail /var/log/messages | \
grep ansible-command | wc -l"

管理 cron 作业

ansible multi -b -m cron -a "name='daily-cron-all-servers' \
hour=4 job='/path/to/daily-script.sh'"

ansible multi -b -m cron -a "name='daily-cron-all-servers' \
state=absent"

部署版本控制的应用

ansible app -b -m git -a "repo=git://example.com/path/to/repo.git \
dest=/opt/myapp update=yes version=1.2.4"

ansible app -b -a "/opt/myapp/update.sh"

Ansible 的 SSH 连接历史





## 关于作者

这个项目是由[Jeff Geerling](https://www.jeffgeerling.com/)创建的，作为[Ansible for DevOps](https://www.ansiblefordevops.com/)的一个例子。