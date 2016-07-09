# 2016 Global Cloud Specialists Summit July

Summit Presentations


#Structure of a HOT (Heat Orchestration Template)


The community guide to writing declarative templates is found here:

[Heat Template Guide](http://docs.openstack.org/developer/heat/template_guide/index.html)

You will notice there are reference to AWS Cloud Formation compatable functions. AWS Cloud Formation has its own specification which includes functions and resources which are unique to AWS.  

OpenStack too has its own specific declarative templating specification, HOT (Heat Orchestration Template). 

HOT is written in YAML. 

[YAML Specification](http://yaml.org/spec/1.2/spec.html)

YAML is particularly cool for this function as it is really trivial to *diff* two YAML documents and immediately do change control on your infrastructure orchestration templates.

The community specification for HOT is maintained in the following document: 

[Heat Orchestration Template (HOT) Specification](http://docs.openstack.org/developer/heat/template_guide/hot_spec.html)

The community guide to using HOT is found here:

[Heat Orchestration Template (HOT) Guide](http://docs.openstack.org/developer/heat/template_guide/hot_guide.html)

HOT has a few key components to understand before we start writing templates in our exercises.

####HOT Versions

Each HOT template must include the`heat_template_version` key with the HOT version value, for example, `2013-05-23`.  

```
heat_template_version: 2015-04-30
```

[A list of HOT template versions](http://docs.openstack.org/developer/heat/template_guide/hot_spec.html#heat-template-version)

####HOT Descriptions

While a description key is optional, it is recommended. We are writting self documenting infrastructure orchestrations after all.

```
description: The most awesome deployment of my application in OpenStack ever!

description: >
  This is how you can provide a longer description
  of your template that goes over several lines.
```

####HOT Resources

The only required section of your template is the `` resources: `` section. In this section we *declare* all resources with their desired end state properties for our deployment.

```
heat_template_version: 2015-04-30

description: The most awesome deployment of my application in OpenStack ever!

resources:
  my_instance:
    type: OS::Nova::Server
    properties:
      key_name: my_ssh_key
      image: b05949a3-b914-4926-838a-de8d2f0ec808
      flavor: ff76c167-996a-4479-bc50-0fdf30f18300
     networks:
         - network: 33fb65c6-d2e7-4247-ba17-7e2ee90ca2d7
```

####HOT Input Parameters (variables for your templates)

Static examples where we put in the UUIDs for our all our resources is pretty lame. We would like to be able to provide these values as variables. We accomplish this with our `` parameters `` section and the `` get_param `` template function.

```
heat_template_version: 2015-04-30

description: The most awesome deployment of my application in OpenStack ever!

parameters:
  key_name:
    type: string
    label: Key Name
    description: Name of key-pair to be used for compute instance
    default: my_ssh_key
  image_id:
    type: string
    label: Image ID
    description: Image to be used for compute instance
    default: b05949a3-b914-4926-838a-de8d2f0ec808
  flavor:
    type: string
    label: Instance Type
    description: Type of instance (flavor) to be used
    default: ff76c167-996a-4479-bc50-0fdf30f18300
  network:
    type: string
    label: Instance Network ID
    description: Network to connect or compute instance
    default: 33fb65c6-d2e7-4247-ba17-7e2ee90ca2d7

resources:
  my_instance:
    type: OS::Nova::Server
    properties:
      key_name: { get_param: key_name }
      image: { get_param: image_id }
      flavor: { get_param: flavor }
      networks:
        - network: { get_param: network }

```

####HOT Output Values (making templates composible)

If our templated orchestrations are going to be the source of information for either a heat client (say an imperative workflow orchestrator) or as a *nested resource* for another HOT template, they need to product output. This is accomplished in the ``` outputs ``` section of our template. 

Each *output attribute* should be explicitly declared.  Most of the time output parameters represent some *attribute* defined on a resource which we access with the ``` get_attr ``` HOT function. 

```
heat_template_version: 2015-04-30

description: The most awesome deployment of my application in OpenStack ever!

parameters:
  key_name:
    type: string
    label: Key Name
    description: Name of key-pair to be used for compute instance
    default: my_ssh_key
  image_id:
    type: string
    label: Image ID
    description: Image to be used for compute instance
    default: b05949a3-b914-4926-838a-de8d2f0ec808
  flavor:
    type: string
    label: Instance Type
    description: Type of instance (flavor) to be used
    default: ff76c167-996a-4479-bc50-0fdf30f18300
  network:
    type: string
    label: Instance Network ID
    description: Network to connect or compute instance
    default: 33fb65c6-d2e7-4247-ba17-7e2ee90ca2d7

resources:
  my_instance:
    type: OS::Nova::Server
    properties:
      key_name: { get_param: key_name }
      image: { get_param: image_id }
      flavor: { get_param: flavor }
      networks:
        - network: { get_param: network }

outputs:
  instance_ip:
    description: The IP address of the deployed instance
    value: { get_attr: [my_instance, first_address] }

```

<sub>
[Table of Contents](01_TOC.md) - Next [Exercise 1 - Discovering Your Resources](06_Exercise1.md) 
</sub>

<sup>
<b>July 2016</b></br>
n.menant@f5.com</br>
j.gruber@f5.com
</sup>
