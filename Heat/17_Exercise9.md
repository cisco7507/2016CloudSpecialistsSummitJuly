# 2016 Global Cloud Specialists Summit July

Summit Presentations


#Exercise 9 - Getting IP Addresses for Your Virtual Servers

One of the consequences of being an ADC proxy is the need for more listener IP addresses then standard servers. In OpenStack each port automatically gets a single *fixed_ip* allocation from a Neutron subnet. In our TMOS VE onboarding we use that address for a non-floating SelfIP on the TMM interface connected to the Neutron port.

When we want to add virtual services we need additional *fixed_ip* addresses attached to our Neutron port.  We could certainly build HOT where we pre-count all needed *fixed_ips* and request them when using our OS::Neutron::Port resources. However this often works counter to composability as it assumes that every time we want a OS::Neutron::Port for a TMOS VE it will always want the same number of *fixed_ips*. 

As an alternative, we've created a custom F5::Neutron::AddFixedIP resource which will allocate one *fixed_ip* given the port ID, and optionally Neutron subnet_id. This provides a simplified and composable method to get IP addresses from your Neuron infrastructure.

> **Note:** Warning.. the default quotta for IP addresses on a Neutron Port is 5. We get one for the VE already!

#### Your Orchestration Process

We are going to:

- Get More IPs Specifying only the Network
- Delete our Stack

**Do you think you could do this yourself?**

*As an input all we want to specify is the Neutron Network to use. *

*As an output we want to get our new *fixed_ip* addresses so we could use them for our VE configuration.*

Let's look at the F5::Neutron::AddFixedIP resource and try:

#### Properties (remember these are your inputs to your resource)
```
ip_address: {description: IP Addresses to attempt to allocate, immutable: false, required: false,
  type: string, update_allowed: false}
port: {description: Neutron Port, immutable: false, required: true, type: string,
  update_allowed: false}
subnet: {description: Neutron Subnet, immutable: false, required: false, type: string,
  update_allowed: false}
```

#### Attributes (remember these are your outputs typically)

```
ip_address: {description: Fixed IP Address.}
show: {description: Detailed information about resource., type: map}
subnet: {description: Fixed IP Subnet ID.}
```
We'll help.. your resource would look something like this:

```
  vs_1_address:
      type: F5::Neutron::AddFixedIP
      properties:
        port: f654fe00-d3de-4714-af06-33c8c0dca736 (Note this is port UUID, not Network)
```
Remember:

```
  { get_resource: <resource_name> }  - gets you a reference to a resource
  { get_attr: [ <resource_name>, <attribute_name> ] } - gets you the attribute value
```

Try to construct the rest of the HOT yourself. 

> **Hint:** If you really can't do this lof654fe00-d3de-4714-af06-33c8c0dca736ok at http://repo.mydemo.rocks/templates/exercise_9.yaml


<sub>
[Table of Contents](01_TOC.md) - Next [Exercise 12 - Allowing Addresses through Port Security](18_Exercise10.md) 
</sub>

<sup>
<b>July 2016</b></br>
n.menant@f5.com</br>
j.gruber@f5.com
</sup>
