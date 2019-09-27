# How to install k8s in the mainland and setup a 3 nodes cluster

## k8s install

NOTE: our OS is CentOS 7.6 x64

### YUM mirror

For OS Base, please follow this [description](https://mirror.tuna.tsinghua.edu.cn/help/centos/).

For Docker-CE, please follow this [description](https://mirror.tuna.tsinghua.edu.cn/help/docker-ce/).

For k8s, we are using Ali's mirror.

```text
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
        http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

### update YUM cache

```bash
# yum clean all  
# yum makecache  
# yum repolist
```

### firewalld

```bash
firewall-cmd --add-port=6443/tcp --permanent
firewall-cmd --add-port=10250/tcp --permanent
firewall-cmd --reload
```

The firewall will be enable by the default settings, even under the mini install.
So, you could just disable it, or add some exception law.
If you forget to do any of them, you could got some warning when trying to init a k8s cluster.
Please do any of them, before running "kubeadm join".

### hostname and DNS

Add the IP-hostName pairs of your cluster nodes in /etc/hosts file.
Or add NDS A/AAAA record in your DNS server.

B.T.W, For a DNS server, [sameersbn-bind](https://hub.docker.com/r/sameersbn/bind/) is a good choice. Could be deploied in docker, and has a web GUI for config.

```text
### /etc/hosts

127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

k8s01 10.10.11.7
k8s02 10.10.11.8
k8s02 10.10.11.9

```

### sysctl.d

Add some config in sysctl.d.

```bash
[root@k8s01 ~]# cat /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
[root@k8s01 ~]# sysctl -p /etc/sysctl.d/k8s.conf
```

### disable swap

```bash
[root@k8s01 ~]# swapoff -a
[root@k8s01 ~]# vi /etc/fstab # remove the swap line
```

### reboot

### install docker-ce

List all available docker-ce version.

```bash
# yum list docker-ce --showduplicates | sort -r
```

Install by yum.

```bash
# yum install -y docker-ce-18.09.8 # for k8s 1.15.3
```

We are using k8s 1.15.3 for now, and the latest supported version of docker-ce is 18.09.8.

### docker daemon config

1. As k8s's repust, the native.cgroupdriver should be "systemd".
2. For more faster speed of image downloading, add some image mirrors registry.
3. For the private insecure registry (Harbor), add a insecure-registries.

```bash
[root@k8s01 k8s_practice]# cat /etc/docker/daemon.json
{
  "registry-mirrors" : [
    "https://dockerhub.azk8s.cn",
    "https://registry.docker-cn.com"
  ],
  "insecure-registries" : [
    "csmssim01:80"
  ],
  "exec-opts": ["native.cgroupdriver=systemd"]
}
[root@k8s01 k8s_practice]#
```

### install k8s

```bash
# yum install -y kubelet kubeadm kubectl
```

### setup autostart and stop

```bash
# systemctl enable --now docker 
# systemctl enable --now kubelet
```

### reboot again

NOTE: If you are using some virtual machines for trying, you can clone two more nodes here. But please DO modify the host-name and IP address.
