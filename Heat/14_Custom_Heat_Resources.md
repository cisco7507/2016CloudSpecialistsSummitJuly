# 2016 Global Cloud Specialists Summit July

Summit Presentations


#Custom Heat Resources

Being an open source project, all Heat resource plugins approved by the community are available for improvement or augmentation. The framework used for Heat resource plugins is natually extensible and Heat resources which are specific to a vendor technology can be installed on the Heat Engine for use within HOT.

> **Note:** To add custom Heat resource plugins you must have permissions to install software on the Heat Engines. Adding custom Heat resources is not a process exposed through the API.  

> **Note:** Heat resource plugin functionality is run from the Heat Engine. If your Heat resource requires interaction with a remote system (say iControl REST to a TMOS device), that access must be allowed from the Heat Engine host.

F5 publishes custom Heat resoruce plugins Here:

[https://github.com/F5Networks/f5-openstack-heat-plugins](https://github.com/F5Networks/f5-openstack-heat-plugins)

We have already installed some custom f5 Heat resouce plugins for you:

| Resource HOT Type | Function |
|--------------------------|-------------|
|F5::BigIP::Device| iControl REST connections to BigIP	Devices|
|F5::BigIQ::Device| iControl REST connections to BigIQ	Devices|
|F5::BigIQ::LicensePoolUnmanagedMember| BIG-IQ 5.0 Unmanaged Devices license orchestration|
|F5::Cm::Cluster| Syn-Failover cluster F5::BigIP::Device(s)|
|F5::CM::Sync| Sync a F5::BigIP::Device to its Sync-Failover cluster|
|F5::LTM::Pool| Manage an LTM Pool|
|F5::LTM::VirtualServer| Manage an LTM Virtual Server|
|F5::Neutron::AddFixedIP| Allocate Neutron IP addresses|
|F5::Neutron::HAPort| Allow f5 floating IP addresses access on a Neutron port|
|F5::Sys::Partition|Change to tenant folder|
|F5::Sys::Save| Save Big::IP::Device configuration|
|F5::Sys::iAppCompositeTemplate|Manage an iApp with dependancies|
|F5::Sys::iAppFullTemplate|Manage an iApp|
|F5::Sys::iAppService|Implement an iApp|

Like all resources you can inspect their properties and attributes. The properties and attributes tell you how to use the resources in your HOT.

<sub>
[Table of Contents](01_TOC.md) - Next [Exercise 9 - BIG-IQ Orchestration](15_Exercise9.md) 
</sub>

<sup>
<b>July 2016</b></br>
n.menant@f5.com</br>
j.gruber@f5.com
</sup>
