# Lab 2: IPSec VPN

## Introduction

The cloud paradigm changes the way applications are developed, architected, and operated. Improved agility, speed of development, and increased automation in operations allow enterprises to truly benefit from cloud infrastructure.

Usually the enterprise would want to setup encrypted connection from on-prem to Oracle cloud for performing POC's and do some small migrations. Later enterprises can later setup Fastconncet, out dedicated private conncetion from on-premises data center to Oracle cloud.

## Objective

This lab walks you to the steps needed to create a VPN connection from a Linux VM to an OCI VPN Connect. On the public VM from the previous lab, we will install an opensource vpn software. In another region, we will create a VPN connection to this VM.

## Pre-requisites

You need the setup from Lab1 to continue working on this Lab

## Process Overview

- [Create an OCI IPSec Connection](#create-an-oci-ipsec-connection)
- [Install and Configure Libreswan](#install-and-configure-libreswan)
- [Setup Firewall Rules](#setup-firewall-rules)
- [Install Quagga](#install-quagga)
- [Tests](#tests)

## Create an OCI IPSec Connection

Navigate to Networking >  Overview and click the “Start VPN Wizard”.

Fill in the Name of the IPSec Connection

![VPN Wizard](images/Picture1.png)

Fill in the public IP address of the public VM created in Lab1 and select Libreswan from the Vendor drop-down. Select the Platform/Version

![VPN Wizard2](images/Picture2.png)

Fill in the CIDR space of the networks that are behind the Libreswan (192.168.23.0/24)

![VPN Wizard3](images/Picture3.png)

Fill in the name of the Tunnel1, and the BGP information. 

We will use ASN65000 and a /30 for connectivity between BGP peers: 10.10.10.0/30.

The CPE will use 10.10.10.1/30 and the oracle side will use 10.10.10.2/30

![VPN Wizard3](images/Picture4.png)

Fill in the name of the Tunnel2, and the BGP information. 

We will use ASN65000 and a /30 for connectivity between BGP peers: 10.10.10.4/30.

The CPE will use 10.10.10.5/30 and the oracle side will use 10.10.10.6/30

![VPN Wizard4](images/Picture5.png)

Specify the CIDR space for the VCN from OCI: 192.168.24.0/24

![VPN Wizard5](images/Picture6.png)

On the next screen, you will have the details of the networking artifacts that will be created.

![VPN Wizard6](images/Picture7.png)

On the next screen you will have the Monitoring options

![VPN Wizard7](images/Picture8.png)

The last screen will present the overall configuration that will be created.

![VPN Wizard8](images/Picture9.png)

Push the “Create Solution” button. You will have the progress of the provisioning:

![VPN Create1](images/Picture10.png)

![VPN Create2](images/Picture11.png)

Wait a few minutes and click on the “Open CPE Configuration Helper”.

You will see the summary of what was configured on the OCI side.

![VPN Summary](images/Picture12.png)

Push the “Create Content” button

![VPN CPE Config](images/Picture13.png)

Download the configuration

## Install and Configure Libreswan

Follow the official documentation for Libreswan [here](#https://docs.cloud.oracle.com/iaas/Content/Network/Reference/libreswanCPE.htm).

Scroll to the “Configuration Process” and start with Task 1. This will enable the routing on the Linux instance.

Login to the public VM from Lab1. Switch user to root and edit the /etc/sysctl.conf file

```bash
nano /etc/syscrl.conf
```

![Sysctl config1](images/Picture14.png)

Add the routing information

![Sysctl config2](images/Picture15.png)

Please be aware that in the documentation, the interface name of the VM is eth0. You can check the name of your interface with ifconfig command. 

In my screenshot, the interface name is ens3.

Save the config with ctrl+X and type “Y” to save

![Sysctl config3](images/Picture16.png)

Hit “Enter” and confirm the filename.

![Sysctl config4](images/Picture17.png)

Reload the routing configuration with 

```bash
sudo sysctl -p
```

Now install the Libreswan software

```bash
sudo yum install libreswan -y
```

![Libreswan Install](images/Picture19.png)

Proceed to Task 3 from the documentation and create the config file. Copy-paste the template in an editor (Notepad) and replace the variables with the information that we already have.

![VPN Wizard3](images/Picture20.png)

Notice that two things are modified from the template: the *vti* interface (I used *vti1* and *vti2*) and the ip addresses assigned to the *vti* interface (*“leftvti=”*)

Take the config from the editor and create the config file (*/etc/ipsec.d/oci-ipsec.conf*)

![IPSEC Config1](images/Picture21.png)

Follow Task 4 and create the secrets file.

The file will look like this:

![IPSEC Config2](images/Picture22.png)

Start the ipsec service 

```bash
sudo systemctl start ipsec
```

Check the status of the connection

```bash
ipsec status
```

You will notice that the tunnels are not up.

![VPN Status](images/Picture23.png)

Check the logs from the Libreswan

```bash
tail /var/log/secure | grep pluto
```

![VPN Logs](images/Picture24.png)

The following line in the log will help identify the problem

```log
oracle-tunnel-1": We cannot identify ourselves with either end of this connection.  152.67.130.140 or 140.204.35.3 are not usable
```

This means that we did not fully configure the libreswan. On ens3 interface we do not have a public IP address, so we need to adjust the config and use as left the private ip address configured on the interface and for the leftid the public ip address. Please make sure you adjust both of the tunnels.

![IPSEC Config2](images/Picture25.png)

Restart the ipsec service 

```bash
sudo systemctl restart ipsec
```

Check the status of the service

```bash
ipsec status
```

![VPN Status2](images/Picture26.png)

You can observe that the tunnels are up (IPsec SA established).

## Setup Firewall Rules

Adjust the NSG of the public VM:

![NSG Rule1](images/Picture27.png)

Add a another rule for the second ipsec peer.

![NSG Rule2](images/Picture28.png)

Push the “Add” button.
Test the connectivity over the ipsec tunnel by pinging the other side tunnel interface

![Ping](images/Picture29.png)

## Install Quagga

Check the Ipsec connection the on DRG. You will notice that the IPsec tunnel is up but the bgp connection is down.

We need to install a bgp service on the public VM.

```bash
sudo yum install quagga -y
```

Create the bgpd config file:

```bash
sudo touch /etc/quagga/bgpd.conf
```

Enable quagga daemons

```bash
systemctl start zebra
systemctl enable zebra
systemctl start bgpd
systemctl enable bgpd
```

![Quagga commands](images/Picture31.png)

Enter in the quagga console

```bash
vtysh
```

Enter in the configuration mode

```bash
conf t
```

Paste the following config:

```bash
interface vti1
ip address 10.10.10.1/30
interface vti2
ip address 10.10.10.5/30
router bgp 65000
 bgp router-id 10.10.10.1
 network 192.168.23.0/24
 neighbor 10.10.10.2 remote-as 31898
 neighbor 10.10.10.2 ebgp-multihop 255
 neighbor 10.10.10.2 timers 6 18
 neighbor 10.10.10.2 next-hop-self
 neighbor 10.10.10.6 remote-as 31898
 neighbor 10.10.10.6 ebgp-multihop 255
 neighbor 10.10.10.6 timers 6 18
 neighbor 10.10.10.6 next-hop-self
```

![Quagga config1](images/Picture32.png)

Issue CTRL+Z to exit the config mode. Check the configuration that you just pasted with 

```bash
sh run
```

![Quagga config2](images/Picture33.png)

Issue *wr* command to save the config

![Quagga config3](images/Picture34.png)

You will notice that the config file can’t be written. This can be fix by following the instruction [here](#https://bugzilla.redhat.com/show_bug.cgi?id=1204920)

![Quagga config4](images/Picture35.png)

Now return to *vtysh* and save the config

![Quagga config5](images/Picture36.png)

## Tests

Check the BGP status

```bash
show ip bgp sum
```

![BGP Status](images/Picture37.png)

Display the routes learn by bgp

```bash
sh hip route bgp
```

![BGP Routes](images/Picture38.png)

Check the status of the IPsec connection on the DRG

![VPN BGP Status](images/Picture39.png)

Both bgp sessions are UP!