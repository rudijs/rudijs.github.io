---
layout: post
title:  "Restricting Docker Container Access with Iptables"
date:   2015-07-16 12:00:00
categories: devops docker security iptables firewall
tags: Devops Docker Security Iptables Firewall
---

## Overview

The [Docker networking documentation](https://docs.docker.com/articles/networking/) shows how to easily restrict external container access to a single IP using Iptables.

```
Docker’s forward rules permit all external source IPs by default. 

To allow only a specific IP or network to access the containers, insert a negated rule at the top of the DOCKER filter chain. 

For example, to restrict external access such that only source IP 8.8.8.8 can access the containers, the following rule could be added:

$ iptables -I DOCKER -i ext_if ! -s 8.8.8.8 -j DROP
```
   
What's the best approach for allowing, say, two external IP addresses?

Adding another rule negating another IP won't work as the 1st negation would have already matched and returned from the table.

One approach is to create a PRE_DOCKER table with a final return of DROP or REJECT before the DOCKER table.

This blog post will detail a method for this approach that's working well for my use case using Ubuntu 14.04 and Docker v1.7.

## Method

To begin create a bash script, let's name it `docker_iptables.sh` (mode 0755 executable).

This script will grant:
 
- Public access to *http* and *https* only.
- Two admin IPs will be granted access to all running containers.
- Two LAN IPs will be granted access to all running containers.

Everything else will be *rejected*

This script must run **after** docker **starts** or **restarts**.

```
#!/usr/bin/env bash

# Usage:
# timeout 10 docker_iptables.sh
#
# Use the builtin shell timeout utility to prevent infinite loop (see below)

if [ ! -x /usr/bin/docker ]; then
    exit
fi

# Create a PRE_DOCKER table
iptables -N PRE_DOCKER

# Default action
iptables -I PRE_DOCKER -j DROP

# Docker Containers Public Admin access (insert your IPs here)
iptables -I PRE_DOCKER -i eth0 -s 192.184.41.144 -j ACCEPT
iptables -I PRE_DOCKER -i eth0 -s 120.29.76.14 -j ACCEPT

# Docker Containers Restricted LAN Access (insert your LAN IP range or multiple IPs here)
iptables -I PRE_DOCKER -i eth1 -s 192.168.1.101 -j ACCEPT
iptables -I PRE_DOCKER -i eth1 -s 192.168.1.102 -j ACCEPT

# Docker internal use
iptables -I PRE_DOCKER -o docker0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
iptables -I PRE_DOCKER -i docker0 ! -o docker0 -j ACCEPT
iptables -I PRE_DOCKER -m state --state RELATED -j ACCEPT
iptables -I PRE_DOCKER -i docker0 -o docker0 -j ACCEPT

# Docker container named www-nginx public access policy
WWW_IP_CMD="/usr/bin/docker inspect --format='{% raw %}{{.NetworkSettings.IPAddress}}{% endraw %}' www-nginx"
WWW_IP=$($WWW_IP_CMD)

# Double check, wait for docker socket (upstart docker.conf already does this)
while [ ! -e "/var/run/docker.sock" ]; do echo "Waiting for /var/run/docker.sock..."; sleep 1; done

# Wait for docker web server container IP
while [ -z "$WWW_IP" ]; do echo "Waiting for www-nginx IP..."; WWW_IP=$($WWW_IP_CMD); done

# Insert web server container filter rules
iptables -I PRE_DOCKER -i eth0 -p tcp -d $WWW_IP --dport 80  -j ACCEPT
iptables -I PRE_DOCKER -i eth0 -p tcp -d $WWW_IP --dport 443 -j ACCEPT

# Finally insert the PRE_DOCKER table before the DOCKER table in the FORWARD chain.
iptables -I FORWARD -o docker0 -j PRE_DOCKER
```

It's important to note the usage of this script uses the builtin shell command `timeout`.

This is to prevent the script from hanging the machine if there's any error while waiting for Docker.

*Note*: As Docker is forwarding to port 80 we wait for the IP address of the container.
 
This is because if we want to forward to port 80 on another container, without the destination IP
 
the one rule will grant access to all containers forwarding to port 80.

### Usage

Using Upstart with Ubuntu 14.04 add this script `/etc/init/docker-iptables.conf`

Whenever docker starts or restarts it will run the `docker_iptables.sh` script.

*Note*: Adjust the path to script and email address to suit your environment.

```
description "Start Docker Custom Iptables Rules"
author "your@email.com"

start on started docker

script
    /usr/bin/timeout 30 /home/ubuntu/docker_iptables.sh
end script
```

### Summary

So far my experience using this approach is working well.

I'm using [Firehol](http://firehol.org/) to manage my firewall policies.

I've updated the firehol start/stop/restart init script to restart docker.

On docker's start/restart upstart event the `docker_iptables.sh` (the above script) will run.

This three stage process:

- Reloads the iptables firewall policies.
- Restarts all docker containers.
- Then filters access to the running docker containers.

I hope this helps, comments and feedback are very much welcomed.

If I've overlooked anything, if you can see room for improvement or if any errors please do let me know.

Thanks!
