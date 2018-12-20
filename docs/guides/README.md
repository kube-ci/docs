---
title: Table of Contents | Guides
description: Table of Contents | Guides
menu:
  product_kubeci_0.1.0:
    identifier: guides-readme
    name: Readme
    parent: guides
    weight: -1
product_name: kubeci
menu_name: product_kubeci_0.1.0
section_menu_id: guides
url: /products/kubeci/0.1.0/guides/
aliases:
  - /products/kubeci/0.1.0/guides/README/
---

# Guides

Guides show you how to perform tasks with KubeCI engine.

## Workflow Basics

- Learn how to configure and trigger a simple workflow [here](/docs/guides/engine/basics/hello_world.md).
- Learn how to manually trigger a workflow [here](/docs/guides/engine/basics/manual_trigger.md).
- Learn how to configure a set of serial tasks [here](/docs/guides/engine/basics/serial_execution.md).
- Learn how to configure a set of parallel tasks [here](/docs/guides/engine/basics/parallel_execution.md).
- Learn how to specify DAG dependency [here](/docs/guides/engine/basics/dag_execution.md).
- Learn how to invoke workflow template [here](/docs/guides/engine/basics/template.md).
- Learn how shared directories works [here](/docs/guides/engine/basics/shared_directory.md).
- Learn which environment variables are set by default [here](/docs/guides/engine/basics/implicit_env_var.md).
- Learn how specify explicit volumes and volume-mounts [here](/docs/guides/engine/basics/volumes.md).
- Learn how to specify environment variables [here](/docs/guides/engine/basics/env_var.md).
- Learn how to set environment variables from json-path data [here](/docs/guides/engine/basics/json_path.md).

## Credential Initializer

- Learn how initialize Docker and Git credentials [here](/docs/guides/engine/credential/credential_initializer.md).

## CLI and Web UI

- Learn how to use workplan-logs CLI and workplan-viewer web-ui [here](/docs/guides/engine/cli/workplan_status_logs.md).

## Build and Deploy

- Learn how to build container from source using 
  - [dind](/docs/guides/engine/build/build-dind.md)
  - [kaniko](/docs/guides/engine/build/build-kaniko.md)
  - [img](/docs/guides/engine/build/build-img.md)
- Learn how to deploy your application from source using a single workflow [here](/docs/guides/engine/build/deploy.md).

## Git Repositories

- Learn how to configure webhook for a Github repository [here](/docs/guides/git-apiserver/webhook.md).
- Learn how to sync a Github public repository [here](/docs/guides/git-apiserver/github_public.md).
- Learn how to sync a Github private repository [here](/docs/guides/git-apiserver/github_private.md).

## Walk-through

- Step by step guide to run go-tests for a Github pull-request and set commit status [here](/docs/guides/walk-through/github_pr.md).