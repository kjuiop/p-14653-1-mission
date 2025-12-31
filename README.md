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