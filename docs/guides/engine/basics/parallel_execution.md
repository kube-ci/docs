---
title: Parallel Execution | Guides
description: Parallel Execution
menu:
  docs_0.1.0:
    identifier: guides-parallel
    name: Parallel Execution
    parent: guides-basics
    weight: 1
menu_name: docs_0.1.0
section_menu_id: guides
---

> New to KubeCI engine? Please start [here](/docs/concepts/README.md).

# Parallel Execution

This tutorial will show you how to use KubeCI engine to configure and trigger a Workflow consisting of a set of parallel tasks.

Before we start, you need to have a Kubernetes cluster, and the kubectl command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [Minikube](https://github.com/kubernetes/minikube). Now, install KubeCI engine in your cluster following the steps [here](/docs/setup/engine/install.md).

To keep things isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$ kubectl create ns demo
namespace/demo created
```

## Configure RBAC

You need to specify a service-account in `spec.serviceAccount` to ensure RBAC for the workflow. This service-account along with operator's service-account must have `list` and `watch` permissions for the resources specified in `spec.triggers`.

In this example, we are going to leave `spec.triggers` empty and trigger the workflow manually. So we don't need any of the permissions specified above. Also, we are going to use the default service account of `demo` namespace.

## Create Workflow

```console
$ kubectl apply -f ./docs/examples/engine/parallel-execution/workflow.yaml
workflow.engine.kube.ci/sample-workflow created
```

```yaml
apiVersion: engine.kube.ci/v1alpha1
kind: Workflow
metadata:
  name: sample-workflow
  namespace: demo
spec:
  serviceAccount: default
  executionOrder: Parallel
  allowManualTrigger: true
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

Dependency graph for the workflow:

<p align="center">
  <img alt="Parallel Execution" height="200px" src="/docs/examples/engine/parallel-execution/parallel-execution.svg">
</p>

Here, `step-echo` step prints `hello world` and `step-wait` step waits for 30s. This two steps runs in parallel.

## Trigger Workflow

You can use KubeCI CLI to trigger workflows. In order to use KubeCI CLI as `kubectl` plugin follow the steps [here](/docs/setup/cli/install.md).

```console
$ kubectl ci trigger sample-workflow -n demo
trigger.extensions.kube.ci/sample-workflow-trigger created
```

Whenever a workflow is triggered, a workplan is created and respective pods are scheduled.

```console
$ kubectl get workplan -l workflow=sample-workflow -n demo
NAME                    CREATED AT
sample-workflow-8c7wd   24s
```

```console
$ kubectl get pods -l workplan=sample-workflow-8c7wd -n demo
NAME                      READY   STATUS      RESTARTS   AGE
sample-workflow-8c7wd-0   0/2     Completed   0          43s
sample-workflow-8c7wd-1   0/1     Completed   0          15s
```

## Get Status

To know the current phase of execution, you need see the workplan status.

```yaml
$ kubectl get workplan sample-workflow-8c7wd -o yaml -n demo
apiVersion: engine.kube.ci/v1alpha1
kind: Workplan
metadata:
  creationTimestamp: 2018-12-20T12:07:14Z
  generateName: sample-workflow-
  generation: 1
  labels:
    workflow: sample-workflow
  name: sample-workflow-8c7wd
  namespace: demo
  ownerReferences:
  - apiVersion: engine.kube.ci/v1alpha1
    blockOwnerDeletion: true
    kind: Workflow
    name: sample-workflow
    uid: ae853148-044f-11e9-b255-08002763753c
  resourceVersion: "38794"
  selfLink: /apis/engine.kube.ci/v1alpha1/namespaces/demo/workplans/sample-workflow-8c7wd
  uid: c974f09f-044f-11e9-b255-08002763753c
spec:
  resources: {}
  serviceAccount: default
  tasks:
  - ParallelSteps:
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
    SerialSteps: []
  - ParallelSteps:
    - args:
      - -c
      - echo deleting files/folders; ls /kubeci; rm -rf /kubeci/home/*; rm -rf /kubeci/workspace/*
      commands:
      - sh
      image: alpine
      name: cleanup-step
    SerialSteps: []
  triggeredFor:
    objectReference: {}
  workflow: sample-workflow
status:
  nodeName: minikube
  phase: Succeeded
  reason: All tasks completed successfully
  stepTree:
  - - containerState:
        terminated:
          containerID: docker://602a87be60e4238120d387862b7cee75d798953878a6f85fe04b2ac4422d9ca9
          exitCode: 0
          finishedAt: 2018-12-20T12:07:23Z
          reason: Completed
          startedAt: 2018-12-20T12:07:23Z
      name: step-echo
      podName: sample-workflow-8c7wd-0
      status: Terminated
    - containerState:
        terminated:
          containerID: docker://e543fc03bc8a31dbd4a304608526b286addb4bc2d4214159686d7c48c58e9893
          exitCode: 0
          finishedAt: 2018-12-20T12:07:41Z
          reason: Completed
          startedAt: 2018-12-20T12:07:31Z
      name: step-wait
      podName: sample-workflow-8c7wd-0
      status: Terminated
  - - containerState:
        terminated:
          containerID: docker://e1a91d30f3cd09b0f12a7b268b9af09097ff361511880167b88bf64cf95df285
          exitCode: 0
          finishedAt: 2018-12-20T12:07:48Z
          reason: Completed
          startedAt: 2018-12-20T12:07:48Z
      name: cleanup-step
      podName: sample-workflow-8c7wd-1
      status: Terminated
  taskIndex: -1
```

## Get Logs

You can use KubeCI CLI to get logs of any step. In order to use KubeCI CLI as `kubectl` plugin follow the steps [here](/docs/setup/cli/install.md).

To get/stream logs of a particular step of a workplan, you need to call the `Get` API of `WorkplanLog` custom resource.

```console
$ kubectl ci logs sample-workflow-8c7wd --step step-echo -n demo
hello world
```

## Cleanup

```console
$ kubectl delete ns demo
namespace "demo" deleted
```
