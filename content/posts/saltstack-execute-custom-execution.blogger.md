---
title: "SaltStack: Execute custom execution module on Master using runner"
date: 2020-03-16T08:37:22Z
draft: true
tag: ["runner", "salt", "SaltStack"]
---
<!--- Below style are also defined in static/css/my.css file.
They are repeatedly defined here so that pandoc can generate
the final HTML with all necessary css styles.
Note: draft: true above. This prevents publishing it to GitHUB.
--->
<style>
/* To highlight text in Green in pre tag */
.hl {color: #008A00;}
/* To highlight text in Bold Green in pre tag */
.hlb {color: #008A00; font-weight: bold;}
/* To highlight text in Bold Red in pre tag */
.hlbr {color:#e90001; font-weight: bold;}
/* <code> tag does not work in blogger. Use following class with span tag */
.code {
    color:#7e168d; 
    background: #f0f0f0; 
    padding: 0.1em 0.4em;
    font-family: SFMono-Regular, Consolas, "Liberation Mono", Menlo, Courier, monospace;
}
</style>

## Introduction:
Execution modules are meant to executed on minions(clients). Therefore only a handful of execution/state modules are visible to salt-master. But sometimes we may need to execute our custom execution module on Master itself (most likely via runner). In order to do this salt-minion must be installed on Salt-Master box. And configure it to accept the commands from salt-master. This will be configured as a normal salt-minion. Nothing special needs to be done.
We will create

* A custom execution module that accepts two arguments
* A custom runner module that accepts two arguments

## Pre-Requisites
* salt-master installed and configured on a machine. I used fedora-24 VM.
* salt-minion installed and configured to accept command from salt-master.

## Preparation
`/etc/salt/master` file used:
```yaml
log_level: debug

fileserver_backend:
  - roots

file_roots:
  base:
    - /srv/salt

runner_dirs:
  - /srv/salt/_runners
```
`/etc/salt/minion` file used:
```yaml
master: salt
```
**Note:** In `/etc/salt/master` file, `log_level` is set to `debug`. Normally this will be `info`.

`my_module.py` will be our execution module
```bash
mkdir /srv/salt/_modules
touch /srv/salt/_modules/my_module.py
mkdir /srv/salt/_runners
touch /srv/salt/_runners/my_runner.py
```
Contents of `/srv/salt/_modules/my_module.py` are as follows:
```python
#! /usr/bin/env python

# Below virtual name can be any string.
__virtualname__ = 'my_module'

def __virtual__():
    return __virtualname__

def my_module_function(arg1, arg2):
    modified_arg1 = '%s_%s' % (arg1, 'modified')
    modified_arg2 = '%s_%s' % (arg2, 'modified')
    return modified_arg1, modified_arg2
```

Contents of `/srv/salt/_runners/my_runner.py` are as follows:
```yaml
#! /usr/bin/env python
import salt.client
import salt.loader
import salt.config


def my_runner_function(arg1, arg2):
    __opts__ = salt.config.minion_config('/etc/salt/master')
    mods = salt.loader.minion_mods(__opts__)
    ret1, ret2 = mods['my_module.my_module_function'](arg1, arg2)
    return ret1, ret2
```
## Execution
Run following command to sync all modules with all minions:
```bash
root@salt-master# salt '*' saltutil.sync_all
fedora2.vagrant.box:
    ----------
    beacons:
    engines:
    grains:
    log_handlers:
    modules:
        - modules.custom_module
        - modules.my_module
    output:
    proxymodules:
    renderers:
    returners:
    sdb:
    states:
    utils:
fedora.vagrant.box:
    ----------
    beacons:
    engines:
    grains:
    log_handlers:
    modules:
        - modules.custom_module
        - modules.my_module
    output:
    proxymodules:
    renderers:
    returners:
    sdb:
    states:
    utils:
```
Above runner can be called from command line as follows:
```bash
root@salt-master# salt-run my_runner.my_runner_function 'a_string' 'b_string'
- a_string_modified
- b_string_modified
[INFO    ] Runner completed: 20170109185717568738
```


