# k8s init a cluster

## generate a default k8s init config file

```bash
# kubeadm config print init-defaults > k8s-init.yaml
```

## modify init file and apply it

```bash
[root@k8s01 ~]# diff k8s-init.yaml k8s-init-xc.yaml
12c12
<   advertiseAddress: 1.2.3.4
---
>   advertiseAddress: 10.10.11.57 # node1 IP
32c32
< imageRepository: k8s.gcr.io
---
> imageRepository: registry.aliyuncs.com/google_containers #using ali source
37a38
>   podSubnet: 192.168.0.0/16  #same value in calico.v3.9.yaml
[root@k8s01 ~]#
[root@k8s01 ~]# kubeadm config --config k8s-init-xc.yaml images pull
[root@k8s01 ~]# kubeadm init --config k8s-init-xc.yaml
```

1. imageRepository: pull k8s image fro Ali mirror.
2. images pull: Before calling "init", we call "images pull" here. If pulling is two slow, we could change the imageRepository and try again.
3. podSubnet: For enable k8s cluster IP, we have to choose one of those [Cluster Networking Implementation](https://kubernetes.io/docs/concepts/cluster-administration/networking/), Calico and Flannel are easy to setup. We are suing Calico here, and you can find the how to in another README file. But in this init file for now, podSubnet should be same with the value in Calico's yaml.

## node join

```text
[root@k8s01 ~]# kubeadm init --config k8s-init-xc.yaml
…
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.10.11.7:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:50c95d9bdeed106538fd1acdb374af15076421c49b882889f7c66a64c7fca949
[root@k8s01 ~]#
```

If everything works, your 1st node - the manager node shoule be settup now.

1. copy the cluster config to a regular user, so this user could run kubectl command.
2. join the (working) node into this cluster by calling "kubeadm join ***" command on them.

NOTE: Please DO go throug the output info of "kubeadm init ***", and try to fix the "warning" and "error".

## get node status

```bash
[root@k8s01 ~]# kubectl get nodes
NAME    STATUS     ROLES    AGE   VERSION
k8s01   NotReady   master   13m   v1.15.3
k8s02   NotReady   <none>   11s   v1.15.3
[root@k8s01 ~]#
```

NOTE: could got more info with "-o wide" parameter.
