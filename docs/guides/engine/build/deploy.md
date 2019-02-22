---
title: Source to Deploy | Guides
description: Source to Deploy
menu:
  docs_0.1.0:
    identifier: guides-deploy
    name: Source to Deploy
    parent: guides-build
    weight: 4
product_name: kubeci
menu_name: docs_0.1.0
section_menu_id: guides
---

> New to KubeCI engine? Please start [here](/docs/concepts/README.md).

# Source to Deployment

This tutorial will show you how to clone a Github source repository, build/push container image from the source and finally deploy it. In this example we are going to use [kaniko](https://github.com/GoogleContainerTools/kaniko) for building container image. 

Before we start, you need to have a Kubernetes cluster, and the kubectl command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [Minikube](https://github.com/kubernetes/minikube). Now, install KubeCI engine in your cluster following the steps [here](/docs/setup/engine/install.md).

To keep things isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$ kubectl create ns demo
namespace/demo created
```

## Create Secret

In order to push image into container registry, you have to configure credential-initializer. First, create a secret with docker credentials and proper annotations. Also, you need to specify this secret in workflow's service-account in order to configure credential-initializer.

```console
$ kubectl apply -f ./docs/examples/engine/deploy/secret.yaml
secret/docker-credential created
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: docker-credential
  annotations:
    credential.kube.ci/docker-0: https://index.docker.io/v1/
type: kubernetes.io/basic-auth
stringData:
  username: ...
  password: ...
```

Now, create a service-account for the workflow and specify previously created secrets.

```console
$ kubectl apply -f ./docs/examples/engine/deploy/service-account.yaml
serviceaccount/wf-sa created
```

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: wf-sa
  namespace: demo
secrets:
- name: docker-credential
```

## Configure RBAC

You need to specify a service-account in `spec.serviceAccount` to ensure RBAC for the workflow. This service-account along with operator's service-account must have `list` and `watch` permissions for the resources specified in `spec.triggers`.

In this example, we are going to leave `spec.triggers` empty and trigger the workflow manually. So we don't need any of the permissions specified above.

## Create Workflow

```console
$ kubectl apply -f ./docs/examples/engine/deploy/workflow.yaml
workflow.engine.kube.ci/sample-workflow created
```

```yaml
apiVersion: engine.kube.ci/v1alpha1
kind: Workflow
metadata:
  name: sample-workflow
  namespace: demo
spec:
  serviceAccount: wf-sa
  executionOrder: Serial
  allowManualTrigger: true
  steps:
  - name: clone
    image: alpine/git
    commands:
    - sh
    args:
    - -c
    - git clone https://github.com/kube-ci/kubeci-gpig .
  - name: build-and-push
    image: gcr.io/kaniko-project/executor
    args:
    - --dockerfile=/kubeci/workspace/Dockerfile
    - --context=/kubeci/workspace
    - --destination=index.docker.io/kubeci/kubeci-gpig:kaniko
  - name: deploy
    image: appscode/kubectl:1.12
    commands:
    - sh
    args:
    - -c
    - kubectl apply -f deploy.yaml
```

Here, step `clone` clones the source repository from Github into current working directory i.e. `/kubeci/workspace`. This repository contains following `Dockerfile` in it's root directory.

```
# build stage
FROM golang:alpine AS build-env
ADD . /go/src/github.com/kube-ci/kubeci-gpig
WORKDIR /go/src/github.com/kube-ci/kubeci-gpig
RUN CGO_ENABLED=0 go build -o goapp && chmod +x goapp

# final stage
FROM alpine
COPY --from=build-env /go/src/github.com/kube-ci/kubeci-gpig/goapp /usr/bin/kubeci-gpig
ENTRYPOINT ["kubeci-gpig"]
```

Step `build-and-push` runs a `kaniko` image which builds container from the specified Dockerfile and pushes it into the specified registry.

Step `deploy` runs a `kubectl` image which deploys your application using following `deploy.yaml` file contained in the source repository.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubeci-gpig-deployment
  labels:
    app: kubeci-gpig
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kubeci-gpig
  template:
    metadata:
      labels:
        app: kubeci-gpig
    spec:
      containers:
      - name: kubeci-gpig
        image: kubeci/kubeci-gpig:kaniko
        ports:
        - containerPort: 9090
```

## Trigger Workflow

Now trigger the workflow by creating a `Trigger` custom-resource which contains a complete ConfigMap resource inside `.request` section.

```console
$ kubectl apply -f ./docs/examples/engine/deploy/trigger.yaml
trigger.extensions.kube.ci/sample-trigger created
```

Whenever a workflow is triggered, a workplan is created and respective pods are scheduled.

```console
$ kubectl get workplan -l workflow=sample-workflow -n demo
NAME                    CREATED AT
sample-workflow-zzqkx   5s
```

```console
$ kubectl get pods -l workplan=sample-workflow-zzqkx -n demo
NAME                      READY   STATUS     RESTARTS   AGE
sample-workflow-zzqkx-0   0/1     Init:2/4   0          47s
```

## Check Logs

You can use KubeCI CLI to get logs of any step. In order to use KubeCI CLI as `kubectl` plugin follow the steps [here](/docs/setup/cli/install.md).

You can check logs of any step to verify if all the operations has been successfully completed or not. For example, run following command to check logs of `build-and-push` step:

```console
$ kubectl ci logs sample-workflow-zzqkx --step build-and-push -n demo
```

## Check Deployment

```console
$ kubectl get deployment -l app=kubeci-gpig -n demo
NAME                     DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubeci-gpig-deployment   1         1         1            1           11s

$ kubectl get pod -l app=kubeci-gpig -n demo
NAME                                      READY   STATUS    RESTARTS   AGE
kubeci-gpig-deployment-74c75c9b79-w89h8   1/1     Running   0          43s

$ kubectl logs kubeci-gpig-deployment-74c75c9b79-w89h8 -n demo
Starting server
```

## Cleanup

```console
$ kubectl delete ns demo
namespace "demo" deleted
```
