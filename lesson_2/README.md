# Lesson 2 - Pods

## 2.1 Launching Docker Containers Using Pods


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

_end of session_

---

## 2.2 Managing Pod Life Cycle


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

_end of section_

---

## 2.3 Implementing Self-Healing Mechanisms


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

## 2.4 Segregating Workloads Using Namespaces

TBC

## 2.5 Selecting Objects Using Labels

TBC

_end of segment_

