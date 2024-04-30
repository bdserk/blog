## k8s编写
- 
- k8s: v1.22.5
- cloudreve: 3.8.3
  
## 安装cloudreve网盘


### 安装前规划
- 腾讯云obs块存储
- 腾讯云NFS

### cloudreve deployment

```shell
apiVersion: v1
kind: ConfigMap
metadata:
  name: cloudreve-config
  namespace: infra
  labels:
    app: cloudreve
data:
  conf.ini: |-
    [System]
    Debug = false
    Mode = master
    Listen = :5212
    [Database]
    DBFile = /data/cloudreve/cloudreve.db

---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: infra
  name: cloudreve
  labels:
    app: cloudreve
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cloudreve
  template:
    metadata:
      labels:
        app: cloudreve
      annotations:
        port: "5212"
        golang: "1.18"
        cloudreve: "latest"
        github: "cloudreve/cloudreve"
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: role
                operator: In
                values:
                - ops

      containers:
        - name: cloudreve
          image: cloudreve/cloudreve:latest
          command: ["./cloudreve"]
          resources:
            limits:
              cpu: 1024m
              memory: 1024Mi
            requests:
              cpu: 1024m
              memory: 1024Mi
          ports:
            - containerPort: 5212
          volumeMounts:
            - mountPath: /cloudreve/conf.ini
              name: conf
              readOnly: true
              subPath: conf.ini
            - mountPath: /data/cloudreve
              name: db
            - mountPath: /data/cloudreve-upload
              name: nfs-uploads

      volumes:
        - name: conf
          configMap:
            name: cloudreve-config
            items:
              - key: conf.ini
                path: conf.ini
        - name: db
          persistentVolumeClaim:
            claimName: cloudreve-pvc
        - name: nfs-uploads
          persistentVolumeClaim:
            claimName: xxx-pvc-cfsclaim
---
apiVersion: v1
kind: Service
metadata:
  namespace: infra
  name: cloudreve
  labels:
    app: cloudreve
spec:
  selector:
    app: cloudreve
  ports:
  - name: cloudreve-port
    protocol: TCP
    port: 5212
    targetPort: 5212
```


### cloudreve ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "1024M"
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "1500"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "1500"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "1500"
  labels:
    app: pan
  name: pan
  namespace: infra
spec:
  ingressClassName: nginx
  rules:
  - host: pan.aaa.com
    http:
      paths:
      - backend:
          service:
            name: cloudreve
            port:
              number: 5212
        path: /
        pathType: Prefix


### cloudreve nginx配置

```shell
upstream pan {
	server  x.x.x.x:5212  max_fails=3 fail_timeout=10s weight=100;  
}
server {
    listen                              80;
    listen                              443 ssl http2;
    server_name                         pan.aaa.com;  #cloudreve

    include                             global.conf;

    include                             ssl_ecarxmap.com.conf;
    access_log                          /data/logs/pan-openresty.log json;

    client_max_body_size                5000M;
    client_body_buffer_size             5000M;

    if ($http_x_forwarded_proto = http ) {
        return 301 https://$host$request_uri;
    }


    location / {

    if ($request_uri ~* "keywords") {
            return 301 https://pan.aaa.com/login;
       }
        proxy_pass                      http://pan;
        proxy_connect_timeout           1500s;
        proxy_read_timeout              1500s;
        proxy_send_timeout              1500s;
        proxy_set_header                X-Forwarded-For $front_real_ip;
        proxy_set_header                X-Request-ID $request_id;
    }
}
```



