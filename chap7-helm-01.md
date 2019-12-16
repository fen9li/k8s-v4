# Helm, the Package Manager for Kubernetes

[How the documentation is organized]https://helm.sh/docs/

## Install Helm

```
[fli@192-168-1-10 ~]$ curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  7164  100  7164    0     0  14044      0 --:--:-- --:--:-- --:--:-- 14074
[fli@192-168-1-10 ~]$ chmod 700 get_helm.sh
[fli@192-168-1-10 ~]$ ./get_helm.sh 
Downloading https://get.helm.sh/helm-v2.16.1-linux-amd64.tar.gz
Preparing to install helm and tiller into /usr/local/bin
[sudo] password for fli: password
helm installed into /usr/local/bin/helm
tiller installed into /usr/local/bin/tiller
Run 'helm init' to configure helm.
[fli@192-168-1-10 ~]$ 

[fli@192-168-1-10 ~]$ sudo ls -l /usr/local/bin
total 297068
-rwxr-xr-x. 1 root root     11939128 Nov 12 18:13 docker-machine-driver-kvm
-rwxr-xr-x. 1 root root     40460288 Dec 12 16:07 helm
-rwxrwxr-x. 1 fli  fli      46681472 Dec  5 17:13 kubectl
-rwxr-xr-x. 1 fli  libvirt  28037312 Jul 16 23:07 minishift
-rwxr-xr-x. 1 fli  libvirt 120350344 Oct 11  2018 oc
-rwxr-xr-x. 1 fli  fli       9941088 Oct 19 06:27 s2i
-rwxr-xr-x. 1 root root     41127936 Dec 12 16:07 tiller
-rwxrwxr-x. 1 root root      5644727 Nov  1 12:04 yq
[fli@192-168-1-10 ~]$ 
```

## Configure Helm And Tiller minikube
```
[fli@192-168-1-10 ~]$ minikube start
🎉  minikube 1.6.1 is available! Download it: https://github.com/kubernetes/minikube/releases/tag/v1.6.1
💡  To disable this notice, run: 'minikube config set WantUpdateNotification false'

🙄  minikube v1.5.2 on Centos 7.6.1810
💡  Tip: Use 'minikube start -p <name>' to create a new cluster, or 'minikube delete' to delete this one.
🔄  Starting existing kvm2 VM for "minikube" ...
⌛  Waiting for the host to be provisioned ...
🐳  Preparing Kubernetes v1.16.2 on Docker '18.09.9' ...
🔄  Relaunching Kubernetes using kubeadm ... 
⌛  Waiting for: apiserver
🏄  Done! kubectl is now configured to use "minikube"
[fli@192-168-1-10 ~]$ 
[fli@192-168-1-10 ~]$ kubectl get pods -n kube-system
NAME                                        READY   STATUS    RESTARTS   AGE
coredns-5644d7b6d9-c7qrx                    1/1     Running   9          6d22h
coredns-5644d7b6d9-jjs4h                    1/1     Running   9          6d22h
etcd-minikube                               1/1     Running   9          6d22h
kube-addon-manager-minikube                 1/1     Running   9          6d22h
kube-apiserver-minikube                     1/1     Running   9          6d22h
kube-controller-manager-minikube            1/1     Running   12         6d22h
kube-proxy-tdbpb                            1/1     Running   9          6d22h
kube-scheduler-minikube                     1/1     Running   12         6d22h
nginx-ingress-controller-6fc5bcc8c9-gh5vw   1/1     Running   1          2d1h
storage-provisioner                         1/1     Running   9          6d22h
[fli@192-168-1-10 ~]$ 

[fli@192-168-1-10 ~]$ eval $(minikube docker-env)

[fli@192-168-1-10 ~]$ helm init --service-account tiller
Creating /home/fli/.helm 
Creating /home/fli/.helm/repository 
Creating /home/fli/.helm/repository/cache 
Creating /home/fli/.helm/repository/local 
Creating /home/fli/.helm/plugins 
Creating /home/fli/.helm/starters 
Creating /home/fli/.helm/cache/archive 
Creating /home/fli/.helm/repository/repositories.yaml 
Adding stable repo with URL: https://kubernetes-charts.storage.googleapis.com 
Adding local repo with URL: http://127.0.0.1:8879/charts 
$HELM_HOME has been configured at /home/fli/.helm.

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.

Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.
To prevent this, run `helm init` with the --tiller-tls-verify flag.
For more information on securing your installation see: https://docs.helm.sh/using_helm/#securing-your-helm-installation
[fli@192-168-1-10 ~]$ 
```

## For some reason, Tiller deploy doesnt scheduled

* Check helm tiller deployment and service

```
[fli@192-168-1-10 ~]$ kubectl -n kube-system get deployments
NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
coredns                    2/2     2            2           6d23h
nginx-ingress-controller   1/1     1            1           2d2h
tiller-deploy              0/1     0            0           18m
[fli@192-168-1-10 ~]$ 
 
[fli@192-168-1-10 ~]$ kubectl -n kube-system get svc
NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
kube-dns        ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP   6d23h
tiller-deploy   ClusterIP   10.102.219.205   <none>        44134/TCP                33m
[fli@192-168-1-10 ~]$ 

[fli@192-168-1-10 ~]$ kubectl -n kube-system describe deployment tiller-deploy
Name:                   tiller-deploy
Namespace:              kube-system
CreationTimestamp:      Thu, 12 Dec 2019 16:09:17 +1100
Labels:                 app=helm
                        name=tiller
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=helm,name=tiller
Replicas:               1 desired | 0 updated | 0 total | 0 available | 1 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:           app=helm
                    name=tiller
  Service Account:  tiller
  Containers:
   tiller:
    Image:       gcr.io/kubernetes-helm/tiller:v2.16.1
    Ports:       44134/TCP, 44135/TCP
    Host Ports:  0/TCP, 0/TCP
    Liveness:    http-get http://:44135/liveness delay=1s timeout=1s period=10s #success=1 #failure=3
    Readiness:   http-get http://:44135/readiness delay=1s timeout=1s period=10s #success=1 #failure=3
    Environment:
      TILLER_NAMESPACE:    kube-system
      TILLER_HISTORY_MAX:  0
    Mounts:                <none>
  Volumes:                 <none>
Conditions:
  Type             Status  Reason
  ----             ------  ------
  Available        False   MinimumReplicasUnavailable
  ReplicaFailure   True    FailedCreate
  Progressing      False   ProgressDeadlineExceeded
OldReplicaSets:    <none>
NewReplicaSet:     tiller-deploy-969865475 (0/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  22m   deployment-controller  Scaled up replica set tiller-deploy-969865475 to 1
[fli@192-168-1-10 ~]$ 
```

* serviceaccount `tiller` missing, which caused the installation failed
```
[fli@192-168-1-10 ~]$ kubectl -n kube-system get serviceaccount
NAME                                 SECRETS   AGE
attachdetach-controller              1         6d23h
bootstrap-signer                     1         6d23h
certificate-controller               1         6d23h
clusterrole-aggregation-controller   1         6d23h
coredns                              1         6d23h
cronjob-controller                   1         6d23h
daemon-set-controller                1         6d23h
default                              1         6d23h
deployment-controller                1         6d23h
disruption-controller                1         6d23h
endpoint-controller                  1         6d23h
expand-controller                    1         6d23h
generic-garbage-collector            1         6d23h
horizontal-pod-autoscaler            1         6d23h
job-controller                       1         6d23h
kube-proxy                           1         6d23h
namespace-controller                 1         6d23h
nginx-ingress                        1         2d2h
node-controller                      1         6d23h
persistent-volume-binder             1         6d23h
pod-garbage-collector                1         6d23h
pv-protection-controller             1         6d23h
pvc-protection-controller            1         6d23h
replicaset-controller                1         6d23h
replication-controller               1         6d23h
resourcequota-controller             1         6d23h
service-account-controller           1         6d23h
service-controller                   1         6d23h
statefulset-controller               1         6d23h
storage-provisioner                  1         6d23h
token-cleaner                        1         6d23h
ttl-controller                       1         6d23h
[fli@192-168-1-10 ~]$ 

[fli@192-168-1-10 ~]$ kubectl -n kube-system get serviceaccount | grep tiller
[fli@192-168-1-10 ~]$ 
```

* Delete `tiller-deploy` deployment and service, rerun `helm init --upgrade` and see it works

```
[fli@192-168-1-10 ~]$ kubectl -n kube-system delete deployment tiller-deploy
deployment.apps "tiller-deploy" deleted
[fli@192-168-1-10 ~]$ 
[fli@192-168-1-10 ~]$ kubectl -n kube-system delete svc tiller-deploy
service "tiller-deploy" deleted
[fli@192-168-1-10 ~]$ 

[fli@192-168-1-10 ~]$ helm init --upgrade
$HELM_HOME has been configured at /home/fli/.helm.

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.

Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.
To prevent this, run `helm init` with the --tiller-tls-verify flag.
For more information on securing your installation see: https://docs.helm.sh/using_helm/#securing-your-helm-installation
[fli@192-168-1-10 ~]$ 

[fli@192-168-1-10 ~]$ kubectl -n kube-system get pods
NAME                                        READY   STATUS    RESTARTS   AGE
coredns-5644d7b6d9-c7qrx                    1/1     Running   9          6d23h
coredns-5644d7b6d9-jjs4h                    1/1     Running   9          6d23h
etcd-minikube                               1/1     Running   9          6d23h
kube-addon-manager-minikube                 1/1     Running   9          6d23h
kube-apiserver-minikube                     1/1     Running   9          6d23h
kube-controller-manager-minikube            1/1     Running   12         6d23h
kube-proxy-tdbpb                            1/1     Running   9          6d23h
kube-scheduler-minikube                     1/1     Running   12         6d23h
nginx-ingress-controller-6fc5bcc8c9-gh5vw   1/1     Running   1          2d2h
storage-provisioner                         1/1     Running   9          6d23h
tiller-deploy-b747845f-rtlrr                1/1     Running   0          49s
[fli@192-168-1-10 ~]$ 
```

* double check it
```
[fli@192-168-1-10 ~]$ kubectl -n kube-system describe pods tiller-deploy-b747845f-rtlrr
Name:         tiller-deploy-b747845f-rtlrr
Namespace:    kube-system
Priority:     0
Node:         minikube/192.168.39.239
Start Time:   Thu, 12 Dec 2019 16:46:11 +1100
Labels:       app=helm
              name=tiller
              pod-template-hash=b747845f
Annotations:  <none>
Status:       Running
IP:           172.17.0.8
IPs:
  IP:           172.17.0.8
Controlled By:  ReplicaSet/tiller-deploy-b747845f
Containers:
  tiller:
    Container ID:   docker://a9ae7104edb5e5bdb3db08adec999539f5c70f184b998e424ff3c65be9d3990d
    Image:          gcr.io/kubernetes-helm/tiller:v2.16.1
    Image ID:       docker-pullable://gcr.io/kubernetes-helm/tiller@sha256:3c70ee359d3ec305ca469395a2481b2375d569c6b4a928389ca07d829d12ec51
    Ports:          44134/TCP, 44135/TCP
    Host Ports:     0/TCP, 0/TCP
    State:          Running
      Started:      Thu, 12 Dec 2019 16:46:41 +1100
    Ready:          True
    Restart Count:  0
    Liveness:       http-get http://:44135/liveness delay=1s timeout=1s period=10s #success=1 #failure=3
    Readiness:      http-get http://:44135/readiness delay=1s timeout=1s period=10s #success=1 #failure=3
    Environment:
      TILLER_NAMESPACE:    kube-system
      TILLER_HISTORY_MAX:  0
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-qt4xt (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-qt4xt:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-qt4xt
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age        From               Message
  ----    ------     ----       ----               -------
  Normal  Scheduled  <unknown>  default-scheduler  Successfully assigned kube-system/tiller-deploy-b747845f-rtlrr to minikube
  Normal  Pulling    4m10s      kubelet, minikube  Pulling image "gcr.io/kubernetes-helm/tiller:v2.16.1"
  Normal  Pulled     3m46s      kubelet, minikube  Successfully pulled image "gcr.io/kubernetes-helm/tiller:v2.16.1"
  Normal  Created    3m44s      kubelet, minikube  Created container tiller
  Normal  Started    3m43s      kubelet, minikube  Started container tiller
[fli@192-168-1-10 ~]$ 

[fli@192-168-1-10 ~]$ kubectl -n kube-system get deployment
NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
coredns                    2/2     2            2           6d23h
nginx-ingress-controller   1/1     1            1           2d2h
tiller-deploy              1/1     1            1           4m6s
[fli@192-168-1-10 ~]$ kubectl -n kube-system describe deployment tiller-deploy
Name:                   tiller-deploy
Namespace:              kube-system
CreationTimestamp:      Thu, 12 Dec 2019 16:46:11 +1100
Labels:                 app=helm
                        name=tiller
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=helm,name=tiller
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=helm
           name=tiller
  Containers:
   tiller:
    Image:       gcr.io/kubernetes-helm/tiller:v2.16.1
    Ports:       44134/TCP, 44135/TCP
    Host Ports:  0/TCP, 0/TCP
    Liveness:    http-get http://:44135/liveness delay=1s timeout=1s period=10s #success=1 #failure=3
    Readiness:   http-get http://:44135/readiness delay=1s timeout=1s period=10s #success=1 #failure=3
    Environment:
      TILLER_NAMESPACE:    kube-system
      TILLER_HISTORY_MAX:  0
    Mounts:                <none>
  Volumes:                 <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   tiller-deploy-b747845f (1/1 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  5m28s  deployment-controller  Scaled up replica set tiller-deploy-b747845f to 1
[fli@192-168-1-10 ~]$ 

[fli@192-168-1-10 ~]$ helm version
Client: &version.Version{SemVer:"v2.16.1", GitCommit:"bbdfe5e7803a12bbdf97e94cd847859890cf4050", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.16.1", GitCommit:"bbdfe5e7803a12bbdf97e94cd847859890cf4050", GitTreeState:"clean"}
[fli@192-168-1-10 ~]$ 
```

## Generate 1st Chart

* Scaffolding out
```
[fli@192-168-1-10 helm-charts]$ pwd
/home/fli/kubernetes/helm-charts
[fli@192-168-1-10 helm-charts]$ 

[fli@192-168-1-10 helm-charts]$ helm create testchart
Creating testchart
[fli@192-168-1-10 helm-charts]$

[fli@192-168-1-10 helm-charts]$ tree
.
└── testchart
    ├── charts
    ├── Chart.yaml
    ├── templates
    │   ├── deployment.yaml
    │   ├── _helpers.tpl
    │   ├── ingress.yaml
    │   ├── NOTES.txt
    │   ├── serviceaccount.yaml
    │   ├── service.yaml
    │   └── tests
    │       └── test-connection.yaml
    └── values.yaml

4 directories, 9 files
[fli@192-168-1-10 helm-charts]$ 

```

* Deploy it
```
[fli@192-168-1-10 helm-charts]$ helm install --name test ./testchart --set service.type=NodePort
NAME:   test
LAST DEPLOYED: Thu Dec 12 20:18:49 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Deployment
NAME            AGE
test-testchart  0s

==> v1/Pod(related)
NAME                            AGE
test-testchart-78659dd75-cwr7f  0s

==> v1/Service
NAME            AGE
test-testchart  0s

==> v1/ServiceAccount
NAME            AGE
test-testchart  0s


NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services test-testchart)
  export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT

[fli@192-168-1-10 helm-charts]$ 
[fli@192-168-1-10 helm-charts]$   export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services test-testchart)
[fli@192-168-1-10 helm-charts]$   export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
[fli@192-168-1-10 helm-charts]$   echo http://$NODE_IP:$NODE_PORT
http://192.168.39.239:30627
[fli@192-168-1-10 helm-charts]$

[fli@192-168-1-10 kubernetes]$ curl http://192.168.39.239:30627
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
[fli@192-168-1-10 kubernetes]$ 

[fli@192-168-1-10 kubernetes]$ kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
test-testchart-78659dd75-cwr7f   1/1     Running   0          3m3s
[fli@192-168-1-10 kubernetes]$ kubectl get svc
NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes       ClusterIP   10.96.0.1        <none>        443/TCP          7d3h
test-testchart   NodePort    10.110.183.3     <none>        80:30627/TCP     3m21s
[fli@192-168-1-10 kubernetes]$ kubectl get deployment
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
test-testchart   1/1     1            1           3m39s
[fli@192-168-1-10 kubernetes]$ 
```
