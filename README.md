# Deploy-Node-Express-Application-kubernetes

In this tutorial, we use an Oracle Cloud Infrastructure account to set up a Kubernetes cluster. Then, we deploy a Node Express application to our cluster. 
Key tasks include how to:

Set up OpenSSL API encryption keys to access our cluster.
Set up an authentication token.
Set up a Kubernetes cluster on OCI.
Set up OCI CLI to access our cluster.
Build a Node Express application and Docker Image.
Push our image to OCIR.
Deploy our Docker application to our cluster.
Connect to our application from the internet.

![image](https://user-images.githubusercontent.com/42166489/107557121-3716b680-6bff-11eb-88c1-ee39c414ed55.png)

What we need to start:
To successfully perform this tutorial, we must have the following:

1.A paid Oracle Cloud Infrastructure account. 
https://docs.oracle.com/iaas/Content/GSG/Tasks/signingup.htm
2. SSH support:
A MacOS or Linux computer with ssh support installed.
A Windows machine with Git Bash installed. 
https://gitforwindows.org/

In this tutorial we are going to use a Mac machine.

1. Gather Required Information
Prepare the information we need from the Oracle Cloud Infrastructure Console.

Check our service limits:
Regions: minimum 2

In the top navigation bar, expand <region>. Example: US East (Ashburn) and US West (Phoenix).

Note: In our case we are going to perform tutorial in Mumbai region.
Compute instances: minimum 3
Click our profile avatar. Select Tenancy. Go to Service Limits and expand Compute.

Block Volume: minimum 50 GBs
In the Service Limits section, expand Block Volume.

Load Balancer: available
In the Service Limits section, expand Networking.

Have a compartment for our cluster.
See Create a compartment: 
(https://docs.oracle.com/iaas/Content/Identity/Tasks/managingcompartments.htm#).

- Create an authorization token:
- In the top navigation bar, click the user avatar.
- Select our username.
- Click Auth Tokens.
- Click Generate Token.
- Give it a description.
- Click Generate Token.
- Copy the token and save it.
 


Note: Make sure we save our token right after we create it. We will not have access to it later.
Find our region identifier and region key from Regions and Availability Domains. Example: us-ashburn-1 and iad for Ashburn.
https://docs.oracle.com/iaas/Content/General/Concepts/regions.htm

In our case for India West (Mumbai) it is “ap-mumbai-1”

Collect the following information and copy them into our notepad.
Auth Token: <auth-token> from step 3.
Region: <region-identifier> from step 4. Example: us-ashburn-1.
Region Key: <region-key> from step 4. Example: iad.
Tenancy name: <tenancy-name> from our user avatar.
Tenancy OCID: <tenancy-ocid> from our user avatar, go to Tenancy:<our-tenancy> and copy OCID.
Username: <user-name> from our user avatar.
User OCID: <user-ocid> from our user avatar, go to User Settings and copy OCID.

2. Create SSH Encryption Keys

Create ssh encryption keys to connect to our compute instance.

Open a terminal window:
MacOS or Linux: Open a terminal window in the directory where we want to store our keys.
Windows: Right-click on the directory where we want to store our keys and select Git Bash Here.

Issue the following OpenSSH command:
ssh-keygen -t rsa -N "" -b 2048 -C <our-ssh-key-name> -f <our-ssh-key-name>

The command generates some random text art used to generate the keys. When complete, we have two files:
The private key file: <our-ssh-key-name>
The public key file: <our-ssh-key-name>.pub
We use these files to connect to our compute instance.

This way we can generate the required encryption keys.

Or, we can use the OCI console for generating the keys.

![image](https://user-images.githubusercontent.com/42166489/107557207-501f6780-6bff-11eb-952a-0cc9c5e5262d.png)

link: https://docs.oracle.com/iaas/Content/GSG/Tasks/creatingkeys.htm

![image](https://user-images.githubusercontent.com/42166489/107557272-662d2800-6bff-11eb-8630-b2c6a016b092.png)

Create a API Key by clicking on the API Keys and download and save the keys to a secured location.

![image](https://user-images.githubusercontent.com/42166489/107557300-6fb69000-6bff-11eb-955f-462b7a352359.png)

3. Create a Virtual Cloud Network (VCN)

1. From the main landing page, select Set up a network with a wizard.

![image](https://user-images.githubusercontent.com/42166489/107557338-7b09bb80-6bff-11eb-8857-fabc4c7fe325.png)

2. In the Start VCN Wizard workflow, select VCN with Internet Connectivity and then click Start VCN Wizard .
3. In the configuration dialog, fill in the VCN Name for our VCN. Our Compartment is already set to its default value of <our-tenancy> (root).
4. In the Configure VCN and Subnets section, keep the default values for the CIDR blocks:
VCN CIDR BLOCK: 10.0.0.0/16
PUBLIC SUBNET CIDR BLOCK: 10.0.0.0/24
PRIVATE SUBNET CIDR BLOCK: 10.0.1.0/24

![image](https://user-images.githubusercontent.com/42166489/107557372-865ce700-6bff-11eb-9313-0501894c1cc5.png)

5. For DNS RESOLUTION uncheck USE DNS HOSTNAMES IN THIS VCN.Picture shows the USE DNS HOSTNAMES IN THIS VCN option is unchecked.
Click Next.
6. The Create a VCN with Internet Connectivity configuration dialog is displayed (not shown here) confirming all the values our just entered.

7. Click Create to create our VCN.
The Creating Resources dialog is displayed (not shown here) showing all VCN components being created.

8. Click View Virtual Cloud Network to view our new VCN.
Our new VCN is displayed. Now we need to add a security rule to allow HTTP connections on port 3000, the default port for our application.

9. With our new VCN displayed, click our Public subnet link.
The public subnet information is displayed with the Security Lists at the bottom of the page. There should be a link to the Default Security List for our VCN.

![image](https://user-images.githubusercontent.com/42166489/107557415-9379d600-6bff-11eb-8b64-20620f2dc3ee.png)

10. Click on the Default Security List link.
The default Ingress Rules for our VCN are displayed.

11. Click Add Ingress Rules.
An Add Ingress Rules dialog is displayed.

12. Fill in the ingress rule with the following information. Once all the data is entered, click Add Ingress Rules.
Fill in the ingress rule as follows:

Stateless: Checked
Source Type: CIDR
Source CIDR: 0.0.0.0/0
IP Protocol: TCP
Source port range: (leave-blank)
Destination Port Range: 3000
Description: VCN for applications
Once we click Add Ingress Rule, HTTP connections are allowed to our public subnet.

4.Install an Ubuntu VM

From the Oracle Cloud Infrastructure main menu, select Compute, then Instances.
From the list of instances screen, click Create Instance.
The Create Compute Instance dialog is displayed. Notice the Show Shape, Network and Storage Options should be expanded to configure the virtual machine.

Fill in the fields for the Create Compute Instance dialog with the following data:
Initial Options

Name of our Instance: <name-for-the-instance>
Image or Operating System (Click Change Image): Canonical Ubuntu 18.04
Availability Domain: <Select-an-always-free-eligible-domain>
Instance Shape: VM.Standard.E2.1.Micro: Virtual Machine, 1 core OCPU, 1 GB Memory, 0.48 Gbps network bandwidth

Configure Networking

VIRTUAL CLOUD NETWORK COMPARTMENT: <our-compartment>
VIRTUAL CLOUD NETWORK: <VCN-we-created>
SUBNET COMPARTMENT: <our-subnet-compartment>
SUBNET: <public-subnet-ou-created>
USE NETWORK SECURITY GROUPS TO CONTROL TRAFFIC: Unchecked
ASSIGN A PUBLIC IP ADDRESS: Selected/Checked
Additional Options

Boot Volume: All options Unchecked
Add SSH Keys: Add the public key file (.pub) we created in the beginning of this tutorial.
Click Create to create the instance.
Provisioning the system may take several minutes.

![image](https://user-images.githubusercontent.com/42166489/107557456-a68ca600-6bff-11eb-934e-0d2e3e32de01.png)

![image](https://user-images.githubusercontent.com/42166489/107557473-ad1b1d80-6bff-11eb-84c7-2812928fc319.png)

we have successfully created an Ubuntu Linux VM to build and test our applications.

5.Create a Local Node Express Application

Next, set up an Express framework on our Ubuntu Linux VM and then create and run a NodeJS application.

- By default, Ubuntu 18.04 does not come with NodeJS, Express, Docker, Kubernetes or virtualenv. To create our application with Express, perform the following steps:

- From the main menu select Compute, then Instances.
- Click the link to the instance we created in the previous step.
- From the Instance Details page look in the Instance Access section. Copy the public IP address the system created for us. We will use this IP address to connect to our instance.

- Open a Terminal or Command Prompt window.
- Change into the directory where we stored the ssh encryption keys we created for this tutorial.
- Connect to our VM with this ssh command.

Use the following command to set the file permissions so that only we can read the file:
chmod 400 <private_key_file_path>

Use the following SSH command to access the instance.
ssh -i <our-private-key-file> ubuntu@<x.x.x.x>






