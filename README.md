# Introduction
The cloud paradigm changes the way applications are developed, architected, and operated. Improved agility, speed of development, and increased automation in operations allow enterprises to truly benefit from cloud infrastructure. 

Security is one of the main concerns when assessing migration and operating an application in the cloud. However, nowadays applications can benefit from several additional layers of security when deployed in the cloud. It is up to the cloud architect to leverage the many features offered by most IaaS providers to ensure that an application remains highly available, robust, and secure while offering a good experience to end-users. 

The instructions in the lab go through the necessary steps of implementing **two VM**, one in a Public Subnet accessible from the Internet, and the second one in a Private Subnet. Both subnets are part of a VCN and for security, they are using NSG.  

# Objective
This lab walks you to the steps needed to the below resources 
1. VCN
2. Public subnet
3. Private subnet
4. VM's in each subnet
5. Gateway
6. Routing Rules
6. NSG's

We will conncet to the VM in public subnet via the Internet Gateway and from that VM then to the VM in the private subnet

# Pre-Requisite
To perform the lab you will need the access to the following:

1. Web Browser
2. OCI Tenenacy with the right permissions
3. Putty and putty-gen for Windows user or Terminal for Mac users

# Best practise and Consideration
Keep in mind that this is a lab, so choose appropriate VM shapes. Use a Naming Convention for your resources to easily identify them. As a suggestion, you can start with your signum followed by the resource type. Please find below some examples:

> **VCN:** *caandrei-vcn-192.168.23.0/24*

> **Subnet:** *caandrei-net-192.168.23.0/28*

> **Route table:** *caandrei-rt-192.168.23.0/28*

> **Security list:** *caandrei-sl-192.168.23.0/28*

# Architecture Overview
The scenario deployed will show the access to a private VM via a bastion host. In the lab we will use relaxed security and will permit ssh access to the bastion from everywhere (0.0.0.0/0). In a production environment, this will be restricted to the authorized Public IP addresses.



# Section
- [Generate the public/private key](#generate-the-public/private-key)
- [Create the network resources](#create-the-network-resources)
- [Create the Public VM](#create-the-public-vm)
- [Create the Private VM](#create-the-private-vm)
- [Adjust the security to permit connectivity](#adjust-the-security-to-permit-conncetivity)
- [Test Connectivity](#test-connectivity)



# Generate the public/private key

## MAC/LINUX

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

## FOR WINDOWS

Open puttygen (or a similar tool), make sure that the key is RSA and the length of the key is 2048 and hit generate:

![Generate Key]{Picture1.png}
