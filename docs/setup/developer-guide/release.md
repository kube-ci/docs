---
title: Release | KubeCI
description: KubeCI Release
menu:
  product_kubeci_0.1.0:
    identifier: release
    name: Release
    parent: developer-guide
    weight: 15
product_name: kubeci
menu_name: product_kubeci_0.1.0
section_menu_id: setup
---

# Release Process

The following steps must be done from a Linux x64 bit machine.

- Do a global replacement of tags so that docs point to the next release.
- Push changes to the `release-x` branch and apply new tag.
- Push all the changes to remote repo.
- Build and push KubeCI Engine docker images:

<ul class="nav nav-tabs" id="buildDockerTab" role="tablist">
  <li class="nav-item">
    <a class="nav-link active" id="engine-tab" data-toggle="tab" href="#engine" role="tab" aria-controls="engine" aria-selected="true">KubeCI Engine</a>
  </li>
  <li class="nav-item">
    <a class="nav-link" id="git-apiserver-tab" data-toggle="tab" href="#git-apiserver" role="tab" aria-controls="git-apiserver" aria-selected="false">Git API Server</a>
  </li>
</ul>
<div class="tab-content" id="buildDockerTabContent">
  <div class="tab-pane fade show active" id="engine" role="tabpanel" aria-labelledby="engine-tab">

```console
$ cd $(go env GOPATH)/src/github.com/kube-ci/engine
$ ./hack/docker/setup.sh; env APPSCODE_ENV=prod ./hack/docker/setup.sh release
```

</div>
<div class="tab-pane fade" id="git-apiserver" role="tabpanel" aria-labelledby="git-apiserver-tab">

```console
$ cd $(go env GOPATH)/src/github.com/kube-ci/git-apiserver
$ ./hack/docker/setup.sh; env APPSCODE_ENV=prod ./hack/docker/setup.sh release
```

</div>

- Now, update the release notes in Github. See previous release notes to get an idea what to include there.
