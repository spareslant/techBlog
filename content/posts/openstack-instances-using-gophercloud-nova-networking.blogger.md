---
title: "Remote OpenStack instances using Gophercloud Library (using Nova Networking)"
date: 2020-03-16T22:46:24Z
draft: true
tags: ["Go", "openstack", "gophercloud", "Nova"]
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
Following program uses `gophercloud `library to spin instances in OpenStack environment.

## Requirments:
An already running OpenStack environment. You can get a free playground for OpenStack to play with at **http://trystack.org/**. At minimum you need following information.

```
Auth URL : It may look like this http://10.0.2.15:5000/v2.0
UserName : This is a user in OpenStack. default: admin
Password : This is an API password. API password can be different from user password.
Tenant   : This is more like a project/Account or Organization. A tenant can have multiple users associated with it.. default: admin
Region   : Default region is set to RegionOne.
```

In `trystack.org` this information can be found at `https://x86.trystack.org/dashboard/project/access_and_security/` and click on Download OpenStack RC file. Contents of RC file will have all the required information.

Following is the source code.
```go
package main

import (
        "crypto/rand"
        "errors"
        "flag"
        "fmt"
        "github.com/rackspace/gophercloud"
        "github.com/rackspace/gophercloud/openstack"
        "github.com/rackspace/gophercloud/openstack/compute/v2/extensions/floatingip"
        "github.com/rackspace/gophercloud/openstack/compute/v2/flavors"
        "github.com/rackspace/gophercloud/openstack/compute/v2/images"
        "github.com/rackspace/gophercloud/openstack/compute/v2/servers"
        "github.com/rackspace/gophercloud/openstack/networking/v2/networks"
        "os"
        "strconv"
        "time"
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

func getFreeFloatingIP(client *gophercloud.ServiceClient) (*string, error) {
        pager := floatingip.List(client)
        singlePage, err := pager.AllPages()
        checkErrorAndExit(err)
        floatingIPList, err := floatingip.ExtractFloatingIPs(singlePage)
        checkErrorAndExit(err)
        for _, floatingIP := range floatingIPList {
            // Check ip floatingIP is assigned to any Instance.
            if len(floatingIP.InstanceID) > 0 {
                //Do nothing
            } else {
               return &floatingIP.IP, nil
            }
        }

        return nil, errors.New("ERROR: Free FloatingIP not available.")
}

func assignFloatingIP(client *gophercloud.ServiceClient, id string, ip string) {
        fmt.Printf("Attaching floating IP with first VM. It can take a few seconds.\n")
        opts := floatingip.AssociateOpts{ServerID: id, FloatingIP: ip}

        for {
            server, err := servers.Get(client, id).Extract()
            checkErrorAndExit(err)
            if server.Status == "ACTIVE" {
            result := floatingip.AssociateInstance(client, opts)
            if result.Err != nil {
                fmt.Println(result.Err)
            } else {
                fmt.Println("Floating IP assigned successfully.")
            }
             os.Exit(0)
         }
         time.Sleep(1000 * time.Millisecond)
         //fmt.Println(details.Status)
       }


}


func main() {

        user := flag.String("user", "admin", "OS Username")
        pass := flag.String("password", "xxxx", "Password")
        authUrl := flag.String("authurl", "http://10.0.2.15:5000/v2.0", "AUTH URL")
        tenant := flag.String("tenant", "admin", "Tenant Name")
        region := flag.String("region", "RegionOne", "Region for VM")
        defaultRun := flag.Bool("runwithdefaultvalues", false, "Use this flag to use default values")
        image := flag.String("image", "cirros", "Image to be used")
        flavor := flag.String("flavor", "m1.tiny", "Size of VM")
        network := flag.String("network", "private", "Network for VM")
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

        networkID, err := networks.IDFromName(netClient, *network)
        checkErrorAndExit(err)

        /* This code is not needed.
        securityGroupID, err := getSecurityGroupID(client, *securityGroup)
        checkErrorAndExit(err)
        fmt.Println(*securityGroupID)
        */
        network1 := servers.Network{UUID: networkID, FixedIP: "", Port: ""}
        allNetworks := []servers.Network{network1}

        noOfInstances, err := strconv.Atoi(*total)
        checkErrorAndExit(err)

        for i := 1; i <= noOfInstances; i++ {
                vmName := *hostNamePrefix + strconv.Itoa(i)
                serverCreateOpts := servers.CreateOpts{Name: vmName,
                        FlavorRef:      flavorID,
                        ImageRef:       imageID,
                        Networks:       allNetworks,
                        UserData:       []byte{},
                        SecurityGroups: []string{*securityGroup}}

                _, err := servers.Create(client, serverCreateOpts).Extract()
                checkErrorAndContinue(err)
                fmt.Printf("%s VM creation request sent successfully.\n Details: { flavor=%s, image=%s, network=%s, sec_group=%s }\n", vmName, *flavor, *image, *network, *securityGroup)

                fmt.Println("===================================")
        }
        vmID, err := servers.IDFromName(client, *hostNamePrefix+strconv.Itoa(1))
        checkErrorAndExit(err)

        fmt.Printf("Obtaining free floating IP: ")
        floatingIP, err := getFreeFloatingIP(client)
        checkErrorAndExit(err)
        fmt.Printf("%s found.\n", *floatingIP)

        assignFloatingIP(client, vmID, *floatingIP)

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

* An executable with name `launchOpenStackInstances` will be there.

Example Run:
```bash
# ./launchOpenStackInstances -authurl=http://10.0.2.15:5000/v2.0 -image=cirros -user=admin -password=xxxxx -tenant=admin -flavor=m1.tiny -region=RegionOne -total=3
Creating VMs:
B081CB981 VM creation request sent successfully.
 Details: { flavor=m1.tiny, image=cirros, network=private, sec_group=default }
===================================
B081CB982 VM creation request sent successfully.
 Details: { flavor=m1.tiny, image=cirros, network=private, sec_group=default }
===================================
B081CB983 VM creation request sent successfully.
 Details: { flavor=m1.tiny, image=cirros, network=private, sec_group=default }
===================================
Assigning Floating IP to first instance. Can take a few seconds.
done.
```

**Observation 1:** This program is using `Nova Networking`.

**Observation 2:** Take a look at function `assignFloatingIP`. There is an infinite loop in this function. This is required as `floatingIP` can be assigned only when VM is in `ACTIVE `state. However this trick can be avoided when `Neutron` networking is used.



