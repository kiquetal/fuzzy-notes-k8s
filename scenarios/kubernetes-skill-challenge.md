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


Port exposes the Kubernetes service on the specified port within the cluster. Other pods within the cluster can communicate with this server on the specified port.

TargetPort is the port on which the service will send requests to, that your pod will be listening on. Your application in the container will need to be listening on this port also.

NodePort exposes a service externally to the cluster by means of the target nodes IP address and the NodePort. NodePort is the default setting if the port field is not specified


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

#### Fifth Task

The voting solution needs to present the Results.

Deploy a Postgres database packaged in a container image postgres:9.6.20-alpine.

Create a single Deployment and name it db.

Also, add the environment variables (using kubectl set env)to the deployed container:

POSTGRES_DB=db, POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_HOST_AUTH_METHOD=trust
Label the Deployment as app=db.

kubectl set env deployment/db  POSTGRES_DB=db
kubectl set env deployment/db  POSTGRES_USER=postgres
kubectl set env deployment/db  POSTGRES_PASSWORD=postgres
kubectl set env deployment/db  POSTGRES_HOST_AUTH_METHOD=trust


apiVersion: apps/v1
kind: Deployment
metadata:
  name: db
  labels:
    app: db
spec:
  selector:
    matchLabels:
      app: db
  replicas: 1
  template:
    metadata:
      labels:
        app: db
    spec:
      containers:
        - name: db
          image: postgres:9.6.20-alpine
          ports:
            - containerPort: 5432


#### Sixth task

A service is needed to access the Results database.

Create a Kubernetes Service called db.

Ensure the service is the type ClusterIP.

Ensure the exposed port is 5432.

Label the Service as app=db.



apiVersion: v1
kind: Service
metadata:
  name: db
  labels:
    app: db
spec:
  type: ClusterIP
  ports:
    - port: 5432
      targetPort: 5432
      protocol: TCP
  selector:
    app: db

#### Seventh task



For the Voting web interface, a Python web application is already written and packaged in a public container image dockersamples/examplevotingapp_result:latest.

Create a single Deployment and name it result.

Label the Deployment as app=result.


apiVersion: apps/v1
kind: Deployment
metadata:
  name: result
  labels:
    app: result
spec:
  selector:
    matchLabels:
      app: result
  replicas: 1
  template:
    metadata:
      labels:
        app: result
    spec:
      containers:
        - name: result
          image: dockersamples/examplevotingapp_result:latest
          ports:
            - containerPort: 80

#### EIght task

A service is needed to access the Voting web interface.

Create a Kubernetes Service called result.

Ensure the service is the type NodePort.

Ensure the exposed Voting nodePort is 32001 that connects to port 80.


apiVersion: v1
kind: Service
metadata:
  name: result
  labels:
    app: result
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
      nodePort: 32001
  selector:
    app: result


#### Nineth task

Lastly, an application is needed that tallies the votes from Redis and publishes totals to Postgres.

A .NET application is already written and packaged in a public container image dockersamples/examplevotingapp_worker:latest.

Create a single Deployment and name it worker.

A Service is not needed since it just works in the background.

apiVersion: apps/v1
kind: Deployment
metadata:
  name: worker
  labels:
    app: worker
spec:
  selector:
    matchLabels:
      app: worker
  replicas: 1
  template:
    metadata:
      labels:
        app: worker
    spec:
      containers:
        - name: worker
          image: dockersamples/examplevotingapp_worker:latest
          ports:
            - containerPort: 80

