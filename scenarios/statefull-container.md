#### Volume and statefull

 Start NFS server
 
```
docker run -d --net=host \
   --privileged --name nfs-server \
   katacoda/contained-nfs-server:centos7 \
   /exports/data-0001 /exports/data-0002
```

Kubernetes requires storagevolume

file: nfs-0002.yaml

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-0002
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    server: 172.17.0.41
    path: /exports/data-0002
``` 

file: nfs-0001.yaml

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-0001
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    server: 172.17.0.41
    path: /exports/data-0001
```

You need to create claims

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: claim-mysql
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: claim-http
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

```
kubectl get pvc
```

    NAME          STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    claim-http    Bound    nfs-0001   2Gi        RWO,RWX                       2s
    claim-mysql   Bound    nfs-0002   5Gi        RWO,RWX                       1s

file: pod-www.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: www
  labels:
    name: www
spec:
  containers:
  - name: www
    image: nginx:alpine
    ports:
      - containerPort: 80
        name: www
    volumeMounts:
      - name: www-persistent-storage
        mountPath: /usr/share/nginx/html
  volumes:
    - name: www-persistent-storage
      persistentVolumeClaim:
        claimName: claim-http
```

file: pod-mysql.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: mysql
  labels:
    name: mysql
spec:
  containers:
  - name: mysql
    image: openshift/mysql-55-centos7
    env:
      - name: MYSQL_ROOT_PASSWORD
        value: yourpassword
      - name: MYSQL_USER
        value: wp_user
      - name: MYSQL_PASSWORD
        value: wp_pass
      - name: MYSQL_DATABASE
        value: wp_db
    ports:
      - containerPort: 3306
        name: mysql
    volumeMounts:
      - name: mysql-persistent-storage
        mountPath: /var/lib/mysql/data
  volumes:
    - name: mysql-persistent-storage
      persistentVolumeClaim:
        claimName: claim-mysql
```

Obtain ip from pod

Write to nfs

```
docker exec -it nfs-server bash -c "echo 'Hello World' > /exports/data-0001/index.html"

``` 


```
ip=$(kubectl get pod www -o yaml |grep podIP | awk '{split($0,a,":"); print a[2]}'); echo $ip
```
