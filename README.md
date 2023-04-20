# plugins

## What is plugin

Sealer provide plugin mechanism that allows users to run sealer image according to different plugin configurations.

This will bring the possibility of flexible configuration of any image default setting, any cluster node, any cluster
startup stage, such as host disk initialization, cluster preflight ,status checker before or after cluster installation
and so on.

At present, sealer only support the `shell` type plugins. it provides a shell operation entry that means user could
execute any shell scripts on any node at any cluster launch stage. user only need to do is writing a yaml file which
defined shell data and its scope to experience sealer plugin feature.

## Plugin Spec

PluginSpec defines the desired state of Plugin

```
type PluginSpec struct {
	Type   string `json:"type,omitempty"`
	Data   string `json:"data,omitempty"`
	Action string `json:"action,omitempty"`
	Scope  string `json:"scope,omitempty"`
}
```

* for `PluginSpec.Type`: plugin type,currently only supported "SHELL".
* for `PluginSpec.Data`: plugin`s real data, sealer will use it to do actual action.
* for `PluginSpec.Scope`: plugin scope, it is usually the role name, support use '|' to specify multiple scopes.
* for `PluginSpec.Action`: phase of this plugin will run. if action is `host type`,will execute on `PluginSpec.Scope`
  specified, if it is `cluster type`, only execute plugin at master0. below is the phase list we currently supported.

plugin will be executed by `PluginSpec.Name` in alphabetical order at the same stage.

The following is a detailed introduction for plugin action.it is divided into 2 categories, one is the `host type`
plugin and the other is the `cluster type` plugin.

| action name | action scope | category| explanation |
| :-----| ----: | ----: |----: |
| pre-init-host | all host |host type | will run before init cluster host |
| post-init-host | all host | host type| will run after init cluster host  |
| pre-clean-host | all host | host type| will run before clean cluster host  |
| post-clean-host | all host | host type| will run after clean cluster host  |
| pre-install | master0 | cluster type| will run before install cluster |
| post-install | master0 |cluster type | will run after install cluster  |
| pre-uninstall | master0 |cluster type | will run before uninstall cluster |
| post-uninstall | master0 | cluster type| will run after uninstall cluster  |
| pre-scaleup | master0 | cluster type| will run before scaleup cluster |
| post-scaleup | master0 | cluster type| will run after scaleup cluster  |
| upgrade-host | all host | host type| will run before upgrade cluster  |
| upgrade | master0 | cluster type| will run for upgrading cluster  |

## How to use

### use plugin though Kubefile

Define the default plugin in Kubefile to build the image and run it.

In many cases it is possible to use plugins without using Clusterfile, essentially sealer stores the Clusterfile plugin
configuration in the Rootfs/Plugins directory before using it, so we can define the default plugin when we build the
image.

For example, build a new sealer image with plugins/example/example.yaml, only need to do is coping your plugin file
to `plugins/` directory.

Kubefile:

```shell script
FROM docker.io/sealerio/kubernetes:v1-22-15-sealerio-2
COPY example.yaml plugins/
```

Build a Sealer Image that contains a plugin (or more plugins):

```shell script
sealer build -t kubernetes-with-plugin:v1 .
```

Run the image and the plugin will also be executed without having to define the plugin in the Clusterfile:
`sealer run kubernetes-with-plugin:v1 -m x.x.x.x -p xxx`

### use plugin though Clusterfile

For example, set plugins/example/example.yaml content at clusterfile:

```yaml
apiVersion: sealer.io/v2
kind: Cluster
metadata:
  name: default-kubernetes-cluster
spec:
  image: docker.io/sealerio/kubernetes:v1.22.15
  ssh:
    passwd: xxx
  hosts:
    - ips: [ 192.168.0.2,192.168.0.3,192.168.0.4 ]
      roles: [ master ]
    - ips: [ 192.168.0.5 ]
      roles: [ node ]
---
apiVersion: sealer.io/v2
kind: Plugin
metadata:
  name: pre_init_host
spec:
  type: SHELL
  action: pre-init-host
  scope: master
  data: |
    set -e;set -x
    echo "i am pre init host plugin"
```

```shell script
sealer apply -f Clusterfile
```

## Available plugin list

| name | owner | category| link | description |
| :-----| ----: | ----: |----: |----: |
| example | sealer community | host type | plugins/example/example.yaml| just a little plugin example for user to quickly experience |