---
title: Kaniko Build | Guides
description: Kaniko Build
menu:
  product_kubeci_0.1.0:
    identifier: guides-kaniko
    name: Kaniko Build
    parent: guides-build
    weight: 3
product_name: kubeci
menu_name: product_kubeci_0.1.0
section_menu_id: guides
---

> New to KubeCI engine? Please start [here](/docs/concepts/README.md).

# Build From Source

This tutorial will show you how to build and push container image from a Github source repository using [kaniko](https://github.com/GoogleContainerTools/kaniko).

Before we start, you need to have a Kubernetes cluster, and the kubectl command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [Minikube](https://github.com/kubernetes/minikube). Now, install KubeCI engine in your cluster following the steps [here](/docs/setup/install.md).

To keep things isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$ kubectl create ns demo
namespace/demo created
```

## Create Secret

In order to push image into container registry, you have to configure credential-initializer. First, create a secret with docker credentials and proper annotations. Also, you need to specify this secret in workflow's service-account in order to configure credential-initializer.

```console
$ kubectl apply -f ./docs/examples/engine/build-kaniko/secret.yaml 
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

## Configure RBAC

You need to specify a service-account in `spec.serviceAccount` to ensure RBAC for the workflow. This service-account along with operator's service-account must have `list` and `watch` permissions for the resources specified in `spec.triggers`.

Now, create a service-account for the workflow and specify previously created secret.

```yaml
# service-account for workflow
apiVersion: v1
kind: ServiceAccount
metadata:
  name: wf-sa
  namespace: demo
secrets:
- name: docker-credential
```

Then, create a cluster-role with ConfigMap `list` and `watch` permissions. Now, bind it with service-accounts of both workflow and operator.

```console
$ kubectl apply -f ./docs/examples/engine/build-kaniko/rbac.yaml  
serviceaccount/wf-sa created
clusterrole.rbac.authorization.k8s.io/wf-role created
rolebinding.rbac.authorization.k8s.io/wf-role-binding created
clusterrolebinding.rbac.authorization.k8s.io/operator-role-binding created
```

## Create Workflow

```console
$ kubectl apply -f ./docs/examples/engine/build-kaniko/workflow.yaml
workflow.engine.kube.ci/sample-workflow created
```

```yaml
apiVersion: engine.kube.ci/v1alpha1
kind: Workflow
metadata:
  name: sample-workflow
  namespace: demo
spec:
  triggers:
  - apiVersion: v1
    kind: ConfigMap
    resource: configmaps
    namespace: demo
    name: sample-config
    onCreateOrUpdate: true
    onDelete: false
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
```

Here, step `clone` clones source repository from Github into current working directory i.e. `/kubeci/workspace`. This repository contains following `Dockerfile` in it's root directory.

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

## Trigger Workflow

Now trigger the workflow by creating a `Trigger` custom-resource which contains a complete ConfigMap resource inside `.request` section.

```console
$ kubectl apply -f ./docs/examples/engine/build-kaniko/trigger.yaml 
trigger.extensions.kube.ci/sample-trigger created
```

Whenever a workflow is triggered, a workplan is created and respective pods are scheduled.

```console
$ kubectl get workplan -l workflow=sample-workflow -n demo
NAME                    CREATED AT
sample-workflow-779ht   5s
```

```console
$ kubectl get pods -l workplan=sample-workflow-779ht -n demo
NAME                      READY   STATUS     RESTARTS   AGE
sample-workflow-779ht-0   0/1     Init:2/3   0          47s
```

## Check Logs

You can check logs of the `build-and-push` step to verify if the image has been successfully built and pushed or not.

```console
$ kubectl get --raw '/apis/extensions.kube.ci/v1alpha1/namespaces/demo/workplanlogs/sample-workflow-779ht?step=build-and-push'
```

## Cleanup

```console
$ kubectl delete ns demo
namespace "demo" deleted
```