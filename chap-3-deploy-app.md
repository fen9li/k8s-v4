## Deploy app to k8s by using 'kubectl run' command

```
[fli@192-168-1-10 ~]$ kubectl run kubia --image=luksa/kubia --port=8080 
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/kubia created
[fli@192-168-1-10 ~]$ 

[fli@192-168-1-10 ~]$ kubectl get all
NAME                        READY   STATUS    RESTARTS   AGE
pod/kubia-fc8ffbf5c-bfd9p   1/1     Running   0          4m18s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   17h

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/kubia   1/1     1            1           4m18s

NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/kubia-fc8ffbf5c   1         1         1       4m18s
[fli@192-168-1-10 ~]$ 
[fli@192-168-1-10 ~]$ kubectl get deployment
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
kubia   1/1     1            1           5m8s
[fli@192-168-1-10 ~]$ kubectl delete deployment kubia
deployment.apps "kubia" deleted
[fli@192-168-1-10 ~]$ 
```

* WHAT HAPPENED BEHIND THE SCENES When the kubectl command was run
  1. It created a new Deployment object in the cluster by sending a REST HTTP request to the Kubernetes API server
  2. The Deployment created a ReplicationSet
  3. The ReplicationSet created a new pod
  4. The pod was scheduled to one of the worker nodes by the Scheduler (The term scheduling means assigning the pod to a worker node.)
  5. The Kubelet on that node saw that the pod was scheduled to it and instructed Docker to pull the specified image from the registry because the image wasn't available locally
  6. After downloading the image, Docker created and ran the container.

![first app on Kubernetes](kubernetes-in-action-images/First-app-on-Kubernetes-1.png)

## Expose kubia service 
```
[fli@192-168-1-10 ~]$ kubectl expose deployment kubia --type=NodePort --name kubia-http 
service/kubia-http exposed
[fli@192-168-1-10 ~]$ 

[fli@192-168-1-10 ~]$ kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP          18h
kubia-http   NodePort    10.109.69.123   <none>        8080:30629/TCP   2m44s
[fli@192-168-1-10 ~]$ 

[fli@192-168-1-10 ~]$ minikube ip
192.168.39.239
[fli@192-168-1-10 ~]$ 
[fli@192-168-1-10 ~]$ curl http://192.168.39.239:30629
You've hit kubia-fc8ffbf5c-8vbsc
[fli@192-168-1-10 ~]$ 

[fli@192-168-1-10 ~]$ kubectl delete svc kubia-http
service "kubia-http" deleted
[fli@192-168-1-10 ~]$ kubectl delete deployment kubia
deployment.apps "kubia" deleted
[fli@192-168-1-10 ~]$ 
```

## Get the deployment yaml output
```
[fli@192-168-1-10 ~]$ kubectl get deployment kubia -o yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2019-12-06T00:41:41Z"
  generation: 1
  labels:
    run: kubia
  name: kubia
  namespace: default
  resourceVersion: "20771"
  selfLink: /apis/apps/v1/namespaces/default/deployments/kubia
  uid: 3559daab-52b2-4f8a-a018-582c061e1c46
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      run: kubia
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: kubia
    spec:
      containers:
      - image: luksa/kubia
        imagePullPolicy: Always
        name: kubia
        ports:
        - containerPort: 8080
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 1
  conditions:
  - lastTransitionTime: "2019-12-06T00:41:49Z"
    lastUpdateTime: "2019-12-06T00:41:49Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: "2019-12-06T00:41:41Z"
    lastUpdateTime: "2019-12-06T00:41:49Z"
    message: ReplicaSet "kubia-fc8ffbf5c" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  observedGeneration: 1
  readyReplicas: 1
  replicas: 1
  updatedReplicas: 1
[fli@192-168-1-10 ~]$ 
```

## Get pod log
```
[fli@192-168-1-10 kubernetes]$ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
kubia-fc8ffbf5c-rts95   1/1     Running   0          9m59s
[fli@192-168-1-10 kubernetes]$ kubectl logs kubia-fc8ffbf5c-rts95
Kubia server starting...
Received request from ::ffff:172.17.0.1
[fli@192-168-1-10 kubernetes]$ 

[fli@192-168-1-10 kubernetes]$ kubectl logs kubia-fc8ffbf5c-rts95 -c kubia
Kubia server starting...
Received request from ::ffff:172.17.0.1
[fli@192-168-1-10 kubernetes]$ 
```

## Get pod details
```
[fli@192-168-1-10 kubernetes]$ kubectl describe pod kubia-fc8ffbf5c-rts95
Name:         kubia-fc8ffbf5c-rts95
Namespace:    default
Priority:     0
Node:         minikube/192.168.39.239
Start Time:   Fri, 06 Dec 2019 16:16:47 +1100
Labels:       pod-template-hash=fc8ffbf5c
              run=kubia
Annotations:  <none>
Status:       Running
IP:           172.17.0.6
IPs:
  IP:           172.17.0.6
Controlled By:  ReplicaSet/kubia-fc8ffbf5c
Containers:
  kubia:
    Container ID:   docker://b07280f42a09b04fe72d57ce71bbbfe4c9a66a89b837c87af0a9dc624fb746cc
    Image:          luksa/kubia
    Image ID:       docker-pullable://luksa/kubia@sha256:3f28e304dc0f63dc30f273a4202096f0fa0d08510bd2ee7e1032ce600616de24
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Fri, 06 Dec 2019 16:16:54 +1100
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-8qbtx (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-8qbtx:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-8qbtx
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age        From               Message
  ----    ------     ----       ----               -------
  Normal  Scheduled  <unknown>  default-scheduler  Successfully assigned default/kubia-fc8ffbf5c-rts95 to minikube
  Normal  Pulling    12m        kubelet, minikube  Pulling image "luksa/kubia"
  Normal  Pulled     12m        kubelet, minikube  Successfully pulled image "luksa/kubia"
  Normal  Created    12m        kubelet, minikube  Created container kubia
  Normal  Started    12m        kubelet, minikube  Started container kubia
[fli@192-168-1-10 kubernetes]$ 
```

## Health Checks

### Liveness Probe & Readiness Probe

* Kubernetes can probe a container using one of the three mechanisms:
  1. An HTTP GET probe performs an HTTP GET request on the container's IP address, a port and path you specify. If the probe receives a response, and the response code doesn't represent an error (in other words, if the HTTP response code is 2xx or 3xx), the probe is considered successful. If the server returns an error response code or if it doesn't respond at all, the probe is considered a failure and the container will be restarted as a result.
  2. A TCP Socket probe tries to open a TCP connection to the specified port of the container. If the connection is established successfully, the probe is successful. Otherwise, the container is restarted.
  3. An Exec probe executes an arbitrary command inside the container and checks the command's exit status code. If the status code is 0, the probe is successful. All other codes are considered failures.

Liveness determines if an application is running properly. Containers that fail liveness checks are restarted. Readiness describes when a container is ready to serve user requests. Containers that fail readiness checks are removed from service load balancers.

```
[fli@192-168-1-10 kubernetes]$ vim k8s-resources/kubia-deployment.yaml 
[fli@192-168-1-10 kubernetes]$ cat k8s-resources/kubia-deployment.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: kubia
    environment: prod
    owner: lifcn
    project: test
    budget: ready
  name: kubia
  namespace: default
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  selector:
    matchLabels:
      run: kubia
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        run: kubia
        environment: prod
        owner: lifcn
        project: test
        budget: ready
    spec:
      containers:
      - image: luksa/kubia
        imagePullPolicy: Always
        name: kubia
        <b>livenessProbe:
          httpGet:
            path: /
            port: 8080
          initialDelaySeconds: 5
          timeoutSeconds: 1
          periodSeconds: 10
          failureThreshold: 3</b>
        ports:
        - containerPort: 8080
          protocol: TCP
      nodeSelector:
        disktype: ssd
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
[fli@192-168-1-10 kubernetes]$ 
[fli@192-168-1-10 kubernetes]$ !918
kubectl diff -f k8s-resources/
diff -u -N /tmp/LIVE-877622347/apps.v1.Deployment.default.kubia /tmp/MERGED-268493614/apps.v1.Deployment.default.kubia
--- /tmp/LIVE-877622347/apps.v1.Deployment.default.kubia	2019-12-07 17:02:04.918054292 +1100
+++ /tmp/MERGED-268493614/apps.v1.Deployment.default.kubia	2019-12-07 17:02:04.937054291 +1100
@@ -6,7 +6,7 @@
     kubectl.kubernetes.io/last-applied-configuration: |
       {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"budget":"ready","environment":"prod","owner":"lifcn","project":"test","run":"kubia"},"name":"kubia","namespace":"default"},"spec":{"progressDeadlineSeconds":600,"replicas":1,"selector":{"matchLabels":{"run":"kubia"}},"strategy":{"rollingUpdate":{"maxSurge":"25%","maxUnavailable":"25%"},"type":"RollingUpdate"},"template":{"metadata":{"labels":{"budget":"ready","environment":"prod","owner":"lifcn","project":"test","run":"kubia"}},"spec":{"containers":[{"image":"luksa/kubia","imagePullPolicy":"Always","name":"kubia","ports":[{"containerPort":8080,"protocol":"TCP"}]}],"dnsPolicy":"ClusterFirst","nodeSelector":{"disktype":"ssd"},"restartPolicy":"Always","terminationGracePeriodSeconds":30}}}}
   creationTimestamp: "2019-12-06T05:16:46Z"
-  generation: 5
+  generation: 6
   labels:
     budget: ready
     environment: prod
@@ -43,6 +43,16 @@
       containers:
       - image: luksa/kubia
         imagePullPolicy: Always
+        livenessProbe:
+          failureThreshold: 3
+          httpGet:
+            path: /
+            port: 8080
+            scheme: HTTP
+          initialDelaySeconds: 5
+          periodSeconds: 10
+          successThreshold: 1
+          timeoutSeconds: 1
         name: kubia
         ports:
         - containerPort: 8080
exit status 1
[fli@192-168-1-10 kubernetes]$ 
[fli@192-168-1-10 kubernetes]$ kubectl apply -f k8s-resources/
deployment.apps/kubia configured
service/kubia-http unchanged
[fli@192-168-1-10 kubernetes]$ 

[fli@192-168-1-10 kubernetes]$ kubectl get pods
NAME                     READY   STATUS        RESTARTS   AGE
kubia-594db96f96-6phvn   1/1     Running       0          23s
kubia-596779555c-5vllr   1/1     Terminating   2          18h
[fli@192-168-1-10 kubernetes]$ 
[fli@192-168-1-10 kubernetes]$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
kubia-594db96f96-6phvn   1/1     Running   0          50s
[fli@192-168-1-10 kubernetes]$ 

[fli@192-168-1-10 kubernetes]$ curl http://localhost:8001/api/v1/namespaces/default/pods/kubia-594db96f96-6phvn | yq r - status.containerStatuses
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  4152    0  4152    0     0   727k      0 --:--:-- --:--:-- --:--:--  810k
- name: kubia
  state:
    running:
      startedAt: "2019-12-07T06:01:30Z"
  lastState: {}
  ready: true
  restartCount: 0
  image: luksa/kubia:latest
  imageID: docker-pullable://luksa/kubia@sha256:3f28e304dc0f63dc30f273a4202096f0fa0d08510bd2ee7e1032ce600616de24
  containerID: docker://fa9c6a2051a090fc3ece620274566b45343357a704da2da4ee3bfefd057fb232
  started: true
[fli@192-168-1-10 kubernetes]$ 
[fli@192-168-1-10 kubernetes]$ curl http://localhost:8001/api/v1/namespaces/default/pods/kubia-594db96f96-6phvn | yq r - status.containerStatuses.0.ready
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  4152    0  4152    0     0   517k      0 --:--:-- --:--:-- --:--:--  579k
true
[fli@192-168-1-10 kubernetes]$ 
```

## Pods resource management
```
[fli@192-168-1-10 kubernetes]$ cat k8s-resources/kubia-deployment.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: kubia
    environment: prod
    owner: lifcn
    project: test
    budget: ready
  name: kubia
  namespace: default
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  selector:
    matchLabels:
      run: kubia
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        run: kubia
        environment: prod
        owner: lifcn
        project: test
        budget: ready
    spec:
      containers:
      - image: luksa/kubia
        imagePullPolicy: Always
        name: kubia
        resources:
          requests:
            cpu: "500m"
            memory: "128Mi"
          limits:
            cpu: "1000m"
            memory: "256Mi"
        livenessProbe:
          httpGet:
            path: /
            port: 8080
          initialDelaySeconds: 5
          timeoutSeconds: 1
          periodSeconds: 10
          failureThreshold: 3
        ports:
        - containerPort: 8080
          protocol: TCP
      nodeSelector:
        disktype: ssd
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
[fli@192-168-1-10 kubernetes]$ 

kubectl diff -f k8s-resources/
kubectl apply -f k8s-resources/

[fli@192-168-1-10 kubernetes]$ curl http://localhost:8001/api/v1/namespaces/default/pods/kubia-6c96674dc5-bxp4x | yq r - spec.containers.0.resources
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  4322    0  4322    0     0   975k      0 --:--:-- --:--:-- --:--:-- 1055k
limits:
  cpu: "1"
  memory: 256Mi
requests:
  cpu: 500m
  memory: 128Mi
[fli@192-168-1-10 kubernetes]$ 
```

## Pods volumes

* emptyDir: A simple empty directory used for storing transient data.
* hostPath: Used for mounting directories from the worker node's filesystem into the pod.
* gitRepo: A volume initialized by checking out the contents of a Git repository.
* nfs: An NFS share mounted into the pod.
* gcePersistentDisk (Google Compute Engine Persistent Disk), awsElastic-BlockStore (Amazon Web Services Elastic Block Store Volume), azureDisk (Microsoft Azure Disk Volume): Used for mounting cloud provider-specific storage.
* cinder,cephfs,iscsi,flocker,glusterfs,quobyte,rbd,flexVolume,vsphere-Volume,photonPersistentDisk,scaleIO: Used for mounting other types of network storage.
* configMap,secret,downwardAPI: Special types of volumes used to expose certain Kubernetes resources and cluster information to the pod.
* persistentVolumeClaim: A way to use a pre- or dynamically provisioned persistent storage.

### Two containers share one volume in a pod

* Create the pod description yaml file
```
[fli@192-168-1-10 kubernetes]$ cat k8s-resources/fortune-pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: fortune
spec:
  containers:
    - image: luksa/fortune
      name: html-generator
      volumeMounts:
      - name: html
        mountPath: /var/htdocs
    - image: nginx:alpine
      name: web-server
      volumeMounts:
      - name: html
        mountPath: /usr/share/nginx/html
        readOnly: true
      ports:
      - containerPort: 80
        protocol: TCP
  volumes:
  - name: html
    emptyDir: {}
[fli@192-168-1-10 kubernetes]$ 
```

* Create the pod
```
[fli@192-168-1-10 kubernetes]$ kubectl apply -f k8s-resources/fortune-pod.yaml 
pod/fortune created
[fli@192-168-1-10 kubernetes]$ 
[fli@192-168-1-10 ~]$ kubectl get all
NAME          READY   STATUS    RESTARTS   AGE
pod/fortune   2/2     Running   0          14m

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   2d2h
[fli@192-168-1-10 ~]$ 
```

* Check pod containers
```
[fli@192-168-1-10 ~]$ kubectl describe pod/fortune -n default
Name:         fortune
Namespace:    default
Priority:     0
Node:         minikube/192.168.39.239
Start Time:   Sat, 07 Dec 2019 19:21:10 +1100
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"fortune","namespace":"default"},"spec":{"containers":[{"image":"luksa...
Status:       Running
IP:           172.17.0.6
IPs:
  IP:  172.17.0.6
Containers:
  html-generator:
    Container ID:   docker://1048b7e09bba5b7c84fc9ed55290818f4303ea1016fbd9004ade4af71da290c5
    Image:          luksa/fortune
    Image ID:       docker-pullable://luksa/fortune@sha256:814dd72ecc4b431560ca6ede8649fc4862c401ba5b6d37520e6672e0657deea9
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sat, 07 Dec 2019 19:22:22 +1100
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/htdocs from html (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-8qbtx (ro)
  web-server:
    Container ID:   docker://67461230082fffdf0b027829def95d4c6f61eb8e8d27403a7119a781e636dcf4
    Image:          nginx:alpine
    Image ID:       docker-pullable://nginx@sha256:0e61b143db3110f3b8ae29a67f107d5536b71a7c1f10afb14d4228711fc65a13
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 07 Dec 2019 19:22:38 +1100
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /usr/share/nginx/html from html (ro)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-8qbtx (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  html:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  default-token-8qbtx:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-8qbtx
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age        From               Message
  ----    ------     ----       ----               -------
  Normal  Scheduled  <unknown>  default-scheduler  Successfully assigned default/fortune to minikube
  Normal  Pulling    7m28s      kubelet, minikube  Pulling image "luksa/fortune"
  Normal  Pulled     6m22s      kubelet, minikube  Successfully pulled image "luksa/fortune"
  Normal  Created    6m18s      kubelet, minikube  Created container html-generator
  Normal  Started    6m18s      kubelet, minikube  Started container html-generator
  Normal  Pulling    6m18s      kubelet, minikube  Pulling image "nginx:alpine"
  Normal  Pulled     6m4s       kubelet, minikube  Successfully pulled image "nginx:alpine"
  Normal  Created    6m3s       kubelet, minikube  Created container web-server
  Normal  Started    6m2s       kubelet, minikube  Started container web-server
[fli@192-168-1-10 ~]$ 

[fli@192-168-1-10 ~]$ kubectl exec -it pod/fortune -c html-generator -- bash
root@fortune:/# ls /var/htdocs
index.html
root@fortune:/# cat /var/htdocs/index.html 
Are you sure the back door is locked?
root@fortune:/# exit
exit
[fli@192-168-1-10 ~]$ 

[fli@192-168-1-10 ~]$ kubectl exec -it pod/fortune -c web-server -- sh
/ # 
/ # ls /usr/share/nginx/html/
index.html
/ # cat /usr/share/nginx/html/index.html 
Are you sure the back door is locked?
/ # exit
[fli@192-168-1-10 ~]$ 
```

* Forward pod/container port 80 to local port 8080
```
[fli@192-168-1-10 ~]$ kubectl port-forward fortune 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080
Handling connection for 8080
Handling connection for 8080
... ...
```

* Consume the service from local port 8080
```
[fli@192-168-1-10 kubernetes]$ curl http://localhost:8080
Q:	Why is it that the more accuracy you demand from an interpolation
	function, the more expensive it becomes to compute?
A:	That's the Law of Spline Demand.
[fli@192-168-1-10 kubernetes]$ 
```

### PersistentVolumes and PersistentVolumeClaims

PersistentVolumes are provisioned by cluster admins and consumed by pods through PersistentVolumeClaims

![PersistentVolumes-PersistentVolumeClaims-1](kubernetes-in-action-images/PersistentVolumes-PersistentVolumeClaims-1.png)

* The cluster administrator creates the PersistentVolume
  1. The cluster administrator sets up the underlying storage 
  2. The cluster administrator registers it in Kubernetes by creating a PersistentVolume resource through the Kubernetes API server
  3. The cluster administrator specifies its size and the access modes it supports when creating the PersistentVolume

* The cluster user creates the PersistentVolumeClaim
  1. The cluster user creates PersistentVolumeClaim manifest by specifying the minimum size and the access mode 
  2. The cluster user submits the PersistentVolumeClaim manifest to the Kubernetes API server
  3. The Kubernetes finds the appropriate PersistentVolume and binds the volume to the claim
  4. The PersistentVolumeClaim can be used as one of the volumes inside a pod now. Other users cannot use the same PersistentVolume until it has been released by deleting the bound PersistentVolumeClaim.

> NOTE: unlike pods and PersistentVolumeClaims, PersistentVolumes don't belong to any namespace. They're cluster-level resources like nodes.   

* access modes of PersistentVolume
  1. RWO: ReadWriteOnce - Only a single node can mount the volume for reading and writing.
  2. ROX: ReadOnlyMany - Multiple nodes can mount the volume for reading.
  3. RWX: ReadWriteMany - Multiple nodes can mount the volume for both reading and writing.

* An example pod using a PersistentVolumeClaim volume: mongodb-pod-pvc.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: mongodb
spec:
  containers:
  - image: mongo
    name: mongodb
    volumeMounts:
    - name: mongodb-data          # Referencing the PersistentVolumeClaim by name in the pod volume
      mountPath: /data/db
    ports:
    - containerPort: 27017
      protocol: TCP
  volumes:
  - name: mongodb-data            # Referencing the PersistentVolumeClaim by name in the pod volume
    persistentVolumeClaim:
      claimName: mongodb-pvc
```

### Dynamic provisioning of PersistentVolumes

1. The cluster administrator deploys a PersistentVolume provisioner (Kubernetes already includes provisioners for the most popular cloud providers) and define one or more StorageClass objects.
2. The users refer to the StorageClass in their PersistentVolumeClaims.
3. The provisioner will take that into account when provisioning the persistent storage.

* An example StorageClass definition: storageclass-fast-gcepd.yaml

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast                              # The custom storage class name
provisioner: kubernetes.io/gce-pd         # The volume plugin to use for provisioning the PersistentVolume
parameters:                               # The parameters passed to the provisioner
  type: pd-ssd
  zone: europe-west1-b
```

* An example PVC with dynamic provisioning: mongodb-pvc-dp.yaml

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc
spec:
  storageClassName: fast                  # This PVC requests the custom storage class by its name
  resources:
    requests:
      storage: 100Mi
  accessModes:
    - ReadWriteOnce
```

## ConfigMap object

* Example 1
```
[fli@192-168-1-10 kubernetes]$ kubectl create configmap fortune-config --from-literal=sleep-interval=25
configmap/fortune-config created
[fli@192-168-1-10 kubernetes]$ kubectl get configmap fortune-config -o json
{
    "apiVersion": "v1",
    "data": {
        "sleep-interval": "25"
    },
    "kind": "ConfigMap",
    "metadata": {
        "creationTimestamp": "2019-12-08T22:38:37Z",
        "name": "fortune-config",
        "namespace": "default",
        "resourceVersion": "113469",
        "selfLink": "/api/v1/namespaces/default/configmaps/fortune-config",
        "uid": "7efc8e65-8256-475f-9c7e-6c8d84914dc8"
    }
}
[fli@192-168-1-10 kubernetes]$ 
```

* Example 2    
`kubectl create configmap my-config --from-file=foo.json --from-file=bar=foobar.conf --from-file=config-opts/ --from-literal=some=thing`

![ConfigMap-1.png](kubernetes-in-action-images/ConfigMap-1.png)

* Example 3: Using ConfigMap's entries in a volume 

This pod definition includes a volume, which references your fortune-config ConfigMap. You mount the volume into the /etc/nginx/conf.d directory to make
Nginx use it.

```
apiVersion: v1
kind: Pod
metadata:
  name: fortune-configmap-volume
spec:
  containers:
  - image: nginx:alpine
    name: web-server
    volumeMounts:
    ...
    - name: config                      # volume name
      mountPath: /etc/nginx/conf.d      # You're mounting the configMap volume at this location.
      readOnly: true
    ...
  volumes:
  ...
  - name: config                        # volume name
    configMap:
      name: fortune-config              # The volume refers to your fortune-config ConfigMap.
  ...
```

## secrets object

* The k8s secrets mounted into every container you run 

```
[fli@192-168-1-10 kubernetes]$ kubectl describe secrets
Name:         default-token-8qbtx
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: default
              kubernetes.io/service-account.uid: 35ef4468-2576-4723-b313-54103c7d0dc2

Type:  kubernetes.io/service-account-token

Data
====
namespace:  7 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6InRmdFpDckZ4VUUweW9DLVFmbnItTDNjOERVRFYwbXdGSHBWQ0VGdllGUFUifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlZmF1bHQtdG9rZW4tOHFidHgiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjM1ZWY0NDY4LTI1NzYtNDcyMy1iMzEzLTU0MTAzYzdkMGRjMiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.hExFUPeBJFNIrsrFxQY7BmPMmR3tzo0_JndL6NEvczovraY9tIqnU1XvwsFMOVqS1_2uVcjPcsf2_Rqmc66KjvV9t9FjMZHFjOdVsupV0VjFGmRtFjcCP0hBJk5ZMrvbHWMmM7UliQIGg88FukzhhDsaRmunzbgCeZzhnyH-6vBlDQJDJwDLpz2GINm8B9Gqq1Kv8w-qz1qvJeCfeH_j6vwFLKxxVnAjkbBLxAsGhIi0OOprf570H9UDOnrFtzkPl58CF-qo4lUx-pqnLzsWEo_eM4xUpQ1TIRUPaCxiZ3_LJfxL9I23xobgPFd58aytT24cAz5FqgYlfv7g9EzZKg
ca.crt:     1066 bytes
[fli@192-168-1-10 kubernetes]$ 

[fli@192-168-1-10 kubernetes]$ kubectl get secrets -o json
{
    "apiVersion": "v1",
    "items": [
        {
            "apiVersion": "v1",
            "data": {
                "ca.crt": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQVRBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwdGFXNXAKYTNWaVpVTkJNQjRYRFRFNU1USXdOREEyTURNME0xb1hEVEk1TVRJd01qQTJNRE0wTTFvd0ZURVRNQkVHQTFVRQpBeE1LYldsdWFXdDFZbVZEUVRDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTlhQCmx1bVcrQUdFOEQvaUI5YndnTEtWOVBvb0FZQnZBV3FWNHFmaUVlWTRNYzFmYUs1SlZ0a2Q5TGVuZHpTd0JuS00KZ1pkRDB1NitlMnVVQzI1dG1abnp0dXAwR21Xc2hLOWJxMFBOYndXTXhnOVVFeUI1d055OXMxbkU1S1lUL3c0SQo3dUxTZVFESHZUMWlVQm9hYTN0ZXhEWWlpK04yRGEyK04xUTU1NThraDJ2b2dCb3IwcjdXTytiVDF5d2tkMEVmCjV0VTloTW1HZlBiclNJTjRmVGI3L05TOHdqT3A2SkRXR1IwU1RpdEVGV1RybHFneDM0M1VnZDVOYTUrTUtMbGIKeXRsQ2JnY0ViT0lEWVlsckxkMXJyQU9UNEgzVzJrb3dyTHBZQTdycnRqUEJxeGduREJmZ3ZxbEtOaDluQ0VFZworWi9yNERjVUFQTTFBVm93QjRFQ0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUIwR0ExVWRKUVFXCk1CUUdDQ3NHQVFVRkJ3TUNCZ2dyQmdFRkJRY0RBVEFQQmdOVkhSTUJBZjhFQlRBREFRSC9NQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFBa1IxL2l4dFVCS1BSYXFxYW1GczM5YzBKcXV6OENQYXNGTG16endGdlZCKzYySGR3TApVVU9DTC9yWENkMnZxcFBpV01LaWlmTldwVXBwaVlEc1d1VjBaZlQvOE1GdjVBZkNmeDZxT0FYZS9Yd3ZDckhVCnJzbk1weU1GMTVVOFQ2MTQyVE1GSkF1a1ZIeUs2Z1RJMVhMdk1zUFhMcFN3QVJGU21Pa0hMMEZTZWI1dXpxWkgKcW0vL1NxT2J1Rit2TzRLL0RCSk9DWEJhQ1IzLzByazBSb25jblJNbVlPdVRaUUliZnRHME1LNzIzSnh0VFU0awo5cnBLRHRreEZoaGUyeHdsSnQwWFpsbUVnQjdUNm5BdFZXaWV1elhML0hZb3pjNEtyV1dqZ1llVUZHTGV3QU02ClVBdXlDVmYzRmpyd1BBRGlzb20xTGw0dU1nT1Q4N21xT09xNAotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==",
                "namespace": "ZGVmYXVsdA==",
                "token": "ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNkluUm1kRnBEY2taNFZVVXdlVzlETFZGbWJuSXRURE5qT0VSVlJGWXdiWGRHU0hCV1EwVkdkbGxHVUZVaWZRLmV5SnBjM01pT2lKcmRXSmxjbTVsZEdWekwzTmxjblpwWTJWaFkyTnZkVzUwSWl3aWEzVmlaWEp1WlhSbGN5NXBieTl6WlhKMmFXTmxZV05qYjNWdWRDOXVZVzFsYzNCaFkyVWlPaUprWldaaGRXeDBJaXdpYTNWaVpYSnVaWFJsY3k1cGJ5OXpaWEoyYVdObFlXTmpiM1Z1ZEM5elpXTnlaWFF1Ym1GdFpTSTZJbVJsWm1GMWJIUXRkRzlyWlc0dE9IRmlkSGdpTENKcmRXSmxjbTVsZEdWekxtbHZMM05sY25acFkyVmhZMk52ZFc1MEwzTmxjblpwWTJVdFlXTmpiM1Z1ZEM1dVlXMWxJam9pWkdWbVlYVnNkQ0lzSW10MVltVnlibVYwWlhNdWFXOHZjMlZ5ZG1salpXRmpZMjkxYm5RdmMyVnlkbWxqWlMxaFkyTnZkVzUwTG5WcFpDSTZJak0xWldZME5EWTRMVEkxTnpZdE5EY3lNeTFpTXpFekxUVTBNVEF6WXpka01HUmpNaUlzSW5OMVlpSTZJbk41YzNSbGJUcHpaWEoyYVdObFlXTmpiM1Z1ZERwa1pXWmhkV3gwT21SbFptRjFiSFFpZlEuaEV4RlVQZUJKRk5JcnNyRnhRWTdCbVBNbVIzdHpvMF9KbmRMNk5FdmN6b3ZyYVk5dElxblUxWHZ3c0ZNT1ZxUzFfMnVWY2pQY3NmMl9ScW1jNjZLanZWOXQ5RmpNWkhGak9kVnN1cFYwVmpGR21SdEZqY0NQMGhCSms1Wk1ydmJIV01tTTdVbGlRSUdnODhGdWt6aGhEc2FSbXVuemJnQ2VaemhueUgtNnZCbERRSkRKd0RMcHoyR0lObThCOUdxcTFLdjh3LXF6MXF2SmVDZmVIX2o2dndGTEt4eFZuQWprYkJMeEFzR2hJaTBPT3ByZjU3MEg5VURPbnJGdHprUGw1OENGLXFvNGxVeC1wcW5MenNXRW9fZU00eFVwUTFUSVJVUGFDeGlaM19MSmZ4TDlJMjN4b2JnUEZkNThheXRUMjRjQXo1RnFnWWxmdjdnOUV6Wktn"
            },
            "kind": "Secret",
            "metadata": {
                "annotations": {
                    "kubernetes.io/service-account.name": "default",
                    "kubernetes.io/service-account.uid": "35ef4468-2576-4723-b313-54103c7d0dc2"
                },
                "creationTimestamp": "2019-12-05T06:11:46Z",
                "name": "default-token-8qbtx",
                "namespace": "default",
                "resourceVersion": "346",
                "selfLink": "/api/v1/namespaces/default/secrets/default-token-8qbtx",
                "uid": "2782a91f-654b-4a45-bd36-8835ce6771f2"
            },
            "type": "kubernetes.io/service-account-token"
        }
    ],
    "kind": "List",
    "metadata": {
        "resourceVersion": "",
        "selfLink": ""
    }
}
[fli@192-168-1-10 kubernetes]$ 

[fli@192-168-1-10 kubernetes]$ kubectl exec fortune -c web-server ls /var/run/secrets/kubernetes.io/serviceaccount
ca.crt
namespace
token
[fli@192-168-1-10 kubernetes]$ 

[fli@192-168-1-10 kubernetes]$ kubectl exec fortune -c web-server cat /var/run/secrets/kubernetes.io/serviceaccount/token
eyJhbGciOiJSUzI1NiIsImtpZCI6InRmdFpDckZ4VUUweW9DLVFmbnItTDNjOERVRFYwbXdGSHBWQ0VGdllGUFUifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlZmF1bHQtdG9rZW4tOHFidHgiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjM1ZWY0NDY4LTI1NzYtNDcyMy1iMzEzLTU0MTAzYzdkMGRjMiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.hExFUPeBJFNIrsrFxQY7BmPMmR3tzo0_JndL6NEvczovraY9tIqnU1XvwsFMOVqS1_2uVcjPcsf2_Rqmc66KjvV9t9FjMZHFjOdVsupV0VjFGmRtFjcCP0hBJk5ZMrvbHWMmM7UliQIGg88FukzhhDsaRmunzbgCeZzhnyH-6vBlDQJDJwDLpz2GINm8B9Gqq1Kv8w-qz1qvJeCfeH_j6vwFLKxxVnAjkbBLxAsGhIi0OOprf570H9UDOnrFtzkPl58CF-qo4lUx-pqnLzsWEo_eM4xUpQ1TIRUPaCxiZ3_LJfxL9I23xobgPFd58aytT24cAz5FqgYlfv7g9EzZKg[fli@192-168-1-10 kubernetes]$ 
```

* Creating a secret for authenticating with a docker registry

`kubectl create secret docker-registry mydockerhubsecret --docker-username=myusername --docker-password=mypassword --docker-email=my.email@provider.com`

Rather than create a generic secret, you're creating a docker-registry Secret called mydockerhubsecret. You're specifying your Docker Hub username, password, and email. If you inspect the contents of the newly created Secret with kubectl describe, you'll see that it includes a single entry called .dockercfg. This is equivalent to the .dockercfg file in your home directory, which is created by Docker when you run the docker login command.

* Using the docker - registry secret in a pod defination
To have Kubernetes use the Secret when pulling images from your private Docker Hub repository, all you need to do is specify the Secret's name in the pod spec ...

```
apiVersion: v1
kind: Pod
metadata:
  name: private-pod
spec:
  imagePullSecrets:
  - name: mydockerhubsecret
  containers:
  - image: username/private:tag
    name: main
```

## ConfigMaps VS Secrets

* `kubectl get secret fortune-https -o yaml` are shown as Base64-encoded strings; `kubectl get configmap fortune-config -o yaml` are shown in clear text.

* `stringData` FIELD for plaintext 

```
kind: Secret
apiVersion: v1
stringData:                                     # The stringData can be used for non-binary Secret data.
  foo: plain text                               # See, "plain text" is not Base64-encoded.
data:
  https.cert: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCekNDQ...
  https.key: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcE...
```

## Downward API/volume

The Downward API enables you to expose the pod's own metadata to the processes running inside that pod. Currently, it allows you to pass the following information to your containers:

* The pod's name
* The pod's IP address
* The namespace the pod belongs to
* The name of the node the pod is running on
* The name of the service account the pod is running under     # a service account is the account that the pod authenticates as when talking to the API server
* The CPU and memory requests for each container
* The CPU and memory limits for each container
* The pod's labels
* The pod's annotations

```
[fli@192-168-1-10 kubernetes]$ cat k8s-resources/downward-api-env.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: downward
spec:
  containers:
  - name: main
    image: busybox
    command: ["sleep", "9999999"]
    resources:
      requests:
        cpu: "500m"
        memory: "128Mi"
      limits:
        cpu: "1000m"
        memory: "256Mi"
    env:
    - name: POD_NAME
      valueFrom:            # Instead of specifying an absolute value, you're referencing the metadata.name field from the pod manifest.
        fieldRef:
          fieldPath: metadata.name
    - name: POD_NAMESPACE
      valueFrom:
        fieldRef:
          fieldPath: metadata.namespace
    - name: POD_IP
      valueFrom:
        fieldRef:
          fieldPath: status.podIP
    - name: NODE_NAME
      valueFrom:
        fieldRef:
          fieldPath: spec.nodeName
    - name: SERVICE_ACCOUNT
      valueFrom:            # A container’s CPU and memory requests and limits are referenced by using resourceFieldRef instead of fieldRef.
        fieldRef:
          fieldPath: spec.serviceAccountName
    - name: CONTAINER_CPU_REQUEST_MILLICORES
      valueFrom:
        resourceFieldRef:
          resource: requests.cpu
          divisor: 1m      # For resource fields, you define a divisor to get the value in the unit you need. 
    - name: CONTAINER_MEMORY_LIMIT_KIBIBYTES
      valueFrom:
        resourceFieldRef:
          resource: limits.memory
          divisor: 1Mi
[fli@192-168-1-10 kubernetes]$ 

[fli@192-168-1-10 kubernetes]$ kubectl apply -f k8s-resources/downward-api-env.yaml 
pod/downward created
[fli@192-168-1-10 kubernetes]$

[fli@192-168-1-10 kubernetes]$ kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
downward   1/1     Running   0          66s
fortune    2/2     Running   2          40h
[fli@192-168-1-10 kubernetes]$ 

[fli@192-168-1-10 kubernetes]$ kubectl exec downward env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=downward
CONTAINER_MEMORY_LIMIT_KIBIBYTES=256
POD_NAME=downward
POD_NAMESPACE=default
POD_IP=172.17.0.7
NODE_NAME=minikube
SERVICE_ACCOUNT=default
CONTAINER_CPU_REQUEST_MILLICORES=500
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
HOME=/root
[fli@192-168-1-10 kubernetes]$ 

[fli@192-168-1-10 kubernetes]$ kubectl exec -it downward -- sh
/ # printenv
POD_IP=172.17.0.7
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.96.0.1:443
HOSTNAME=downward
SHLVL=1
HOME=/root
NODE_NAME=minikube
CONTAINER_MEMORY_LIMIT_KIBIBYTES=256
TERM=xterm
POD_NAME=downward
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
SERVICE_ACCOUNT=default
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
KUBERNETES_PORT_443_TCP_PORT=443
CONTAINER_CPU_REQUEST_MILLICORES=500
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
POD_NAMESPACE=default
KUBERNETES_SERVICE_HOST=10.96.0.1
PWD=/
/ # exit
[fli@192-168-1-10 kubernetes]$ 

```

## Service Account
```
[fli@192-168-1-10 ~]$ kubectl get serviceaccount
NAME      SECRETS   AGE
default   1         3d19h
[fli@192-168-1-10 ~]$ 
[fli@192-168-1-10 ~]$ kubectl get serviceaccount default -o json
{
    "apiVersion": "v1",
    "kind": "ServiceAccount",
    "metadata": {
        "creationTimestamp": "2019-12-05T06:11:46Z",
        "name": "default",
        "namespace": "default",
        "resourceVersion": "350",
        "selfLink": "/api/v1/namespaces/default/serviceaccounts/default",
        "uid": "35ef4468-2576-4723-b313-54103c7d0dc2"
    },
    "secrets": [
        {
            "name": "default-token-8qbtx"
        }
    ]
}
[fli@192-168-1-10 ~]$ 

```

## RBAC

```
[fli@192-168-1-10 ~]$ kubectl get clusterrolebindings
NAME                                                   AGE
cluster-admin                                          3d19h
kubeadm:kubelet-bootstrap                              3d19h
kubeadm:node-autoapprove-bootstrap                     3d19h
kubeadm:node-autoapprove-certificate-rotation          3d19h
kubeadm:node-proxier                                   3d19h
kubernetes-dashboard                                   3d16h
minikube-rbac                                          3d19h
storage-provisioner                                    3d19h
system:basic-user                                      3d19h
system:controller:attachdetach-controller              3d19h
system:controller:certificate-controller               3d19h
system:controller:clusterrole-aggregation-controller   3d19h
system:controller:cronjob-controller                   3d19h
system:controller:daemon-set-controller                3d19h
system:controller:deployment-controller                3d19h
system:controller:disruption-controller                3d19h
system:controller:endpoint-controller                  3d19h
system:controller:expand-controller                    3d19h
system:controller:generic-garbage-collector            3d19h
system:controller:horizontal-pod-autoscaler            3d19h
system:controller:job-controller                       3d19h
system:controller:namespace-controller                 3d19h
system:controller:node-controller                      3d19h
system:controller:persistent-volume-binder             3d19h
system:controller:pod-garbage-collector                3d19h
system:controller:pv-protection-controller             3d19h
system:controller:pvc-protection-controller            3d19h
system:controller:replicaset-controller                3d19h
system:controller:replication-controller               3d19h
system:controller:resourcequota-controller             3d19h
system:controller:route-controller                     3d19h
system:controller:service-account-controller           3d19h
system:controller:service-controller                   3d19h
system:controller:statefulset-controller               3d19h
system:controller:ttl-controller                       3d19h
system:coredns                                         3d19h
system:discovery                                       3d19h
system:kube-controller-manager                         3d19h
system:kube-dns                                        3d19h
system:kube-scheduler                                  3d19h
system:node                                            3d19h
system:node-proxier                                    3d19h
system:public-info-viewer                              3d19h
system:volume-scheduler                                3d19h
[fli@192-168-1-10 ~]$ 

```