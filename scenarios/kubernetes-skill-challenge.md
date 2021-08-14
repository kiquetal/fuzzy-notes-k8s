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

#### Third Task

For the Voting web interface, a Python web application is already written and packaged in a public container image dockersamples/examplevotingapp_vote:latest.

Create a single Deployment and name it voting.

Label the Deployment as app=voting.


apiVersion: apps/v1
kind: Deployment
metadata:
  name: voting
  labels:
    app: voting
spec:
  selector:
    matchLabels:
      app: voting
  replicas: 1
  template:
    metadata:
      labels:
        app: voting
    spec:
      containers:
        - name: voting
          image: dockersamples/examplevotingapp_vote:latest
          ports:
            - containerPort: 80


#### Fourth Task

A service is needed to access the Voting app.

Create a Kubernetes Service called voting. Ensure the service is the type NodePort.

Ensure the exposed Voting nodePort is 32000 which connects to port 80. Label the Service as app=voting.


apiVersion: v1
kind: Service
metadata:
  name: voting
  labels:
    app: voting
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
      nodePort: 32000
  selector:
    app: voting

