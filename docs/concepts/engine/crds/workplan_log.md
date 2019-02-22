---
title: Workplan Log
menu:
  docs_0.1.0:
    identifier: workplan-log
    name: Workplan L0g
    parent: kubeci-crds-engine
    weight: 30
menu_name: docs_0.1.0
section_menu_id: concepts
---

# Workplan Log

## What is Workplan Log

A `WorkplanLog` is a representation of a Kubernetes object with the help of [Aggregated API Servers](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/api-machinery/aggregated-api-servers.md). User can stream logs of any workplan step by calling `Get` API of this custom resource.

API url for workplan logs:

`https://{master-ip}/apis/extensions.kube.ci/v1alpha1/namespaces/{namespace}/workplanlogs/{workplan-name}?step={step-name}&follow={true|false}`

Get using kubectl:

`kubectl get --raw /apis/extensions.kube.ci/v1alpha1/namespaces/{namespace}/workplanlogs/{workplan-name}?step={step-name}&follow={true|false}`