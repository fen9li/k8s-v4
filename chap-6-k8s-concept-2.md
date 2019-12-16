## Understanding Kubernetes Objects

### Object Spec and Status

Every Kubernetes object includes two nested object fields that govern the object's configuration: the object spec and the object status. The spec, which you must provide, describes your desired state for the object - the characteristics that you want the object to have. The status describes the actual state of the object, and is supplied and updated by the Kubernetes system. At any given time, the Kubernetes Control Plane actively manages an object's actual state to match the desired state you supplied.

### Describing a Kubernetes Object
When you create an object in Kubernetes, you must provide the object spec that describes its desired state, as well as some basic information about the object (such as a name). When you use the Kubernetes API to create the object (either directly or via kubectl), that API request must include that information as JSON in the request body. **Most often, you provide the information to** `kubectl` **in a .yaml file**. 

Here's an example .yaml file that shows the required fields and object spec for a Kubernetes Deployment:

```
# application/deployment.yaml 

apiVersion: apps/v1         # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2               # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

One way to create a Deployment using a .yaml file like the one above is to use the [kubectl apply](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply) command in the `kubectl` command-line interface, passing the `.yaml` file as an argument. Here's an example:

`kubectl apply -f https://k8s.io/examples/application/deployment.yaml --record`

The output is similar to this:

`deployment.apps/nginx-deployment created`

### Required Fields
In the `.yaml` file for the Kubernetes object you want to create, you'll need to set values for the following fields:

* apiVersion - Which version of the Kubernetes API you're using to create this object
* kind - What kind of object you want to create
* metadata - Data that helps uniquely identify the object, including a `name` string, `UID`, and optional `namespace`
* spec - What state you desire for the object

The precise format of the object `spec` is different for every Kubernetes object, and contains nested fields specific to that object. The [Kubernetes API Reference](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.16/) can help you find the spec format for all of the objects you can create using Kubernetes. For example, the `spec` format for a `Pod` can be found [here](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.16/#podspec-v1-core), and the `spec` format for a `Deployment` can be found [here](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.16/#deploymentspec-v1-apps).

## Kubernetes Object Management
The kubectl command-line tool supports several different ways to create and manage Kubernetes objects. Read the [Kubectl book](https://kubectl.docs.kubernetes.io/) for details of managing objects by Kubectl.

### Management techniques
> Warning: A Kubernetes object should be managed using only one technique. Mixing and matching techniques for the same object results in undefined behavior.    

| Management technique | Operates on | Recommended environment | Supported writers |
| ------------- | ------------------ | ------------------ | ------------------ |
| Imperative commands  | Live objects | Development projects | 1+ |
| Imperative object configuration |	Individual files | Production projects | 1 |
| Declarative object configuration | Directories of files |	Production projects	 | 1+ |

* Imperative commands Examples
Run an instance of the nginx container by creating a Deployment object:

`kubectl run nginx --image nginx`

Do the same thing using a different syntax:

`kubectl create deployment nginx --image nginx`

* Imperative object configuration Examples
Create the objects defined in a configuration file:

`kubectl create -f nginx.yaml`

Delete the objects defined in two configuration files:

`kubectl delete -f nginx.yaml -f redis.yaml`

Update the objects defined in a configuration file by overwriting the live configuration:

`kubectl replace -f nginx.yaml`

* Declarative object configuration Examples

Process all object configuration files in the configs directory, and create or patch the live objects. You can first diff to see what changes are going to be made, and then apply:

```
kubectl diff -f configs/
kubectl apply -f configs/
```

Recursively process directories:

```
kubectl diff -R -f configs/
kubectl apply -R -f configs/
```

## Names
Each object in your cluster has a [Name](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names) that is unique for that type of resource. Every Kubernetes object also has a [UID](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#uids) that is unique across your whole cluster.

For example, you can only have one Pod named `myapp-1234` within the same namespace, but you can have one Pod and one Deployment that are each named `myapp-1234`.

For non-unique user-provided attributes, Kubernetes provides [labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) and [annotations](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/).

* Names
A client-provided string that refers to an object in a resource URL, such as `/api/v1/pods/some-name`.

Here's an example manifest for a Pod named nginx-demo.

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-demo
spec:
  containers:
  - name: nginx
    image: nginx:1.7.9
    ports:
    - containerPort: 80
```

* UIDs
A Kubernetes systems-generated string to uniquely identify objects.

Every object created over the whole lifetime of a Kubernetes cluster has a distinct UID. It is intended to distinguish between historical occurrences of similar entities.

Kubernetes UIDs are universally unique identifiers (also known as UUIDs).

## Namespaces
Kubernetes supports multiple virtual clusters backed by the same physical cluster. These virtual clusters are called namespaces.

### When to Use Multiple Namespaces
Namespaces are intended for use in environments with many users spread across multiple teams, or projects. 

Namespaces are a way to divide cluster resources between multiple users (via [resource quota](https://kubernetes.io/docs/concepts/policy/resource-quotas/)).

It is not necessary to use multiple namespaces just to separate slightly different resources, such as different versions of the same software: use [labels](https://kubernetes.io/docs/user-guide/labels) to distinguish resources within the same namespace.

### Working with Namespaces
Creation and deletion of namespaces are described in the [Admin Guide documentation for namespaces](https://kubernetes.io/docs/admin/namespaces).

```
[fli@192-168-1-10 ~]$ kubectl get namespace
NAME                   STATUS   AGE
default                Active   30h
kube-node-lease        Active   30h
kube-public            Active   30h
kube-system            Active   30h
kubernetes-dashboard   Active   27h
[fli@192-168-1-10 ~]$ 
```

Kubernetes starts with three initial namespaces:

* `default` The default namespace for objects with no other namespace
* `kube-system` The namespace for objects created by the Kubernetes system
* `kube-public` This namespace is created automatically and is readable by all users (including those not authenticated). This namespace is mostly reserved for cluster usage, in case that some resources should be visible and readable publicly throughout the whole cluster. The public aspect of this namespace is only a convention, not a requirement.

To set the namespace for a current request, use the --namespace flag.

```
kubectl run nginx --image=nginx --namespace=<insert-namespace-name-here>
kubectl get pods --namespace=<insert-namespace-name-here>
```

You can permanently save the namespace for all subsequent kubectl commands in that context. `kubectl config set-context --current --namespace=<insert-namespace-name-here>`

Validate it. `kubectl config view --minify | grep namespace:`

### Namespaces and DNS

When you create a [Service](https://kubernetes.io/docs/user-guide/services), it creates a corresponding [DNS entry](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/). This entry is of the form `<service-name>.<namespace-name>.svc.cluster.local`, which means that if a container just uses `<service-name>`, it will resolve to the service which is local to a namespace. This is useful for using the same configuration across multiple namespaces such as Development, Staging and Production. If you want to reach across namespaces, you need to use the fully qualified domain name (FQDN).

### Not All Objects are in a Namespace
Most Kubernetes resources (e.g. pods, services, replication controllers, and others) are in some namespaces. However namespace resources are not themselves in a namespace. And low-level resources, such as [nodes](https://kubernetes.io/docs/admin/node) and persistentVolumes, are not in any namespace.

To see which Kubernetes resources are and aren't in a namespace:

* In a namespace
```
[fli@192-168-1-10 ~]$ kubectl api-resources --namespaced=true
NAME                        SHORTNAMES   APIGROUP                    NAMESPACED   KIND
bindings                                                             true         Binding
configmaps                  cm                                       true         ConfigMap
endpoints                   ep                                       true         Endpoints
events                      ev                                       true         Event
limitranges                 limits                                   true         LimitRange
persistentvolumeclaims      pvc                                      true         PersistentVolumeClaim
pods                        po                                       true         Pod
podtemplates                                                         true         PodTemplate
replicationcontrollers      rc                                       true         ReplicationController
resourcequotas              quota                                    true         ResourceQuota
secrets                                                              true         Secret
serviceaccounts             sa                                       true         ServiceAccount
services                    svc                                      true         Service
controllerrevisions                      apps                        true         ControllerRevision
daemonsets                  ds           apps                        true         DaemonSet
deployments                 deploy       apps                        true         Deployment
replicasets                 rs           apps                        true         ReplicaSet
statefulsets                sts          apps                        true         StatefulSet
localsubjectaccessreviews                authorization.k8s.io        true         LocalSubjectAccessReview
horizontalpodautoscalers    hpa          autoscaling                 true         HorizontalPodAutoscaler
cronjobs                    cj           batch                       true         CronJob
jobs                                     batch                       true         Job
leases                                   coordination.k8s.io         true         Lease
events                      ev           events.k8s.io               true         Event
ingresses                   ing          extensions                  true         Ingress
ingresses                   ing          networking.k8s.io           true         Ingress
networkpolicies             netpol       networking.k8s.io           true         NetworkPolicy
poddisruptionbudgets        pdb          policy                      true         PodDisruptionBudget
rolebindings                             rbac.authorization.k8s.io   true         RoleBinding
roles                                    rbac.authorization.k8s.io   true         Role
[fli@192-168-1-10 ~]$ 
```

* Not in a namespace
```
[fli@192-168-1-10 ~]$ kubectl api-resources --namespaced=false
NAME                              SHORTNAMES   APIGROUP                       NAMESPACED   KIND
componentstatuses                 cs                                          false        ComponentStatus
namespaces                        ns                                          false        Namespace
nodes                             no                                          false        Node
persistentvolumes                 pv                                          false        PersistentVolume
mutatingwebhookconfigurations                  admissionregistration.k8s.io   false        MutatingWebhookConfiguration
validatingwebhookconfigurations                admissionregistration.k8s.io   false        ValidatingWebhookConfiguration
customresourcedefinitions         crd,crds     apiextensions.k8s.io           false        CustomResourceDefinition
apiservices                                    apiregistration.k8s.io         false        APIService
tokenreviews                                   authentication.k8s.io          false        TokenReview
selfsubjectaccessreviews                       authorization.k8s.io           false        SelfSubjectAccessReview
selfsubjectrulesreviews                        authorization.k8s.io           false        SelfSubjectRulesReview
subjectaccessreviews                           authorization.k8s.io           false        SubjectAccessReview
certificatesigningrequests        csr          certificates.k8s.io            false        CertificateSigningRequest
runtimeclasses                                 node.k8s.io                    false        RuntimeClass
podsecuritypolicies               psp          policy                         false        PodSecurityPolicy
clusterrolebindings                            rbac.authorization.k8s.io      false        ClusterRoleBinding
clusterroles                                   rbac.authorization.k8s.io      false        ClusterRole
priorityclasses                   pc           scheduling.k8s.io              false        PriorityClass
csidrivers                                     storage.k8s.io                 false        CSIDriver
csinodes                                       storage.k8s.io                 false        CSINode
storageclasses                    sc           storage.k8s.io                 false        StorageClass
volumeattachments                              storage.k8s.io                 false        VolumeAttachment
[fli@192-168-1-10 ~]$ 
```

## Labels and Selectors

Labels are key/value pairs that are attached to objects, such as pods. Labels are intended to be used to specify identifying attributes of objects that are meaningful and relevant to users, but do not directly imply semantics to the core system. Labels can be used to organize and to select subsets of objects. Labels can be attached to objects at creation time and subsequently added and modified at any time. Each object can have a set of key/value labels defined. Each Key must be unique for a given object.

```
"metadata": {
  "labels": {
    "key1" : "value1",
    "key2" : "value2"
  }
}
```

Labels allow for efficient queries and watches and are ideal for use in UIs and CLIs. Non-identifying information should be recorded using [annotations](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/).

Example labels:

* "release" : "stable", "release" : "canary"
* "environment" : "dev", "environment" : "qa", "environment" : "production"
* "tier" : "frontend", "tier" : "backend", "tier" : "cache"
* "partition" : "customerA", "partition" : "customerB"
* "track" : "daily", "track" : "weekly"

Via a label selector, the client/user can identify a set of objects. The API currently supports two types of selectors: equality-based and set-based. A label selector can be made of multiple requirements which are comma-separated. In the case of multiple requirements, all must be satisfied so the comma separator acts as a logical AND (&&) operator.

Equality- or inequality-based requirements allow filtering by label keys and values. Matching objects must satisfy all of the specified label constraints, though they may have additional labels as well. Three kinds of operators are admitted `=`,`==`,`!=`. The first two represent equality (and are simply synonyms), while the latter represents inequality. For example:

```
environment = production
tier != frontend
```

The former selects all resources with key equal to environment and value equal to production. The latter selects all resources with key equal to tier and value distinct from frontend, and all resources with no labels with the tier key. One could filter for resources in production excluding frontend using the comma operator: 

`environment=production,tier!=frontend`

One usage scenario for equality-based label requirement is for Pods to specify node selection criteria. For example, the sample Pod below selects nodes with the label "accelerator=nvidia-tesla-p100".

```
apiVersion: v1
kind: Pod
metadata:
  name: cuda-test
spec:
  containers:
    - name: cuda-test
      image: "k8s.gcr.io/cuda-vector-add:v0.1"
      resources:
        limits:
          nvidia.com/gpu: 1
  nodeSelector:
    accelerator: nvidia-tesla-p100
```

Set-based label requirements allow filtering keys according to a set of values. Three kinds of operators are supported: `in`,`notin` and `exists` (only the key identifier). 

For example:

* `environment in (production, qa)`       # selects all resources with key equal to `environment` and value equal to `production` or `qa`
* `tier notin (frontend, backend)`        # selects all resources with key equal to `tier` and values other than `frontend` and `backend`, and all resources with no labels with the `tier` key
* `partition`                             # selects all resources including a label with key `partition`; no values are checked
* `!partition`                            # selects all resources without a label with key `partition`; no values are checked.

*Set-based* requirements can be mixed with *equality-based* requirements. For example: `partition in (customerA, customerB),environment!=qa`.

A real example:

```
[fli@192-168-1-10 kubernetes]$ kubectl get pods -l environment=lifcn,project=test
No resources found in default namespace.
[fli@192-168-1-10 kubernetes]$ kubectl get pods -l owner=lifcn,project=test
NAME                     READY   STATUS    RESTARTS   AGE
kubia-596779555c-5vllr   1/1     Running   1          11h
[fli@192-168-1-10 kubernetes]$ kubectl get pods -l 'owner in (lifcn,baba),project notin (qa,prod)'
NAME                     READY   STATUS    RESTARTS   AGE
kubia-596779555c-5vllr   1/1     Running   1          11h
[fli@192-168-1-10 kubernetes]$ 
```

## Annotations
You can use Kubernetes annotations to attach arbitrary non-identifying metadata to objects. Clients such as tools and libraries can retrieve this metadata.

### Attaching metadata to objects
You can use either labels or annotations to attach metadata to Kubernetes objects. Labels can be used to select objects and to find collections of objects that satisfy certain conditions. In contrast, annotations are not used to identify and select objects. The metadata in an annotation can be small or large, structured or unstructured, and can include characters not permitted by labels.

Annotations, like labels, are key/value maps:

```
"metadata": {
  "annotations": {
    "key1" : "value1",
    "key2" : "value2"
  }
}
```

### examples of annotations

* Fields managed by a declarative configuration layer. Attaching these fields as annotations distinguishes them from default values set by clients or servers, and from auto-generated fields and fields set by auto-sizing or auto-scaling systems.
* Build, release, or image information like timestamps, release IDs, git branch, PR numbers, image hashes, and registry address.
* Pointers to logging, monitoring, analytics, or audit repositories.
* Client library or tool information that can be used for debugging purposes: for example, name, version, and build information.
* User or tool/system provenance information, such as URLs of related objects from other ecosystem components.
* Lightweight rollout tool metadata: for example, config or checkpoints.
* Phone or pager numbers of persons responsible, or directory entries that specify where that information can be found, such as a team web site.
* Directives from the end-user to the implementations to modify behavior or engage non-standard features.

For example, here’s the configuration file for a Pod that has the annotation `imageregistry: https://hub.docker.com/` :

```
apiVersion: v1
kind: Pod
metadata:
  name: annotations-demo
  annotations:
    imageregistry: "https://hub.docker.com/"
    owner: "lif@yahoo.com"
spec:
  containers:
  - name: nginx
    image: nginx:1.7.9
    ports:
    - containerPort: 80
```

## Field Selectors

Field selectors let you select Kubernetes resources based on the value of one or more resource fields. Here are some example field selector queries:

* metadata.name=my-service
* metadata.namespace!=default
* status.phase=Pending

### Get the pod information from API server
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

### `field-selector` by using `status`
```
[fli@192-168-1-10 kubernetes]$ kubectl get pods --field-selector status.phase=Running
NAME                     READY   STATUS    RESTARTS   AGE
kubia-596779555c-5vllr   1/1     Running   1          12h
[fli@192-168-1-10 kubernetes]$ 
```

### `field-selector` by using `metadata`
```
[fli@192-168-1-10 kubernetes]$ kubectl get services  --all-namespaces --field-selector metadata.namespace!=default
NAMESPACE              NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
kube-system            kube-dns                    ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP,9153/TCP   41h
kubernetes-dashboard   dashboard-metrics-scraper   ClusterIP   10.109.255.7    <none>        8000/TCP                 38h
kubernetes-dashboard   kubernetes-dashboard        ClusterIP   10.96.205.122   <none>        80/TCP                   38h
[fli@192-168-1-10 kubernetes]$ 
```

### Chained selectors
Field selectors can be chained together as a comma-separated list. 

```
[fli@192-168-1-10 kubernetes]$ kubectl get pods --field-selector=status.phase=Running,spec.restartPolicy=Always
NAME                     READY   STATUS    RESTARTS   AGE
kubia-596779555c-5vllr   1/1     Running   1          12h
[fli@192-168-1-10 kubernetes]$ 
```

### Multiple resource types
Field selectors can across multiple resource types. 

```
[fli@192-168-1-10 kubernetes]$ kubectl get statefulsets,services --all-namespaces --field-selector metadata.namespace!=default
NAMESPACE              NAME                                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
kube-system            service/kube-dns                    ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP,9153/TCP   41h
kubernetes-dashboard   service/dashboard-metrics-scraper   ClusterIP   10.109.255.7    <none>        8000/TCP                 38h
kubernetes-dashboard   service/kubernetes-dashboard        ClusterIP   10.96.205.122   <none>        80/TCP                   38h
[fli@192-168-1-10 kubernetes]$ 
```

## Recommended Labels

In order to take full advantage of using these labels, they should be applied on every resource object.

| Key	| Description	| Example	| Type |
| --- | --- | --- | --- | --- |
| app.kubernetes.io/name       | The name of the application	                                                          | mysql | string |
| app.kubernetes.io/instance   | A unique name identifying the instance of an application	                              | wordpress-abcxzy | string |
| app.kubernetes.io/version	   | The current version of the application (e.g., a semantic version, revision hash, etc.)	| 5.7.21 | string |
| app.kubernetes.io/component  | The component within the architecture	                                                | database | string |
| app.kubernetes.io/part-of    | The name of a higher level application this one is part of	                            | wordpress | string |
| app.kubernetes.io/managed-by | The tool being used to manage the operation of an application	                        | helm | string |

To illustrate these labels in action, consider the following StatefulSet object:

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  labels:
    app.kubernetes.io/name: mysql
    app.kubernetes.io/instance: wordpress-abcxzy
    app.kubernetes.io/version: "5.7.21"
    app.kubernetes.io/component: database
    app.kubernetes.io/part-of: wordpress
    app.kubernetes.io/managed-by: helm
```

### Applications And Instances Of Applications

An application can be installed one or more times into a Kubernetes cluster and, in some cases, the same namespace. For example, wordpress can be installed more than once where different websites are different installations of wordpress.

The name of an application and the instance name are recorded separately. For example, WordPress has a `app.kubernetes.io/name` of `wordpress` while it has an instance name, represented as `app.kubernetes.io/instance` with a value of `wordpress-abcxzy`. This enables the application and instance of the application to be identifiable. Every instance of an application must have a unique name.

### Web Application With A Database Example
A slightly more complicated application: a web application (WordPress) using a database (MySQL), installed using Helm. The following snippets illustrate the start of objects used to deploy this application.

* The start to the following Deployment is used for WordPress:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/name: wordpress
    app.kubernetes.io/instance: wordpress-abcxzy
    app.kubernetes.io/version: "4.9.4"
    app.kubernetes.io/managed-by: helm
    app.kubernetes.io/component: server
    app.kubernetes.io/part-of: wordpress
...
```

* The Service is used to expose WordPress:
```
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/name: wordpress
    app.kubernetes.io/instance: wordpress-abcxzy
    app.kubernetes.io/version: "4.9.4"
    app.kubernetes.io/managed-by: helm
    app.kubernetes.io/component: server
    app.kubernetes.io/part-of: wordpress
...
```

* MySQL is exposed as a StatefulSet with metadata for both it and the larger application it belongs to:
```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  labels:
    app.kubernetes.io/name: mysql
    app.kubernetes.io/instance: mysql-abcxzy
    app.kubernetes.io/version: "5.7.21"
    app.kubernetes.io/managed-by: helm
    app.kubernetes.io/component: database
    app.kubernetes.io/part-of: wordpress
...
```

* The Service is used to expose MySQL as part of WordPress:
```
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/name: mysql
    app.kubernetes.io/instance: mysql-abcxzy
    app.kubernetes.io/version: "5.7.21"
    app.kubernetes.io/managed-by: helm
    app.kubernetes.io/component: database
    app.kubernetes.io/part-of: wordpress
...
```

With the MySQL StatefulSet and Service you’ll notice information about both MySQL and Wordpress, the broader application, are included.

## Cluster Architecture

### Nodes

`kubectl get node minikube -o yaml` 
`kubectl describe node minikube`
`kubectl cordon $NODENAME`          # To mark a node unschedulable

* Node Status
A node's status contains the following information:

  - Addresses : HostName, ExternalIP, InternalIP
  - Conditions : Ready, MemoryPressure,	PIDPressure, DiskPressure, NetworkUnavailable
  - Capacity and Allocatable: Describes the resources available on the node, CPU, memory and the maximum number of pods that can be scheduled onto the node. Normally, nodes register themselves and report their capacity when creating the node object.
  - Info : Describes general information about the node, such as kernel version, Kubernetes version (kubelet and kube-proxy version), Docker version (if used), and OS name. This information is gathered by Kubelet from the node.

* Management
* Node Controller

```
[fli@192-168-1-10 kubernetes]$ kubectl get node minikube -o yaml
apiVersion: v1
kind: Node
metadata:
  annotations:
    kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
    node.alpha.kubernetes.io/ttl: "0"
    volumes.kubernetes.io/controller-managed-attach-detach: "true"
  creationTimestamp: "2019-12-05T06:11:28Z"
  labels:
    beta.kubernetes.io/arch: amd64
    beta.kubernetes.io/os: linux
    disktype: ssd
    kubernetes.io/arch: amd64
    kubernetes.io/hostname: minikube
    kubernetes.io/os: linux
    node-role.kubernetes.io/master: ""
  name: minikube
  resourceVersion: "79508"
  selfLink: /api/v1/nodes/minikube
  uid: 29548c26-375e-4f8e-9d6f-46e8189e1e68
spec: {}
status:
  addresses:
  - address: 192.168.39.239
    type: InternalIP
  - address: minikube
    type: Hostname
  allocatable:
    cpu: "2"
    ephemeral-storage: 16954240Ki
    hugepages-2Mi: "0"
    memory: 3843760Ki
    pods: "110"
  capacity:
    cpu: "2"
    ephemeral-storage: 16954240Ki
    hugepages-2Mi: "0"
    memory: 3843760Ki
    pods: "110"
  conditions:
  - lastHeartbeatTime: "2019-12-07T01:45:56Z"
    lastTransitionTime: "2019-12-05T06:11:24Z"
    message: kubelet has sufficient memory available
    reason: KubeletHasSufficientMemory
    status: "False"
    type: MemoryPressure
  - lastHeartbeatTime: "2019-12-07T01:45:56Z"
    lastTransitionTime: "2019-12-05T06:11:24Z"
    message: kubelet has no disk pressure
    reason: KubeletHasNoDiskPressure
    status: "False"
    type: DiskPressure
  - lastHeartbeatTime: "2019-12-07T01:45:56Z"
    lastTransitionTime: "2019-12-05T06:11:24Z"
    message: kubelet has sufficient PID available
    reason: KubeletHasSufficientPID
    status: "False"
    type: PIDPressure
  - lastHeartbeatTime: "2019-12-07T01:45:56Z"
    lastTransitionTime: "2019-12-05T06:11:24Z"
    message: kubelet is posting ready status
    reason: KubeletReady
    status: "True"
    type: Ready
  daemonEndpoints:
    kubeletEndpoint:
      Port: 10250
  images:
  - names:
    - luksa/kubia@sha256:3f28e304dc0f63dc30f273a4202096f0fa0d08510bd2ee7e1032ce600616de24
    - luksa/kubia:latest
    sizeBytes: 665277552
  - names:
    - k8s.gcr.io/etcd@sha256:12c2c5e5731c3bcd56e6f1c05c0f9198b6f06793fa7fca2fb43aab9622dc4afa
    - k8s.gcr.io/etcd:3.3.15-0
    sizeBytes: 246640776
  - names:
    - k8s.gcr.io/kube-apiserver@sha256:8c31925d4d86a8a4b45edbebba17f0a9bc00acb8a80796085f18218feb9471cf
    - k8s.gcr.io/kube-apiserver:v1.16.2
    sizeBytes: 217083230
  - names:
    - k8s.gcr.io/kube-controller-manager@sha256:d24ea0e3d735fcd81801482e3ba14e60f2b5b71c639c60db303f7287fbfc5eec
    - k8s.gcr.io/kube-controller-manager:v1.16.2
    sizeBytes: 163318174
  - names:
    - k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.1
    sizeBytes: 121711221
  - names:
    - k8s.gcr.io/kube-scheduler@sha256:00a8ef7932173272fc6352c799f4e9c33240e4f6b92580d24989008faa31cb0d
    - k8s.gcr.io/kube-scheduler:v1.16.2
    sizeBytes: 87269918
  - names:
    - k8s.gcr.io/kube-proxy@sha256:dea8ba1d374a2e5bf0498d32a6fd32b6bf09d18f74db3758c641b54d7010a01f
    - k8s.gcr.io/kube-proxy:v1.16.2
    sizeBytes: 86065116
  - names:
    - kubernetesui/dashboard@sha256:a35498beec44376efcf8c4478eebceb57ec3ba39a6579222358a1ebe455ec49e
    - kubernetesui/dashboard:v2.0.0-beta4
    sizeBytes: 84034786
  - names:
    - k8s.gcr.io/kube-addon-manager:v9.0
    sizeBytes: 83077558
  - names:
    - k8s.gcr.io/kube-addon-manager@sha256:3e315022a842d782a28e729720f21091dde21f1efea28868d65ec595ad871616
    - k8s.gcr.io/kube-addon-manager:v9.0.2
    sizeBytes: 83076028
  - names:
    - gcr.io/k8s-minikube/storage-provisioner:v1.8.1
    sizeBytes: 80815640
  - names:
    - k8s.gcr.io/k8s-dns-kube-dns-amd64:1.14.13
    sizeBytes: 51157394
  - names:
    - k8s.gcr.io/coredns@sha256:12eb885b8685b1b13a04ecf5c23bc809c2e57917252fd7b0be9e9c00644e8ee5
    - k8s.gcr.io/coredns:1.6.2
    sizeBytes: 44100963
  - names:
    - k8s.gcr.io/k8s-dns-sidecar-amd64:1.14.13
    sizeBytes: 42852039
  - names:
    - k8s.gcr.io/k8s-dns-dnsmasq-nanny-amd64:1.14.13
    sizeBytes: 41372492
  - names:
    - kubernetesui/metrics-scraper@sha256:35fcae4fd9232a541a8cb08f2853117ba7231750b75c2cb3b6a58a2aaa57f878
    - kubernetesui/metrics-scraper:v1.0.1
    sizeBytes: 40101504
  - names:
    - k8s.gcr.io/pause@sha256:f78411e19d84a252e53bff71a4407a5686c46983a2c2eeed83929b888179acea
    - k8s.gcr.io/pause:3.1
    sizeBytes: 742472
  nodeInfo:
    architecture: amd64
    bootID: faa361c5-e31f-4af6-876b-a9e8fdbf587c
    containerRuntimeVersion: docker://18.9.9
    kernelVersion: 4.19.76
    kubeProxyVersion: v1.16.2
    kubeletVersion: v1.16.2
    machineID: 572d50239b7b4138aeba314d8bf348f3
    operatingSystem: linux
    osImage: Buildroot 2019.02.6
    systemUUID: 572d5023-9b7b-4138-aeba-314d8bf348f3
[fli@192-168-1-10 kubernetes]$ 
[fli@192-168-1-10 kubernetes]$ kubectl describe node minikube
Name:               minikube
Roles:              master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    disktype=ssd
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
  MemoryPressure   False   Sat, 07 Dec 2019 13:01:58 +1100   Thu, 05 Dec 2019 17:11:24 +1100   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Sat, 07 Dec 2019 13:01:58 +1100   Thu, 05 Dec 2019 17:11:24 +1100   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure      False   Sat, 07 Dec 2019 13:01:58 +1100   Thu, 05 Dec 2019 17:11:24 +1100   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready            True    Sat, 07 Dec 2019 13:01:58 +1100   Thu, 05 Dec 2019 17:11:24 +1100   KubeletReady                 kubelet is posting ready status
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
 Boot ID:                    faa361c5-e31f-4af6-876b-a9e8fdbf587c
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
  default                    kubia-596779555c-5vllr                        0 (0%)        0 (0%)      0 (0%)           0 (0%)         15h
  kube-system                coredns-5644d7b6d9-c7qrx                      100m (5%)     0 (0%)      70Mi (1%)        170Mi (4%)     43h
  kube-system                coredns-5644d7b6d9-jjs4h                      100m (5%)     0 (0%)      70Mi (1%)        170Mi (4%)     43h
  kube-system                etcd-minikube                                 0 (0%)        0 (0%)      0 (0%)           0 (0%)         43h
  kube-system                kube-addon-manager-minikube                   5m (0%)       0 (0%)      50Mi (1%)        0 (0%)         43h
  kube-system                kube-apiserver-minikube                       250m (12%)    0 (0%)      0 (0%)           0 (0%)         43h
  kube-system                kube-controller-manager-minikube              200m (10%)    0 (0%)      0 (0%)           0 (0%)         43h
  kube-system                kube-proxy-tdbpb                              0 (0%)        0 (0%)      0 (0%)           0 (0%)         43h
  kube-system                kube-scheduler-minikube                       100m (5%)     0 (0%)      0 (0%)           0 (0%)         43h
  kube-system                storage-provisioner                           0 (0%)        0 (0%)      0 (0%)           0 (0%)         43h
  kubernetes-dashboard       dashboard-metrics-scraper-76585494d8-7qqnf    0 (0%)        0 (0%)      0 (0%)           0 (0%)         41h
  kubernetes-dashboard       kubernetes-dashboard-57f4cb4545-g9mtt         0 (0%)        0 (0%)      0 (0%)           0 (0%)         41h
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

### Master-Node Communication

* Cluster to Master
All communication paths from the cluster to the master terminate at the apiserver. In a typical deployment, the apiserver is configured to listen for remote connections on a secure HTTPS port (443) with one or more forms of client [authentication](https://kubernetes.io/docs/reference/access-authn-authz/authentication/) enabled. One or more forms of [authorization](https://kubernetes.io/docs/reference/access-authn-authz/authorization/) should be enabled, especially if [anonymous requests](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#anonymous-requests) or [service account tokens](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#service-account-tokens) are allowed.

Nodes should be provisioned with the public root certificate for the cluster such that they can connect securely to the apiserver along with valid client credentials. See [kubelet TLS bootstrapping](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/) for automated provisioning of kubelet client certificates.

Pods that wish to connect to the apiserver can do so securely by leveraging a service account so that Kubernetes will automatically inject the public root certificate and a valid bearer token into the pod when it is instantiated. The kubernetes service (in all namespaces) is configured with a virtual IP address that is redirected (via kube-proxy) to the HTTPS endpoint on the apiserver.

* Master to Cluster
There are two primary communication paths from the master (apiserver) to the cluster. The first is from the apiserver to the kubelet process which runs on each node in the cluster. The second is from the apiserver to any node, pod, or service through the apiserver's proxy functionality.

## Scale 
`kubectl scale rs kubia --replicas=3`
`kubectl get rs kubia`

## Ingress

```
[fli@192-168-1-10 kubernetes]$ minikube addons list
- addon-manager: enabled
- dashboard: enabled
- default-storageclass: enabled
- efk: disabled
- freshpod: disabled
- gvisor: disabled
- heapster: disabled
- helm-tiller: disabled
- ingress: disabled
- ingress-dns: disabled
- logviewer: disabled
- metrics-server: disabled
- nvidia-driver-installer: disabled
- nvidia-gpu-device-plugin: disabled
- registry: disabled
- registry-creds: disabled
- storage-provisioner: enabled
- storage-provisioner-gluster: disabled
[fli@192-168-1-10 kubernetes]$ minikube addons enable ingress
✅  ingress was successfully enabled
[fli@192-168-1-10 kubernetes]$ kubectl get pods --all-namespaces
NAMESPACE              NAME                                         READY   STATUS    RESTARTS   AGE
default                kubia-6c96674dc5-hhcrn                       1/1     Running   0          14m
kube-system            coredns-5644d7b6d9-c7qrx                     1/1     Running   8          4d21h
kube-system            coredns-5644d7b6d9-jjs4h                     1/1     Running   8          4d21h
kube-system            etcd-minikube                                1/1     Running   8          4d21h
kube-system            kube-addon-manager-minikube                  1/1     Running   8          4d21h
kube-system            kube-apiserver-minikube                      1/1     Running   8          4d21h
kube-system            kube-controller-manager-minikube             1/1     Running   11         4d21h
kube-system            kube-proxy-tdbpb                             1/1     Running   8          4d21h
kube-system            kube-scheduler-minikube                      1/1     Running   11         4d21h
kube-system            nginx-ingress-controller-6fc5bcc8c9-gh5vw    1/1     Running   0          2m42s
kube-system            storage-provisioner                          1/1     Running   8          4d21h
kubernetes-dashboard   dashboard-metrics-scraper-76585494d8-7qqnf   1/1     Running   8          4d18h
kubernetes-dashboard   kubernetes-dashboard-57f4cb4545-g9mtt        1/1     Running   10         4d18h
[fli@192-168-1-10 kubernetes]$ 
```

### Create ingress on kubia-http service

* The 'kubia-deployment','kubia-http' service, 'kubia-ingress' templates
```
[fli@192-168-1-10 kubernetes]$ cat k8s-resources/kubia-ingress.yaml 
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: kubia
spec:
  rules:
  - host: kubia.example.com
    http:
      paths:
      - path: /
        backend:
          serviceName: kubia-http
          servicePort: 30260
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

```

* The resources

```
[fli@192-168-1-10 kubernetes]$ kubectl get all
NAME                         READY   STATUS    RESTARTS   AGE
pod/kubia-6c96674dc5-hhcrn   1/1     Running   0          31m

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP          4d21h

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/kubia   1/1     1            1           31m

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/kubia-6c96674dc5   1         1         1       31m
[fli@192-168-1-10 kubernetes]$ 
 
[fli@192-168-1-10 kubernetes]$ kubectl get ingress
NAME    HOSTS               ADDRESS          PORTS   AGE
kubia   kubia.example.com   192.168.39.239   80      11m
[fli@192-168-1-10 kubernetes]$ 

[fli@192-168-1-10 kubernetes]$ sudo vim /etc/hosts
[sudo] password for fli: 
[fli@192-168-1-10 kubernetes]$ sudo tail -n 1 /etc/hosts
192.168.39.239 kubia.example.com kubia
[fli@192-168-1-10 kubernetes]$ 

[fli@192-168-1-10 kubernetes]$ kubectl describe ingress kubia
Name:             kubia
Namespace:        default
Address:          192.168.39.239
Default backend:  default-http-backend:80 (<none>)
Rules:
  Host               Path  Backends
  ----               ----  --------
  kubia.example.com  
                     /   kubia-http:30260 (172.17.0.6:8080)
Annotations:
  kubectl.kubernetes.io/last-applied-configuration:  {"apiVersion":"extensions/v1beta1","kind":"Ingress","metadata":{"annotations":{},"name":"kubia","namespace":"default"},"spec":{"rules":[{"host":"kubia.example.com","http":{"paths":[{"backend":{"serviceName":"kubia-http","servicePort":30260},"path":"/"}]}}]}}

Events:
  Type    Reason  Age                From                      Message
  ----    ------  ----               ----                      -------
  Normal  CREATE  16m                nginx-ingress-controller  Ingress default/kubia
  Normal  UPDATE  10m (x3 over 15m)  nginx-ingress-controller  Ingress default/kubia
[fli@192-168-1-10 kubernetes]$ 

```