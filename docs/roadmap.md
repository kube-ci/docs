---
title: Roadmap | KubeCI
description: Roadmap of KubeCI
menu:
  docs_0.1.0:
    identifier: roadmap-kubeci
    name: Roadmap
    parent: welcome
    weight: 15
menu_name: docs_0.1.0
section_menu_id: welcome
url: /docs/0.1.0/welcome/roadmap/
aliases:
  - /docs/0.1.0/roadmap/
---

# Project Status

## Versioning Policy

There are 2 parts to versioning policy:

 - Operator version: KubeCI operators __does not follow semver__. Currently KubeCI operator's implementation are considered alpha. Please report any issues you via Github. Once released, the _major_ version of operator is going to point to the Kubernetes [client-go](https://github.com/kubernetes/client-go#branches-and-tags) version. You can verify this from the `glide.yaml` file. This means there might be breaking changes between point releases of the operator. This generally manifests as changed annotation keys or their meaning. Please always check the release notes for upgrade instructions.
 - CRD version: `engine.kube.ci/v1alpha1`, `extensions.kube.ci/v1alpha1`, `git.kube.ci/v1alpha1` and `webhooks.git.kube.ci/v1alpha1` are considered in alpha. This means breaking changes to the YAML format might happen among different releases of the operator.
