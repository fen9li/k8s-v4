# [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac#service-account-permissions)

## Enable role based access control

To authenticate API requests, k8s uses the following options: 
* client certificates
* bearer tokens
* authenticating proxy
* HTTP basic auth

### Create a client certificate for user 'developer'

* Create a directory to save the certificates
```
[fli@192-168-1-10 ~]$ mkdir cert && cd cert
```

* Generate a key using OpenSSL
```
[fli@192-168-1-10 cert]$ openssl genrsa -out developer.key 2048
Generating RSA private key, 2048 bit long modulus
..........................+++
.......+++
e is 65537 (0x10001)
[fli@192-168-1-10 cert]$ 
```

* Generate a Client Sign Request (CSR)
```
[fli@192-168-1-10 cert]$ openssl req -new -key developer.key -out developer.csr -subj "/CN=developer/O=devgroup"
[fli@192-168-1-10 cert]$ ll -a
total 12
drwxrwxr-x.  2 fli fli   48 Dec 14 17:33 .
drwx-----x. 33 fli fli 4096 Dec 14 17:31 ..
-rw-rw-r--.  1 fli fli  915 Dec 14 17:33 developer.csr
-rw-rw-r--.  1 fli fli 1679 Dec 14 17:31 developer.key
[fli@192-168-1-10 cert]$ 
```

* Check that the files ca.crt and ca.key exists in `~/.minikube/`
```
[fli@192-168-1-10 ~]$ ls -l .minikube/ca.*
-rw-r--r--. 1 fli fli 1066 Dec  5 17:03 .minikube/ca.crt
-rw-------. 1 fli fli 1675 Dec  5 17:03 .minikube/ca.key
-rwxrwxrwx. 1 fli fli 1029 Dec 14 10:09 .minikube/ca.pem
[fli@192-168-1-10 ~]$  
```

* Generate the certificate (CRT)
```
[fli@192-168-1-10 cert]$ openssl x509 -req -in developer.csr -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -out developer.crt -days 3650
Signature ok
subject=/CN=developer/O=devgroup
Getting CA Private Key
[fli@192-168-1-10 cert]$ 
```

### Create a user 'developer' using this new client certificate
* Set a user entry in kubeconfig for 'developer'
```
[fli@192-168-1-10 cert]$ ll
total 12
-rw-rw-r--. 1 fli fli 1005 Dec 14 17:44 developer.crt
-rw-rw-r--. 1 fli fli  915 Dec 14 17:33 developer.csr
-rw-rw-r--. 1 fli fli 1679 Dec 14 17:31 developer.key
[fli@192-168-1-10 cert]$ kubectl config set-credentials developer --client-certificate=developer.crt --client-key=developer.key
User "developer" set.
[fli@192-168-1-10 cert]$ 
```

* Set a context entry in kubeconfig for 'developer'
```
[fli@192-168-1-10 ~]$ kubectl config set-context developer-context --cluster=minikube --user=developer
Context "developer-context" created.
[fli@192-168-1-10 ~]$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /home/fli/.minikube/ca.crt
    server: https://192.168.39.239:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    user: developer
  name: developer-context
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: ""
kind: Config
preferences: {}
users:
- name: developer
  user:
    client-certificate: /home/fli/cert/developer.crt
    client-key: /home/fli/cert/developer.key
- name: minikube
  user:
    client-certificate: /home/fli/.minikube/client.crt
    client-key: /home/fli/.minikube/client.key
[fli@192-168-1-10 ~]$ 
```

* Switching to the 'developer' user
```
[fli@192-168-1-10 ~]$ kubectl config use-context developer-context
Switched to context "developer-context".
[fli@192-168-1-10 ~]$ kubectl config current-context
developer-context
[fli@192-168-1-10 ~]$ 
```

* User 'developer' is forbidden to list 'pods' in 'default' namespaces 
```
[fli@192-168-1-10 ~]$ kubectl get pods -n default
Error from server (Forbidden): pods is forbidden: User "developer" cannot list resource "pods" in API group "" in the namespace "default"
[fli@192-168-1-10 ~]$  
```

### Grant access to the user

* Create a Role

> Resources: pod, deployment, namespace, secret, configmap, service, persistentvolume ...   
> Verbs: get, list, watch, create, delete, update, edit, exec.   

```
[fli@192-168-1-10 k8s-resources]$ vim role.yaml
[fli@192-168-1-10 k8s-resources]$ cat role.yaml 
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]     # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
[fli@192-168-1-10 k8s-resources]$ 
```

* Create a RoleBinding
```
[fli@192-168-1-10 k8s-resources]$ vim role-binding.yaml
[fli@192-168-1-10 k8s-resources]$ cat role-binding.yaml 
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: developer       # Name is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role            # This must be Role or ClusterRole
  name: pod-reader      # Must match the name of the Role
  apiGroup: rbac.authorization.k8s.io
[fli@192-168-1-10 k8s-resources]$ 
```

* Deploy both the role.yaml and role-binding.yaml to k8s

> Don’t forget to use the context of minikube.     
> We used Role to scope the rules to only one namespace, but we can also use ClusterRole to define more than one namespace. RoleBinding is used to bind to Role and ClusterRoleBinding is used to bind to ClusterRole.    

```
[fli@192-168-1-10 k8s-resources]$ kubectl config current-context
developer-context
[fli@192-168-1-10 k8s-resources]$ kubectl config use-context minikube
Switched to context "minikube".
[fli@192-168-1-10 k8s-resources]$ kubectl config current-context
minikube
[fli@192-168-1-10 k8s-resources]$ 

[fli@192-168-1-10 k8s-resources]$ kubectl apply -f role.yaml
role.rbac.authorization.k8s.io/pod-reader created
[fli@192-168-1-10 k8s-resources]$ kubectl apply -f role-binding.yaml
rolebinding.rbac.authorization.k8s.io/read-pods created
[fli@192-168-1-10 k8s-resources]$ 

[fli@192-168-1-10 k8s-resources]$ kubectl get roles -n default
NAME         AGE
pod-reader   99s
[fli@192-168-1-10 k8s-resources]$ kubectl get rolebindings -n default
NAME        AGE
read-pods   111s
[fli@192-168-1-10 k8s-resources]$    
```

* Testing user 'developer' can read pods in 'default' namespace
```
[fli@192-168-1-10 k8s-resources]$ kubectl config current-context
minikube
[fli@192-168-1-10 k8s-resources]$ kubectl config use-context developer-context
Switched to context "developer-context".
[fli@192-168-1-10 k8s-resources]$ kubectl config current-context
developer-context
[fli@192-168-1-10 k8s-resources]$ kubectl get pods -n default
No resources found in default namespace.
[fli@192-168-1-10 k8s-resources]$ 
```
