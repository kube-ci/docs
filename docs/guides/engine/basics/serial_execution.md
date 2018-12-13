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

# Serial Execution

This tutorial will show you how to use KubeCI engine to configure a Workflow consisting of a set of serial tasks and trigger it by creating a ConfigMap.

Before we start, you need to have a Kubernetes cluster, and the kubectl command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [Minikube](https://github.com/kubernetes/minikube). Now, install KubeCI engine in your cluster following the steps [here](/docs/setup/install.md).

To keep things isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$ kubectl create ns demo
namespace/demo created
```

## Configure RBAC

You need to specify a service-account in `spec.serviceAccount` to ensure RBAC for the workflow. This service-account along with operator's service-account must have `list` and `watch` permissions for the resources specified in `spec.triggers`.

First, create a service-account for the workflow. Then, create a cluster-role with ConfigMap `list` and `watch` permissions. Now, bind it with service-accounts of both workflow and operator.

```console
$ kubectl apply -f ./docs/examples/engine/serial-execution/rbac.yaml
serviceaccount/wf-sa created
clusterrole.rbac.authorization.k8s.io/wf-role created
rolebinding.rbac.authorization.k8s.io/wf-role-binding created
clusterrolebinding.rbac.authorization.k8s.io/operator-role-binding created
```

## Create Workflow

```console
$ kubectl apply -f ./docs/examples/engine/serial-execution/workflow.yaml
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
  - name: step-echo
    image: alpine
    commands:
    - echo
    args:
    - hello world
  - name: step-wait
    image: alpine
    commands:
    - sleep
    args:
    - 10s
```

Here, `step-echo` step prints `hello world` and `step-wait` step waits for 30s. This two steps runs sequentially in the given order.

## Trigger Workflow

Create a ConfigMap with name `sample-config` to trigger the workflow.

```console
$ kubectl apply -f ./docs/examples/engine/serial-execution/configmap.yaml
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
sample-workflow-sg9h4   24s
```

```console
$ kubectl get pods -l workplan=sample-workflow-sg9h4 -n demo
NAME                      READY   STATUS      RESTARTS   AGE
sample-workflow-sg9h4-0   0/1     Completed   0          58s
```

## Get Status

To know the current phase of execution, you need see the workplan status.

```yaml
$ kubectl get workplan sample-workflow-sg9h4 -o yaml -n demo
apiVersion: engine.kube.ci/v1alpha1
kind: Workplan
metadata:
  creationTimestamp: 2018-11-08T06:14:12Z
  generateName: sample-workflow-
  generation: 1
  labels:
    workflow: sample-workflow
  name: sample-workflow-sg9h4
  namespace: demo
  ownerReferences:
  - apiVersion: engine.kube.ci/v1alpha1
    blockOwnerDeletion: true
    kind: Workflow
    name: sample-workflow
    uid: 6948b0f8-e31d-11e8-a7e0-080027868e9e
  resourceVersion: "7738"
  selfLink: /apis/engine.kube.ci/v1alpha1/namespaces/demo/workplans/sample-workflow-sg9h4
  uid: 82e7195a-e31d-11e8-a7e0-080027868e9e
spec:
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
      name: step-echo
    - args:
      - 10s
      commands:
      - sleep
      image: alpine
      name: step-wait
  triggeredFor:
    objectReference:
      apiVersion: v1
      kind: ConfigMap
      name: sample-config
      namespace: demo
    resourceGeneration: 0$11733787535907091283
  workflow: sample-workflow
status:
  phase: Succeeded
  reason: All tasks completed successfully
  stepTree:
  - - containerState:
        terminated:
          containerID: docker://28dabd37dec53e4e06f16fcbcd0a8e4594882ae3f2cdfa85db44e8ed065c933b
          exitCode: 0
          finishedAt: 2018-11-08T06:14:28Z
          reason: Completed
          startedAt: 2018-11-08T06:14:28Z
      name: step-echo
      podName: sample-workflow-sg9h4-0
      status: Terminated
  - - containerState:
        terminated:
          containerID: docker://e9a28c2577403ef1ef68ad02dce4b573af9471f44da1dd9beca30460bb2ee654
          exitCode: 0
          finishedAt: 2018-11-08T06:14:48Z
          reason: Completed
          startedAt: 2018-11-08T06:14:38Z
      name: step-wait
      podName: sample-workflow-sg9h4-0
      status: Terminated
  - - containerState:
        terminated:
          containerID: docker://f994e704fc6a81d66590d6ce2ce0e08824d74968b5c9ae4f1f56cbfc715c0d38
          exitCode: 0
          finishedAt: 2018-11-08T06:14:59Z
          reason: Completed
          startedAt: 2018-11-08T06:14:59Z
      name: cleanup-step
      podName: sample-workflow-sg9h4-0
      status: Terminated
  taskIndex: -1
```

## Get Logs

To get/stream logs of a particular step of a workplan, you need to call the `Get` API of `WorkplanLog` custom resource.

```console
$ kubectl get --raw '/apis/extensions.kube.ci/v1alpha1/namespaces/demo/workplanlogs/sample-workflow-sg9h4?step=step-echo'
hello world
```

## Cleanup

```console
$ kubectl delete ns demo
namespace "demo" deleted
```
