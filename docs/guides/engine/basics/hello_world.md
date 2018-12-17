---
title: Serial Execution | Guides
description: Serial Execution
menu:
  product_kubeci_0.1.0:
    identifier: guides-serial
    name: Serial Execution
    parent: guides-basics
    weight: 1
product_name: kubeci
menu_name: product_kubeci_0.1.0
section_menu_id: guides
---

> New to KubeCI engine? Please start [here](/docs/concepts/README.md).

# Hello World

This tutorial will show you how to use KubeCI engine to configure a Workflow consisting of a simple hello-world step and trigger it by creating a ConfigMap.

Before we start, you need to have a Kubernetes cluster, and the kubectl command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [Minikube](https://github.com/kubernetes/minikube). Now, install KubeCI engine in your cluster following the steps [here](/docs/setup/engine/install.md).

To keep things isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$ kubectl create ns demo
namespace/demo created
```

## Configure RBAC

You need to specify a service-account in `spec.serviceAccount` to ensure RBAC for the workflow. This service-account along with operator's service-account must have `list` and `watch` permissions for the resources specified in `spec.triggers`.

First, create a service-account for the workflow. Then, create a cluster-role with ConfigMap `list` and `watch` permissions. Now, bind it with service-accounts of both workflow and operator.

```console
$ kubectl apply -f ./docs/examples/engine/hello-world/rbac.yaml
serviceaccount/wf-sa created
clusterrole.rbac.authorization.k8s.io/wf-role created
rolebinding.rbac.authorization.k8s.io/wf-role-binding created
clusterrolebinding.rbac.authorization.k8s.io/operator-role-binding created
```

## Create Workflow

```console
$ kubectl apply -f ./docs/examples/engine/hello-world/workflow.yaml
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
  steps:
  - name: step-hello
    image: alpine
    commands:
    - echo
    args:
    - hello world
```

## Trigger Workflow

Create a ConfigMap with name `sample-config` to trigger the workflow.

```console
$ kubectl apply -f ./docs/examples/engine/hello-world/configmap.yaml
configmap/sample-config created
```

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: sample-config
  namespace: demo
data:
  example.property.1: hello
  example.property.2: world
```

Whenever a workflow is triggered, a workplan is created and respective pods are scheduled.

```console
$ kubectl get workplan -l workflow=sample-workflow -n demo
NAME                    CREATED AT
sample-workflow-fv7fq   24s
```

```console
$ kubectl get pods -l workplan=sample-workflow-fv7fq -n demo
NAME                      READY   STATUS      RESTARTS   AGE
sample-workflow-fv7fq-0   0/1     Completed   0          58s
```

## Get Status

To know the current phase of execution, you need see the workplan status.

```yaml
$ kubectl get workplan sample-workflow-fv7fq -o yaml -n demo
apiVersion: engine.kube.ci/v1alpha1
kind: Workplan
metadata:
  creationTimestamp: 2018-12-17T05:55:44Z
  generateName: sample-workflow-
  generation: 1
  labels:
    workflow: sample-workflow
  name: sample-workflow-fv7fq
  namespace: demo
  ownerReferences:
  - apiVersion: engine.kube.ci/v1alpha1
    blockOwnerDeletion: true
    kind: Workflow
    name: sample-workflow
    uid: 40db71df-01c0-11e9-9b94-080027982ddc
  resourceVersion: "4351"
  selfLink: /apis/engine.kube.ci/v1alpha1/namespaces/demo/workplans/sample-workflow-fv7fq
  uid: 64582405-01c0-11e9-9b94-080027982ddc
spec:
  resources: {}
  serviceAccount: wf-sa
  tasks:
  - ParallelSteps:
    - args:
      - -c
      - echo deleting files/folders; ls /kubeci; rm -rf /kubeci/home/*; rm -rf /kubeci/workspace/*
      commands:
      - sh
      image: alpine
      name: cleanup-step
    SerialSteps:
    - args:
      - hello world
      commands:
      - echo
      image: alpine
      name: step-hello
  triggeredFor:
    objectReference:
      apiVersion: v1
      kind: ConfigMap
      name: sample-config
      namespace: demo
    resourceGeneration: 0$8329273088337470134
  workflow: sample-workflow
status:
  nodeName: minikube
  phase: Succeeded
  reason: All tasks completed successfully
  stepTree:
  - - containerState:
        terminated:
          containerID: docker://506fd5628dde957e13408cfab11743ce18cc5ab5cb67eac1f3227802f89eb321
          exitCode: 0
          finishedAt: 2018-12-17T05:55:51Z
          reason: Completed
          startedAt: 2018-12-17T05:55:51Z
      name: step-hello
      podName: sample-workflow-fv7fq-0
      status: Terminated
  - - containerState:
        terminated:
          containerID: docker://2a00a120b970ecf4adc0f4192a73a49567393b0c3563ddf8330327aaf5828259
          exitCode: 0
          finishedAt: 2018-12-17T05:55:58Z
          reason: Completed
          startedAt: 2018-12-17T05:55:58Z
      name: cleanup-step
      podName: sample-workflow-fv7fq-0
      status: Terminated
  taskIndex: -1
```

## Get Logs

To get/stream logs of a particular step of a workplan, you need to call the `Get` API of `WorkplanLog` custom resource.

```console
$ kubectl get --raw '/apis/extensions.kube.ci/v1alpha1/namespaces/demo/workplanlogs/sample-workflow-fv7fq?step=step-hello'
hello world
```

## Cleanup

```console
$ kubectl delete ns demo
namespace "demo" deleted
```
