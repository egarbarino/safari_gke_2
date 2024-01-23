# Lesson 5 - Configuration and StatefulSets

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
watch -n 1 kubectl get configmap
```

Pane 3.left: Leave empty for now

Pane 3.right: Leave empty for now

Pane 4

```
cd ~/safari_gke_2/lesson_5/
```
## Lesson 5: Configuration and StatefulSets

Welcome to lesson 5, in which we will first explore the process of externalizing configuration and then the art of implementing StatefulSets.

This lesson is organized into four sublessons.

* 5.1 Externalizing Configuration Using ConfigMap
* 5.2 Protecting Credentials Using Secrets
* 5.3 Instrumenting Stateful Applications Using StatefulSets - Part I
* 5.4 Instrumenting Stateful Applications Using StatefulSets - Part II

## Learning Objectives

By the end of this lesson, you’ll understand:

* How to externalize configuration using the ConfigMap controller
* How to protect credentials using the Secret controller

and finaly, ...

* How to instrument stateful applications using the StatefulSet controller

## 5.1 Externalizing Configuration Using ConfigMap

One key principle of cloud native applications is that of _externalized configuration._ In the Twelve-Factor App methodology, this architectural property is best described by factor III.

The ConfigMap controller is used for storing and propagating common configuration properties across various Pods.

## Externalizing Configuration Using ConfigMap (Architecture)

Containers within Pods typically run a regular Linux distribution—such as Alpine—which means that Kubernetes can assume the presence of a shell and environment variables, unlike a low-level virtualization platform that is agnostic to the operating system. 

As suggested by the Twelve-Factor App passage for factor III, nearly all programming languages can access environment variables, so this is certainly a universal and portable approach to pass configuration details into applications. 

Kubernetes is not limited to using environment variables; it can also make configuration available through a _virtual file system._ It also has a few other tricks under its sleeve, such as the ability to parse key/value pairs from files and obfuscate sensitive data.

Let's now see how to externalize configuration using this mechanism...

### (Start Share)

### Example of Hard Coded Configuration

```
vi podHardCodedEnv.yaml
```

```
kubectl apply -f podHardCodedEnv.yaml
```

```
kubectl logs pod/my-pod
```

### Define ConfigMap Separately


```
vi simpleconfigmap.yaml
```

```
kubectl apply -f simpleconfigmap.yaml
```

```
kubectl describe configmap/data-sources
```

Note there is an imperative form: `kubectl create configmap --help`

### Reference External Configuration from Pod

Delete Old Pod

```
kubectl delete pod/my-pod
```

Explore


```
vi podWithConfigMapReference.yaml
```

Apply

```
kubectl apply -f podWithConfigMapReference.yaml
```

Check logs

```
kubectl logs pod/my-pod
```

### Referencing Specific Fields

Explore:

```
vi podManifest.yaml
```

### Live Configuration Changes using Virtual File System

Pane 3.right: Delete everything

```
kubectl delete all --all
```

Explore

```
vi configMapLongText.yaml
```

Apply

```
kubectl apply -f configMapLongText.yaml
```

Explore `volume`, `volumeMounts`

```
vi podManifestVolume.yaml
```

Apply 

```
kubectl apply -f podManifestVolume.yaml
```

Pane 3.left:

```
kubectl logs -f pod/my-pod
```

Explore 

```
vi configMapLongText_changed.yaml
```

Apply

```
kubectl apply -f configMapLongText_changed.yaml
```

### (Stop Sharing)

## 5.2 Protecting Credentials Using Secrets

Secrets is ConfigMap’s sister capability.

The Secrets controller supports nearly all the same functionality as ConfigMap except that it is more appropriate for passwords and other types of sensitive data. 

## Protecting Credentials Using Secrets (Architecture)

The ConfigMap object is intended for clear-text, nonsensitive data that often originates in a centralized SCM. 

For passwords, and other sensitive data, the Secret object should be used instead. The Secret object is---in most cases---a “drop-in” replacement for the ConfigMap—when running in generic mode—for all imperative and declarative use cases except for the fact that clear-text data is meant to be encoded in base64 and is automatically decoded when made available through environment variables and volumes.

Let's now see how similar Secrets and ConfigMaps are....


### (Start Sharing)



Pane 3.right: Delete everything

```
kubectl delete all --all
```

Set up secrets monitoring 

Pane 2:

```
watch -n 1 kubectl get secret
```

Pane 3.left

```
watch -n 1 kubectl get secret/my-secrets -o yaml
```

### Imperative Form

```
kubectl create secret generic my-secrets \
    --from-literal=mysql_user=ernie \
    --from-literal=mysql_pass=HushHush
```

### Declarative Form

```
kubectl delete secret/my-secrets
```

```
echo -n ernie | base64
```

```
echo -n HushHush | base64
```

```
vi secrets/secretManifest.yaml
```

```
kubectl apply -f secrets/secretManifest.yaml
```

### Injecting Secrets

```
vi secrets/podManifestFromEnv.yaml
```

```
kubectl apply -f secrets/podManifestFromEnv.yaml
```

```
kubectl logs pod/my-pod
```

### Further Use Cases (Student Only)

* Select specific variables: `secrets/podManifesSelectedEnvs.yaml`
* Mount secrets as volume: `secrets/podManifestVolume.yaml`

### Docker Secrets (Student Only)

* Check out `../getting_started/docker_hub_fix.sh`
* Note `spec.imagePullSecrets` on `secrets-docker/podFromPrivate.yaml`

### (Stop Sharing)


## 5.3 Instrumenting Stateful Applications Using StatefulSets - Part I

Now that we've learned how to externalize configuration, we can look at the most complex controller type, the StatefulSet controller.

## Instrumenting Stateful Applications Using StatefulSets - Part I

Stateful workloads have different dynamics than simple stateless Twelve-Factor Apps. 

For example, scaling is not a trivial matter; scaling “down” may result in the loss of data, and scaling up may result in the inappropriate replication or resharding of the existing cluster. 

Some stateful services are not meant to scale at all, or not at least in an automatic fashion. 

Stateful services also vary greatly in how they achieve high scalability. Some use a supervisor-worker strategy (e.g., MySQL and MongoDB), whereas some others have a multi-master architecture instead, like Memcached and Cassandra. 

The StatefulSet controller in Kubernetes cannot make broad, sweeping assumptions about the nature of each data store; therefore, it focuses on low-level, primitive properties—such as stable network identity—that may, selectively, assist in their implementation depending on the discrete problem or required property at hand.

In this module, we will build a primitive key/value data store from scratch, which will serve to internalize the principles of implementing StatefulSets. 

In this first part, the key/value data store will run directly in memory, in the second part, we will make it persistent.

Let's begin.

### (Start Sharing)

Delete everything

```
kubectl delete all --all
```

Pane 3.right: 

```
watch -n 1 kubectl get statefulset
```

### Primitive Key/Value Store

Explore Server

```
vi server.py
```

Explore ConfigMap

```
cd wip
```

```
vi configmap.sh
```

Run ConfigMap

```
./configmap.sh
```

Explore StatefulSet-based Server

```
vi server.yaml
```

Run Server

```
kubectl apply -f server.yaml
```

Panel 3.left:

```
kubectl port-forward server-0 1080:80
```

Experiment saving and loading keys

```
curl http://localhost:1080/save/title/Sapiens
```

```
curl http://localhost:1080/load/title
```

```
curl http://localhost:1080/save/author/Yuval
```

```
curl http://localhost:1080/load/author
```

```
curl http://localhost:1080/allKeys
```

Connect to another server and see that no keys are stored

### Sequential Pod Creation and Termination

Now that we have implemented a simple database using a StatefulSet, let's introduce the first property of StatefulSets which is **Sequential Pod Creation and Termination**.

This property guarantees the orderly and numerically ascending creation of Pods, as well as the numerically descending and reverse deletion.

Let's now see this property in action...


(!) Destructive

Increment to 5 

```
kubectl scale statefulset/server --replicas=5
```

Decrement to 3

```
kubectl scale statefulset/server --replicas=3
```

### Stable Network Identity and Headless Service

The second StatefulSet's property is that of Stable Network Identity.

In a stateless application, the specific identity and location of each replica is ephemeral. As long as we can reach the load balancer, it doesn’t matter which specific replica serves our request. This is not necessarily the case for a stateful service, such as our primitive key/value store. 

The way in which many multi-master stores solve the problem of scale, without creating a central contention point, is by making each client (or equivalent proxy agent) aware of every single server host so that the client itself decides where to store and retrieve the data. 

It follows that, in the case of StatefulSets, and depending on the data store’s horizontal scaling strategy, it may be crucial for clients to know exactly the Pods to which they have saved data so that following scaling, restarts, and failures events, the LOAD request always matches the original Pod used for original SAVE request. 

Let's now quickly demonstrate this property.

Panel 3.left:

```
watch -n 1 -w kubectl get service
```

* No loadbalancer, just a DNS entry
* Note on service.yaml that ClusterIP is set to None

```
vi service.yaml
```

```
kubectl apply -f service.yaml
```

```
kubectl run --image=alpine --restart=Never --rm -i test -- nslookup -type=srv server | head -n 7
```

### Client for Headless Service

Pane 2: `kubectl logs -f client` (do not press ENTER yet)

(!) Concept of sharding using modulo arithmetic

```
python3
```

Inside Python 3's shell:

```
ord('a') % 3
```

Explore and run smart client

```
cd ~/safari_gke_2/lesson_5/
```

```
vi client.py
```

```
vi client.yaml
```

```
vi wip/configmap2.sh
```

```
cd wip
```

```
./configmap2.sh
```

```
kubectl apply -f ../client.yaml
```

Explore logs

```
kubectl logs client | head
```

Delete one random server Pod

```
kubectl delete pod/server-1
```

### (Stop Sharing)

## 5.4 Instrumenting Stateful Applications Using StatefulSets - Part II

In the previous module, we implemented a distributed key/value database using the StatefulSet controller saving the data on a memory-based file system.

In this second part, we will be switching to a fully persistent disk-based file system.

## Instrumenting Stateful Applications Using StatefulSets - Part II

As we can see in this diagram, we will now connect our StatefulSet with Google Cloud Storage. 

This works through a mechanism called Persistent Volume Claims which allows Kubernetes to create disks on demand, based on the identity of each Pod.

This means that if a Pod crashes or gets rescheduled, a 1:1 link is maintained between each Pod and its associated volume. No matter on which Node a Pod “wakes up,” Kubernetes will always attach its corresponding bound volume, for server-0, it will attach disk-0, ... for server-1, disk-1, and so on...

Let's now upgrade our key/value pair database to disk persistance and see this mechanism in action.

### (Start Sharing)

### Disk Persistence using Persistent Volume Claims

Delete everything

```
kubectl delete all --all
```

Explore and apply

Pane 3.left

```
watch -n 1 -w kubectl get pvc
```

Pane 3.right

```
watch -n 1 "gcloud compute disks list | grep NAME"
```

Note preStart and postStop hooks on server.yaml

```
cd ~/safari_gke_2/lesson_5/
```

Note return of 503 code on shutting_down

```
vi server.py
```

Note preStart/postStart hooks and Disk Volume

```
vi server-disk.yaml     
```

```
vi configmap.sh
```

```
./configmap.sh
```

```
kubectl apply -f server-disk.yaml
```

Pane 2: (do not press ENTER yet)

```
kubectl logs -f client
```

Pane 4

```
kubectl apply -f client.yaml
```

Now go to Pane 2 and press Enter 

### Chaos Engineering

```
kubectl delete pod/server-2
```

```
kubectl delete statefulset/server 
```

```
kubectl apply -f server-disk.yaml
```

### (Stop Sharing)

## Kubernetes on GKE (Summary)

You have reached the end of this course. Let's recap what we've learned.

In the first lesson we saw how to access the Google Cloud Shell and create a Kubernetes Cluster.

In the second lesson we learned how to deploy Docker containers from DockerHub by wrapping them into Pods. We saw how Pods give us greater control over containers by allowing us to control their life cycle and establish self-healing mechanisms.

In the third lesson we learned how to scale Pods and make them highly available, applying common release patterns such as blue/green and zero-downtime deployments.

In the fourth lesson, we explored the use of Pods in a context different than that of microservices: batch processing using Jobs, and persistent processes using DaemonSets.

Finally, in the fifth lesson, we learned how to externalize configuration, and setup a distributed key/value database using the StatefulSet controller.

While the Kubernetes landscape is immense, the topics covered in this course are the most essential ones, given that they map to equivalent processes in traditional data on-prem virtualization architectures.

## Summary

This entire course is open source and you can find all the examples and course narrative on GitHub. Also, as a subscriber of the O'Reilly platform, you have access to my book, in which all of the examined topics are covered further in depth. 

All the best in your future Kubernetes adventures and good bye!






