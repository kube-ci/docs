---
title: Trigger
menu:
  product_kubeci_0.1.0:
    identifier: trigger-crd
    name: Trigger
    parent: kubeci-crds-engine
    weight: 25
product_name: kubeci
menu_name: product_kubeci_0.1.0
section_menu_id: concepts
---

# Trigger

## What is Trigger

A `Trigger` is a representation of a Kubernetes object with the help of [Aggregated API Servers](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/api-machinery/aggregated-api-servers.md). User can manually trigger a workflow by creating a `Trigger` resource. For this, `workflow.spec.allowManualTrigger` must be `true`. Note that, only `create` verb is available for this custom resource.

## Trigger structure

As with all other Kubernetes objects, a Trigger needs `apiVersion`, `kind`, and `metadata` fields. It also includes `.workflows` and `.request` sections. Below is an example Trigger object:

```yaml
apiVersion: extensions.kube.ci/v1alpha1
kind: Trigger
metadata:
  name: testing-manual-trigger
  namespace: default
workflows:
- wf-manual-trigger
request:
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: my-config
    namespace: default
  data:
    hello: world
```

Here, we are going to describe some important sections of `Trigger` object.

### .workflows

A list of workflows in the same namespace which will be considered for trigger. Note that, you need to specify at least one workflow name. You can also trigger all workflows in the same namespace by specifying `*` instead workflow names.

### .request

A complete representation of a Kubernetes object along with `apiVersion`, `kind`, and `metadata` fields. When a trigger is created, it will act as a fake create event for the object and can only be used to manually trigger a workflow without actually creating the object.

There are two possible triggering scenarios:

- `workflow.spec.triggers` empty: You can trigger such workflows only using manual trigger. For that you should leave `trigger.spec.request` empty.
- `workflow.spec.triggers` not empty: Your `trigger.spec.request` object must satisfy one of `workflow.spec.triggers`. You can also leave `trigger.spec.request` empty, but `envFromPath` will set to empty in that case.
