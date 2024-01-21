# Lesson 1 - Getting Started with GKE

Instructor Only

Main Browser

* Tab 1: https://console.cloud.google.com
* Tab 2: https://shell.cloud.google.com/?show=terminal
* Tab 3: https://crontab.guru/
* Tab 4: https://garba.org/posts/2018/k8s_pod_lc/
* Tab 5: https://garba.org/posts/2020/k8s-life-cycle/

TMUX Setup (Tab 2)


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

Pane 4

```
git clone https://github.com/egarbarino/safari_gke_2
cd ~/safari_gke_2/lesson_1/
```



Student Only

Fundamentals

1. Create an account at [https://cloud.google.com/](https://cloud.google.com/)
2. Set up billing (e.g., enter your credit card)
3. Create a project (we use 'safari-gke' in our examples)
4. Associate billing with the project

Set up TMUX 

Create file `~/.tmux.conf` and add the below line to have
the status bar visible at all times:

```
set -g status on
```

Command Prompt 

Set a shorter command prompt if useful:

```
export PS1='$ '
```

## About Your Instructor

Hello, and welcome to this Kubernetes on GKE Video Course. 

My name is Ernesto Garbarino, or just 'Ernie' and I'll be your instructor. 

I work as an Enterprise Architect helping global enterprises migrate to the cloud and adopt microservices-based architectures.

Kubernetes, and in particular, the Google Cloud's managed offering, GKE, picked my interest from an early beginning, which led to me write a book on the subject. 

Now, I have summarized the book's key lessons into this video course.

## Course Learning Objectives

This course was designed for the absolute beginner in mind, who is looking to internalize the foundational capabilities that Kubernetes brings to the table.

This course is also self-contained and does not demand advanced knowledge in areas such as cloud, virtualization, or programming. 

Not even proficiency in Docker is required. Since Kubernetes is effectively a Docker host, you can experiment with Docker and public Docker images found in Docker Hub for the first time on the Google Cloud Platform itself without installing the Docker runtime on your local machine.

Now, in terms of concrete learning objectives, ...

you'll first learn how to deploy containers wrapped as Pods using the Deployment and Service controllers, so you can achieve high scalability and high availability.

Then, you'll learn how to externalize configuration using the ConfigMap and Secrets controllers. 

We'll also cover advanced topics such as DaemonSets and Stateful services.

...

This knowledge will enable you to set up your own Kubernetes clusters on GKE, so that you can launch and control containers, and manage their life cycle. 

You'll also have the opportunity to discover how a blue/green deployment is implemented, and how to deploy your applications with zero downtime.

## Lesson 1: Getting Started with GKE

Welcome to the first lesson, Getting Started with GKE. In this lesson you'll learn how set up a cloud-based environment that allows you to explore Kubernetes using your own account.

## (Blank)

By the end of this lesson, you'll understand what are the key GKE's concepts and controller types,

how to access the Google Cloud Shell so that you manage Kubernetes from any web browser, 

and how to create your first Kubernetes cluster. 

## 1.1 Introducing GKE (Splash)

Before we begin, let's understand a few things about GKE first...

## GKE In a Nutshell

Kubernetes was originally designed by Google, and the project is now maintained by the Cloud Native Computing Foundation.

GKE, which stands for Google Kubernetes Engine, is the first and oldest mainstream managed Kubernetes service. 

Kubernetes is today the most popular and widely adopted container orchestrator in the world.

Even if you were to adopt a different Kubernetes distribution, be it managed or on-prem, GKE still remains the most developer-friendly implementation.

The free credit that Google offers, together with a built-in shell and editor, make the learning experience accessible from any machine that can run a web browser without the need of a complex local development environment.

## Architecture Overview

A best way to understand how this course is organized is by looking at the architectural areas that we'll be covering.

In this first lesson, color-coded in gray, you'll learn how to access the Google Cloud Shell and stand up a Kubernetes Cluster.

In the second lesson, color-coded in green, you'll learn how to launch containers using Pods and control their life cycle.

In the third lesson, color-coded in blue, you'll learn how to scale containers---wrapped in Pods---and make them highly available.

In the fourth lesson, color-coded in purple, you'll learn how to run batch processes using the Job controller, and how to run steady processes using the DaemonSet controller.

Finally, the fifth lesson, color-coded in red, you'll learn how to externalize configuration using the ConfigMap and Secrets controllers, and you'll also deep dive into the art of implementing StatefulSets.

You don't need to memorize this picture. We'll be painting it from scratch, one step at a time, as we progress through this course.

## Course Resources

All of the code that I'll demonstrate, as well as my own narrative is open source and available at the URL that you see on the screen.

You can also read my book on the O'Reilly platform to explore some of the more advanced topics such as StatefulSets.

## 1.2 Setting up The Google Cloud Shell and GKE (Splash)

We will now get practical by setting up the Google Cloud Shell and enabling access to GKE.

## Setting Up The Google Cloud Shell and GKE 

As I said a few moments ago, this course is structured in such a way that we will work on exploring Kubernetes one step at a time. As such, the screen that you see now is largely empty.

In the beginning, our first goal is to gain access to the gcloud command. The gcloud command acts as so-called God command since it allows interacting with all Google Cloud services. Nautrally, the gcloud command uses the Google Cloud APIs behind the scenes. 

In this course, we won't be using the web console because web-based workflows are hard to automate, script, and repeat. While using the command line is not quite the same as defining infrastructure as code---using the likes of Terraform, it takes us a step closer in terms of understanding what resources are created as a result of our interactions.

As I work with the terminal, you'll notice that I divide my screen in multiple panes. I use a tool called TMUX for this purpose, which comes included with the Google Cloud Shell. You can get TMUX for your favorite Linux distribution as well as MacOS from github.com/tmux (T M U X)

Let's get started

## (Start Sharing - Chrome)

###  Access Google Cloud Shell

Go to [https://cloud.google.com/](https://cloud.google.com/)

I assume that you've already registered your Google Cloud account. To access the main Google Cloud's main console you click on Console or go to console.cloud.google.com.

Once in the console, we need to do things in the context of a project. Our sample project is called safari-gke.

If you don't have a default project selected, or you want to select a different one, you can click on the top left dropdown menu, and select a different project, or click on NEW PROJECT.

As we'll be using the command line, rather than the web console, we need to locate the Google Cloud Shell icon which has a command line interface icon.

### Once Google Cloud Shell is in Full Screen

Some of you may wonder, why not use your own terminal application? Of course, you'll eventually download the Google Cloud SDK and run commands from your laptop, but the advantage of the Google Cloud Shell is that it comes with all the tools that we need preinstalled, and it is also pre-authenticated to create and manage our cloud resources.

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

### (Stop Sharing)

## 1.3 Creating and Destroying Kubernetes Clusters (Splash)

All we have accomplished so far is have access to a command line interface, set the context to our project named safari-gke, and enable access to the Kubernetes service API. We are also pre-authenticated with the Google Cloud Platform so we have everything we need to create our first Kubernetes cluster.

## Creating and Destroying Kubernetes Clusters (Architecture Step)

Now that we can access the gcloud command, we will it use to create our first Kubernetes cluster, and then show you how to delete it as well.

The gcloud command also creates the configuration file used by the kubectl command, which is the native command used to manage Kubernetes. You can think of as the gcloud command handing over Kubernetes control to the kubectrl command once the cluster is created... just as shown on the diagram.

Let's now see this process in action...


### (Screen Share) Google Cloud Shell

### Cluster Creation and Deletion

Before we proceed, in addition to painting our Kubernetes Architecture diagram one figure at a time, there's also another peculiarity about this course I wanted to mention.

I'm obsessed about following an observability-first approach. By that I mean that I want you to observe the side effects of everything we so that you can understand what are the consequences of our actions. 

For this reason, before we create our first Kubernetes cluster, I want you to understand what resources will be created as a result.

Pane 1 - Clusters

```
gcloud container clusters list
```

Pane 2 - Compute

```
gcloud compute instances list | grep NAME
```

Pane 3 - Disks

```
gcloud compute disks list | grep NAME
```

Pane 4 - Cluster Create Command

```
gcloud container clusters create my-cluster \
    --zone europe-west2-a \
    --project safari-gke
```

(!) This will take a while so we will fast forward to the end

Now that the command has completed, there's one more thing to do:

```
./docker_hub_fix.sh
```

We can run again the previous commands to see what resources we ended up with.

Rerun commands in Panes 1-3 after creation

### Introduce Nodes and Objects

Now that the gcloud command has completed, it has created the configuration settings to allow the kubectl command to take control.

An interesting and useful aspect of Kubernetes is that most components are defined as objects (also called resources or resource types) that can be manipulated using similar actions.

The nodes that represent a Kubernetes cluster are also treated as regular resources. For example...


Google Cloud Perspective

```
gcloud compute instances list | grep NAME
```

Kubernetes Perspective

```
kubectl get node -o wide
```

We can get more details about each specific resource instance by referencing its identifier

```
kubectl get node/XXX -o yaml
```

We can also ask Kubernetes to explain what is the resource type

```
kubectl explain node
```

And we can also delete the resource type. Let's delete one VM.

```
kubectl delete node/XXX
```

### Deletion



```
gcloud container clusters delete my-cluster \
        --async \
        --quiet \
        --zone europe-west2-a
```

This command would delete our cluster asynchronously and without producing any output. We won't run it now, though, as we will use our cluster to run launch our first workloads in the next lesson.


### End




















