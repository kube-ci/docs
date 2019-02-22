---
title: Status and Logs | Guides
description: Status and Logs
menu:
  docs_0.1.0:
    identifier: guides-logs
    name: Status and Logs
    parent: guides-cli
    weight: 1
menu_name: docs_0.1.0
section_menu_id: guides
---

> New to KubeCI engine? Please start [here](/docs/concepts/README.md).

# Workplan Status and Logs

This tutorial will show you how to use interactive `workplan-logs` CLI to collect logs of different steps and `workplan-viewer` web-ui to view workplan-status and logs.

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
sample-workflow-wjt8p   24s
```

```console
$ kubectl get pods -l workplan=sample-workflow-wjt8p -n demo
NAME                      READY   STATUS      RESTARTS   AGE
sample-workflow-wjt8p-0   0/1     Completed   0          58s
```

## Workplan Logs using API

To get/stream logs of a particular step of a workplan, you need to call the `Get` API of `WorkplanLog` custom resource.

```console
$ kubectl get --raw '/apis/extensions.kube.ci/v1alpha1/namespaces/demo/workplanlogs/sample-workflow-wjt8p?step=step-hello'
hello world
```

## Workplan Logs using CLI

You can also use workplan-logs CLI to get logs of any step. In order to use KubeCI CLI as `kubectl` plugin follow the steps [here](/docs/setup/cli/install.md).

You need to provide workplan-name as argument and step-name using flag. For example:

```console
$ kubectl ci logs sample-workflow-wjt8p --step step-hello -n demo
hello world
```

Alternatively, we can choose them interactively:

```console
$ kubectl ci logs -n demo
? Choose a Workflow: sample-workflow
? Choose a Workplan: sample-workflow-wjt8p
? Choose a running/terminated step: step-hello
hello world
```

## Workplan Viewer

The web-ui is deployed as a sidecar container during installation process. You can expose it with port forwarding. The status page will refresh after every 30 seconds. For `Running` and `Terminated` steps, there are links for accessing logs.

- Status URL: `http://127.0.0.1:9090/namespaces/{namespace}/workplans/{workplan-name}`
- Logs URL: `http://127.0.0.1:9090/namespaces/{namespace}/workplans/{workplan-name}/steps/{step-name}`

First, get the operator pod:

```console
$ kubectl get pods --all-namespaces | grep kubeci-engine
kube-system   kubeci-engine-5944797bb5-qxn5n                 3/3     Running     0          65m
```

Now, forward `9090` port of operator pod:

```console
$ kubectl port-forward -n kube-system kubeci-engine-5944797bb5-qxn5n 9090:9090
Forwarding from 127.0.0.1:9090 -> 9090
Forwarding from [::1]:9090 -> 9090
```

Go to following URL to get current of status workplan `sample-workflow-wjt8p`:

`http://127.0.0.1:9090/namespaces/demo/workplans/sample-workflow-wjt8p`

Go to following URL to get logs of step `step-hello`:

`http://127.0.0.1:9090/namespaces/demo/workplans/sample-workflow-wjt8p/steps/step-hello`

## Cleanup

```console
$ kubectl delete ns demo
namespace "demo" deleted
```
