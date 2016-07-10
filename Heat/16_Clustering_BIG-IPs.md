# 2016 Global Cloud Specialists Summit July

Summit Presentations


#Clustering BIG-IPs

To license a BIG-IP as an unmanaged device with a BIG-IQ license pool you will need:

* Two to Four active BIG-IQ VEs
* Credentials to the BIG-IPs
* iControl REST access to the BIG-IQ from the Heat Engine

When an TMOS VE is connected to a tenant network, a Neutron FloatingIP allocation will bw needed for the TMOS VE management IP address so BIG-IQ can use iControl REST to license the device.

>**Note:** the F5::Cm::Cluster resource assumes the TMOS VE already have config sync, failover, and mirror interface setting configured. If you use the TMOS VE onboarding your HOT *user_data* should include the *device_is_sync*, *device_is_failover*, *device_is_mirror_primary* interface attributes set appropriately.

>**Note:** the F5::Cm::Cluster resource builds an iApp to perform the clustering. The iApp currently does not look for the presence of the ASM sync-only device service group (11.6+) and clustering will fail if you enable ASM (thanks ASM folks). It will be fixed shortly.

```

heat_template_version: 2015-04-30

resources:
  bigip1:
    type: F5::BigIP::Device
    properties:
      username: admin
      password: password
      ip: 10.121.55.167
  bigip2:
    type: F5::BigIP::Device
    properties:
      username: admin
      password: password
      ip: 10.121.55.168
  cluster:
    type: F5::Cm::Cluster
    depends_on: [bigip1, bigip2]
    properties:
      devices: [{ get_resource: bigip1 }, { get_resource: bigip2 }]
      device_group_type: sync-failover
      device_group_partition: Common
      device_group_name: sync-failover-dg

```

<sub>
[Table of Contents](01_TOC.md) - Next [Exercise 11 - Getting IP Addresses for Your Virtual Servers](17_Exercise11.md) 
</sub>

<sup>
<b>July 2016</b></br>
n.menant@f5.com</br>
j.gruber@f5.com
</sup>
