# 构建openshift集群

## virtualbox安装

```shell
# 安装virtualbox
$ sudo apt install virtualbox

# 启动virtualbox
$ virtualbox
```

## 集群环境搭建

### 操作系统镜像

镜像采用CentOS-7-x86_64-Minimal-1804.iso

### 网络设置

使用两个网卡，采用两种网络格式

* 网络地址转换(NAT):用于主机端口映射连接虚拟机

* 桥接网卡：用于虚拟机集群互连

### 构建集群

基于master节点，采用复制方式构建node1和node2节点，需要注意的是复制是需要选择刷新MAC，否则虚拟机内部网络地址不会发生变化

## 免密登录

```shell
# 修改hostname,分别修改为master,node1,node2,重启生效
$ vim /etc/hostname

# 修改hosts 分别将master,node1,node2加入hosts
$ vim /etc/hosts

# 生成ssh密钥对
$ ssh-keygen -t rsa

# 将公钥拷贝到authorized_keys文件
$ cat id_rsa.pub >> authorized_keys

# 登录其他服务器，将公钥拷贝到master中
$ ssh-copy-id -i master

# 将 master 服务器的authorized_keys分发到其他服务器
$ scp /root/.ssh/authorized_keys node1:/root/.ssh/
```

## ansible安装

```shell
# 在 master 服务器上安装 ansible
$ sudo yum install ansible

# 配置受控主机
# 将受控主机地址加入/etc/ansible/hosts文件

# 验证
$ ansible all -m ping -u root # -u 指定用户
```