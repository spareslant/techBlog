---
title: "Multihop SSH Tunnel to access Vagrant VM Service"
date: 2021-01-05T21:09:04Z
draft: true
tags: ["vagrantfile", "vagrant", "ssh", "ssh tunnel", "multi hop tunnel"]
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

## Multihop SSH tunnel to access Vagrant VM Service
### Following is the scenario:
* A localmachine (`192.168.0.17`)
* A remote machine (`192.168.0.19`)
* Another vagrant machine (call it `master`) running inside above remote machine. This Vagrant VM has private IP `10.0.0.10`.
* A web-service is listening on `10.0.0.10:31421` port inside Vagrant VM.
* We want this web-service to be accessible in the browser running in localmachine (i.e `192.168.0.17`)

### Get ssh port of Vagrant VM
```bash
vagrant port master
```
In my case it was 2200. 

### what does vagrant ssh port means?
Normally ssh into Vagrant VM is done by running following command.
```bash
vagrant ssh master
```
But you can also login to `master` using following command.
```bash
ssh -p 2200 vagrant@127.0.0.1 -i ~/VAGRANT/.vagrant/machines/master/virtualbox/private_key
```

### Create Tunnel
Run the following command from host having `192.168.0.17` IP.

```bash
ssh  -L "*":9090:localhost:8080 user1@192.168.0.19 "ssh -L 8080:10.0.0.10:31421 vagrant@127.0.0.1 -p 2200 -i ~/VAGRANT/.vagrant/machines/master/virtualbox/private_key"
```
Above command will create an ssh tunnel that will forawrd the port of the service listening on `10.0.0.10:31421` to `192.168.0.17`. You can then access this service on `https://192.168.0.17:9090` and `https://localhost:9090/`


#### VagrantFile used to create VM.
Following Vagrantfile was used to create VM inside remote-machine (192.168.0.19)

```ruby
all_hosts = [
    {
        vagrant_hostname: "master",
        full_hostname: "master.virtual.machine",
        vmbox: "ubuntu/bionic64",
        #vmbox_version: "31.20191023.0",
        ip: "10.0.0.10",
        memory: 4096,
        cpus: 3
    },
    # {
    #     vagrant_hostname: "worker1",
    #     full_hostname: "worker1.virtual.machine",
    #     vmbox: "ubuntu/bionic64",
    #     #vmbox_version: "31.20191023.0",
    #     ip: "10.0.0.12",
    #     memory: 2048,
    #     cpus: 1
    # },
]

# individual machine names must be mentioned is below command line in
# order to bring machines. (due to autostart: false)
# vagrant up master worker1
Vagrant.configure("2") do |config|

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
end
```