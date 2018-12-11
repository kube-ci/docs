> New to KubeCI engine? Please start [here](/docs/concepts/README.md).

# JSON Path Data

Your steps might need some information about the resource for which the workflow was triggered. This tutorial will show you how to populate explicit environment variables from json-path data of the triggering resource. When `envFromPath` is used for triggering resource, a secret is created with json-path data and this secret is used for populating environment variables.

Before we start, you need to have a Kubernetes cluster, and the kubectl command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [Minikube](https://github.com/kubernetes/minikube). Now, install KubeCI engine in your cluster following the steps [here](/docs/setup/install.md).

To keep things isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$ kubectl create ns demo
namespace/demo created
```

## Configure RBAC

You need to specify a service-account in `spec.serviceAccount` to ensure RBAC for the workflow. This service-account along with operator's service-account must have `list` and `watch` permissions for the resources specified in `spec.triggers`.

First, create a service-account for the workflow. Then, create a cluster-role with ConfigMap get, `list` and `watch` permissions. We also need secret create and get permissions. Now, bind the cluster-role with service-accounts of both workflow and operator.

```console
$ kubectl apply -f ./docs/examples/json-path/rbac.yaml
serviceaccount/wf-sa created
clusterrole.rbac.authorization.k8s.io/wf-role created
rolebinding.rbac.authorization.k8s.io/wf-role-binding created
clusterrolebinding.rbac.authorization.k8s.io/operator-role-binding created
```

## Create Workflow

```console
$ kubectl apply -f ./docs/examples/json-path/workflow.yaml
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
    envFromPath:
      ENV_ONE: '{$.data.example\.property\.1}'
      ENV_TWO: '{$.data.example\.property\.2}'
  serviceAccount: wf-sa
  allowManualTrigger: true
  steps:
  - name: step-one
    image: alpine
    commands:
    - sh
    args:
    - -c
    - echo ENV_ONE=$ENV_ONE; echo ENV_TWO=$ENV_TWO
```

## Trigger Workflow

Now trigger the workflow by creating a `Trigger` custom-resource which contains a complete ConfigMap resource inside `.request` section.

```console
$ kubectl apply -f ./docs/examples/json-path/trigger.yaml
trigger.extensions.kube.ci/sample-trigger created
```

Whenever a workflow is triggered, a workplan is created and respective pods are scheduled.

```console
$ kubectl get workplan -l workflow=sample-workflow -n demo
NAME                    CREATED AT
sample-workflow-8nfcr   5s
```

```console
$ kubectl get pods -l workplan=sample-workflow-8nfcr -n demo
NAME                      READY   STATUS      RESTARTS   AGE
sample-workflow-8nfcr-0   0/1     Completed   0          25s
```

## Check Logs

The `step-one` prints the values of explicit environment variables populated from json-path data of the triggering resource (which is `sample-config` configmap in this example).

```console
$ kubectl get --raw '/apis/extensions.kube.ci/v1alpha1/namespaces/demo/workplanlogs/sample-workflow-8nfcr?step=step-one'
ENV_ONE=hello
ENV_TWO=world
```

Here, we can see that, `HOME`, `NAMESPACE` and `WORKPLAN` environment variables are available in both containers.

## Cleanup

```console
$ kubectl delete ns demo
namespace "demo" deleted
```
