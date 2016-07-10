# 2016 Global Cloud Specialists Summit July

Summit Presentations


#Exercise 10 - Allowing Addresses through Port Security

By default, the reference implementations of Neutron implement Port anti-spoofing security. This means they will only let traffic through the port which is destinated for the correct MAC address and to the *fixed_ips* associated with the port. 

Neutron port security represents a challenge for L2 clustering solutions like our Group Traffic Group  solutions. If a failover is triggered, while the adjacent devices will be notified via GARP frames, the port security will remain unchanged. Since the *fixed_ips* are associated with only one port, we need a way to loosen the port security to allow certain addresses through on our port. Neutron implements the *allowed_address_pairs* attribute on port for just this situation.

Neutron port *allowed_address_pairs* attribute specifies a list of MAC:IP Address pairs to allow through the port. 

To facilitate the augmentation and removal of *allowed_address_pair* in a way that make sense in a declarative HOT, we've created the F5::Neutron::HAPort custom Heat resource.

>**Warning:** Unlike the F5::Neutron::AddFixedIP resource, F5::Neutron::HAPort assumes the HOT owns the port and replaces the *allowed_address_pairs* attribute for the port with the properties specified. When the F5::Neutron::HAPort is deleted, all *allowed_address_pairs* will be 

#### Your Orchestration Process

We are going to:

- Create two Ports (one active / one standby)
- Create *fixed_ips* on the first port (active)
- Implement *allowed_address_pairs* on the second port (standby) to allow our new *fixed_ips* through port security
- Delete our Stack

This exercise will use the last exercise's F5::Neutron::AddFixedIP resources to create the *fixed_ips*. 

Once we have resources for:

* 2 OS::Neutron::Port(s)
* 4 F5::Neutron::AddFixedIp(s)

We will add the *allowed_address_pairs* through a resource that would look something like this:

```
  ha_port:
    type: F5::Neutron::HAPort
    properties:
      port: [standby port's UUID]
      ip_addresses:
        - [IP address from the first added fixed_ip]
        - [IP address from the first added fixed_ip]
        - [IP address from the first added fixed_ip]
        - [IP address from the first added fixed_ip]

```
**Step 1: Construct a HOT with all the resources defined** 

Look at the HOT below. Make sure you understand it.

```
heat_template_version: 2014-10-16

description: Allocating additional IP addresses for a Port and allowing access to them on a second port.

parameters:

  vs_network:
    type: string
    label: Virtual Server Network
    description: Network to listen for service requests
    default: None
    constraints:
      - custom_constraint: neutron.network
      
resources:

  test_1_port:
    type: OS::Neutron::Port
    properties:
      network: { get_param: vs_network }
      security_groups:
        - default
  test_2_port:
    type: OS::Neutron::Port
    properties:
      network: { get_param: vs_network }
      security_groups:
        - default
  vs_1_address:
    type: F5::Neutron::AddFixedIP
    properties:
      port: { get_resource: test_1_port }
  vs_2_address:
    type: F5::Neutron::AddFixedIP
    properties:
      port: { get_resource: test_1_port }
  vs_3_address:
    type: F5::Neutron::AddFixedIP
    properties:
      port: { get_resource: test_1_port }
  vs_4_address:
    type: F5::Neutron::AddFixedIP
    properties:
      port: { get_resource: test_1_port }
  ha_port_1:
    type: F5::Neutron::HAPort
    properties:
      port: { get_resource: test_2_port }
      ip_addresses:
        - { get_attr: [ vs_1_address, ip_address ] }
        - { get_attr: [ vs_2_address, ip_address ] }
        - { get_attr: [ vs_3_address, ip_address ] }
        - { get_attr: [ vs_4_address, ip_address ] }
      
outputs:

    test_port_id:
      description: seed port for IP addresses
      value: { get_resource: test_2_port }
  
```

**Step 2: Launch Your Stack** 

You should know how to do that by now.

Unfortuately the Horizon GUI doesn't show *allowed_address_pairs*, so we wil have to use the CLI Neutron client.

Show the port details of the second port's UUID (output from your Stack)

```
neutron port-show [port UUID]

```
Here is a truncated sample output showing the *allowed_address_pairs*

```
+-----------------------+---------------------------------------------------------------------+
| Field                 | Value                                                               |
+-----------------------+---------------------------------------------------------------------+
| allowed_address_pairs | {"ip_address": "172.15.1.145", "mac_address": "fa:16:3e:e1:26:c1"}  |
|                       | {"ip_address": "172.15.1.146", "mac_address": "fa:16:3e:e1:26:c1"}  |
|                       | {"ip_address": "172.15.1.147", "mac_address": "fa:16:3e:e1:26:c1"}  |
|                       | {"ip_address": "172.15.1.148", "mac_address": "fa:16:3e:e1:26:c1"}  |


```
>**Question:** Where did the MAC address come from in your pairs?

>**Answer:** If you don't specify one in the properties to your F5::Neutron::HAPort, the MAC address of the port is used.

>**Question:** If you wanted to use MAC Masq on Traffic Groups what would you have to do?

>**Answer:** You would need to specify the MAC Masq address using F5::Neutron::HAPort on both ports.

**Step 3: Delete Your Stack** 

<sub>
[Table of Contents](01_TOC.md) - Next [Heat meet iApp](19_Heat_meet_iApp.md) 
</sub>

<sup>
<b>July 2016</b></br>
n.menant@f5.com</br>
j.gruber@f5.com
</sup>
