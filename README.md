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

# Creating the machines

# Creating the echo-chamber

# Validation
With all the machines running we are going to log in to our client machine and then bounce some data off the docker container.

```
$ ssh user@client

client $ echo 'Bounce!' >( telnet echo-chamber )

# We expect to get 'Bounce!' back
```

# License
The work contained in this repository is licensed [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).

# Sources and references


* Diagram(s) drawn using [http://draw.io/]()