# Ceph RBD with k8s CSI driver

https://kubernetes.io/zh/docs/concepts/storage/volumes/#csi

https://github.com/ceph/ceph-csi/tree/master/examples
https://github.com/ceph/ceph-csi/tree/master/examples/rbd

## deploy Ceph RBD CSI driver

```bash
[root@k8s01 ceph]# git clone https://github.com/ceph/ceph-csi.git
```

Modify the url of image from quay.io to quay.azk8s.cn

```bash
[root@k8s01 rbd]# grep "quay" -R ceph-csi/deploy/rbd/kubernetes/v1.14+/
ceph-csi/deploy/rbd/kubernetes/v1.14+/csi-rbdplugin.yaml:          image: quay.azk8s.cn/k8scsi/csi-node-driver-registrar:v1.1.0
ceph-csi/deploy/rbd/kubernetes/v1.14+/csi-rbdplugin.yaml:          image: quay.azk8s.cn/cephcsi/cephcsi:canary
ceph-csi/deploy/rbd/kubernetes/v1.14+/csi-rbdplugin.yaml:          image: quay.azk8s.cn/cephcsi/cephcsi:canary
ceph-csi/deploy/rbd/kubernetes/v1.14+/csi-rbdplugin-provisioner.yaml:          image: quay.azk8s.cn/k8scsi/csi-provisioner:v1.3.0
ceph-csi/deploy/rbd/kubernetes/v1.14+/csi-rbdplugin-provisioner.yaml:          image: quay.azk8s.cn/k8scsi/csi-snapshotter:v1.2.1
ceph-csi/deploy/rbd/kubernetes/v1.14+/csi-rbdplugin-provisioner.yaml:          image: quay.azk8s.cn/k8scsi/csi-attacher:v1.2.0
ceph-csi/deploy/rbd/kubernetes/v1.14+/csi-rbdplugin-provisioner.yaml:          image: quay.azk8s.cn/cephcsi/cephcsi:canary
ceph-csi/deploy/rbd/kubernetes/v1.14+/csi-rbdplugin-provisioner.yaml:          image: quay.azk8s.cn/cephcsi/cephcsi:canary
[root@k8s01 rbd]#
```

Deploy the plugin

```bash
[root@k8s01 ceph]# cd ceph-csi/examples/rbd
[root@k8s01 rbd]# ./plugin-deploy.sh
```

## apply storage class

Create a new pool

```bash
[root@k8s01 ceph-cluster]# ceph osd pool create  rbd 32
```

Get <cluster-id>

```bash
[root@k8s01 ceph-cluster]# ceph fsid
```

Get monitors list

```bash
[root@k8s01 ceph-cluster]# ceph mon dump
```

csi-config-map.yaml

> A k8s ConfigMap for clusterID and monitors list

```bash
[root@k8s01 rbd]# cat csi-config-map.yaml
---
# This is a sample config map that helps define a Ceph cluster configuration
# as required by the CSI plugins.
apiVersion: v1
kind: ConfigMap
# The <cluster-id> is used by the CSI plugin to uniquely identify and use a
# Ceph cluster, the value MUST match the value provided as `clusterID` in the
# StorageClass
# The <MONValue#> fields are the various monitor addresses for the Ceph cluster
# identified by the <cluster-id>
# If a CSI plugin is using more than one Ceph cluster, repeat the section for
# each such cluster in use.
# To add more clusters or edit MON addresses in an existing config map, use
# the `kubectl replace` command.
# NOTE: Changes to the config map is automatically updated in the running pods,
# thus restarting existing pods using the config map is NOT required on edits
# to the config map.
data:
  config.json: |-
    [
      {
        "clusterID": "5ae4c017-2452-49fc-a4c8-16079e80810e",
        "monitors": [
          "10.10.11.57:6789",
          "10.10.11.58:6789",
          "10.10.11.59:6789"
        ]
      }
    ]
metadata:
  name: ceph-csi-config
[root@k8s01 rbd]#
```

secret.yaml

> A k8s Secret for account, admin here.

```bash
[root@k8s01 rbd]# cat secret.yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: csi-rbd-secret
  namespace: default
stringData:
  # Key values correspond to a user name and its key, as defined in the
  # ceph cluster. User ID should have required access to the 'pool'
  # specified in the storage class
  userID: admin
  userKey: AQCZA4tdhN/0BBAAdffFb+6odeFYd5H7gaSZ2w==
[root@k8s01 rbd]#
```

storageclass.yaml

> A k8s StorageClass, with ceph-csi-config and csi-rbd-secret, data pool name is "rbd" here.

```bash
[root@k8s01 rbd]# cat storageclass.yaml
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: csi-rbd-sc
provisioner: rbd.csi.ceph.com
parameters:
   # String representing a Ceph cluster to provision storage from.
   # Should be unique across all Ceph clusters in use for provisioning,
   # cannot be greater than 36 bytes in length, and should remain immutable for
   # the lifetime of the StorageClass in use.
   # Ensure to create an entry in the config map named ceph-csi-config, based on
   # csi-config-map-sample.yaml, to accompany the string chosen to
   # represent the Ceph cluster in clusterID below
   clusterID: 5ae4c017-2452-49fc-a4c8-16079e80810e
   # If you want to use erasure coded pool with RBD, you need to create
   # two pools. one erasure coded and one replicated.
   # You need to specify the replicated pool here in the `pool` parameter, it is
   # used for the metadata of the images.
   # The erasure coded pool must be set as the `dataPool` parameter below.
   # dataPool: ec-data-pool
   pool: rbd

   # RBD image features, CSI creates image with image-format 2
   # CSI RBD currently supports only `layering` feature.
   imageFeatures: layering

   # The secrets have to contain Ceph credentials with required access
   # to the 'pool'.
   csi.storage.k8s.io/provisioner-secret-name: csi-rbd-secret
   csi.storage.k8s.io/provisioner-secret-namespace: default
   csi.storage.k8s.io/node-stage-secret-name: csi-rbd-secret
   csi.storage.k8s.io/node-stage-secret-namespace: default
   # Specify the filesystem type of the volume. If not specified,
   # csi-provisioner will set default as `ext4`.
   csi.storage.k8s.io/fstype: ext4
   # uncomment the following to use rbd-nbd as mounter on supported nodes
   # mounter: rbd-nbd
reclaimPolicy: Delete
mountOptions:
   - discard

[root@k8s01 rbd]#
```

For now, you can use the pvc and pod demo in this github repo for testing or verifying.

