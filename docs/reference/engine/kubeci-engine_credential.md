---
title: Kubeci-Engine Credential
menu:
  product_kubeci_0.1.0:
    identifier: kubeci-engine-credential
    name: Kubeci-Engine Credential
    parent: reference-engine
product_name: kubeci
menu_name: product_kubeci_0.1.0
section_menu_id: reference
---
## kubeci-engine credential

Run credential initializer

### Synopsis

Run credential initializer

```
kubeci-engine credential [flags]
```

### Options

```
      --basic-docker dockerConfig   List of secret=url pairs.
      --basic-git basicGitConfig    List of secret=url pairs.
  -h, --help                        help for credential
      --ssh-git sshGitConfig        List of secret=url pairs.
```

### Options inherited from parent commands

```
      --alsologtostderr                  log to standard error as well as files
      --bypass-validating-webhook-xray   if true, bypasses validating webhook xray checks
      --enable-analytics                 Send analytical events to Google Analytics (default true)
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

* [kubeci-engine](/docs/reference/engine/kubeci-engine.md)	 - Kubeci-engine by AppsCode

