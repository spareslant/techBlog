---
title: "Terraform CDK Create OCI OracleCloudInfraStructure VM"
date: 2021-09-20T23:40:41+01:00
draft: true
tags: ["terraform", "OCI", "oci-cli", "cdk", "python", "Terraform CDK", "cdktf", "Oracle Cloud Infrastructure"]
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

# Introduction
We will be using `terraform-cdk` toolkit in this article. We shall be using an `OCI (Oracle Cloud Infrastructure)` account to spin up a VM. 

We shall be doing following.

##### Manual steps to prepare cloud environment first (using `oci-cli`)

* Create a new user account `cdk-user` (apart from the tenancy admin) and new `compartment` called as `CDK` for this user.
* `cdk-user` will have full privleges to `CDK` compartment.
* This is done as per Oracle recommended practices for `OCI`.

##### Steps performed by terraform CDK.
* Create a brand new `VCN` in `CDK` compartment.
* Create 1 public subnet.
* A key-pair for ssh
* An internet-gateway in this VCN (It is needed to get bi-directional internet connectity to new VCN.)
* A new VM created in above mentioned VCN, and public subnet having internet connectivity and allows ssh connection from internet.

**Note1:** We shall be using `python` for terraform-cdk.

## Prepare OCI environment
* You need an `OCI` account. Its free. SignUp at https://cloud.oracle.com. This sign-up account is called `Tenancy Admin` account.
* Login to this `Tenancy Admin` account. Make sure you have selected `Oracle Cloud Infrastructure Direct Sign-In` option on the login page.
* click hamburger icon on the top-left corner 
    * click `Identity & Security`
    * click `users`
    * click your email ID here (the one you used for sign-up)
    * click `API Keys`
    * click `Add API Key`
    * select `Generate API Key Pair`
    * click `Download private key`
    * click `Add` button
    * Copy the content in `Configuration File Preview` and save it. We need it later on.
    * click `close`

## Prepare local development environment
We shall be doing everything in a docker image. I am using fedora-34.

#### Start fedora-34 in interactive mode.
```bash
$ docker run -it fedora:34 /bin/bash

Unable to find image 'fedora:34' locally
34: Pulling from library/fedora
b9705287bb9f: Pull complete
Digest: sha256:d18bc88f640bc3e88bbfacaff698c3e1e83cae649019657a3880881f2549a1d0
Status: Downloaded newer image for fedora:34
[root@248b335b1e23 /]#
```

#### Install NVM (node version manager) inside docker container.
```bash
[root@248b335b1e23 /]# curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 14984  100 14984    0     0  43057      0 --:--:-- --:--:-- --:--:-- 43057
=> Downloading nvm as script to '/root/.nvm'

=> Appending nvm source string to /root/.bashrc
=> Appending bash_completion source string to /root/.bashrc
=> Close and reopen your terminal to start using nvm or run the following to use it now:

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
[root@248b335b1e23 /]#
```

#### Enable NVM in current shell
```bash
[root@248b335b1e23 /]# source /root/.bashrc
```

#### Install latest stable node.
```bash
[root@248b335b1e23 /]# nvm install --lts
Installing latest LTS version.
Downloading and installing node v14.17.6...
Downloading https://nodejs.org/dist/v14.17.6/node-v14.17.6-linux-x64.tar.gz...
################################################################################################################################################################################################## 100.0%
Computing checksum with sha256sum
Checksums matched!
Now using node v14.17.6 (npm v6.14.15)
Creating default alias: default -> lts/* (-> v14.17.6 *)
[root@248b335b1e23 /]#
```

#### Install VIM, unzip, less
```bash
[root@248b335b1e23 /]# dnf install vim unzip less openssh-clients jq openssl -y
```

#### Install oci-cli
```bash
[root@248b335b1e23 /]# dnf install python -y
[root@248b335b1e23 /]# pip install oci-cli
```

## Configure Tenancy Admin account to access OCI via APIs
You can run `oci setup config` command to setup the oci config. But we will be following direct manual method as we already have config saved in previous step when we prepared the oci envrionment.
```bash
[root@248b335b1e23 /]# mkdir ~/.oci
[root@248b335b1e23 /]# chmod g-rwx,o-rwx /root/.oci
[root@248b335b1e23 /]# ls -ld /root/.oci/
drwx------ 2 root root 4096 Sep 20 23:36 /root/.oci/
[root@248b335b1e23 /]# touch ~/.oci/tenancyAdmin_private_api_key.pem
[root@248b335b1e23 /]# vim ~/.oci/tenancyAdmin_private_api_key.pem
```
Paste the contents from file that you downloaded during the step `download private key` above in file `~/.oci/tenancyAdmin_private_api_key.pem`

```bash
[root@248b335b1e23 /]# chmod 600 ~/.oci/tenancyAdmin_private_api_key.pem
[root@248b335b1e23 /]# touch ~/.oci/config
[root@248b335b1e23 /]# chmod 600 ~/.oci/config
[root@248b335b1e23 /]# vim  ~/.oci/config
```
Paste the contents from file that you saved during the step `Configuration file preview` above in file `~/.oci/config`

Contents of `~/.oci/config` will be similar to the following.
```ini
[DEFAULT]
user=ocid1.user.oc1..<a very long string>
fingerprint=xx:yy:11:22:33:44:d4:56:b6:67:89:b7:b1:7f:4f:7a
tenancy=ocid1.tenancy.oc1..<a very long string>
region=uk-london-1
key_file=~/.oci/tenancyAdmin_private_api_key.pem
```
Please note `key_file=` above. You need to have exactly the same entry as above.

#### Verify connectivity to OCI
```bash
[root@248b335b1e23 /]# oci iam user list
```
Above command must run successfully.

## Create and setup `cdk-user` 

We will be creating a new user `cdk-user` and new compartment `CDK`, where this user can manage anything.
Following are the manual steps. But there is a script `setup_oci_user_account.sh` as well in git repo that can also do the same.
Link to the git repo is mentioned at the bottom of this article.

#### Get reuired info in variables.
```bash
[root@248b335b1e23 /]# export tenancyID=$(cat ~/.oci/config | egrep '^tenancy' | awk -F"=" '{print $2}')
[root@248b335b1e23 /]# export rootCompartmentID=$tenancyID
[root@248b335b1e23 /]# export tenancyRegion=$(cat ~/.oci/config | egrep region |  awk -F"=" '{print $2}')
```
Note: `Root compartmentID` is same as `TenancyID`


#### Create `CDK` compartment
```bash
[root@248b335b1e23 /]# oci iam compartment create --name CDK --compartment-id $rootCompartmentID --description "CDK Compartment"
```

Take a note of `CDK` compartment id from above output.

#### Create `cdk-user`
```bash
[root@248b335b1e23 /]# oci iam user create --name cdk-user --compartment-id $rootCompartmentID --description "cdk user"
```

#### Create `cdk-group`
```bash
[root@248b335b1e23 /]# oci iam group create --name cdk-group --compartment-id $rootCompartmentID --description "cdk group"
```

#### Add `cdk-user` to `cdk-group`
```bash
[root@248b335b1e23 /]# export cdk_user_ocid=$(oci iam user list --name cdk-user | jq -r '.data[0].id')
[root@248b335b1e23 /]# export cdk_group_ocid=$(oci iam group list --name cdk-group | jq -r '.data[0].id')
[root@248b335b1e23 /]# oci iam group add-user --user-id $cdk_user_ocid --group-id $cdk_group_ocid
```

#### Allow `cdk-group` to do anything in compartment `CDK`
```bash
[root@248b335b1e23 /]# export CDKcompartmentID=$(oci iam compartment list  --compartment-id $rootCompartmentID --lifecycle-state ACTIVE | jq -r '.data[] | select(.name == "CDK") | .id')

[root@248b335b1e23 /]# oci iam policy create --name "CDK_Policies" --compartment-id $CDKcompartmentID --description "Policies for CDK" --statements '["Allow group cdk-group to manage all-resources in compartment CDK"]'
```

#### Generate API keys for `cdk-user`
```bash
[root@248b335b1e23 /]# cd ~/.oci/
[root@248b335b1e23 .oci]# openssl genrsa -out cdk-user_private_api_key.pem 2048
[root@248b335b1e23 .oci]# openssl rsa -pubout -in cdk-user_private_api_key.pem -out cdk-user_public_api_key.pem
[root@248b335b1e23 .oci]# chmod go-rwx cdk-user_private_api_key.pem
```
#### Upload public key for `cdk-user`
```bash
[root@248b335b1e23 .oci]# oci iam user api-key upload --user-id $cdk_user_ocid --key-file ~/.oci/cdk-user_public_api_key.pem
```

#### preapre `cdk-user` profile in `~/.oci/config` file
```bash
[root@248b335b1e23 .oci]# export cdk_user_fingerprint=$(openssl rsa -pubout -outform DER -in ~/.oci/cdk-user_private_api_key.pem 2> /dev/null | openssl md5 -c | awk '{print $2}')
[root@248b335b1e23 .oci]# cat <<HELLO >> ~/.oci/config
[cdk-user]
user=$cdk_user_ocid
fingerprint=$cdk_user_fingerprint
tenancy=$tenancyID
region=$tenancyRegion
key_file=~/.oci/cdk-user_private_api_key.pem
HELLO
```
A new section has been appened to `~/.oci/config file`

#### verify `cdk-user` access
```bash
[root@248b335b1e23 .oci]# oci iam user get --user-id=$cdk_user_ocid --profile cdk-user
```

Note: `--profile cdk-user` in above command

## Install tools required for development

#### Install terraform binary
```bash
[root@248b335b1e23 /]# dnf install -y dnf-plugins-core
[root@248b335b1e23 /]# dnf config-manager --add-repo https://rpm.releases.hashicorp.com/fedora/hashicorp.repo
[root@248b335b1e23 /]# dnf -y install terraform
```

#### Install `terraform-cdk` kit
```bash
[root@248b335b1e23 /]# npm install --global cdktf-cli
[root@248b335b1e23 /]# cdktf --version
0.6.2
```

#### Install `pipenv`
```bash
[root@248b335b1e23 /]# pip install pipenv
```
`pipenv` is better than `pip`. It is similar to nodejs `npm` i.e keeps a lockfile as well as packages file in working directory. `cdktf-cli` can use both `pip` and `pipenv`, but we will be using `pipenv` mode.


## Start coding.

#### Create a project with `python` template
```bash
[root@248b335b1e23 ~]# mkdir ~/oci_terraform_cdk_python
[root@248b335b1e23 ~]# cd ~/oci_terraform_cdk_python/

[root@248b335b1e23 oci_terraform_cdk_python]# cdktf init --template="python" --local

Note: By supplying '--local' option you have chosen local storage mode for storing the state of your stack.
This means that your Terraform state file will be stored locally on disk in a file 'terraform.<STACK NAME>.tfstate' in the root of your project.
? projectName: oci_terraform_cdk_python
? projectDescription: A public VM in OCI
```

#### Install OCI sdk and other required libraries
```bash
[root@248b335b1e23 oci_terraform_cdk_python]# pipenv install pycryptodome oci
```

#### Files in current direcotry
```bash
[root@248b335b1e23 oci_terraform_cdk_python]# ls -a
.  ..  .gitignore  Pipfile  Pipfile.lock  cdktf.json  help  main.py
```

#### Download OCI terraform modules libraries

##### Add terraform provider information in `cdktf.json` file
```python
{
  "language": "python",
  "app": "pipenv run python main.py",
  "projectId": "ae857ab0-8c78-424b-97cf-aeb5f3b56d63",
  "terraformProviders": [
    "oci@~> 4.44.0"
  ],
  "terraformModules": [],
  "codeMakerOutput": "imports",
  "context": {
    "excludeStackIdFromLogicalIds": "true",
"allowSepCharsInLogicalIds": "true"
  }
}
```

##### Get the OCI terraform libraries
```bash
[root@248b335b1e23 oci_terraform_cdk_python]# cdktf get

[root@248b335b1e23 oci_terraform_cdk_python]# ls
Pipfile  Pipfile.lock  account.py  cdktf.json  help  imports  main.py
```

#### Create a helper library `account.py` with following contents
```bash
[root@248b335b1e23 oci_terraform_cdk_python]# touch account.py
```

##### `account.py` contents
```python
#! /usr/bin/env python

import oci
from Crypto.PublicKey import RSA
import os

keys_dir = "keys"
profile_name = "cdk-user"
compartment_name = "CDK"

config = oci.config.from_file("~/.oci/config", profile_name)
identity = oci.identity.IdentityClient(config)
user = identity.get_user(config["user"]).data
compartment_id = user.compartment_id

# compartments = identity.list_compartments(compartment_id, compartment_id_in_subtree=True, lifecycle_state="ACTIVE", access_level="ACCESSIBLE")

# print(compartments.data)
# print(compartments.data[0])

def get_availability_domain():
    list_availability_domains_response = oci.pagination.list_call_get_all_results(
        identity.list_availability_domains,
        compartment_id
    )
    availability_domain = list_availability_domains_response.data[0]

    return availability_domain.name

def get_compartment_id(comp_name=compartment_name) -> str:

    desired_compartment_id: str = ""

    for comp in oci.pagination.list_call_get_all_results_generator(
            identity.list_compartments,
            'record',
            compartment_id,
            compartment_id_in_subtree=True,
            lifecycle_state="ACTIVE"):
        if comp.name == comp_name:
            desired_compartment_id = comp.id
    return desired_compartment_id

def generate_key_pair():
    os.mkdir("keys")
    key = RSA.generate(2048)
    private_key = key.export_key("PEM")
    file_out = open(f"{keys_dir}/private.pem", "wb")
    file_out.write(private_key)
    file_out.close()
    os.chmod(f"{keys_dir}/private.pem", 0o600)

    public_key = key.publickey().export_key("OpenSSH")
    file_out = open(f"{keys_dir}/public.pem", "wb")
    file_out.write(public_key)
    file_out.close()

    return public_key.decode("utf-8")


def get_key_pair(use_existing_keys=True):
    if use_existing_keys:
        if not os.path.isfile(f"{keys_dir}/private.pem"):
            return generate_key_pair()
        else:
            with open(f"{keys_dir}/public.pem", 'rb') as f:
                public_key = f.read()
            return public_key.decode("utf-8")
    else:
        return generate_key_pair()


if  __name__ == '__main__':
    print(f"desired_compartment_id = {get_compartment_id()}")
    print(f"availability_domain = {get_availability_domain()}")
    print(get_key_pair())
```

#### update `main.py` with following contents

##### `main.py` contents
```python
#!/usr/bin/env python

from constructs import Construct
from cdktf import App, TerraformOutput, TerraformStack
from imports.oci import (CoreDhcpOptionsOptions,
    CoreRouteTableRouteRules,
    CoreVcn,
    OciProvider,
    CoreInstance,
    CoreSubnet,
    CoreDhcpOptions,
    CoreDhcpOptionsOptions,
    CoreInstanceCreateVnicDetails,
    CoreInternetGateway,
    CoreRouteTable,
    CoreRouteTableAttachment,
    )
from account import get_compartment_id, get_availability_domain, get_key_pair, compartment_name, profile_name

class MyStack(TerraformStack):
    def __init__(self, scope: Construct, ns: str):
        super().__init__(scope, ns)
        desired_compartment_id: str = get_compartment_id(comp_name=compartment_name)
        desired_availability_domain = get_availability_domain()
        desired_image_id = "ocid1.image.oc1.uk-london-1.aaaaaaaa7p27563e2wyhmn533gp7g3wbohrhjacsy3r5rpujyr6n6atqppuq"
        public_key = get_key_pair()

        # define resources here
        OciProvider(self, "oci",
                config_file_profile=profile_name)

        vcn = CoreVcn(self, "OCI_VCN",
                cidr_block="10.0.0.0/16",
                display_name="OCI_VCN",
                compartment_id=desired_compartment_id)

        dhcp_options = CoreDhcpOptions(self, "DHCP_OPTIONS",
                compartment_id=desired_compartment_id,
                vcn_id=vcn.id,
                options=[
                    CoreDhcpOptionsOptions(
                    type="DomainNameServer",
                    server_type="VcnLocalPlusInternet")
                ]
            )

        public_subnet = CoreSubnet(self, "PUBLIC_SUBNET",
                cidr_block="10.0.0.0/24",
                vcn_id=vcn.id,
                compartment_id=desired_compartment_id,
                display_name="public_subnet",
                dhcp_options_id=dhcp_options.id)

        internet_gateway = CoreInternetGateway(self, "INTERNET_GATEWAY",
                compartment_id=desired_compartment_id,
                vcn_id=vcn.id)

        route_table = CoreRouteTable(self, "ROUTE_TABLE",
                compartment_id=desired_compartment_id,
                vcn_id=vcn.id,
                route_rules=[
                    CoreRouteTableRouteRules(
                        network_entity_id=internet_gateway.id,
                        destination="0.0.0.0/0"
                        )
                    ])
        CoreRouteTableAttachment(self, "ROUTE_ATTACHMENT",
                subnet_id=public_subnet.id,
                route_table_id=route_table.id)


        vm = CoreInstance(self, "VM_INSTANCE",
                compartment_id=desired_compartment_id,
                shape="VM.Standard.E2.1.Micro",
                availability_domain=desired_availability_domain,
                image=desired_image_id,
                create_vnic_details=[
                    CoreInstanceCreateVnicDetails(
                        subnet_id=public_subnet.id)
                    ],
                metadata={
                    "ssh_authorized_keys": public_key
                    })

        TerraformOutput(self, "vcn",
                value=vcn.cidr_block)
        TerraformOutput(self, "public_subnet",
                value=public_subnet.cidr_block )
        TerraformOutput(self, "vm_public_ip",
                value=vm.public_ip)

app = App()
MyStack(app, "oci_terraform_cdk_python")

app.synth()
```

NOTE: `desired_image_id` in above script was obtained from https://docs.oracle.com/en-us/iaas/images/image/33995e8a-13e8-4ebe-8a27-8beae9e57043/

```bash
[root@248b335b1e23 oci_terraform_cdk_python]# ls
Pipfile  Pipfile.lock  account.py  cdktf.json  help  imports  main.py
```

#### check what will be deployed
```bash
[root@248b335b1e23 oci_terraform_cdk_python]# cdktf diff
Stack: oci_terraform_cdk_python
Resources
 + OCI_CORE_DHCP_OPTION dhcp                oci_core_dhcp_options.dhcp
   S
 + OCI_CORE_INSTANCE    instance            oci_core_instance.instance
 + OCI_CORE_INTERNET_GA InternetGateway     oci_core_internet_gateway.InternetGatew
   TEWAY                                    ay
 + OCI_CORE_ROUTE_TABLE route_table         oci_core_route_table.route_table
 + OCI_CORE_ROUTE_TABLE RouteAttachment     oci_core_route_table_attachment.RouteAt
   _ATTACHMENT                              tachment
 + OCI_CORE_SUBNET      public_subnet       oci_core_subnet.public_subnet
 + OCI_CORE_VCN         OCI_VCN             oci_core_vcn.OCI_VCN

Diff: 7 to create, 0 to update, 0 to delete.
```

#### deploy to OCI
```bash
[root@248b335b1e23 oci_terraform_cdk_python]# cdktf deploy --auto-approve
Deploying Stack: oci_terraform_cdk_python
Resources
 ✔ OCI_CORE_DHCP_OPTION dhcp                oci_core_dhcp_options.dhcp
   S
 ✔ OCI_CORE_INSTANCE    instance            oci_core_instance.instance
 ✔ OCI_CORE_INTERNET_GA InternetGateway     oci_core_internet_gateway.InternetGatew
   TEWAY                                    ay
 ✔ OCI_CORE_ROUTE_TABLE route_table         oci_core_route_table.route_table
 ✔ OCI_CORE_ROUTE_TABLE RouteAttachment     oci_core_route_table_attachment.RouteAt
   _ATTACHMENT                              tachment
 ✔ OCI_CORE_SUBNET      public_subnet       oci_core_subnet.public_subnet
 ✔ OCI_CORE_VCN         OCI_VCN             oci_core_vcn.OCI_VCN

Summary: 7 created, 0 updated, 0 destroyed.

Output: VM_public_ip = 150.230.119.194
        publicSubnet = 10.0.0.0/24
        vcn = 10.0.0.0/16
[root@248b335b1e23 oci_terraform_cdk_python]#
```

Note: The public IP of the VM is there in above output.

#### Verify deployment by doing ssh into the VM
```bash
[root@248b335b1e23 oci_terraform_cdk_python]# ssh -i keys/private.pem opc@150.230.119.194
The authenticity of host '150.230.119.194 (150.230.119.194)' can't be established.
ED25519 key fingerprint is SHA256:5UKRn4VrJLhgkK40WaNmW7O0jgdAAsRE+1vNzDwFbAQ.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '150.230.119.194' (ED25519) to the list of known hosts.
[opc@instance20210921093338 ~]$ uptime
 09:35:49 up 1 min,  1 user,  load average: 0.70, 0.25, 0.09
[opc@instance20210921093338 ~]$
```

#### Verify the idempotency
```bash
[root@248b335b1e23 oci_terraform_cdk_python]# cdktf deploy --auto-approve
No changes for Stack: oci_terraform_cdk_python
```

#### Destroy the deployment
```bash
[root@248b335b1e23 oci_terraform_cdk_python]# cdktf destroy --auto-approve
Destroying Stack: oci_terraform_cdk_python
Resources
 ✔ OCI_CORE_DHCP_OPTION dhcp                oci_core_dhcp_options.dhcp
   S
 ✔ OCI_CORE_INSTANCE    instance            oci_core_instance.instance
 ✔ OCI_CORE_INTERNET_GA InternetGateway     oci_core_internet_gateway.InternetGatew
   TEWAY                                    ay
 ✔ OCI_CORE_ROUTE_TABLE route_table         oci_core_route_table.route_table
 ✔ OCI_CORE_ROUTE_TABLE RouteAttachment     oci_core_route_table_attachment.RouteAt
   _ATTACHMENT                              tachment
 ✔ OCI_CORE_SUBNET      public_subnet       oci_core_subnet.public_subnet
 ✔ OCI_CORE_VCN         OCI_VCN             oci_core_vcn.OCI_VCN

Summary: 7 destroyed.
```

#### Repo link
https://github.com/spareslant/oci_terraform_cdk_python.git

