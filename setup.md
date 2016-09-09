Here's how you can find the list of available snappy images on Azure:

[doesn't work]

```
$ azure vm image list | grep "Ubuntu-.*-Snappy"
data:    b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-15.04-Snappy-core-amd64-edge-201507020801-108-en-us-30GB
The naming convention is: "<publisher guuid>__Ubuntu-<version>-Snappy-<flavor>-<arch>-<publish_date>-<version>-<locale>-<disk_size>". Canonical's publisher GUUID on Microsoft Azure is "b39f27a8b8c64d52b05eac6a62ebad85".
```

[works]

```
$ azure vm image list --location westeurope --publisher canonical | grep "Ubuntu_Snappy"
data:    Publisher  Offer                      Sku                OS     Version          Location    Urn                                                     
data:    ---------  -------------------------  -----------------  -----  ---------------  ----------  --------------------------------------------------------
...
data:    canonical  Ubuntu_Snappy_Core         15.04              Linux  2016.0318.1949   westeurope  canonical:Ubuntu_Snappy_Core:15.04:2016.0318.1949
...
```

In our example, we'll use the latest image at the time of this writing. We suggest you to replace the tags for date and version to use the latest one available.
b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-15.04-Snappy-core-amd64-edge-201507020801-108-en-us-30GB
SSH is disabled by default on snappy systems, for enhanced security. You can turn it on by providing some configuration when you launch the instance, and for that you will need to create a cloud-init configuration file that will turn on SSH so you can login to play with Ubuntu Core.

Create a file called cloud.cfg with the exact lines of text you see below:
```
#cloud-config
snappy:
    ssh_enabled: True
```
Now you are ready to launch the image on Azure. The general form of this command is:

[doesn't work]

```
$ azure vm create <NAME> <IMAGE> <USER> <PASSWORD> <flags>
The following is an example command. Remember to replace the image name by the latest available, and to replace {UNIQUE_ID} with something that uniquely identifies your machine, like snappy-test-{your_nickname} :
$ azure vm create {UNIQUE_ID} \
b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-15.04-Snappy-core-amd64-edge-201507020801-108-en-us-30GB ubuntu \
--location "North Europe" --no-ssh-password \
--ssh-cert ~/.ssh/azure_pub.pem --custom-data ~/cloud.cfg -e
```

[works, after creating a Virtual Network, and Network Interface]
```
$ azure vm create -Q "canonical:Ubuntu_Snappy_Core:15.04:2016.0318.1949" -u <username> --ssh-publickey-file ~/.ssh/azure_pub.pem --custom-data cloud.cfg -g <resource-group> -n <machine-name> --location <location, e.g. westeurope> -y Linux --nic-id /subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Network/networkInterfaces/<networkInterface>
```

You will need to wait a minute or two while Azure provisions and launches the instance. It will show up in the list of your running instances:
```
$ azure vm list
info:    Executing command vm list
+ Getting virtual machines
data:    Name         Status     Location      DNS Name                  IP Address
data:    -----------  ---------  ------------  ------------------------  ----------
data:    snappy-test  ReadyRole  North Europe  snappy-test.cloudapp.net  <VARIABLE>
info:    vm list command OK
When the image state is "ReadyRole" you can make a note of the hostname from the listing above, and login with SSH to the instance (replace snappy-test.cloudapp.net with the DNS name from your azure vm list command):
$ ssh -i ~/.ssh/azure.key ubuntu@snappy-test.cloudapp.net
Congratulations! Now you are ready to start exploring snappy Ubuntu Core ›
```

[note]
the hostname isn't auto-assigned, and Status is no longer a column. I see:
```
data:    MV                 snappy-test-tradle  Succeeded          VM running      westeurope  Standard_DS1
```
