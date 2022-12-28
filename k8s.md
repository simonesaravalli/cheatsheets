## Get K8S cluster details

```
kubectl cluster-info
```

The output will be:

```
Kubernetes control plane is running at https://kubernetes.docker.internal:6443
CoreDNS is running at https://kubernetes.docker.internal:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

## Get K8S nodes details

```
kubectl get nodes
```

The output will be:

```
NAME             STATUS   ROLES                  AGE     VERSION
docker-desktop   Ready    control-plane,master   3h13m   v1.21.1
```

## Get K8S namespaces details

```
kubectl get namespaces
```

The output on a fresh cluster will be:

```
NAME                   STATUS   AGE
default                Active   3h13m
kube-node-lease        Active   3h13m
kube-public            Active   3h13m
kube-system            Active   3h13m
kubernetes-dashboard   Active   3h6m
```

## Create a new K8S deployment base on a standard nginx image

**IMPORTANT**

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
```

The output will be:

```
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
nginx-depl   1/1     1            1           98m
```

## Get replicaset details

```
kubectl get replicaset
```

The output will be:

```
NAME                    DESIRED   CURRENT   READY   AGE
nginx-depl-5c8bf76b5b   1         1         1       98m
```

## Get pod detauls

```
kubectl get pod
```

The output will be:

```
NAME                          READY   STATUS    RESTARTS   AGE
nginx-depl-5c8bf76b5b-5gg2q   1/1     Running   0          99m
```

The name of the pod is *nginx-depl-5c8bf76b5b-5gg2q* where

* nginx-depl: is the name of the deployment
* 5c8bf76b5b: is the ID of the replicaset
* 5gg2q: is the ID of the pod

## Make a change to the deployment

Now we make a change to the deployment of nginx forcing K8S to use nginx image version 1.20. Run kubectl edit and change line "- image: nginx" to "-image: nginx:1.20", then save and exit the editor>

```
kubectl edit deployment nginx-depl
```

The output will be:

```
deployment.apps/nginx-depl edited
```

Get again the details of the deployment, nothing changed:

```
kubectl get deployments

NAME         READY   UP-TO-DATE   AVAILABLE   AGE
nginx-depl   1/1     1            1           103m
```

Get again replicaset detauls, the old replicaset is not in use anymore and a new one has been created with 1 pod running

```
kubectl get replicaset

NAME                    DESIRED   CURRENT   READY   AGE
nginx-depl-58f65d4b6f   1         1         1       17s
nginx-depl-5c8bf76b5b   0         0         0       104m
```

Get again pod detauls, now there's a new running pod attached to the new replicaset (58f...)

```
kubectl get pod

NAME                          READY   STATUS    RESTARTS   AGE
nginx-depl-58f65d4b6f-wr8qp   1/1     Running   0          29s
```

## Create a deployment for MongoDB

```
kubectl create deployment mongo-depl --image=mongo-depl
```

## Get pod details

```
kubectl get pods

NAME                          READY   STATUS              RESTARTS   AGE
mongo-depl-5fd6b7d4b4-k25zw   0/1     ContainerCreating   0          6s
nginx-depl-58f65d4b6f-wr8qp   1/1     Running             0          24m
```

## Get logs of the MongoDB pod

Getting logs of the MongoDB pod too early will end in an error because the pod is not running yet

```
kubectl logs mongo-depl-5fd6b7d4b4-k25zw
```

It will output:

```
Error from server (BadRequest): container "mongo" in pod "mongo-depl-5fd6b7d4b4-k25zw" is waiting to start: ContainerCreating
```

While we wait a bit, let's take a look at the pod status with more details:

```
kubectl describe pod mongo-depl-5fd6b7d4b4-k25zw
```

It will output:

```
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
```

Ok now the pod is started, so we can look at its logs:

```
kubectl logs mongo-depl-5fd6b7d4b4-k25zw
```

It will ooutput:

```
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
```

## Login into a running pod

```
kubectl exec -it mongo-depl-5fd6b7d4b4-k25zw -- /bin/bash

root@mongo-depl-5fd6b7d4b4-k25zw:/# ls
bin  boot  data  dev  docker-entrypoint-initdb.d  etc  home  js-yaml.js  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

## Delete a deployment

```
kubectl delete deployment mongo-depl
deployment.apps "mongo-depl" deleted
```

Deployment mongo-depl has gone:

```
kubectl get deployment
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
nginx-depl   1/1     1            1           143m
```

Mongo replicaset has gone:

```
kubectl get replicaset
NAME                    DESIRED   CURRENT   READY   AGE
nginx-depl-58f65d4b6f   1         1         1       40m
nginx-depl-5c8bf76b5b   0         0         0       143m
```

Mongo pod has gone:

```
kubectl get pod
NAME                          READY   STATUS    RESTARTS   AGE
nginx-depl-58f65d4b6f-wr8qp   1/1     Running   0          40m
```

Delete also nginx deployment

```
kubectl delete deployment nginx-depl
deployment.apps "nginx-depl" deleted
```

**IMPORTANT**

All CRUD operations are performed by the user on deployments. K8S then takes care of corresponding operations on other layers automatically.

## yml deployments

Instead of managing deployments parameters from the cli, you can create a yml file with all the necessary configuration, then run kubectl apply specifyig the name of the deployment with the -f flag.

This is an example of a deployment yml file for nginx:

```
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
```

Now apply this deployment and see all the resources created by K8S:

```
kubectl apply -f nginx-deployment.yaml
deployment.apps/nginx-deployment created
```

```
kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   1/1     1            1           7s
```

```
kubectl get replicaset
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-6897679c4b   1         1         1       15s
```

```
kubectl get pod
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6897679c4b-qkb76   1/1     Running   0          22s
```

From the configuration file it's possible to add, change, or delete any parameter. Saving the file and applying again with kubectl will perform the requested changes. For example, if we set replicatet from 1 to 2 and we apply the deployment again, the number of running pods will go from 1 to 2:

```
kubectl apply -f nginx-deployment.yaml
deployment.apps/nginx-deployment configured
```

```
kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   2/2     2            2           7m24s
```

```
kubectl get pod
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6897679c4b-42lph   1/1     Running   0          14s
nginx-deployment-6897679c4b-qkb76   1/1     Running   0          7m31s
```

Now destroy the deployment by referencing its configuration file

```
kubectl delete -f nginx-deployment.yaml
deployment.apps "nginx-deployment" deleted
```

```
kubectl get deployment
No resources found in default namespace.
```

```
kubectl get pod
No resources found in default namespace.
```

Now we reuse the yml of the nginx deployment and we change the number of pods to 2 instances, listening on port 8080. Then we add a service linked to the nginx deployment to forward incoming traffic on port 80 to port 8080 of the 2 pods:

nginx-deployment.yaml:

```
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
```

nginx-service.yaml:

```
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
```

The service is connected to the deployment through the selector app:nginx

Now we can apply the deployment and the service:

```
kubectl apply -f nginx-deployment.yaml
kubectl apply -f nginx-service.yaml
```

We can see that 2 pods are running with internal IP addresses:

```
kubectl get pods -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP          NODE             NOMINATED NODE   READINESS GATES
nginx-deployment-85d6c5d8d6-hvpjf   1/1     Running   0          44m   10.1.0.17   docker-desktop   <none>           <none>
nginx-deployment-85d6c5d8d6-nxfjt   1/1     Running   0          44m   10.1.0.16   docker-desktop   <none>           <none>
```

The new nginx service has been created also:

```
kubectl get service
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes      ClusterIP   10.96.0.1      <none>        443/TCP   6h5m
nginx-service   ClusterIP   10.99.21.187   <none>        80/TCP    36m          <--------- new service
```

We can describe it and see that it's listening on port 80 and it references the 2 nginx pods listening each on local port 8080:

```
kubectl describe service nginx-service
```

```
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
```