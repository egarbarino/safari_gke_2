# Lesson 3 - High Availability and High Scalability

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
watch -n 1 kubectl get pod 
```

Pane 2

```
watch -n 1 kubectl get replicaset
```

Pane 3.left

```
watch -n 1 kubectl get deployment
```

Pane 3.right: Leave empty for now

Panel 4

```
cd ~/safari_gke_2/lesson_3/
```

## High Availability and Service Discovery

Welcome to lesson 3, High Availability and Service Discovery.

This lesson is organized into five modules.

* 3.1 Defining and Launching Deployments
* 3.2 Performing Rolling and Blue/Green Deployments
* 3.3 Instrumenting Static Scaling and Autoscaling
* 3.4 Configuring Pod-to-Pod Service Access
* 3.5 Publishing Services on the Public Internet
* 3.6 Performing Zero Downtime Deployments

## Learning Objectives

By the end of this lesson, you’ll understand:

* How to define and launch Deployments using the Deployment and ReplicaSet controllers
* How to perform rolling and blue/green deployments using the Deployment manifest's configuration settings
* How to instrument static scaling and autoscaling using the imperative and declarative scale properties
* How to configure Pod-to-Pod Service access using the Service controller
* How to publish Services on the public Internet, also using the Service controller

And finally,

* How to perform zero downtime deployments using the Deployment's manifest's availability properties.

## 3.1 Defining and Launching Deployments

We start this lesson by understanding how to define and launch a deployment.

A Deployment is a uniformly managed set of Pod instances, all based on the same Docker image or images in the case of a multi-container Pod.

## Defining and Launching Deployments

In the last lesson we saw how to launch Pods on their own. A Pod acts like a little virtual machine that runs in a single Kubernetes worker node. 

But...what if a worker node crashes? Well, our single Pod goes down with it as well.

The mechanism that we use to launch multiple Pods so that they are distributed across various worker nodes is called a Deployment, which is managed by a resource type called a Deployment controller.

The Deployment controller uses multiple replicas to achieve high scalability, by providing more compute capacity than is otherwise possible with a single monolithic Pod, and in-cluster high availability, by diverting traffic away from unhealthy Pods (with the aid of the Service controller, as we will see soon). 

In may seem that in Kubernetes, Deployment stands for “Pod cluster,” but “Deployment” is actually NOT a misnomer; the Deployment controller’s true power lies in its actual release capabilities—the deployment of new Pod versions with near-zero downtime to its consumers.

In this picture we see that the Deployment controller doesn't talk directly to Pods; instead, it manages another resource type called ReplicaSet. The ReplicaSet controller is the one that manages the Pods, although we normally interact only with the Deployment controller. We'll understand this relationship with practical examples later on.

Let's now learn how easy is to create a new deployment, in which we set the number of Pod replicas that we want, and how we can switch a Docker image in real time.

Let's go to the command line...

### (Start Share) Google Cloud Shell

Create deployment

```
kubectl create deployment nginx --image=nginx
```

Delete deployment

```
kubectl delete deployment/nginx
```

Specify number of replicas 

```
kubectl create deployment nginx --image=nginx --replicas=3
```

### Updating Docker Image

Update the pod's image

```
kubectl set image deploy/nginx nginx=nginx:1.9.1
```

### (Stop Share)


## 3.2 Performing Rolling and Blue/Green Deployments

Now, we will explore how Rolling and Blue/Green Deployments work.

## Performing Rolling and Blue/Green Deployments

There are three fundamental release approaches in Kubernetes, a release being the introduction of  a new Pod's Docker image. For example, when upgrading from version 1 to version 2.

The first strategy is called a recreate deployment.

A Recreate Deployment is one that literally terminates all existing Pods and creates the new set as per the specified target state. In this sense, a Recreate Deployment is downtime-causing since once the number of Pods reaches zero, there will be some delay before the new Pods are created. Recreate Deployments are useful for non-production scenarios in which we want to see the expected changes as soon as possible without waiting for the rolling update ritual to complete.

The second, and most common strategy is called a rolling update. A rolling update is typically one in which one Pod is updated at a time (and the load balancer is managed accordingly behind the scenes) so that the consumer does not experience downtime and resources (e.g., number of Nodes) are rationalized. 

Finally, a blue/green deployment is one in which we deploy the entire new version of the Pod cluster in advance, and when ready, we switch the traffic away from the baseline Pod cluster in one go. The entire process is orchestrated transparently with the help of the Service controller, which we'll learn about also in this lesson.

Let's now see all of these strategies in action.

### (Start Share)

### Recreate Deployments

Delete everything 

```
kubectl delete deploy --all
```

Be sure you have monitoring panes for deployments, pods, and replicas.

Note the difference in terms of Nginx versions in `nginx-v1-initial-v1.yaml` and `nginx-v2-recreate.yaml`.

Launch first version:

```
vi nginx-v1-initial.yaml
```

```
kubectl apply -f nginx-v1-initial.yaml
```


Apply `v2`, which produces an upgrade to nginx 1.9.1:

```
vi nginx-v2-recreate.yaml
```

```
kubectl apply -f nginx-v2-recreate.yaml
```

Observe all Pods being killed before producing the update

### One at a time Rolling Updates 

* Both progressive and blue/green deployments are implemented using the same approach
* Understand `maxSurge` vs `maxUnavailable` (book, slides)

```
vi nginx-v3-rolling-one-at-a-time.yaml
```

```
kubectl apply -f nginx-v3-rolling-one-at-a-time.yaml
```

### Blue/Green Update

```
vi nginx-v4-blue-green.yaml
```

```
sleep 5 ; kubectl apply -f nginx-v4-blue-green.yaml
```


### (Stop Share)

Further details at: https://garba.org/posts/2020/k8s-life-cycle/

## 3.3 Instrumenting Static Scaling and Autoscaling

Static scaling involves simply stating the number of desired replicas, while autoscaling involves dynamically altering said numbers to accommodate fluctuating demand.

## Instrumenting Static Scaling and Autoscaling

For static scaling we can specify the desired number of replicas in the Deployment's manifest, or by using the kubectl scale command.

Autoscaling, however, involves introducing a new controller type, the Horizontal Pod Autoscaler, or HPA.

The Horizontal Pod Autoscaler is a regular Kubernetes API resource and controller which manages a Pod’s number of replicas in an unattended manner, based on observed resource utilization. 

It can be thought of as a bot which issues kubectl scale commands on behalf of the human administrator based on a Pod’s scaling criteria, typically average CPU load.

Let's see both static and dynamic scaling in action...

### (Start Share) Google Cloud Shell

Delete everything and launch a new deployment

```
kubectl delete deploy --all
```

### Static Scaling

```
vi nginx-limited.yaml
```

```
kubectl apply -f nginx-limited.yaml 
```

Scale first to 3, then to 5, and finally down to 1:

```
kubectl scale deploy/nginx-declarative --replicas=3
```

### Setting Up Autoscaling

Note: We assume that the previous nginx deployment is still running

Pane 3.right

```
watch -n 1 kubectl get hpa
```

Now launch an autoscaler against the nginx deployment

```
kubectl autoscale deployment/nginx-declarative --min=1 --max=3 --cpu-percent=5
```

### Spiking CPU Internally

Generate infinite loop to spike the CPU and see auto-scaling in action. Ensure you pick the name of an actual Pod, rather than the one used below.

```
kubectl exec -it nginx-XXXXX-XXXX -- sh
```

Inside Pod type this:

```
while true; do true; done
```

### (Stop Sharing)

## 3.4 Configuring Pod-to-Pod Service Access

So far, we've seen how to deploy and scale a highly available fleet of Pods, but the picture isn't complete if we don't have a mechanism to connect to such Pods.

We will first start with the most basic connectivity use case which is that in which Pods can talk to one another, as it would be the case in a typical microservices-oriented architecture.

## Configuring Pod-to-Pod Service Access

Finding a Pod from within another Pod involves creating a Service object which will observe the associated Pod(s) so that they are added or removed from a virtual IP address—called a ClusterIP —-as they become ready and unready, respectively. 

The instrumentation of a container’s “readiness” may be implemented via custom probes as explained in the last lesson. 

The Service controller can target a variety of objects such as bare Pods and ReplicaSets, but we will concentrate on Deployments only. When the Service controller watches a Deployment, it can round robin---in other words, it acts as a load balancer---across the Deployment's Pods.

Let's see how all of this works together...

## (Start Sharing) Google Cloud Shell

Delete all previous deployments and Pods

```
kubectl delete all --all
```

Pane 3.right:

```
watch -n 1 kubectl get service
```

Create an Nginx deployment

```
kubectl create deployment nginx --image=nginx --replicas=3
```

Explore `update_hostname.sh` and run it:

```
vi update_hostname.sh
```

```
./update_hostname.sh
```

Create a service for the deployment

```
kubectl expose deploy/nginx --port=80
```

Explore `service.yaml` which is the declarative version and apply it:

```
kubectl apply -f service.yaml
```

Access nginx from a new Pod

```
kubectl run test --rm -ti --image=alpine --restart=Never -- sh
```

```
wget -q -O - http://nginx 
```

```
wget -q -O - http://nginx2 
```

Try wget multiple times to see different hostnames

### (Stop Sharing)

## 3.5 Publishing Services on the Public Internet

Both the cases of Pod-to-Pod connectivity, and Public Internet access use the same Service controller, ... except that we configure it differently.

## Publishing Services on the Public Internet

Accessing Pods from the public Internet involves creating a LoadBalancer service type. 

Even though the Service controller acts as a load balancer when used internally, when we set its type explicitly to LoadBalancer, what we are saying is that we want an actual Google Cloud Platform's external load balancer to be created and then associated with our service. Such external load balancer then in turn enables access from the public Internet.

Let's see this process in action...

### (Start Sharing) Google Cloud Shell

Pane 3.left:

```
watch -n 1 gcloud compute forwarding-rules list 
```

Pane 3.right

```
watch "kubectl get service/nginx3"    
```

Create a service of type LoadBalancer

```
kubectl expose deploy/nginx --name nginx3 --type=LoadBalancer --port=80
```

Now try public IP address:

```
while true; do curl -s http://IP_ADDRESS ; sleep 1 ; done
```

### (Stop Sharing)

## 3.6 Performing Zero Downtime Deployments

Zero-downtime deployments are achieved by the Kubernetes’ Deployment controller by registering new Pods with the Service controller and removing old ones, in a coordinated fashion, so that the end user is always being served by a minimum number of Pod replicas.

## Performing Zero Downtime Deployments

Implementing a zero downtime deployment does not require any new resource types or commands. All we have to do is to set a new attribute called `session affinity` to `ClientIP` so that the consuming client experiences a seamless transition to the new upgraded Pod(s) once ready.

Let's see this process in action...

### (Start Sharing) 

Pane 3.left: delete all previous deployments and services

```
kubectl delete all --all
```

Pane 3.right:

```
watch -n 1 kubectl get service
```

Create a new Deployment called `site`

```
kubectl create deployment site --image=nginx --replicas=3
```

Expose deployment and note `--session-affinity`

```
kubectl expose deploy/site --port=80 --type=LoadBalancer --session-affinity=ClientIP
```

Wait for public IP address

Pane 3.left: Watch server with IP 

```
cd ~/safari_gke_2/lesson_3/
```

```
cat watch.sh ; sleep 10 ; ./watch.sh site
```

Change base image

```
kubectl set image deploy/site *=httpd
```

Understand that applying a new manifest would cause the same effect

Use `kubectl describe pod/nginx` for diagnosis

## (Stop Sharing)





