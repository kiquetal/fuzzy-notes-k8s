#### Where is located?

    In etcd.

#### First Task


A ConfigMap is simple data associated with a unique key.

```
kubectl create configmap mountains --from-literal=aKey=aValue --from-literal=14ers=www.14ers.com
```


To see the actual data

```
kubectl get configmap mountains -o yaml
kubectl describe configmap mountains
kubectl delete configmap mountains
```

file: ucs-org.yaml

``` 
apiVersion: v1
kind: ConfigMap
metadata:
  name: ucs-info
  namespace: default
data:
  property.1: hello
  property.2: world
  ucs-org: |-
    description="Our scientists and engineers develop and implement innovative, practical solutions to some of our planet's most pressing problems"
    formation=1969
    headquarters="Cambridge, Massachusetts, US"
    membership="over 200,000"
    director="Kathleen Rest"
    president="Kenneth Kimmell"
    founder="Kurt Gottfried"
    concerns="Global warming and developing sustacinable ways to feed, power, and transport ourselves, to fighting misinformation, advancing racial equity, and reducing the threat of nuclear war."
    website="ucsusa.org"
```

```
kubectl create -f ucs-org.yaml
kubectl describe configmap ucs-info
```


file: consume-cli.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: consume-via-cli
spec:
  containers:
    - name: consuming-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "echo $(PROPERTY_ONE_KEY); echo $(UCS_INFO); env" ]
      env:
        - name: PROPERTY_ONE_KEY
          valueFrom:
            configMapKeyRef:
              name: ucs-info
              key: property.1
        - name: UCS_INFO
          valueFrom:
            configMapKeyRef:
              name: ucs-info
              key: ucs-org
  restartPolicy: Never
```

kubectl create -f consume-via-cli.yaml

### Three access techniques

#### Once the configuration data is stored in ConfigMaps, the containers can access the data. Pods grant their containers access to the ConfigMaps through these three techniques: 

-   through the application command-line arguments,
-    through the system environment variables accessible by the application,
-    through a specific read-only file accessible by the application.

file: consume-via-env.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: consume-via-env
spec:
  containers:
    - name: consuming-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "env" ]
      envFrom:
      - configMapRef:
          name: ucs-info
  restartPolicy: Never
```

file: consume-via-vol.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: consume-via-vol
spec:
  containers:
    - name: consuming-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh","-c","cat /etc/config/keys" ]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: ucs-info
        items:
        - key: ucs-org
          path: keys
  restartPolicy: Never
```

 #### Creating password

```
kubectl create secret generic db-password --from-literal=password=MyDbPassw0rd
```

see the file

```
kubectl get secret db-password -o yaml
```

secret.yaml

```
apiVersion: v1
kind: Secret
metadata:
  name: db-creds
type: Opaque
data:
  username: dXNlcgo=
  password: TXlEYlBhc3N3MHJkCg==
```

from 

```
echo MyDbPassw0rd | base64
```

Using `dry-run` for create base64

```
kubectl create secret generic db-password --from-literal=password=MyDbPassw0rd --dry-run -o yaml > my-secret.yaml
```

kuard.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: kuard
  labels:
    app: kuard
spec:
  containers:
  - image: gcr.io/kuar-demo/kuard-amd64:1
    name: kuard
    ports:
    - containerPort: 8080
      name: http
      protocol: TCP
    env:
    - name: SECRET_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-creds
          key: username
    - name: SECRET_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-creds
          key: password
```
