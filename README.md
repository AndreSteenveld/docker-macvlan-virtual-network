# Introduction


# Goals
* Creating a repeatable linux install without any human intervention
* Running a docker container with total control over an ethernet device

# Software needed on the host machine

* A bash shell
* Virtualbox
* Docker -- well, not really see "A Windows note aside"
* 7zip

# The setup

![setup overview](./test-setup.svg)

To keep the setup easy and straight forward we are going to create two virtual machiens which are not connected to the internet and both have two network connections. One, the host network, over which we can manage the machines from our host machine and another, the internal network, which is the network neccesary for our experiment.

## Creating the client machine

Firstly we are going to create the run off the mill linux client machine, the steps we'll be going through are: Creating the install media, create the hardware, putting it together.

### Create a linux image

To create a linux image we need an linux environment first so we are going to start up a clean ubuntu image using docker. This step can obviously be skipped if you're on the correct environment ;). Other things to make sure are; docker desktop is running and your drives are shared.

```bash
# Because tty is a bit of a hassle on windows make sure to use winpty (this works in bash)
# Running a container with a mount to the current directory in other shells on windows check: https://stackoverflow.com/a/41489151/95019
$ winpty docker run --rm -it -v "/$(pwd -W):/host" ubuntu:18.10 //bin/bash
```

Suddenly... intermission! As it turns out setting up a preseed enviroment is really easy but all the tools are as user friendly as you'd expect from a debian tooling. I want to get cracking with the networking first so I'm just going to grab a ready to go Ubuntu image.

1. To get a decent image I've downloaded the Ubuntu server image from: ()[https://www.osboxes.org/ubuntu-server/#ubuntu-server-1804-vbox]
    
    ```
    Information attached to the image:
    Username: osboxes
    Password: osboxes.org
    Root Account Password: osboxes.org
    VB Guest Additions & VMware Tools: Not Installed
    Keyboard Layout: US (Qwerty)
    VMware Compatibility: Version 10+
    ```

2. Using windows the file will located in the ```~/Downloads``` directory and we can pick it up there and extract a copy of the hard-disk image to the client-vm and docker-vm directory.

    ```
    $ 7z e -bd -bt -o./client-vm/ ~/Downloads/1804-264.7z '64bit/Ubuntu Server 18.04.2 (64bit).vdi' && mv ./client-vm/'Ubuntu Server 18.04.2 (64bit).vdi' ./client-vm/ubuntu-server-18.04.2.vdi
    $ 7z e -bd -bt -o./docker-vm/ ~/Downloads/1804-264.7z '64bit/Ubuntu Server 18.04.2 (64bit).vdi' && mv ./docker-vm/'Ubuntu Server 18.04.2 (64bit).vdi' ./docker-vm/ubuntu-server-18.04.2.vdi
    ```

Using these images we will create the virtual machines and in a later revision this will be replaced by a preseeder.

### Creating the machine

```bash
#
# First we are going to create and install the client VM because it is a 
# straight up linux machine without any bells and whistles.
#
$ VBoxManage createvm \
    --name client \
    --ostype linux_64 \
    --basefolder $( realpath ./client-vm/ ) \
    --register \
    --default 

#
# On my machine this did print outsome errors although it didn't seem to 
# impeed the creation and registration of the machine.
#
# $ VBoxManage createvm --name client --ostype linux --basefolder ./client-vm/ --default --register
# VBoxManage.exe: error: The machine is not mutable (state is PoweredOff)
# VBoxManage.exe: error: Details: code VBOX_E_INVALID_VM_STATE (0x80bb0002), component MachineWrap, interface IMachine, callee IUnknown
# VBoxManage.exe: error: Context: "ApplyDefaults(bstrDefaultFlags.raw())" at line 290 of file VBoxManageMisc.cpp
# 

# Removing the entire machine can be done using;
$ VBoxManage unregistervm --delete client 

# 
# Attach our harddisk, first we need to attach a controller and then add 
# we're going to add the image we downloaded in the previous section. 
#
$ VBoxManage storagectl client \
    --name client-disk-controller \
    --add sata \
    --portcount 2 \
    --bootable on

$ VBoxManage storageattach client \
    --storagectl client-disk-controller \
    --port 0 \
    --type hdd \
    --medium $( realpath ./client-vm/ubuntu-server-18.04.2.vdi )

#
# Configuring the network hardware
#

# Before we start configuring the networking hardware we need to make sure 
# there is a host only network available for VBox. The following will list 
# all available host-only network interfaces
$ VBoxManage list hostonlyifs

# If this list is empty a host network can be created by running
$ VBoxManage hostonlyif create 

# Assuming the default host network here, YMMV
$ VBoxManage modifyvm client \
    --nic1 hostonly \
    --hostonlyadapter1 "VirtualBox Host-Only Ethernet Adapter" \
    --nic2 intnet \
    --intnet2 "internal-network"

# The VBox documentation seems to suggest that new internal networks are 
# configured and created as needed. When we are going to run all the machines 
# we'll add a DHCP server to the "internal-network" so that the client machine 
# will automatically get a IP number assigned. For now though we'll just fix 
# the IP address of the client to 
```

#### A Windows note aside
Now that we have the hardware configured and ready to go I ran in to a specific windows issue, starting the virtual machine using the VirtualBox Manager threw the following error:
```
Failed to open a session for the virtual machine client.

Raw-mode is unavailable courtesy of Hyper-V. (VERR_SUPDRV_NO_RAW_MODE_HYPER_V_ROOT).

Result Code: E_FAIL (0x80004005)
Component: ConsoleWrap
Interface: IConsole {872da645-4a9b-1727-bee2-5585105b9eed}
```

A quick search lead me to the following stackoverflow post (virtualbox Raw-mode is unavailable courtesy of Hyper-V windows 10)[https://stackoverflow.com/questions/50053255/virtualbox-raw-mode-is-unavailable-courtesy-of-hyper-v-windows-10]. And this thread on the virual box forum (link)[https://forums.virtualbox.org/viewtopic.php?f=6&t=87237]. Summarizing the common thread seems to be to disable a option called "hypervisorlaunchtype".

```bash
# In a privileged shell run, and reboot
$ bcdedit //set hypervisorlaunchtype off

# Validate the setting by running
$ bcdedit
```

There seems to be a significant downside to this though; which is that docker desktop will fail to start using the default configuration. Even for this there seem to but multiple solutions;

1. Ignore this situation because we can do our experiment and then re-enable it
2. Run the docker deamon somewhere else, although it doesn't seem possible to configure docker desktop using it's gui the commandline tooling seems perfectly happy to do this [ (1)[https://medium.com/@peorth/using-docker-with-virtualbox-and-windows-10-b351e7a34adc] ]
3. Write a version of this experiment using Hyper-V

### Installing linux and configuring the client machine

## Creating the docker host

```bash
#
# Creating the docker host will follow the client creation roughly as it is simply a 
# linux machine as well. The noteable difference for this machine is that we'd allow
# the internal-network nic (2) in promiscues mode.
#
```

### Installing linux and configuring the docker machine

## Creating the echo-chamber

# Validation
With all the machines running we are going to log in to our client machine and then bounce some data off the docker container.

```bash
$ ssh user@client

client $ echo 'Bounce!' >( telnet echo-chamber )

# We expect to get 'Bounce!' back
```

# License
The work contained in this repository is licensed [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).

# Sources and references

* Diagram(s) drawn using [](http://draw.io/)
* Documentation about the VirtualBox commandline [](https://www.virtualbox.org/manual/ch08.html)
* Ubuntu 16.04 Desktop unattended installation [](http://gyk.lt/ubuntu-16-04-desktop-unattended-installation/)
* Virtualbox: Creation and controlling virtual maschines from commandline [](https://michlstechblog.info/blog/virtualbox-creating-and-controling-virtual-maschines-from-command-line/)