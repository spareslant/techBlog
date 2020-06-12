---
title: "AWS Cli"
date: 2020-04-19T21:06:24+01:00
draft: true
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


### List all images of ubuntu.
* aws --profile personal ec2 describe-images --filters 'Name=architecture, Values=x86_64' 'Name=hypervisor, Values=xen' 'Name=state, Values=available' 'Name=virtualization-type, Values=hvm' 'Name=name, Values=*ubuntu*' --query='Images[*].["Architecture", "ImageId", "Name"]' --output table

### List all users
* aws --profile personal iam list-users

### List all VPCs
* aws --profile personal ec2 describe-vpcs

### Get AWS password policy
* aws --profile personal iam get-account-password-policy

### Get all permissions for users/roles
* aws --profile personal iam get-account-authorization-details

### Get total summary 
*  aws --profile personal iam get-account-summary

### Get current user
* aws --profile personal iam get-user 

### List all instance types
* aws --profile personal ec2 describe-instance-types --query 'InstanceTypes[*].{Type:InstanceType, Free:FreeTierEligible, MemMB:MemoryInfo.SizeInMiB, Hypervisor:Hypervisor}' --output table



### Other AWS concepts:

* AWS load balancers don’t consist of a single server, but multiple servers that can run in separate subnets (and therefore, separate datacenters). AWS automatically scales the number of load balancer servers up and down based on traffic and handles failover if one of those servers goes down, so you get scalability and high availability out of the box.

* Internet Gateway in AWS is more like 1:1 NAT
* NAT gateway is more like traditional Source NAT (Masquerading).


