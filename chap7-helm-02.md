## [Quickstart Guide](https://v2.helm.sh/docs/using_helm/#quickstart)

## PREREQUISITES

* A k8s cluster, helm and RBAC
```
[fli@192-168-1-10 ~]$ kubectl version
Client Version: version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.3", GitCommit:"b3cbbae08ec52a7fc73d334838e18d17e8512749", GitTreeState:"clean", BuildDate:"2019-11-13T11:23:11Z", GoVersion:"go1.12.12", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.2", GitCommit:"c97fe5036ef3df2967d086711e6c0c405941e14b", GitTreeState:"clean", BuildDate:"2019-10-15T19:09:08Z", GoVersion:"go1.12.10", Compiler:"gc", Platform:"linux/amd64"}
[fli@192-168-1-10 ~]$ 

[fli@192-168-1-10 ~]$ helm version
Client: &version.Version{SemVer:"v2.16.1", GitCommit:"bbdfe5e7803a12bbdf97e94cd847859890cf4050", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.16.1", GitCommit:"bbdfe5e7803a12bbdf97e94cd847859890cf4050", GitTreeState:"clean"}
[fli@192-168-1-10 ~]$ 

[fli@192-168-1-10 ~]$ kubectl get roles --all-namespaces
NAMESPACE              NAME                                             AGE
kube-public            kubeadm:bootstrap-signer-clusterinfo             8d
kube-public            system:controller:bootstrap-signer               8d
kube-system            extension-apiserver-authentication-reader        8d
kube-system            kube-proxy                                       8d
kube-system            kubeadm:kubelet-config-1.16                      8d
kube-system            kubeadm:nodes-kubeadm-config                     8d
kube-system            system::leader-locking-kube-controller-manager   8d
kube-system            system::leader-locking-kube-scheduler            8d
kube-system            system::nginx-ingress-role                       4d2h
kube-system            system:controller:bootstrap-signer               8d
kube-system            system:controller:cloud-provider                 8d
kube-system            system:controller:token-cleaner                  8d
kubernetes-dashboard   kubernetes-dashboard                             8d
[fli@192-168-1-10 ~]$ 

[fli@192-168-1-10 ~]$ pwd
/home/fli
[fli@192-168-1-10 ~]$ cat .kube/config
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /home/fli/.minikube/ca.crt
    server: https://192.168.39.239:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /home/fli/.minikube/client.crt
    client-key: /home/fli/.minikube/client.key
[fli@192-168-1-10 ~]$ 

[fli@192-168-1-10 ~]$ kubectl config current-context
minikube
[fli@192-168-1-10 ~]$ 

[fli@192-168-1-10 ~]$ kubectl cluster-info
Kubernetes master is running at https://192.168.39.239:8443
KubeDNS is running at https://192.168.39.239:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
[fli@192-168-1-10 ~]$ 
```

## [Role-based Access Control](https://v2.helm.sh/docs/using_helm/#role-based-access-control)

### [Example: Deploy Tiller in a namespace, restricted to deploying resources only in that namespace](https://v2.helm.sh/docs/using_helm/#example-deploy-tiller-in-a-namespace-restricted-to-deploying-resources-only-in-that-namespace)

* Create namespace 'tiller'

```
[fli@192-168-1-10 k8s-resources]$ kubectl config current-context
developer-context
[fli@192-168-1-10 k8s-resources]$ kubectl config use-context minikube
Switched to context "minikube".
[fli@192-168-1-10 k8s-resources]$ kubectl config current-context
minikube
[fli@192-168-1-10 k8s-resources]$ 

[fli@192-168-1-10 k8s-resources]$ vim tiller-namespace.yaml
[fli@192-168-1-10 k8s-resources]$ cat tiller-namespace.yaml 
apiVersion: v1
kind: Namespace
metadata: 
  name: tiller
  labels: 
    name: tiller
[fli@192-168-1-10 k8s-resources]$ kubectl apply -f tiller-namespace.yaml 
namespace/tiller created
[fli@192-168-1-10 k8s-resources]$ 

[fli@192-168-1-10 k8s-resources]$ kubectl get namespaces
NAME                   STATUS   AGE
default                Active   141m
kube-node-lease        Active   141m
kube-public            Active   141m
kube-system            Active   141m
kubernetes-dashboard   Active   140m
tiller                 Active   38s
[fli@192-168-1-10 k8s-resources]$ kubectl describe namespace tiller
Name:         tiller
Labels:       name=tiller
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"v1","kind":"Namespace","metadata":{"annotations":{},"labels":{"name":"tiller"},"name":"tiller"}}
Status:       Active

No resource quota.

No resource limits.
[fli@192-168-1-10 k8s-resources]$ 
```

* Create serviceaccount 'tiller' in namespace 'tiller'

```
[fli@192-168-1-10 k8s-resources]$ vim tiller-service-account.yaml 
[fli@192-168-1-10 k8s-resources]$ cat tiller-service-account.yaml 
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: tiller
automountServiceAccountToken: true
[fli@192-168-1-10 k8s-resources]$ 

[fli@192-168-1-10 k8s-resources]$ kubectl apply -f tiller-service-account.yaml 
serviceaccount/tiller created
[fli@192-168-1-10 k8s-resources]$

[fli@192-168-1-10 k8s-resources]$ kubectl get serviceaccount tiller -n tiller -o yaml
apiVersion: v1
automountServiceAccountToken: true
kind: ServiceAccount
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","automountServiceAccountToken":true,"kind":"ServiceAccount","metadata":{"annotations":{},"name":"tiller","namespace":"tiller"}}
  creationTimestamp: "2019-12-14T09:47:22Z"
  name: tiller
  namespace: tiller
  resourceVersion: "12933"
  selfLink: /api/v1/namespaces/tiller/serviceaccounts/tiller
  uid: 581002a3-0289-4072-8c36-839ff058b3a8
secrets:
- name: tiller-token-cgzlz
[fli@192-168-1-10 k8s-resources]$ 

[fli@192-168-1-10 k8s-resources]$ kubectl get secret tiller-token-cgzlz -n tiller -o yaml
apiVersion: v1
data:
  ca.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQVRBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwdGFXNXAKYTNWaVpVTkJNQjRYRFRFNU1USXdOREEyTURNME0xb1hEVEk1TVRJd01qQTJNRE0wTTFvd0ZURVRNQkVHQTFVRQpBeE1LYldsdWFXdDFZbVZEUVRDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTlhQCmx1bVcrQUdFOEQvaUI5YndnTEtWOVBvb0FZQnZBV3FWNHFmaUVlWTRNYzFmYUs1SlZ0a2Q5TGVuZHpTd0JuS00KZ1pkRDB1NitlMnVVQzI1dG1abnp0dXAwR21Xc2hLOWJxMFBOYndXTXhnOVVFeUI1d055OXMxbkU1S1lUL3c0SQo3dUxTZVFESHZUMWlVQm9hYTN0ZXhEWWlpK04yRGEyK04xUTU1NThraDJ2b2dCb3IwcjdXTytiVDF5d2tkMEVmCjV0VTloTW1HZlBiclNJTjRmVGI3L05TOHdqT3A2SkRXR1IwU1RpdEVGV1RybHFneDM0M1VnZDVOYTUrTUtMbGIKeXRsQ2JnY0ViT0lEWVlsckxkMXJyQU9UNEgzVzJrb3dyTHBZQTdycnRqUEJxeGduREJmZ3ZxbEtOaDluQ0VFZworWi9yNERjVUFQTTFBVm93QjRFQ0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUIwR0ExVWRKUVFXCk1CUUdDQ3NHQVFVRkJ3TUNCZ2dyQmdFRkJRY0RBVEFQQmdOVkhSTUJBZjhFQlRBREFRSC9NQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFBa1IxL2l4dFVCS1BSYXFxYW1GczM5YzBKcXV6OENQYXNGTG16endGdlZCKzYySGR3TApVVU9DTC9yWENkMnZxcFBpV01LaWlmTldwVXBwaVlEc1d1VjBaZlQvOE1GdjVBZkNmeDZxT0FYZS9Yd3ZDckhVCnJzbk1weU1GMTVVOFQ2MTQyVE1GSkF1a1ZIeUs2Z1RJMVhMdk1zUFhMcFN3QVJGU21Pa0hMMEZTZWI1dXpxWkgKcW0vL1NxT2J1Rit2TzRLL0RCSk9DWEJhQ1IzLzByazBSb25jblJNbVlPdVRaUUliZnRHME1LNzIzSnh0VFU0awo5cnBLRHRreEZoaGUyeHdsSnQwWFpsbUVnQjdUNm5BdFZXaWV1elhML0hZb3pjNEtyV1dqZ1llVUZHTGV3QU02ClVBdXlDVmYzRmpyd1BBRGlzb20xTGw0dU1nT1Q4N21xT09xNAotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
  namespace: dGlsbGVy
  token: ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNklreENkV05UVUhWclkwbEhWekpQV0hkbFFWSXRibTgyTnpaVFoxRm1URm81ZUdGM2JsaEdOM0ozY0ZVaWZRLmV5SnBjM01pT2lKcmRXSmxjbTVsZEdWekwzTmxjblpwWTJWaFkyTnZkVzUwSWl3aWEzVmlaWEp1WlhSbGN5NXBieTl6WlhKMmFXTmxZV05qYjNWdWRDOXVZVzFsYzNCaFkyVWlPaUowYVd4c1pYSWlMQ0pyZFdKbGNtNWxkR1Z6TG1sdkwzTmxjblpwWTJWaFkyTnZkVzUwTDNObFkzSmxkQzV1WVcxbElqb2lkR2xzYkdWeUxYUnZhMlZ1TFdObmVteDZJaXdpYTNWaVpYSnVaWFJsY3k1cGJ5OXpaWEoyYVdObFlXTmpiM1Z1ZEM5elpYSjJhV05sTFdGalkyOTFiblF1Ym1GdFpTSTZJblJwYkd4bGNpSXNJbXQxWW1WeWJtVjBaWE11YVc4dmMyVnlkbWxqWldGalkyOTFiblF2YzJWeWRtbGpaUzFoWTJOdmRXNTBMblZwWkNJNklqVTRNVEF3TW1FekxUQXlPRGt0TkRBM01pMDRZek0yTFRnek9XWm1NRFU0WWpOaE9DSXNJbk4xWWlJNkluTjVjM1JsYlRwelpYSjJhV05sWVdOamIzVnVkRHAwYVd4c1pYSTZkR2xzYkdWeUluMC5xREZXUTlCNmhNdnhLXzlGUGt4SGFERmVlM1N4ejl5T2RFYWxMdUdEWmw0b1JITkliTVN5X0tOYVdLMU8xSFl1eVdSQmtPOXJQU2ZTUk45Sk9JMHl3T3p6VUxHNVctcnpac1k4aTIzMXp3d25jclBpU0RWV21hRFRXeVpVT19ZUEtFRjNQNTZTVXUxMHlTNTdQZmZKWkY1QU5QY2hIMFBfbUdJTUZjZlFuOU15OXZTVUYzVHV1UDBiMUpiWmRIOWNyeThoSmZZampQalZsd0VpMVpWZnNEdmZaS1pyaUVFdzBmamdyYnBJNkJDT2JfVHVYd1Q5d0lkOVVJT05aNGQ4eHpyUUZMdHFuV01rT256VDRERUFlVVh6RVZBT1Y1TWZnVWw5UEFLd0lzcGNVZi01OVl5QWdtQlhJeWcxYUpib3BaRWpQX1NwM291NTRQYXFVMG5RblE=
kind: Secret
metadata:
  annotations:
    kubernetes.io/service-account.name: tiller
    kubernetes.io/service-account.uid: 581002a3-0289-4072-8c36-839ff058b3a8
  creationTimestamp: "2019-12-14T09:47:22Z"
  name: tiller-token-cgzlz
  namespace: tiller
  resourceVersion: "12932"
  selfLink: /api/v1/namespaces/tiller/secrets/tiller-token-cgzlz
  uid: 33706452-f50c-4756-b1c9-0131bcfec652
type: kubernetes.io/service-account-token
[fli@192-168-1-10 k8s-resources]$ 
```

* Create a Role and RoleBinding that allows 'tiller' to manage all resources in 'tiller'
```
[fli@192-168-1-10 k8s-resources]$ vim tiller-role.yaml
[fli@192-168-1-10 k8s-resources]$ cat tiller-role.yaml 
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: tiller
  namespace: tiller
rules:
- apiGroups: ["", "batch", "extensions", "apps"]
  resources: ["*"]
  verbs: ["*"]
[fli@192-168-1-10 k8s-resources]$ 

[fli@192-168-1-10 k8s-resources]$ vim tiller-rolebinding.yaml
[fli@192-168-1-10 k8s-resources]$ cat tiller-rolebinding.yaml 
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: tiller
  namespace: tiller
subjects:
- kind: ServiceAccount
  name: tiller
  namespace: tiller
roleRef:
  kind: Role
  name: tiller
  apiGroup: rbac.authorization.k8s.io
[fli@192-168-1-10 k8s-resources]$ 

[fli@192-168-1-10 k8s-resources]$ kubectl apply -f tiller-role.yaml 
role.rbac.authorization.k8s.io/tiller created
[fli@192-168-1-10 k8s-resources]$ kubectl apply -f tiller-rolebinding.yaml 
rolebinding.rbac.authorization.k8s.io/tiller created
[fli@192-168-1-10 k8s-resources]$ 
```

* Install Tiller in the 'tiller' namespace and test it all good

```
[fli@192-168-1-10 k8s-resources]$ helm init --service-account tiller --tiller-namespace tiller
$HELM_HOME has been configured at /home/fli/.helm.

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.

Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.
To prevent this, run `helm init` with the --tiller-tls-verify flag.
For more information on securing your installation see: https://docs.helm.sh/using_helm/#securing-your-helm-installation
[fli@192-168-1-10 k8s-resources]$ 

[fli@192-168-1-10 k8s-resources]$ kubectl get all -n tiller
NAME                                 READY   STATUS    RESTARTS   AGE
pod/tiller-deploy-6b6ff5dd96-5bwdp   1/1     Running   0          73s

NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)     AGE
service/tiller-deploy   ClusterIP   10.97.253.165   <none>        44134/TCP   73s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/tiller-deploy   1/1     1            1           73s

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/tiller-deploy-6b6ff5dd96   1         1         1       73s
[fli@192-168-1-10 k8s-resources]$ 

[fli@192-168-1-10 k8s-resources]$ helm --tiller-namespace tiller version 
Client: &version.Version{SemVer:"v2.16.1", GitCommit:"bbdfe5e7803a12bbdf97e94cd847859890cf4050", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.16.1", GitCommit:"bbdfe5e7803a12bbdf97e94cd847859890cf4050", GitTreeState:"clean"}
[fli@192-168-1-10 k8s-resources]$ 
```

### Test install stable/mysql in namespace 'tiller'

* Install mysql
```
[fli@192-168-1-10 k8s-resources]$ helm install stable/mysql --tiller-namespace tiller --namespace tiller
NAME:   dining-pike
LAST DEPLOYED: Sat Dec 14 22:53:24 2019
NAMESPACE: tiller
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME                    AGE
dining-pike-mysql-test  1s

==> v1/Deployment
NAME               AGE
dining-pike-mysql  1s

==> v1/PersistentVolumeClaim
NAME               AGE
dining-pike-mysql  1s

==> v1/Pod(related)
NAME                                AGE
dining-pike-mysql-7cf7964c87-phqnw  1s

==> v1/Secret
NAME               AGE
dining-pike-mysql  1s

==> v1/Service
NAME               AGE
dining-pike-mysql  1s


NOTES:
MySQL can be accessed via port 3306 on the following DNS name from within your cluster:
dining-pike-mysql.tiller.svc.cluster.local

To get your root password run:

    MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace tiller dining-pike-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)

To connect to your database:

1. Run an Ubuntu pod that you can use as a client:

    kubectl run -i --tty ubuntu --image=ubuntu:16.04 --restart=Never -- bash -il

2. Install the mysql client:

    $ apt-get update && apt-get install mysql-client -y

3. Connect using the mysql cli, then provide your password:
    $ mysql -h dining-pike-mysql -p

To connect to your database directly from outside the K8s cluster:
    MYSQL_HOST=127.0.0.1
    MYSQL_PORT=3306

    # Execute the following command to route the connection:
    kubectl port-forward svc/dining-pike-mysql 3306

    mysql -h ${MYSQL_HOST} -P${MYSQL_PORT} -u root -p${MYSQL_ROOT_PASSWORD}
    

[fli@192-168-1-10 k8s-resources]$ 

[fli@192-168-1-10 k8s-resources]$ helm ls --tiller-namespace tiller --namespace tiller
NAME       	REVISION	UPDATED                 	STATUS  	CHART      	APP VERSION	NAMESPACE
dining-pike	1       	Sat Dec 14 22:53:24 2019	DEPLOYED	mysql-1.6.1	5.7.27     	tiller   
[fli@192-168-1-10 k8s-resources]$ 
```

* Get the mysql server root password

```
[fli@192-168-1-10 k8s-resources]$ MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace tiller dining-pike-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)
[fli@192-168-1-10 k8s-resources]$ echo $MYSQL_ROOT_PASSWORD
Y9j17uhdEX
[fli@192-168-1-10 k8s-resources]$ 
```

* Connect to mysql service successfully
```
[fli@192-168-1-10 k8s-resources]$ kubectl get all -n tiller
NAME                                     READY   STATUS    RESTARTS   AGE
pod/dining-pike-mysql-7cf7964c87-phqnw   1/1     Running   0          21m
pod/tiller-deploy-6b6ff5dd96-5bwdp       1/1     Running   0          113m

NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)     AGE
service/dining-pike-mysql   ClusterIP   10.110.4.123    <none>        3306/TCP    21m
service/tiller-deploy       ClusterIP   10.97.253.165   <none>        44134/TCP   113m

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/dining-pike-mysql   1/1     1            1           21m
deployment.apps/tiller-deploy       1/1     1            1           113m

NAME                                           DESIRED   CURRENT   READY   AGE
replicaset.apps/dining-pike-mysql-7cf7964c87   1         1         1       21m
replicaset.apps/tiller-deploy-6b6ff5dd96       1         1         1       113m
[fli@192-168-1-10 k8s-resources]$ 
[fli@192-168-1-10 k8s-resources]$ kubectl port-forward svc/dining-pike-mysql 3306 -n tiller &
[1] 13056
[fli@192-168-1-10 k8s-resources]$ Forwarding from 127.0.0.1:3306 -> 3306
Forwarding from [::1]:3306 -> 3306

[fli@192-168-1-10 k8s-resources]$ 

[fli@192-168-1-10 k8s-resources]$ MYSQL_HOST=127.0.0.1
[fli@192-168-1-10 k8s-resources]$ echo $MYSQL_HOST
127.0.0.1
[fli@192-168-1-10 k8s-resources]$ MYSQL_PORT=3306
[fli@192-168-1-10 k8s-resources]$ echo $MYSQL_PORT
3306
[fli@192-168-1-10 k8s-resources]$ echo $MYSQL_ROOT_PASSWORD
Y9j17uhdEX
[fli@192-168-1-10 k8s-resources]$ 

[fli@192-168-1-10 k8s-resources]$ mysql -h ${MYSQL_HOST} -P${MYSQL_PORT} -u root -p${MYSQL_ROOT_PASSWORD}
Handling connection for 3306
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 330
Server version: 5.7.14 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.01 sec)

MySQL [(none)]> QUIT;
Bye
[fli@192-168-1-10 k8s-resources]$ 
```

* Delete mysql

```
[fli@192-168-1-10 k8s-resources]$ helm ls --tiller-namespace tiller --namespace tiller
NAME       	REVISION	UPDATED                 	STATUS  	CHART      	APP VERSION	NAMESPACE
dining-pike	1       	Sat Dec 14 22:53:24 2019	DEPLOYED	mysql-1.6.1	5.7.27     	tiller   
[fli@192-168-1-10 k8s-resources]$ 

[fli@192-168-1-10 k8s-resources]$ helm delete dining-pike --tiller-namespace tiller
release "dining-pike" deleted
[fli@192-168-1-10 k8s-resources]$ helm ls --tiller-namespace tiller --namespace tiller
[fli@192-168-1-10 k8s-resources]$ kubectl get all -n tiller
NAME                                 READY   STATUS    RESTARTS   AGE
pod/tiller-deploy-6b6ff5dd96-5bwdp   1/1     Running   0          133m

NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)     AGE
service/tiller-deploy   ClusterIP   10.97.253.165   <none>        44134/TCP   133m

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/tiller-deploy   1/1     1            1           133m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/tiller-deploy-6b6ff5dd96   1         1         1       133m
[fli@192-168-1-10 k8s-resources]$  
```
