# p-14653-1-mission

### 0001 : docker-desktop k8s 설정 완료

<br />

### 0002 : kubectl 기본 명령어

```
kubectl get nodes

NAME             STATUS   ROLES                  AGE     VERSION
docker-desktop   Ready    control-plane,master   6h10m   v1.22.5
```


```
kubectl describe node docker-desktop

Name:               docker-desktop
Roles:              control-plane,master
Labels:             beta.kubernetes.io/arch=arm64
beta.kubernetes.io/os=linux
kubernetes.io/arch=arm64
kubernetes.io/hostname=docker-desktop
kubernetes.io/os=linux
node-role.kubernetes.io/control-plane=
node-role.kubernetes.io/master=
node.kubernetes.io/exclude-from-external-load-balancers=
...
```

```
kubectl describe node docker-desktop

Name:               docker-desktop
Roles:              control-plane,master
Labels:             beta.kubernetes.io/arch=arm64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=arm64
                    kubernetes.io/hostname=docker-desktop
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/control-plane=
                    node-role.kubernetes.io/master=
                    node.kubernetes.io/exclude-from-external-load-balancers=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock

```

```
kubectl cluster-info

Kubernetes control plane is running at https://kubernetes.docker.internal:6443
CoreDNS is running at https://kubernetes.docker.internal:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

```


```
kubectl config current-context

docker-desktop
```


```
kubectl get namespaces
kubectl get ns

NAME              STATUS   AGE
default           Active   6h15m
kube-node-lease   Active   6h15m
kube-public       Active   6h15m
kube-system       Active   6h15m

```

```

kubectl get all -n kube-system

NAME                                         READY   STATUS    RESTARTS       AGE
pod/coredns-78fcd69978-gr5bb                 1/1     Running   0              6h15m
pod/coredns-78fcd69978-jtqcg                 1/1     Running   0              6h15m
pod/etcd-docker-desktop                      1/1     Running   0              6h15m
pod/kube-apiserver-docker-desktop            1/1     Running   0              6h15m
pod/kube-controller-manager-docker-desktop   1/1     Running   0              6h15m
pod/kube-proxy-bgfp4                         1/1     Running   0              6h15m
pod/kube-scheduler-docker-desktop            1/1     Running   5 (90m ago)    6h15m
pod/storage-provisioner                      1/1     Running   5 (89m ago)    6h15m
pod/vpnkit-controller                        1/1     Running   28 (14m ago)   6h15m

```

<br />

### 0003

```
kubectl run nginx-pod --image=nginx:latest
```

```
kubectl get pods

NAME        READY   STATUS              RESTARTS   AGE
nginx-pod   0/1     ContainerCreating   0          10s
```

```
kubectl get pods -w

NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          38s
```

```
kubectl describe pod nginx-pod

Name:         nginx-pod
Namespace:    default
Priority:     0
Node:         docker-desktop/192.168.65.4
Start Time:   Wed, 31 Dec 2025 15:37:35 +0900
Labels:       run=nginx-pod
Annotations:  <none>
Status:       Running
IP:           10.1.0.20
IPs:

```

```
kubectl exec -it nginx-pod -- bash

cat /etc/nginx/nginx.conf

user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /run/nginx.pid;


events {
    worker_connections  1024;
}


...
```

```
kubectl logs nginx-pod
kubectl logs nginx-pod -f        # 실시간 로그 (tail -f처럼)
kubectl logs nginx-pod --tail=50 # 마지막 50줄만

/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2025/12/31 06:37:45 [notice] 1#1: using the "epoll" event method
2025/12/31 06:37:45 [notice] 1#1: nginx/1.29.4
2025/12/31 06:37:45 [notice] 1#1: built by gcc 14.2.0 (Debian 14.2.0-19) 

...

```

```
kubectl delete pod nginx-pod

pod "nginx-pod" deleted

```

<br />

### 0004

```
kubectl apply -f nginx-pod.yaml

pod/nginx-pod created
```

```
kubectl get pods -o wide

NAME        READY   STATUS    RESTARTS   AGE   IP          NODE             NOMINATED NODE   READINESS GATES
nginx-pod   1/1     Running   0          20s   10.1.0.21   docker-desktop   <none>           <none>
```

```
kubectl get pod nginx-pod -o yaml

apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"app":"nginx","environment":"dev"},"name":"nginx-pod","namespace":"default"},"spec":{"containers":[{"image":"nginx:1.25","name":"nginx","ports":[{"containerPort":80}],"resources":{"limits":{"cpu":"200m","memory":"128Mi"},"requests":{"cpu":"100m","memory":"64Mi"}}}]}}
  creationTimestamp: "2025-12-31T06:43:27Z"
  labels:
    app: nginx
    environment: dev
  name: nginx-pod
  namespace: default
  resourceVersion: "28633"
...
```

```
kubectl get pods -l app=nginx
kubectl get pods -l environment=dev
```

```
kubectl delete -f nginx-pod.yaml

pod "nginx-pod" deleted
```

<br />

### 0005

```
# Pod 생성
kubectl apply -f multi-container-pod.yaml

pod/multi-container-pod created
```

```
kubectl get pods

NAME                  READY   STATUS    RESTARTS   AGE
multi-container-pod   2/2     Running   0          23s
```

```
kubectl logs multi-container-pod -c log-sidecar
```

```
kubectl exec -it multi-container-pod -c main-app -- bash
```


```
kubectl exec multi-container-pod -c main-app -- curl localhost

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
100   615  100   615    0     0  41537      0 --:--:-- --:--:-- --:--:-- 55909

```

```
kubectl logs multi-container-pod -c log-sidecar

127.0.0.1 - - [31/Dec/2025:06:47:42 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.88.1" "-"

```

```
kubectl delete -f multi-container-pod.yaml

pod "multi-container-pod" deleted
```

<br />

### 0006

```
kubectl apply -f nginx-deployment.yaml

deployment.apps/nginx-deployment created
```

```
kubectl get deployments
kubectl get deploy

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           35s
```

```
kubectl get replicasets
kubectl get rs

NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-7dc7bfbb4b   3         3         3       68s
```

```
kubectl get pods -l app=nginx

NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7dc7bfbb4b-5spkq   1/1     Running   0          85s
nginx-deployment-7dc7bfbb4b-fbrqs   1/1     Running   0          85s
nginx-deployment-7dc7bfbb4b-qb8l4   1/1     Running   0          85s
```

```
kubectl delete pod nginx-deployment-7dc7bfbb4b-5spkq 
kubectl get pods -l app=nginx

NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7dc7bfbb4b-fbrqs   1/1     Running   0          2m12s
nginx-deployment-7dc7bfbb4b-j7lvp   1/1     Running   0          3s
nginx-deployment-7dc7bfbb4b-qb8l4   1/1     Running   0          2m12s
```

```
kubectl delete -f nginx-deployment.yaml
kubectl delete deployment nginx-deployment

deployment.apps "nginx-deployment" deleted
```

<br />

### 0007

```
kubectl scale deployment nginx-deployment --replicas=5

deployment.apps/nginx-deployment scaled
```

```
kubectl get pods -l app=nginx -w

NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7dc7bfbb4b-2hbrd   1/1     Running   0          15s
nginx-deployment-7dc7bfbb4b-fmzjw   1/1     Running   0          20s
nginx-deployment-7dc7bfbb4b-r4ttc   1/1     Running   0          20s
nginx-deployment-7dc7bfbb4b-sjjh7   1/1     Running   0          20s
nginx-deployment-7dc7bfbb4b-ws99t   1/1     Running   0          15s
```

```
kubectl get deployment nginx-deployment

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   5/5     5            5           41s
```

```
kubectl scale deployment nginx-deployment --replicas=2

deployment.apps/nginx-deployment scaled
```

```
kubectl get deployment nginx-deployment

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   2/2     2            2           79s
```

```
kubectl apply -f nginx-deployment.yaml

deployment.apps/nginx-deployment configured
```

```
kubectl get deployment nginx-deployment

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   5/5     5            5           2m25s
```

```
kubectl get pods -l app=nginx

NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7dc7bfbb4b-2hbrd   1/1     Running   0          3m10s
nginx-deployment-7dc7bfbb4b-b5ks7   1/1     Running   0          56s
nginx-deployment-7dc7bfbb4b-fmzjw   1/1     Running   0          3m15s
nginx-deployment-7dc7bfbb4b-wdtfm   1/1     Running   0          56s
nginx-deployment-7dc7bfbb4b-zbw52   1/1     Running   0          56s

kubectl delete pod nginx-deployment-7dc7bfbb4b-2hbrd

kubectl get pods -l app=nginx

NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7dc7bfbb4b-b5ks7   1/1     Running   0          78s
nginx-deployment-7dc7bfbb4b-fmzjw   1/1     Running   0          3m37s
nginx-deployment-7dc7bfbb4b-pncr9   1/1     Running   0          5s
nginx-deployment-7dc7bfbb4b-wdtfm   1/1     Running   0          78s
nginx-deployment-7dc7bfbb4b-zbw52   1/1     Running   0          78s
```