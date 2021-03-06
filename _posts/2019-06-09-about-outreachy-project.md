---
layout: post
title: Outreachy | A Quick Dissection of my project at Openstack Ironic
date: 2019-06-09 12:54
summary: A brief description of the project in line with my understanding
categories: outreachy openstack
---

It's only been three weeks since I embarked on my Outreachy internship at Openstack and I already feel like my brain has evolved so much in terms of the tremendous amount of learning that has taken place the past few weeks. This post is aimed at sharing some of that knowledge with my fellow readers.

Before we begin, let me give you a small introduction to the organization that I am working with: <span class="red">__Openstack__</span>.

> OpenStack is a set of open source, scalable software tools for building and managing cloud computing platforms for public and private clouds. OpenStack is managed by the OpenStack Foundation, a non-profit that oversees both development and community-building around the project.

There are many teams working on different projects under the banner of Openstack, one of which is <span class="red">__Ironic__</span>. <ins>Openstack Ironic is a set of projects that perform bare metal (no underlying hypervisor) provisioning</ins> in use cases where a virtualized environment might be inappropriate and a user would perfer to have an actual, physical, bare-metal server. Thus, bare metal provisioning means a customer can use hardware directly, deploying the workload (image) onto a real physical machine instead of a virtualized instance on a hypervisor.

The title for the project that I am working on is:

### "Extend sushy to support RAID"

Reading the topic, the first word that easily catches one's attention is <span class="red">__RAID__</span>. RAID is an acronym for _Redundant Array of Independent Disks_. It is <ins>a technique of storing the same data in different places on multiple disks instead of a  single disk for increased I/O performance, data storage reliability or both.</ins> There are different RAID levels that use a mix/match of striping, parity or mirroring, each optimized for a specific situation. 

<ins>The main theme behind the existence of Ironic is to allow it's users (mostly system administrators) to access the hardware, running their servers, remotely.</ins> This gives them the ability to manage/control the servers 24X7 which is crucial in case of any server failures at odd hours. This is where <span class="red">_Baseboard Management Controller (BMC)_</span> comes to the rescue. It is an independent satellite computer that can typically be a system-on-chip microprocessor or a standalone computer. It is used to perform system monitoring and management related tasks.

At the heart of a BMC lies <span class="red">__Redfish__</span>, which is a Protocol used by the BMC (on bare metal machines) to communicate remotely via JSON and OData. It is a standard RESTful API offered by DMTF to get and set hardware configuration items on physical platforms.

<span class="red">__Sushy__</span> is a <ins>client side Python library used by Ironic to communicate with Redfish based systems.</ins> 

To test and support the development of the `sushy` library, a set of simple simulation tools called <span class="red">__sushy-tools__</span> is used, since it is not always possible to have an actual bare metal machine with Redfish at hand. The package offers two emulators, a _static emulator_ and a _dynamic emulator_. The <span class="red">_static emulator_</span> simulates a read only BMC and is a simple REST API server that responds the same things to client queries. The <span class="red">_dynamic emulator_</span> simulates the BMC of a Redfish bare metal machine and is used to manage libvirt VMs (mimicking actual bare metal machines), resembling how a real Redfish BMC manages actual bare metal machine instances. Like it's counterpart, the dynamic emulator also exposes REST API endpoints that can be consumed by a client using the sushy library.


![SushyDiagram](https://user-images.githubusercontent.com/20443665/59162360-42057380-8b0d-11e9-8634-d0bf9b5b2106.png)


Thus, the aim of my project is <ins>to add functionality to the existing sushy library so that the clients are able to configure RAID based volumes for storage on their bare metal instances remotely.</ins> There are two aspects to this project, one is the addition of code in the sushy library and the other is adding support for emulating the storage subsystem in sushy-tools so that we are actually able to test the added functionality in sushy against something. Since there is already one contributor working on adding RAID implementation to sushy, my task in the project involves adding the RAID support for emulation to the sushy-tools dynamic emulator (more specifically the libvirt driver). 

#### Challenges faced till now

The first task that the mentors asked me to do was to learn how to get/receive data from the libvirt VMs using sushy via the sushy-tools dynamic emulator. Honestly, the task did feel a bit intimidating to me in the beginning. I had to go back to the networking basics and brush them up. After that, I spent quite some time reading blogs and following videos on libvirt VMs. I encountered some problems while setting up/creating the VMs using the virt-install command and had to ultimately fall back to the virt-manager GUI to spin up the VMs.

I also wasn't able to set up the local development environment for sushy-tools due to the absence of instructions for the same on the README. I have to admit that I thought 'maybe the instructions aren't mentioned because they are too obvious' and felt a bit embarrased about them not being obvious to me. But then, I reminded myself that I am here to learn and grow so I went ahead and asked my mentor about the same who very warmly helped me out with the commands to be used for setting up the repository. After successfully setting up the environment, I even submitted a patch for adding the corresponding instructions to the docs so as to make it a bit easier for new-comers like me to start contributing to the project.

While working the first task, I came across some anomalies in the behaviour of the dynamic emulator which led me to digging up the code to find out what was happening. I ultimately found out that it was actually a bug and submitted a patch for the same. 

Right now, I am working on exposing the Volume API in the dynamic emulator. There are a lot of decisions that are to be made, related to the mapping of the sushy storage resources to the libvirt VMs. I have been researching about them and am regularly having discussions with the mentors on the same and will hopefully include the final implementational mappings in my next blog post.
