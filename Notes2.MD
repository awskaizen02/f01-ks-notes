## https://kubernetes.io/docs/concepts/workloads/pods/

pods contain containers in it

pods have 3 types of containers

	containers: This contain application containers and they execute in parallel. This is Mandatory
	
	init containers: These containers will start first and once they finish executing the containers will start. Containers here will start in sequence
	we will have containers which act as pre-condition optional container

	ephemeral containers: for debugging purpose (optional)

I want a httpd container inside a pod

```
---
apiVersion: v1
kind: Pod
metadata: 
  name: sample-1
  labels:
    app: httpd
    purpose: study
    enc: dev
spec:
  containers:
   - name: httpd
     image: httpd:latest
     ports:
       - name: httpd
         protocol: TCP
         containerPort: 80
```

* Pods contd *
Each pod get a unique ip address
each pos has a lifecycle ## https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/

```
---
apiVersion: v1
kind: Pod
metadata: 
  name: sample-2
  labels:
    app: hello-world
    env: dev
spec:
  containers:
   - name: test2
     image: hello-world:latest
```

* kubectl get pods -w

## pod with two containers
* lets create a pod with two contianers

first container will be nginx
second container will be tomcat


```
---
apiVersion: v1
kind: Pod
metadata: 
  name: sample-3
  labels:
    app: web
    purpose: multi-container
spec:
  containers:
   - name: nginx
     image: naginx:latest
     ports:
      - contianerPort: 80
   - name: tomcat
     image: tomcat:latest
     ports:
      - contianerPort: 8080
```

## lets use init contianers with sleep 3s and 4s and nginx

# https://kubernetes.io/docs/concepts/workloads/pods/init-containers/

```
---
apiVersion: v1
kind: Pod
metadata: 
  name: sample-4
  labels:
    app: nginx
    purpose: init-container-demo
spec:
  initcontainers:
   - name: init-0
     image: alpine
     command:
       - sleep
       - "3s"
   - name: init-1
     image: alpine
     command:
       - sleep
       - "4s"
  containers:
   - name: nginx
     image: nginx:latest
     ports:
       - containerPort: 80
```

* pods contd *

## https://kubernetes.io/docs/concepts/workloads/pods/pod-qos/

Kubernetes supports three quality of service classes for containers in pods
	* Guranteed
	* Burstable
	* Best Efforts


The containers running a pod without  mentioning resources it requireds falls under best effort.

The containers  running in a pod with minimum resources required and maximum resources required mentioned fall under 
	# Burstable (If minimum < maximum)
	# Granteed (If minimum == maximum)


## https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/  => for requests and limits

* best-efforts *

```
---
apiVersion: v1
kind: Pod
metadata: 
  name: best-efforts
  labels:
    app: httpd
    purpose: training
    env: dev
spec:
  containers:
   - name: httpd
     image: httpd:latest
     ports:
       - name: httpd
         protocol: TCP
         containerPort: 80
```
scheduler will look into nodes and assigns this pods to any node

** Burstable ** 

### https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-types
 for mentioning resources


```
---
apiVersion: v1
kind: Pod
metadata: 
  name: burstable
  labels:
    app: httpd
    purpose: training
    env: dev
spec:
  containers:
   - name: httpd
     image: httpd:latest
     resources:
       requests:
         cpu: 100m
         memory: 64Mi
       limits:
         cpu: 500m
         memory: 128Mi
     ports:
       - name: httpd
         protocol: TCP
         containerPort: 80
```


## guaranteed ##

```
---
apiVersion: v1
kind: Pod
metadata: 
  name: guranteed
  labels:
    app: httpd
    purpose: training
    env: dev
spec:
  containers:
   - name: httpd
     image: httpd:latest
     resources:
       requests:
         cpu: 100m
         memory: 128Mi
       limits:
         cpu: 100m
         memory: 128Mi
     ports:
       - name: httpd
         protocol: TCP
         containerPort: 80
```

** deployments **

## https://kubernetes.io/docs/concepts/workloads/controllers/deployment/ 

Deployments create replicasets which in turn create pods
Deployments have strategy
	Rolling Update: Roll new pods and kill older ones according to percentages or numbers defined (default is 25%)
	Recreate: Delete all existing and recreate new pods

** sample-1 **

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-1
  labels:
    app: web
spec:
  minReadySeconds: 2
  replicas: 4
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: web
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25% 
  template:
    metadata:
      labels:
        app: web
        env: dev
    spec:
      containers:
      - name: web
        image: nginx:latest
        ports:
        - containerPort: 80
```
