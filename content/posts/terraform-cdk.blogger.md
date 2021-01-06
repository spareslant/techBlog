---
title: "Terraform Cdk"
date: 2020-12-28T17:54:12Z
draft: true
tags: ["terraform", "aws", "cdk", "typescript", "node.js"]
---

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
We will be exploring the `terraform-cdk` toolkit in this article. We shall be using an AWS account to spin up a VM. We shall be doing following.

* Create A brand New VPC (Not touching existing default VPC)
* Create 3 subnets ( 1 public, 2 private)
* A security Group to allow incoming SSH traffic and allow all outgoing.
* A key-pair for ssh
* An additional internet-gateway in AWS (It is needed to get internet connectity to new VPC.)
* A new VM created in above mentioned VPC, and public subnet having internet connectivity and allows ssh connection from internet.

**Note1:** We shall be using `typescript` for terraform-cdk.

**Note2:** We need to create an additional Internet-Gateway because, existing internet-gateway is attached to default VPC in the account, and an Internet-gateway cannot be attched to more than one VPC.


## Prepare environment.
We shall be doing everything in a docker image. I am using fedora-33.

### Start fedora-33 in interactive mode.
```bash
$ docker run -it fedora:33 /bin/bash

Unable to find image 'fedora:33' locally
33: Pulling from library/fedora
ae7b613df528: Pull complete
Digest: sha256:aa889c59fc048b597dcfab40898ee3fcaad9ed61caf12bcfef44493ee670e9df
Status: Downloaded newer image for fedora:33
[root@72b337cb65e6 /]#
```

### Install NVM (node version manager) inside docker container.
```bash
[root@72b337cb65e6 ~]# curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 13633  100 13633    0     0  34082      0 --:--:-- --:--:-- --:--:-- 34082
=> Downloading nvm as script to '/root/.nvm'

=> Appending nvm source string to /root/.bashrc
=> Appending bash_completion source string to /root/.bashrc
=> Close and reopen your terminal to start using nvm or run the following to use it now:

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

### Enable NVM in current shell.
```bash
[root@f159c206739f /]# source /root/.bashrc
```

### Install latest stable node.
```bash
[root@72b337cb65e6 ~]# nvm install --lts
Installing latest LTS version.
Downloading and installing node v14.15.3...
Downloading https://nodejs.org/dist/v14.15.3/node-v14.15.3-linux-x64.tar.gz...
##################################################################################################################################################################### 100.0%
Computing checksum with sha256sum
Checksums matched!
Now using node v14.15.3 (npm v6.14.9)
Creating default alias: default -> lts/* (-> v14.15.3 *)
```

### Install VIM, unzip, less
```bash
[root@f159c206739f ~]# dnf install vim unzip less openssh-clients -y
```

### Install aws-cli
```bash
[root@f159c206739f /]# curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
[root@f159c206739f /]# unzip awscliv2.zip
[root@f159c206739f /]# sudo ./aws/install
```

### configure AWS account
```bash
[root@f159c206739f /]# aws configure --profile personal
AWS Access Key ID [None]: XXXXXXXXXXXXX
AWS Secret Access Key [None]: YYYYYYYYYYYYYYYYYYYY
Default region name [None]: <Press enter or use your AWS region. (We will be defining region in the code also.)>
Default output format [None]: json
```

### Verify the connectivity to AWS apis
```bash
[root@f159c206739f /]# aws iam list-users --profile personal

{
    "Users": [
        {
            "Path": "/",
            "UserName": "<YourAdminUserHere>",
            "UserId": "XXXXXXXXXXXXXXXXX",
            "Arn": "arn:aws:iam::99999999999:user/<YourAdminUserHere>",
            "CreateDate": "Createtion date and time"
        }
    ]
}
```

### Install terraform binary
```bash
[root@f159c206739f /]# dnf install -y dnf-plugins-core
[root@f159c206739f /]# dnf config-manager --add-repo https://rpm.releases.hashicorp.com/fedora/hashicorp.repo
[root@f159c206739f /]# dnf -y install terraform
```

### Install `terraform-cdk` kit (latest development version)

```bash
[root@f159c206739f /]# mkdir deployInternetFacingVM
[root@f159c206739f /]# cd deployInternetFacingVM


[root@f159c206739f deployInternetFacingVM]# npm install --global cdktf-cli@next
/root/.nvm/versions/node/v14.15.3/bin/cdktf -> /root/.nvm/versions/node/v14.15.3/lib/node_modules/cdktf-cli/bin/cdktf
npm WARN eslint-plugin-react@7.21.5 requires a peer of eslint@^3 || ^4 || ^5 || ^6 || ^7 but none is installed. You must install peer dependencies yourself.

+ cdktf-cli@0.0.19-pre.153
added 228 packages from 145 contributors in 68.454s
[root@f159c206739f deployInternetFacingVM]#
```

## Start coding.

### Create a project with `typescript` tempplate
```bash
[root@f159c206739f deployInternetFacingVM]# cdktf init --template="typescript" --local

Newer version of Terraform CDK is available [0.0.19] - Upgrade recommended
Note: By supplying '--local' option you have chosen local storage mode for storing the state of your stack.
This means that your Terraform state file will be stored locally on disk in a file 'terraform.tfstate' in the root of your project.

We will now set up the project. Please enter the details for your project.
If you want to exit, press ^C.

Project Name: (default: 'deployInternetFacingVM')
Project Description: (default: 'A simple getting started project for cdktf.')
npm notice created a lockfile as package-lock.json. You should commit this file.
+ constructs@3.2.82
+ cdktf@0.0.19-pre.153
added 4 packages from 4 contributors and audited 4 packages in 2.601s
found 0 vulnerabilities

npm WARN eslint-plugin-react@7.21.5 requires a peer of eslint@^3 || ^4 || ^5 || ^6 || ^7 but none is installed. You must install peer dependencies yourself.

+ @types/node@14.14.16
+ cdktf-cli@0.0.19-pre.153
+ typescript@4.1.3
added 226 packages from 144 contributors and audited 230 packages in 25.81s

57 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities


> deployInternetFacingVM@1.0.0 build /deployInternetFacingVM
> cdktf get && tsc

Generated typescript constructs in the output directory: .gen
========================================================================================================

  Your cdktf typescript project is ready!
  ......
  .....
  ......
  <output snipped>
```

### output files in the directory
```bash
[root@f159c206739f deployInternetFacingVM]# ls
cdktf.json  help  main.d.ts  main.js  main.ts  node_modules  package-lock.json  package.json  tsconfig.json
```

### download relevant provider
```bash
[root@f159c206739f deployInternetFacingVM]# cdktf get

Generated typescript constructs in the output directory: .gen
```

## install sshpk module and its typescript definitions.
```bash
[root@f159c206739f deployInternetFacingVM]# npm install sshpk @types/sshpk

npm WARN eslint-plugin-react@7.21.5 requires a peer of eslint@^3 || ^4 || ^5 || ^6 || ^7 but none is installed. You must install peer dependencies yourself.

+ sshpk@1.16.1
added 10 packages from 17 contributors and audited 240 packages in 5.885s

57 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
```

### Modify `main.ts` with following contents
```Typescript
import { Construct } from 'constructs';
import { App, TerraformStack, TerraformOutput } from 'cdktf';
import { AwsProvider, Instance, Vpc, KeyPair, Subnet, SecurityGroup, InternetGateway, DefaultRouteTable } from './.gen/providers/aws'
import { sshPrivateKey, sshPublicKey } from './generatSSHkeys'

class MyStack extends TerraformStack {
  constructor(scope: Construct, name: string) {
    super(scope, name);

    // define resources here
    new AwsProvider(this, 'aws', {
      region: 'eu-west-2',
      profile: 'personal'
    });

    const myKey = new KeyPair(this, 'myKeyPair', {
      keyName: "myKey",
      publicKey: sshPublicKey
    });

    const vpc = new Vpc(this, 'myVpc', {
      cidrBlock: '10.0.0.0/16',
    });

    const myGateway = new InternetGateway(this, 'myGateway', {
      vpcId: vpc.id
    });

    new DefaultRouteTable(this, 'add_default_route', {
      defaultRouteTableId: vpc.defaultRouteTableId,
      route: [
        {
          cidrBlock: '0.0.0.0/0',
          gatewayId: myGateway.id,
          egressOnlyGatewayId: '',
          instanceId: '',
          ipv6CidrBlock: '',
          natGatewayId: '',
          networkInterfaceId: '',
          transitGatewayId: '',
          vpcPeeringConnectionId: ''
        }
      ]
    });

    const publicSubnet = new Subnet(this, 'publicSubnet', {
      vpcId: vpc.id,
      cidrBlock: '10.0.1.0/24',
      mapPublicIpOnLaunch: true
    });
    new Subnet(this, 'privateSubnet1', {
      vpcId: vpc.id,
      cidrBlock: '10.0.2.0/24',
    });
    new Subnet(this, 'privateSubnet2', {
      vpcId: vpc.id,
      cidrBlock: '10.0.3.0/24',
    });

    const allowIncomingSSH = new SecurityGroup(this, 'allowIncomingSSH', {
      name: "allowIncomingSSH",
      vpcId: vpc.id,
      ingress: [
        {
          toPort: 0,
          cidrBlocks: ['0.0.0.0/0'],
          fromPort: 0,
          protocol: "-1",
          securityGroups: [],
          description: "allow ssh",
          ipv6CidrBlocks: [],
          prefixListIds: [],
          selfAttribute: false
        }
      ],
      egress: [
        {
          fromPort: 0,
          toPort: 0,
          cidrBlocks: ['0.0.0.0/0'],
          protocol: "-1",
          securityGroups: [],
          description: "allow all outgoing",
          ipv6CidrBlocks: [],
          prefixListIds: [],
          selfAttribute: false
        }
      ]
    });

    const instance = new Instance(this, 'myVmInstance', {
      instanceType: 't2.micro',
      ami: 'ami-0e9ae639e4d979a9f',
      keyName: myKey.keyName,
      subnetId: publicSubnet.id,
      vpcSecurityGroupIds: [
        allowIncomingSSH.id
      ]
    });

    new TerraformOutput(this, 'public_ip', {
      value: instance.publicIp
    })
    new TerraformOutput(this, 'privateKey', {
      value: sshPrivateKey
    })

  }
}

const app = new App();
new MyStack(app, 'deployInternetFacingVM');
app.synth();
```

### create a new file `generatSSHkeys.ts` with following contents.
This file will create openssh key-pair only once.
```Typescript
import { generateKeyPairSync } from 'crypto';
import { parseKey } from 'sshpk'
import * as fs from 'fs';
import * as path from 'path';


const sshKeysDir = path.join(__dirname, 'protected', 'sshKeys')
const privateKeyFile = path.join(sshKeysDir, 'privateKey.pem')
const publicKeyFile = path.join(sshKeysDir, 'publicKey.pem')
fs.mkdirSync(path.join(__dirname, 'protected', 'sshKeys'), { recursive: true});

function getKeyPair() {
  const { publicKey, privateKey} = generateKeyPairSync('rsa', {
    modulusLength: 2048,
    publicKeyEncoding: {
      format: 'pem',
      type: 'pkcs1'
    },
    privateKeyEncoding: {
      format: 'pem',
      type: 'pkcs1'
    }
  });
  return {publicKey, privateKey}
};

let sshPrivateKey = '';
let sshPublicKey = '';
if (fs.existsSync(privateKeyFile) && fs.existsSync(publicKeyFile)) {
  sshPrivateKey = fs.readFileSync(privateKeyFile).toString()
  sshPublicKey = fs.readFileSync(publicKeyFile).toString()
} else {
  let {publicKey, privateKey} = getKeyPair();
  sshPublicKey = parseKey(publicKey, 'pem').toString('ssh');
  sshPrivateKey = privateKey;
  fs.writeFileSync(privateKeyFile, sshPrivateKey);
  fs.writeFileSync(publicKeyFile, sshPublicKey)
}

export {sshPublicKey, sshPrivateKey};
```

### with the addition of new file, directory contents now look like this.
```bash
[root@f159c206739f deployInternetFacingVM]# ls
cdktf.json  generatSSHkeys.ts  help  main.d.ts  main.js  main.ts  node_modules  package-lock.json  package.json  tsconfig.json
```

### check what is being deployed
```bash
[root@f159c206739f deployInternetFacingVM]# cdktf diff
Stack: deployInternetFacingVM
Resources
 + AWS_DEFAULT_ROUTE_TA add_default_route   aws_default_route_table.add_default_route
 + AWS_INSTANCE         myVmInstance        aws_instance.myVmInstance
 + AWS_INTERNET_GATEWAY myGateway           aws_internet_gateway.myGateway
 + AWS_KEY_PAIR         myKeyPair           aws_key_pair.myKeyPair
 + AWS_SECURITY_GROUP   allowIncomingSSH    aws_security_group.allowIncomingSSH
 + AWS_SUBNET           privateSubnet1      aws_subnet.privateSubnet1
 + AWS_SUBNET           privateSubnet2      aws_subnet.privateSubnet2
 + AWS_SUBNET           publicSubnet        aws_subnet.publicSubnet
 + AWS_VPC              myVpc               aws_vpc.myVpc

Diff: 9 to create, 0 to update, 0 to delete.
```

### deploy to AWS
```bash
[root@f159c206739f deployInternetFacingVM]# cdktf deploy

Deploying Stack: deployInternetFacingVM
Resources
 ✔ AWS_DEFAULT_ROUTE_TA add_default_route   aws_default_route_table.add_default_route
 ✔ AWS_INSTANCE         myVmInstance        aws_instance.myVmInstance
 ✔ AWS_INTERNET_GATEWAY myGateway           aws_internet_gateway.myGateway
 ✔ AWS_KEY_PAIR         myKeyPair           aws_key_pair.myKeyPair
 ✔ AWS_SECURITY_GROUP   allowIncomingSSH    aws_security_group.allowIncomingSSH
 ✔ AWS_SUBNET           privateSubnet1      aws_subnet.privateSubnet1
 ✔ AWS_SUBNET           privateSubnet2      aws_subnet.privateSubnet2
 ✔ AWS_SUBNET           publicSubnet        aws_subnet.publicSubnet
 ✔ AWS_VPC              myVpc               aws_vpc.myVpc

Summary: 9 created, 0 updated, 0 destroyed.

Output: privateKey = -----BEGIN RSA PRIVATE KEY-----
        <Private key contents>
        -----END RSA PRIVATE KEY-----

        public_ip = <public ip>
[root@f159c206739f deployInternetFacingVM]#
```

### Verify deployment by ssh into the vm
```bash
[root@f159c206739f deployInternetFacingVM]# chmod 400 protected/sshKeys/privateKey.pem
[root@f159c206739f deployInternetFacingVM]# ssh -i protected/sshKeys/privateKey.pem ubuntu@<public_ip_displayed_above>
```
**Note:** Contents of `protected/sshKeys/privateKey.pem` is displayed above in `cdktf deploy` output as well.

### Verify the idempotency
```bash
[root@f159c206739f deployInternetFacingVM]# cdktf deploy
No changes for Stack: deployInternetFacingVM
```

### Destroy the deployment
```bash
[root@f159c206739f deployInternetFacingVM]# cdktf destroy
Destroying Stack: deployInternetFacingVM
Resources
 ✔ AWS_DEFAULT_ROUTE_TA add_default_route   aws_default_route_table.add_default_route
 ✔ AWS_INSTANCE         myVmInstance        aws_instance.myVmInstance
 ✔ AWS_INTERNET_GATEWAY myGateway           aws_internet_gateway.myGateway
 ✔ AWS_KEY_PAIR         myKeyPair           aws_key_pair.myKeyPair
 ✔ AWS_SECURITY_GROUP   allowIncomingSSH    aws_security_group.allowIncomingSSH
 ✔ AWS_SUBNET           privateSubnet1      aws_subnet.privateSubnet1
 ✔ AWS_SUBNET           privateSubnet2      aws_subnet.privateSubnet2
 ✔ AWS_SUBNET           publicSubnet        aws_subnet.publicSubnet
 ✔ AWS_VPC              myVpc               aws_vpc.myVpc

Summary: 9 destroyed.
```

#### GitHub Repo Link:
https://github.com/spareslant/terraform_cdk_deployInternetFacingVM