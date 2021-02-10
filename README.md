# Deploy a Node Express Application - using Kubernetes

In this tutorial, we use an Oracle Cloud Infrastructure account to set up a Kubernetes cluster. Then, we deploy a Node Express application to our cluster. 
Key tasks include how to:

- Set up OpenSSL API encryption keys to access our cluster.
- Set up an authentication token.
- Set up a Kubernetes cluster on OCI.
- Set up OCI CLI to access our cluster.
- Build a Node Express application and Docker Image.
- Push our image to OCIR.
- Deploy our Docker application to our cluster.
- Connect to our application from the internet.

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

Terminal Commands:

![image](https://user-images.githubusercontent.com/42166489/107557761-04b98900-6c00-11eb-9698-2aff5bee87c0.png)

Since we identified our public key when we created the VM, this command should log we into our VM. we can now issue sudo commands to install and start our server.

- Update firewall settings.
The Ubuntu firewall is disabled by default. However, it is still necessary to update our iptables configuration to allow HTTP traffic. Execute the following commands to update iptables.
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 3000 -j ACCEPT

sudo netfilter-persistent save

These commands add a rule to allow HTTP traffic through port 3000 and saves the changes to the iptables configuration files.

Install NodeJS version 8.10+ and express:

sudo apt update
sudo apt install nodejs
sudo apt install node-express-generator
express --version

Create a directory for our application and type the following commands. 
mkdir node-hello-app
cd node-hello-app
vi app.js

In the file, input the following text and save the file:
const express = require('express')
const app = express()
app.get('/', function (req, res) {
  res.send('Hello World!')
})
app.listen(3000, function() {
  console.log('Hello World app listening on port 3000!');
})

Run the NodeJS program:
node app.js

![image](https://user-images.githubusercontent.com/42166489/107557797-113de180-6c00-11eb-896f-7dd6f85cd35d.png)

Test the application using the command line or a browser:
To test with curl, from a new terminal, connect to our Ubuntu VM with our SSH keys, and then in the command line enter: curl -X GET http://localhost:3000
From a browser, connect to the public IP address assigned to our VM: http://<x.x.x.x>:3000
We should see Hello World! on our VM, or in our browser.

![image](https://user-images.githubusercontent.com/42166489/107557855-1c910d00-6c00-11eb-9208-dff96e5446fa.png)

We have successfully created a local NodeJS application in an Express framework, on our Oracle Cloud Infrastructure VM.

6. Build and Push the Node Express Docker Image

Next, create a Docker image on our Ubuntu Linux VM and then push it to Oracle Cloud Infrastructure Registry.

Package the Application

Package our application and then point to the package in our Dockerfile.
First, make sure we are in the node-hello-app directory.

sudo apt install npm
npm --version

Create a package.json file:
vi package.json

In the file, input the following text, update the optional author and repository fields and then save the file:
{
  "name": "node-hello-app",
  "version": "1.0.0",
  "description": "Node Express Hello application",
  "author": "Example User <username@example.com>",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "repository": {
  "type": "git",
  "url": "git://github.com/username/repository.git"
  },
  "dependencies": {
    "express": "^4.0.0"
  },
  "license": "UPL-1.0"
}

Install the package with npm.
npm install

Install Docker on our Oracle Linux VM.

Install Docker 18.0.6+ on our VM
sudo snap install docker
docker --version


Build a Docker image for our application.

Create a file named Dockerfile
vi Dockerfile

In the file, input the following text and save the file:
FROM node:12.18.1
WORKDIR /app
COPY app.js .
COPY package.json .
RUN npm install
EXPOSE 3000
CMD [ "node", "app.js" ]

Build a Docker image:
sudo docker build -t node-hello-app .

We should get a message of success.

![image](https://user-images.githubusercontent.com/42166489/107557898-2b77bf80-6c00-11eb-9689-6a983b0d07a3.png)

Run the Docker image: sudo docker run --rm -p 3000:3000 node-hello-app:latest
Test the application using the command line or a browser:
To test with curl, in a terminal, enter curl localhost:3000
From a browser, connect to the public IP address assigned to our VM: http://<x.x.x.x>:3000.

Stop the Hello world application.
From another terminal, run sudo docker ps. Find the container id for our application and then run sudo docker stop <node-hello-app-container-id>.

![image](https://user-images.githubusercontent.com/42166489/107557934-36325480-6c00-11eb-8158-5a80b505bf90.png)

Push our Docker image to OCIR
With our Docker image created, now we need to push it to OCIR.

Open a terminal window.
Use <region-key> from the gather information section to log Docker into OCIR

sudo docker login <region-key>.ocir.io

We are prompted for our login name and password.

Username: <tenancy-name>/<user-name> -> bmdrgwy1wsjh/oracleidentitycloudservice/Saikat
Password: <auth-token>

Note: 
<tenancy-name> is the registry name in our case. “bmdrgwy1wsjh”
<user-name> For a federated user, use the username like this: oracleidentitycloudservice/Saikat
<auth-token> This was created earlier in the beginning

![image](https://user-images.githubusercontent.com/42166489/107557976-45190700-6c00-11eb-9b13-3df99c0b7785.png)

![image](https://user-images.githubusercontent.com/42166489/107557983-49452480-6c00-11eb-9e00-fdf2543e06d7.png)

List our local Docker images.
sudo docker images

The Docker images on our system are displayed. Identify the image we created in the last section: node-hello-app

Before we push our local image to OCIR, we must reference it with a new name that includes the path to OCIR. Then we can push the image with the new name to OCIR. Use the Docker tag command to create a shortcut to our image using the new name:

sudo docker tag <our-local-image> <registry-image>

For example, a short version of our username is often used. "John Doe" might use: sudo docker tag node-hello-app iad.ocir.io/<tenancy-name>/johnd/node-hello-app.

I am running it like this:

ubuntu@Instance-NodeExpress:~/node-hello-app$ sudo docker tag node-hello-app bom.ocir.io/bmdrgwy1wsjh/saikat/node-hello-app

Check our Docker images to see if the reference has been created.
sudo docker images

ubuntu@Instance-NodeExpress:~/node-hello-app$ sudo docker images
REPOSITORY                                       TAG                 IMAGE ID            CREATED             SIZE
bom.ocir.io/bmdrgwy1wsjh/saikat/node-hello-app   latest              ada86c45328c        19 hours ago        921MB
node-hello-app                                   latest              ada86c45328c        19 hours ago        921MB

Push the image to OCIR:
sudo docker push bom.ocir.io/bmdrgwy1wsjh/saikat/node-hello-app:latest

![image](https://user-images.githubusercontent.com/42166489/107558184-7db8e080-6c00-11eb-9d14-d64b8d604914.png)

View the OCIR Repository we created

![image](https://user-images.githubusercontent.com/42166489/107558218-88737580-6c00-11eb-812f-128479da5def.png)

OCIR registry:

![image](https://user-images.githubusercontent.com/42166489/107558264-932e0a80-6c00-11eb-8439-761cb00ee4bc.png)

Congratulations! We created a Node Express Docker image. Now we can create a Kubernetes cluster and deploy this image to the cluster.

7.Create a Kubernetes Cluster

Set up the Kubernetes cluster we will deploy our application to. We will use a wizard to set up our first cluster.

From the OCI main menu select Developer Services then Container Clusters.
Click Create Cluster.
Select Quick Create.
Click Launch Workflow.
The Cluster Creation dialog is displayed.

Fill in the following information:
Name: <our-cluster-name>
Compartment: <our-compartment-name>
Kubernetes Version: <take-default>
Visibility Type: <Private>
Shape: VM.Standard.E2.1
Number of Nodes: 3
Add Ons: <none-selected>
Click Next.
All our choices are displayed. Review them to make sure everything is configurated correctly.

Click Create Cluster.
The services set up for our cluster are displayed.
Click Close.

our have successfully created a Kubernetes cluster.

8.Set up OCI Command Line Interface

We can use the OCI Command Line Interface (CLI) to push our application to Registry and then pull and deploy it with Container Engine for Kubernetes. In this step, we install and set up the CLI to run on the VM that we are using as our local machine.



Install virtualenv:
With a virtual environment, we can manage dependencies for our project. Every project can be in its own virtual environment to host independent groups of Python libraries. our will use virtualenv for the CLI.

sudo apt install python3-venv

Install a virtual environment wrapper.
sudo apt install python3-pip
sudo pip3 install virtualenvwrapper

Set up our virtual environment wrapper in .bashrc.
Update the file:

sudo vi .bashrc

In the file, append the following text and save the file:
# set up Python env
export WORKON_HOME=~/envs
export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
export VIRTUALENVWRAPPER_VIRTUALENV_ARGS=' -p /usr/bin/python3 '
source /usr/local/bin/virtualenvwrapper.sh

Activate the above commands in the current window.
source ~/.bashrc

Start a virtual environment.
mkvirtualenv cli-app

We should see something like: (cli-app) ubuntu@<ubuntu-instance-name>:~$

Install OCI CLI .
pip3 install oci-cli

Test the installation:
oci -v

Set up the OCI CLI config file:
oci setup config

Enter basic information: (Get the answers from "Gather Required Information" step.)

Location for our config [$HOME/.oci/config]: <take-default>
User OCID: <user-ocid>
Tenancy OCID: <tenancy-ocid>
Region (e.g. us-ashburn-1): <region-identifier>
Set up our OpenSSL API encryption keys:

Generate a new API Signing RSA key pair? [Y/n]: Y
Directory for our keys [$HOME/.oci]: <take-default>
Name for our key [oci_api_key] <take-default>

![image](https://user-images.githubusercontent.com/42166489/107558332-a640da80-6c00-11eb-9404-8c60eacac341.png)

![image](https://user-images.githubusercontent.com/42166489/107558366-b0fb6f80-6c00-11eb-91a5-022e9bc5728c.png)

![image](https://user-images.githubusercontent.com/42166489/107558385-b48ef680-6c00-11eb-8c00-6dbd5111afdd.png)

Activate the cli-app environment:
workon cli-app

Copy the public key. In the terminal enter:
cat $HOME/.oci/oci_api_key_public.pem

Add the public key to our user account.
From our user avatar, go to User Settings.
Click Add Public Key.
Select Paste Public Keys.
Paste value from previous step, including the lines with BEGIN PUBLIC KEY and END PUBLIC KEY
Click Add.










