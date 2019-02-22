---
title: Github Private Repo | Guides
description: Github Private Repo
menu:
  docs_0.1.0:
    identifier: guides-github-private
    name: Github Private Repo
    parent: guides-repositories
    weight: 3
menu_name: docs_0.1.0
section_menu_id: guides
---

> New to Git API server? Please start [here](/docs/concepts/README.md).

# Sync Public Repository

This tutorial will show you how to use Git API server to sync a public repository hosted in Github.

Before we start, you need to have a Kubernetes cluster, and the kubectl command-line tool must be configured to communicate with your cluster. If you do not already have a cluster, you can create one by using [Minikube](https://github.com/kubernetes/minikube). Now, install Git API server in your cluster following the steps [here](/docs/setup/git-apiserver/install.md). Also, configure Github webhook following the steps [here](/docs/guides/git-apiserver/webhook.md).

To keep things isolated, we are going to use a separate namespace called `demo` throughout this tutorial.

```console
$ kubectl create ns demo
namespace/demo created
```

## Create Secret

For syncing private repository, you have to create a secret with key `token` and value of Github API token.

```yaml
$ kubectl create secret -n demo generic github-credential --from-literal=token={github-api-token}
secret/github-credential created
```

## Create Repository CRD

Now, create the repository custom resource by specifying clone URL and secret.

```console
$ kubectl apply -f ./docs/examples/git-apiserver/repository-private.yaml
repository.git.kube.ci/private-test-repo created
```

```yaml
apiVersion: git.kube.ci/v1alpha1
kind: Repository
metadata:
  name: private-test-repo
  namespace: demo
spec:
  host: github
  owner: tamalsaha
  repo: private-test-repo
  cloneUrl: https://github.com/tamalsaha/private-test-repo.git
  tokenFormSecret: github-credential
```

## Get Synced Resources

```console
$ kubectl get all -n demo -l repository=private-test-repo
NAME                                          AGE
pullrequest.git.kube.ci/private-test-repo-1   16m

NAME                                          AGE
branch.git.kube.ci/private-test-repo-b001     16m
branch.git.kube.ci/private-test-repo-master   16m
```

## Cleanup

```console
$ kubectl delete ns demo
namespace "demo" deleted
```
