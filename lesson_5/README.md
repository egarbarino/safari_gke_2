# Lesson 5 - Configuration and StatefulSets


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


## 5.1 Externalizing Configuration Using ConfigMap

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

_end of section_

---

## 5.2 Protecting Credentials Using Secrets

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

_end of section_

---


## 5.3 Instrumental Stateful Applications Using StatefulSets - Part I


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
cd ~/safari_gke/further
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



## 5.4 Instrumental Stateful Applications Using StatefulSets - Part II


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
cd ~/safari_gke/further
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

_end of segment_

---

