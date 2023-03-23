# plugins

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

1. for `PluginSpec.Type`: plugin type,currently only supported "SHELL".
2. for `PluginSpec.Data`: plugin`s real data, sealer will use it to do actual action.
3. for `PluginSpec.Scope`: plugin`s scope, it is usually the role name, support use '|' to specify multiple scopes.
4. for `PluginSpec.Action`: phase of this plugin will run. below is the phase list we currently supported.

plugin will be executed by `PluginSpec.Name` in alphabetical order at the same stage.

The following is a detailed introduction for plugin action.

| action name | action scope | explanation |
| :-----| ----: | :----: |
| pre-init-host | cluster host | will run before init cluster host |
| post-init-host | cluster host | will run after init cluster host  |
| pre-clean-host | cluster host | will run before clean cluster host  |
| post-clean-host | cluster host | will run after clean cluster host  |
| pre-install | master0 | will run before install cluster |
| post-install | master0 | will run after install cluster  |
| pre-uninstall | master0 | will run before uninstall cluster |
| post-uninstall | master0 | will run after uninstall cluster  |
| pre-scaleup | master0 | will run before scaleup cluster |
| post-scaleup | master0 | will run after scaleup cluster  |
| upgrade-host | cluster host | will run before upgrade cluster  |
| upgrade | master0 | will run for upgrading cluster  |

