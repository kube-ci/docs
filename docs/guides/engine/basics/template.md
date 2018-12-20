---
title: Workflow Template | Guides
description: Workflow Template
menu:
  product_kubeci_0.1.0:
    identifier: guides-template
    name: Workflow Template
    parent: guides-basics
    weight: 4
product_name: kubeci
menu_name: product_kubeci_0.1.0
section_menu_id: guides
---

> New to KubeCI engine? Please start [here](/docs/concepts/README.md).

# Workflow Template

This tutorial will show you how to create a workflow-template and invoke it from a workflow. You can invoke same template from multiple workflows in the same namespace.

Before we start, you need to have a Kubernetes cluster, and the kubectl command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [Minikube](https://github.com/kubernetes/minikube). Now, install KubeCI engine in your cluster following the steps [here](/docs/setup/engine/install.md).

To keep things isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$ kubectl create ns demo
namespace/demo created
```

## Configure RBAC

You need to specify a service-account in `spec.serviceAccount` to ensure RBAC for the workflow. This service-account along with operator's service-account must have `list` and `watch` permissions for the resources specified in `spec.triggers`.

In this example, we are going to leave `spec.triggers` empty and trigger the workflow manually. So we don't need any of the permissions specified above. Also, we are going to use the default service account of `demo` namespace.

## Create Template and Workflow

```console
$ kubectl apply -f ./docs/examples/engine/template/template.yaml
workflowtemplate.engine.kube.ci/sample-template created
```

```yaml
apiVersion: engine.kube.ci/v1alpha1
kind: Workflow
metadata:
  name: sample-workflow
  namespace: demo
spec:
  serviceAccount: default
  executionOrder: Serial
  allowManualTrigger: true
  template:
    name: sample-template
    arguments:
      image: alpine
      cmd: echo hello world
```

```console
$ kubectl apply -f ./docs/examples/engine/template/workflow.yaml
workflow.engine.kube.ci/sample-workflow created
```

```yaml
apiVersion: engine.kube.ci/v1alpha1
kind: WorkflowTemplate
metadata:
  name: sample-template
  namespace: demo
spec:
  steps:
  - name: step-echo
    image: ${image=busybox}
    commands:
    - ${shell=sh}
    args:
    - -c
    - ${cmd}
```

After resolving template steps will look like below:

```yaml
steps:
- name: step-echo
  image: alpine      # substituted value
  commands:
  - sh               # default value
  args:
  - -c
  - echo hello world # substituted value
```

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
sample-workflow-smld7   14s
```

```console
$ kubectl get pods -l workplan=sample-workflow-smld7 -n demo
NAME                      READY   STATUS      RESTARTS   AGE
sample-workflow-smld7-0   0/1     Completed   0          48s
```

## Cleanup

```console
$ kubectl delete ns demo
namespace "demo" deleted
```
