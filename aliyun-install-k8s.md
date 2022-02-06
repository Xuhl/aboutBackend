##  资源准备
1. 三台服务器： master 1 台，2 台 work node

## 安装步骤
1. 添加`yum` 源
```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```
2. 安装`kubelet`,`kubeadm`,`kubectl`
```
#安装
yum install -y kubelet kubeadm kubectl
# 设置kubelet开机启动
systemctl enable kubelet && systemctl start kubelet
```
