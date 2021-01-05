---
title: "Multihop Ssh"
date: 2021-01-05T21:09:04Z
draft: true
---

### multihop tunnel
Following is the scenario:
* A localmachine (`192.168.0.17`)
* A remote machine (`192.168.0.19`)
* Another vagrant machine running inside remote machine. This Vagrant VM has private IP `10.0.0.10`.
* A web-service is listening on `10.0.0.10:31421` port inside Vagrant VM.
* We want the web-service to be accessible in the browser running in localmachine (i.e `192.168.0.17`)

```bash
ssh  -L 9090:localhost:8080 user1@192.168.0.19 "ssh -L 8080:10.0.0.10:31421 vagrant@127.0.0.1 -p 2200 -i ~/VAGRANT/.vagrant/machines/master/virtualbox/private_key"
```