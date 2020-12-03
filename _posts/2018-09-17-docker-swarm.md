---
layout: post
title:  Local Docker Swarm
date:   2018-09-17 14:02:00 +0200
categories: General
published: true
---

Lately I have been spending a lot of my time dealing with Docker Engines and
performing orchestration of containers. Now, two tools exist that help deal with
container orchestration. The first tool is Kubernetes, which is amazing, awesome
and a lot of work to configure and run. Not really ideal for quick test environments
locally. It does work pretty well when using the built in option in the Docker
Engine that creates a single node kubernetes cluster. The problem is that you are
not really testing a cluster then with one node. The other option is built right
into docker, docker swarm, which is what I feel works better in our production
environments, since it requires far less configuration and tooling to run.

I really wanted to be able to spin up a local test environment and play with
deleting nodes or added replicas without needing a massive production environment
first. Of course building clusters locally is an extremely complex problem that
seems so simple at first glance. What follows is some of the learnings that I
have gone through in order to build a local test cluster.

# Prerequisites

In order to understand what I am doing here I highly recommend that you get a
basic understanding of the following

1. Docker Remote Access: The docker engine can expose a remote api, which is
   critical for most management tools.
2. Chef: Understanding how Chef works will be a benefit

# Attempt 1: Vagrant

Vagrant is an awesome wrapper tool for building VM's. Spinning up a single VM
is so simple, its almost painful. For example

```ruby
Vagrant.configure(2) do |config|
  config.vm.box = "hashicorp/precise64"
end
```

Of course, I want to run a cluster of machines and we use Chef to deploy into
production. So instead of going through and creating RAW vagrant machines I
instead looked towards using Kitchen instead.

# Attempt 2: Kitchen

Kitchen is a cool wrapper around Vagrant that helps manage and test Chef cookbooks
on a real VM. It provides a set of default configuration to help bootstrap the
VM with a chef-client and pretends to have a local chef server for attributes,
roles, data bags and so on.

Using my custom docker cookbook, I can install and configure the docker engine
on any number of nodes I want. The first draft looks as follows

```yaml
suites:
- name: local1
  driver:
    network:
    - ["private_network", {ip: "192.168.50.2"}]
- name: local2
  driver:
    network:
    - ["private_network", {ip: "192.168.50.3"}]
```

Which starts two VM's with the given IPs. But for some reason I could never get
the nodes to communicate with each other correctly, and so I wanted something a
little less difficult to configure.

# Attempt 3: Docker-machine

Finally a really simple solution (when it works). To create multiple nodes only
requires a single simple command `docker-machine create local1`. Sure, you can
do more configuration, but the simple case is so good and so painless that you
barely need anything else.

# Build a swarm

Building a swarm is not hard, you really just have to select which nodes are the
managers and which nodes are workers. For a local environment you could cheat and
make all of the nodes managers, but for good practice you should maybe make a
split between the different types. Just remember that only managers can actually
do things in the swarm (such as deploying stacks).

You can configure the docker machines to work as a swarm with the following commands

```bash
# Create some docker machine nodes
docker-machine create manager1 worker1 worker2
# Get the Docker Machine IPs
docker-machine ls
# Initialize the manager node
docker-machine ssh manager1 "docker swarm init --advertise-addr <MANAGER_IP>"
# Join the worker nodes
docker-machine ssh worker1 "docker swarm join --token <WORKER_TOKEN> <MANAGER_IP>"
docker-machine ssh worker2 "docker swarm join --token <WORKER_TOKEN> <MANAGER_IP>"
```

Once you have executed this command you will have a set of three machines in the swarm,
with one manager and two workers. From here you configure your local docker engine to use
the swarm manager instead by running `eval $(docker-machine env manager)` and you can deploy
your container stacks directly.

Docker Machine also allows the management and deployment of the docker engine to other environments
beside your local machine by using different providers during the creation command. Which would allow
you to build out swarms on stacks like AWS or Openstack if required.