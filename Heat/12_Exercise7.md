# 2016 Global Cloud Specialists Summit July

Summit Presentations


#Exercise 7 - Using LBaaSv2 in Heat

##<span style="color:red;">Discussion Only</span>

In the Mitaka release neww LBaaSv2 heat resources were added:

[https://specs.openstack.org/openstack/heat-specs/specs/mitaka/lbaasv2-support.html](https://specs.openstack.org/openstack/heat-specs/specs/mitaka/lbaasv2-support.html) 

[OS::Neutron::LBaaS::LoadBalancer](http://docs.openstack.org/developer/heat/template_guide/openstack.html#OS::Neutron::LBaaS::LoadBalancer)

[OS::Neutron::LBaaS::Listener](http://docs.openstack.org/developer/heat/template_guide/openstack.html#OS::Neutron::LBaaS::Listener)

[OS::Neutron::LBaaS::Pool](http://docs.openstack.org/developer/heat/template_guide/openstack.html#OS::Neutron::LBaaS::Pool)

[OS::LBaaS::PoolMember](http://docs.openstack.org/developer/heat/template_guide/openstack.html#OS::Neutron::LBaaS::PoolMember)

[OS::LBaaS::HealthMonitor](http://docs.openstack.org/developer/heat/template_guide/openstack.html#OS::Neutron::LBaaS::HealthMonitor)

Let's consider and example which builds on your last composibility exercise.

```
heat_template_version: 2015-04-30

description: Web servers in a LBaaS Pool

parameters:

  lb_subnet:
    type: string
    label: Virtual Service Subnet
    description: Subnet to listen on for the virtual service
    default: None
    constraints:
      - custom_constrant: neutron.subnet
  lb_port:
    type: number
    label: Virtual Service TCP Port
    description: TCP port to listen on for the virtual service
    default: 80
    constraints:
      - range: { min: 1, max: 65535 } 
  web_app_image:
    type: string
    label: Web App Image
    description: The image to be used for the web applicatoin.
    constraints:
      - custom_constraint: glance.image 
  web_app_flavor:
    type: string
    label: Web App Flavor
    description: Type of instance (flavor) to be used for the web application.
    default: m1.small
    constraints:
      - custom_constraint: nova.flavor
  web_app_port:
    type: number
    label: Web App TCP POrt
    description: TCP port for the web application
    default: 80
    constraints:
      - range: { min: 1, max: 65535 }
  use_config_drive:
    type: boolean
    label: Use Config Drive
    description: Use config drive to provider meta and user data.
    default: true 
  web_app_network:
    type: string
    label: Web App Network
    description: Network to listen for web requests
    default: None
    constraints:
      - custom_constraint: neutron.network
  package_to_install:
    type: string
    label: Software Package
    description: The package to install the web server
    default: apache2
  password:
    type: string
    label: password
    description: ubuntu user password
    hidden: true
  external_network:
    type: string
    label: External Network
    description: Specify a specific Neutron external network for Floating IP creation.
    default: public 
    constraints:
      - custom_constraint: neutron.network

parameter_groups:
  - parameters:
    - web_app_image
    - web_app_flavor
    - web_app_network
  - parameters:
    - package_to_install
    - password
    - external_network

resources:

  loadbalancer:
    type: OS::Neutron::LBaaS::LoadBalancer
    properties:
      vip_subnet: { get_param: lb_subnet }

  listener:
    type: OS::Neutron::LBaaS::Listener
    properties:
      loadbalancer: { get_resource: loadbalancer }
      protocol: HTTP
      protocol_port: { get_param: lb_port }
      
  pool:
    type: OS::Neutron::LBaaS::Pool
    properties:
      lb_algorithm: ROUND_ROBIN
      protocol: HTTP
      listener: { get_resource: listener }

  monitor:
    type: OS::Neutron::LBaaS::HealthMonitor
    properties:
      delay: 3
      type: HTTP
      timeout: 3
      max_retries: 3
      pool: { get_resource: pool }

  web_app_security_group:
    type: 'http://repo.mydemo.rocks/templates/exercise_6_web_server_security_group.yaml'

  web_server_1:
    type: 'http://repo.mydemo.rocks/templates/exercise_6_web_server.yaml'
    depends_on: web_app_security_group
    properties:
      web_app_image: { get_param: web_app_image }
      web_app_flavor: { get_param: web_app_flavor }
      web_app_network: { get_param: web_app_network }
      package_to_install: { get_param: package_to_install }
      password: { get_param: password }
      external_network: { get_param: external_network }

  pool_member1:
    type: OS::Neutron::LBaaS::PoolMember
    properties:
      pool: { get_resource: pool }
      address: { get_attr: [web_server_1_port, fixed_ips, 0, ip_address] }
      protocol_port: { get_param: web_app_port }
      subnet: { get_attr: [web_server_1_port, fixed_ips, 0, subnet_id] }

  web_server_2:
    type: 'http://repo.mydemo.rocks/templates/exercise_6_web_server.yaml'
    depends_on: web_app_security_group
    properties:
      web_app_image: { get_param: web_app_image }
      web_app_flavor: { get_param: web_app_flavor }
      web_app_network: { get_param: web_app_network }
      package_to_install: { get_param: package_to_install }
      password: { get_param: password }
      external_network: { get_param: external_network }

  pool_member2:
    type: OS::Neutron::LBaaS::PoolMember
    properties:
      pool: { get_resource: pool }
      address: { get_attr: [web_server_2_port, fixed_ips, 0, ip_address] }
      protocol_port: { get_param: web_app_port }
      subnet: { get_attr: [web_server_2_port, fixed_ips, 0, subnet_id] }

  web_server_3:
    type: 'http://repo.mydemo.rocks/templates/exercise_6_web_server.yaml'
    depends_on: web_app_security_group
    properties:
      web_app_image: { get_param: web_app_image }
      web_app_flavor: { get_param: web_app_flavor }
      web_app_network: { get_param: web_app_network }
      package_to_install: { get_param: package_to_install }
      password: { get_param: password }
      external_network: { get_param: external_network }
      
  pool_member3:
    type: OS::Neutron::LBaaS::PoolMember
    properties:
      pool: { get_resource: pool }
      address: { get_attr: [web_server_3_port, fixed_ips, 0, ip_address] }
      protocol_port: { get_param: web_app_port }
      subnet: { get_attr: [web_server_3_port, fixed_ips, 0, subnet_id] } 


outputs:
  lburl:
    value:
      str_replace:
        template: http://IP_ADDRESS:PORT
        params:
          IP_ADDRESS: { get_attr: [ loadbalancer, vip_address ] }
          PORT: { get_param: lb_port }
    description: >
      This URL for the Load Balancer.
  web_server_1_name:
    description: Name of the web_server_1 instance
    value: { get_attr: [web_server_1, resource.web_server, name] }
  web_server_1_instance_id:
    description: ID of the web server 1 instance
    value: { get_attr: [web_server_1, resource.web_server] }
  web_server_1_ip:
    description: The IP address of the web server 1 instance
    value: { get_attr: [web_server_1, resource.web_server_port, fixed_ips, 0, ip_address] }
  web_server_1_public_ip:
    description: The public IP address of the web server 1 instance
    value: { get_attr: [web_server_1, resource.web_server_floatingip, floating_ip_address] }
  web_server_2_name:
    description: Name of the web_server_2 instance
    value: { get_attr: [web_server_2, resource.web_server, name] }
  web_server_2_instance_id:
    description: ID of the web server 2 instance
    value: { get_attr: [web_server_2, resource.web_server] }
  web_server_2_ip:
    description: The IP address of the web server 2 instance
    value: { get_attr: [web_server_2, resource.web_server_port, fixed_ips, 0, ip_address] }
  web_server_2_public_ip:
    description: The public IP address of the web server 2 instance
    value: { get_attr: [web_server_2, resource.web_server_floatingip, floating_ip_address] }
  web_server_3_name:
    description: Name of the web_server_3 instance
    value: { get_attr: [web_server_3, resource.web_server, name] }
  web_server_3_instance_id:
    description: ID of the web server 3 instance
    value: { get_attr: [web_server_3, resource.web_server] }
  web_server_3_ip:
    description: The IP address of the web server 3 instance
    value: { get_attr: [web_server_3, resource.web_server_port, fixed_ips, 0, ip_address] }
  web_server_3_public_ip:
    description: The public IP address of the web server 3 instance
    value: { get_attr: [web_server_3, resource.web_server_floatingip, floating_ip_address] }  

```


<sub>
[Table of Contents](01_TOC.md) - Next [Exercise 8 - TMOS VE Onboarding](13_Exercise8.md) 
</sub>

<sup>
<b>July 2016</b></br>
n.menant@f5.com</br>
j.gruber@f5.com
</sup>
