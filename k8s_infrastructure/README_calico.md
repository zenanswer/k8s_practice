# How to using calico

## firewall

```bash
# Calico networking (BGP)
firewall-cmd --zone=public --add-port=179/tcp --permanent
# Calico networking with Typha enabled
# firewall-cmd --zone=public --add-port=5473/tcp --permanent
# flannel networking (VXLAN)
# firewall-cmd --zone=public --add-port=4789/udp --permanent
firewall-cmd --reload
firewall-cmd --list-ports

```

calico support some protocol, and they are using diff port for communicating to each other. So, we have to add some exception here.

By default, we are using BGP.

## Get yaml and apply it

```bash
[root@k8s01 ~]# curl https://docs.projectcalico.org/v3.9/manifests/calico.yaml -o calico.v3.9.yaml
[root@k8s01 ~]# kubectl apply -f  calico.v3.9.yaml
```

If everything works, "calico-kube-controllers", "calico-node" and "coredns" should be READY and running for a little while.

```bash
[root@k8s01 ~]# watch kubectl get pods --all-namespaces

Every 2.0s: kubectl get pods --all-namespaces                                                                            Tue Sep 17 10:27:00 2019

NAMESPACE     NAME                                      READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-f78989564-wn5s2   1/1     Running   0          107s
kube-system   calico-node-22nhj                         1/1     Running   0          107s
kube-system   calico-node-8ftkg                         1/1     Running   0          107s
kube-system   coredns-bccdc95cf-mqr8v                   1/1     Running   0          3m34s
kube-system   coredns-bccdc95cf-q7f7h                   1/1     Running   0          3m34s
kube-system   etcd-k8s01                                1/1     Running   0          2m45s
kube-system   kube-apiserver-k8s01                      1/1     Running   0          2m44s
kube-system   kube-controller-manager-k8s01             1/1     Running   0          2m28s
kube-system   kube-proxy-mbdll                          1/1     Running   0          3m2s
kube-system   kube-proxy-nfpxn                          1/1     Running   0          3m34s
kube-system   kube-scheduler-k8s01                      1/1     Running   0          2m26s

```

```bash
[root@k8s01 k8s_practice]# kubectl -n kube-system get svc
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   7d7h
[root@k8s01 k8s_practice]# kubectl -n kube-system get svc -o wide
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE    SELECTOR
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   7d7h   k8s-app=kube-dns
[root@k8s01 k8s_practice]#
```
