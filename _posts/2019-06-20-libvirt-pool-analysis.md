---
layout: post
title: XML Analysis of Libvirt Storage Pools
date: 2019-06-20 17:51
summary: A piece-by-piece analysis of the XML for Directory, Filesystem and Disk Storage pools of libvirt.
categories: outreachy openstack
---

While preparing for my first patch to the sushy-tools repository, I was required to do an in-depth analysis of how Libvirt provides storage managment on the physical host, the different ways of configuring a physical storage device to be used by a domain/guest/Virtual Machine and how these devices are finally reflected in the respective domain XML.

According to the official Libvirt documentation [^1],

>Libvirt provides storage management on the physical host through storage pools and volumes. A storage pool is a quantity of storage set aside by an administrator, often a dedicated storage administrator, for use by virtual machines. Storage pools are divided into storage volumes either by the storage administrator or the system administrator, and the volumes are assigned to VMs as block devices.

This post will be talking about three different types of Libvirt storage pools (Directory, Filesystem and Disk Storage), primarily focussed towards how the devices configured through these pools are finally manifested in the domain XML to which they are attached.

On a sidenote, I used _virt-manager_ which provides a user-friendly GUI for interacting with the libvirt virtual machines for creating the storage pools and volumes [^2] and also for attaching those volumes to different libvirt VMs [^3].

## Directory Pool
This is a pool with type _dir_ and provides the means to manage files within a folder/directory. On the virt-manager, I configured it by providing one of the folders on my m/c as the source for the pool and created a raw storage volume within it of type _img_. 

I configured a Directory pool called _testPool_ and a storage volume called _testVol1.img_ which was attached to a VM called _varshaVM1_.

Output for `virsh pool-dumpxml testPool`:
{% highlight xml %}
<pool type='dir'>
  <name>testPool</name>
  <uuid>05f3950a-95c2-49f7-bb45-0a91296e443a</uuid>
  <capacity unit='bytes'>164161908736</capacity>
  <allocation unit='bytes'>139271409664</allocation>
  <available unit='bytes'>24890499072</available>
  <source>
  </source>
  <target>
    <path>/home/varsha/Documents/testPool</path>
    <permissions>
      <mode>0755</mode>
      <owner>1000</owner>
      <group>1000</group>
    </permissions>
  </target>
</pool>
{% endhighlight %}

Output for `virsh vol-dumpxml testVol1.img --pool testPool`:
{% highlight xml %}
<volume type='file'>
  <name>testVol1.img</name>
  <key>/home/varsha/Documents/testPool/testVol1.img</key>
  <source>
  </source>
  <capacity unit='bytes'>3221225472</capacity>
  <allocation unit='bytes'>3221229568</allocation>
  <physical unit='bytes'>3221225472</physical>
  <target>
    <path>/home/varsha/Documents/testPool/testVol1.img</path>
    <format type='raw'/>
    <permissions>
      <mode>0600</mode>
      <owner>0</owner>
      <group>0</group>
    </permissions>
    <timestamps>
      <atime>1561032566.367096192</atime>
      <mtime>1560768344.450720961</mtime>
      <ctime>1561029730.460979027</ctime>
    </timestamps>
  </target>
</volume>
{% endhighlight %}

And finally, this volume was manifested on the domain xml of _varshaVM1_ (`virsh dumpxml varshaVM1`) as follows:
{% highlight xml %}
...
<devices>
...
    <disk type='file' device='disk'>
      <driver name='qemu' type='raw'/>
      <source file='/home/varsha/Documents/testPool/testVol1.img'/>
      <target dev='vde' bus='virtio'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x09' function='0x0'/>
    </disk>
...
</devices>
...
{% endhighlight %}


## Filesystem Pool
This is a pool of type _fs_ and is more or less similar to a directory pool. In a filesystem pool, instead of a directory, a source block device will be mounted and files will be managed in the directory of it's mount point. I formatted my 32GB pen drive and created a single partition for it to act as a block device for configuring a filesystem pool and created raw volumes just as done in the directory pool. `/dev/sdb1` was used as a source for the pool.

I configured a pool called _fsPool_ and a raw storage volume called _fsVol.qcow2_ which was attached to a VM called _varshaVM1_.

The output for `virsh pool-dumpxml fsPool` was:
{% highlight xml %}
<pool type='fs'>
  <name>fsPool</name>
  <uuid>42a601bc-a563-4dc9-b42b-c0fc0f0f9839</uuid>
  <capacity unit='bytes'>31440814080</capacity>
  <allocation unit='bytes'>16384</allocation>
  <available unit='bytes'>31440797696</available>
  <source>
    <device path='/dev/sdb1'/>
    <format type='auto'/>
  </source>
  <target>
    <path>/var/lib/libvirt/images/fsPool</path>
    <permissions>
      <mode>0755</mode>
      <owner>1000</owner>
      <group>1000</group>
    </permissions>
  </target>
</pool>
{% endhighlight %}

The output for `virsh vol-dumpxml fsVol.qcow2 --pool fsPool` was:
{% highlight xml %}
<volume type='file'>
  <name>fsVol.qcow2</name>
  <key>/var/lib/libvirt/images/fsPool/fsVol.qcow2</key>
  <source>
  </source>
  <capacity unit='bytes'>2147483648</capacity>
  <allocation unit='bytes'>2148073472</allocation>
  <physical unit='bytes'>2148073472</physical>
  <target>
    <path>/var/lib/libvirt/images/fsPool/fsVol.qcow2</path>
    <format type='qcow2'/>
    <permissions>
      <mode>0644</mode>
      <owner>1000</owner>
      <group>1000</group>
    </permissions>
    <timestamps>
      <atime>1561055113</atime>
      <mtime>1561055029</mtime>
      <ctime>1561055112</ctime>
    </timestamps>
    <compat>1.1</compat>
    <features>
      <lazy_refcounts/>
    </features>
  </target>
</volume>
{% endhighlight %}

And finally, the attached volume manifested itself in the domain XML (`virsh dumpxml varshaVM1`) as:
{% highlight xml %}
...
<device>
  ...
  <disk type='file' device='disk'>
    <driver name='qemu' type='qcow2'/>
    <source file='/var/lib/libvirt/images/fsPool/fsVol.qcow2'/>
    <target dev='vdd' bus='virtio'/>
    <address type='pci' domain='0x0000' bus='0x00' slot='0x0b' function='0x0'/>
  </disk>
  ...
  </device>
...
{% endhighlight %}


## Disk Pool
This is a pool of type _disk_ and provides a pool based on a physical disk where volumes are created by adding partitions to the disk. Since I needed a physical storage device to implement a disk pool, I formatted my 32GB pen drive and created partitions on it using _fdisk_. The physical device was found at `/dev/sdb` and the partitions were found at `/dev/sdb1`, `/dev/sdb2` and `/dev/sdb3`.

I configured a Disk pool called _DiskPool_ and the automatically configured volumes were _sdb1_, _sdb2_ and _sdb3_. I attached the volume _sdb1_ to a VM called _varshaVM1_.

The output for `virsh pool-dumpxml DiskPool` was:
{% highlight xml %}
<pool type='disk'>
  <name>DiskPool</name>
  <uuid>0b60b9b2-7040-43f2-b3fb-880f422ff33b</uuid>
  <capacity unit='bytes'>31453470720</capacity>
  <allocation unit='bytes'>5368709120</allocation>
  <available unit='bytes'>26084697088</available>
  <source>
    <device path='/dev/sdb'>
      <freeExtent start='32256' end='1048576'/>
      <freeExtent start='1080832' end='2148532224'/>
      <freeExtent start='7517241344' end='31453470720'/>
    </device>
    <format type='dos'/>
  </source>
  <target>
    <path>/dev</path>
  </target>
</pool>
{% endhighlight %}

The output for `virsh vol-dumpxml sdb1 --pool DiskPool` was:
{% highlight xml %}
<volume type='block'>
  <name>sdb1</name>
  <key>/dev/sdb1</key>
  <source>
    <device path='/dev/sdb'>
      <extent start='1048576' end='2148532224'/>
    </device>
  </source>
  <capacity unit='bytes'>2147483648</capacity>
  <allocation unit='bytes'>0</allocation>
  <physical unit='bytes'>1024</physical>
  <target>
    <path>/dev/sdb1</path>
    <format type='none'/>
    <permissions>
      <mode>0660</mode>
      <owner>0</owner>
      <group>6</group>
    </permissions>
    <timestamps>
      <atime>1561032530.604694882</atime>
      <mtime>1561032530.604694882</mtime>
      <ctime>1561032530.604694882</ctime>
    </timestamps>
  </target>
</volume>
{% endhighlight %}

And finally, the attached partition `sdb1` was manifested in the _varshaVM1_ domain XML (`virsh dumpxml varshaVM1`) as below:
{% highlight xml %}
...
<devices>
...
    <disk type='block' device='disk'>
      <driver name='qemu' type='raw' cache='none' io='native'/>
      <source dev='/dev/sdb1'/>
      <target dev='vdc' bus='virtio'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x0a' function='0x0'/>
    </disk>
...
</devices>
...
{% endhighlight %}


All the XML descriptions for the various pools and volumes comply with what should actually come up for the stated types. The main reason I decided to document this is because I read multiple blogs but nowhere could I find a plausible explanation as to how the volumes attached to a VM manifests itself in the domain XML. This is something which might be of use in cases where one is trying to find out what all volumes are attached to a particular VM/domain.

---

[^1]: Libvirt Storage Management Official Documentation - [https://libvirt.org/storage.html](https://libvirt.org/storage.html)
[^2]: Manage KVM Storage Pools and Volumes - [https://www.tecmint.com/manage-kvm-storage-volumes-and-pools/](https://www.tecmint.com/manage-kvm-storage-volumes-and-pools/)
[^3]: Adding volumes to KVM Virtual Machines - [https://unix.stackexchange.com/questions/92967/how-to-add-extra-disks-on-kvm-based-vm](https://unix.stackexchange.com/questions/92967/how-to-add-extra-disks-on-kvm-based-vm)
