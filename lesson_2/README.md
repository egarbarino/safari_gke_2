# Lesson 2 - Pods

Segment TMUX Setup

```
|------------------|
|        1         |
|------------------|
|        2         |
|------------------|
| 3.left | 3.right | 
|------------------|
|        4         |
--------------------
```

Pane 1

```
watch -n 1 kubectl get pod -o wide
```

Pane 2

```
kubectl get event --watch
```

Pane 3.left: Leave empty for now

Pane 3.right: Leave empty for now

Panel 4

```
cd ~/safari_gke_2/lesson_2/
```

## Lesson 2: Pods

Welcome to Lesson 2, Pods. A pod is the most fundamental and atomic resource type in Kubernetes. A Pod wraps a container so that can we can define extra properties that help describe how the container runs and behaves in our environment. 

This lesson is organized into five sublessons:

- 2.1 Launching Docker Containers Using Pods
- 2.2 Managing the Pod’s Lifecycle
- 2.3 Implementing Self-Healing Mechanisms
- 2.4 Segregating Workloads Using Namespaces
- 2.5 Selecting Objects Using Labels

## Learning Objectives

By the end of this lesson, you’ll understand:

- How to launch Docker containers wrapped as Pods using the the kubectl run command
- How to manage the Pod’s life cycle by defining properties in the Pod's manifest
- How to implement self-healing mechanisms using the Readiness and Liveness Probes
- How to segregate workloads by creating and selecting labels and namespaces

and finally,

- How to select objects using namespaces and labels flags

## 2.1 Launching Docker Containers Using Pods

The first thing we need to do is learn how to launch Docker containers using the Kubernetes Pod mechanism.

## Launching Docker Containers Using Pods (Architecture)

We return again to our Kubernetes architecture diagram.

As we can see, a Pod is a resource that gets deployed onto a given Kubernetes worker node. In turn, a Pod wraps one or more Docker containers. Most Pods, and the ones that we will configure and launch in this course contain only a single Docker container.

Containers are normally hosted in so-called container registry. We'll be using popular containers from Docker Hub, which is the largest public container registry.

We'll look at three use cases involving the running of containers using the Pod mechanism.

The first use is case is that of running one off commands such as the date command.
The second use case is that of creating a tiny steady virtual machine that we can log onto.

The third case, and most common one, is that of a web server or microservice.

We'll also be looking at the use of a Pod manifest, which is a configuration file in which can establish the port on which each container will mapped to, what labels it has, and where to find its image in a container registry.

Let's now see all of these use cases in practice...

### (Start Share) Google Cloud Shell


### Run a Single Command

Obtain date using Docker Hub's alpine image

```
kubectl run my-pod --image=alpine --restart=Never --attach --rm date
```


### Run a One-Off Shell

```
kubectl run my-pod --image=alpine --restart=Never -it --rm sh
```

Experiments inside shell

- Launch various background processes such as 'sleep 30 &'
- Try hostname
- Fetch an Internet resource using 'wget'
- Check IP address using 'ifconfig eth0' and 

Experiments outside

Show 'kubectl get pod/my-pod -o yaml' outside

### Running 'Steady' Pods

**This won't work**

This command won't complete. The pod will keep crashing
and will need to be killed with `kubectl delete pod/my-pod`

```
kubectl run my-pod --image=alpine sh 
```

Delete it!

```
kubectl delete pod/my-pod
```

**This works instead**

This will leave Pod running in the background

```
kubectl run my-pod --image=alpine sleep infinity 
```

Run `date` on container

```
kubectl exec my-pod -- date
```

Open shell

```
kubectl exec my-pod -ti -- sh
```

Experiments

- Exit and enter shell again 
- Delete Pod

### Running a Web Server

Launch web server

```
kubectl run nginx --image=nginx 
```

Prove that is it a steady Pod like any other

```
kubectl exec nginx -- date
```

Pane 3.left: Watch logs

```
kubectl logs -f nginx
```

Pane 3.right: Establish port-forwarding

```
kubectl port-forward nginx 8080:80
```

Generate web requests

```
curl http://127.0.0.1:8080
```

Further experiments:

- Attach to nginx using `kubectl attach nginx` (and generate more web requests!)
- Detach by pressing CTRL+C
- Destroy Pod

### Pod Manifest

Pane 3.left: Stop and clear

Pane 3.right: Stop and clear

Pane 4

```
kubectl delete pod/nginx
```

Kubectl run can be seen as creating a Pod manifest on the fly

```
kubectl run nginx --image=nginx --restart=Never \
    --dry-run=client -o yaml
```   

Save manifest

```
kubectl run nginx --image=nginx --restart=Never \
    --dry-run=client -o yaml > /tmp/nginx.yaml
```  

Apply

```
kubectl apply -f /tmp/nginx.yaml
```

### (Stop Sharing)


## 2.3 Managing the Pod's Life Cycle

Now that we've seen how to run Pods in different modes, let's delve into the Pod's life cycle.

## Managing The Pod's Life Cycle

At a high level, and specially, when querying the status of a Pod from the command line, we normally see the Pod's status in the Pending state when they are initializing, and in the Running state once the initialization has completed. 

The Pod then terminates either successfully or with a failed status.

When entering the Pending state, the first think that happens is the mounting of storage. Containers have their own virtual file system but we may also mount block devices, network file systems, virtual configuration drives and much more. All of this takes place first.

Once all applicable file systems are mounted, we have the opportunity to run initialization containers. These are containers other than the primary container which we may want to run to initialize the Pod.

When entering the Running state, we can specify our own startup command to override whatever initialization process comes by default with the container. 

But that's not the only thing we can do. Right after the container's command start, we can launch a _secondary_ process using the PostStart hook. 

At this point the Pod is running steadily but we may decide to delete it, or perhaps a scaling strategy may decide that it is no longer necessary. Whatever the reason, we can intercept any termination event using the PreStop hook, which allow us to perform some cleanup or run a graceful shutdown process within a grace period window. 

Let's now explore some of these options

### Screen Share


### Startup Arguments, PostStart and PreStop Hooks

Pane 3.left:

```
watch -n 1 "gcloud compute disks list | grep NAME"
```

Pane 4: Create an external disk first

```
gcloud compute disks create my-disk --size=10GB \
    --zone=europe-west2-a
```

Explore 

```
vi life_cycle.yaml
```

Notice `lifecycle.postStart` and `lifecycle.preStop`

Apply manifest

```
kubectl apply -f life_cycle.yaml
```

Check logs

```
kubectl exec -i life-cycle -- cat /data/log.txt
```

Kill life-cycle and apply again

```
kubectl delete pod/life-cycle
```

### (Stop Sharing)

## 2.3 Implementing Self-Healing Mechanisms

Pods benefit from a number of mechanisms, not only to control their life cycle, but also to monitor for erratic behavior.

## Implementing Self-Healing Mechanisms

The implementation of self-healing mechanisms is performed using two mechanisms: liveness and readiness probes. 

We use a liveness probe to interrogate whether the container is alive and in good health.

The readiness probe, instead, is used to verify whether a container is ready to service requests. Instrumenting the readiness probe is fundamental in applications that have a long startup time, or that undergo maintenance blackouts.

Both liveness and readiness probes can be instrumented using different strategies. 

The first is running a command which returns a successful exit code if the validation succeeds. For example, we could use this mechanism to check the presence of an error code on a log file.

The second strategy is checking whether a designated HTTP service returns a 200 status code. A status code in this range is meant to represent a normal, non-error response.

The last strategy is connecting to a TCP port in which the opening of that port would be counted as a success.

We can further configure how long each probe waits before it starts running the defined verification strategies, how long it waits before trying again, and how many validation failures must be reached to determine that the container has become unready, or dead, depending on the probe type.

Let's now see the configuration of self-healing properties in action...

### (Start Share) Google Cloud Shell

### Implementing Readiness and Liveness Probes

Explore 

```
vi nginx-self-healing.yaml
```

Apply manifest

```
kubectl apply -f nginx-self-healing.yaml
```

Notice the readiness probe's failure and fix the issue

```
kubectl exec pod/nginx-self-healing -- touch /ready.txt
```

Delete index.html

```
kubectl exec nginx-self-healing -- rm /usr/share/nginx/html/index.html
```

Notice Pod being restarted

_Optional_

Use `kubectl describe pod/nginx` for diagnosis

### (Stop Sharing)

## 2.4 Segregating Workloads Using Namespaces

Namespaces are a universal concept in Kubernetes and not the exclusivity of Pods, but it is convenient to learn about them in the context of Pods, because, ultimately, all workloads in Kubernetes are housed in Pods and all Pods live in a namespace.

## Segregating Workloads Using Namespaces

Namespaces is the mechanism that Kubernetes uses to segregate resources by a user-defined criteria. For example, namespaces can isolate development life cycle environments such as development, testing, staging, and production. 

They can also help group related resources, without necessarily intending to establish a Chinese wall; for example, a namespace may be used to separate front-end services from backend services.

Let's switch to the command line again....

### (Start Share) Google Cloud Shell


Pane 2: 

```
watch "kubectl get namespace --sort-by='.metadata.creationTimestamp' --no-headers | tac"
```

Unless we specify otherwise, whatever we launch runs in the default namespace.

```
kubectl run nginx --image=nginx 
```

(Do not run)

This is the same as...

```
kubectl run nginx --image=nginx -n default
```

Go to Pane 1 and change the watcher....

```
watch kubectl get pod -n default
```

Now change it kube-system

```
watch kubectl get pod -n kube-system
```

Now change it to all

```
watch kubectl get pod -A
```


Let's now create a full custom namespaces:

```
kubectl create namespace ns1
```

```
kubectl create namespace ns2
```

```
kubectl create namespace ns3
```


Go back to Pane 1: watch for NGINX pods

```
watch -n 1 "kubectl get pods -A | grep nginx"
```

And now let's place a Pod in each namespace

```
kubectl run nginx --image=nginx --restart=Never --namespace=ns1
```

```
kubectl run nginx --image=nginx --restart=Never --namespace=ns2
```

```
kubectl run nginx --image=nginx --restart=Never --namespace=ns3
```

Note there are no clashes

### (Stop Sharing)

## 2.5 Selecting Objects Using Labels

Labels are simply user-defined (or Kubernetes-generated) key/value pairs that are associated with a Pod (as well as any other Kubernetes object).

## Selecting Objects Using Labels

Labels are useful for describing small snippets of metadata, for example, 

- a family of related objects (e.g., replicas of the same Pod) 
- Version numbers 
- Environments (e.g., dev, staging, production) 
- Deployment type (e.g., canary release or A/B testing) 

Labels are a fundamental concept in Kubernetes because it is the mechanism that facilitates orchestration. When multiple Pods (or other object types) are “orchestrated,” the way in which a controller object (such as Deployment) manages a swarm of Pods is by selecting their label.

Let's get back to the command line again...

### (Start Sharing) Google Cloud Shell

First, let us display what labels apply to the Pods that are already running...

Pane 1:

```
watch -n 1 "kubectl get pods --show-labels"
```

Let us now launch new Pods using a custom label to indicate the author and the environment:

```
kubectl run nginx1 --image=nginx -l "env=prod,author=Bert"
```

```
kubectl run nginx2 --image=nginx -l "env=dev,author=Ernie"
```

```
kubectl run nginx3 --image=nginx -l "env=dev,author=Mohit"
```

We can now display the defined labels as proper columns....

Pane 1

```
watch -n 1 "kubectl get pods -L env,author"
```

Labels are not used just for display purposes, but to select Kubernetes resources using so-called selector and set-based expressions.

Example 1: Pods whose author ISN'T Ernie

```
kubectl get pods -l author!=Ernie
```

Example 2: Authors who are either Ernie or Mohit

```
kubectl get pods -l "author in (Ernie,Mohit)"
```

### Stop (Sharing)













