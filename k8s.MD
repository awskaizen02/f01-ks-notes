“Kubernetes” is the Greek word for a ship’s captain.

What is Kubernetes
	- orchestration platform
	- To manage containers
	- Developed by Google using GO language
	- Google donated K8s to CNCF
	- K8S first version released in 2015
	- it is free & opensource

Note: K8S will use Docker internally. using k8s we will manage Docker containers

Docker Swarm vs K8S
	- Docker swarm doesn't have autoscaling (scaling is manual process)
	- K8S support Auto scaling
	- for production deployment K8S is highly recommended
	- K8S is replacement for Docker swarm

AS: increasing and decreasing containers count based on incoming request for our app.

what is cluster
	- Group of servers
	- Master Node(s)
	- Worker Node(s)
	- Devops Engineer / Developer will give the task to K8S master Node
	- Master Node will manage worker nodes
	- Master node will Schedule tasks to worker nodes
	- our containers will be created in worker Nodes
	- using cluster we can achieve High Availability

# https://www.cncf.io/phippy/

what does kK8s do:
	it can create pods
	it can create clones (replication controllers)
	it can use labels
	it can enable communications b/w pods (services)
	it can enable storage (volumes)


pod
replicaset
secrets
deployments
Daemonset
Ingress
Cronjobs
CRDs


# https://kubernetes.io/images/docs/kubernetes-cluster-architecture.svg	

K8S Architecture
	- Control Plane / Master Node / Manager Node
		- API server
		- Schedular
		- Control Manager
		- ETCD
	- Node Components
		- Kubelet
		- Container Runtime
		- kube-proxy

API server : This is responsible for communication
ECTD       : This is where the data is stored (memory of K8S cluster)
Schedular  : Tis is responsible for creating any new container (pod)
Control    : This is not one component but internally many components	
cloud controller manager (optional): This exists on K8S hosted by cloud providers (AKS,EKS,GKE)
		this is aware of cloud specifics

kubelet : This is agent of K8S master. This receives instructions and executes them
kube-proxy: This is responsible for communication (part of networking)
container runtime: this is container engine

Interfaces
	K8S uses 3 major interfaces to support multiple options/vendors

  Containers: CRI (Container Runtime Interface)
  networking: CNI (Container Network Interface)
  storage   : CSI (Container Storage Interface)

# https://kubernetes.io/docs/setup/learning-environment/

K8s bringup options:
Laptop/Desktop : For dev purpose
  minikube
  kind 	

# https://kubernetes.io/docs/setup/production-environment/tools/
Physical/VIrtual Machines:
  Kube-adm
  kube-spray
  kops

Managed K8S on cloud
	AKS : Azure Kubernetes Service
	EKS : Elastic Kubernetes service
	GKE : Google Kubernetes Engine

# https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html

we focus on EKS for learning purpose
install AWS CLI on ec2 instance
provide role to access ec2 instance, LB, Cloud formation (provide Admin access)



curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.34.2/2025-11-13/bin/linux/amd64/kubectl
chmod +x ./kubectl
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH
kubectl version --client

# https://docs.aws.amazon.com/eks/latest/eksctl/installation.html

# for ARM systems, set ARCH to: `arm64`, `armv6` or `armv7`
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH

curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"

# (Optional) Verify checksum
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check

tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz

sudo install -m 0755 /tmp/eksctl /usr/local/bin && rm /tmp/eksctl


## Pod
This is atomic object/workload which K8s creates
Pod contains container(s) in it
A pod expects container to be in running state, if it exits pod restarts the container
if the pod is restarting the container continuously this state is referred as crash loopbackoff
A pod can contain one or more containers
In pod the first container where we run the application is referred as maincar and all other containers are referred as sidecar's


we used YAML file Manifest
we have the following structure for manifest

# https://kubernetes.io/docs/concepts/workloads/pods/

apiVersion:
kind:
metadata:
spec:

# https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.35/#api-groups
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-example
spec:
  containers:
  - name: ubuntu
    image: ubuntu:trusty
    command: ["echo"]
    args: ["Hello World"]
```
K8s API Groups: <group>/<version>
        core => <version>
        apps => apps/v1, batch/v1

apiVersion - Which version of the Kubernetes API you're using to create this object

kind - What kind of object you want to create. which we are creating (nouns of k8s) On the resources we can generally perform 4 basic operations
	Get
	Create
	Update
	Delete

metadata - Data that helps uniquely identify the object, including a name string, UID, and optional namespace

spec - What state you desire for the object

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-example
  labels:
    app: nginx
spec:
  containers:
   - name: nginx
     image: nginx:1.14.2
     ports:
      - containerPort: 80

```
https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/

apiVersion: apps/v1
kind: DaemonSet

# https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/

apiVersion: v1
kind: PersistentVolume

# https://kubernetes.io/docs/tasks/configure-pod-container/
apiVersion: v1
kind: Pod

https://kubernetes.io/docs/concepts/services-networking/service/
apiVersion: v1
kind: Service

samplepod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: httpd
spec:
  containers:
   - name: web
     image: httpd
     ports:
      - containerPort: 80
```  	
