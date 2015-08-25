
Getting Started on Azure!
=========================


#### <i class="icon-file"></i> Install Node.js
Get the azure cli client running. For this the easiest way is to install Node.js and use the node client

```console
[vagrant@azure ~]$ sudo su -
[vagrant@azure ~]$ npm install azure-cli -g
[vagrant@azure ~]$ sudo npm install azure-cli -g
```


#### <i class="icon-file"></i>Get your azure account credentials
The details are here [Azure CLI](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-command-line-tools/)
but you need to have your microsoft account information if you are using a trial license or using the usage provided by a MSDN subscription
```console
vagrant@azure ~]$ azure account download
info:    Executing command account download
info:    Launching browser to http://go.microsoft.com/fwlink/?LinkId=254432
help:    Save the downloaded file, then execute the command
help:      account import <file>
info:    account download command OK
```

Then import the credentials. In this case i renamed them to azure-account.publishsettings and put them in my /vagrant folder
```console
vagrant@azure ~]$ azure account import /vagrant/azure-account.publishsettings
```

Now you should be able to see your account info
```console
[vagrant@azure ~]$ azure account list
```

Install Docker on the host OS
```console
[vagrant@azure ~]$ sudo yum install docker.io
[vagrant@azure ~]$ curl -sSL https://get.docker.com/ | sh
[vagrant@azure ~]$ sudo curl -sSL https://get.docker.com/ | sh
[vagrant@azure ~]$ sudo yum -y update
[vagrant@azure ~]$ sudo usermod -aG docker your_username
[vagrant@azure ~]$ sudo service docker start
[vagrant@azure ~]$ sudo docker run hello-world
```

Find a VM image on Azure using the azure-cli (in this case, a centos distro) that you will use to specify when you create a vm
```console
[vagrant@azure ~]$ azure vm image list | grep CentOS-71 |less
data:    5112500ae3b842c8b9c604889f8753c3__OpenLogic-CentOS-70-20150605                                                                    Public    Linux    OpenLogic
data:    5112500ae3b842c8b9c604889f8753c3__OpenLogic-CentOS-71-20150410                                                                    Public    Linux    OpenLogic
data:    5112500ae3b842c8b9c604889f8753c3__OpenLogic-CentOS-71-20150605                                                                    Public    Linux    OpenLogic
data:    5112500ae3b842c8b9c604889f8753c3__OpenLogic-CentOS-71-20150731                                                                    Public    Linux    OpenLogic  
```

Pick one of those and then create the necessary X:509 certificate


The current version of the Azure Management Portal only accepts SSH public keys that are encapsulated in an X509 certificate.

To SSH into your azure vm you need to generate a pair of public and private keys execute the following command:
```console
[vagrant@azure ~]$ ssh-keygen -t rsa -b 2048
```

OpenSSH private keys are directly readable by the openssl utility. The following command will take an existing SSH private key
(id_rsa in the example below) and create the .pem public key that is needed for Azure:
```console
[vagrant@azure ~]$ openssl req -x509 -key ~/.ssh/id_rsa -nodes -days 365 -newkey rsa:2048 -out myCert.pem
```


Create a VM on Azure using the azure-cli
```console
[vagrant@azure ~]$ REGION="West US"
[vagrant@azure ~]$ AZURE_VM_MACHINE_NAME="estreaming-demo"
[vagrant@azure ~]$ MACHINE_IMG_NAME="0b11de9248dd4d87b18621318e037d37__RightImage-CentOS-7.0-x64-v14.2.1"
[vagrant@azure ~]$ MACHINE_USER="peterdoyle"
[vagrant@azure ~]$ MACHINE_USER_PW="Celebration99$"
[vagrant@azure ~]$ azure vm docker create -e 22 --ssh-cert=./myCert.pem --no-ssh-password -l $REGION $AZURE_VM_MACHINE_NAME $MACHINE_IMG_NAME $MACHINE_USER $MACHINE_USER_PW
```


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


Azure Portal
The easiest way to sign in is to connect with your Office 365 Account first and then go to through the Portal. The Portal is located here [https://portal.azure.com](https://portal.azure.com/). The portal is organized around cloud resources, so use the Browse All links. Scroll down on the resources listing and find Virtual Machines (classic) and go in there. You should see the vm you create using the $AZURE_VM_MACHINE_NAME value with the azure-cli command 'azure vm create...'.



CentOS notes
Stop SSH timeouts
[follow these instructions](https://docs.oseems.com/general/application/ssh/disable-timeout)
Stop sudo password.
Make sure that the user you created is part of the wheel group
```console
[peterdoyle@estreaming-demo-air ~]$ sudo usermod -aG wheel peterdoyle
```
Specify that that wheel group needs no sudo password
```console
[peterdoyle@estreaming-demo-air ~]$ sudo visudo
```
Specify this entry at the end of the sudoers file (!last line)
```bash
%wheel  ALL=(ALL)       NOPASSWD: ALL
```
