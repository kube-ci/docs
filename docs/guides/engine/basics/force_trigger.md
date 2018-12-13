---
title: Manual Trigger | Guides
description: Manual Trigger
menu:
  product_kubeci_0.1.0:
    identifier: guides-trigger
    name: Manual Trigger
    parent: guides-basics
    weight: 2
product_name: kubeci
menu_name: product_kubeci_0.1.0
section_menu_id: guides
---

> New to KubeCI engine? Please start [here](/docs/concepts/README.md).

# Manual Trigger

This tutorial will show you how to trigger a Workflow manually. Here, we will use the same workflow used in previous [serial-execution](serial_execution.md) example, but trigger it without creating any ConfigMap. Note that, for force trigger we have to set `allowManualTrigger` to true.

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
$ kubectl apply -f ./docs/examples/engine/force-trigger/rbac.yaml
serviceaccount/wf-sa created
clusterrole.rbac.authorization.k8s.io/wf-role created
rolebinding.rbac.authorization.k8s.io/wf-role-binding created
clusterrolebinding.rbac.authorization.k8s.io/operator-role-binding created
```

## Create Workflow

```console
$ kubectl apply -f ./docs/examples/engine/force-trigger/workflow.yaml
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

## Trigger Workflow

Now trigger the workflow by creating a `Trigger` custom-resource which contains a complete ConfigMap resource inside `.request` section.

```console
$ kubectl apply -f ./docs/examples/engine/force-trigger/trigger.yaml
trigger.extensions.kube.ci/sample-trigger created
```

```yaml
apiVersion: extensions.kube.ci/v1alpha1
kind: Trigger
metadata:
  name: sample-trigger
  namespace: demo
workflows:
- sample-workflow
request:
  apiVersion: v1
  kind: ConfigMap
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
sample-workflow-wnmw2   13s
```

```console
$ kubectl get pods -l workplan=sample-workflow-wnmw2 -n demo
NAME                      READY   STATUS      RESTARTS   AGE
sample-workflow-wnmw2-0   0/1     Completed   0          29s
```

## Cleanup

```console
$ kubectl delete ns demo
namespace "demo" deleted
```
