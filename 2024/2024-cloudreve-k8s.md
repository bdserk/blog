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
            claimName: hgpan-pvc-cfsclaim
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


### 





## 
