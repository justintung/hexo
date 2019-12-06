title: 国内源安装kubernetes的工具
date: 2018-11-17 12:20:36
tags:
- kubernetes k8s
categories:
- 安装
---
一、debian安装kubectl、kubelet、kubeadm

```shell
$ apt-get update && apt-get install -y apt-transport-https
$ curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 
$ cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF  
$ apt-get update
$ apt-get install -y kubelet kubeadm kubectl
# 安装指定版本：
$ apt-get install kubeadm=1.10.2-00 kubectl=1.10.2-00 kubelet=1.10.2-00
```

centos安装kubectl、kubelet、kubeadm

```shell
$cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
$yum install -y kubelet kubeadm kubectl
$systemctl enable kubelet && systemctl start kubelet
```



二、minikube k8s单机测试

1.安装minikubi

```shell
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube
sudo mkdir -p /usr/local/bin/
sudo install minikube /usr/local/bin/  
```

2.minikube 使用

创建 minikube.sh,创建minikube cluster

```shell
#!/bin/bash
sudo minikube start --vm-driver=none --registry-mirror=https://registry.docker-cn.com --registry-mirror=https://8eoqixdq.mirror.aliyuncs.com --registry-mirror=https://dockerhub.azk8s.cn --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers  --memory=4096
```

3.创建deployment

```shell
kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.10
```

4.创建services

```shell
kubectl expose deployment hello-minikube --type=NodePort --port=8080
#查看pod状态
kubectl get pod
#查看service信息
minikube service hello-minikube --url
```

5.删除service

```shell
kubectl delete service hello-minikube
```

6.删除deployment

```shell
kubectl delete deployment hello-minikube
```

7.关闭minikube cluster

```shell
minikube stop
```

8.删除minikube cluster

```shell
minikube delete
```

