# Lab 1: VCN Basics

## Introduction
The cloud paradigm changes the way applications are developed, architected, and operated. Improved agility, speed of development, and increased automation in operations allow enterprises to truly benefit from cloud infrastructure.

Security is one of the main concerns when assessing migration and operating an application in the cloud. However, nowadays applications can benefit from several additional layers of security when deployed in the cloud. It is up to the cloud architect to leverage the many features offered by most IaaS providers to ensure that an application remains highly available, robust, and secure while offering a good experience to end-users.

The instructions in the lab go through the necessary steps of implementing **two VM**, one in a Public Subnet accessible from the Internet, and the second one in a Private Subnet. Both subnets are part of a VCN and for security, they are using NSG.  

## Objective

This lab walks you to the steps needed to the below resources

1. VCN
2. Public subnet
3. Private subnet
4. VM's in each subnet
5. Gateway
6. Routing Rules
7. NSG's

We will conncet to the VM in public subnet via the Internet Gateway and from that VM then to the VM in the private subnet

## Pre-Requisite

To perform the lab you will need the access to the following:

1. Web Browser
2. OCI Tenenacy with the right permissions
3. Putty and putty-gen for Windows user or Terminal for Mac users

## Best practise and Consideration

Keep in mind that this is a lab, so choose appropriate VM shapes. Use a Naming Convention for your resources to easily identify them. As a suggestion, you can start with your signum followed by the resource type. Please find below some examples:

> **VCN:** *caandrei-vcn-192.168.23.0/24*
> **Subnet:** *caandrei-net-192.168.23.0/28*
> **Route table:** *caandrei-rt-192.168.23.0/28*
> **Security list:** *caandrei-sl-192.168.23.0/28*

## Architecture Overview

The scenario deployed will show the access to a private VM via a bastion host. In the lab we will use relaxed security and will permit ssh access to the bastion from everywhere (0.0.0.0/0). In a production environment, this will be restricted to the authorized Public IP addresses.

## Section

- [Generate the public/private key](#generate-the-publicprivate-key)
- [Create the network resources](#create-the-network-resources)
- [Create the Public VM](#create-the-public-vm)
- [Create the Private VM](#create-the-private-vm)
- [Adjust the security to permit connectivity](#adjust-the-security-to-permit-connectivity)
- [Test the Connectivity](#test-the-connectivity)
- [Conclusion](#conclusion)

> ## Generate the public/private key

> ### MAC/LINUX

1. Generate ssh-keys for your machine if you don’t have one. As long as an id_rsa and id_rsa.pub key pair is present they can be reused. By default these are stored in ~/.ssh folder. Enter the following command if you are using MAC or Linux Desktop:

```bash
ssh-keygen
```

2. Make sure permissions are restricted, sometimes ssh will fail if private keys have permissive permissions.

```bash
chmod 0700 ~/.ssh  
chmod 0600 ~/.ssh/id_rsa  
chmod 0644 ~/.ssh/id_rsa.pub
```

> ### FOR WINDOWS

Open puttygen (or a similar tool), make sure that the key is RSA and the length of the key is 2048 and hit generate:

![Generate Key](images/Picture1.png)

Wait for the process to finish and save the private key (putty format and openssh format).

![Save Key](images/Picture2.png)

To save the private key in the putty format (.ppk) click on the “Save private key” button. To save the private key to openssh format, navigate to conversions and select “Export OpenSSH key”.
Keep this application open, we will use it later in the lab.

> ## Create the network resources

Open your browser and navigate to the OCI webUI. Once you login navigate to the Networking section

![Console](images/Picture3.png)

![Networking](images/Picture4.png)

Navigate to the VCN section and select Create VCN

![Create VCN](images/Picture5.png)

Put the name, select compartment and the CIDR Block. I disabled the DNS resolution inside the VCN

![VCN Form](images/Picture6.png)

Click on the “Create Subnet” button

![Create Subnet](images/Picture7.png)

Click the “Create Subnet” button and we will configure the Public Subnet.
Fill the name, select “Regional”, fill the “CIDR Block”. I will use for now the Default route table and the Default Security List.

![Public Subnet](images/Picture8.png)

Perform the same Steps to create the Private Subnet

![Private Subnet](images/Picture9.png)

The resulting 2 subnets will look like this

![Subnets](images/Picture10.png)

Navigate to the Internet Gateway section and create a new item

![Internet Gateway](images/Picture11.png)

At this point, we want to create a routing table for each subnet. Navigate at the “route Tables” section and click the “Create Route Table” button

![Route Table](images/Picture13.png)

We will create the route table for the Public Subnet

![Public RT](images/Picture14.png)

Notice that we created a default route that uses the Internet Gateway.

Create a route table for the private subnet. This table will be empty, and it will be used in a future lab.

![Private RT](images/Picture15.png)

Edit each Subnet and associate the correct route table. For example, below is a screenshot for the public subnet

![Route Association](images/Picture16.png)

> ## Create the Public VM

Navigate to the Compute > Instances section and click on the “Create Instance: Button

![Create Instance](images/Picture17.png)

Fill in the Name, choose a VM shape (considering that we are doing a test, provision a small shape). Now we reach the Networking details: select the VCN, select the subnet (the difference between the public VM and the private VM is the subnet in which they will be provisioned). For the Public VM choose “Assign a Public IP Address”.
In the puttygen that we used earlier, copy the public key and paste it in the Key section of the OCI webUI

![Copy SSH Key](images/Picture18.png)

![Paste Key](images/Picture19.png)

At this point, we have all the mandatory information and we are ready to click the “Create” button

> ## Create the Private VM

Follow the same steps and create also the Private VM

> ## Adjust the security to permit connectivity

At this step, we will adjust the security to permit ssh connection from the Internet for the Public VM and from that VM we will connect to the private VM.

Navigate to the Networking>Virtual Cloud Networks>{Your VCN}>Security Lists and click on the Default security List

Remove the ssh access

![SSH access](images/Picture20.png)

Navigate to the Networking>Virtual Cloud Networks>{Your VCN}>Network Security Groups and create a NSG for the public VM:
Fill in the Name and click next

![NSG](images/Picture21.png)

Add the following rules and click **Create**

![Rules in NSG](images/Picture22.png)

At this point, the public VM is reachable via SSH from the Internet and it is allowed to initiate any connection.

Create a second NSG for the private vm with the same rules.
After the creation you will have two NSGs

![Private NSG](images/Picture23.png)

To use them, we need to associate them with the VMs.
Navigate to Compute > Instances > {Public VM} and edit the NSG section

![Associate NSG](images/Picture24.png)

Add the configured NSG for the public VM.

Repeat the step for the private instance.

Navigate to the Networking>Virtual Cloud Networks>{Your VCN}>Network Security Groups, click on each NSG and check the VNIC section

![VNIC section](images/Picture25.png)

Please notice that the NSG is associated with the VM

> ## Test the Connectivity

Connect to the Public IP address of the public VM and create a file called training.key using your favorite Linux editor (I used nano in the screenshot):

> nano training.key

Paste the private key (the one generated at the beginning of the lab) information

![Paste Key](images/Picture26.png)

Hit **CTRL+X** to close the file. It will ask you if you want to save it, Press **Y** then press ENTER

Connect to the private VM

![SSH](images/Picture29.png)

Accept the fingerprint.
Notice that we are getting an error (file permissions for the key are too open

![Error](images/Picture30.png)

Adjust the security for the key:

```bash
chmod 600 training.key
```

Connect again

![SSH again](images/Picture31.png)

Notice the change in the hostname prompt. Now you are connected to the private VM

> ## Conclusion

By Completing this lab you should have learned:

- Create a VCN
- Create Subnets
- Create routing tables
- Adjust Network Security to have connectivity to a VM.
- Create a VM in either private or public subnet