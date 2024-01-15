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
watch -n 1 kubectl get configmap
```

Pane 3.left: Leave empty for now

Pane 3.right: Leave empty for now

Pane 4

```
cd ~/safari_gke/config_jobs/
```

## 4.1 Implementing Batch Processes Using Jobs

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


## 4.2 Scheduling Recurring Tasks Using CronJobs

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

_end of segment_



## 4.3 Running Server-Wide Services Using DaemonSets

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

_end of section_

---











