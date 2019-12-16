##[kubectl](https://kubectl.docs.kubernetes.io/)

### kubectl config commands
```
kubectl config set-context my-context --namespace=mystuff
kubectl config use-context my-context
```

### kubectl commands 
```
kubectl get <resource-name>
kubectl get <resource-name> <obj-name> 
kubectl describe <resource-name> <obj-name>
kubectl edit <resource-name> <obj-name>
kubectl delete <resource-name> <obj-name>

kubectl label pods bar color=red        // To add the color=red label to a Pod named bar
kubectl label pods bar color-           // To remove a label by using the <label-name>- syntax

kubectl annotate pod kubia-manual mycompany.com/someannotation="foo bar" 

kubectl scale replicaset kubia --replicas=3
kubectl get replicaset kubia


kubectl apply -f obj.yaml
kubectl delete -f obj.yaml

kubectl logs <pod-name>
kubectl logs <pod-name> --previous
kubectl logs <pod-name> -c <container-name>
kubectl logs <pod-name> -c <container-name> --previous

kubectl exec <pod-name> <command>
kubectl exec -it <pod-name> -- bash
kubectl attach -it <pod-name>

kubectl cp <pod-name>:</path/to/remote/file> </path/to/local/file>
kubectl cp </path/to/local/file> <pod-name>:</path/to/remote/file> 

kubectl port-forward <pod-name> <local port>:<pod port>

kubectl top nodes
kubectl top pods
kubectl top pods --all-namespaces

kubectl help
kubectl help <command-name>
```

* `kubectl get` commands can be used with below flags
```
-o wide 
-o json 
-o yaml
--show-labels
```

* By default, `label` and `annotate` will not let you overwrite an existing label. To do this, you need to add the `--overwrite` flag.

### Extracting specific fields from the object by using JSONPath query language 

```
[fli@192-168-1-10 ~]$ kubectl get pods kubia-596779555c-5vllr -o jsonpath --template={.status.hostIP}
192.168.39.239[fli@192-168-1-10 ~]$ 

[fli@192-168-1-10 ~]$ kubectl get pods kubia-596779555c-5vllr -o jsonpath --template={.status.podIP}
172.17.0.6[fli@192-168-1-10 ~]$ 
```

### Executing a command in a running container
```
[fli@192-168-1-10 ~]$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
kubia-596779555c-5vllr   1/1     Running   2          18h
[fli@192-168-1-10 ~]$

[fli@192-168-1-10 ~]$ kubectl exec -it kubia-596779555c-5vllr -- bash
root@kubia-596779555c-5vllr:/# uname -a
Linux kubia-596779555c-5vllr 4.19.76 #1 SMP Tue Oct 29 14:56:42 PDT 2019 x86_64 GNU/Linux
root@kubia-596779555c-5vllr:/# cat /etc/os-release 
PRETTY_NAME="Debian GNU/Linux 8 (jessie)"
NAME="Debian GNU/Linux"
VERSION_ID="8"
VERSION="8 (jessie)"
ID=debian
HOME_URL="http://www.debian.org/"
SUPPORT_URL="http://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
root@kubia-596779555c-5vllr:/# exit
exit
[fli@192-168-1-10 ~]$ 

[fli@192-168-1-10 ~]$ kubectl exec kubia-596779555c-5vllr cat /etc/os-release
PRETTY_NAME="Debian GNU/Linux 8 (jessie)"
NAME="Debian GNU/Linux"
VERSION_ID="8"
VERSION="8 (jessie)"
ID=debian
HOME_URL="http://www.debian.org/"
SUPPORT_URL="http://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
[fli@192-168-1-10 ~]$ 
```

### Attaching to the running process if don’t have bash or some other terminal available within container
