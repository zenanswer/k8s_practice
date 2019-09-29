# ceph as k8s's storage

# cephfs

## PV and PVC (without StorageClass)

### Create a folder on the test fs for k8s mount

```bash
sudo mkdir /mnt/mycephfs/data01
```

### k8s ceph-admin-secret and PV

#### ceph-admin-secret

```bash
sudo ceph auth get-key client.admin |base64
```

```bash
[root@k8s01 k8s]# cat ceph-admin-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: ceph-admin-secret
  namespace: tag
data:
  key: QVFDRm40bGRjL3ZES3hBQWhyUGJkcS92UEFEa08vMHpFM0RPcWc9PQ==
type:
  kubernetes.io/rbd

[root@k8s01 k8s]# kubectl apply -f ceph-admin-secret.yaml
[root@k8s01 k8s]# 
```

#### PV

```bash
[root@k8s01 k8s]# cat cephfs-pv-01.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: cephfs-pv-01
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  cephfs:
    monitors:
      - 10.10.11.7:6789
      - 10.10.11.8:6789
      - 10.10.11.9:6789
    path: /data01
    user: admin
    secretRef:
      name: ceph-admin-secret
    readOnly: false
  persistentVolumeReclaimPolicy: Recycle

[root@k8s01 k8s]#
[root@k8s01 k8s]# kubectl apply -f cephfs-pv-01.yaml
[root@k8s01 k8s]#
```

### volumeClaimTemplates sample

```bash
[root@k8s01 k8s]# vi console/jhipster-elasticsearch.yml
…
---
apiVersion: apps/v1
kind: StatefulSet
  volumeClaimTemplates:
    - metadata:
        name: storage
      spec:
        accessModes: ['ReadWriteOnce']
        resources:
          requests:
            storage: 1Gi
---
[root@k8s01 k8s]# 
```

```bash
[root@k8s01 cephfs]# kubectl get pv --all-namespaces
NAME           CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                                         STORAGECLASS   REASON   AGE
cephfs-pv-01   2Gi        RWO            Recycle          Bound       tag/storage-jhipster-elasticsearch-master-0                           4d2h
cephfs-pv-02   2Gi        RWO            Recycle          Bound       tag/storage-jhipster-elasticsearch-data-0                             4d2h
cephfs-pv-99   2Gi        RWO            Recycle          Available                                                                         90s
[root@k8s01 cephfs]#
```
