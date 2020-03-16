---
title: "Remote OpenStack Instances using Gophercloud (using neutron networking)"
date: 2020-03-16T22:24:21Z
draft: true
tags: ["Go", "gopherCloud", "openstack", "neutron"]
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
Following program uses `gophercloud` library to spin instances in OpenStack environment. This program uses `Neutron` Networking. This program will also assign `FloatingIP` to very first instance.

## Requirments:
An already running OpenStack environment. You can get a free playground for OpenStack to play with at `http://trystack.org/`. At minimum you need following information.
```
Auth URL : It may look like this http://10.0.2.15:5000/v2.0
UserName : This is a user in OpenStack. default: admin
Password : This is an API password. API password can be different from user password.
Tenant   : This is more like a project/Account or Organization. A tenant can have multiple users associated with it.. default: admin
Region   : Default region is set to RegionOne.
```

In `trystack.org` this information can be found at **https://x86.trystack.org/dashboard/project/access_and_security/** and click on Download OpenStack RC file. Contents of RC file will have all the required information.

Following is the source code.
```go
package main

import (
        "crypto/rand"
        "flag"
        "fmt"
        "github.com/rackspace/gophercloud"
        "github.com/rackspace/gophercloud/openstack"
        "github.com/rackspace/gophercloud/openstack/compute/v2/flavors"
        "github.com/rackspace/gophercloud/openstack/compute/v2/images"
        "github.com/rackspace/gophercloud/openstack/compute/v2/servers"
        "github.com/rackspace/gophercloud/openstack/networking/v2/extensions/layer3/floatingips"
        "github.com/rackspace/gophercloud/openstack/networking/v2/networks"
        "github.com/rackspace/gophercloud/openstack/networking/v2/ports"
        "os"
        "strconv"
)

func checkErrorAndExit(err error) {
        if err != nil {
                fmt.Printf("Error: %v\n", err.Error())
                os.Exit(3)
        }
}

func checkErrorAndContinue(err error) {
        if err != nil {
                fmt.Printf("Error: %v\n", err.Error())
        }
}

func generateRandomString() string {
        b := make([]byte, 8)
        _, err := rand.Read(b)
        checkErrorAndExit(err)
        name := fmt.Sprintf("%X", b[0:4])
        return name
}

func main() {

        user := flag.String("user", "admin", "OS Username")
        pass := flag.String("password", "acf0d596fbb44b1d", "Password")
        authUrl := flag.String("authurl", "http://10.0.2.15:5000/v2.0", "AUTH URL")
        tenant := flag.String("tenant", "admin", "Tenant Name")
        region := flag.String("region", "RegionOne", "Region for VM")
        defaultRun := flag.Bool("runwithdefaultvalues", false, "Use this flag to use default values")
        image := flag.String("image", "cirros", "Image to be used")
        flavor := flag.String("flavor", "m1.tiny", "Size of VM")
        private_network := flag.String("private_network", "private", "private network for VM")
        public_network := flag.String("public_network", "public", "public network for VM")
        securityGroup := flag.String("securitygroup", "default", "Security Group for VM")

        hostNamePrefix := flag.String("hostnameprefix", generateRandomString(), "Hostname Prefix. default: randomly generated.")
        total := flag.String("total", "5", "Total number of instances")

        flag.Parse()

        os.Setenv("OS_USERNAME", *user)
        os.Setenv("OS_PASSWORD", *pass)
        os.Setenv("OS_AUTH_URL", *authUrl)
        os.Setenv("OS_TENANT_NAME", *tenant)
        os.Setenv("OS_REGION_NAME", *region)

        if flag.NFlag() == 0 {
                flag.PrintDefaults()
                os.Exit(1)
        } else if *defaultRun == true {
                fmt.Println("Creating VMs: ")
        } else {
                fmt.Println("Creating VMs: ")
        }

        authOpts, err := openstack.AuthOptionsFromEnv()
        checkErrorAndExit(err)

        provider, err := openstack.AuthenticatedClient(authOpts)
        checkErrorAndExit(err)

        otherOpts := gophercloud.EndpointOpts{Region: *region}
        client, err := openstack.NewComputeV2(provider, otherOpts)

        imageID, err := images.IDFromName(client, *image)
        checkErrorAndExit(err)

        flavorID, err := flavors.IDFromName(client, *flavor)
        checkErrorAndExit(err)

        netClient, err := openstack.NewNetworkV2(provider, gophercloud.EndpointOpts{Name: "neutron", Region: *region})
        checkErrorAndExit(err)

        privateNetworkID, err := networks.IDFromName(netClient, *private_network)
        checkErrorAndExit(err)

        publicNetworkID, err := networks.IDFromName(netClient, *public_network)
        checkErrorAndExit(err)

        portOpts := ports.CreateOpts{NetworkID: privateNetworkID,
                AdminStateUp: ports.Up}

        netport, err := ports.Create(netClient, portOpts).Extract()
        checkErrorAndExit(err)

        network1 := servers.Network{UUID: privateNetworkID}
        network2 := servers.Network{Port: netport.ID}

        allNetworks := []servers.Network{network1}
        allPortNetworks := []servers.Network{network2}

        fmt.Printf("Obtaining free floating IP: ")

        floatingIPOpts := floatingips.CreateOpts{FloatingNetworkID: publicNetworkID,
                PortID: netport.ID}

        floatingIP, err := floatingips.Create(netClient, floatingIPOpts).Extract()
        checkErrorAndExit(err)

        fmt.Printf("%s found.\n", floatingIP.FloatingIP)

        noOfInstances, err := strconv.Atoi(*total)
        checkErrorAndExit(err)

        for i := 1; i <= noOfInstances; i++ {
                networks := allNetworks
                if i == 1 {
                        networks = allPortNetworks
                } else {
                        networks = allNetworks
                }
                vmName := *hostNamePrefix + strconv.Itoa(i)
                serverCreateOpts := servers.CreateOpts{Name: vmName,
                        FlavorRef:      flavorID,
                        ImageRef:       imageID,
                        Networks:       networks,
                        UserData:       []byte{},
                        SecurityGroups: []string{*securityGroup}}

                _, err := servers.Create(client, serverCreateOpts).Extract()
                checkErrorAndContinue(err)
                fmt.Printf("%s VM creation request sent successfully.\n Details: { flavor=%s, image=%s, network=%s, sec_group=%s }\n", vmName, *flavor, *image, *private_network, *securityGroup)

                fmt.Println("===================================")
        }

}
```
Create a file `launchOpenStackInstances.go` and populate it with above contents. Use following steps on Linux and MacOS to compile/build it.

* Make sure GO Lang compiler is installed.
* Place go source code in a directory. (most likely home directory)

```bash
mkdir MYGO
export GOPATH=$HOME/MYGO
go get github.com/rackspace/gophercloud
go build launchOpenStackInstances.go
```
* An executable with name `launchOpenStackInstances` will be there .

Below is the sample run:
```bash
# ./launchOpenStackInstances.go -authurl=http://10.1.0.10:5000/v2.0 -flavor=m1.tiny -image=cirros -private_network=private_network -public_network=public_network -user=admin -password=106d84bf40fb4413 -total=2
Creating VMs:
Obtaining free floating IP: 10.1.0.159 found.
5F52A0801 VM creation request sent successfully.
 Details: { flavor=m1.tiny, image=cirros, network=private_network, sec_group=default }
===================================
5F52A0802 VM creation request sent successfully.
 Details: { flavor=m1.tiny, image=cirros, network=private_network, sec_group=default }
===================================
```
**Observation 1:** This program uses `Neutron` Networking.

**Observation 2:** Compare this program with the program using `Nova` Networking. There is no infinite loop required to assign `floatingIP` in this program.

