
# Google&#x27;s Infrastructure for the Rest of Us
## Why do I need a Kubernetes cluster?
### The roots of containers
### Enter the container
* Cgroups
* Namespaces
* Putting the pieces together
### Here, schedule this...
### The basics of Kubernetes
* The pod
* Labeling all the things
* Replica sets
* Services
## Under the hood
### API server
### Controller manager
### Scheduler
### Kubelet
## Summary

# Start Your Engines
## Your own Kubernetes
### Installation
* macOS
* Linux
* Windows
### Starting Minikube
### First steps with kubectl
### Building Docker containers inside the cluster
## Building and launching a simple application on Minikube
### What just happened?
* Rolling out changes
* Resilience and scaling
### Using the dashboard
### Configuration as code
### Troubleshooting Minikube
## Summary

# Reach for the Cloud
## Cluster architecture
## Creating an AWS account
* Creating an IAM user
* Getting the CLI
* Setting up a key pair
* Preparing the network
## Setting up a bastion

* sshuttle

It is possible to forward traffic from your workstation to the private network by just using SSH. However, we can make accessing servers via the bastion instance much more convenient by using the `sshuttle` tool.

To transparently proxy traffic to the instances inside the private network, we can run the following command:

```
$ sshuttle -r ubuntu@$BASTION_IP 10.0.0.0/16 --dns
[local sudo] password:
client: Connected.
```

Firstly, we pass the SSH login details of our `ubuntu@$BASTION_IP` bastion instance, followed by the CIDR of our VPC (so that only traffic destined for the private network passes over the tunnel); this can be found by running `aws ec2 describe-vpcs`. Finally, we pass the `--dns` flag so that DNS queries on your workstation will be resolved by the DNS servers of the remote instance.

Using `sshuttle` requires you to enter your local sudo password in order to set up its proxy server.

> You might want to run `sshuttle` in a separate terminal or in the background so that you still have access to the shell variables we have been using.    

We can validate that this setup is working correctly by trying to login to our instance through its private DNS name, as follows:

```
$ aws ec2 describe-instances --instance-ids $BASTION_ID --query "Reservations[0].Instances[0].PrivateDnsName"
"ip-10-0-21-138.eu-west-1.compute.internal"
# ssh ubuntu@ip-10-0-21-138.eu-west-1.compute.internal
```

This tests whether you can resolve a DNS entry from the private DNS provided by AWS to intances running within your VPC, and whether the private IP address now returned by that query is reachable.

If you have any difficulties, check `sshuttle` for any connection errors and ensure that you have remembered to enable DNS support in your VPC.


## Instance profiles
## Kubernetes software
* Docker
## Installing Kubeadm
* Building an AMI
## Bootstrapping the cluster
* What just happened?
## Access the API from your workstation
## Setting up pod networking
## Launching worker nodes
## Demo time
## Summary

# Managing Change in Your Applications
## Running pods directly
## Jobs
## CronJob
* Cron syntax
* Concurrency policy
* History limits
## Managing long running processes with deployments
* kubectl patch
* kubectl edit
* kubectl apply
* Kubernetes dashboard
* Greater control of your deployments
* RollingUpdate deployment
* Recreate deployment
## DaemonSet
## Summary

# Managing Complex Applications with Helm
## Installing Helm
* macOS
* Linux and Windows
* Installing Tiller
* Installing a chart
## Configuring a chart
## Creating your own charts
* Chart.yaml
* values.yaml
* templates
* Making it your own
* Developing and debugging
* Templating language
* Functions
* Flow control
## Hooks
## Packaging Helm charts
* You can test building an index
* Using your repository
## Organizational patterns for Helm
* Chart per application
* Shared charts
* Library charts
## Next steps

# Planning for Production
## The design process
* Initial planning
* Planning for success
* Planning for a successful roll out
## Discovering requirements
## Availability
## Capacity
### EC2 instance types
* EC2 instance types
* Breadth versus depth
## Performance
### Disk performance
* gp2
* io2
* st1
* sc1
### Networking
## Security
### Always be updating
* In-place updates
* Immutable images
### Network security
* Infra-node networking
### Node-master networking
* External networking
* Kubernetes infra-pod networking
### IAM roles
### Validation
## Observability
* Logging
* Monitoring
* Blackbox monitoring
* Alerting
* Tracing
## Summary

# A Production-Ready Cluster
## Building a cluster
## Getting started with Terraform
* Variables
* Networking
* Plan and apply
## Control Plane
## Preparing node images
* Installing Packer
* Packer configuration
## Node group
## Provisioning add-ons
## Managing change
## Summary

# Sorry My App Ate the Cluster
## Resource requests and limits
* Resource units
* How pods with resource limits are managed
* Quality of Service (QoS)
* Resource quotas
* Default limits
## Horizontal Pod Autoscaling
* Deploying the metrics server
* Verifying the metrics server and troubleshooting
* Autoscaling pods based on CPU usage
* Autoscaling pods based on other metrics
* Autoscaling the cluster
* Deploying the cluster autoscaler
## Summary

# Storing State
## Volumes
### EBS volumes
### Persistent volumes
* Persistent volumes example
## Storage classes
## StatefulSet
## Summary
## Further reading

# Managing Container Images
## Pushing Docker images to ECR
* Creating a repository
* Pushing and pulling images from your workstation
* Setting up privileges for pushing images
* Use images stored on ECR in Kubernetes
## Tagging images
* Version Control System (VCS) references
* Semantic versions
* Upstream version numbers
## Labelling images
## Summary
