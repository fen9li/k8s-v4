# Concepts(https://kubernetes.io/docs/concepts/)

To work with Kubernetes, you use *Kubernetes API objects* to describe your cluster's desired state.
You set your desired state by creating objects using the Kubernetes API, typically via the command-line interface, `kubectl`. You can also use the Kubernetes API directly to interact with the cluster and set or modify your desired state.

Once you've set your desired state, the *Kubernetes Control Plane* makes the cluster's current state match the desired state via the Pod Lifecycle Event Generator (PLEG). The Kubernetes Control Plane consists of a collection of processes running on your cluster:

* The **Kubernetes Master** is a collection of three processes that run on a single node in your cluster, which is designated as the master node. Those processes are: 
  - [kube-apiserver](https://kubernetes.io/docs/admin/kube-apiserver/)
  - [kube-controller-manager](https://kubernetes.io/docs/admin/kube-controller-manager/) 
  - [kube-scheduler](https://kubernetes.io/docs/admin/kube-scheduler/).
* Each individual non-master node in your cluster runs two processes:
  - [kubelet](https://kubernetes.io/docs/admin/kubelet/), which communicates with the Kubernetes Master.
  - [kube-proxy](https://kubernetes.io/docs/admin/kube-proxy/), a network proxy which reflects Kubernetes networking services on each node.

## Kubernetes API Objects

[Understanding Kubernetes Objects](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/).

The basic Kubernetes objects include:

* [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/)
* [Service](https://kubernetes.io/docs/concepts/services-networking/service/)
* [Volume](https://kubernetes.io/docs/concepts/storage/volumes/)
* [Namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

Kubernetes also contains higher-level abstractions that rely on [Controllers](https://kubernetes.io/docs/concepts/architecture/controller/) to build upon the basic objects, and provide additional functionality and convenience features. These include:

* [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
* [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
* [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
* [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)
* [Job](https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/)

## Commands List

To list all supported API versions: `kubectl api-versions` 
To open connection to kube-apiserver: `kubectl proxy &`
To discover all API objects paths from kube-apiserver: `curl http://localhost:8001/`
To discover detail API object path `ingresses` from kube-apiserver: `curl http://localhost:8001/apis/extensions/v1beta1`
To discover detail API object path `cronjob` from kube-apiserver: `curl http://localhost:8001/apis/batch/v1beta1`

```
[fli@192-168-1-10 ~]$ kubectl api-versions
admissionregistration.k8s.io/v1
admissionregistration.k8s.io/v1beta1
apiextensions.k8s.io/v1
apiextensions.k8s.io/v1beta1
apiregistration.k8s.io/v1
apiregistration.k8s.io/v1beta1
apps/v1
authentication.k8s.io/v1
authentication.k8s.io/v1beta1
authorization.k8s.io/v1
authorization.k8s.io/v1beta1
autoscaling/v1
autoscaling/v2beta1
autoscaling/v2beta2
batch/v1
batch/v1beta1
certificates.k8s.io/v1beta1
coordination.k8s.io/v1
coordination.k8s.io/v1beta1
events.k8s.io/v1beta1
extensions/v1beta1
networking.k8s.io/v1
networking.k8s.io/v1beta1
node.k8s.io/v1beta1
policy/v1beta1
rbac.authorization.k8s.io/v1
rbac.authorization.k8s.io/v1beta1
scheduling.k8s.io/v1
scheduling.k8s.io/v1beta1
storage.k8s.io/v1
storage.k8s.io/v1beta1
v1
[fli@192-168-1-10 ~]$ 

[fli@192-168-1-10 ~]$ kubectl proxy &
[1] 24038
[fli@192-168-1-10 ~]$ Starting to serve on 127.0.0.1:8001
[fli@192-168-1-10 ~]$ 

[fli@192-168-1-10 ~]$ curl http://localhost:8001
{
  "paths": [
    "/api",
    "/api/v1",
    "/apis",
    "/apis/",
    "/apis/admissionregistration.k8s.io",
    "/apis/admissionregistration.k8s.io/v1",
    "/apis/admissionregistration.k8s.io/v1beta1",
    "/apis/apiextensions.k8s.io",
    "/apis/apiextensions.k8s.io/v1",
    "/apis/apiextensions.k8s.io/v1beta1",
    "/apis/apiregistration.k8s.io",
    "/apis/apiregistration.k8s.io/v1",
    "/apis/apiregistration.k8s.io/v1beta1",
    "/apis/apps",
    "/apis/apps/v1",
    "/apis/authentication.k8s.io",
    "/apis/authentication.k8s.io/v1",
    "/apis/authentication.k8s.io/v1beta1",
    "/apis/authorization.k8s.io",
    "/apis/authorization.k8s.io/v1",
    "/apis/authorization.k8s.io/v1beta1",
    "/apis/autoscaling",
    "/apis/autoscaling/v1",
    "/apis/autoscaling/v2beta1",
    "/apis/autoscaling/v2beta2",
    "/apis/batch",
    "/apis/batch/v1",
    "/apis/batch/v1beta1",
    "/apis/certificates.k8s.io",
    "/apis/certificates.k8s.io/v1beta1",
    "/apis/coordination.k8s.io",
    "/apis/coordination.k8s.io/v1",
    "/apis/coordination.k8s.io/v1beta1",
    "/apis/events.k8s.io",
    "/apis/events.k8s.io/v1beta1",
    "/apis/extensions",
    "/apis/extensions/v1beta1",
    "/apis/networking.k8s.io",
    "/apis/networking.k8s.io/v1",
    "/apis/networking.k8s.io/v1beta1",
    "/apis/node.k8s.io",
    "/apis/node.k8s.io/v1beta1",
    "/apis/policy",
    "/apis/policy/v1beta1",
    "/apis/rbac.authorization.k8s.io",
    "/apis/rbac.authorization.k8s.io/v1",
    "/apis/rbac.authorization.k8s.io/v1beta1",
    "/apis/scheduling.k8s.io",
    "/apis/scheduling.k8s.io/v1",
    "/apis/scheduling.k8s.io/v1beta1",
    "/apis/storage.k8s.io",
    "/apis/storage.k8s.io/v1",
    "/apis/storage.k8s.io/v1beta1",
    "/healthz",
    "/healthz/autoregister-completion",
    "/healthz/etcd",
    "/healthz/log",
    "/healthz/ping",
    "/healthz/poststarthook/apiservice-openapi-controller",
    "/healthz/poststarthook/apiservice-registration-controller",
    "/healthz/poststarthook/apiservice-status-available-controller",
    "/healthz/poststarthook/bootstrap-controller",
    "/healthz/poststarthook/ca-registration",
    "/healthz/poststarthook/crd-informer-synced",
    "/healthz/poststarthook/generic-apiserver-start-informers",
    "/healthz/poststarthook/kube-apiserver-autoregistration",
    "/healthz/poststarthook/rbac/bootstrap-roles",
    "/healthz/poststarthook/scheduling/bootstrap-system-priority-classes",
    "/healthz/poststarthook/start-apiextensions-controllers",
    "/healthz/poststarthook/start-apiextensions-informers",
    "/healthz/poststarthook/start-kube-aggregator-informers",
    "/healthz/poststarthook/start-kube-apiserver-admission-initializer",
    "/livez",
    "/livez/autoregister-completion",
    "/livez/etcd",
    "/livez/log",
    "/livez/ping",
    "/livez/poststarthook/apiservice-openapi-controller",
    "/livez/poststarthook/apiservice-registration-controller",
    "/livez/poststarthook/apiservice-status-available-controller",
    "/livez/poststarthook/bootstrap-controller",
    "/livez/poststarthook/ca-registration",
    "/livez/poststarthook/crd-informer-synced",
    "/livez/poststarthook/generic-apiserver-start-informers",
    "/livez/poststarthook/kube-apiserver-autoregistration",
    "/livez/poststarthook/rbac/bootstrap-roles",
    "/livez/poststarthook/scheduling/bootstrap-system-priority-classes",
    "/livez/poststarthook/start-apiextensions-controllers",
    "/livez/poststarthook/start-apiextensions-informers",
    "/livez/poststarthook/start-kube-aggregator-informers",
    "/livez/poststarthook/start-kube-apiserver-admission-initializer",
    "/logs",
    "/metrics",
    "/openapi/v2",
    "/readyz",
    "/readyz/autoregister-completion",
    "/readyz/etcd",
    "/readyz/log",
    "/readyz/ping",
    "/readyz/poststarthook/apiservice-openapi-controller",
    "/readyz/poststarthook/apiservice-registration-controller",
    "/readyz/poststarthook/apiservice-status-available-controller",
    "/readyz/poststarthook/bootstrap-controller",
    "/readyz/poststarthook/ca-registration",
    "/readyz/poststarthook/crd-informer-synced",
    "/readyz/poststarthook/generic-apiserver-start-informers",
    "/readyz/poststarthook/kube-apiserver-autoregistration",
    "/readyz/poststarthook/rbac/bootstrap-roles",
    "/readyz/poststarthook/scheduling/bootstrap-system-priority-classes",
    "/readyz/poststarthook/start-apiextensions-controllers",
    "/readyz/poststarthook/start-apiextensions-informers",
    "/readyz/poststarthook/start-kube-aggregator-informers",
    "/readyz/poststarthook/start-kube-apiserver-admission-initializer",
    "/readyz/shutdown",
    "/version"
  ]
}[fli@192-168-1-10 ~]$ 
[fli@192-168-1-10 ~]$ curl http://localhost:8001/apis/extensions/v1beta1
{
  "kind": "APIResourceList",
  "groupVersion": "extensions/v1beta1",
  "resources": [
    {
      "name": "ingresses",
      "singularName": "",
      "namespaced": true,
      "kind": "Ingress",
      "verbs": [
        "create",
        "delete",
        "deletecollection",
        "get",
        "list",
        "patch",
        "update",
        "watch"
      ],
      "shortNames": [
        "ing"
      ],
      "storageVersionHash": "ZOAfGflaKd0="
    },
    {
      "name": "ingresses/status",
      "singularName": "",
      "namespaced": true,
      "kind": "Ingress",
      "verbs": [
        "get",
        "patch",
        "update"
      ]
    }
  ]
}[fli@192-168-1-10 ~]$ 

[fli@192-168-1-10 ~]$ curl http://localhost:8001/apis/batch/v1beta1
{
  "kind": "APIResourceList",
  "apiVersion": "v1",
  "groupVersion": "batch/v1beta1",
  "resources": [
    {
      "name": "cronjobs",
      "singularName": "",
      "namespaced": true,
      "kind": "CronJob",
      "verbs": [
        "create",
        "delete",
        "deletecollection",
        "get",
        "list",
        "patch",
        "update",
        "watch"
      ],
      "shortNames": [
        "cj"
      ],
      "categories": [
        "all"
      ],
      "storageVersionHash": "h/JlFAZkyyY="
    },
    {
      "name": "cronjobs/status",
      "singularName": "",
      "namespaced": true,
      "kind": "CronJob",
      "verbs": [
        "get",
        "patch",
        "update"
      ]
    }
  ]
}[fli@192-168-1-10 ~]$ 
```

## Kubernetes Control Plane

![components-of-kubernetes.png](kubernetes-in-action-images/components-of-kubernetes.png)

### Master Components
Master components provide the cluster's control plane. Master components make global decisions about the cluster (for example, scheduling), and they detect and respond to cluster events (for example, starting up a new pod when a deployment's replicas field is unsatisfied).

Master components can be run on any machine in the cluster. However, for simplicity, set up scripts typically start all master components on the same machine, and do not run user containers on this machine. See [Building High-Availability Clusters](https://kubernetes.io/docs/admin/high-availability/) for an example multi-master-VM setup.

* kube-apiserver
The API server is a component of the Kubernetes control plane that exposes the Kubernetes API. The API server is the front end for the Kubernetes control plane.

The main implementation of a Kubernetes API server is [kube-apiserver](https://kubernetes.io/docs/reference/generated/kube-apiserver/). kube-apiserver is designed to scale horizontally - that is, it scales by deploying more instances. You can run several instances of kube-apiserver and balance traffic between those instances.

* etcd
Consistent and highly-available key value store used as Kubernetes' backing store for all cluster data.

If your Kubernetes cluster uses etcd as its backing store, make sure you have a [back up](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster) plan for those data.

You can find in-depth information about etcd in the official [documentation](https://etcd.io/docs/).

* kube-scheduler
Component on the master that watches newly created pods that have no node assigned, and selects a node for them to run on.

Factors taken into account for scheduling decisions include individual and collective resource requirements, hardware/software/policy constraints, affinity and anti-affinity specifications, data locality, inter-workload interference and deadlines.

* kube-controller-manager
Component on the master that runs controllers.

Logically, each controller is a separate process, but to reduce complexity, they are all compiled into a single binary and run in a single process.

These controllers include:

  - Node Controller: Responsible for noticing and responding when nodes go down.
  - Replication Controller: Responsible for maintaining the correct number of pods for every replication controller object in the system.
  - Endpoints Controller: Populates the Endpoints object (that is, joins Services & Pods).
  - Service Account & Token Controllers: Create default accounts and API access tokens for new namespaces.

* cloud-controller-manager
[cloud-controller-manager](https://kubernetes.io/docs/tasks/administer-cluster/running-cloud-controller/) runs controllers that interact with the underlying cloud providers. The cloud-controller-manager binary is an alpha feature introduced in Kubernetes release 1.6.

cloud-controller-manager runs cloud-provider-specific controller loops only. You must disable these controller loops in the kube-controller-manager. You can disable the controller loops by setting the `--cloud-provider` flag to `external` when starting the kube-controller-manager.

cloud-controller-manager allows the cloud vendor's code and the Kubernetes code to evolve independently of each other. In prior releases, the core Kubernetes code was dependent upon cloud-provider-specific code for functionality. In future releases, code specific to cloud vendors should be maintained by the cloud vendor themselves, and linked to cloud-controller-manager while running Kubernetes.

The following controllers have cloud provider dependencies:

  - Node Controller: For checking the cloud provider to determine if a node has been deleted in the cloud after it stops responding
  - Route Controller: For setting up routes in the underlying cloud infrastructure
  - Service Controller: For creating, updating and deleting cloud provider load balancers
  - Volume Controller: For creating, attaching, and mounting volumes, and interacting with the cloud provider to orchestrate volumes

### Node Components
Node components run on every node, maintaining running pods and providing the Kubernetes runtime environment.

* kubelet
An agent that runs on each node in the cluster. It makes sure that containers are running in a pod.

The kubelet takes a set of PodSpecs that are provided through various mechanisms and ensures that the containers described in those PodSpecs are running and healthy. The kubelet doesn't manage containers which were not created by Kubernetes.

* kube-proxy
[kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/) is a network proxy that runs on each node in your cluster, implementing part of the Kubernetes Service concept.

kube-proxy maintains network rules on nodes. These network rules allow network communication to your Pods from network sessions inside or outside of your cluster.

kube-proxy uses the operating system packet filtering layer if there is one and it's available. Otherwise, kube-proxy forwards the traffic itself.

* Container Runtime
The container runtime is the software that is responsible for running containers.

Kubernetes supports several container runtimes: [Docker](http://www.docker.com/), containerd, cri-o, rktlet and any implementation of the [Kubernetes CRI (Container Runtime Interface)](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md).

### Addons
Addons use Kubernetes resources (DaemonSet, Deployment, etc) to implement cluster features. Because these are providing cluster-level features, namespaced resources for addons belong within the kube-system namespace.

Selected addons are described below; for an extended list of available addons, please see [Addons](https://kubernetes.io/docs/concepts/cluster-administration/addons/).

* DNS
While the other addons are not strictly required, all Kubernetes clusters should have [cluster DNS](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/), as many examples rely on it.

Cluster DNS is a DNS server, in addition to the other DNS server(s) in your environment, which serves DNS records for Kubernetes services.

Containers started by Kubernetes automatically include this DNS server in their DNS searches.

* Web UI (Dashboard)
[Dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/) is a general purpose, web-based UI for Kubernetes clusters. It allows users to manage and troubleshoot applications running in the cluster, as well as the cluster itself.

* Container Resource Monitoring
[Container Resource Monitoring](https://kubernetes.io/docs/tasks/debug-application-cluster/resource-usage-monitoring/) records generic time-series metrics about containers in a central database, and provides a UI for browsing that data.

* Cluster-level Logging
A [cluster-level logging](https://kubernetes.io/docs/concepts/cluster-administration/logging/) mechanism is responsible for saving container logs to a central log store with search/browsing interface.

## The Kubernetes API

Overall API conventions are described in the [API conventions doc](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md).

API endpoints, resource types and samples are described in [API Reference](https://kubernetes.io/docs/reference).

Remote access to the API is discussed in the [Controlling API Access doc](https://kubernetes.io/docs/reference/access-authn-authz/controlling-access/).

The Kubernetes API also serves as the foundation for the declarative configuration schema for the system. The kubectl command-line tool can be used to create, update, delete, and get API objects.

Kubernetes also stores its serialized state (currently in etcd) in terms of the API resources.


## docker commands

`docker build -t kubia .`
`docker images`
`docker run --name kubia-container -p 8080:8080 -d kubia`
`docker container ls`
`docker inspect kubia-container`
`docker exec -it kubia-container bash`
`docker stop kubia-container`
`docker rm kubia-container`
`docker tag kubia luksa/kubia`
`docker push luksa/kubia`

