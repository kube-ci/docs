---
title: Table of Contents | Setup
description: Table of Contents | Setup
menu:
  docs_0.1.0:
    identifier: setup-readme
    name: Readme
    parent: setup
    weight: -1
product_name: kubeci
menu_name: docs_0.1.0
section_menu_id: setup
url: /docs/0.1.0/setup/
aliases:
  - /docs/0.1.0/setup/README/
---

# Setup

Setup contains instructions for installing the KubeCI Engine and Git API Server in Kubernetes.

## KubeCI Engine

KubeCI engine is a Kubernetes native workflow engine. For more details see [here](/docs/concepts/engine/what-is-kubeci-engine/overview.md).

- [Install KubeCI engine](/docs/setup/engine/install.md). Installation instructions for KubeCI engine.
- [Uninstall KubeCI engine](/docs/setup/engine/uninstall.md). Instructions for uninstallating KubeCI engine.
- [Upgrading KubeCI engine](/docs/setup/engine/upgrade.md). Instructions for upgrading KubeCI engine.
  
## Git API Server

Git API server is a Kubernetes operator for syncing Git repositories as Kubernetes resources. For more details see [here](/docs/concepts/git-apiserver/what-is-git-apiserver/overview.md).

- [Install Git API server](/docs/setup/git-apiserver/install.md). Installation instructions for Git API server.
- [Uninstall Git API server](/docs/setup/git-apiserver/uninstall.md). Instructions for uninstallating Git API server.
- [Upgrading Git API server](/docs/setup/git-apiserver/upgrade.md). Instructions for upgrading Git API server.

## KubeCI CLI

[Install KubeCI CLI](/docs/setup/cli/install.md). Installation instructions for KubeCI CLI as `kubectl` plugin.
  
## Developer Guide

- [Overview](/docs/setup/developer-guide/overview.md). Outlines everything you need to know from setting up your dev environment to how to build and test KubeCI.
- [Release](/docs/setup/developer-guide/release.md). Steps for releasing a new version of KubeCI.
