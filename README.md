
Getting Started on Azure!
=========================


The following is a quick start guide to creating and accessing a Linux based Virtual Machine on the Azure Cloud.


#### <i class="icon-file"></i> Install azure-cli
Azure provides a Command Line Interface into the Azure Cloud ("azure-cli") to define and manage resources up there. While the Azure Portal provides easy to use menu-driven interfaces, a CLI allows scripts to be written and automated. So, it is important and relatively easy to use and automate. While there are Windows Powershell Commands and Linux Commands available, I have chose to use a Node.js azure-cli library. This means no matter what platform you are connecting from (that runs Node.js), you can use a common command set on top of Node rather than using platform-specific ones. If you are not familiar with Node.js, think of it as what a JVM is to Java - it is a runtime for Javascript (plus a lot of other platform stuff).

So with that in mind, the first step is to install Node. So once you download and follow the instructions [here](https://nodejs.org/download/) for your platform, you should be able to use NPM or the Node Package Manager to install the azure-cli node module. Make sure NPM is installed by checking with the directions [here](https://docs.npmjs.com/getting-started/installing-node). Okay now install the azure-cli package. I am running this from Linux and in order to make the installation global, it requires root authority so I am using the sudo command. You will need to run this with Administrator privileges on Windows.

```console
[vagrant@azure ~]$ sudo npm install azure-cli -g
```
If you have Virtualbox and Vagrant installed you can clone this git repo and follow along this (Linux-based command set) using a vm that is created using the Vagrantfile that cloned repo. So this would be done before the npm install.

```console
$ cd [repo-location]
$ vagrant up
```

Once you install node and the azure-cli, you should have the "azure" command available to you from a console. If not, you may need to back up a bit and try again.

#### <i class="icon-file"></i>Get your azure account credentials
The details are here [Azure CLI](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-command-line-tools/)
but you need to have your microsoft account information if you are using a trial license or using the usage provided by a MSDN subscription, then you will have to obtain those credentials first.
The easiest way for this to work is to sign in is to connect with your Office 365 Account first and then go to through the Portal. The Portal is located here [https://portal.azure.com](https://portal.azure.com/). The portal is organized around cloud resources, so use the Browse All links. Scroll down on the resources listing and find Virtual Machines (classic) and go in there. You should see the vm you create using the $AZURE_VM_MACHINE_NAME value with the azure-cli command 'azure vm create...'.


```console
vagrant@azure ~]$ azure account download
info:    Executing command account download
info:    Launching browser to http://go.microsoft.com/fwlink/?LinkId=254432
help:    Save the downloaded file, then execute the command
help:      account import <file>
info:    account download command OK
```

You should get some type of file downloaded that ends in .publishsettings. That is the acount file that needs to be registered with the azure-cli. So import the credentials. In this case i renamed them to azure-account.publishsettings and put them in my /vagrant folder
```console
vagrant@azure ~]$ azure account import /vagrant/azure-account.publishsettings
```

Now you should be able to see your account info by typing the following azure-cli command.
```console
[vagrant@azure ~]$ azure account list
```

(Optional) Install Docker on the host OS. If you plan on creating a virual machine capalbe of being a Docker container, then you should also install Docker on the machine that you are running the azure-cli from. Again, these are host-platfrom specific for Linux (CentOS) commands. Follow the directions for your platform [here](https://github.com/petergdoyle/azureupandrunning.git).
```console
[vagrant@azure ~]$ sudo yum install docker.io
[vagrant@azure ~]$ curl -sSL https://get.docker.com/ | sh
[vagrant@azure ~]$ sudo curl -sSL https://get.docker.com/ | sh
[vagrant@azure ~]$ sudo yum -y update
[vagrant@azure ~]$ sudo usermod -aG docker <username>
[vagrant@azure ~]$ sudo service docker start
[vagrant@azure ~]$ sudo docker run hello-world
```

Now find a VM image on Azure using the azure-cli (in this case, a centos distro) that you will use to specify when you create a vm. In this case I am looking for a CentOS-71 image. The same command without the grep filter will show images for all the os types supported on Azure.
```console
[vagrant@azure ~]$ azure vm image list | grep CentOS-71 |less
data:    5112500ae3b842c8b9c604889f8753c3__OpenLogic-CentOS-70-20150605                                                                    Public    Linux    OpenLogic
data:    5112500ae3b842c8b9c604889f8753c3__OpenLogic-CentOS-71-20150410                                                                    Public    Linux    OpenLogic
data:    5112500ae3b842c8b9c604889f8753c3__OpenLogic-CentOS-71-20150605                                                                    Public    Linux    OpenLogic
data:    5112500ae3b842c8b9c604889f8753c3__OpenLogic-CentOS-71-20150731                                                                    Public    Linux    OpenLogic  
```

Pick one of those and then create the necessary X:509 certificate

The current version of the Azure Management Portal only accepts SSH public keys that are encapsulated in an X509 certificate. ***SO ONCE YOU GENERATE THESE KEYS, YOU HAVE TO KEEP THEM IN A SAFE PLACE OR YOU MAY NOT BE ABLE TO CONNECT TO YOUR VM!***

To SSH into your azure vm you need to generate a pair of public and private keys execute the following command. This will end up generating a rsa_key and rsa_key.pub public and private key in the .ssh folder under your home directory. If you already have generated rsa keys, then skip this step.
```console
[vagrant@azure ~]$ ssh-keygen -t rsa -b 2048
```

OpenSSH private keys are directly readable by the openssl utility. The following command will take an existing SSH private key
(id_rsa in the example below) and create the .pem public key that is needed for Azure. Note you may want to consider a naming strategy for these certs if you plan on creating more than one.
```console
[vagrant@azure ~]$ openssl req -x509 -key ~/.ssh/id_rsa -nodes -days 365 -newkey rsa:2048 -out myAzureMachine.pem
```


Create a VM on Azure using the azure-cli. Note I am creating a machine that is intended to be used as a remote Docker container and as such will provisioned differently by Azure. So if you want just a regular vm, drop the docker directive on the azure command. ***AGAIN NOTE THAT WE ARE USING A no-ssh-password DIRECTIVE, SO YOU NEED THOSE SSH KEYS YOU GENERATED TO CONNECT***
```console
[vagrant@azure ~]$ REGION="West US"
[vagrant@azure ~]$ AZURE_VM_MACHINE_NAME="estreaming-demo"
[vagrant@azure ~]$ MACHINE_IMG_NAME="0b11de9248dd4d87b18621318e037d37__RightImage-CentOS-7.0-x64-v14.2.1"
[vagrant@azure ~]$ MACHINE_USER="<username>"
[vagrant@azure ~]$ MACHINE_USER_PW="<password>"
[vagrant@azure ~]$ azure vm docker create -e 22 --ssh-cert=./myAzureMachine.pem --no-ssh-password -l $REGION $AZURE_VM_MACHINE_NAME $MACHINE_IMG_NAME $MACHINE_USER $MACHINE_USER_PW
```

You should get some type of message indicating that the vm was created successfully. If not, you'll have to troubleshoot based on the messages returned. If you are successful you should be able now to ssh into it.

```console
[vagrant@azure ~]$ ssh <username> $AZURE_VM_MACHINE_NAME.cloudapp.net
```

As well you should be able to see and manage your vm in the Azure portal app.

It is located [https://portal.azure.com/?r=1](https://portal.azure.com/?r=1). It is organized by resources, so click on Browse All and find the Virtual Machine resources links and go in from there. With the portal you can reprovision your vm to something other than the default (1 core, 1Gb), open up endpoints (port mappings to services on your vm like http/80, https/443, etc.)


#### <i class="icon-file"></i> Common Commands to Control the VM
Start the azure vm
```console
[vagrant@azure ~]$ azure vm start $AZURE_VM_OPTS $AZURE_VM_MACHINE_NAME
```

Shutdown the azure vm
```console
[vagrant@azure ~]$ azure vm shutdown $AZURE_VM_OPTS $AZURE_VM_MACHINE_NAME
```

Delete/Remove/Destroy the azure vm
```console
[vagrant@azure ~]$ azure vm delete $AZURE_VM_OPTS $AZURE_VM_MACHINE_NAME
```

Online help for the vm commands.  
```console
[vagrant@azure ~]$ azure vm help
```

Online help - general
```console
[vagrant@azure ~]$ azure help
```

More azure-cli info is located [here](http://www.hanselman.com/blog/ManagingTheCloudFromTheCommandLine.aspx)


CentOS notes (since I created a CENTOS vm)
Stop SSH timeouts
[follow these instructions](https://docs.oseems.com/general/application/ssh/disable-timeout)
Stop sudo password.
Make sure that the user you created is part of the wheel group
```console
[peterdoyle@estreaming-demo-air ~]$ sudo usermod -aG wheel <username>
```
Specify that that wheel group needs no sudo password
```console
[peterdoyle@estreaming-demo-air ~]$ sudo visudo
```
Specify this entry at the end of the sudoers file (!last line)
```bash
%wheel  ALL=(ALL)       NOPASSWD: ALL
```
