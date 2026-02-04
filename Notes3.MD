Deployments create replicasets which in turn create pods Deployments have strategy Rolling Update: Roll new pods and kill older ones according to percentages or numbers defined (default is 25%) Recreate: Delete all existing and recreate new pods

** sample-1 **
---
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
        image: nginx
        ports:
        - containerPort: 80


with the workloads that support rollout (deployment, daemonset, statefulset) we can use kubectl rollout commands in version 2 will be httpd containers

```
---
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
        image: httpd
        ports:
        - containerPort: 80

commands which we used
	kubectl rollout status deployment sample-1
	kubectl rollout history deployment sample-1
	kubectl rollout undo deployment sample-1 -to-revision=1

Annotations
	Annotations are also keypair. they are designed for external tools to fetch info and add some metadata for tools. K8s has well known annotations with specific purposes


Networking in Kubernetes

Golder rule of k8S networking:  Every podcan talk to every other pod using -ip - without NAT
Every pod:
	has its own IP
	IS routable
	Is isolated via namespace
K8s itself does not implement this, it delegates to CNI
CNI is specification + execution plugins
K8S calls the CNI plugin with pod details, N/W namespace path, Action(ADD/DEL)

## https://kubernetes.io/docs/concepts/services-networking/service/

*K8S service*
-----------
k8s services is used to expose PODS outside cluster

we have 3 types of K8S services

* Cluster IP *
* Node Port *
* Load Balancer *


apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - port: 80
      targetPort: 80
      # Optional field
      # By default and for convenience, the Kubernetes control plane
      # will allocate a port from a range (default: 30000-32767)
      nodePort: 30007
```
==========
Node Port
==========
Node port servide is used to expose pods outside cluster also
when we use Node Port service we can specify Node Port number in service manifest file
if we dont specify Node port number the K8S will assign random node port number for our service

*** Node Port Range: 3000 - 32767 ***

```
---
apiVersion: v1
kind: Service
metadata:
  name: webappsvc
spec:
  type: NodePort
  selector:
    app: webapp # POD lable
  ports:
   - port: 80
     targetPort: 8080
```
* kubectl get svc // Display existing servies *

* kubectl apply -f <svc-maniest.yaml> // create k8s service *

* kubectl get svc // Display exising services *

* kubectl delete -f <svc-namiest.yaml> // to delete the svc *

* kubectl get pod -o wide // to list the pods *

write POD manifest yaml
create POD using kubectl apply command
check POD details and POD logs & in which worker nod it is running
write service Manifest yaml
create service to expose PODS
check service details and Node port number assigned by K8S
enable Node Port number in worker node security Group
Access application using worker node IP

==========
Cluster Ip
==========

When we create PODS, every pod will get unique IP address
we can access POD inside cluster using its IP address
PODS are short lived objects, when POD is recreated its ip will be changed so we cant depend on POD IP to access
to expose POD access with in the cluster we can use Cluster Ip service
Cluster IP will generate one IP address to access our PODS with in cluster

** Cluster IP will not change when PODS are re-created **

```
---
apiVersion: v1
kind: Service
metadata:
  name: webappsvc
spec:
  type: CluterIP
  selector:
    app: webapp # POD lable
  ports:
   - port: 80
     targetPort: 8080
```
==============
Load Balancer
==============

IT is one type of K8S service
It is used to expose our pods outside cluster using Load Balancer
When we use Load Balancer as service type then one Load Balancer will be related in AWS cloud
using LB URL we can access our application
LB will distribute the traffic to multiple worker nodes in Round RObin fashion

```
---
apiVersion: v1
kind: Service
metadata:
  name: webappsvc
spec:
  type: LoadBalancer
  selector:
    app: webapp # POD lable
  ports:
   - port: 80
     targetPort: 8080
```

*** kubectl delete all --all ***
*** kubectl apply -f <manifes.yaml> ***
*** kubectl get pods ***
*** kubectl get svc ***

In service , we can load balancer URL as exterl IP. Using that URL we can access our applications

===============
Pod Life Cycle
===============

* Pod is a smallest building block that we can run in K8S cluster
* we are using kubectl to send request to control plane to create POD
* API server will receive our request and will store request details in ETCD
* Schedular will find un-scheduled PODS and it will schedule in worker nodes
* Node Agent (Kubelet) will see POD schedule and it will fire Docker Engine
* Docker engine will run our container

** PODS are ephemeral (short lived objects) **

### https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/

==========
Name Space
==========

NameSpaces are equal to  packages in the Java
NamwSpaces are used to group our K8S components logically

app pods ---> kaizen-app-ns
db pods ----> kaizen-db-ns

we can create multiple namespaces in K8S cluster

Namespaces are logically isolated with each other

** when we delete namespace all the components created under that namespace will be deleted **

kubectl get ns (or) kubectl get namespace

Kubernetes starts with four initial namespaces:

default: 
	where resources go if no namespace is specified
kube-system: 
	system components (kube-dns, kube-proxy)
kube-public:
	Readable by all users (rarely used)
kube-node-lease:
	Node heartbeat tracking

*** when we create K8s components without using namespace then k8s will conside 'default' namespace for that.***
*** kube-public, kube-system and kube-node-lease namespaces will be used by k8s cluster. we should not create our k8s components in these 3 namespaces. ***

```
apiVersion: v1
kind: Namespace
metadata:
  name: kaizen

```
