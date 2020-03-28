---
layout: post
title: K8s study notes
key: 20191116
tags: K8s notes
---

## Introduction

This is the second part of kubernetes study notes. History:

1. From google Brog system

### Resource

#### scope

1. Namespace level
   1. eg: kubeadmin, k8s, kube-system

2. Cluster level
   1. eg: role

3. Metadata level
   1. eg: HPA


#### types

1. workload
   1. Pod
   2. Deployment
   3. RC, RS
   4. DaemonSet
   5. Job / CronJob

2. service discovery and loadbalance
   1. service
   2. ingress

3. storage and configuration
   1. volume
   2. CSI

4. special
   1. secret
   2. configMap
   3. downwardAPI : outside info into containers


### Pod

Smallest resource in k8s

#### compulsory fields
1. apiVersion ( string ): eg v1, app/v1
2. kind ( string ): eg Pod, Deployment
3. metadata ( object )
   1. name ( string )

4. containers ( object[] )
   1. name ( string )
   2. image ( string )

#### important fields
1. metadata
   1. namespace
2. containers
   1. ImagePullPolicy
      1. IfNotPresent 
      2. Never
      3. Always
   2. Ports
   3. Command
   4. WorkingDir
   
```yaml
apiVersion: v1
kind: Pod
metadata:
    name: nginx-pod
spec:
    containers:
        - name: nginx-pod
          image: nginx
          ports:
            - containerPort: 80
```

### Lifecycle

bc -> init Cs -> Main C ( start -> liveness ( readness ) -> stop )

![life cycle](/assets/img/k8s/pod_lifecycle.png) 

#### Init Container

1. under linux namespace so can access resource which main container can not
2. start one by one ( previous full stop ) after network and volume init ( basic container jobs )
3. If fails, the Pod will fail

#### Main container Probe ( iterant )

1. Actions
   1. ExecAction: exec command inside container
   2. TCPSocketAction: check ports open or not
   3. HTTPGetAction: check return code between 2xx and 3xx

2. Types
   1. LivenessProbe: check container is running or not if fails Pod will fail
   2. ReadnessProbe: check service ready or not if fails all the services will delete the Pod ip


```yaml
apiVersion: v1
kind: Pod
metadata:
    name: nginx-pod
spec:
    containers:
        - name: nginx-pod
          image: nginx
          ports:
            - containerPort: 80
          readnessProbe:
            httpGet:
                port: 80
                path: /index.html
            initialDelaySeconds: 1
            periodSeconds: 3

```

#### Status
1. Pending: Pod Yaml been submitted to k8s. API objects been created and saved into etcd. But container not been created for some reason. eg: scheduler
2. Running: Pod has been scheduled and bind to a node. All containers inside Pod are created and ready to service.
3. Succeeded: All container inside Pod runs successfully, usually for one time job
4. Failed: At least one container inside Pod fails
5. Unknown: Pod stats not update by kubelet to api-server, usually means communication between master and nodes issue


### Controller

Controller -> state machine

1. ReplicationController And ReplicaSet
2. Development
3. DaemonSet
4. StateFulSet
5. Job / Cronjob
6. Horizontal Pod Autoscaling

#### RC and RS
1. RC -> RS
2. RS support selector label
3. maintain user defined replic number

#### Development
1. support declarative statement ( apply ) to create Pod and RS, NB: Deployment -> RS -> Pods
2. roll-update and rollback
   1. update -> create a new rs -> decrease old rs replic number and increase new rs replic number until old one replic number become zero
   2. rollback reverse process as update
3. scale up and scale down
4. pause and continue Deployment

*create*
> `kubectl apply -f xxx.yaml --record`

*scale*
> `kubectl scale deployment nameOfDeployment --replicas 10`

*autoscale*
> `kubectl autoscale nameOfDeployment --min 2 --max 15 --cpu-percent=80`

*update image*
> `kubectl set image deployment/nameOfDeployment nginx=nginx:1.9.1`

*rollback*
> `kubectl rollout undo deployment/nameOfDeployment`


#### DaemonSet

ensure all ( some ) nodes have at lease one running

eg:
1. storage: `ceph` `glusterd`
2. logs: `fluentd` `logstash`
3. monitor `collectd` `new replic` `gmond`


#### Job / CronJob

CronJob -> run a Job in certain time

*CronJob spec*
1. `spec.scheduler` - same as Cron min hour day month weekday 
2. `spec.jobTemplate` - Job template


#### StateFulSet
1. persistent storage
2. persistent network identifiers
3. deploy in order from 0 to n-1 and the previous pod must be ready and running
4. delete in order from n-1 to 0


#### Horizontal Pod Autoscaling

   
### Services

1. With **Label selector**, match a group of pods to provide outside access.
2. Only support 4 layers ( IP and port )of loadbalance not 7 layers, but with ingress can support 7 layers

#### Types
1. ClusterIP (default): assign cluster internal ip ( restrict access within cluster ) to service
2. NodePort: Bind a node port above Cluster IP to service for each node, port number > 30000
3. LoadBalance: use cloud-provider add a loadbalance to NodePort( NodeIP:NodePort ), so it can be accessed outside cluster

```yaml
apiVersion: v1
kind: Service
metadata:
    name: nginx-svc
spec:
    type: LoadBalancer
    selector:
        app: nginx
    ports:
        - name: http
          port: 8081
          targetPort: 80
    externalIPs:
        - 192.168.99.102 # minikube IP, usually cloud provider
```

4. ExternalName: import out-cluster service into cluster, so it can be access

#### Kube-proxy and VIP

Each node has kube-proxy running inside and will provide a virtual ip for each service. 1.1 `iptables` -> 1.14 `ipvs`

*ipvs*
1. kube-proxy will regularly monitor `services` and `endpoints` then call `netlink` to create  or update ipvs rules.
2. ipvs can provide multiple loadbalance algorithm
   1. rr - round robin
   2. lc - least connection
   3. dh - destination hash
   4. sh - source hash
   5. sed - least delay
   6. nq - not queued
3. if install IPVS module will fallback to iptables

### Ingress

ingress -> nodePort

[https://kubernetes.github.io/ingress-nginx/](https://kubernetes.github.io/ingress-nginx/)

### configMap

```sh
kubectl create configmap nameOfconfigMap --from-file=PathToFile
```

