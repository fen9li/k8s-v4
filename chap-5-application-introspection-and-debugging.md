## [Application Introspection and Debugging](https://kubernetes.io/docs/tasks/debug-application-cluster/)

## Using kubectl describe pod to fetch details about pods
```
[fli@192-168-1-10 kubernetes]$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
kubia-6cdd54d56b-tcmp4   1/1     Running   1          61m
[fli@192-168-1-10 kubernetes]$ kubectl describe pod kubia-6cdd54d56b-tcmp4
Name:         kubia-6cdd54d56b-tcmp4
Namespace:    default
Priority:     0
Node:         minikube/192.168.39.239
Start Time:   Fri, 06 Dec 2019 19:31:36 +1100
Labels:       environment=prod
              owner=lifcn
              pod-template-hash=6cdd54d56b
              run=kubia
Annotations:  <none>
Status:       Running
IP:           172.17.0.6
IPs:
  IP:           172.17.0.6
Controlled By:  ReplicaSet/kubia-6cdd54d56b
Containers:
  kubia:
    Container ID:   docker://5f4ba790a576a54b98f5473d593df99199fbab903aaf29ab04ab1dd270d90a65
    Image:          luksa/kubia
    Image ID:       docker-pullable://luksa/kubia@sha256:3f28e304dc0f63dc30f273a4202096f0fa0d08510bd2ee7e1032ce600616de24
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Fri, 06 Dec 2019 20:27:07 +1100
    Last State:     Terminated
      Reason:       Error
      Exit Code:    255
      Started:      Fri, 06 Dec 2019 19:31:43 +1100
      Finished:     Fri, 06 Dec 2019 20:25:05 +1100
    Ready:          True
    Restart Count:  1
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
  Type    Reason          Age        From               Message
  ----    ------          ----       ----               -------
  Normal  Scheduled       <unknown>  default-scheduler  Successfully assigned default/kubia-6cdd54d56b-tcmp4 to minikube
  Normal  Pulling         62m        kubelet, minikube  Pulling image "luksa/kubia"
  Normal  Pulled          62m        kubelet, minikube  Successfully pulled image "luksa/kubia"
  Normal  Created         62m        kubelet, minikube  Created container kubia
  Normal  Started         62m        kubelet, minikube  Started container kubia
  Normal  SandboxChanged  7m40s      kubelet, minikube  Pod sandbox changed, it will be killed and re-created.
  Normal  Pulling         7m27s      kubelet, minikube  Pulling image "luksa/kubia"
  Normal  Pulled          7m24s      kubelet, minikube  Successfully pulled image "luksa/kubia"
  Normal  Created         7m17s      kubelet, minikube  Created container kubia
  Normal  Started         7m10s      kubelet, minikube  Started container kubia
[fli@192-168-1-10 kubernetes]$ 
```

## Using kubectl get events to check what happened

```
[fli@192-168-1-10 kubernetes]$ kubectl get events
LAST SEEN   TYPE     REASON                    OBJECT                        MESSAGE
<unknown>   Normal   Scheduled                 pod/kubia-6cdd54d56b-tcmp4    Successfully assigned default/kubia-6cdd54d56b-tcmp4 to minikube
71m         Normal   Pulling                   pod/kubia-6cdd54d56b-tcmp4    Pulling image "luksa/kubia"
71m         Normal   Pulled                    pod/kubia-6cdd54d56b-tcmp4    Successfully pulled image "luksa/kubia"
71m         Normal   Created                   pod/kubia-6cdd54d56b-tcmp4    Created container kubia
71m         Normal   Started                   pod/kubia-6cdd54d56b-tcmp4    Started container kubia
16m         Normal   SandboxChanged            pod/kubia-6cdd54d56b-tcmp4    Pod sandbox changed, it will be killed and re-created.
16m         Normal   Pulling                   pod/kubia-6cdd54d56b-tcmp4    Pulling image "luksa/kubia"
16m         Normal   Pulled                    pod/kubia-6cdd54d56b-tcmp4    Successfully pulled image "luksa/kubia"
16m         Normal   Created                   pod/kubia-6cdd54d56b-tcmp4    Created container kubia
15m         Normal   Started                   pod/kubia-6cdd54d56b-tcmp4    Started container kubia
71m         Normal   SuccessfulCreate          replicaset/kubia-6cdd54d56b   Created pod: kubia-6cdd54d56b-tcmp4
71m         Normal   Killing                   pod/kubia-fc8ffbf5c-rts95     Stopping container kubia
71m         Normal   SuccessfulDelete          replicaset/kubia-fc8ffbf5c    Deleted pod: kubia-fc8ffbf5c-rts95
71m         Normal   ScalingReplicaSet         deployment/kubia              Scaled up replica set kubia-6cdd54d56b to 1
71m         Normal   ScalingReplicaSet         deployment/kubia              Scaled down replica set kubia-fc8ffbf5c to 0
54m         Normal   RegisteredNode            node/minikube                 Node minikube event: Registered Node minikube in Controller
41m         Normal   RegisteredNode            node/minikube                 Node minikube event: Registered Node minikube in Controller
17m         Normal   Starting                  node/minikube                 Starting kubelet.
16m         Normal   NodeHasSufficientMemory   node/minikube                 Node minikube status is now: NodeHasSufficientMemory
16m         Normal   NodeHasNoDiskPressure     node/minikube                 Node minikube status is now: NodeHasNoDiskPressure
16m         Normal   NodeHasSufficientPID      node/minikube                 Node minikube status is now: NodeHasSufficientPID
17m         Normal   NodeAllocatableEnforced   node/minikube                 Updated Node Allocatable limit across pods
15m         Normal   Starting                  node/minikube                 Starting kube-proxy.
15m         Normal   RegisteredNode            node/minikube                 Node minikube event: Registered Node minikube in Controller
[fli@192-168-1-10 kubernetes]$ 

```

## Check nodes
```
[fli@192-168-1-10 kubernetes]$ kubectl get nodes
NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    master   28h   v1.16.2
[fli@192-168-1-10 kubernetes]$ kubectl describe node minikube
Name:               minikube
Roles:              master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=minikube
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/master=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Thu, 05 Dec 2019 17:11:28 +1100
Taints:             <none>
Unschedulable:      false
Conditions:
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----             ------  -----------------                 ------------------                ------                       -------
  MemoryPressure   False   Fri, 06 Dec 2019 21:49:43 +1100   Thu, 05 Dec 2019 17:11:24 +1100   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Fri, 06 Dec 2019 21:49:43 +1100   Thu, 05 Dec 2019 17:11:24 +1100   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure      False   Fri, 06 Dec 2019 21:49:43 +1100   Thu, 05 Dec 2019 17:11:24 +1100   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready            True    Fri, 06 Dec 2019 21:49:43 +1100   Thu, 05 Dec 2019 17:11:24 +1100   KubeletReady                 kubelet is posting ready status
Addresses:
  InternalIP:  192.168.39.239
  Hostname:    minikube
Capacity:
 cpu:                2
 ephemeral-storage:  16954240Ki
 hugepages-2Mi:      0
 memory:             3843760Ki
 pods:               110
Allocatable:
 cpu:                2
 ephemeral-storage:  16954240Ki
 hugepages-2Mi:      0
 memory:             3843760Ki
 pods:               110
System Info:
 Machine ID:                 572d50239b7b4138aeba314d8bf348f3
 System UUID:                572d5023-9b7b-4138-aeba-314d8bf348f3
 Boot ID:                    ecd40454-8383-4971-a818-ab0db553c527
 Kernel Version:             4.19.76
 OS Image:                   Buildroot 2019.02.6
 Operating System:           linux
 Architecture:               amd64
 Container Runtime Version:  docker://18.9.9
 Kubelet Version:            v1.16.2
 Kube-Proxy Version:         v1.16.2
Non-terminated Pods:         (12 in total)
  Namespace                  Name                                          CPU Requests  CPU Limits  Memory Requests  Memory Limits  AGE
  ---------                  ----                                          ------------  ----------  ---------------  -------------  ---
  default                    kubia-7945f86f4c-7jjwd                        0 (0%)        0 (0%)      0 (0%)           0 (0%)         22m
  kube-system                coredns-5644d7b6d9-c7qrx                      100m (5%)     0 (0%)      70Mi (1%)        170Mi (4%)     28h
  kube-system                coredns-5644d7b6d9-jjs4h                      100m (5%)     0 (0%)      70Mi (1%)        170Mi (4%)     28h
  kube-system                etcd-minikube                                 0 (0%)        0 (0%)      0 (0%)           0 (0%)         28h
  kube-system                kube-addon-manager-minikube                   5m (0%)       0 (0%)      50Mi (1%)        0 (0%)         28h
  kube-system                kube-apiserver-minikube                       250m (12%)    0 (0%)      0 (0%)           0 (0%)         28h
  kube-system                kube-controller-manager-minikube              200m (10%)    0 (0%)      0 (0%)           0 (0%)         28h
  kube-system                kube-proxy-tdbpb                              0 (0%)        0 (0%)      0 (0%)           0 (0%)         28h
  kube-system                kube-scheduler-minikube                       100m (5%)     0 (0%)      0 (0%)           0 (0%)         28h
  kube-system                storage-provisioner                           0 (0%)        0 (0%)      0 (0%)           0 (0%)         28h
  kubernetes-dashboard       dashboard-metrics-scraper-76585494d8-7qqnf    0 (0%)        0 (0%)      0 (0%)           0 (0%)         26h
  kubernetes-dashboard       kubernetes-dashboard-57f4cb4545-g9mtt         0 (0%)        0 (0%)      0 (0%)           0 (0%)         26h
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests    Limits
  --------           --------    ------
  cpu                755m (37%)  0 (0%)
  memory             190Mi (5%)  340Mi (9%)
  ephemeral-storage  0 (0%)      0 (0%)
Events:              <none>
[fli@192-168-1-10 kubernetes]$ 

```

## Node lebels and assign Pods to nodes

You can constrain a Pod to only be able to run on particular Node(s), or to prefer to run on particular nodes. There are several ways to do this, and the recommended approaches all use label selectors to make the selection. Generally such constraints are unnecessary, as the scheduler will automatically do a reasonable placement (e.g. spread your pods across nodes, not place the pod on a node with insufficient free resources, etc.) but there are some circumstances where you may want more control on a node where a pod lands, e.g. to ensure that a pod ends up on a machine with an SSD attached to it, or to co-locate pods from two different services that communicate a lot into the same availability zone.    

```
[fli@192-168-1-10 kubernetes]$ kubectl get nodes
NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    master   28h   v1.16.2
[fli@192-168-1-10 kubernetes]$ 
[fli@192-168-1-10 kubernetes]$ kubectl label nodes minikube disktype=ssd
node/minikube labeled
[fli@192-168-1-10 kubernetes]$ 
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
        ports:
        - containerPort: 8080
          protocol: TCP
      <b>nodeSelector:
        disktype: ssd</b>
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
[fli@192-168-1-10 kubernetes]$ 
[fli@192-168-1-10 kubernetes]$ kubectl apply -f k8s-resources/
deployment.apps/kubia configured
service/kubia-http unchanged
[fli@192-168-1-10 kubernetes]$ 

[fli@192-168-1-10 kubernetes]$ kubectl describe pod kubia-596779555c-5vllr | grep -i node-selector
Node-Selectors:  disktype=ssd
[fli@192-168-1-10 kubernetes]$ 

```

## Node affinity
* Node affinity is conceptually similar to nodeSelector – it allows you to constrain which nodes your pod is eligible to be scheduled on, based on labels on the node.

* There are currently two types of node affinity
  1. `requiredDuringSchedulingIgnoredDuringExecution` - must be met for a pod to be scheduled onto a node
  2. `preferredDuringSchedulingIgnoredDuringExecution` - the scheduler will try to enforce but will not guarantee
  
> The 'IgnoredDuringExecution' part of the names means that, similar to how nodeSelector works, if labels on a node change at runtime such that the affinity rules on a pod are no longer met, the pod will still continue to run on the node. In the future we plan to offer `requiredDuringSchedulingRequiredDuringExecution` which will be just like `requiredDuringSchedulingIgnoredDuringExecution` except that it will evict pods from nodes that cease to satisfy the pods' node affinity requirements.

## Taints and Tolerations
* Node affinity is a property of pods that attracts them to a set of nodes (either as a preference or a hard requirement). Taints are the opposite – they allow a node to repel a set of pods.

* Taints and tolerations work together to ensure that pods are not scheduled onto inappropriate nodes. One or more taints are applied to a node; this marks that the node should not accept any pods that do not tolerate the taints. Tolerations are applied to pods, and allow (but do not require) the pods to schedule onto nodes with matching taints.
