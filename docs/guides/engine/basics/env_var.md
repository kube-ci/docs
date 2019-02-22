---
title: Environment Variables | Guides
description: Environment Variables
menu:
  docs_0.1.0:
    identifier: guides-env-var
    name: Environment Variables
    parent: guides-basics
    weight: 7
menu_name: docs_0.1.0
section_menu_id: guides
---

> New to KubeCI engine? Please start [here](/docs/concepts/README.md).

# Explicit Environment Variables

This tutorial will show you how to specify explicit environment variables and populate them from source(configmaps/secrets) inside all step-containers.

Before we start, you need to have a Kubernetes cluster, and the kubectl command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [Minikube](https://github.com/kubernetes/minikube). Now, install KubeCI engine in your cluster following the steps [here](/docs/setup/engine/install.md).

To keep things isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$ kubectl create ns demo
namespace/demo created
```

## Configure RBAC

You need to specify a service-account in `spec.serviceAccount` to ensure RBAC for the workflow. This service-account along with operator's service-account must have `list` and `watch` permissions for the resources specified in `spec.triggers`.

In this example, we are going to leave `spec.triggers` empty and trigger the workflow manually. So we don't need any of the permissions specified above. But secret get permission is required since we will populate environment variables from secret.

First, create a service-account for the workflow. Then, create a cluster-role with Secret `get` permission. Now, bind the cluster-role with service-accounts of both workflow and operator.

```console
$ kubectl apply -f ./docs/examples/engine/env-var/rbac.yaml
serviceaccount/wf-sa created
clusterrole.rbac.authorization.k8s.io/wf-role created
rolebinding.rbac.authorization.k8s.io/wf-role-binding created
clusterrolebinding.rbac.authorization.k8s.io/operator-role-binding created
```

## Create Secret

Create secret which will be used as source for populating environment variables.

```console
$ kubectl apply -f ./docs/examples/engine/env-var/secret.yaml
secret/sample-secret created
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: sample-secret
  namespace: demo
type: Opaque
data:
  KEY_ONE: c2VjcmV0LWRhdGEtb25l # echo -n 'secret-data-one' | base64
  KEY_TWO: c2VjcmV0LWRhdGEtdHdv # echo -n 'secret-data-two' | base64
```

## Create Workflow

```console
$ kubectl apply -f ./docs/examples/engine/env-var/workflow.yaml
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
  allowManualTrigger: true
  steps:
  - name: step-one
    image: alpine
    commands:
    - sh
    args:
    - -c
    - echo ENV_ONE=$ENV_ONE; echo ENV_TWO=$ENV_TWO; echo KEY_ONE=$KEY_ONE; echo KEY_TWO=$KEY_TWO
  envVar:
  - name: ENV_ONE
    value: one
  - name: ENV_TWO
    valueFrom:
      secretKeyRef:
        name: sample-secret
        key: KEY_TWO
  envFrom:
  - secretRef:
      name: sample-secret
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
sample-workflow-blkh9   5s
```

```console
$ kubectl get pods -l workplan=sample-workflow-blkh9 -n demo
NAME                      READY   STATUS      RESTARTS   AGE
sample-workflow-blkh9-0   0/1     Completed   0          25s
```

## Check Logs

You can use KubeCI CLI to get logs of any step. In order to use KubeCI CLI as `kubectl` plugin follow the steps [here](/docs/setup/cli/install.md).

The `step-one` prints the values of explicit environment variables.

```console
$ kubectl ci logs sample-workflow-blkh9 --step step-one -n demo
ENV_ONE=one
ENV_TWO=secret-data-two
KEY_ONE=secret-data-one
KEY_TWO=secret-data-two
```

Here, we can see that, `HOME`, `NAMESPACE` and `WORKPLAN` environment variables are available in both containers.

## Cleanup

```console
$ kubectl delete ns demo
namespace "demo" deleted
```
