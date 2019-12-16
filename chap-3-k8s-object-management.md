## [Kubernetes Object Management](https://kubernetes.io/docs/concepts/overview/working-with-objects/object-management/)

Everything contained in Kubernetes is represented by a RESTful resource or Kubernetes object. Each Kubernetes object exists at a unique HTTP path, for example, `http://127.0.0.1:8001/api/v1/namespaces/default/pods/kubia-596779555c-5vllr` leads to the representation of a Pod in the default namespace named `kubia-596779555c-5vllr`.

```
[fli@192-168-1-10 kubernetes]$ curl http://127.0.0.1:8001/api/v1/namespaces/default/pods/kubia-596779555c-5vllr
{
  "kind": "Pod",
  "apiVersion": "v1",
  "metadata": {
    "name": "kubia-596779555c-5vllr",
    "generateName": "kubia-596779555c-",
    "namespace": "default",
    "selfLink": "/api/v1/namespaces/default/pods/kubia-596779555c-5vllr",
    "uid": "e2beab7b-eebd-462f-9538-610c514f8057",
    "resourceVersion": "62051",
    "creationTimestamp": "2019-12-06T11:01:50Z",
    "labels": {
      "budget": "ready",
      "environment": "prod",
      "owner": "lifcn",
      "pod-template-hash": "596779555c",
      "project": "test",
      "run": "kubia"
    },
    "ownerReferences": [
      {
        "apiVersion": "apps/v1",
        "kind": "ReplicaSet",
        "name": "kubia-596779555c",
        "uid": "836ba528-0fd6-4031-806b-120db919b648",
        "controller": true,
        "blockOwnerDeletion": true
      }
    ]
  },
  "spec": {
    "volumes": [
      {
        "name": "default-token-8qbtx",
        "secret": {
          "secretName": "default-token-8qbtx",
          "defaultMode": 420
        }
      }
    ],
    "containers": [
      {
        "name": "kubia",
        "image": "luksa/kubia",
        "ports": [
          {
            "containerPort": 8080,
            "protocol": "TCP"
          }
        ],
        "resources": {
          
        },
        "volumeMounts": [
          {
            "name": "default-token-8qbtx",
            "readOnly": true,
            "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount"
          }
        ],
        "terminationMessagePath": "/dev/termination-log",
        "terminationMessagePolicy": "File",
        "imagePullPolicy": "Always"
      }
    ],
    "restartPolicy": "Always",
    "terminationGracePeriodSeconds": 30,
    "dnsPolicy": "ClusterFirst",
    "nodeSelector": {
      "disktype": "ssd"
    },
    "serviceAccountName": "default",
    "serviceAccount": "default",
    "nodeName": "minikube",
    "securityContext": {
      
    },
    "schedulerName": "default-scheduler",
    "tolerations": [
      {
        "key": "node.kubernetes.io/not-ready",
        "operator": "Exists",
        "effect": "NoExecute",
        "tolerationSeconds": 300
      },
      {
        "key": "node.kubernetes.io/unreachable",
        "operator": "Exists",
        "effect": "NoExecute",
        "tolerationSeconds": 300
      }
    ],
    "priority": 0,
    "enableServiceLinks": true
  },
  "status": {
    "phase": "Running",
    "conditions": [
      {
        "type": "Initialized",
        "status": "True",
        "lastProbeTime": null,
        "lastTransitionTime": "2019-12-06T11:01:50Z"
      },
      {
        "type": "Ready",
        "status": "True",
        "lastProbeTime": null,
        "lastTransitionTime": "2019-12-06T21:46:10Z"
      },
      {
        "type": "ContainersReady",
        "status": "True",
        "lastProbeTime": null,
        "lastTransitionTime": "2019-12-06T21:46:10Z"
      },
      {
        "type": "PodScheduled",
        "status": "True",
        "lastProbeTime": null,
        "lastTransitionTime": "2019-12-06T11:01:50Z"
      }
    ],
    "hostIP": "192.168.39.239",
    "podIP": "172.17.0.6",
    "podIPs": [
      {
        "ip": "172.17.0.6"
      }
    ],
    "startTime": "2019-12-06T11:01:50Z",
    "containerStatuses": [
      {
        "name": "kubia",
        "state": {
          "running": {
            "startedAt": "2019-12-06T21:46:09Z"
          }
        },
        "lastState": {
          "terminated": {
            "exitCode": 255,
            "reason": "Error",
            "startedAt": "2019-12-06T11:01:56Z",
            "finishedAt": "2019-12-06T21:45:11Z",
            "containerID": "docker://7634077ab6f1bb03d46621a01271ef1084d82589c8cd07bb1fbb2df9cfd2f8d8"
          }
        },
        "ready": true,
        "restartCount": 1,
        "image": "luksa/kubia:latest",
        "imageID": "docker-pullable://luksa/kubia@sha256:3f28e304dc0f63dc30f273a4202096f0fa0d08510bd2ee7e1032ce600616de24",
        "containerID": "docker://a18036032eaa788a2b41cab5de9bb0aaf24a1201b0e19bd5f7bcac5bf6f3ef9f",
        "started": true
      }
    ],
    "qosClass": "BestEffort"
  }
}[fli@192-168-1-10 kubernetes]$
```

## Management techniques

> Warning: A Kubernetes object should be managed using only one technique. Mixing and matching techniques for the same object results in undefined behavior.  

| Management technique | Operates on  | Recommended environment | Supported writers |
| -------------------- | ------------ | ----------------------- | ----------------- |
| Imperative commands  | Live objects |	Development projects    | 1+  |
| Imperative object configuration | Individual files | Production projects | 1 |
| Declarative object configuration | Directories of files | Production projects | 1+ |

### Imperative commands

* When using imperative commands, a user operates directly on live objects in a cluster. The user provides operations to the kubectl command as arguments or flags. This is the simplest way to get started or to run a one-off task in a cluster. Because this technique operates directly on live objects, it provides no history of previous configurations.

* Examples    
  - Run an instance of the nginx container by creating a Deployment object: `kubectl run nginx --image nginx`    
  - Do the same thing using a different syntax: `kubectl create deployment nginx --image nginx`    

### Imperative object configuration

* In imperative object configuration, the kubectl command specifies the operation (create, replace, etc.), optional flags and at least one file name. The file specified must contain a full definition of the object in YAML or JSON format.

* See the [API reference](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.16/) for more details on object definitions.

> Warning: The imperative replace command replaces the existing spec with the newly provided one, dropping all changes to the object missing from the configuration file. This approach should not be used with resource types whose specs are updated independently of the configuration file. Services of type LoadBalancer, for example, have their externalIPs field updated independently from the configuration by the cluster.    

* Examples
  - Create the objects defined in a configuration file: `kubectl create -f nginx.yaml`
  - Delete the objects defined in two configuration files: `kubectl delete -f nginx.yaml -f redis.yaml`
  - Update the objects defined in a configuration file by overwriting the live configuration: `kubectl replace -f nginx.yaml`

### Declarative object configuration

* When using declarative object configuration, a user operates on object configuration files stored locally, however the user does not define the operations to be taken on the files. Create, update, and delete operations are automatically detected per-object by kubectl. This enables working on directories, where different operations might be needed for different objects.   

> Note: Declarative object configuration retains changes made by other writers, even if the changes are not merged back to the object configuration file. This is possible by using the patch API operation to write only observed differences, instead of using the replace API operation to replace the entire object configuration.

* Examples
  - Process all object configuration files in the configs directory, and create or patch the live objects. You can first diff to see what changes are going to be made, and then apply:
   ```
   kubectl diff -f configs/
   kubectl apply -f configs/
   ```
  - Recursively process directories:
   ``` 
   kubectl diff -R -f configs/
   kubectl apply -R -f configs/
   ```

## Deploy by using deployment defination 

```
[fli@192-168-1-10 kubernetes]$ pwd
/home/fli/kubernetes
[fli@192-168-1-10 kubernetes]$ mkdir k8s-resources
[fli@192-168-1-10 kubernetes]$ cd k8s-resources/
[fli@192-168-1-10 k8s-resources]$ 
[fli@192-168-1-10 k8s-resources]$ vim kubernetes/kubia-deployment.yaml 
[fli@192-168-1-10 k8s-resources]$ cat kubernetes/kubia-deployment.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: kubia
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
    spec:
      containers:
      - image: luksa/kubia
        imagePullPolicy: Always
        name: kubia
        ports:
        - containerPort: 8080
          protocol: TCP
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
[fli@192-168-1-10 k8s-resources]$ cd .. 
[fli@192-168-1-10 kubernetes]$ ls k8s-resources/
kubia-deployment.yaml
[fli@192-168-1-10 kubernetes]$ 

[fli@192-168-1-10 kubernetes]$ kubectl diff -f k8s-resources/
diff -u -N /tmp/LIVE-827521949/apps.v1.Deployment.default.kubia /tmp/MERGED-988531800/apps.v1.Deployment.default.kubia
--- /tmp/LIVE-827521949/apps.v1.Deployment.default.kubia	2019-12-06 16:16:35.339888871 +1100
+++ /tmp/MERGED-988531800/apps.v1.Deployment.default.kubia	2019-12-06 16:16:35.358888871 +1100
@@ -0,0 +1,45 @@
+apiVersion: apps/v1
+kind: Deployment
+metadata:
+  creationTimestamp: "2019-12-06T05:15:29Z"
+  generation: 1
+  labels:
+    run: kubia
+  name: kubia
+  namespace: default
+  selfLink: /apis/apps/v1/namespaces/default/deployments/kubia
+  uid: 39ccdbf0-4b1f-4e41-8254-3b556ce0138b
+spec:
+  progressDeadlineSeconds: 600
+  replicas: 1
+  revisionHistoryLimit: 10
+  selector:
+    matchLabels:
+      run: kubia
+  strategy:
+    rollingUpdate:
+      maxSurge: 25%
+      maxUnavailable: 25%
+    type: RollingUpdate
+  template:
+    metadata:
+      creationTimestamp: null
+      labels:
+        run: kubia
+    spec:
+      containers:
+      - image: luksa/kubia
+        imagePullPolicy: Always
+        name: kubia
+        ports:
+        - containerPort: 8080
+          protocol: TCP
+        resources: {}
+        terminationMessagePath: /dev/termination-log
+        terminationMessagePolicy: File
+      dnsPolicy: ClusterFirst
+      restartPolicy: Always
+      schedulerName: default-scheduler
+      securityContext: {}
+      terminationGracePeriodSeconds: 30
+status: {}
exit status 1
[fli@192-168-1-10 kubernetes]$ 
 
[fli@192-168-1-10 kubernetes]$ kubectl apply -f k8s-resources/
deployment.apps/kubia created
[fli@192-168-1-10 kubernetes]$ 

[fli@192-168-1-10 kubernetes]$ kubectl diff -f k8s-resources/
[fli@192-168-1-10 kubernetes]$ 
[fli@192-168-1-10 kubernetes]$ vim k8s-resources/kubia-http-service.yaml 
[fli@192-168-1-10 kubernetes]$ cat k8s-resources/kubia-http-service.yaml 
apiVersion: v1
kind: Service
metadata:
  labels:
    run: kubia
  name: kubia-http
  namespace: default
spec:
  clusterIP: 10.104.242.214
  externalTrafficPolicy: Cluster
  ports:
  - nodePort: 30260
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    run: kubia
  sessionAffinity: None
  type: NodePort
[fli@192-168-1-10 kubernetes]$ 

[fli@192-168-1-10 kubernetes]$ kubectl diff -f k8s-resources/
diff -u -N /tmp/LIVE-871289689/v1.Service.default.kubia-http /tmp/MERGED-416179940/v1.Service.default.kubia-http
--- /tmp/LIVE-871289689/v1.Service.default.kubia-http	2019-12-06 16:24:14.461878836 +1100
+++ /tmp/MERGED-416179940/v1.Service.default.kubia-http	2019-12-06 16:24:14.467878836 +1100
@@ -0,0 +1,24 @@
+apiVersion: v1
+kind: Service
+metadata:
+  creationTimestamp: "2019-12-06T05:23:08Z"
+  labels:
+    run: kubia
+  name: kubia-http
+  namespace: default
+  selfLink: /api/v1/namespaces/default/services/kubia-http
+  uid: 21238af8-b46d-4f97-9247-26dfcd5567eb
+spec:
+  clusterIP: 10.104.242.214
+  externalTrafficPolicy: Cluster
+  ports:
+  - nodePort: 30260
+    port: 8080
+    protocol: TCP
+    targetPort: 8080
+  selector:
+    run: kubia
+  sessionAffinity: None
+  type: NodePort
+status:
+  loadBalancer: {}
exit status 1
[fli@192-168-1-10 kubernetes]$ 

[fli@192-168-1-10 kubernetes]$ kubectl apply -f k8s-resources/
deployment.apps/kubia unchanged
service/kubia-http created
[fli@192-168-1-10 kubernetes]$ 

[fli@192-168-1-10 kubernetes]$ minikube ip
192.168.39.239
[fli@192-168-1-10 kubernetes]$ 

[fli@192-168-1-10 kubernetes]$ kubectl get svc kubia-http
NAME         TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubia-http   NodePort   10.104.242.214   <none>        8080:30260/TCP   27s
[fli@192-168-1-10 kubernetes]$ curl 192.168.39.239:30260
You've hit kubia-fc8ffbf5c-rts95
[fli@192-168-1-10 kubernetes]$ 
 
[fli@192-168-1-10 kubernetes]$ kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
kubia-fc8ffbf5c-rr78x   1/1     Running   0          6m39s   172.17.0.6   minikube   <none>           <none>
[fli@192-168-1-10 kubernetes]$ 
```

## DaemonSet object
Pods created by a DaemonSet already have a target node specified and skip the Kubernetes Scheduler.

* An example YAML for a DaemonSet: ssd-monitor-daemonset.yaml
```
apiVersion: apps/v1beta2
kind: DaemonSet
metadata:
  name: ssd-monitor
spec:
  selector:
    matchLabels:
      app: ssd-monitor
  template:
    metadata:
      labels:
        app: ssd-monitor
    spec:
      nodeSelector:               # The pod template includes a node selector, which selects nodes with the disk=ssd label.              
        disk: ssd
      containers:
      - name: main
        image: luksa/ssd-monitor
```

## Job object

* If you need a Job to run more than once, you set completions to how many times you want the Job's pod to run.
* If you need a Job to run in parallel, you set the parallelism Job spec property.
* Limiting the time allowed for a Job pod to complete by setting the activeDeadlineSeconds property

* An example YAML definition of a Job: exporter.yaml
```
apiVersion: batch/v1
kind: Job
metadata:
  name: batch-job
spec:
  completions: 5
  parallelism: 2
  template:
    metadata:
      labels:
        app: batch-job
    spec:
      restartPolicy: OnFailure
      containers:
      - name: main
        image: luksa/batch-job
```

## CronJob object

* An example YAML for a CronJob resource: cronjob.yaml
```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: batch-job-every-fifteen-minutes
spec:
  schedule: "0,15,30,45 * * * *"
  jobTemplate:
    spec:
      template:
        metadata:
          labels:
            app: periodic-batch-job
        spec:
          restartPolicy: OnFailure
          containers:
          - name: main
            image: luksa/batch-job
```

## Watch how existing object was updated

* Add new lable to existing deployment
```
[fli@192-168-1-10 kubernetes]$ cat k8s-resources/kubia-deployment.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: kubia
    <b>environment: prod
    owner: lifcn</b>
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
        <b>environment: prod
        owner: lifcn</b>
    spec:
      containers:
      - image: luksa/kubia
        imagePullPolicy: Always
        name: kubia
        ports:
        - containerPort: 8080
          protocol: TCP
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
[fli@192-168-1-10 kubernetes]$ 
```

* Start k8s proxy to access API server and run curl command to watch events

```
[fli@192-168-1-10 kubernetes]$ kubectl proxy &
[2] 18247
[fli@192-168-1-10 kubernetes]$ Starting to serve on 127.0.0.1:8001

[fli@192-168-1-10 kubernetes]$ 
[fli@192-168-1-10 kubernetes]$ curl -s 127.0.0.1:8001/api/v1/watch/events 
```

* Apply changes to k8s objects now
```
[fli@192-168-1-10 kubernetes]$ kubectl diff -f k8s-resources/
diff -u -N /tmp/LIVE-781521625/apps.v1.Deployment.default.kubia /tmp/MERGED-976178788/apps.v1.Deployment.default.kubia
--- /tmp/LIVE-781521625/apps.v1.Deployment.default.kubia	2019-12-06 19:32:01.079632573 +1100
+++ /tmp/MERGED-976178788/apps.v1.Deployment.default.kubia	2019-12-06 19:32:01.085632573 +1100
@@ -6,8 +6,10 @@
     kubectl.kubernetes.io/last-applied-configuration: |
       {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"run":"kubia"},"name":"kubia","namespace":"default"},"spec":{"progressDeadlineSeconds":600,"replicas":1,"selector":{"matchLabels":{"run":"kubia"}},"strategy":{"rollingUpdate":{"maxSurge":"25%","maxUnavailable":"25%"},"type":"RollingUpdate"},"template":{"metadata":{"labels":{"run":"kubia"}},"spec":{"containers":[{"image":"luksa/kubia","imagePullPolicy":"Always","name":"kubia","ports":[{"containerPort":8080,"protocol":"TCP"}]}],"dnsPolicy":"ClusterFirst","restartPolicy":"Always","terminationGracePeriodSeconds":30}}}}
   creationTimestamp: "2019-12-06T05:16:46Z"
-  generation: 1
+  generation: 2
   labels:
+    environment: prod
+    owner: lifcn
     run: kubia
   name: kubia
   namespace: default
@@ -30,6 +32,8 @@
     metadata:
       creationTimestamp: null
       labels:
+        environment: prod
+        owner: lifcn
         run: kubia
     spec:
       containers:
exit status 1
[fli@192-168-1-10 kubernetes]$ 
[fli@192-168-1-10 kubernetes]$ kubectl apply -f k8s-resources/
deployment.apps/kubia configured
service/kubia-http unchanged
[fli@192-168-1-10 kubernetes]$ 
```

* Check the event watch

```
[fli@192-168-1-10 kubernetes]$ curl -s 127.0.0.1:8001/api/v1/watch/events 
{"type":"ADDED","object":{"kind":"Event","apiVersion":"v1","metadata":{"name":"kubia.15ddbb09f0114793","namespace":"default","selfLink":"/api/v1/namespaces/default/events/kubia.15ddbb09f0114793","uid":"9e06e8af-10c4-47ff-a92c-3bab519acbd7","resourceVersion":"42648","creationTimestamp":"2019-12-06T08:31:36Z"},"involvedObject":{"kind":"Deployment","namespace":"default","name":"kubia","uid":"50f887ad-f90f-4787-89dd-ffb03fb4ddb1","apiVersion":"apps/v1","resourceVersion":"42643"},"reason":"ScalingReplicaSet","message":"Scaled up replica set kubia-6cdd54d56b to 1","source":{"component":"deployment-controller"},"firstTimestamp":"2019-12-06T08:31:36Z","lastTimestamp":"2019-12-06T08:31:36Z","count":1,"type":"Normal","eventTime":null,"reportingComponent":"","reportingInstance":""}}
{"type":"ADDED","object":{"kind":"Event","apiVersion":"v1","metadata":{"name":"kubia-6cdd54d56b.15ddbb09f5e5aba3","namespace":"default","selfLink":"/api/v1/namespaces/default/events/kubia-6cdd54d56b.15ddbb09f5e5aba3","uid":"a9a0d68e-5d9b-470b-89f8-3893e8019e9e","resourceVersion":"42650","creationTimestamp":"2019-12-06T08:31:36Z"},"involvedObject":{"kind":"ReplicaSet","namespace":"default","name":"kubia-6cdd54d56b","uid":"aa9761bf-14cb-4c73-8d8b-a2b69e49698b","apiVersion":"apps/v1","resourceVersion":"42644"},"reason":"SuccessfulCreate","message":"Created pod: kubia-6cdd54d56b-tcmp4","source":{"component":"replicaset-controller"},"firstTimestamp":"2019-12-06T08:31:36Z","lastTimestamp":"2019-12-06T08:31:36Z","count":1,"type":"Normal","eventTime":null,"reportingComponent":"","reportingInstance":""}}
{"type":"ADDED","object":{"kind":"Event","apiVersion":"v1","metadata":{"name":"kubia-6cdd54d56b-tcmp4.15ddbb09f703b3ad","namespace":"default","selfLink":"/api/v1/namespaces/default/events/kubia-6cdd54d56b-tcmp4.15ddbb09f703b3ad","uid":"e1434d92-9bd1-43a1-93c9-9311cd5dce04","resourceVersion":"42654","creationTimestamp":"2019-12-06T08:31:36Z"},"involvedObject":{"kind":"Pod","namespace":"default","name":"kubia-6cdd54d56b-tcmp4","uid":"33bf9437-9ff9-4919-9287-403df18bab5d","apiVersion":"v1","resourceVersion":"42647"},"reason":"Scheduled","message":"Successfully assigned default/kubia-6cdd54d56b-tcmp4 to minikube","source":{"component":"default-scheduler"},"firstTimestamp":null,"lastTimestamp":null,"type":"Normal","eventTime":"2019-12-06T08:31:36.122825Z","action":"Binding","reportingComponent":"default-scheduler","reportingInstance":"default-scheduler-minikube"}}
{"type":"ADDED","object":{"kind":"Event","apiVersion":"v1","metadata":{"name":"kubia-6cdd54d56b-tcmp4.15ddbb0aaeaae5f3","namespace":"default","selfLink":"/api/v1/namespaces/default/events/kubia-6cdd54d56b-tcmp4.15ddbb0aaeaae5f3","uid":"49a4c71f-7806-4b41-8b84-5631d12963c5","resourceVersion":"42660","creationTimestamp":"2019-12-06T08:31:39Z"},"involvedObject":{"kind":"Pod","namespace":"default","name":"kubia-6cdd54d56b-tcmp4","uid":"33bf9437-9ff9-4919-9287-403df18bab5d","apiVersion":"v1","resourceVersion":"42649","fieldPath":"spec.containers{kubia}"},"reason":"Pulling","message":"Pulling image \"luksa/kubia\"","source":{"component":"kubelet","host":"minikube"},"firstTimestamp":"2019-12-06T08:31:39Z","lastTimestamp":"2019-12-06T08:31:39Z","count":1,"type":"Normal","eventTime":null,"reportingComponent":"","reportingInstance":""}}
{"type":"ADDED","object":{"kind":"Event","apiVersion":"v1","metadata":{"name":"kubia-6cdd54d56b-tcmp4.15ddbb0b5ed50f99","namespace":"default","selfLink":"/api/v1/namespaces/default/events/kubia-6cdd54d56b-tcmp4.15ddbb0b5ed50f99","uid":"4baa2d93-bc9a-4062-94c0-bb65d62d832e","resourceVersion":"42665","creationTimestamp":"2019-12-06T08:31:42Z"},"involvedObject":{"kind":"Pod","namespace":"default","name":"kubia-6cdd54d56b-tcmp4","uid":"33bf9437-9ff9-4919-9287-403df18bab5d","apiVersion":"v1","resourceVersion":"42649","fieldPath":"spec.containers{kubia}"},"reason":"Pulled","message":"Successfully pulled image \"luksa/kubia\"","source":{"component":"kubelet","host":"minikube"},"firstTimestamp":"2019-12-06T08:31:42Z","lastTimestamp":"2019-12-06T08:31:42Z","count":1,"type":"Normal","eventTime":null,"reportingComponent":"","reportingInstance":""}}
{"type":"ADDED","object":{"kind":"Event","apiVersion":"v1","metadata":{"name":"kubia-6cdd54d56b-tcmp4.15ddbb0b9b588fcd","namespace":"default","selfLink":"/api/v1/namespaces/default/events/kubia-6cdd54d56b-tcmp4.15ddbb0b9b588fcd","uid":"e394edd9-8aaf-4abd-9691-43e7435bbc86","resourceVersion":"42668","creationTimestamp":"2019-12-06T08:31:43Z"},"involvedObject":{"kind":"Pod","namespace":"default","name":"kubia-6cdd54d56b-tcmp4","uid":"33bf9437-9ff9-4919-9287-403df18bab5d","apiVersion":"v1","resourceVersion":"42649","fieldPath":"spec.containers{kubia}"},"reason":"Created","message":"Created container kubia","source":{"component":"kubelet","host":"minikube"},"firstTimestamp":"2019-12-06T08:31:43Z","lastTimestamp":"2019-12-06T08:31:43Z","count":1,"type":"Normal","eventTime":null,"reportingComponent":"","reportingInstance":""}}
{"type":"ADDED","object":{"kind":"Event","apiVersion":"v1","metadata":{"name":"kubia-6cdd54d56b-tcmp4.15ddbb0bbecfccb7","namespace":"default","selfLink":"/api/v1/namespaces/default/events/kubia-6cdd54d56b-tcmp4.15ddbb0bbecfccb7","uid":"619d7a69-7450-46ec-bcb3-e154a50b42a0","resourceVersion":"42669","creationTimestamp":"2019-12-06T08:31:43Z"},"involvedObject":{"kind":"Pod","namespace":"default","name":"kubia-6cdd54d56b-tcmp4","uid":"33bf9437-9ff9-4919-9287-403df18bab5d","apiVersion":"v1","resourceVersion":"42649","fieldPath":"spec.containers{kubia}"},"reason":"Started","message":"Started container kubia","source":{"component":"kubelet","host":"minikube"},"firstTimestamp":"2019-12-06T08:31:43Z","lastTimestamp":"2019-12-06T08:31:43Z","count":1,"type":"Normal","eventTime":null,"reportingComponent":"","reportingInstance":""}}
{"type":"ADDED","object":{"kind":"Event","apiVersion":"v1","metadata":{"name":"kubia.15ddbb0c08407ff4","namespace":"default","selfLink":"/api/v1/namespaces/default/events/kubia.15ddbb0c08407ff4","uid":"d778d2cd-c083-4daa-a95c-e0d5c6dbbc3a","resourceVersion":"42677","creationTimestamp":"2019-12-06T08:31:45Z"},"involvedObject":{"kind":"Deployment","namespace":"default","name":"kubia","uid":"50f887ad-f90f-4787-89dd-ffb03fb4ddb1","apiVersion":"apps/v1","resourceVersion":"42655"},"reason":"ScalingReplicaSet","message":"Scaled down replica set kubia-fc8ffbf5c to 0","source":{"component":"deployment-controller"},"firstTimestamp":"2019-12-06T08:31:45Z","lastTimestamp":"2019-12-06T08:31:45Z","count":1,"type":"Normal","eventTime":null,"reportingComponent":"","reportingInstance":""}}
{"type":"ADDED","object":{"kind":"Event","apiVersion":"v1","metadata":{"name":"kubia-fc8ffbf5c-rts95.15ddbb0c111a5ddc","namespace":"default","selfLink":"/api/v1/namespaces/default/events/kubia-fc8ffbf5c-rts95.15ddbb0c111a5ddc","uid":"887a1f89-e10c-4d36-ab34-147ce01b229d","resourceVersion":"42679","creationTimestamp":"2019-12-06T08:31:45Z"},"involvedObject":{"kind":"Pod","namespace":"default","name":"kubia-fc8ffbf5c-rts95","uid":"96e6cf00-af9f-497f-bc17-0996a51f5edf","apiVersion":"v1","resourceVersion":"28428","fieldPath":"spec.containers{kubia}"},"reason":"Killing","message":"Stopping container kubia","source":{"component":"kubelet","host":"minikube"},"firstTimestamp":"2019-12-06T08:31:45Z","lastTimestamp":"2019-12-06T08:31:45Z","count":1,"type":"Normal","eventTime":null,"reportingComponent":"","reportingInstance":""}}
{"type":"ADDED","object":{"kind":"Event","apiVersion":"v1","metadata":{"name":"kubia-fc8ffbf5c.15ddbb0c116d992d","namespace":"default","selfLink":"/api/v1/namespaces/default/events/kubia-fc8ffbf5c.15ddbb0c116d992d","uid":"f96f1094-845b-45c2-93ee-24fc4700c2f2","resourceVersion":"42681","creationTimestamp":"2019-12-06T08:31:45Z"},"involvedObject":{"kind":"ReplicaSet","namespace":"default","name":"kubia-fc8ffbf5c","uid":"72c8677d-0452-4be9-ac93-65461dd78b0e","apiVersion":"apps/v1","resourceVersion":"42675"},"reason":"SuccessfulDelete","message":"Deleted pod: kubia-fc8ffbf5c-rts95","source":{"component":"replicaset-controller"},"firstTimestamp":"2019-12-06T08:31:45Z","lastTimestamp":"2019-12-06T08:31:45Z","count":1,"type":"Normal","eventTime":null,"reportingComponent":"","reportingInstance":""}}
{"type":"ADDED","object":{"kind":"Event","apiVersion":"v1","metadata":{"name":"etcd-minikube.15ddbb2bf1088866","namespace":"kube-system","selfLink":"/api/v1/namespaces/kube-system/events/etcd-minikube.15ddbb2bf1088866","uid":"4607505f-8f90-43aa-abb1-99d012536844","resourceVersion":"42840","creationTimestamp":"2019-12-06T08:34:02Z"},"involvedObject":{"kind":"Pod","namespace":"kube-system","name":"etcd-minikube","uid":"beb830e8677c9bbb55199796721352cc","apiVersion":"v1","fieldPath":"spec.containers{etcd}"},"reason":"Unhealthy","message":"Liveness probe failed: HTTP probe failed with statuscode: 503","source":{"component":"kubelet","host":"minikube"},"firstTimestamp":"2019-12-06T08:34:02Z","lastTimestamp":"2019-12-06T08:34:02Z","count":1,"type":"Warning","eventTime":null,"reportingComponent":"","reportingInstance":""}}
{"type":"ADDED","object":{"kind":"Event","apiVersion":"v1","metadata":{"name":"kube-apiserver-minikube.15ddad6fff3cf529","namespace":"kube-system","selfLink":"/api/v1/namespaces/kube-system/events/kube-apiserver-minikube.15ddad6fff3cf529","uid":"4f515f9b-973d-4f31-9d90-9a3cf43ddfc2","resourceVersion":"42844","creationTimestamp":"2019-12-06T08:34:10Z"},"involvedObject":{"kind":"Pod","namespace":"kube-system","name":"kube-apiserver-minikube","uid":"f2dbdabacd63342ec4cd72aac5fa7374","apiVersion":"v1","fieldPath":"spec.containers{kube-apiserver}"},"reason":"Unhealthy","message":"Liveness probe failed: HTTP probe failed with statuscode: 500","source":{"component":"kubelet","host":"minikube"},"firstTimestamp":"2019-12-06T04:22:21Z","lastTimestamp":"2019-12-06T08:34:02Z","count":3,"type":"Warning","eventTime":null,"reportingComponent":"","reportingInstance":""}}
{"type":"MODIFIED","object":{"kind":"Event","apiVersion":"v1","metadata":{"name":"etcd-minikube.15ddbb2bf1088866","namespace":"kube-system","selfLink":"/api/v1/namespaces/kube-system/events/etcd-minikube.15ddbb2bf1088866","uid":"4607505f-8f90-43aa-abb1-99d012536844","resourceVersion":"42870","creationTimestamp":"2019-12-06T08:34:02Z"},"involvedObject":{"kind":"Pod","namespace":"kube-system","name":"etcd-minikube","uid":"beb830e8677c9bbb55199796721352cc","apiVersion":"v1","fieldPath":"spec.containers{etcd}"},"reason":"Unhealthy","message":"Liveness probe failed: HTTP probe failed with statuscode: 503","source":{"component":"kubelet","host":"minikube"},"firstTimestamp":"2019-12-06T08:34:02Z","lastTimestamp":"2019-12-06T08:34:32Z","count":2,"type":"Warning","eventTime":null,"reportingComponent":"","reportingInstance":""}}
{"type":"MODIFIED","object":{"kind":"Event","apiVersion":"v1","metadata":{"name":"kube-apiserver-minikube.15ddad6fff3cf529","namespace":"kube-system","selfLink":"/api/v1/namespaces/kube-system/events/kube-apiserver-minikube.15ddad6fff3cf529","uid":"4f515f9b-973d-4f31-9d90-9a3cf43ddfc2","resourceVersion":"42872","creationTimestamp":"2019-12-06T08:34:10Z"},"involvedObject":{"kind":"Pod","namespace":"kube-system","name":"kube-apiserver-minikube","uid":"f2dbdabacd63342ec4cd72aac5fa7374","apiVersion":"v1","fieldPath":"spec.containers{kube-apiserver}"},"reason":"Unhealthy","message":"Liveness probe failed: HTTP probe failed with statuscode: 500","source":{"component":"kubelet","host":"minikube"},"firstTimestamp":"2019-12-06T04:22:21Z","lastTimestamp":"2019-12-06T08:34:32Z","count":4,"type":"Warning","eventTime":null,"reportingComponent":"","reportingInstance":""}}
{"type":"MODIFIED","object":{"kind":"Event","apiVersion":"v1","metadata":{"name":"etcd-minikube.15ddbb2bf1088866","namespace":"kube-system","selfLink":"/api/v1/namespaces/kube-system/events/etcd-minikube.15ddbb2bf1088866","uid":"4607505f-8f90-43aa-abb1-99d012536844","resourceVersion":"42896","creationTimestamp":"2019-12-06T08:34:02Z"},"involvedObject":{"kind":"Pod","namespace":"kube-system","name":"etcd-minikube","uid":"beb830e8677c9bbb55199796721352cc","apiVersion":"v1","fieldPath":"spec.containers{etcd}"},"reason":"Unhealthy","message":"Liveness probe failed: HTTP probe failed with statuscode: 503","source":{"component":"kubelet","host":"minikube"},"firstTimestamp":"2019-12-06T08:34:02Z","lastTimestamp":"2019-12-06T08:35:02Z","count":3,"type":"Warning","eventTime":null,"reportingComponent":"","reportingInstance":""}}
{"type":"MODIFIED","object":{"kind":"Event","apiVersion":"v1","metadata":{"name":"kube-apiserver-minikube.15ddad6fff3cf529","namespace":"kube-system","selfLink":"/api/v1/namespaces/kube-system/events/kube-apiserver-minikube.15ddad6fff3cf529","uid":"4f515f9b-973d-4f31-9d90-9a3cf43ddfc2","resourceVersion":"42898","creationTimestamp":"2019-12-06T08:34:10Z"},"involvedObject":{"kind":"Pod","namespace":"kube-system","name":"kube-apiserver-minikube","uid":"f2dbdabacd63342ec4cd72aac5fa7374","apiVersion":"v1","fieldPath":"spec.containers{kube-apiserver}"},"reason":"Unhealthy","message":"Liveness probe failed: HTTP probe failed with statuscode: 500","source":{"component":"kubelet","host":"minikube"},"firstTimestamp":"2019-12-06T04:22:21Z","lastTimestamp":"2019-12-06T08:35:02Z","count":5,"type":"Warning","eventTime":null,"reportingComponent":"","reportingInstance":""}}
^C
[fli@192-168-1-10 kubernetes]$ 

```

## How k8s update pod label

1. A new pod with new label is created
2. New pod is ready and starts to serve requests
3. Old pod is terminiated

* Run below command to access k8s service 
```
[fli@192-168-1-10 kubernetes]$ while true; do curl 192.168.39.239:30260; done
You've hit kubia-7945f86f4c-7jjwd
You've hit kubia-7945f86f4c-7jjwd
...
```

* Apply label change to pod, and watching new pod (kubia-7945f86f4c-7jjwd) creating and old pod (kubia-6955698b69-zthl4) terminating
```
[fli@192-168-1-10 kubernetes]$ kubectl diff -f k8s-resources/
diff -u -N /tmp/LIVE-345479254/apps.v1.Deployment.default.kubia /tmp/MERGED-237440445/apps.v1.Deployment.default.kubia
--- /tmp/LIVE-345479254/apps.v1.Deployment.default.kubia	2019-12-06 21:29:22.012515120 +1100
+++ /tmp/MERGED-237440445/apps.v1.Deployment.default.kubia	2019-12-06 21:29:22.021515120 +1100
@@ -6,8 +6,9 @@
     kubectl.kubernetes.io/last-applied-configuration: |
       {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"environment":"prod","owner":"lifcn","project":"test","run":"kubia"},"name":"kubia","namespace":"default"},"spec":{"progressDeadlineSeconds":600,"replicas":1,"selector":{"matchLabels":{"run":"kubia"}},"strategy":{"rollingUpdate":{"maxSurge":"25%","maxUnavailable":"25%"},"type":"RollingUpdate"},"template":{"metadata":{"labels":{"environment":"prod","owner":"lifcn","project":"test","run":"kubia"}},"spec":{"containers":[{"image":"luksa/kubia","imagePullPolicy":"Always","name":"kubia","ports":[{"containerPort":8080,"protocol":"TCP"}]}],"dnsPolicy":"ClusterFirst","restartPolicy":"Always","terminationGracePeriodSeconds":30}}}}
   creationTimestamp: "2019-12-06T05:16:46Z"
-  generation: 3
+  generation: 4
   labels:
+    budget: ready
     environment: prod
     owner: lifcn
     project: test
@@ -33,6 +34,7 @@
     metadata:
       creationTimestamp: null
       labels:
+        budget: ready
         environment: prod
         owner: lifcn
         project: test
exit status 1
[fli@192-168-1-10 kubernetes]$ kubectl apply -f k8s-resources/
deployment.apps/kubia configured
service/kubia-http unchanged
[fli@192-168-1-10 kubernetes]$ kubectl get pods
NAME                     READY   STATUS              RESTARTS   AGE
kubia-6955698b69-zthl4   1/1     Running             0          2m16s
kubia-7945f86f4c-7jjwd   0/1     ContainerCreating   0          2s
[fli@192-168-1-10 kubernetes]$ 

[fli@192-168-1-10 kubernetes]$ kubectl get pods
NAME                     READY   STATUS        RESTARTS   AGE
kubia-6955698b69-zthl4   1/1     Terminating   0          2m23s
kubia-7945f86f4c-7jjwd   1/1     Running       0          9s
[fli@192-168-1-10 kubernetes]$ 

[fli@192-168-1-10 kubernetes]$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
kubia-7945f86f4c-7jjwd   1/1     Running   0          53s
[fli@192-168-1-10 kubernetes]$ 
```

> Note: during the change, command `while true; do curl 192.168.39.239:30260; done` is kept running in another terminal and never break.   

## The selflink in k8s object

* Get the selflink
```
[fli@192-168-1-10 kubernetes]$ kubectl get pods -o yaml | grep -i selflink
    selfLink: /api/v1/namespaces/default/pods/kubia-596779555c-5vllr
  selfLink: ""
[fli@192-168-1-10 kubernetes]$ 
```

* Start k8s api proxy
```
[fli@192-168-1-10 ~]$ kubectl proxy &
[1] 21034
[fli@192-168-1-10 ~]$ Starting to serve on 127.0.0.1:8001

[fli@192-168-1-10 ~]$ 
```

* Get the pod details from API server
```
[fli@192-168-1-10 kubernetes]$ curl http://127.0.0.1:8001/api/v1/namespaces/default/pods/kubia-596779555c-5vllr
{
  "kind": "Pod",
  "apiVersion": "v1",
  "metadata": {
    "name": "kubia-596779555c-5vllr",
    "generateName": "kubia-596779555c-",
    "namespace": "default",
    "selfLink": "/api/v1/namespaces/default/pods/kubia-596779555c-5vllr",
    "uid": "e2beab7b-eebd-462f-9538-610c514f8057",
    "resourceVersion": "62051",
    "creationTimestamp": "2019-12-06T11:01:50Z",
    "labels": {
      "budget": "ready",
      "environment": "prod",
      "owner": "lifcn",
      "pod-template-hash": "596779555c",
      "project": "test",
      "run": "kubia"
    },
    "ownerReferences": [
      {
        "apiVersion": "apps/v1",
        "kind": "ReplicaSet",
        "name": "kubia-596779555c",
        "uid": "836ba528-0fd6-4031-806b-120db919b648",
        "controller": true,
        "blockOwnerDeletion": true
      }
    ]
  },
  "spec": {
    "volumes": [
      {
        "name": "default-token-8qbtx",
        "secret": {
          "secretName": "default-token-8qbtx",
          "defaultMode": 420
        }
      }
    ],
    "containers": [
      {
        "name": "kubia",
        "image": "luksa/kubia",
        "ports": [
          {
            "containerPort": 8080,
            "protocol": "TCP"
          }
        ],
        "resources": {
          
        },
        "volumeMounts": [
          {
            "name": "default-token-8qbtx",
            "readOnly": true,
            "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount"
          }
        ],
        "terminationMessagePath": "/dev/termination-log",
        "terminationMessagePolicy": "File",
        "imagePullPolicy": "Always"
      }
    ],
    "restartPolicy": "Always",
    "terminationGracePeriodSeconds": 30,
    "dnsPolicy": "ClusterFirst",
    "nodeSelector": {
      "disktype": "ssd"
    },
    "serviceAccountName": "default",
    "serviceAccount": "default",
    "nodeName": "minikube",
    "securityContext": {
      
    },
    "schedulerName": "default-scheduler",
    "tolerations": [
      {
        "key": "node.kubernetes.io/not-ready",
        "operator": "Exists",
        "effect": "NoExecute",
        "tolerationSeconds": 300
      },
      {
        "key": "node.kubernetes.io/unreachable",
        "operator": "Exists",
        "effect": "NoExecute",
        "tolerationSeconds": 300
      }
    ],
    "priority": 0,
    "enableServiceLinks": true
  },
  "status": {
    "phase": "Running",
    "conditions": [
      {
        "type": "Initialized",
        "status": "True",
        "lastProbeTime": null,
        "lastTransitionTime": "2019-12-06T11:01:50Z"
      },
      {
        "type": "Ready",
        "status": "True",
        "lastProbeTime": null,
        "lastTransitionTime": "2019-12-06T21:46:10Z"
      },
      {
        "type": "ContainersReady",
        "status": "True",
        "lastProbeTime": null,
        "lastTransitionTime": "2019-12-06T21:46:10Z"
      },
      {
        "type": "PodScheduled",
        "status": "True",
        "lastProbeTime": null,
        "lastTransitionTime": "2019-12-06T11:01:50Z"
      }
    ],
    "hostIP": "192.168.39.239",
    "podIP": "172.17.0.6",
    "podIPs": [
      {
        "ip": "172.17.0.6"
      }
    ],
    "startTime": "2019-12-06T11:01:50Z",
    "containerStatuses": [
      {
        "name": "kubia",
        "state": {
          "running": {
            "startedAt": "2019-12-06T21:46:09Z"
          }
        },
        "lastState": {
          "terminated": {
            "exitCode": 255,
            "reason": "Error",
            "startedAt": "2019-12-06T11:01:56Z",
            "finishedAt": "2019-12-06T21:45:11Z",
            "containerID": "docker://7634077ab6f1bb03d46621a01271ef1084d82589c8cd07bb1fbb2df9cfd2f8d8"
          }
        },
        "ready": true,
        "restartCount": 1,
        "image": "luksa/kubia:latest",
        "imageID": "docker-pullable://luksa/kubia@sha256:3f28e304dc0f63dc30f273a4202096f0fa0d08510bd2ee7e1032ce600616de24",
        "containerID": "docker://a18036032eaa788a2b41cab5de9bb0aaf24a1201b0e19bd5f7bcac5bf6f3ef9f",
        "started": true
      }
    ],
    "qosClass": "BestEffort"
  }
}
[fli@192-168-1-10 kubernetes]$ 
```