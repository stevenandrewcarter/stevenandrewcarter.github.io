---
layout: post
title:  "Terraform and Docker Swarm"
date:   2019-03-11 18:00:00 +0200
categories: General
published: true
---

In order to test a local docker swarm cluster you will quickly find a fairly cool tool called [Docker Machine](https://docs.docker.com/machine/), which
automates the building of a local cluster. The first time you start playing around with it you will quickly see how easy
it can be to create docker machines and _manually_ add them to a docker swarm. The sort of commands would look like the
following

```bash
# Create some docker machine nodes
docker-machine create manager1 worker1 worker2
# Initialize the manager node
docker-machine ssh manager1 "docker swarm init"
# Join the worker nodes
docker-machine ssh worker1 "docker swarm join"
docker-machine ssh worker2 "docker swarm join"
```

The only problem with this approach is that you will quickly find that tearing down and creating the swarm can become a
little tedious. Also, you may want to build something a little more custom when building the swarm (Installing certificates for example).

# Using Terraform

Luckily we can use a simple tool to perform the automation for us instead. [Terraform](https://www.terraform.io/) provides a pretty amazing
framework for spinning up our cluster. As of this article, there is no real provider that will automatically build
a solution like this. Instead we have to build a custom module that performs this type of operation. The steps below
are what we would want in order to achieve this.

1. Build our docker-machines
2. Initialize our swarm manager
3. Get the token from the swarm manager and the IP address (Since these would be randomly generated)
4. Initialize our swarm workers and join the cluster

In order to make our terraform module resilent to change we need to put in a couple of checks. Ideally, it would be
better if the functionality was provided as a provider instead of a module, but for the benefit of learning and to
also provide a bit of insight on creating more complex modules, we will create a module instead.

Now, we could create a really simple module that get invokes a bash script that builds the cluster. Just like the example
given above. The problem with this approach is that the benefits of terraform really don't apply, and it can be a
pain to extend out the script with additional features (for example customizing the docker machines). Initally I did
just use a bash script, but quickly felt it was out of place with the rest of the terraform configuration that was being
applied to the rest of the cluster. For example, when I wanted to deploy services to the cluster I had to invoke a
docker machine command to find out the IP was for the manager of the cluster. 

When you start using terraform remote state instead, you can get a much better flow from creating the swarm to adding
services to the swarm. So, in order to separate our concepts, we would actually need two modules, one for the docker
machines and one for the initialization of the swarm. The only problem is that connecting to the docker machine
requires the `docker-machine ssh` command by default. In order to avoid that we will need to use a standard _ssh_ 
connection instead.

## Docker Machine Module

Terraform by default attempts to reach eventual consistency and idempotency by default. It achieves this by keeping a
state and only implementing the difference between the existing state and the terraform configuration. In our case, 
docker machine also adds a certain amount of idempotency by default. For example if you attempt to create a machine
called _manager_, and attempt to create a docker machine with the same name, then the second call will fail. The problem
that can sometimes occur is that the docker machine will *recreate* a machine if something goes wrong with it. This might
not seem like a problem at first, and it really isn't much of one when you are just working with a single docker machine,
but it can cause a lot of chaos when you consider mutliple docker machines in a swarm. In order to prevent our swarm
from collapsing, we would also want our terraform play to include some self healing properties. This will be critical
when we consider our swarm, and it will be nice if our terraform play can determine the health state of the cluster.

So we will need some logic in order to determine the state of our cluster, which is when terraform starts to show a few
problems. Mainly that terraform does not have logical operators (As of version 0.11 at least). So instead of having a 
complex work around, we can instead just invoke bash scripts instead. 

As an example you can look at [this repo](https://github.com/stevenandrewcarter/terraform-docker-machine), which gives
a simple example of how to create the docker machines using terraform.
