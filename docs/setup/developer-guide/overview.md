---
title: Overview | Developer Guide
description: Developer Guide Overview
menu:
  docs_0.1.0:
    identifier: developer-guide-readme
    name: Overview
    parent: developer-guide
    weight: 15
menu_name: docs_0.1.0
section_menu_id: setup
---

# Development Guide

This document is intended to be the canonical source of truth for things like supported toolchain versions for building KubeCI. If you find a requirement that this doc does not capture, please submit an issue on github.

This document is intended to be relative to the branch in which it is found. It is guaranteed that requirements will change over time for the development branch, but release branches of KubeCI should not change.

## Build

Some of the KubeCI development helper scripts rely on a fairly up-to-date GNU tools environment, so most recent Linux distros should work just fine out-of-the-box.

### Setup GO

KubeCI is written in Google's GO programming language. Currently, KubeCI is developed and tested on **go 1.10**. If you haven't set up a GO development environment, please follow [these instructions](https://golang.org/doc/code.html) to install GO.

### Download Sources

```console
$ cd $(go env GOPATH)/src/github.com/kube-ci
$ git clone https://github.com/kube-ci/engine.git        # download KubeCI Engine
$ git clone https://github.com/kube-ci/git-apiserver.git # download Git API Server
```

### Install Dev tools

To install various dev tools for KubeCI engine, run the following command:

```console
$ ./hack/builddeps.sh
```

### Build Binary

<ul class="nav nav-tabs" id="buildBinaryTab" role="tablist">
  <li class="nav-item">
    <a class="nav-link active" id="engine-tab" data-toggle="tab" href="#engine" role="tab" aria-controls="engine" aria-selected="true">KubeCI Engine</a>
  </li>
  <li class="nav-item">
    <a class="nav-link" id="git-apiserver-tab" data-toggle="tab" href="#git-apiserver" role="tab" aria-controls="git-apiserver" aria-selected="false">Git API Server</a>
  </li>
</ul>
<div class="tab-content" id="buildBinaryTabContent">
  <div class="tab-pane fade show active" id="engine" role="tabpanel" aria-labelledby="engine-tab">

```console
$ cd $(go env GOPATH)/src/github.com/kube-ci/engine
$ ./hack/make.py
$ kubeci-engine version
```

</div>
<div class="tab-pane fade" id="git-apiserver" role="tabpanel" aria-labelledby="git-apiserver-tab">

```
$ cd $(go env GOPATH)/src/github.com/kube-ci/git-apiserver
$ ./hack/make.py
$ git-apiserver version
```

</div>

### Run Binary Locally

<ul class="nav nav-tabs" id="runBinaryTab" role="tablist">
  <li class="nav-item">
    <a class="nav-link active" id="engine-tab-01" data-toggle="tab" href="#engine-01" role="tab" aria-controls="engine-01" aria-selected="true">KubeCI Engine</a>
  </li>
  <li class="nav-item">
    <a class="nav-link" id="git-apiserver-tab-01" data-toggle="tab" href="#git-apiserver-01" role="tab" aria-controls="git-apiserver-01" aria-selected="false">Git API Server</a>
  </li>
</ul>
<div class="tab-content" id="runBinaryTabContent">
  <div class="tab-pane fade show active" id="engine-01" role="tabpanel" aria-labelledby="engine-tab-01">

```console
$ kubeci-engine run \
  --secure-port=8443 \
  --kubeconfig="$HOME/.kube/config" \
  --authorization-kubeconfig="$HOME/.kube/config" \
  --authentication-kubeconfig="$HOME/.kube/config" \
  --authentication-skip-lookup
```

</div>
<div class="tab-pane fade" id="git-apiserver-01" role="tabpanel" aria-labelledby="git-apiserver-tab-01">

```console
$ git-apiserver run \
  --secure-port=8443 \
  --kubeconfig="$HOME/.kube/config" \
  --authorization-kubeconfig="$HOME/.kube/config" \
  --authentication-kubeconfig="$HOME/.kube/config" \
  --authentication-skip-lookup
```

</div>

### Dependency management

KubeCI engine uses [Glide](https://github.com/Masterminds/glide) to manage dependencies. Dependencies are already checked in the `vendor` folder. If you want to update/add dependencies, run:

```console
$ glide slow
```

### Build Docker images

To build and push your custom Docker image, follow the steps below. To release a new version of KubeCI engine, please follow the [release guide](/docs/setup/developer-guide/release.md).

<ul class="nav nav-tabs" id="runBinaryTab" role="tablist">
  <li class="nav-item">
    <a class="nav-link active" id="engine-tab-01" data-toggle="tab" href="#engine-02" role="tab" aria-controls="engine-02" aria-selected="true">KubeCI Engine</a>
  </li>
  <li class="nav-item">
    <a class="nav-link" id="git-apiserver-tab-02" data-toggle="tab" href="#git-apiserver-02" role="tab" aria-controls="git-apiserver-02" aria-selected="false">Git API Server</a>
  </li>
</ul>
<div class="tab-content" id="runBinaryTabContent">
  <div class="tab-pane fade show active" id="engine-02" role="tabpanel" aria-labelledby="engine-tab-02">

```console
# Build Docker image
$ ./hack/docker/setup.sh; ./hack/docker/setup.sh push

# Add docker tag for your repository
$ docker tag kubeci/kubeci-engine:<tag> <image>:<tag>

# Push Image
$ docker push <image>:<tag>
```

</div>
<div class="tab-pane fade" id="git-apiserver-02" role="tabpanel" aria-labelledby="git-apiserver-tab-02">

```console
# Build Docker image
$ ./hack/docker/setup.sh; ./hack/docker/setup.sh push

# Add docker tag for your repository
$ docker tag kubeci/git-apiserver:<tag> <image>:<tag>

# Push Image
$ docker push <image>:<tag>
```

</div>

### Generate CLI Reference Docs

```console
$ ./hack/gendocs/make.sh
```

## Testing KubeCI engine

### Unit tests

```console
$ ./hack/make.py test unit
```

### Run e2e tests

KubeCI uses [Ginkgo](http://onsi.github.io/ginkgo/) to run e2e tests.

```console
$ ./hack/make.py test e2e
```

To run e2e tests against remote backends, you need to set cloud provider credentials in `./hack/config/.env`. You can see an example file in `./hack/config/.env.example`.
