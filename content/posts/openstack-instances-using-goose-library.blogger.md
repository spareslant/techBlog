---
title: "Remote OpenStack Instances Using Goose library (using Nova networking)"
date: 2020-03-16T23:04:49Z
draft: true
tags: ["goose", "openstack", "Go"]
---

## Introduction:
Following program uses `goose` library to spin instances in OpenStack environment. Program will also assign a public IP to first instance.

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
 "gopkg.in/goose.v1/client"
 "gopkg.in/goose.v1/glance"
 "gopkg.in/goose.v1/identity"
 "gopkg.in/goose.v1/nova"
 "log"
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

func getImageID(clientHandle client.Client, name string) (*string, error) {
 c1 := glance.New(clientHandle)
 entities, _ := c1.ListImages()
 for _, entity := range entities {
  if entity.Name == name {
   return &entity.Id, nil
  }
 }
 return nil, errors.New("ERROR: Image " + name + " not found. Does it exist?")
}

func getFlavorID(clientHandle client.Client, name string) (*string, error) {
 c1 := nova.New(clientHandle)
 entities, _ := c1.ListFlavors()
 for _, entity := range entities {
  if entity.Name == name {
   return &entity.Id, nil
  }
 }
 return nil, errors.New("ERROR: Flavor " + name + " not found. Does it exist?")
}

func getNetworkID(clientHandle client.Client, name string) (*string, error) {
 c1 := nova.New(clientHandle)
 entities, _ := c1.ListNetworks()
 for _, entity := range entities {
  if entity.Label == name {
   return &entity.Id, nil
  }
 }
 return nil, errors.New("ERROR: Network " + name + " not found. Does it exist?")
}

func getSecurityGroupID(clientHandle client.Client, name string) (*string, error) {
 c1 := nova.New(clientHandle)
 entities, _ := c1.ListSecurityGroups()
 for _, entity := range entities {
  if entity.Name == name {
   return &entity.Id, nil
  }
 }
 return nil, errors.New("ERROR: SecurityGroup " + name + "  not found. Does it exist?")
}

func generateRandomString() string {
 b := make([]byte, 8)
 _, err := rand.Read(b)
 checkErrorAndExit(err)
 name := fmt.Sprintf("%X", b[0:4])
 return name
}

func getFreeFloatingIP(clientHandle client.Client) (*string, error) {
 c1 := nova.New(clientHandle)
 entities, _ := c1.ListFloatingIPs()
 for _, entity := range entities {
  // Make sure this floatingIP is not assigned to a VM already
  if entity.InstanceId == nil {
   return &entity.IP, nil
  }
 }
 return nil, errors.New("ERROR: FloatingIP not available. Has it been allocated to project?")
}

func getVmIdfromName(clientHandle client.Client, vmName string) (*string, error) {
 filter := nova.NewFilter()
 filter.Set(nova.FilterServer, vmName)
 filterHandle := nova.New(clientHandle)
 serverList, err := filterHandle.ListServers(filter)
 checkErrorAndContinue(err)
 for _, server := range serverList {
  if server.Name == vmName {
   return &server.Id, nil
  }
 }
 return nil, errors.New("ERROR: Instance " + vmName + " not found. Does it exist?")

}

func assignFloatingIP(clientHandle client.Client, id string, ip string) {
 fmt.Println("Assigning Floating IP to first instance. Can take a few seconds.")
 serverHandle := nova.New(clientHandle)
 for {
  details, err := serverHandle.GetServer(id)
  checkErrorAndContinue(err)
  if details.Status == "ACTIVE" {
   err := serverHandle.AddServerFloatingIP(id, ip)
   checkErrorAndExit(err)
   fmt.Println("done.")
   os.Exit(0)
  }
  time.Sleep(1000 * time.Millisecond)
  //fmt.Println(details.Status)
 }

}

func main() {

 user := flag.String("user", "admin", "OS Username")
 pass := flag.String("password", "xxxxx", "Password")
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

 creds := identity.CredentialsFromEnv()

 logger := log.New(os.Stdout, "INFO: ", log.Lshortfile)

 clientHandle := client.NewClient(creds, identity.AuthUserPass, logger)

 imageID, err := getImageID(clientHandle, *image)
 checkErrorAndExit(err)

 flavorID, err := getFlavorID(clientHandle, *flavor)
 checkErrorAndExit(err)

 networkID, err := getNetworkID(clientHandle, *network)
 checkErrorAndExit(err)

 securityGroupID, err := getSecurityGroupID(clientHandle, *securityGroup)
 checkErrorAndExit(err)

 network1 := nova.ServerNetworks{NetworkId: *networkID, FixedIp: "", PortId: ""}
 allNetworks := []nova.ServerNetworks{network1}

 securityGroup1 := nova.SecurityGroupName{Name: *securityGroupID}
 allSecurityGroups := []nova.SecurityGroupName{securityGroup1}

 noOfInstances, err := strconv.Atoi(*total)
 checkErrorAndExit(err)

 for i := 1; i <= noOfInstances; i++ {
  vmName := *hostNamePrefix + strconv.Itoa(i)
  vm := nova.RunServerOpts{Name: vmName,
   FlavorId:           *flavorID,
   ImageId:            *imageID,
   Networks:           allNetworks,
   UserData:           []byte{},
   SecurityGroupNames: allSecurityGroups}

  vmHandle := nova.New(clientHandle)
  _, err := vmHandle.RunServer(vm)
  checkErrorAndContinue(err)
  fmt.Printf("%s VM creation request sent successfully.\n Details: { flavor=%s, image=%s, network=%s, sec_group=%s }\n", vmName, *flavor, *image, *network, *securityGroup)

  fmt.Println("===================================")
 }

 vmID, err := getVmIdfromName(clientHandle, *hostNamePrefix+strconv.Itoa(1))
 checkErrorAndExit(err)

 floatingIP, err := getFreeFloatingIP(clientHandle)
 checkErrorAndExit(err)

 assignFloatingIP(clientHandle, *vmID, *floatingIP)

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

Below are the sample runs:
```bash
# ./launchOpenStackInstances
  -authurl string
        AUTH URL (default "http://10.0.2.15:5000/v2.0")
  -flavor string
        Size of VM (default "m1.tiny")
  -hostnameprefix string
        Hostname Prefix. default: randomly generated. (default "3F96E1E5")
  -image string
        Image to be used (default "cirros")
  -network string
        Network for VM (default "private")
  -password string
        Password (default "acf0d596fbb44b1d")
  -region string
        Region for VM (default "RegionOne")
  -runwithdefaultvalues
        Use this flag to use default values
  -securitygroup string
        Security Group for VM (default "default")
  -tenant string
        Tenant Name (default "admin")
  -total string
        Total number of instances (default "5")
  -user string
        OS Username (default "admin")
```

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
**Note:** B081CB981, B081CB982 and B081CB983 are VM names, that are generated dynamically. Program also tries to assign a `floatingIP` if available to the first VM in above list.

```bash
# ./launchOpenStackInstances -authurl=http://10.0.2.15:5000/v2.0 -image=cirros -user=admin -password=xxxxx -tenant=admin -flavor=m1.tiny -region=RegionOne -total=3 -hostnameprefix=mymachines
Creating VMs:
mymachines1 VM creation request sent successfully.
 Details: { flavor=m1.tiny, image=cirros, network=private, sec_group=default }
===================================
mymachines2 VM creation request sent successfully.
 Details: { flavor=m1.tiny, image=cirros, network=private, sec_group=default }
===================================
mymachines3 VM creation request sent successfully.
 Details: { flavor=m1.tiny, image=cirros, network=private, sec_group=default }
===================================
Error: ERROR: FloatingIP not available. Has it been allocated to project?
```

**Note 1:** `-hostnameprefix` flag will generate VMs with the user specified names. Take a look at above output. `mymachines1`, `mymachines2` etc. Also note that program failed to get `FloatingIP`. Because it was not available.

**Note 2:** You can run this program on your desktop machine to spin instances in a remote OpenStack environment.

**Observation 1:** `goose` library is not very polished yet. In another post I have created same program using gophercloud library which is much more leaner. 

**Observation 2:** If OpenStack installation does not have Object Storage configured or available, then goose library will not work.

**Observation 3:** This program is using Nova Networking. Which is not very flexible.

**Observation 4:**Take a look at function `assignFloatingIP`. There is an infinite loop in this function. This is required as `floatingIP` can be assigned only when VM is in `ACTIVE` state. However this trick can be avoided when `Neutron` networking is used.





