## Get K8S cluster detauls

```
kubectl cluster-info

Kubernetes control plane is running at https://kubernetes.docker.internal:6443
CoreDNS is running at https://kubernetes.docker.internal:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

## Get K8S nodes details

```
kubectl get nodes

NAME             STATUS   ROLES                  AGE     VERSION
docker-desktop   Ready    control-plane,master   3h13m   v1.21.1
```

## Get K8S namespaces details

```
kubectl get namespaces

NAME                   STATUS   AGE
default                Active   3h13m
kube-node-lease        Active   3h13m
kube-public            Active   3h13m
kube-system            Active   3h13m
kubernetes-dashboard   Active   3h6m
```

## Create a new K8S deployment base on a standard nginx image

*IMPORTANT*

* deployment is an abstraction layer that manages a replicaset
* replicaset is an abstraction layer that manages a pod
* pod is an abstraction layer that manages a container (for example a Docker container)
* In Kubernetes, just work with deployments, don't take care of other layers, K8S will do that for you

```
kubectl create deployment nginx-depl --image=nginx
```

## Get deployment details

```
kubectl get deployment
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
nginx-depl   1/1     1            1           98m
```

# GET REPLICASET DETAILS

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl get replicaset
NAME                    DESIRED   CURRENT   READY   AGE
nginx-depl-5c8bf76b5b   1         1         1       98m


# GET POD DETAILS

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl get pod
NAME                          READY   STATUS    RESTARTS   AGE
nginx-depl-5c8bf76b5b-5gg2q   1/1     Running   0          99m

# NAME OF POD IS nginx-depl-5c8bf76b5b-5gg2q WHERE nginx-depl IS THE NAME OF THE DEPLOYMENT, 5c8bf76b5b IS THE ID OF THE REPLICASET AND 5gg2q IS THE ID OF THE POD


# MAKE A CHANGE TO THE DEPLOYMENT BY FORCING NGINX IMAGE VERSION TO 1.20. CHANGE LINE "- image: nginx" TO "-image: nginx:1.20" THAN SAVE AND EXIT FROM THE EDITOR

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl edit deployment nginx-depl
deployment.apps/nginx-depl edited


# GET AGAIN DEPLOYMENT DETAILS. NOTHING HAS CHANGED

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl get deployments
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
nginx-depl   1/1     1            1           103m


# GET AGAIN REPLICASET DETAILS. THE OLD REPLICASET IS NOT USED ANYMORE AND A NEW REPLICASET HAS BEEN CREATED INSTEAD WITH 1 RUNNING POD

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl get replicaset
NAME                    DESIRED   CURRENT   READY   AGE
nginx-depl-58f65d4b6f   1         1         1       17s
nginx-depl-5c8bf76b5b   0         0         0       104m


# GET AGAIN POD DETAILS. NOW THERE'S A NEW RUNNING POD ON THE NEW REPLICASET (58F...)

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl get pod
NAME                          READY   STATUS    RESTARTS   AGE
nginx-depl-58f65d4b6f-wr8qp   1/1     Running   0          29s


# CREATE A NEW DEPLOYMENT WITH MONGO IMAGE

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl create deployment mongo-depl --image=mongo-depl
deployment.apps/mongo-depl created


# GET PODS DETAILS. NEW MONGO POD IS STARTING

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl get pods
NAME                          READY   STATUS              RESTARTS   AGE
mongo-depl-5fd6b7d4b4-k25zw   0/1     ContainerCreating   0          6s
nginx-depl-58f65d4b6f-wr8qp   1/1     Running             0          24m


# GET LOGS OF THE MONGO POD TO SEE WHAT'S HAPPENING. NOTE THAT AN ERROR IS SHOWN STATING THAT MONGO CONTAINER IS NOT RUNNING YET

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl logs mongo-depl-5fd6b7d4b4-k25zw
Error from server (BadRequest): container "mongo" in pod "mongo-depl-5fd6b7d4b4-k25zw" is waiting to start: ContainerCreating


# WHILE WE WAIT A LITTLE BIT, LET'S TAKE A LOOK AT THE POD STATUS WITH MORE DETAILS

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl describe pod mongo-depl-5fd6b7d4b4-k25zw

Name:         mongo-depl-5fd6b7d4b4-k25zw
Namespace:    default
Priority:     0
Node:         docker-desktop/192.168.65.4
Start Time:   Fri, 30 Jul 2021 14:35:12 +0200
Labels:       app=mongo-depl
              pod-template-hash=5fd6b7d4b4
Annotations:  <none>
Status:       Running
IP:           10.1.0.11
IPs:
  IP:           10.1.0.11
Controlled By:  ReplicaSet/mongo-depl-5fd6b7d4b4
Containers:
  mongo:
    Container ID:   docker://96b73b738753d52d63788671f7209f37cdb06ed349b4ff35f5f29e0b5ccd3420
    Image:          mongo
    Image ID:       docker-pullable://mongo@sha256:d78c7ace6822297a7e1c7076eb9a7560a81a6ef856ab8d9cde5d18438ca9e8bf
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Fri, 30 Jul 2021 14:36:20 +0200
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-dtcww (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-dtcww:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  111s  default-scheduler  Successfully assigned default/mongo-depl-5fd6b7d4b4-k25zw to docker-desktop
  Normal  Pulling    110s  kubelet            Pulling image "mongo"
  Normal  Pulled     44s   kubelet            Successfully pulled image "mongo" in 1m6.3158403s
  Normal  Created    43s   kubelet            Created container mongo
  Normal  Started    43s   kubelet            Started container mongo
  

# OK, NOW THE CONTAINER IS STARTED, SO IT SHOULD BE POSSIBLE TO SEE THE POD'S LOGS

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl logs mongo-depl-5fd6b7d4b4-k25zw

{"t":{"$date":"2021-07-30T12:36:20.191+00:00"},"s":"I",  "c":"NETWORK",  "id":4915701, "ctx":"-","msg":"Initialized wire specification","attr":{"spec":{"incomingExternalClient":{"minWireVersion":0,"maxWireVersion":13},"incomingInternalClient":{"minWireVersion":0,"maxWireVersion":13},"outgoing":{"minWireVersion":0,"maxWireVersion":13},"isInternalClient":true}}}
{"t":{"$date":"2021-07-30T12:36:20.192+00:00"},"s":"I",  "c":"CONTROL",  "id":23285,   "ctx":"main","msg":"Automatically disabling TLS 1.0, to force-enable TLS 1.0 specify --sslDisabledProtocols 'none'"}
{"t":{"$date":"2021-07-30T12:36:20.193+00:00"},"s":"W",  "c":"ASIO",     "id":22601,   "ctx":"main","msg":"No TransportLayer configured during NetworkInterface startup"}
{"t":{"$date":"2021-07-30T12:36:20.193+00:00"},"s":"I",  "c":"NETWORK",  "id":4648601, "ctx":"main","msg":"Implicit TCP FastOpen unavailable. If TCP FastOpen is required, set tcpFastOpenServer, tcpFastOpenClient, and tcpFastOpenQueueSize."}
{"t":{"$date":"2021-07-30T12:36:20.196+00:00"},"s":"W",  "c":"ASIO",     "id":22601,   "ctx":"main","msg":"No TransportLayer configured during NetworkInterface startup"}
...
{"t":{"$date":"2021-07-30T12:36:20.631+00:00"},"s":"I",  "c":"NETWORK",  "id":23015,   "ctx":"listener","msg":"Listening on","attr":{"address":"/tmp/mongodb-27017.sock"}}
{"t":{"$date":"2021-07-30T12:36:20.631+00:00"},"s":"I",  "c":"NETWORK",  "id":23015,   "ctx":"listener","msg":"Listening on","attr":{"address":"0.0.0.0"}}
{"t":{"$date":"2021-07-30T12:36:20.631+00:00"},"s":"I",  "c":"NETWORK",  "id":23016,   "ctx":"listener","msg":"Waiting for connections","attr":{"port":27017,"ssl":"off"}}
...


# NEED FURTHER DEBUGGING? LOGIN INTO THE MONGODB POD

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl exec -it mongo-depl-5fd6b7d4b4-k25zw -- /bin/bash

root@mongo-depl-5fd6b7d4b4-k25zw:/# ls
bin  boot  data  dev  docker-entrypoint-initdb.d  etc  home  js-yaml.js  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@mongo-depl-5fd6b7d4b4-k25zw:/# exit
simone@ITBLQ1LPTL0052:~/kubernetes$


# DELETE A DEPLOYMENT

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl delete deployment mongo-depl
deployment.apps "mongo-depl" deleted

# DEPLOYMENT mongo-depl HAS GONE

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl get deployment
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
nginx-depl   1/1     1            1           143m

# MONGO REPLICASET HAS GONE, ONLY NGINX REPLICASETS ARE STILL HERE

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl get replicaset
NAME                    DESIRED   CURRENT   READY   AGE
nginx-depl-58f65d4b6f   1         1         1       40m
nginx-depl-5c8bf76b5b   0         0         0       143m

# MONGO POD HAS GONE

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl get pod
NAME                          READY   STATUS    RESTARTS   AGE
nginx-depl-58f65d4b6f-wr8qp   1/1     Running   0          40m


# DELETE ALSO NGINX DEPLOYMENT

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl delete deployment nginx-depl
deployment.apps "nginx-depl" deleted

# NGINX POD IS TERMINATING

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl get pod
NAME                          READY   STATUS        RESTARTS   AGE
nginx-depl-58f65d4b6f-wr8qp   0/1     Terminating   0          40m

# ALL REPLICASETS HAVE GONE

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl get replicaset
No resources found in default namespace.

# ALL PODS ARE GONE

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl get pod
No resources found in default namespace.


#
# NOTE: ALL CRUD OPERATIONS ARE PERFORMED BY THE USER ON DEPLOYMENTS. K8S TAKES CARE OF CORRESPONDING OPERATIONS ON OTHER LAYERS AUTOMATICALLY
#


# INSTEAD OF MANAGING DEPLOYMENT PARAMETERS THROUGH COMMAND LINE, YOU CAN CREATE A TEXT FILE IN YAML FORMAT AND PUSH
# ALL THE NECESSARY CONFIGURATION IN IT, THEN RUN KUBECTL APPLY COMMAND BY SPECIFYING WITH -F THE FILE NAME WITH THE
# DEPLOYMENT'S CONFIGURATION TO INSTANTIATE IT

# EXAMPLE DEPLOYMENT FILE FOR NGINX

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:							# deployment specification
  selector:
    matchLabels:
      app: nginx
  replicas: 1					# tells deployment to run 1 pod matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:						# pod specification
      containers:
      - name: nginx
        image: nginx:1.20
        ports:
        - containerPort: 80


# APPLY THE NEW DEPLOYMENT AND SEE THAT DEPLOYMENT, REPLICASET AND POD ARE CREATED, WITH 1 RUNNING NGINX CONTAINER

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl apply -f nginx-deployment.yaml
deployment.apps/nginx-deployment created

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   1/1     1            1           7s

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl get replicaset
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-6897679c4b   1         1         1       15s

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl get pod
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6897679c4b-qkb76   1/1     Running   0          22s


# FROM THE CONFIGURATION FILE IT'S POSSIBLE TO ADD, CHANGE OR DELETE ANY PARAMETER, THEN SAVING THE FILE AND APPLYING AGAIN WITH KUBECTL WILL PERFORM THE REQUESTED CHANGES
# FOR EXAMPLE, IF WE SET REPLICASET FROM 1 TO 2 AND WE APPLY AGAIN, THE NUMBER OF RUNNING PODS WILL BECOME 2

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl apply -f nginx-deployment.yaml
deployment.apps/nginx-deployment configured

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   2/2     2            2           7m24s

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl get pod
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6897679c4b-42lph   1/1     Running   0          14s
nginx-deployment-6897679c4b-qkb76   1/1     Running   0          7m31s


# DESTROY THE DEPLOYMENT REFERENCING IT'S CONFIGURATION FILE

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl delete -f nginx-deployment.yaml
deployment.apps "nginx-deployment" deleted

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl get deployment
No resources found in default namespace.

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl get pod
No resources found in default namespace.


# REUSE NGINX DEPLOYMENT BY CHANGING NUMBER OF PODS TO 2 INSTANCES LISTENING ON PORT 8080
# THEN ADD A SERVICE LINKED TO NGINX DEPLOYMENT TO FORWARD INCOMING TRAFFIC ON PORT 80 TO PORT 8080 OF THE 2 NGINX PODS

nginx-deployment.yaml:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 1 pod matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.20
        ports:
        - containerPort: 8080


nginx-service.yaml:

apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP

# THE SERVICE IS CONNECTED TO THE DEPLOYMENT THROUGH THE SELECTOR app:nginx

# APPLY THE DEPLOYMENT AND THE SERVICE

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl apply -f nginx-deployment.yaml
simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl apply -f nginx-service.yaml

# 2 PODS ARE RUNNING WITH INTERNAL IP

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl get pods -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP          NODE             NOMINATED NODE   READINESS GATES
nginx-deployment-85d6c5d8d6-hvpjf   1/1     Running   0          44m   10.1.0.17   docker-desktop   <none>           <none>
nginx-deployment-85d6c5d8d6-nxfjt   1/1     Running   0          44m   10.1.0.16   docker-desktop   <none>           <none>

# THE NEW NGINX SERVICE HAS BEEN CREATED

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl get service
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes      ClusterIP   10.96.0.1      <none>        443/TCP   6h5m
nginx-service   ClusterIP   10.99.21.187   <none>        80/TCP    36m          <--------- new service

# DESCRIBE THE NEW SERVICE. IT'S LISTENING ON PORT 80 AND REFERENCING THE 2 PODS LISTENING ON PORT 8080

simone@ITBLQ1LPTL0052:~/kubernetes$ kubectl describe service nginx-service
Name:              nginx-service
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=nginx
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.99.21.187
IPs:               10.99.21.187
Port:              <unset>  80/TCP
TargetPort:        8080/TCP
Endpoints:         10.1.0.16:8080,10.1.0.17:8080
Session Affinity:  None
Events:            <none>