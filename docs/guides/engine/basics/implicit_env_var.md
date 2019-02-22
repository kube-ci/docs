---
title: Implicit Environment Variables | Guides
description: Implicit Environment Variables
menu:
  docs_0.1.0:
    identifier: guides-implicit-env
    name: Implicit Environment Variables
    parent: guides-basics
    weight: 6
menu_name: docs_0.1.0
section_menu_id: guides
---

> New to KubeCI engine? Please start [here](/docs/concepts/README.md).

# Implicit Environment Variables

By default `HOME`(shared `HOME` directory), `NAMESPACE` (namespace of workflow/workplan/pod) and `WORKPLAN` (name of the workplan) environment variables are set to all step-containers. If user specify environment variables with same name using `workflow.spec.envVar`, they will be replaced by default values.

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
$ kubectl apply -f ./docs/examples/engine/implicit-env-var/workflow.yaml
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
  executionOrder: Serial
  allowManualTrigger: true
  steps:
  - name: step-one
    image: alpine
    commands:
    - sh
    args:
    - -c
    - echo HOME=$HOME; echo NAMESPACE=$NAMESPACE; echo WORKPLAN=$WORKPLAN;
  - name: step-two
    image: alpine
    commands:
    - sh
    args:
    - -c
    - echo HOME=$HOME; echo NAMESPACE=$NAMESPACE; echo WORKPLAN=$WORKPLAN;
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
sample-workflow-gwd7c   5s
```

```console
$ kubectl get pods -l workplan=sample-workflow-gwd7c -n demo
NAME                      READY   STATUS      RESTARTS   AGE
sample-workflow-gwd7c-0   0/1     Completed   0          25s
```

## Check Logs

You can use KubeCI CLI to get logs of any step. In order to use KubeCI CLI as `kubectl` plugin follow the steps [here](/docs/setup/cli/install.md).

The `step-one` and `step-two` prints the values of `HOME`, `NAMESPACE` and `WORKPLAN` environment variables.

```console
$ kubectl ci logs sample-workflow-gwd7c --step step-one -n demo
HOME=/kubeci/home
NAMESPACE=default
WORKPLAN=sample-workflow-gwd7c
```

```console
$ kubectl ci logs sample-workflow-gwd7c --step step-two -n demo
HOME=/kubeci/home
NAMESPACE=default
WORKPLAN=sample-workflow-gwd7c
```

Here, we can see that, `HOME`, `NAMESPACE` and `WORKPLAN` environment variables are available in both containers.

## Cleanup

```console
$ kubectl delete ns demo
namespace "demo" deleted
```
