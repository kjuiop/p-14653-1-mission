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

<br />

### 0008

```
kubectl set image deployment/nginx-deployment nginx=nginx:1.26

deployment.apps/nginx-deployment image updated
```

```
kubectl rollout status deployment/nginx-deployment

deployment "nginx-deployment" successfully rolled out
```

```
kubectl get rs

nginx-deployment-7dc7bfbb4b   0         0         0       8m21s
nginx-deployment-866fbb9bfc   5         5         5       43s
```

```
kubectl rollout history deployment/nginx-deployment

REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

```
kubectl rollout undo deployment/nginx-deployment
kubectl rollout undo deployment/nginx-deployment --to-revision=1

deployment.apps/nginx-deployment rolled back
```

```
kubectl describe deployment nginx-deployment

Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:      nginx:1.25
    Port:       80/TCP
    Host Port:  0/TCP
    Limits:
      cpu:     200m
      memory:  128Mi
```

```
kubectl delete deployment nginx-deployment

deployment.apps "nginx-deployment" deleted
```

<br />

### 0009

```
kubectl apply -f nginx-deployment.yaml
kubectl get pods -l app=nginx

NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7dc7bfbb4b-5vmld   1/1     Running   0          12s
nginx-deployment-7dc7bfbb4b-b8bbp   1/1     Running   0          12s
nginx-deployment-7dc7bfbb4b-h8njf   1/1     Running   0          12s
nginx-deployment-7dc7bfbb4b-hc2hd   1/1     Running   0          12s
nginx-deployment-7dc7bfbb4b-r74tj   1/1     Running   0          12s
```

```
kubectl apply -f nginx-service-clusterip.yaml

service/nginx-service created
```

```
kubectl get services
kubectl get svc

NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes      ClusterIP   10.96.0.1      <none>        443/TCP   7h
nginx-service   ClusterIP   10.108.68.21   <none>        80/TCP    11s
```

```
kubectl describe service nginx-service

NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes      ClusterIP   10.96.0.1      <none>        443/TCP   7h
nginx-service   ClusterIP   10.108.68.21   <none>        80/TCP    11s
jake@KimJungIns-MacBook-Air p-14653-1-mission % kubectl describe service nginx-service
Name:              nginx-service
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=nginx
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.108.68.21
IPs:               10.108.68.21
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.1.0.46:80,10.1.0.47:80,10.1.0.48:80 + 2 more...
Session Affinity:  None
Events:            <none>
```

```
kubectl run curl-test \
  --image=curlimages/curl \
  --restart=Never \
  --rm -i -- \
  curl -v http://nginx-service/

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0   0     0   0     0     0     0  --:--:-- --:--:-- --:--:--     0* Host nginx-service:80 was resolved.
* IPv6: (none)
* IPv4: 10.108.68.21
*   Trying 10.108.68.21:80...
* Established connection to nginx-service (10.108.68.21 port 80) from 10.1.0.52 port 54180 
* using HTTP/1.x
> GET / HTTP/1.1
> Host: nginx-service
> User-Agent: curl/8.17.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 200 OK
< Server: nginx/1.25.5
< Date: Wed, 31 Dec 2025 07:21:25 GMT
< Content-Type: text/html
< Content-Length: 615
< Last-Modified: Tue, 16 Apr 2024 14:29:59 GMT
< Connection: keep-alive
< ETag: "661e8b67-267"
< Accept-Ranges: bytes
< 
{ [615 bytes data]
<!DOCTYPE html>
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
100   615 100   615   0     0  8960     0  --:--:-- --:--:-- --:--:--  9044
* Connection #0 to host nginx-service:80 left intact
pod "curl-test" deleted
```

```
kubectl delete -f nginx-service-clusterip.yaml
kubectl delete service nginx-service

service "nginx-service" deleted
```

<br />

### 0010

```
kubectl apply -f nginx-deployment.yaml

kubectl get pods -l app=nginx

NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7dc7bfbb4b-5vmld   1/1     Running   0          7m52s
nginx-deployment-7dc7bfbb4b-b8bbp   1/1     Running   0          7m52s
nginx-deployment-7dc7bfbb4b-h8njf   1/1     Running   0          7m52s
nginx-deployment-7dc7bfbb4b-hc2hd   1/1     Running   0          7m52s
nginx-deployment-7dc7bfbb4b-r74tj   1/1     Running   0          7m52s
```

```
kubectl apply -f nginx-service-nodeport.yaml

service/nginx-nodeport created
```

```
kubectl get service nginx-nodeport

NAME             TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
nginx-nodeport   NodePort   10.98.42.165   <none>        80:30080/TCP   13s

# 브라우저에서 접근
# http://localhost:30080

curl http://localhost:30080

<!DOCTYPE html>
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
```

```
kubectl delete -f nginx-service-nodeport.yaml
kubectl delete service nginx-nodeport

service "nginx-nodeport" deleted
```

<br />

### 0011

```
kubectl apply -f nginx-service-lb.yaml
kubectl get service nginx-lb

NAME       TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
nginx-lb   LoadBalancer   10.105.26.14   localhost     80:32418/TCP   3s
```

```
curl http://localhost

<!DOCTYPE html>
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
```

```
kubectl describe service nginx-lb

Name:                     nginx-lb
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=nginx
Type:                     LoadBalancer
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.105.26.14
IPs:                      10.105.26.14
LoadBalancer Ingress:     localhost
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  32418/TCP
Endpoints:                10.1.0.46:80,10.1.0.47:80,10.1.0.48:80 + 2 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

```
kubectl delete -f nginx-service-lb.yaml
kubectl delete service nginx-lb

service "nginx-lb" deleted
```

```
kubectl delete service nginx-service nginx-nodeport nginx-lb
kubectl delete deployment nginx-deployment

deployment.apps "nginx-deployment" deleted
```

<br />

### 0012

```
# 직접 값 지정
kubectl create configmap app-config \
  --from-literal=APP_ENV=development \
  --from-literal=APP_DEBUG=true

# 파일에서 생성
kubectl create configmap nginx-config \
  --from-file=app-configmap.yaml

# 디렉토리의 모든 파일
kubectl create configmap app-configs \
  --from-file=./
  
configmap/nginx-config created
```

```
kubectl get configmap
kubectl get cm

NAME               DATA   AGE
kube-root-ca.crt   1      7h16m
nginx-config       1      112m
```

```
kubectl delete configmap nginx-config
kubectl delete cm nginx-config

configmap "nginx-config" deleted
```

```
kubectl apply -f pod-with-configmap.yaml

pod/app-pod created
```

```
kubectl exec -it app-pod -- bash

KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_SERVICE_PORT=443
HOSTNAME=app-pod
PWD=/
PKG_RELEASE=1~bookworm
HOME=/root
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
NJS_VERSION=0.8.4
TERM=xterm
SHLVL=1
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PORT=443
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
NGINX_VERSION=1.25.5
NJS_RELEASE=3~bookworm
CUSTOM_ENV=development
_=/usr/bin/env
```

```
kubectl delete -f pod-with-configmap.yaml
kubectl delete -f app-configmap.yaml
kubectl delete pod app-pod
kubectl delete configmap app-config

pod "app-pod" deleted
configmap "app-config" deleted
```

### 0013

```
# 명령어로 Secret 생성
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=secretpassword123
```

```
# Secret 확인 (base64 인코딩됨)
kubectl get secret db-secret -o yaml
  
apiVersion: v1
data:
  password: c2VjcmV0cGFzc3dvcmQxMjM=
  username: YWRtaW4=
kind: Secret
metadata:
  creationTimestamp: "2025-12-31T06:20:30Z"
  name: db-secret
  namespace: default
  resourceVersion: "26953"
  uid: aad89d6f-6bd5-42fc-8755-1f537ac4b0fe
type: Opaque


# base64 디코딩해서 확인
kubectl get secret db-secret -o jsonpath='{.data.username}' | base64 -d
admin
```

```
# base64 인코딩
echo -n "admin" | base64
YWRtaW4=

echo -n "secretpass123" | base64
c2VjcmV0cGFzczEyMw==


kubectl apply -f db-secret.yaml
secret/db-secret configured
```

```
kubectl apply -f pod-with-secret.yaml
pod/db-app-pod created

kubectl exec -it db-app-pod -- sh

cat /etc/secrets/username
admin

cat /etc/secrets/password
secretpass123
```

```
kubectl delete -f pod-with-secret.yaml
pod "db-app-pod" deleted

kubectl delete -f db-secret.yaml
secret "db-secret" deleted
```

<br />

### 0014

```
# 스토리지 생성

kubectl apply -f pv-local.yaml
persistentvolume/local-pv created

kubectl apply -f pvc-local.yaml
persistentvolumeclaim/local-pvc created

kubectl apply -f pod-with-pvc.yaml
pod/pvc-pod created

```

```
# PV 상태 확인
kubectl get pv

NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS   REASON   AGE
local-pv   1Gi        RWO            Retain           Bound    default/local-pvc   manual                  50s
```

```
# 1. Pod에 데이터 쓰기
kubectl exec pvc-pod -- sh -c "echo 'Hello K8s' > /usr/share/nginx/html/index.html"

# 2. 확인
kubectl exec pvc-pod -- cat /usr/share/nginx/html/index.html
Hello K8s

# 3. Pod 삭제
kubectl delete pod pvc-pod
pod "pvc-pod" deleted

# 4. Pod 다시 생성
kubectl apply -f pod-with-pvc.yaml
pod/pvc-pod created

# 5. 데이터 확인 
kubectl exec pvc-pod -- cat /usr/share/nginx/html/index.html
Hello K8s
```

```
kubectl delete -f pod-with-pvc.yaml
pod "pvc-pod" deleted

kubectl delete -f pvc-local.yaml
persistentvolumeclaim "local-pvc" deleted

kubectl delete -f pv-local.yaml
persistentvolume "local-pv" deleted
```

<br />

### 0015

```
kubectl apply -f namespace.yaml
namespace/demo-app created

kubectl apply -f backend-deployment.yaml
deployment.apps/backend created
service/backend-service created

kubectl apply -f k8s-demo/


kubectl apply -f frontend-deployment.yaml
deployment.apps/frontend created
service/frontend-service created
```

```
kubectl get all -n demo-app

NAME                            READY   STATUS    RESTARTS   AGE
pod/backend-b5c8df878-8kv74     1/1     Running   0          2m33s
pod/backend-b5c8df878-vkx4l     1/1     Running   0          2m33s
pod/frontend-5fcfc7fdf4-5qh5s   1/1     Running   0          20s
pod/frontend-5fcfc7fdf4-q8jkz   1/1     Running   0          20s

NAME                       TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/backend-service    ClusterIP      10.107.159.113   <none>        8080/TCP       2m33s
service/frontend-service   LoadBalancer   10.111.16.111    localhost     80:31552/TCP   20s

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/backend    2/2     2            2           2m33s
deployment.apps/frontend   2/2     2            2           20s

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/backend-b5c8df878     2         2         2       2m33s
replicaset.apps/frontend-5fcfc7fdf4   2         2         2       20s
```

```
kubectl exec -n demo-app -it \
  $(kubectl get pod -n demo-app -l app=frontend -o jsonpath='{.items[0].metadata.name}') \
  -- curl backend-service:8080
  
Hello from Backend!
```

```
kubectl delete namespace demo-app
namespace "demo-app" deleted

# 개별 삭제
kubectl delete -f frontend-deployment.yaml -n demo-app
kubectl delete -f backend-deployment.yaml -n demo-app
kubectl delete -f namespace.yaml
```

<br />

### 0016

```
# 특정 네임스페이스의 모든 리소스 삭제
kubectl delete all --all -n demo-app
kubectl delete namespace demo-app

# 특정 라벨의 모든 리소스 삭제
kubectl delete all -l app=nginx

kubectl get pods -n demo-app
No resources found in demo-app namespace.
```

```
kubectl describe pod <pod-name>

Name:         nginx-deployment-7dc7bfbb4b-8d2fq
Namespace:    default
Priority:     0
Node:         docker-desktop/192.168.65.4
Start Time:   Thu, 01 Jan 2026 20:26:18 +0900
Labels:       app=nginx
              pod-template-hash=7dc7bfbb4b
Annotations:  <none>
Status:       Running
IP:           10.1.0.67
IPs:
  IP:           10.1.0.67
Controlled By:  ReplicaSet/nginx-deployment-7dc7bfbb4b
Containers:
  nginx:
    Container ID:   docker://c2fd6ef8896a70f2a24f991cbaf1544daedccafe057f062171d99599b1d3a822
    Image:          nginx:1.25
    Image ID:       docker-pullable://nginx@sha256:a484819eb60211f5299034ac80f6a681b06f89e65866ce91f356ed7c72af059c
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 01 Jan 2026 20:26:20 +0900
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     200m
      memory:  128Mi
    Requests:
      cpu:        100m
      memory:     64Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-q2ntf (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-q2ntf:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  25s   default-scheduler  Successfully assigned default/nginx-deployment-7dc7bfbb4b-8d2fq to docker-desktop
  Normal  Pulled     24s   kubelet            Container image "nginx:1.25" already present on machine
  Normal  Created    24s   kubelet            Created container nginx
  Normal  Started    24s   kubelet            Started container nginx
```

```
kubectl logs nginx-deployment-7dc7bfbb4b-pp422 --previous
```

```
kubectl apply -f multi-container-pod.yaml
pod/multi-container-pod created

kubectl logs multi-container-pod --all-containers

/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
```

```
# Metrics Server 설치
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

kubectl get pods -n kube-system | grep metrics-server
metrics-server-6f6dc87d47-zhfxl          0/1     Running   0              7s

# metric server 에 tls 설정을 위한 args 추가
kubectl patch deployment metrics-server -n kube-system \
  --type='json' \
  -p='[
    {"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}
  ]'

# 설정 적용을 위한 재시작
kubectl rollout status -n kube-system deploy/metrics-server

kubectl get pod -n kube-system | grep metrics-server
metrics-server-5f5f754c64-k4fwg          1/1     Running   0              46s
```

```
kubectl top nodes

NAME             CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
docker-desktop   215m         5%     1549Mi          82%   
```

```
kubectl top pods -n demo-app

NAME                        CPU(cores)   MEMORY(bytes)   
backend-b5c8df878-f9xw4     0m           1Mi             
backend-b5c8df878-sh8nx     0m           1Mi             
frontend-5fcfc7fdf4-vfww4   0m           5Mi             
frontend-5fcfc7fdf4-w8sk2   0m           7Mi      
```

```
kubectl get events --sort-by='.lastTimestamp'

LAST SEEN   TYPE      REASON                    OBJECT                                   MESSAGE
38m         Normal    Scheduled                 pod/pvc-pod                              Successfully assigned default/pvc-pod to docker-desktop
19m         Normal    Scheduled                 pod/nginx-deployment-7dc7bfbb4b-8d2fq    Successfully assigned default/nginx-deployment-7dc7bfbb4b-8d2fq to docker-desktop
16m         Normal    Scheduled                 pod/nginx-deployment-5f7b46f5bd-gr5hz    Successfully assigned default/nginx-deployment-5f7b46f5bd-gr5hz to docker-desktop
19m         Normal    Scheduled                 pod/nginx-deployment-7dc7bfbb4b-jzvmc    Successfully assigned default/nginx-deployment-7dc7bfbb4b-jzvmc to docker-desktop
19m         Normal    Scheduled                 pod/nginx-deployment-7dc7bfbb4b-lnfb4    Successfully assigned default/nginx-deployment-7dc7bfbb4b-lnfb4 to docker-desktop
```

```
kubectl get deployment nginx-deployment -o yaml > backup.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"app":"nginx"},"name":"nginx-deployment","namespace":"default"},"spec":{"replicas":5,"selector":{"matchLabels":{"app":"nginx"}},"template":{"metadata":{"labels":{"app":"nginx"}},"spec":{"containers":[{"image":"nginx:1.25","name":"nginx","ports":[{"containerPort":80}],"resources":{"limits":{"cpu":"200m","memory":"128Mi"},"requests":{"cpu":"100m","memory":"64Mi"}}}]}}}}
  creationTimestamp: "2026-01-01T11:48:04Z"
  generation: 1
  labels:
    app: nginx
  name: nginx-deployment
  namespace: default
  resourceVersion: "46037"
  uid: caf008f6-23f9-4be2-9064-457d80bae37d
spec:
  progressDeadlineSeconds: 600
  replicas: 5
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: nginx
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.25
        imagePullPolicy: IfNotPresent
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
        resources:
          limits:
            cpu: 200m
            memory: 128Mi
          requests:
            cpu: 100m
            memory: 64Mi
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 5
  conditions:
  - lastTransitionTime: "2026-01-01T11:48:08Z"
    lastUpdateTime: "2026-01-01T11:48:08Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: "2026-01-01T11:48:04Z"
    lastUpdateTime: "2026-01-01T11:48:08Z"
    message: ReplicaSet "nginx-deployment-7dc7bfbb4b" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  observedGeneration: 1
  readyReplicas: 5
  replicas: 5
  updatedReplicas: 5
```

```
kubectl get all -n demo-app -o yaml > demo-app-backup.yaml
```

```
kubectl config current-context

docker-desktop
```

```
kubectl config get-contexts

CURRENT   NAME                                                          CLUSTER                                                       AUTHINFO                                                      NAMESPACE
          arn:aws:eks:ap-northeast-2:866741171631:cluster/eks-cluster   arn:aws:eks:ap-northeast-2:866741171631:cluster/eks-cluster   arn:aws:eks:ap-northeast-2:866741171631:cluster/eks-cluster   
*         docker-desktop                                                docker-desktop                                                docker-desktop                                                
          gke_careful-record-358905_us-central1-c_my-cluster            gke_careful-record-358905_us-central1-c_my-cluster            gke_careful-record-358905_us-central1-c_my-cluster            
          tools                                                         tools                                                         tools                                                         
          tools-dv05kb04                                                tools-dv05kb04                                                tools    
```

```
kubectl config set-context --current --namespace=demo-app
Context "docker-desktop" modified.

kubectl config set-context --current --namespace=default
Context "docker-desktop" modified.
```

```
kubectl get pods --all-namespaces

NAMESPACE     NAME                                     READY   STATUS    RESTARTS       AGE
default       nginx-deployment-7dc7bfbb4b-5fgz2        1/1     Running   0              3m38s
default       nginx-deployment-7dc7bfbb4b-b782t        1/1     Running   0              3m38s
default       nginx-deployment-7dc7bfbb4b-fm6bx        1/1     Running   0              3m38s
default       nginx-deployment-7dc7bfbb4b-t8jh6        1/1     Running   0              3m38s
default       nginx-deployment-7dc7bfbb4b-vz5gt        1/1     Running   0              3m38s
kube-system   coredns-78fcd69978-gr5bb                 1/1     Running   1 (63m ago)    35h
kube-system   coredns-78fcd69978-jtqcg                 1/1     Running   1 (63m ago)    35h
kube-system   etcd-docker-desktop                      1/1     Running   1 (63m ago)    35h
kube-system   kube-apiserver-docker-desktop            1/1     Running   2 (63m ago)    35h
kube-system   kube-controller-manager-docker-desktop   1/1     Running   1 (63m ago)    35h
kube-system   kube-proxy-bgfp4                         1/1     Running   1 (63m ago)    35h
kube-system   kube-scheduler-docker-desktop            1/1     Running   12 (63m ago)   35h
kube-system   metrics-server-5f5f754c64-k4fwg          1/1     Running   0              8m59s
kube-system   storage-provisioner                      1/1     Running   15 (63m ago)   35h
kube-system   vpnkit-controller                        1/1     Running   45 (28m ago)   35h
```