# Introduction


# Goals
* Creating a repeatable linux install without any human intervention
* Running a docker container with total control over an ethernet device

# Software needed on the host machine

* A bash shell
* Virtualbox
* Docker 

# The setup

![setup overview](./test-setup.svg)

To keep the setup easy and straight forward we are going to create two virtual machiens which are not connected to the internet and both have two network connections. One, the host network, over which we can manage the machines from our host machine and another, the internal network, which is the network neccesary for our experiment.



## Creating the client machine

Firstly we are going to create the run off the mill linux client machine, the steps we'll be going through are: Creating the install media, create the hardware, putting it together.

### Create a linux image

### Creating the machine

```bash
#
# First we are going to create and install the client VM because it is a straight up linux machine 
# without any bells and whistles.
#
$ VBoxManage createvm \
    --name client \
    --ostype linux \
    --basefolder ./client-vm/ \
    --register \
    --default 

#
# On my machine this did print outsome errors although it didn't seem to impeed the creation and
# registration of the machine.
#
# $ VBoxManage createvm --name client --ostype linux --basefolder ./client-vm/ --default --register
# VBoxManage.exe: error: The machine is not mutable (state is PoweredOff)
# VBoxManage.exe: error: Details: code VBOX_E_INVALID_VM_STATE (0x80bb0002), component MachineWrap, interface IMachine, callee IUnknown
# VBoxManage.exe: error: Context: "ApplyDefaults(bstrDefaultFlags.raw())" at line 290 of file VBoxManageMisc.cpp
# 

# Removing the entire machine can be done using;
$ VBoxManage unregistervm --delete client 

# Configuring the network hardware
$ VBoxManage modifyvm
    --name client
    --nic1 hostonly
    --intnet2 internal-network

# The VBox documentation seems to suggest that new internal networks are configured
# and created as needed. When we are going to run all the machines we'll add a DHCP
# server to the "internal-network" so that the client machine will automatically 
# get a IP number assigned.
```

### Installing linux on the client machine

## Creating the docker host

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