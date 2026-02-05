Replication Controller (RC)

## https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/

* It is a K8s component which is used to create PODs
* It will make sure always give no of pod are running for our application

* when POD is crashed / damaged / deleted then RC will create new pod *
* we can scale up and down PODs count using RC


```
---
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginxrc
spec:
  replicas: 3
  selector:
    app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name:nginxsvc
spec:
  type: NodePort
  selector:
    app: webapp
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30085
```

** kubectl apply -f replication.yaml **

** kubectl describe replicationcontrollers/nginx **

** kubectl scale rc nginxrc --replicas 5 **

** kubectl scale rc nginxrc --replicas 2 **

** kubectl delete rc nginxrc  **

----------
ReplicaSet
----------

# https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/

ReplicaSet is replacement for ReplicationController ( It is nextgen component in K8S)
ReplicaSet is also used to manage POD lifecycle
ReplicaSet also will maintain given No of pod always

** the only difference between ReplicationController and ReplicaSet is 'selector' **

## ReplicationController supports Equality Based Selector

selector:
  app: demoapp

## ReplicaSet Supports Set Based Selector

select:
  matchLables:
    app: demoapp
    version: v1
    type: fe


```

---
apiVersion: v1
kind: ReplicaSet
metadata:
  name: nginxrs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name:nginxsvc
spec:
  type: NodePort
  selector:
    app: webapp
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30085
```

**kubectl  apply -f rs.yaml**
**kubectl get rs**
**kubectl get svc**
**kubectl scale rs nginxrs --replicas 5**
**kubectl scale rs nginxrs --replicas 2**
**kubectl delete rs nginxrs**


----------
Deployment
----------

Deployment is one of the K8S resource / component
Deployment is the ost recommended approach to deploy our applications in K8s cluster
using Deployment we can scale up and we can scale down our pods
Deployment supports roll out and roll back

**To deploy latest code we have to delete EC and RS then PODs will be deleted and application will be down**

## https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
  annotations:
    kuberbetes.io/change-cause: "nginx"
spec:
  minReadySeconds: 5
  replicas: 3
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: nginx
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
  template:
    metadata:
      labels:
        app: nginx
        env: dev
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: dep-svc-external
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - name: web
      port: 80
      targetPort: 80
      protocol: TCP
      nodePort: 30009

```
---------------------
Deployment Strategies
---------------------

we have below Deployment stratagies

*Recreate*
*Rolling update*

Recrate means it will delete all the existing pods and it will create new pods (downtime will be there)

RollingUpdate starategy means it will delete the pod create new pod one by one.
**if we don't specify deployment strategy, then by default it will consider as RollingUpdate**


**kubectl apply -f deployment.yaml**
**kubectl get pods -o wide**
**kubectl get svc**
**kubectl get deployment**
**kubectl scale deployment nginx-deployment --replicas 5**
**kubectl scale deployment nginx-deployment --replicas 3**
**kubectl delete deployment nginx-deployment**

## https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/

DaemonSet ensure a pod is schedule on eery node ( on subset of nodes)
DaemonSet support rollingupdates
Lets run a daemonSet with alpine pod with sleep 1d
DaemonSet generally run agent kind of applications (log agent, sync agents)


```
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: sample-1
spec:
  minReadySeconds: 2
  selector:
    matchLabels:
      app: logagent
  template:
    metadata:
      labels:
        app: logagent
    spec:
      containers:
      - name: logagent
        image: alpine:latest
        command:
          - sleep
          - "86400s"

``` 
