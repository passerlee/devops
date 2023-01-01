> 环境初始化

> 虚拟机环境准备

:::info
主机名IP地址配置
:::
master  192.168.238.100  2cpu  2g<br />node1  192.168.238.101  2cpu  2g<br />node2  192.168.238.102  2cpu  2g
:::info
三台机器都要做：
:::
```
 192.168.238.100  master
 192.168.238.101  node1
 192.168.238.102  node1
```
```
systemctl enable chronyd --now
```
```
systemctl stop firewalld.service
systemctl disabled firewalld.service
setenforce 0
vim /etc/selinux.conf
```
![image.png](https://cdn.nlark.com/yuque/0/2022/png/34381447/1669127632366-ff3fc041-d0a4-43fb-8f23-08ce2542ca26.png#averageHue=%231f1e1e&clientId=ud96bd418-80c4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=197&id=uc07b86cc&margin=%5Bobject%20Object%5D&name=image.png&originHeight=246&originWidth=888&originalType=binary&ratio=1&rotation=0&showTitle=false&size=24155&status=done&style=none&taskId=u421db5cb-fc16-416f-a3c8-207c45e1c17&title=&width=710.4)
```
vim /etc/fstab   #注释即可
```
![image.png](https://cdn.nlark.com/yuque/0/2022/png/34381447/1669127725866-ce058275-f5eb-42ee-87c6-41c7c9241835.png#averageHue=%231e1e1e&clientId=ud96bd418-80c4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=33&id=u7fd8cb88&margin=%5Bobject%20Object%5D&name=image.png&originHeight=41&originWidth=808&originalType=binary&ratio=1&rotation=0&showTitle=false&size=2903&status=done&style=none&taskId=ucd672a69-fa64-4bbb-b0fb-8869b73a204&title=&width=646.4)
```
vim /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
#重新加载配置
sysctl -p
#加载过滤模块
modprobe br_netfilter
#查看网桥过滤模块是否加载成
lsmod | grep br_netfilter
```
```
#安装ipset和ipvsadm
yum -y install ipset ipvsadm
#添加需要加载的模块并写入脚本
cat <<EOF> /etc/sysconfig/modules/ipvs.modules 
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF
#给脚本添加权限执行脚本并查看是否加载成
chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep -e ip_vs -e nf_conntrack_ipv4
#重启服务器
reboot
```
```
#安装依赖包
yum install -y yum-utils device-mapper-persistent-data lvm2
#安装docker
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
#查看可安装的版本
yum list docker-ce --showduplicates
#指定版本安装
yum -y install --setopt=obosletes=0 docker-ce-18.06.3.ce-3.el7
```
```
mkdir /etc/docker
vim /etc/docker/daemon.json
#用到的阿里源加速器可自行在aliyun开源镜像站里找
{
 "exec-opts": ["native.cgroupdrive=systemd"],
 "registry-mirrors": ["https://lillrb1m.mirror.aliyuncs.com"]
}
#启动docker并设置开机自启
systemctl enable docker --now 
```
```
#切换成国内源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
yum -y install --nogpgcheck --setopt=obsoletes=0 kubeadm-1.17.4-0  kubelet-1.17.4-0 kubectl-1.17.4-0
```
```
vim /etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS=
KUBELET_CGROUP_ARGS="--cgroup-driver=systemd"
KUBE_PROXY_MODE="ipvs"
#加入开机自启
systemctl enable kubelet.service
```
```
#查看镜像列表
kubeadm config images list
#由于网络原因，使用以下方法下载镜像
images=(
kube-apiserver:v1.17.4
kube-controller-manager:v1.17.4
kube-scheduler:v1.17.4
kube-proxy:v1.17.4
pause:3.1
etcd:3.4.3-0
coredns:1.6.5
)
for imageName in ${images[@]} ; do
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName   k8s.gcr.io/$imageName
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
done
```
:::info
以下操作只需要在master上操作
:::
```
kubeadm init --apiserver-advertise-address=192.168.238.100   --kubernetes-version=v1.17.4 --service-cidr=10.96.0.0/12 --pod-network-cidr=10.244.0.0/16
#初始化看见successfully后在下面命令中找
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
#查看集群结点状态（此时只有master）
kubectl get nodes
```
:::info
在node结点操作
:::
```
#初始化看见successfully后在下面命令中找（每个集群会有不同）
kubeadm join 192.168.238.100:6443 --token fqb3tx.uoi7ec80anlcz420    --discovery-token-ca-cert-hash sha256:41ef438fd19d4722da9d4164f0d715cab5773c226f1d866bfcb90295e773031 
#查看集群结点状态（此时有master和node且status为notready）
kubectl get nodes
```
:::info
只在master结点操作
:::
```
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubectl apply -f kube-flannel.yml
#查看集群结点状态（此时有master和node且status为ready）
kubectl get nodes
```
![image.png](https://cdn.nlark.com/yuque/0/2022/png/34381447/1669131227499-0be69409-2dbe-432f-9088-fcaac65ed1a9.png#averageHue=%232e2b28&clientId=ud96bd418-80c4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=88&id=uf9eec9fc&margin=%5Bobject%20Object%5D&name=image.png&originHeight=110&originWidth=443&originalType=binary&ratio=1&rotation=0&showTitle=false&size=9333&status=done&style=none&taskId=u7acd3bdd-3c7c-432c-a643-5bb9733ed7b&title=&width=354.4)
:::info
到此整个集群搭建就结束啦
:::

