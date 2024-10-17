---
layout: post
title: "Baremetal automation"
date: 2024-10-17 00:00:00 +0000
tags: baremetal
categories: General
published: false
---

I recently completed a massive project which involved migrating and rebuilding a set of datacenters. The ask
was to build a few hundred servers and cluster them together for virtual machine placement. The requirement
was to take a clean server, install the required operating system (In this case it was to install them as ESXi
hosts) and then configure the clustering solution (Sticking with VMware was VCF).

In the 2024 with so much automation and DevOps tooling being made available, this seems like it would be a 
_"solved"_ problem. I assumed I would find multiple solutions to complete these tasks, but instead I found
that the installation of baremetal servers is the dirty secret kept from most developers.

Unlike the virtualization or containerized solutions, which have multiple solutions and often mature API
options. Baremetal servers almost have little options, or the options available have only a few realistic
solutions.

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

Baremetal installation as a result is also more complicated when considering being remote from the device, which
is the usual scenario for a datacenter deployment (Gone are the days where engineers would need to drive to a
datacenter and install the servers, at least in most cases). Automating the install of the baremetal also
introduces new challenges, such as how to provide the required defaults for an unattended install process.

Without going into detail about networking complexities that also arise from baremetal installation, lets just
look at how current server hardware allows for remote management and installation.

## Remote Management

Server hardware vendors provide a remote management device. The broad category of the technology used is
refered to as IPMI (Intergrated Platform Management Interface), but the adoption and implementation of the
interface is often up to the server hardware vendor. For example Dell uses iDRAC (Integrated Dell Remote Access
Controller), while HP uses iLO (Integrated Lights out). Luckily today there is a standard API that was defined
called RedFish which aims to provide a common interface to interact and control a baremetal server remotely.

These interfaces however only cover the direct hardware management. You will be able for example reboot the
server, inspect system health metrics, control physical devices, configure BIOS settings, or mount CD-ROM ISOs.
What the interfaces do not cover is direct control over the Operating System installation for example. That is
instead managed by the operating system itself, via unattended processes.

## Remote Installation

Automating the operating system installation can be achieved via directly mounting the installation media, 