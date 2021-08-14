#### First commands

	kubectl version --short && \
	kubectl get componentstatus && \
	kubectl get nodes && \
	kubectl cluster-info

#### For Helm

	helm version --short

### file nginx.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-two-deployment
spec:
  selector:
    matchLabels:
      app: nginx-two
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx-two
    spec:
      containers:
      - name: nginx-two
        image: nginx:1.17-alpine
        ports:
        - containerPort: 80
          name: nginx-pod-port
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-two
  labels:
    app: nginx-two
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: nginx-pod-port
    nodePort: 31112
    protocol: TCP
  selector:
    app: nginx-two
