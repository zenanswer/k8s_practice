# How to setup metallb

## metallb

https://github.com/danderson/metallb

A network load-balancer implementation for Kubernetes using standard routing protocols.

We can find lots mature and powerful load-balancer implementation on those Public Cloud Provider, but for trying or in-lab development, metallb is a good choice.

## how to

```bash
[root@k8s01 ~]#  curl https://raw.githubusercontent.com/google/metallb/v0.8.1/manifests/metallb.yaml -o metallb.v0.8.1.yaml
[root@k8s01 ~]#  kubectl apply -f metallb.v0.8.1.yaml
```

## status check of metallb-system

```bash
[root@k8s01 ~]# watch kubectl -n metallb-system get pods
NAME                        READY   STATUS    RESTARTS   AGE
controller-55d74449-cwzq9   1/1     Running   0          74s
speaker-8nxdr               1/1     Running   0          74s
speaker-vfxrk               1/1     Running   0          74s

```

## adding a address-pool for EXTERNAL-IP

```bash
[root@k8s01 ~]# kubectl apply -f metallb-layer2-config.yaml
configmap/config created
```

You can find the sample file "metallb-layer2-config.yaml" in this repo.

Please modify the "addresses" range for adapting your network environment.

It is in the same subnet of nodes' host addresses, in my sample.

## two Nginx testing

```bash
[root@k8s01 ~]# kubectl create deployment nginx --image=nginx:alpine
[root@k8s01 ~]# kubectl scale deployment nginx --replicas=2
[root@k8s01 ~]# kubectl get pods -l app=nginx -o wide
NAME                   READY   STATUS    RESTARTS   AGE   IP                NODE    NOMINATED NODE   READINESS GATES
nginx-8f6959bd-28sb7   1/1     Running   0          40s   192.168.73.65     k8s01   <none>           <none>
nginx-8f6959bd-s9lhb   1/1     Running   0          50s   192.168.236.132   k8s02   <none>           <none>
[root@k8s01 ~]#
```

```bash
[root@k8s01 ~]# kubectl expose deployment nginx --port=80 --type=LoadBalancer
[root@k8s01 ~]# kubectl get services
NAME            TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes      ClusterIP      10.96.0.1        <none>        443/TCP        70m
nginx           LoadBalancer   10.100.78.106    10.10.11.70   80:30889/TCP   43m
[root@k8s01 ~]# curl 10.10.11.70:80
```

