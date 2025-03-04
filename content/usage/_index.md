---
title: "Usage"
date: 2019-01-08T17:58:08+05:30
draft: false
weight: 40
---

### Debugging deployments using BotKube

@BotKube allows you to execute kubectl commands on your Kubernetes cluster.
Run **/botkubehelp** to find more information about the supported commands.


![help](/images/help.png)

By default, kubectl command execution is disabled. To enable this feature, set `settings.kubectl.enabled: true` in <a href=/configuration/#resource-config-yaml-syntax>resource_config</a>.

As suggested in help message, to execute kubectl commands, send message in following format in the channel where BotKube is already added or as a direct message to BotKube.
```
@BotKube <kubectl command without `kubectl` prefix> [--cluster-name <cluster_name>]
```

#### Checking allowed commands

While deploying BotKube controller, you can specify which kubectl commands you want to allow BotKube to execute through <a href=/configuration>configuration</a>.

To check which commands are allowed for users to execute through BotKube, run **@BotKube commands list**

![commands_list](/images/commands_list.png)

#### Run Kubectl commands

BotKube configuration allows you to give execution of kubectl commands

Run **@BotKube < kubectl command without kubectl prefix >** to get kubectl response from the cluster configured with the channel.
This command needs to be executed from configured channel else use `--cluster-name` flag described below.

![get_pods](/images/get_namespaces.png)
![get_pods](/images/mm_get_ns.png)

#### Specify cluster name
If you have installed BotKube backend on multiple clusters, you can pass `--cluster-name` flag to execute kubectl command on specific cluster.

To get the list of all clusters configured in botkube, you can use the ping command.

![ping](/images/ping.png)

For cluster-specific response,
use `--cluster-name` flag to specify the cluster's name on which command needs to be executed.
Use of this flag allows you to get response from any channel or group where BotKube is added.
The flag is ignored in notifier commands as they can be executed from the configured channel only.

![flag_clustername_ping](/images/flag_clustername_ping.png)
![flag_clustername_ping](/images/mm_flag_clustername_ping.png)
![flag_clustername_kubectl](/images/flag_clustername_kubectl.png)
![flag_clustername_kubectl](/images/mm_flag_clustername_kubectl.png)

See [Examples](/examples/#h-examples) for the use cases.

### Check BotKube health

Run **@BotKube ping** to the channel where BotKube is added. The BotKube will respond you with the **PONG** message from all the configured clusters. Use `--cluster-name` flag to get response from the cluster mentioned in the flag. Else check the deployment in Kubernetes cluster in the **botkube** namespace.

![ping](/images/ping.png)
![ping](/images/mm_ping.png)

### Managing notifications

Depending upon your configuration, you will receive notifications about Kubernetes resources lifecycle events and their health.
BotKube bot allows you to enable/disable notifications only from the configured channel. Run **/botkubehelp**, the bot will reply with the help message about the supported message formats.

#### View BotKube configuration

Run **@BotKube notifier showconfig** message from the configured channel where BotKube is added. The bot will reply you with the configuration with which the controller is running.

If you wish to change the configuration, you can update config section in **helm/botkube/values.yaml** and then run **helm upgrade**.

```bash
$ helm upgrade botkube \
--set config.settings.clusterName=<CLUSTER_NAME> \
--set config.settings.allowkubectl=<ALLOW_KUBECTL> \
helm/botkube
```
OR

You can also modify the controller configuration at runtime. You have to edit the configmap which will also restart the BotKube pod to update mounted configuration in the pod.

```bash
kubectl edit configmap botkube-global-config -n botkube
```

This command will open configmap specs in vim editor. Do the required changes, save and exit. The BotKube pod will automatically restart to have these configuration in effect.

#### Disable notifications

If you want to stop receiving notifications from BotKube, run **@BotKube notifier stop** from the configured channel where BotKube is added. You will no longer receive notifications from the BotKube in a given communication platform.

![notifier-stop](/images/notifier-stop.png)

#### Enable notifications

If you want to receive BotKube notifications again, run **@BotKube notifier start** from the configured channel where BotKube is added.

{{% notice note %}}
For MS Teams integration notifications are disabled by default. You need to turn them on manually using this command.
{{% /notice%}}

![notifier-start](/images/notifier-start.png)

#### Check notifier status

Run **@BotKube notifier status** to check if notifications are enabled for a given communication platform.

![notifier-status](/images/notifier-status.png)

### Manage filters

BotKube allows you to manage filters using @BotKube commands

#### List available filters

Run **@BotKube filters list** to get list of available filters and their running status
![](/images/filters_list.png)

#### Disable filter

Run **@BotKube filters disable {filter-name}** to disable perticular filter to skip checks on resource specs
![](/images/filters_disable.png)

#### Enable filter

Run **@BotKube filters enable {filter-name}** to enable perticular filter to run checks on resource specs
![](/images/filters_enable.png)

### Filter events using Annotations

Using Annotations, it is possible -

- to make BotKube ignore events on a specific resource
- Send notification about specific resource to different channel and the configured one

#### Ignore events

Annotation `botkube.io/disable: true` disables event notifications for the annotated object.

#### Send notification to non-default channel

Annotation `botkube.io/channel: <channel_name>` sends events notifications of the annotated object to the mentioned channel.

{{% notice note %}}
Make sure that you have added BotKube in the channel you want to send notification in
{{% /notice%}}
