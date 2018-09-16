# 使用Kubespray部署Kubernetes服务器集群

Kubespray是Google开源的一个部署生产级别的Kubernetes服务器集群的开源项目，它整合了Ansible作为部署的工具。项目地址：https://github.com/kubernetes-incubator/kubespray

## 部署历程

目前为止，对于Kubernetes集群的部署，我只谈的上是一个入门者，涉及到了众多的运维知识，对于一个开发来说，确实挺难的。万事开头难，好事多磨，经过一个多星期的反复尝试，终于搭建好了。对比市面上的部署方式，主流的有三种方式。一是完全手动部署，非常的繁琐，容易部署。二是采用kubeAdmin开源项目进行部署，这个也是谷歌官方开源的一个项目。三是，采用kubeSpray进行部署。我的理念是有好的工具当然是用好工具，所以手动部署是不可能的，完全排除，所以Kubeadmin和KubeSpray。而我对Ansibe这个运维组件兴趣非常的大，所以我最终选择了KubeSpray进行了部署。

部署的工程是非常艰难的，在我决定搞Kubernetes之时，为了学习不难么枯燥和孤独，我专门组建了一个群，找了一些朋友一起来学习和交流，采用的方式是大家一起学习，一起写文档，一起交流，另外有主机的出主机。所以，一开始的主机是几个朋友自己的主机，不在一个局域网内，计算机操作系统也不太一样，这为后面的部署带来了一个大坑。另外由于国内的屏蔽了谷歌的网络，导致谷歌的相关镜像下载不下来，这也是一个坑。

坑点1，不在一个局域网不能部署Kubernetes？我专门打电话问了阿里云，客户说不可以，是不是真的不可以，我是不确定的。另外集群的型号不同和操作系统不同也会导致失败。

坑点2，长城屏蔽了谷歌的镜像，所以我刚开始是根据谷歌的镜像在阿里云镜像仓库一顿搜索，导致Kubernetes各个版本组件不兼容，出现了问题。

现在我也这篇文章来详细讲解我的部署过程，供其他人参考，如果有其他人想加入我们的Kubernetes兴趣群，加我微信miles02和我联系。

## 主机相关

主机需要在同一局域网内？所以我们重新租了三台机器，进行了操作。现在列举主机相关的信息如下：


|主机| 系统版本 | 配置 | ip |
| ------ | ------ | ------ | ------ |
|Ansible| CentOS  7.2 | 1核1G | 172.31.84.154 |
| Mater、Node | CentOS  7.2 | 2核2G | 172.31.84.155|
| Node | CentOS  7.2 | 2核2G | 172.31.84.156|

Ansible那台主机使用KubeSpray进行部署，这台机器不做Kubernetes相关集群的部署。另外2台机器，一台既作为Master，又作为Node，另外一台是一个Node。

本次部署，使用的KubeSpray版本为v2.1.2。

## Master、Node节点的操作

因为本次使用KubeSpray操作部署，所以所有的主机都需要关闭防火墙等相关的操作。

所以的主机都需要关闭selinux，执行的命令如下：

```
setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux

```

防火墙和网络设置，所有的主机都执行以下命令：

```
systemctl stop firewalld
modprobe br_netfilter
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
sysctl -w net.ipv4.ip_forward=1

```

这样与Kubernetes集群相关的集群设置就完毕了。

## Ansibe主机操作

Ansibe主机也需要关闭selinux和关闭防火墙以及网络设置，同上面。

### 在Ansible主机上设置免密码操作其它主机

首先生成ssh公钥和私钥。

```
ssh-keygen

```
按三次回车。

建立ssh通道，

```
ssh-copy-id root@172.31.84.155 将秘钥分发给master主机。
ssh-copy-id root@172.31.84.156

```

### 安装Ansible


安装ansible和jinja2，安装命令如下。

```

sudo yum install epel-release
sudo yum install ansible

easy_install pip
pip2 install jinja2 --upgrade


```

如果执行 pip2 install jinja2--upgrade 提示升级，则升级，再执行一次命令。


### 安装python36

```
sudo yum install python36 -y 

```

### 在Ansible集群上安装KubeSpray


在ansible机器上下载KubeSpray代码，并解压，执行如下的命令：

```
wget https://github.com/kubernetes-incubator/kubespray/archive/v2.1.2.tar.gz
tar -zxvf v2.1.2.tar.gz
mv kubespray-2.1.2 kuberspray

```

## 安装KubeSpray所需的包

执行如下命令：

```
cd kubespray
pip install r requirements.txt
```

## 定义集群

执行以下的命令。

```
IP=(
172.31.84.155
172.31.84.156
)

CONFIG_FILE=./kubespray/inventory/inventory.cfg python36 ./kubespray/contrib/inventory_builder/inventory.py ${IP[*]
```
vim ~./kubespray/inventory/inventory.cfg

```
[all]
node1    ansible_host=172.31.84.156 ip=172.31.84.156
node2    ansible_host=172.31.84.155 ip=172.31.84.155

[kube-master]
node1

[kube-node]
node1
node2

[etcd]
node1

[k8s-cluster:children]
kube-node
kube-master

[calico-rr]

[vault]
node1
```

### 替换镜像

在kuberspay源码源代码中搜索包含 gcr.io/google_containers 和 quay.io 镜像的文件，并替换为我们之前已经上传到阿里云的进行，替换脚步如下：

```
grc_image_files=(
./kubespray/extra_playbooks/roles/dnsmasq/templates/dnsmasq-autoscaler.yml
./kubespray/extra_playbooks/roles/download/defaults/main.yml
./kubespray/extra_playbooks/roles/kubernetes-apps/ansible/defaults/main.yml
./kubespray/roles/download/defaults/main.yml
./kubespray/roles/dnsmasq/templates/dnsmasq-autoscaler.yml
./kubespray/roles/kubernetes-apps/ansible/defaults/main.yml
)

for file in ${grc_image_files[@]} ; do
    sed -i 's/gcr.io\/google_containers/registry.cn-hangzhou.aliyuncs.com\/szss_k8s/g' $file
done

quay_image_files=(
./kubespray/extra_playbooks/roles/download/defaults/main.yml
./kubespray/roles/download/defaults/main.yml
)

for file in ${quay_image_files[@]} ; do
    sed -i 's/quay.io\/coreos\//registry.cn-hangzhou.aliyuncs.com\/szss_quay_io\/coreos-/g' $file
    sed -i 's/quay.io\/calico\//registry.cn-hangzhou.aliyuncs.com\/szss_quay_io\/calico-/g' $file
    sed -i 's/quay.io\/l23network\//registry.cn-hangzhou.aliyuncs.com\/szss_quay_io\/l23network-/g' $file
done

```

### 使用ansible playbook部署Kubernetes集群

以上全部完成，执行安装操作：


```
cd kubespray
ansible-playbook -i inventory/inventory.cfg cluster.yml -b -v --private-key=~/.ssh/id_rsa

```
大约过了10分钟，如果顺利的话，集群会成功搭建。

### 验证几点是否成功

登录Kubernete集群的Mater集群，执行如下命令：

```
kubectl get no

```
控制台打印出了正确的Kubernetes节点信息，则安装成功。

### 增加节点

```
cd kubespray
ansible-playbook -i inventory/inventory.cfg cluster.yml -b -v --private-key=~/.ssh/id_rsa --limit node3

```

## 遇到问题卸载

ansible执行卸载操作：

```

ansible-playbook -i inventory/mycluster/hosts.ini reset.yml

```

安装失败清理Kubernetes机器

```

rm -rf /etc/kubernetes/
rm -rf /var/lib/kubelet
rm -rf /var/lib/etcd
rm -rf /usr/local/bin/kubectl
rm -rf /etc/systemd/system/calico-node.service
rm -rf /etc/systemd/system/kubelet.service
systemctl stop etcd.service
systemctl disable etcd.service
systemctl stop calico-node.service
systemctl disable calico-node.service
docker stop $(docker ps -q)
docker rm $(docker ps -a -q)
service docker restart

```

## 参考资料

参考了一下的文章：

https://github.com/kubernetes-incubator/kubespray

https://mp.weixin.qq.com/s/-SXuXhY7KIFl1zYvVT93ZA

https://blog.csdn.net/zhuchuangang/article/details/77712614
