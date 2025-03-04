---
title: "Configuration"
date: 2019-01-08T17:58:08+05:30
draft: false
weight: 20
---

BotKube backend allows you to specify [resource](/configuration/resource), [communication](/configuration/communication), and [global](/configuration/global) settings. Check the related documents for more detailed explanation.

The configuration settings are read from two sources:

- the configuration files specified by the `BOTKUBE_CONFIG_PATHS` environment variable or `--config/-c` flag. For example:

  ```bash
  export BOTKUBE_CONFIG_PATHS="global.yaml,team-b-specific.yaml"
  # or
  ./botkube --config "global.yaml,team-b-specific.yaml"
  ```

  {{% notice note %}}
  You can split individual settings into multiple configuration files. The priority will be given to the last (right-most) file specified. See the [merging strategy](#merging-strategy) section for more details.
  {{% /notice%}}

- the exported [environment variables](#environment-variables) that overrides the configuration specified in the files.


### Updating the configuration at runtime

You can update the configuration and use `helm upgrade` to update configuration values for the BotKube.

You can also change configuration directly in ConfigMap and Secret - is not recommended but is great for quick experimentation.

```bash
# Change resources related settings
kubectl edit configmap botkube-global-config -n botkube
```

```bash
# Change communication related settings
kubectl edit secret botkube-communication-secret -n botkube
```

This command opens ConfigMap `specs` in default editor. Do the required changes, save and exit. The BotKube Pod will automatically restart to have these configurations in effect.

## Helm install options

Advanced Helm install options are documented [here](/configuration/helm-chart-parameters).

## Environment variables

The individual communication settings can be specified via environment variables. They take priority and override the configuration specified in the file.

To construct the environment variable name, take any property from the configuration file and make it uppercase. Use the underscore for properties that are nested. Use the double underscore for all camelCase properties. Finally, add the `BOTKUBE_` prefix.

For example, such configuration property from YAML:

```yaml
settings:
  kubectl:
    defaultNamespace: 'NAMESPACE'
```

is mapped to the `BOTKUBE_SETTINGS_KUBECTL_DEFAULT__NAMESPACE` environment variable.

This is a useful feature that allows you to store the overall configuration in a file, where sensitive data, such as tokens, can be put in environment variables. See the [**Tokens from Vault via CSI driver**](/configuration/communication/vault-csi/) tutorial for an example use-case.

## Merging strategy

BotKube allows you to split individual settings into multiple configuration files. The following rules apply:
- The priority will be given to the last (right-most) file specified.
- Objects are merged together and primitive fields are overridden. For example:
  ```yaml
  # a.yaml - first file
  settings:
    clusterName: dev-cluster
    configWatcher: true
    kubectl:
      enabled: false
  ```
  ```yaml
  # b.yaml - second file
  settings:
    kubectl:
      enabled: true
  ```

  ```yaml
  # result
  settings:
    clusterName: dev-cluster
    configWatcher: true
    kubectl:
      enabled: true
  ```

- The arrays items are not merged, they are overridden. For example:

  ```yaml
  # a.yaml
  settings:
    kubectl:
      enabled: true
      commands:
        verbs: ["api-resources", "api-versions", "cluster-info", "describe", "diff", "explain", "get", "logs", "top", "auth"]
  ```
  ```yaml
  # b.yaml
  settings:
    kubectl:
      commands:
        verbs: ["get", "logs", "top", "auth"]
  ```

  ```yaml
  # b.yaml
  settings:
    kubectl:
      enabled: true
      commands:
        verbs: ["get", "logs", "top", "auth"]
  ```
