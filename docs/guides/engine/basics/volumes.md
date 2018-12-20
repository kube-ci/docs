---
title: Volumes | Guides
description: Volumes
menu:
  product_kubeci_0.1.0:
    identifier: guides-volumes
    name: Volumes
    parent: guides-basics
    weight: 8
product_name: kubeci
menu_name: product_kubeci_0.1.0
section_menu_id: guides
---

> New to KubeCI engine? Please start [here](/docs/concepts/README.md).

# Volumes

This tutorial will show you how to specify explicit volumes and mount them inside step-containers using volume-mounts.

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
$ kubectl apply -f ./docs/examples/engine/volumes/workflow.yaml
workflow.engine.kube.ci/wf-volumes created
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
  steps:
  - name: step-one
    image: alpine
    commands:
    - touch
    args:
    - /path-one/file-01
    volumeMounts: # explicit volume mounts
    - name: sample-volume
      mountPath: /path-one
  - name: step-two
    image: alpine
    commands:
    - ls
    args:
    - /path-two
    volumeMounts: # explicit volume mounts
    - name: sample-volume
      mountPath: /path-two
  volumes: # explicit volumes
  - name: sample-volume
    hostPath:
      path: /tmp/sample-volume
      type: DirectoryOrCreate
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
sample-workflow-gmjrl   13s
```

```console
$ kubectl get pods -l workplan=sample-workflow-gmjrl -n demo
NAME                      READY   STATUS      RESTARTS   AGE
sample-workflow-gmjrl-0   0/1     Completed   0          29s
```

## Check Logs

You can use KubeCI CLI to get logs of any step. In order to use KubeCI CLI as `kubectl` plugin follow the steps [here](/docs/setup/cli/install.md).

Both `step-one` and `step-two` mounts the same hostpath volume but in different paths. So the content created by `step-one` in path `/path-one` is also available to `step-two` in path `/path-two`.

```console
$ kubectl ci logs sample-workflow-gmjrl --step step-two -n demo
file-01
```

## Cleanup

```console
$ kubectl delete ns demo
namespace "demo" deleted
```
