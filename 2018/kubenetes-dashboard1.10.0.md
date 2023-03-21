# kubeadmin dashboard
### 
[dashboard](https://github.com/kubernetes/dashboard)
```
mkdir /data/k8s
cd /data/k8s
wget https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```
### modify kubernetes-dashboard.yaml
并在此文件中的Service部分下添加type: NodePort和nodePort: 30001，添加位置如下
```
vim /data/k8s/kubernetes-dashboard.yaml
```

![image.png](https://upload-images.jianshu.io/upload_images/1542757-23aca9ec71765224.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/900)


### download dashboard images
```
docker pull anjia0532/google-containers.kubernetes-dashboard-amd64:v1.10.0
```

```
[root@inspur2 k8s]# docker images
REPOSITORY                                               TAG                 IMAGE ID            CREATED             SIZE
anjia0532/google-containers.kubernetes-dashboard-amd64   v1.10.0             0dab2435c100        3 weeks ago         122MB
k8s.gcr.io/kube-proxy-amd64                              v1.11.1             d5c25579d0ff        8 weeks ago         97.8MB
k8s.gcr.io/kube-scheduler-amd64                          v1.11.1             272b3a60cd68        8 weeks ago         56.8MB
k8s.gcr.io/kube-controller-manager-amd64                 v1.11.1             52096ee87d0e        8 weeks ago         155MB
k8s.gcr.io/kube-apiserver-amd64                          v1.11.1             816332bd9d11        8 weeks ago         187MB
k8s.gcr.io/coredns                                       1.1.3               b3b94275d97c        3 months ago        45.6MB
k8s.gcr.io/etcd-amd64                                    3.2.18              b8df3b177be2        5 months ago        219MB
quay.io/coreos/flannel                                   v0.10.0-amd64       f0fad859c909        7 months ago        44.6MB
k8s.gcr.io/pause                                         3.1                 da86e6ba6ca1        8 months ago        742kB
```
### modify docker images tag
```
[root@inspur2 k8s]# docker tag anjia0532/google-containers.kubernetes-dashboard-amd64:v1.10.0  k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.0
```
![image.png](https://upload-images.jianshu.io/upload_images/1542757-81a6148f642e9ba7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/900)


delete images
```
[root@inspur2 k8s]# docker rmi anjia0532/google-containers.kubernetes-dashboard-amd64:v1.10.0
Untagged: anjia0532/google-containers.kubernetes-dashboard-amd64:v1.10.0
Untagged: anjia0532/google-containers.kubernetes-dashboard-amd64@sha256:1d2e1229a918f4bc38b5a3f9f5f11302b3e71f8397b492afac7f273a0008776a
```
### kubectl create
``` 
[root@inspur2 k8s]# kubectl apply -f kubernetes-dashboard.yaml
secret/kubernetes-dashboard-certs created
serviceaccount/kubernetes-dashboard created
role.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
deployment.apps/kubernetes-dashboard created
service/kubernetes-dashboard created
```
### look pods
```
[root@inspur2 k8s]# kubectl get pods -n kube-system
NAME                                            READY     STATUS              RESTARTS   AGE
coredns-78fcdf6894-m9kwr                        1/1       Running             1          20h
coredns-78fcdf6894-tgjn6                        1/1       Running             1          20h
etcd-inspur2.ops.haodf.bj1                      1/1       Running             1          20h
kube-apiserver-inspur2.ops.bds.bj1            1/1       Running             1          20h
kube-controller-manager-inspur2.ops.bds.bj1   1/1       Running             1          20h
kube-flannel-ds-amd64-d8vxk                     0/1       CrashLoopBackOff    40         20h
kube-flannel-ds-amd64-lh2jf                     1/1       Running             1          3h
kube-proxy-24gvj                                1/1       Running             1          20h
kube-proxy-n29ch                                1/1       Running             0          3h
kube-scheduler-inspur2.ops.bds.bj1            1/1       Running             1          20h
kubernetes-dashboard-767dc7d4d-8d5nf            0/1       ContainerCreating   0          12s
```
### dashboard info 
```
[root@inspur2 k8s]# kubectl describe svc kubernetes-dashboard -n kube-system
Name:                     kubernetes-dashboard
Namespace:                kube-system
Labels:                   k8s-app=kubernetes-dashboard
Annotations:              kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"k8s-app":"kubernetes-dashboard"},"name":"kubernetes-dashboard","namespace":...
Selector:                 k8s-app=kubernetes-dashboard
Type:                     NodePort
IP:                       10.110.35.200
Port:                     <unset>  443/TCP
TargetPort:               8443/TCP
NodePort:                 <unset>  30001/TCP
Endpoints:                10.244.1.2:8443
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```
###  where is node run dashboard 
```  
[root@inspur2 k8s]# kubectl  get pods -n kube-system -o wide
NAME                                            READY     STATUS             RESTARTS   AGE       IP             NODE
coredns-78fcdf6894-m9kwr                        1/1       Running            1          20h       10.244.0.5     inspur2.ops.haodf.bj1
coredns-78fcdf6894-tgjn6                        1/1       Running            1          20h       10.244.0.4     inspur2.ops.haodf.bj1
etcd-inspur2.ops.haodf.bj1                      1/1       Running            1          20h       10.1.101.202   inspur2.ops.haodf.bj1
kube-apiserver-inspur2.ops.bds.bj1            1/1       Running            1          20h       10.1.101.202   inspur2.ops.bds.bj1
kube-controller-manager-inspur2.ops.bds.bj1   1/1       Running            1          20h       10.1.101.202   inspur2.ops.bds.bj1
kube-flannel-ds-amd64-d8vxk                     0/1       CrashLoopBackOff   41         20h       10.1.101.202   inspur2.ops.bds.bj1
kube-flannel-ds-amd64-lh2jf                     1/1       Running            1          3h        10.1.101.203   inspur3.ops.haodf.bj1
kube-proxy-24gvj                                1/1       Running            1          20h       10.1.101.202   inspur2.ops.haodf.bj1
kube-proxy-n29ch                                1/1       Running            0          3h        10.1.101.203   inspur3.ops.haodf.bj1
kube-scheduler-inspur2.ops.bds.bj1            1/1       Running            1          20h       10.1.101.202   inspur2.ops.bds.bj1
kubernetes-dashboard-767dc7d4d-8d5nf            1/1       Running            0          4m        10.244.1.2     inspur3.ops.bds.bj1
```
### get nodeport
```
[root@inspur2 k8s]#  kubectl get service -n kube-system -o wide
NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE       SELECTOR
kube-dns               ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP   21h       k8s-app=kube-dns
kubernetes-dashboard   NodePort    10.110.35.200   <none>        443:30001/TCP   48m       k8s-app=kubernetes-dashboard
```

### create admin user

```
# cd /data/k8s/ &&  cat admin-user.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: admin
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin
  namespace: kube-system
```
### get tokens
```
[root@inspur2 k8s]# kubectl describe serviceaccount admin -n kube-system
Name:                admin
Namespace:           kube-system
Labels:              k8s-app=kubernetes-dashboard
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   admin-token-dpdmt
Tokens:              admin-token-dpdmt
Events:              <none>
```
### token info 
 
```
kubectl describe secret admin-token-dpdmt  -n kube-system
```

![image.png](https://upload-images.jianshu.io/upload_images/1542757-91482f0db77574a9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/900)

### access url
```
https://10.1.101.203:30001
```
### login dashboard
通过上面生成的一堆字符串token令牌来登录 
1 
![image.png](https://upload-images.jianshu.io/upload_images/1542757-844308eebfd69fb8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/900)


2 
![image.png](https://upload-images.jianshu.io/upload_images/1542757-c80335b51ab8c5db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/900)

