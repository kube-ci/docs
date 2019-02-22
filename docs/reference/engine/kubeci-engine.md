---
title: Kubeci-Engine
menu:
  docs_0.1.0:
    identifier: kubeci-engine
    name: Kubeci-Engine
    parent: reference-engine
    weight: 0

product_name: kubeci
menu_name: docs_0.1.0
section_menu_id: reference
aliases:
  - /docs/0.1.0/reference/engine/

---
## kubeci-engine

Kubeci-engine by AppsCode

### Synopsis

Kubeci-engine by AppsCode

### Options

```
      --alsologtostderr                  log to standard error as well as files
      --bypass-validating-webhook-xray   if true, bypasses validating webhook xray checks
      --enable-analytics                 Send analytical events to Google Analytics (default true)
  -h, --help                             help for kubeci-engine
      --log-flush-frequency duration     Maximum number of seconds between log flushes (default 5s)
      --log_backtrace_at traceLocation   when logging hits line file:N, emit a stack trace (default :0)
      --log_dir string                   If non-empty, write log files in this directory
      --logtostderr                      log to standard error instead of files (default true)
      --stderrthreshold severity         logs at or above this threshold go to stderr
      --use-kubeapiserver-fqdn-for-aks   if true, uses kube-apiserver FQDN for AKS cluster to workaround https://github.com/Azure/AKS/issues/522 (default true)
  -v, --v Level                          log level for V logs
      --vmodule moduleSpec               comma-separated list of pattern=N settings for file-filtered logging
```

### SEE ALSO

* [kubeci-engine credential](/docs/reference/engine/kubeci-engine_credential.md)	 - Run credential initializer
* [kubeci-engine run](/docs/reference/engine/kubeci-engine_run.md)	 - Launch kubeci-engine
* [kubeci-engine version](/docs/reference/engine/kubeci-engine_version.md)	 - Prints binary version number.
* [kubeci-engine workplan-viewer](/docs/reference/engine/kubeci-engine_workplan-viewer.md)	 - Start workplan-viewer server

