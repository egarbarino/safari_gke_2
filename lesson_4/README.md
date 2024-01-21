# Lesson 4 - Jobs and DaemonSets

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
watch -n 1 kubectl get jobs
```

Pane 3.left: Leave empty for now

Pane 3.right: Leave empty for now

Pane 4

```
cd ~/safari_gke_2/lesson_4/
```

## Lesson 4: Jobs and Daemon Set

Welcome to lesson 4, in which we will focus on two new controller types: Jobs and DaemonSets.

This lesson is organized into five modules:

* 4.1 Implementing Batch Processes Using Jobs
* 4.2 Scheduling Recurring Tasks Using CronJobs
* 4.3 Running Server-Wide Services Using DaemonSets

## Learning Objectives

By the end of this lesson, you'll understand:

* How to implement batch processes using the Job controller
* How to schedule recurring tasks using the CronJob controller

and finally,

* How to run server-wide Services using the DaemonSet controller

## 4.1 Implementing Batch Processes Using Jobs

Batch processing differs from persistent applications—such as web servers—in that programs are expected to complete and terminate once they achieve their goal. Typical examples of batch processing include database migration scripts, video encoding jobs, and Extract-Load-Transform (ETL) processes.

## Implementing Batch Processes Using Jobs

Similarly to the ReplicaSet controller, Kubernetes has a dedicated Job controller that manages Pods for the purpose of running batch-oriented workloads.

Instrumenting batch processes using Jobs is relatively straightforward as we will soon learn. 

We classify Jobs into three fundamental types:

* **Single Batch Process** in which running a Pod successfully just once is sufficient to finish the job. 

* **Completion Count–Based Batch Processes** in which a determined number of Pods must complete successfully to finish the job

and finally,...

* **Externally Coordinated Batch Processes** in which a pool of worker Pods works on a centrally coordinated job. The number of required completions is not known in advance.

Let's now explore each of these Job types in practice.

### (Start Share) Google Cloud Shell


Pane 4

```
kubectl delete all --all
```

Pane 2

```
watch -n 1 kubectl get job
```

Pane 3.right

```
kubectl get -w job
```

### Single Batch Process

```
kubectl create job two-times --image=alpine \
    -- sh -c \
    "for i in \$(seq 10);do echo \$((\$i*2));done"
```

Check out results and note label selector:

```
kubectl logs -l job-name=two-times
```

Note declarative version:

```
vi two-times.yaml
```

Note that job **won't go away** unless deleted!

```
kubectl delete job/two-times
```

### Completion Count-Based Batch Process

Pane 3.right:

```
while true; do kubectl describe job/even-seconds \
    | grep Statuses ; sleep 1 ; done
```

Explore:

```
vi even-seconds.yaml
```

Note:

* `spec.completions` (need to succeed 6 times)
* `spec.parallelism` (may run two pods in parallel to achieve the job)
* Script which randomly produces SUCCESS or FAILURE

Apply

```
kubectl apply -f even-seconds.yaml
```

Note activity across all panes

Explore logs

```
kubectl logs -l job-name=even-seconds 
```

_end of section_

### Externally Coordinated Batch Process


Pane 3.left:

```
kubectl delete job --all
```

Pane 3.right: stop and clear

Explore queue and **note** `--expose`:

```
vi startQueue.sh
```

```
./startQueue.sh
```

Test queue

```
kubectl run test --rm -ti --image=alpine \
    --restart=Never -- sh
```

Inside Pod: Try a few times and then exit

```
nc -w 1 queue 1080
```

Explore 

```
vi multiplier.yaml
```

Note:

* The attribute `spec.completion` is left empty
* The attribute `spec.parallelism` is set to 3


Restart Queue by running again:

```
./startQueue.sh
```

Apply Multiplier

```
kubectl apply -f multiplier.yaml
```

Explore results and note that `--tail=-1` is to override the label selector's limit

```
kubectl logs -l job-name=multiplier --tail=-1 | wc
```

```
kubectl logs -l job-name=multiplier --tail=-1 | sort -g
```

### (Stop Sharing)

## 4.2 Scheduling Recurring Tasks Using CronJobs

Some tasks may need to be run periodically at regular intervals; for example, we may want to compress and archive logs every week or create backups every month. 

Bare Pods could be used to instrument said tasks, but this approach would require the administrator to set up a scheduling system outside of the Kubernetes cluster itself. 

Luckily, Kubernetes implements a dedicated controller type for this purpose: the CronJob controller.

## Scheduling Recurring Tasks Using CronJobs

The need to schedule recurrent tasks brought the Kubernetes team to design a distinct controller modeled after the traditional cron utility found in Unix-like operating systems. 

Unsurprisingly, the controller is called CronJob, and it uses the same scheduling specification format as the crontab file.

In terms of implementation, the CronJob object is similar to the Deployment controller in the sense that it does not control Pods directly; it creates a regular Job object which is, in turn, responsible for managing Pods—The Deployment controller uses a ReplicaSet controller for this purpose. 

A key advantage of Kubernetes’ out-of-the-box CronJob controller is that---unlike its Unix cousin---it does not require a “pet” server that needs to be managed and nursed back to health when it fails. 

Let's now see the CronJob controller in action ...

## (Start Sharing) Google Cloud Shell

Pane 3.right: Delete jobs

```
kubectl delete job --all
```

Pane 3.left

```
watch -n 1 kubectl get cronjob
```

### Understand crontab file format

* Use an utility like https://crontab.guru/
* For example: `*/5 * * * *` is every five minutes

### Run Simple Cron Job

Note imperative version using `kubectl create cronjob --help`

```
vi simple.yaml
```

```
kubectl apply -f simple.yaml
```

Wait for it to run and then get logs:

```
kubectl logs simple-XXXXXXXXXX
```

### Suspending and Resuming Cron Jobs

Suspend:

```
kubectl patch cronjob/simple \
    --patch '{"spec" : { "suspend" : true }}'
```

Unsuspend:

```
kubectl patch cronjob/simple \
    --patch '{"spec" : { "suspend" : false }}'
```

### (Stop Sharing)

## 4.3 Running Server-Wide Services Using DaemonSets

While Jobs allow instrumenting processes that eventually complete, the DaemonSet controller allows running a persistent process.

## Running Server-Wide Services Using DaemonSets

Not only the DaemonSet controller keeps a persistance process at all times, but it ensures that every Node runs a single instance of said process. Naturally, like most things in Kubernetes, the actual process is implemented as a Pod.

In a Kubernetes Node that consists of three nodes like ours, a DaemonSet would therefore stand up three Pods, one in each worker node.

The DaemonSet controller is useful for uniform, horizontal services, that need to be deployed at the Node level, such as log collectors, caching agents, proxies, or any other kind of system-level capability. 

You may ask, why would, a distributed system such as Kubernetes, promote “tight coupling” between Pods and “boxes”? 

It is because of performance advantages. Pods deployed within the same Node can share the local network interface as well as the local file system; the benefit is significantly lower latency. 

I could be said, that a DaemonSet treats Nodes in the same way that Pods treat containers: while Pods ensure that two or more containers are collocated together, DaemonSets guarantee that daemons are always locally available in every Node---of course, so that consumer Pods running on those nodes can reach them. 

We identify two fundamental types of Daemons, depending on how we plan to communicate with then: tcp-based daemons and file-based daemons.

A TCP-based Daemon is a regular Pod managed by a DaemonSet controller whereby the services are accessed via TCP. The difference is that because every Daemon-controlled Pod is guaranteed to be deployed in every Node, there is no need for a service discovery mechanism since client Pods can simply establish a connection to the local Node they run on. 

A File-based Daemon, instead, uses the file system as a common medium to exchange information with Pods running on the same node.


Let's now look at examples of both TCP and file-based Daemons....

### (Start Sharing) Google Cloud Shell

Segment TMUX setup

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
watch -n 1 -w kubectl get pod -o wide
```

Pane 2

```
watch -n 1 kubectl get daemonset
```

Pane 3.left

```
watch -n 1 kubectl get deployment
```

Pane 3.right: Leave empty for now

Pane 4


-------

### TCP-Based Daemons

Explore

```
vi logDaemon.yaml
```

Apply

```
kubectl apply -f logDaemon.yaml
```

Note that there is exactly one Pod per Node

(!) Note use of Downward API for getting `HOST_IP`

```
vi logDaemonClient.yaml
```

Apply

```
kubectl apply -f logDaemonClient.yaml
```

Pane 3.right: Check logs per a given node

```
kubectl exec logd-XXXXXXXX -- cat /var/node_log
```

### File-Based Daemons

Delete everything

```
kubectl delete all --all
```

Explore 

```
vi logCompressor.yaml
```

Apply

```
kubectl apply -f logCompressor.yaml
```

Pane 3.right:

```
kubectl exec logcd-XXXXXXX -- find /var/log -name "*.gz"
```

Explore 

```
vi logCompressorClient.yaml
```

Apply

```
kubectl apply -f logCompressorClient.yaml
```

Check

```
kubectl exec logcd-XXXXXX -- tar -tf /var/log/all-logs-XXXXXXX.tar.gz
```

### (Stop Sharing)















