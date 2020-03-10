---
title: "Multiple Vms Vagrantfile"
date: 2020-03-10T17:50:49Z
draft: true
tags: ["Ruby", "Vagrantfile", "Vagrant"]
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

## Introduction
A `Vagrantfile` for multiple VMS.

## Prepare Environment

```bash
mkdir Vagrant_VMS
cd Vagrant_VMS
vagrant init
```

## Populate `Vagrantfile` with following contents.

```Ruby
all_hosts = [
    {
        vagrant_hostname: "amachine",
        full_hostname: "amachine.virtual.machine",
        vmbox: "fedora/31-cloud-base",
        vmbox_version: "31.20191023.0",
        ip: "10.0.0.10",
        memory: 2048,
        cpus: 1
    },
#    {
#        vagrant_hostname: "anotherMachine",
#        full_hostname: "another.virtual.machine",
#        vmbox: "fedora/31-cloud-base",
#        vmbox_version: "31.20191023.0",
#        ip: "10.0.0.12",
#        memory: 2048,
#        cpus: 1
#    },
]

# individual machine names must be mentioned is below command line in
# order to bring machines. (due to autostart: false)
# vagrant up amachine anotherMachine
Vagrant.configure("2") do |config|
    #config.vm.box = "fedora/31-cloud-base"
    #config.vm.box_version = "31.20191023.0"

    all_hosts.each do |host|
        config.vm.define host[:vagrant_hostname], autostart: false do |this_host|
            this_host.vm.network :private_network, ip: host[:ip]
            this_host.vm.hostname = host[:full_hostname]
            this_host.vm.box = host[:vmbox]
            this_host.vm.box_version = host[:vmbox_version]

            this_host.vm.provider "virtualbox" do |m|
                m.memory = host[:memory]
                m.cpus = host[:cpus]
            end
        end
    end
    config.vm.synced_folder ".", "/vagrant", type: "nfs"
end
```

## Start VMs
```bash
vagrant up amachine
```
**Note:** You need to mention vm name in `vagrant up` command due to presense of `autostart: false` in above Vagrantfile. 

