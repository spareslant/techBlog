---
title: "Saltstack: Call runner from minion"
date: 2020-03-15T21:26:03Z
draft: false
tags: ["runner", "salt", "SaltStack"]
---

## Introduction:
Sometimes a minion may need to execute something which is central to whole system. e.g. Minion may need to update an external common Database whose write access is given only to master.
Therefore minion may need to ask salt-master to execute on its behalf. This can be done by

* Minion calling runner on salt-master
* runner executing a custom execution module that provides access to DB

As a custom execution module needs to be executed on salt-master, therefore salt-minion also needs to be installed on salt-master. And configure salt-minion to accept the commands from salt-master. This will be configured as a normal salt-minion. Nothing special needs to be done. Thus salt-master machine would be running both salt-master and salt-minion.

## Pre-Requisites
* salt-master installed and configured on a machine. I used fedora-24 VM.
* salt-minion installed and configured to accept command from salt-master.


## Preparation
`/etc/salt/master` file used.
```yaml
log_level: debug

peer_run:
  .*:
    - .*

fileserver_backend:
  - roots

file_roots:
  base:
    - /srv/salt

runner_dirs:
  - /srv/salt/_runners
```

`/etc/salt/minion` file used.
```yaml
master: salt
```
**Note:** In `/etc/salt/master` file, `log_level` is set to `debug`. Normally this will be `info`. Also take a note of `peer_run` config. This is must. At the moment this is very open and you may like to narrow it down.

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
```python
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

Above runner can be called from command line as follows:
```bash
root@salt-master# salt-run my_runner.my_runner_function 'a_string' 'b_string'
- a_string_modified
- b_string_modified
[INFO    ] Runner completed: 20170109185717568738
```

Now we will ask minion to execute runner on master.

Create a state file `/srv/salt/state_file_calling_runner_on_master.sls` as following:
```yaml
Below_is_state_that_calls_runner_from_minion:
  module.run:
    - name: publish.runner
    - m_fun: my_runner.my_runner_function
    - arg:
      - 'a_string'
      - 'b_string'


# Following will not work. Because in following case, publish.runner is a state module which does not exists in minions.
# In above case, publish.runner is actually an execution module.
# Below_is_state_that_calls_runner_from_minion:
#   publish.runner:
#     - name: my_runner.my_runner_function
#
#
```

## Execution
Run following command to sync all modules with all minions.
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

Now execute above state file on remote minion by running following command.
```bash
root@salt-master# salt 'fedora2.vagrant.box' state.sls state_file_calling_runner_on_master
fedora2.vagrant.box:
----------
          ID: Below_is_state_that_calls_runner_from_minion
    Function: module.run
        Name: publish.runner
      Result: True
     Comment: Module function publish.runner executed
     Started: 19:36:53.706742
    Duration: 462.667 ms
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
Total run time: 462.667 ms
```







