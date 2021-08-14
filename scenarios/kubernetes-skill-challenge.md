#### Challenge

	Assembling a Multi-Component Application

#### First task

Redis Deployment
The Voting application needs a single Redis datastore where all the votes will be recorded.

Create a Deployment called redis that references container image redis:6.0.9-alpine.

Label the Deployment as app=redis. Deploy all resources to the default namespace.


apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  labels:
    app: redis
spec:
  selector:
    matchLabels:
      app: redis
  replicas: 1
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
        - name: redis
          image: redis:6.0.9-alpine
          ports:
            - containerPort: 6379

#### Second task

A service is needed to access Redis. Create a Kubernetes Service called redis.
Ensure the service is the type ClusterIP.
Label the Deployment as app=redis. Ensure the exposed Redis port is 6379.

apiVersion: v1
kind: Service
metadata:
  name: redis
  labels:
    app: redis
spec:
  type: ClusterIP
  ports:
    - port: 6379
      protocol: TCP
  selector:
    app: redis
