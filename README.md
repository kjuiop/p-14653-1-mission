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