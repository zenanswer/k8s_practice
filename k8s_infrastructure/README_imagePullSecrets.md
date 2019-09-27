# imagePullSecrets and secret

Some registry need auth befor pulling image.

https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/

The official doc is very informative, so, only some memo here.

## Create a secret

```bash
kubectl -n tag create secret docker-registry regcred \
--docker-server=csmssim01:80 \
--docker-username=xuechenwang \
--docker-password=Zenanswer03 \
--docker-email=xuechenwang@cienet.com.cn
```

1. -n tag: here is a namespace, you can omit it for default namespace or specify to another namespace.
2. secret name is "regcred"
3. --docker-server: the registry server's address.


```bash
[root@k8s01 k8s_practice]# kubectl -n tag get secret regcred --output=yaml
apiVersion: v1
data:
  .dockerconfigjson: eyJhdXRocyI6eyJjc21zc2ltMDE6ODAiOnsidXNlcm5hbWUiOiJ4dWVjaGVud2FuZyIsInBhc3N3b3JkIjoiWmVuYW5zd2VyMDMiLCJlbWFpbCI6Inh1ZWNoZW53YW5nQGNpZW5ldC5jb20uY24iLCJhdXRoIjoiZUhWbFkyaGxibmRoYm1jNldtVnVZVzV6ZDJWeU1ETT0ifX19
kind: Secret
metadata:
  creationTimestamp: "2019-09-20T08:39:07Z"
  name: regcred
  namespace: tag
  resourceVersion: "40954"
  selfLink: /api/v1/namespaces/tag/secrets/regcred
  uid: 4bf99590-7688-454a-86c7-e948edd122ac
type: kubernetes.io/dockerconfigjson
[root@k8s01 k8s_practice]#
```

1. k8s is going to auth against the server, if successful, an ".dockerconfigjson" will be generated. It is a base64 incoded "~/.docker/config.json".
2. namespace is tag in this sample.

NOTE: the decoded ".dockerconfigjson" is:

```text
{"auths":{"csmssim01:80":{"username":"xuechenwang","password":"Zenanswer03","email":"xuechenwang@cienet.com.cn","auth":"eHVlY2hlbndhbmc6WmVuYW5zd2VyMDM="}}}
```

## others

### If you want to create a secret.yaml file manually, here are some notice

https://stackoverflow.com/questions/34848422/how-to-debug-imagepullbackoff

Additional debuging steps:

1. Identify the node by doing a 'kubectl/oc get pods -o wide'
2. ssh into the node that can not pull the docker image
3. check that the node can resolve the DNS of the docker registry by performing a ping.
4. try to pull the docker image manually on the node
5. If you are using a private registry, check that your secret exists and the secret is correct. Your secret should also be in the same namespace. Thanks swenzel
6. Try to pull the image locally
7. for ".dockerconfigjson", the origin text before incoding shoule be unformatted (no newline, space or tab), you can find a sample above.

### A sample usage of imagePullSecrets

```text
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
  namespace: <name-space>
spec:
  containers:
  - name: private-reg-container
    image: <your-private-image>
  imagePullSecrets:
  - name: <secret-name>
  ```
  
