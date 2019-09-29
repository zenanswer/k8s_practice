# ceph

Using "ceph-deploy" to deploy a sample DFS(distributed file system).

https://docs.ceph.com/docs/master/start/
https://docs.ceph.com/docs/master/start/quick-start-preflight/
https://docs.ceph.com/docs/master/start/quick-ceph-deploy/
https://docs.ceph.com/docs/master/cephfs/
https://docs.ceph.com/docs/master/start/quick-cephfs/

# A quick how to

Here is a quick "how to" for setting up a sample DFS on Centos7.6 x64.

# Architecture

One admin node
Two or three worker node

![](https://docs.ceph.com/docs/master/_images/ditaa-fc1f16db6801b202a4cc57c98bc87943031d9f80.png)

# Preflight

https://docs.ceph.com/docs/master/start/quick-start-preflight/

## YUM mirrors

For CentOS Base and EPEL

https://mirrors.tuna.tsinghua.edu.cn/help/centos/

https://mirrors.tuna.tsinghua.edu.cn/help/epel/

For ceph

```bash
cat << EOM > /etc/yum.repos.d/ceph.repo
[ceph-noarch]
name=Ceph noarch packages
baseurl=https://mirrors.tuna.tsinghua.edu.cn/ceph/rpm-nautilus/el7/noarch
enabled=1
gpgcheck=1
type=rpm-md
gpgkey=https://mirrors.tuna.tsinghua.edu.cn/ceph/keys/release.asc
EOM
bash
```

## Install ceph-deploy, python2-pip and ntpd

```bash
sudo yum update
sudo yum install -y ceph-deploy
sudo yum -y install python2-pip

sudo yum install ntp ntpdate ntp-doc
sudo systemctl enable ntpd --now
```

## An ceph user with sudo permission

```bash
groupadd cephtest
useradd -g cephtest cephtest
echo cephtest | passwd --stdin cephtest

echo "cephtest ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/cephtest
sudo chmod 0440 /etc/sudoers.d/cephtest
```

## firewall

```bash
sudo firewall-cmd --zone=public --add-service=ceph-mon --permanent
sudo firewall-cmd --zone=public --add-service=ceph --permanent
sudo firewall-cmd --reload
```

## SELinux

```bash
# Set SELinux in permissive mode
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

## hosts

```bash
[cephtest@cephadmin ~]# sudo cat /etc/hosts

10.10.11.7 node1
10.10.11.8 node2
10.10.11.9 node3
[cephtest@cephadmin ~]#
```

## reboot

NOTE: Clone this node into 2 or 3 replicas, and please DO modify the hostname and ip addresses.

# Set up cluster on the admin node

https://docs.ceph.com/docs/master/start/quick-ceph-deploy/

## SSH login without username and password

Login with cephtest account.

```bash
ssh-keygen
```

```bash
ssh-copy-id cephtest@node1
ssh-copy-id cephtest@node2
ssh-copy-id cephtest@node3
```

```bash
[cephtest@cephadmin ~]$ cat ~/.ssh/config
Host node1
   Hostname node1
   User cephtest
Host node2
   Hostname node2
   User cephtest
Host node3
   Hostname node3
   User cephtest
[cephtest@cephadmin ~]$ 
[cephtest@cephadmin ~]$ chmod 600 ~/.ssh/config
```

## Create a empty folder for storing the config files

```bash
[cephtest@cephadmin]$ mkdir my-cluster
[cephtest@cephadmin]$ cd my-cluster
```

## Set up cluster

```bash
[cephtest@cephadmin my-cluster]$ ceph-deploy new node1
[cephtest@cephadmin my-cluster]$ 
[cephtest@cephadmin my-cluster]$ ceph-deploy install \
--repo-url https://mirror.tuna.tsinghua.edu.cn/ceph/rpm-nautilus/el7/ \
--gpg-url https://mirror.tuna.tsinghua.edu.cn/ceph/keys/jessie-stable-release.asc \
node1 node2 node3
[cephtest@cephadmin my-cluster]$ 
[cephtest@cephadmin my-cluster]$ ceph-deploy mon create-initial
[cephtest@cephadmin my-cluster]$ 
[cephtest@cephadmin my-cluster]$ ceph-deploy admin node1 node2 node3
[cephtest@cephadmin my-cluster]$ 
[cephtest@cephadmin my-cluster]$ ceph-deploy mgr create node1
```

NOTE: "rpm-nautilus", "rpm" is for RedHat or CentOS, "nautilus" is ceph v14.2.4.

## Check your cluster’s health

```bash
[cephtest@cephadmin my-cluster]$ ssh node1 sudo ceph health
```

```bash
[cephtest@cephadmin my-cluster]$ ssh node1 sudo ceph -s
```

# cephfs

https://docs.ceph.com/docs/master/cephfs/
https://docs.ceph.com/docs/master/start/quick-cephfs/

## create a cephfs

```bash
sudo ceph-deploy mds create node1 node2 node3

sudo ceph osd pool create test_fs_data 32
sudo ceph osd pool create test_fs_metadata 32

sudo ceph fs new cephfs_test test_fs_metadata test_fs_data

```

### Mount it

Mount this "cephfs_test" on the admin node with admin.keyring

Get admin.keyring:

```bash
[cephtest@cephadmin my-cluster]$ cat ceph.client.admin.keyring
[client.admin]
	key = AQCFn4ldc/vDKxAAhrPbdq/vPADkO/0zE3DOqg==
	caps mds = "allow *"
	caps mgr = "allow *"
	caps mon = "allow *"
	caps osd = "allow *"
[cephtest@cephadmin my-cluster]$
```

OR

```bash
[cephtest@cephadmin my-cluster]$ sudo ceph auth get-key client.admin
```

Mount

```bash
[cephtest@cephadmin my-cluster]$ sudo mkdir /mnt/mycephfs
[cephtest@cephadmin my-cluster]$ sudo mount -t ceph 10.10.11.7:6789,10.10.11.8:6789,10.10.11.9:6789:/ /mnt/mycephfs -o name=admin,secret=AQCFn4ldc/vDKxAAhrPbdq/vPADkO/0zE3DOqg==
```

