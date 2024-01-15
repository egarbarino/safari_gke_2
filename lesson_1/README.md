# Lesson 1 - Getting Started with GKE

## Pre-Course Set Up

### Instructor Only

Main Browser

* Tab 1: https://console.cloud.google.com
* Tab 2: https://shell.cloud.google.com/?show=terminal
* Tab 3: https://crontab.guru/
* Tab 4: https://garba.org/posts/2018/k8s_pod_lc/
* Tab 5: https://garba.org/posts/2020/k8s-life-cycle/

On Tab 2 (Google Cloud Shell)

Create three TMUX Windows (CTRL+B C) and name them (CTRL+B ,) as follows:

0. BEFORE
1. AFTER
3. Main
4. Debugging

### Windows 0 (BEFORE) and 1 (AFTER)

On both Window 0 (BEFORE) and Window 1 (AFTER), set up panes as follows:

```
|------------------|
|        1         |
|------------------|
|        2         |
|------------------|
|        3         | 
|------------------|
|        4         |
--------------------
```

**Note:** Do not run any of the commands yet

Pane 1 - Monitor clusters

```
gcloud container clusters list
```

Pane 2 - Monitor compute

```
gcloud compute instances list | grep NAME
```

Pane 3 - Monitor disks

```
gcloud compute disks list | grep NAME
```

Pane 4 - Cluster Create Command

```
gcloud container clusters create my-cluster \
    --zone europe-west2-a \
    --project safari-gke
```


### Window 0 (BEFORE)

Run only the commands on Panes 1-3 but **not** on Pane 4

### Window 1 (AFTER)

Step 1

Run the `kubectl container clusters create ...` command on Pane 4 and wait until it completes.

Step 2

Run the commands that you pasted on Panes 2-3

Step 3

Run the Docker Hub fix on Window 2 (MAIN):

```
cd ~/safari_gke/getting_started/
```

```
./docker_hub_fix.sh
```

### Window 2 (Main)

```
|------------------|
|        1         |
|------------------|
|        2         |
|------------------|
|        3         | 
|------------------|
|        4         |
--------------------
```

Pane 1: Monitor Pod activity

```
watch -n 1 kubectl get pod
```

Pane 2: Monitor events

```
kubectl get -w events
```

Pane 3: Leave empty

Pane 4: Run commands here


### Window 3 (Debugging)

```
|------------------|
|        1         |
|------------------|
|        2         |
|------------------|
```

Pane 1 - Monitor Events

```
kubectl get -w events
```

Pane 2: - Watch Node Activity

```
watch -n 1 kubectl top node
```

**Get back to Window 2 (Main) before starting**

### Student Only

Fundamentals

1. Create an account at [https://cloud.google.com/](https://cloud.google.com/)
2. Set up billing (e.g., enter your credit card)
3. Create a project (we use 'safari-gke' in our examples)
4. Associate billing with the project

Set up TMUX 

Create file `.tmux.conf` and add the below line to have
the status bar visible at all times:

```
set -g status on
```

Command Prompt 

Set a shorter command prompt if useful:

```
export PS1='$ '
```



## 1.1 TBC



## 1.2 Setting up The Google Cloud Shell and GKE

```
I use TMUX to split my terminal session into multiple panes. It comes preinstalled with the Google Cloud Shell, and you can also get it for MacOS and Linux, including Windows Subsystem for Linux (WSL). You can install it by following these instructions: https://github.com/tmux/tmux/wiki/Installing
```

```
👍 I use TMUX already. You are preaching to the converted. 😲 I don't use TMUX but I definitely want to look into it 👎 I'm not a terminal hacker. I'd rather open multiple terminal windows.
```

### Access Google Cloud Shell

Browse [https://cloud.google.com/](https://cloud.google.com/) and click on the Google Cloud Shell Icon

Why Google Cloud Shell?
* The gcloud command is installed and pre-authenticated
* kubectl is preinstalled
* TMUX, python, etc.
* Closer to IaC

### Set up project

```
gcloud config set project safari-gke
```

You may add it to `~/.bashrc`

```
tail ~/.bashrc
```

### Enable Container Service

```
gcloud services enable container.googleapis.com
```

_end of section_

---


## 1.3 Creating and Destroying Kubernetes Clusters


### Cluster Creation and Deletion

Step 1 - Switch to Window 0 (and explore `gcloud container clusters ...`

Step 2 - Switch to Window 1 

Step 3 - Understand (but do not run!) cluster deletion

```
gcloud container clusters delete my-cluster \
	--async \
	--quiet \
	--zone europe-west2-a
```

### Introduce Nodes and Objects

Everything is an object

```
kubectl get node
```

```
kubectl get node/XXX -o yaml
```

```
kubectl explain node
```

```
kubectl delete node/XXX
```

_end of section_

---

