---
title: "SaltStack: execute state modules on Master and minions from runner"
date: 2020-03-16T13:21:12Z
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
We will be creating a SaltStack runner module. This runner module will

* call a state module on salt-master itself
* execute a state file (hence state module) on a client machine

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

runner_dirs:
  - /srv/salt/_runners
```
`/etc/salt/minion` file used:
```yaml
master: salt
```
**Note1:** In `/etc/salt/master` file, `log_level` is set to `debug`. Normally this will be `info`.

**Note2:** we have to explicitly specify runners directory. In case of executions modules and state modules this is not required. we also need to restart salt-master if any entry is added to `/etc/salt/master` file.

```bash
mkdir /srv/salt/_runners
touch /srv/salt/_runners/custom_runner.py
```
Contents of `/srv/salt/_runners/custom_runner.py` are as follows:
```python
import salt.client
import salt.loader
import salt.config
import pprint

client = salt.client.LocalClient()

def call_state_mod_on_saltmaster_from_runner():
    __opts__ = salt.config.minion_config('/etc/salt/master')
    __grains__ = salt.loader.grains(__opts__)
    __opts__['grains'] = __grains__
    __utils__ = salt.loader.utils(__opts__)
    __salt__ = salt.loader.minion_mods(__opts__, utils=__utils__)

    # state_mods is not being populated with all the state modules. Perhaps arguments need to be set.
    state_mods = salt.loader.states(__opts__, None, None, None)
    execution_mods = salt.loader.minion_mods(__opts__)

    # print all state modules available in this runner
    #for mod in state_mods.items():
    #    print(mod)

    # print all execution modules available in this runner
    #for mod in execution_mods.items():
    #    print(mod)

    # Note that salt.wait_for_event is a state module that is available. But pkg.installed state module is not available.
    ret = state_mods['salt.wait_for_event'](name='myevent/test', id_list=['fedora'], timeout=10)
    #pprint.pprint(ret)
    return ret

def execute_state_file_on_minion_from_runner():
    #ret = client.cmd('*', 'state.sls', ['first'])

    # Another Variation
    ret = client.cmd('*', 'state.sls', kwarg={'mods': 'first'})
    #pprint.pprint(ret)
    return ret


if __name__ == '__main__':
    #call_state_mod_from_runner()
    execute_state_mod_on_minion_from_runner()
```

Now we are ready to execute this runner.

Call a state module on salt-master itself via runner.

```bash
root@salt-master# salt-run custom_runner.call_state_mod_on_saltmaster_from_runner
changes:
    ----------
comment:
    Timeout value reached.
name:
    myevent/test
result:
    False
[INFO    ] Runner completed: 20170109175501129807
```
**Note:** Please note that `wait_for_event` is a state module available to salt-Master. There are only a handful of state modules available to salt-master. Most of the state modules are for clients. Read comments in `custom_runner.py` file for more information.

Execute a state file from runner onto minion (client):
```bash
root@salt-master# salt-run custom_runner.execute_state_file_on_minion_from_runner
fedora2.vagrant.box:
    ----------
    pkg_|-install tcpdump_|-tcpdump_|-installed:
        ----------
        __id__:
            install tcpdump
        __run_num__:
            0
        changes:
            ----------
        comment:
            Package tcpdump is already installed
        duration:
            355.026
        name:
            tcpdump
        result:
            True
        start_time:
            17:59:58.929729
[INFO    ] Runner completed: 20170109175953282083
```

