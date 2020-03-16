---
title: "Saltstack Custom Execution Module With Arguments"
date: 2020-03-16T13:41:26Z
draft: true
tags: ["salt", "SaltStack"]
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
We will be creating a simple SaltStack execution module. And will see how can this module be called

* from a state file
* from command line

## Pre-Requisites
* salt-master installed and configured on a machine. I used fedora-24 VM.
* salt-minion installed and configured to accept command from salt-master.

`/etc/salt/master` file used:

```yaml
log_level: debug

fileserver_backend:
  - roots

file_roots:
  base:
    - /srv/salt
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

Now we are ready to execute this module to our minion (client machine). Run following command to sync modules first to all minions.

```bash
root@salt-master# salt '*' saltutil.sync_all
fedora2.vagrant.box:
    ----------
    beacons:
    engines:
    grains:
    log_handlers:
    modules:
        - modules.my_module
    output:
    proxymodules:
    renderers:
    returners:
    sdb:
    states:
    utils:
```

Now we are ready to execute this module to our minion (client machine). Run following command to sync modules first to all minions.

```bash
root@salt-master# salt 'fedora2.vagrant.box' my_module.my_module_function 'a_string' 'b_string2'
fedora2.vagrant.box:
    - a_string_modified
    - b_string2_modified
```

Now run above execution module using a state file.
```bash
root@salt-master# touch /srv/salt/state_file_call_custom_exec_module.sls
```
Contents of `/srv/salt/state_file_call_custom_exec_module.sls` are as follows:
```yaml
run_custom_module:
  module.run:
    - name: my_module.my_module_function
    - arg1: a_string
    - arg2: b_string
```

Run the following command to execute above state file on client machine:
```bash
root@salt-master# salt 'fedora2.vagrant.box' state.sls state_file_call_custom_exec_module
fedora2.vagrant.box:
----------
          ID: run_custom_module
    Function: module.run
        Name: my_module.my_module_function
      Result: True
     Comment: Module function my_module.my_module_function executed
     Started: 16:38:50.250250
    Duration: 1.085 ms
     Changes:
              ----------
              ret:
                  - a_string_modified
                  - b_string_modified

Summary for fedora2.vagrant.box
------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
Total run time:   1.085 ms
```


