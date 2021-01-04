# Openvino Demo README.md

当前 demo 镜像仅集成 openvino 的环境与一个汽车牌照识别的 demo

>> 构建环境: ubuntu 18.04

>> 运行环境: ubuntu 18.04 或 centos7

## Docker Image Running

```

# pull image
$ sudo docker pull openvino-demo

# running image
$ sudo docker run -it --name=openvino-demo  --net=host
openvino-demo:latest

# running demo
# 注：下述 demo 依赖 OS GUI，无 GUI 请自行安装 GUI，GUI 安装和问题见附录 1.1
$ ./start_demo_security_barrier_camera.sh

```

## Dokerfile Build Image

```

# 依赖 docker 18+，docker 安装详情见附录 1.2
# 网络环境：能科学上网，访问 google、intel、docker-repo

# build
$ cd demo/dockerfile
$ sudo docker build -f Dockerfile -t openvino-demo:latest .

```

## Kubenerts Running Pod

```

# 依赖环境 kuberntes 1.18+，kuberntes 1.18 安装详情见附录 1.2

# Running Pod
$ cd demo/k8s-yaml
$ kubectl apply -f openvino-demo.yaml

# list pod
$ kubectl get po
NAME                            READY   STATUS    RESTARTS   AGE
openvino-demo-bbb577d54-grxvg   1/1     Running   0          52s

# into pod
$ kubectl exec -it openvino-demo-bbb577d54-grxvg /bin/bash
$ ./start_demo_security_barrier_camera.sh

```

## Helm running app

```

# 依赖环境 helm2, 若为 startlinx 环境，则暂不支持 helm3
# helm2 安装详情见附录 1.3
$ cd demo/helm
$ helm install --name=openvino-demo ./openvino-demo

# list app
$ helm list
NAME         	REVISION	UPDATED                 	STATUS  	CHART              	APP VERSION	NAMESPACE
openvino-demo	1       	Sun Jan  3 20:44:34 2021	DEPLOYED	openvino-demo-0.1.0	1.16.0     	default

$ kubectl get po
NAME                            READY   STATUS    RESTARTS   AGE
openvino-demo-bbb577d54-grxvg   1/1     Running   0          52s

```

## Armada running app

```

# 依赖环境 python3、armada
# 安装详情见附录 1.4

$ cd demo/armada/stx-openvino-demo
$ armada apply stx-openvino-demo.yaml

# list app
$ armada tiller --releases
2021-01-03 20:41:12.927 31919 INFO armada.cli [-] Tiller Service: True
2021-01-03 20:41:12.929 31919 INFO armada.cli [-] Tiller Version: v2.16.6

$ helm list
NAME         	REVISION	UPDATED                 	STATUS  	CHART              	APP VERSION	NAMESPACE
openvino-demo	1       	Sun Jan  3 20:44:34 2021	DEPLOYED	openvino-demo-0.1.0	1.16.0     	default

$ kubectl get po
NAME                            READY   STATUS    RESTARTS   AGE
openvino-demo-bbb577d54-grxvg   1/1     Running   0          52s

```

## Startlingx running application

```

# 暂无

```

# 附录

## 1.1 OS 图形化界面安装

```

# Centos7 GUI Install
$ yum groupinstall "X Window System"
$ yum groupinstall -y "GNOME Desktop"
$ startx

# Ubuntu 18.04 GUI Install
$ sudo apt-get update
$ sudo apt-get install xinit
$ sudo apt-get install gdm
$ sudo apt-get install kubuntu-desktop
$ reboot

```

## 1.2 centos7 部署 docker 和 kubernetes

> 1. 若熟悉 caas 部署，推荐 [caas4.1](http://gitlab.sh.99cloud.
net/openshift_origin/k8s-installer/tree/master) 或 [caas4.0]
(http://gitlab.sh.99cloud.net/openshift_origin/
k8s-deployment-ansible/blob/master/docs/setup/
caas4-ha-install.md) 进行 kubernetes 离线部署，无需依赖外网。

> 2. 也可采用下述的方案进行在线部署。


```

# k8s 在线部署

# 关闭防火墙
$ systemctl stop firewalld.service
$ systemctl disable firewalld.service

# 关闭swap
$ swapoff -a
$ vi /etc/fstab
# 注释掉最后一行 ‘/dev/mapper/centos-swap swap’

# 关闭selinux
$ setenforce 0
$ vi /etc/selinux/config
# 修改 SELINUX=enforcing 为
SELINUX=disabled

# 重启系统
$ reboot

# 修改主机名
$ hostnamectl set-hostname master

# 同步系统时间（单节点或者不通外网可选择不同步时间, 不影响部署）
$ yum -y install ntpdate
$ ntpdate cn.pool.ntp.org

# 安装 docker
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine

# 更新 yum
$ yum update -y
$ yum install -y yum-utils

# 设置 yum 源
$ yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 更新 yum 包索引
$ yum makecache fast

# 安装 docker-ce
$ yum install docker-ce-18.06.1.ce -y

# 配置 docker 镜像仓库
$ mkdir /etc/docker
$ vi /etc/docker/daemon.json
# 写入下述内容
{
"registry-mirrors": ["http://hub-mirror.c.163.com"]
}

# 启动 docker
$ systemctl start docker && systemctl enable docker

# 设置 k8s 阿里云的 yum 软件源
$ vi /etc/yum.repos.d/kubernetes.repo
# 写入下述内容
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg

# 打开 iptables 内生的桥接相关功能，通常系统默认开启
$ cat /proc/sys/net/bridge/bridge-nf-call-ip6tables
# 输出：1
$ cat /proc/sys/net/bridge/bridge-nf-call-iptables
# 输出： 1

# 安装 kubelet、kubeadm、kubectl
$ yum install -y kubelet-1.18.0 kubeadm-1.18.0 kubectl-1.18.0

# 启动和开机自启 kubelet
$ systemctl enable kubelet && systemctl start kubelet

# 初始化master
$ kubeadm init --kubernetes-version=1.18.0 --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.92.137 --image-repository registry.aliyuncs.com/google_containers

# 参数说明：--kubernetes-version 指定 k8 s版本，--pod-network-cidr 为 pod 的虚拟内网，--apiserver-advertise-address 这个 ip 给 master 装 apiserver（这个是master的标志），--image-repository为镜像源

# 输出 'Your Kubernetes control-plane has initialized successfully!' 则表示安装成功，安装提示执行下述命令即可

$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 安装 cni 组件，这里采用 flannel
$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# 验证
$ kubectl get po --all-namespaces

NAMESPACE     NAME                             READY   STATUS    RESTARTS   AGE
kube-system   coredns-7ff77c879f-l85mq         1/1     Running   0          15m
kube-system   coredns-7ff77c879f-s4qq2         1/1     Running   0          15m
kube-system   etcd-master                      1/1     Running   0          15m
kube-system   kube-apiserver-master            1/1     Running   0          15m
kube-system   kube-controller-manager-master   1/1     Running   0          15m
kube-system   kube-flannel-ds-s6h46            1/1     Running   0          77s
kube-system   kube-proxy-mxvlf                 1/1     Running   0          15m
kube-system   kube-scheduler-master            1/1     Running   0          15m

# 去除 master 污点
$ kubectl edit no master
# 去除 taints 下面的
- effect: NoSchedule
  key: node-role.kubernetes.io/master
# 保存退出即可


```

## 1.3 centos7 部署 helm2

```

# 准备 helm2 安装包
$ wget https://github.com/helm/helm/releases/tag/v2.16.9

$ tar -zxvf helm-v2.16.9-linux-amd64.tar.gz

$ cp linux-amd64/helm /usr/local/bin/helm

# 初始化 tiiler
$ helm init --upgrade --tiller-image registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.16.9  --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts

# 授权
$ kubectl create serviceaccount --namespace kube-system tiller
$ kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

# 为 Tiller 设置帐号
$ kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'

# 查看授权是否成功
$ kubectl get deploy --namespace kube-system   tiller-deploy  --output yaml | grep  serviceAccount
# 输出如下
    f:serviceAccount: {}
    f:serviceAccountName: {}
serviceAccount: tiller
serviceAccountName: tiller

# 查看 tiller pod 是否正常
$ kubectl get po --all-namespaces | grep tiller

kube-system   tiller-deploy-7844cb7b4d-9qh7t   1/1     Running   0          85s

```

## 1.4 安装 python3 和 armada

```

# 安装依赖
$ yum install -y openssl-devel openssl-static zlib-devel lzma tk-devel xz-devel bzip2-devel ncurses-devel gdbm-devel readline-devel sqlite-devel gcc libffi-devel

# 准备 python 安装包
$ wget https://www.python.org/ftp/python/3.7.0/Python-3.7.0.tgz

# 解压安装包
$ tar -xvf Python-3.7.0.tgz
$ mv Python-3.7.0 /usr/local
$ cd /usr/local/Python-3.7.0/

# 执行配置文件，编译安装
$ ./configure
$ make
$ make install

# 在/usr/bin路径下生成python3的软链接
$ ln -s /usr/local/Python-3.7.0/python /usr/bin/python3

# 安装 armada

# 我们通过虚拟的 python 环境进行示例，熟悉的 armada 部署后可直接环境部署在实际环境中
$ yum install python-virtualenv -y

# 下载 armada 源码

$ yum install git -y

$ git clone https://opendev.org/airship/armada.git

$ cd armada

# 创建虚拟环境

$ virtualenv -p python3 venv

$ source /root/armada/venv/bin/activate

# 编译
$ pip install --upgrade pip
$ pip install --upgrade setuptools
$ make build

# 验证
$ armada --help

Usage: armada [OPTIONS] COMMAND [ARGS]...

  Multi Helm Chart Deployment Manager

  Common actions from this point include:

  $ armada apply
  $ armada apply_chart
  $ armada delete
  $ armada rollback
  $ armada test
  $ armada tiller
  $ armada validate
  ...

```