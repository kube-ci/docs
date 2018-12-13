---
title: Shared Directories | Guides
description: Shared Directories
menu:
  product_kubeci_0.1.0:
    identifier: guides-shared-dir
    name: Shared Directories
    parent: guides-basics
    weight: 5
product_name: kubeci
menu_name: product_kubeci_0.1.0
section_menu_id: guides
---

> New to KubeCI engine? Please start [here](/docs/concepts/README.md).

# Shared Directories

The home directory and current working directory are shared among all step-containers. The shared working-directory helps to share input/output files among step-containers. For example, two step-containers might need same input files, again outputs of one step might be inputs of next step. The shared `HOME` directory helps to put common configuration files (like docker and git config) in a shared `HOME` directory.

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
$ kubectl apply -f ./docs/examples/engine/shared-directory/rbac.yaml
serviceaccount/wf-sa created
clusterrole.rbac.authorization.k8s.io/wf-role created
rolebinding.rbac.authorization.k8s.io/wf-role-binding created
clusterrolebinding.rbac.authorization.k8s.io/operator-role-binding created
```

## Create Workflow

```console
$ kubectl apply -f ./docs/examples/engine/shared-directory/workflow.yaml
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
  - name: step-print-dir
    image: alpine
    commands:
    - sh
    args:
    - -c
    - echo working-dir $(pwd); echo home-dir $HOME
  - name: step-create
    image: alpine
    commands:
    - sh
    args:
    - -c
    - touch file-01; touch $HOME/file-02
  - name: step-list-files
    image: alpine
    commands:
    - sh
    args:
    - -c
    - echo files in working-dir $(ls); echo files in home-dir $(ls $HOME)
```

## Trigger Workflow

Now trigger the workflow by creating a `Trigger` custom-resource which contains a complete ConfigMap resource inside `.request` section.

```console
$ kubectl apply -f ./docs/examples/engine/shared-directory/trigger.yaml
trigger.extensions.kube.ci/sample-trigger created
```

Whenever a workflow is triggered, a workplan is created and respective pods are scheduled.

```console
$ kubectl get workplan -l workflow=sample-workflow -n demo
NAME                    CREATED AT
sample-workflow-cxg4k   5s
```

```console
$ kubectl get pods -l workplan=sample-workflow-cxg4k -n demo
NAME                      READY   STATUS      RESTARTS   AGE
sample-workflow-cxg4k-0   0/1     Completed   0          25s
```

## Check Logs

The `step-print-dir` prints the path of `HOME` directory and current working directory. The working directory is set to `/kubeci/workspace` and `HOME` directory is set to `/kubeci/home` for all step-containers.

```console
$ kubectl get --raw '/apis/extensions.kube.ci/v1alpha1/namespaces/demo/workplanlogs/sample-workflow-cxg4k?step=step-print-dir'
working-dir /kubeci/workspace
home-dir /kubeci/home
```

The `step-create` creates `file-01` in working directory and `file-02` `HOME` directory. And the `step-list-files` lists the contents of working directory and `HOME` directory.

```console
$ kubectl get --raw '/apis/extensions.kube.ci/v1alpha1/namespaces/demo/workplanlogs/sample-workflow-cxg4k?step=step-list-files'
files in working-dir file-01
files in home-dir file-02
```

Here, we can see that, files created in `step-create` is also accessible by `step-list-files`.

## Cleanup

```console
$ kubectl delete ns demo
namespace "demo" deleted
```
