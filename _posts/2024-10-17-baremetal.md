---
layout: post
title: "Baremetal automation"
date: 2024-10-17 00:00:00 +0000
tags: baremetal
categories: General
published: true
---

I recently completed a massive project which involved migrating and rebuilding a set of datacenters. The ask
was to build a few hundred servers and cluster them together for virtual machine placement. The requirement
was to take a clean server, install the required operating system (In this case it was to install them as ESXi
hosts) and then configure the clustering solution (Sticking with VMware was VCF).

In the 2024 with so much automation and DevOps tooling being made available, this seems like it would be a 
_"solved"_ problem. I assumed I would find multiple solutions to complete these tasks, but instead I found
that the installation of baremetal servers is the dirty secret kept from most developers.

Unlike the virtualization or containerized solutions, which have multiple solutions and often mature API
options. Baremetal installs have few options, or the options available have only a few realistic
solutions. The processes used are also not as smooth as the ones available for containers or virtualized
environments. Often feeling complex and _incomplete_.

The following set of posts will detail my journey through the wild world of baremetal installation and
hopefully it will help future engineers with a little bit of insight on how to solve these problems. This
post will detail the overall landscape of baremetal provisioning with details about the available technology
and terminology. The next post will go into detail about how I went about automating baremetal installs and
some of the unexpected challenges with the solution I went with.

# What is Baremetal?

Every device in existance uses baremetal installation. Most of these devices are installed via a semi-automated
process, so that users of the devices are not left with manual complicated tasks. It is reasonable to expect
your phone, laptop, tablet, etc to come installed with the operating system already in place. For a single
device with a single user, having the device installation being guided is acceptable, since the configuration
of the device cannot be known at the factory. Unlike virtualized environments, installation of the device can
be challenging to reset, since just deleting everything and starting again is not as trivial.

Baremetal installation is also more complicated when being remote from the device, which is the usual scenario
for a datacenter deployment (Gone are the days where engineers would need to drive to a
datacenter to install the servers, at least in most cases). Automating the install of the baremetal also
introduces new challenges, such as how to provide the required defaults for an unattended install process.

Without going into detail about networking complexities that also arise from baremetal installation, lets just
look at how current server hardware allows for remote management and installation.

## Remote Management

Server hardware vendors provide a remote management device. The broad category of the technology used is
refered to as IPMI (Intergrated Platform Management Interface), but the adoption and implementation of the
interface is often up to the server hardware vendor. For example Dell uses iDRAC (Integrated Dell Remote Access
Controller), while HP uses iLO (Integrated Lights out). Luckily today there is a standard API that was defined
called RedFish which aims to provide a common interface to interact and control a baremetal server remotely.
A added benefit is that tools exist that allow you to simulate the RedFish APIs and multiple libraries are
available for different languages.

These interfaces however only cover the direct hardware management. You will be able for example reboot the
server, inspect system health metrics, control physical devices, configure BIOS settings, or mount CD-ROM ISOs.
What the interfaces do not cover is direct control over the Operating System installation for example. That is
instead managed by the operating system itself, via unattended processes.

## Remote Installation

Automating the operating system installation can be achieved via directly mounting the installation media via
a physically attached CD-ROM drive or USB, or by a virtual media attachment. For the purpose of automation we
will only consider the virtual media option. Mounting the installation media by itself will not be enough, 
since most operating system installations require certain options to be selected, such as where the installation
should be placed, primary network information and default credentials. Using a IPMI solution would often provide
a remote console that would allow manual inputs of those details as if the installation was happening in front 
of the user. This is not viable if you want to scale however, since manually inputing the details for each
server takes time and mistakes are common.

The last option to consider is called PXE booting, which allows the BIOS to send a network request to a 
specified endpoint which contains the installation media. This approach, unlike the virtual media mounting,
requires less packaging to achieve. It does however require a suitable host server and the appropriate network 
configuration to achieve. PXE booting utilizes DHCP to achieve what is required, but this approach will not be
detailed in this article. 

When using the virtual media approach, we need a method to automate the manual steps that are required to 
install the host operating system. This is commonly called _"unattended"_ installs, since the installation does
not require manual inputs. The _"unattended"_ installs are usually a script, but each operating system will 
provide its own flavour of achieving this. For example, linux operating systems use a **kick start** file, 
which is normal shell commands that will be executed on first boot. Windows on the otherhand uses a XML file 
called **unattended.xml** which provides different XML options for configuring network, executing commands and
so on.

In my scenario, ESXi servers are a flavour of linux and as such utilize the **kick start** script. The challenge
is how to provide this script to the ISO during boot, since the default process is to boot into the manual 
installation mode when directly booting from the installation media.

## Custom installation media

At this point we have covered the remote management of baremetal servers via IPMI solutions and the concept of
unattended installs via scripts such as **kick start** files. Tying the two solutions together can be achieved
either by using a network share which hosts the default installation media and a suitable **kick start** file,
and then overriding the installation startup to use the **kick start** file via the network share. Or by 
injecting the **kick start** into the installation media and creating a custom installation media. Be aware 
that both options require the installation media to be customized, just in different ways.

The first option is to override the default boot sequence in the installation media to instruct the installation
to retrieve the **kick start** from a network share. Since this requires a network connection to be available,
it lends itself towards the DHCP style of network configuration instead of static configuration. The benefits of
using this approach is that the same installation media could be used for each server, since the **kick start**
customization is stored outside of the media. The drawback is that the installation process must have a working
network connection in order to retrieve the **kick start**, which normally requires a working DHCP server and
PXE boot environment.

The other option is create bespoke custom installation media for each server. This means that the **kick start**
is injected into the installation media and each server will have its own installation media. This approach has
the advantage of not requiring the network connection to be defined before installation. However it does have
the drawback that a custom ISO is required for each server in the environment, which depending on the number of
servers could end up being a large amount of disk space.

It is important to know that existing tools exist that provide these solutions, even if some of these tools have
a few limitations. Two tools worth looking into are _cobbler_ and _MaaS_ (Metal as a Service), which attempt to
provide an out the box solution for baremetal provisioning. The tools lean towards the PXE boot style of
install, so it might not always be possible for every environment. 

The next article will cover building a lab environment for baremetal installs and take a quick tour through the
existing options and tools available.
