# 2016 Global Cloud Specialists Summit July

Summit Presentations


#BIG-IQ License Orchestration

To license a BIG-IP as an unmanaged device with a BIG-IQ license pool you will need:

* An active BIG-IQ CM 5.0+
* An active BIG-IQ VE License Pool (you'll need the name)
* Credentials to the BIG-IP
* Credentials to the BIG-IQ
* iControl REST access to the BIG-IQ from the Heat Engine

When BIG-IQ is connected to only tenant networks, a Neutron FloatingIP allocation will be needed for the Heat Engine to reach the BIG-IQ with iControl REST.

When an TMOS VE is connected to a tenant network, a Neutron FloatingIP allocation will bw needed for the TMOS VE management IP address so BIG-IQ can use iControl REST to license the device.



```
heat_template_version: 2015-04-30

description: license a BIG-IP from a BIG-IQ Unmanaged License Pool

resources:
  bigip1:
    type: F5::BigIP::Device
    properties:
      username: admin
      password: openstack
      ip: 10.121.55.155
  bigiq1:
    type: F5::BigIQ::Device
    properties:
      username: admin
      password: openstack
      ip: 10.121.55.167
  license1:
    type: F5::BigIQ::LicensePoolUnmanagedMember
    depends_on: [bigip1, bigiq1] 
    properties:
      license_pool_name: best_ve_base
      bigip_server: { get_resource: bigip1 }
      bigiq_server: { get_resource: bigiq1 }

outputs:
  license_id:
    description: ID of the license
    value: { get_attr: [license1, license_uuid] }

```



<sub>
[Table of Contents](01_TOC.md) - Next [Clustering BIG-IPs](16_Clustering_BIG-IPs.md) 
</sub>

<sup>
<b>July 2016</b></br>
n.menant@f5.com</br>
j.gruber@f5.com
</sup>
