# 2016 Global Cloud Specialists Summit July

Summit Presentations


#Exercise 11 - A Full Application Deployment

##<span style="color:red;">Discussion Only</span>

See if we can follow the dependencies in this one?

```

heat_template_version: 2014-10-16

description: Deploy Single Tenant Application

parameters:

  app_image:
    type: string
    label: Application Image
    description: The image to be used on the compute instance.
    constraints:
      - custom_constraint: glance.image 
  app_flavor:
    type: string
    label: Application Flavor
    description: Type of instance (flavor) to be used for the VE.
    default: m1.small
    constraints:
      - custom_constraint: nova.flavor
  app_network:
    type: string
    label: Application Network
    description: Application Interface Network.
    default: None
    constraints:
      - custom_constraint: neutron.network
    default: None
  bigiq_host:
    type: string
    label: BigIQ Host
    description: BigIQ license activation host.
    default: 169.53.3.162
  bigiq_username:
    type: string
    label: BigIQ Username
    description: BigIQ license user.
    default: admin
  bigiq_password:
    type: string
    label: BigIQ Password
    description: BigIQ license user.
    default: admin
    hidden: true
  bigiq_license_pool_name:
    type: string
    label: BigIQ License Pool Name
    description: BigIQ license pool to use for this VE     
  ve_os_image:
    type: string
    label: F5 VE Image
    description: The image to be used on the TMOS VE instance.
    constraints:
      - custom_constraint: glance.image 
  ve_os_flavor:
    type: string
    label: F5 VE Flavor
    description: Type of instance (flavor) to be used for the TMOS VE instance.
    default: m1.small
    constraints:
      - custom_constraint: nova.flavor 
  ve_admin_password:
    type: string
    label: F5 VE Admin User Password
    description: TMOS admin password for the VE instances.
    default: admin
    hidden: true
    constraints:
      - allowed_pattern: "[a-zA-Z0-9_-]*"
        description: Password in templates are limited to alpha, numeric, underscore, and dashes
  ve_root_password:
    type: string
    label: F5 VE Root User Password
    description: TMOS root password for the VE instances.
    default: admin
    hidden: true
    constraints:
      - allowed_pattern: "[a-zA-Z0-9_-]*"
        description: Password in templates are limited to alpha, numeric, underscore, and dashes
  ve_os_external_network:
    type: string
    label: F5 VE External Network
    description: Specify a specific Neutron external network for managment interface Floating IP creation.
    default: public
  ve_os_mgmt_network:
    type: string
    label: F5 VE Management Network
    description: Neutron network for the VE management interface.
    default: None
    constraints:
      - custom_constraint: neutron.network
  ve_os_ha_network:
    type: string
    label: F5 VE HA Network
    description: Neutron network for the VE HA config sycn and mirroring interface.
    default: None
    constraints:
      - custom_constraint: neutron.network
  ve_os_network_1:
    type: string
    label: F5 VE 1.2 Network
    description: Neutron network for the VE 1.2 data interface.
    default: None
    constraints:
      - custom_constraint: neutron.network
  ve_os_network_1_name:
    type: string
    label: F5 VE Network Name for the 1.2 Interface
    description: TMM network name for the untagged VLAN associated with the 1.2 data interface.
    default: internal      
  ve_os_network_2:
    type: string
    label: F5 VE 1.3 Network
    description: Neutron network for the VE 1.3 data interface.
    default: None
    constraints:
      - custom_constraint: neutron.network
  ve_os_network_2_name:
    type: string
    label: F5 VE Network Name for the 1.3 Interface
    description: TMM network name for the untagged VLAN associated with the 1.3 data interface.
    default: external


parameter_groups:
- parameters:
  - app_image
  - app_flavor
  - app_network
  - bigiq_host
  - bigiq_username
  - bigiq_password
  - bigiq_license_pool_name
  - ve_os_image
  - ve_os_flavor
  - ve_admin_password
  - ve_root_password
  - ve_os_external_network
  - ve_os_mgmt_network
  - ve_os_ha_network
  - ve_os_network_1
  - ve_os_network_1_name
  - ve_os_network_2
  - ve_os_network_2_name  
  
resources:

  appservers:
    type: http://repo.mydemo.rocks/templates/web_app_3.yaml
    properties:
      web_app_image: { get_param: app_image }
      web_app_flavor: { get_param: app_flavor }
      web_app_network: { get_param: app_network }      
      
  adccluster:
    type: http://repo.mydemo.rocks/templates/ve_cluster_4_networks_license_with_bigiq.yaml
    properties:
      f5_bigiq_host: { get_param: bigiq_host }
      f5_bigiq_username: { get_param: bigiq_username }
      f5_bigiq_password: { get_param: bigiq_password }
      f5_bigiq_license_pool_name: { get_param: bigiq_license_pool_name }
      f5_ve_os_image: { get_param: ve_os_image }
      f5_ve_os_flavor: 	{ get_param: ve_os_flavor }
      f5_ve_os_external_network: { get_param: ve_os_external_network }
      f5_ve_admin_password: { get_param: ve_admin_password }
      f5_ve_root_password: { get_param: ve_root_password }
      f5_ve_os_mgmt_network: { get_param: ve_os_mgmt_network }
      f5_ve_os_ha_network: { get_param: ve_os_ha_network}
      f5_ve_os_network_1: { get_param: ve_os_network_1 }
      f5_ve_os_network_1_name: { get_param: ve_os_network_1_name }
      f5_ve_os_network_2: { get_param: ve_os_network_2 }
      f5_ve_os_network_2_name: { get_param: ve_os_network_2_name }

  vip:
    type: F5::Neutron::AddFixedIP
    depends_on: adccluster
    properties:
      port: { get_attr: [ adccluster, resource.ve1.resource.network_1_port ] } 
  
  app_floatingip:
    type: OS::Neutron::FloatingIP
    depends_on: vip
    properties:
      floating_network: { get_param: ve_os_external_network }
      port_id: { get_attr: [ adccluster, resource.ve1.resource.network_1_port ] }
      fixed_ip_address: { get_attr: [vip, ip_address] }
      
  allowaddress1:
    type: F5::Neutron::HAPort
    depends_on: adccluster
    properties:
      port: { get_attr: [ adccluster, resource.ve2.resource.network_1_port ] }
      ip_addresses: 
        - { get_attr: [ vip, ip_address ] }

  app_adc_deployment:
    type: http://repo.mydemo.rocks/templates/web_app_3_adc_config.yaml
    depends_on: vip
    properties:
      f5_bigip_mgmt_ip: { get_attr: [ adccluster, resource.ve1_floating_ip.floating_ip_address ] }
      f5_bigip_username: admin
      f5_bigip_password: { get_param: ve_admin_password }
      vip_address: { get_attr: [vip, ip_address] }
      vip_network_name: /Common/external
      client_ssl_profile: /Common/clientssl
      pool_member_1_ip: { get_attr: [appservers, resource.web_server_1_port, fixed_ips, 0, ip_address] }
      pool_member_1_protocol_port: 80
      pool_member_1_weight: 0
      pool_member_2_ip: { get_attr: [appservers, resource.web_server_2_port, fixed_ips, 0, ip_address] }
      pool_member_2_protocol_port: 80
      pool_member_2_weight: 0
      pool_member_3_ip: { get_attr: [appservers, resource.web_server_3_port, fixed_ips, 0, ip_address] }
      pool_member_3_protocol_port: 80
      pool_member_3_weight: 0
      web_app_fqdn: 'test.mydemo.rocks'
  
  save_and_sync:
    type: http://repo.mydemo.rocks/templates/save_and_sync.yaml
    depends_on: allowaddress
    properties:
      f5_mgmt_ip: { get_attr: [ adccluster, resource.ve1_floating_ip.floating_ip_address ] }
      f5_admin_password: openstack
      f5_device_group_name: sync-failover-dg    

outputs:
  application_address:
    description: Name of the instance
    value: { get_attr: [ app_floatingip, floating_ip_address ] }


```