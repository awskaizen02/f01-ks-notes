## https://kubernetes.io/docs/concepts/configuration/configmap/

--------------------
Config map & Secrets
--------------------

**For every applications Multiple environments will be available for testing purpose**
	- Dev
	- SIT
	- UAT
	- PILOT
**Once application testing completed in all above environments then it will be deployed into production environment (Live Environment)**

Config Map & secret concepts are used to avoid hard coded propertiers in the application
Config Map is used stored data in key-value (non-confidential)
config Map allows us to de-couple application properties from docker images, do that our application can be deployed into any environment without making any changes for our docker images.

```
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-db
  labels:
    storage: game-db-storage
data:
  MYSQL_ROOT_PASSWORD: admin123
  MYSQL_DATABASE: mydb
  MYSQL_USER: myuser
  mysql_password: MYPASSWORD

---
apiVersion: V1
kind: Pod
metadata:
  name: test-pod
  labels:
    app: alpine-demo
spec:
  containers:
    - name: alpine
      image: alpine:latest
      command: ["sh", "-c", "echo 'Hello from Alpine Pod'; sleep 3600"]
      volumeMounts:
        - name: test-vol
          mountPath: /test-demo
  volumes:
    - name: test-vol
      configMap:
        name: game-db
```
# https://kubernetes.io/docs/concepts/storage/volumes/

#https://www.base64encode.org/

```
---
apiVersion: v1
kind: Secret
metadata:
  name: web-db-config-secret
  labels:
    secret: web-db-config-secret
data:
  DB_USER_NAME_VALUE: YWRtaW4=
  DB_PASSWORD_VALUE: YWRtaW4=
type: Opaque

```

kubectl get secret

kubectl apply -f sec.yaml


**configMap = url + svc + schema + port**
**secret    = username & password (or) tokens **

statefull application vs stateless application

*stateless applications cant store the data permanently. for every request thet will get new data and it will process that data. every request will be treated as a new request*

*Stateful application means they will store data permanently and they will keep track of data*

# https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/

to run stateful containers in K8S cluster we will use statfulset concept
statefulset will assign a sticky identity for each pod starting with zero (0)

in stateful set primary and secondary pods will be created
once primary pod is created then by copying primry pod data secondary pod will be created

**New Pod will be created by copying previous pod data. If previous pod is pending state then new pod will not be created**

in statefulset, pod will be deleted in revers order (last to first)


# https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/

Deployment -> stateless app [deploy Frontend use statefulset]

create pod name randam-name assign

Deployment will create the pods with random ids
Deployment will scale down pods in random order
Deployment pods are stateless pods

Statefulset --> statefull app [ deploy DB use statefulset]

Statefulset will create the PODs with sticky identity
statefulset will scale down pods in reverse order
statefulset pods are  statefull

-> For application deployment we will use 'DEPLOYMENT'
-> For database deployment we will use 'STATEFULSET'

A statefulset is the kubernetes controller used to run the stateful application as containers(PODS) in the k8s cluster
statefulset assign a sticky identity an ordinal number starting from 0 to each pos instead of assign random ID's for eac replica pod

a new pod is created by cloning the previous Pod's data. If the revious Pod s in the pending state, then the new Pod will ot be created. if you delete a pod, it will delete the pod in revers order, not in ranom order

Creation: Pods are created sequentially (0, 1, 2,...)

Scaling: New pods get the next ordinal number

Deletion: Pods are terminated in reverse order

Updates: Pods are updated in reverse ordinal order

Storage: Each pod gets its own PV based on the volume claim template

kubectl apply -f name.yml

kubectl get pods

kubectl  get deployment

kubectl scale deployment <deployment-name>  --replicas 2

kubectl get pods

kubectl scale deployment <deployment-name>  --replicas 1

kubectl get pods

kubectl get all --all

 
kubectl delete deployment
