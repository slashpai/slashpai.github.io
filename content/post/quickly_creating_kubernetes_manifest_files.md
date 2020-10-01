---
author: Pai
categories:
- kubernetes
- tips and tricks
date: "2020-10-01"
description: Quickly creating kubernetes manifest files using kubectl.
featured: true
series:
- Kubernetes
tags:
- Kubernetes
- featured
thumbnail: /img/loik-marras-Sq0L3SPWLHI-unsplash.jpg
title: Quickly creating kubernetes manifest files
---


When someone is just starting with kubernetes, writing manifest files can be a bit difficult. In this post I will try to give some tips in writing manifest files easily.

## Pods

Use the [generator](https://kubernetes.io/docs/reference/kubectl/conventions/#generators) to create pods. Verify if the given entries are syntactically correct by doing a dry run.

```go
kubectl run --generator=run-pod/v1 <pod name> --image <image to use> --dry-run
```

Save the pod definition to a yaml file for future changes as well as reference

```go
kubectl run --generator=run-pod/v1 <pod name> --image <image to use> --dry-run -o yaml > pod.yaml
```

**Example:**

```go
slashpai@pai  ~  (⎈ |kind-pai:default) kubectl run --generator=run-pod/v1 nginx-pod --image nginx --dry-run
pod/nginx-pod created (dry run)
slashpai@pai  ~  (⎈ |kind-pai:default) kubectl run --generator=run-pod/v1 nginx-pod --image nginx --dry-run -o yaml > pod.yaml
```

```bash
slashpai@pai  ~  (⎈ |kind-pai:default) cat pod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx-pod
  name: nginx-pod
spec:
  containers:
  - image: nginx
    name: nginx-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
slashpai@pai  ~  (⎈ |kind-pai:default) 
```

## Deployment

Generator for deployment is deprecated so following way helps. Verify if given entries are syntactically correct by doing a dry run

```go
kubectl create deployment <deployment name> --image <image to use> --dry-run
```

Save the deployment definition to a yaml file for future changes as well as reference

```go
kubectl create deployment <deployment name> --image <image to use> --dry-run -o yaml > deployment.yaml
```

**Example:**

```go
slashpai@pai  ~  (⎈ |kind-pai:default) kubectl create deployment nginx-deployment --image nginx --dry-run
deployment.apps/nginx-deployment created (dry run)
slashpai@pai  ~  (⎈ |kind-pai:default) kubectl create deployment nginx-deployment --image nginx --dry-run -o yaml > deployment.yaml
```

```bash
slashpai@pai  ~  (⎈ |kind-pai:default) cat deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx-deployment
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-deployment
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
slashpai@pai  ~  (⎈ |kind-pai:default)

```

## ReplicaSet

There is no direct way I could find to create this from kubectl. I found this trick though till I could find a better way. Use the same way to create deployment and modify `kind` to ReplicaSet in the yaml file and remove fields specific to deployment. I found this with trial and error, so this is the diff file.

```bash
slashpai@pai  ~  (⎈ |kind-pai:default) diff deployment.yaml replicaset.yaml
2c2
< kind: Deployment
---
> kind: ReplicaSet
13d12
<   strategy: {}
24c23
< status: {}
---
>
```

I removed the fields strategy as well as status from replicaset.yaml. While creating ReplicaSet file this way, give the deployment name suffix like replicaset instead of deployment. I haven't done that in this example since I wanted to show what fields I exactly need to change.

## ConfigMap

Even if there are more than one entry to be added in a config map, just add one and then add rest in manifest file directly so you can save time in adding one by one in command line. Verify first syntax wise correct using dry-run.

```go
kubectl create configmap <config map name> --from-literal key=value --dry-run
```

Save the deployment definition to a yaml file for future changes as well as reference

```go
kubectl create configmap <config map name> --from-literal key=value --dry-run -o yaml > configmap.yaml
```

**Example:**

```go
slashpai@pai  ~  (⎈ |kind-pai:default) kubectl create configmap mongo-config --from-literal user_id=mongo --dry-run
configmap/mongo created (dry run)
slashpai@pai  ~  (⎈ |kind-pai:default) kubectl create configmap mongo-config --from-literal user_id=mongo --dry-run -o yaml > configmap.yaml
```

```bash
slashpai@pai  ~  (⎈ |kind-pai:default) cat configmap.yaml
apiVersion: v1
data:
  user_id: mongo
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: mongo
```

You may want to move the data section below metadata since that's where we place this normally.

## Adding more entries in manifest file

Now you must have got an idea how to create manifest files for most used kubernetes objects. You can tweak like this for other objects not mentioned. The fields in manifest file created this way is not enough, if you have specific needs or need to add more fields.

Let's see how we can add more fields in a pod definition file. For example, you want to inject an environment variable into a container from a ConfigMap. Since we know container related info is inside spec section of the pod, we can use `kubectl explain pod.spec.containers` to get the exact field. See how this is done below.

```go
slashpai@pai  ~  (⎈ |kind-pai:default) kubectl explain pod.spec.containers|grep env
     container's environment. If a variable cannot be resolved, the reference in
     are expanded using the container's environment. If a variable cannot be
   env	<[]Object>
     List of environment variables to set in the container. Cannot be updated.
   envFrom	<[]Object>
     List of sources to populate environment variables in the container. The
```

I grepped for env, since I know field starts with env. If you don't know the field, you would need to scroll down the explain output and figure this out.

From this I got to know it's the  `envFrom` field that we need to use configmap and it's a list type (important to note the type). We can check the fields that comes under `envFrom` similar way like above. But if we are only interested to know the fields without it's description, just add the `recursive` flag

```go
slashpai@pai  ~  (⎈ |kind-pai:default) kubectl explain pod.spec.containers.envFrom --recursive
KIND:     Pod
VERSION:  v1

RESOURCE: envFrom <[]Object>

DESCRIPTION:
     List of sources to populate environment variables in the container. The
     keys defined within a source must be a C_IDENTIFIER. All invalid keys will
     be reported as an event when the container is starting. When a key exists
     in multiple sources, the value associated with the last source will take
     precedence. Values defined by an Env with a duplicate key will take
     precedence. Cannot be updated.

     EnvFromSource represents the source of a set of ConfigMaps

FIELDS:
   configMapRef	<Object>
      name	<string>
      optional	<boolean>
   prefix	<string>
   secretRef	<Object>
      name	<string>
      optional	<boolean>
slashpai@pai  ~  (⎈ |kind-pai:default) 
```

That drilled down the config to use as given below in the spec section of pod. Note the type for `envFrom`, which we had figured out to be of list type.

```yaml
spec:
  containers:
  - name: mongo
    image: mongo
    envFrom:
    - configMapRef:
        name: mongo-config
```

Surely you can go to kubernetes documentation and get this info. But I found this is more easier since I don't like leaving my terminal so often.

I hope this helps you create the manifest files with ease now. Thank you for reading this post.
