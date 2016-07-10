# 2016 Global Cloud Specialists Summit July

Summit Presentations


#Heat meet iApp

We have already explored how we map HOT resources and their attributes into two other templated configuration management systems.

* cloud-init
* TMOS VE OpenStack startup agent

iApps are just another configuration management we can expose through HOT. 

#### Mapping Declarative HOT to iApps

The mapping to iApps is not as trivial as the purposely declarative cloud-init and TMOS VE OpenStack startup agent *user_data*. iApps can be declarative, but the majority of the published iApps are not. There is embedded flow logic inherent to the interactions possible with the TMOS GUI presentation widgets and imperative controls available in the implementation section. 

We can see the same issues coming with iAppsLX when you consider the most popular iAppsLX was created to augment the GUI. Yikes!

 **Don't bother trying to rig a messy iApp into HOT**

There are custom Heat resources which will help you manage the L4-L7 service configuration of your BIG-IP in a declarative HOT way. 

| Heat Resoruce | Purpose
|-----------------|-----------------|
|F5::Sys::iAppFullTemplate| used to publish a declarative iApp to a BIG-IP|
|F5::Sys::iAppCompositeTemplate| used to publish a declarative iApp to a BIG-IP with separate implementation and presentation properties|
|F5::Sys::iAppService|used to implement an Application Service from your publiched iApps|

####Example of using F5::Sys::iAppFullTemplate to publish an iApp

```
resources: 

  bigip_rsrc:
    type: F5::BigIP::Device
    properties:
      username: { get_param: bigip_un }
      password: { get_param: bigip_pw }
      ip: { get_param: ip_address }

  partition:
    type: F5::Sys::Partition
    properties:
      name: Common
      bigip_server: { get_resource: bigip_rsrc }

  iapp_templ:
    type: F5::Sys::iAppFullTemplate
    properties:
      bigip_server: { get_resource: bigip_rsrc }
      partition: { get_resource: partition }
      full_template:
        str_replace:
          params:
            __partition__: { get_param: partition_name }
          template: |
            sys application template thanks_world {
              actions replace-all-with {
                definition {
                  html-help {
                    <!-- insert html help text -->
                  }
                  implementation {
                    ...
                  }
                  macro {
                    ...
                  }
                  presentation {
                    ...
                  }
                  role-acl none
                  run-as none
                }
              }
              partition __partition__
              description none
              requires-modules none
            }

```
####Example of using F5::Sys::iAppCompositeTemplate to publish an iApp 

```
resources:
  bigip_rsrc:
    type: F5::BigIP::Device
    properties:
      username: { get_param: bigip_un }
      password: { get_param: bigip_pw }
      ip: { get_param: bigip_ip }
  partition:
    type: F5::Sys::Partition
    properties:
      name: Common
      bigip_server: { get_resource: bigip_rsrc }
  iapp_template:
    type: F5::Sys::iAppCompositeTemplate
    properties:
      name: { get_param: iapp_template_name }
      bigip_server: { get_resource: bigip_rsrc }
      partition: { get_resource: partition }
      requires_modules: [ ltm ]
      implementation:
        str_replace:
          params:
            __pool_name__: { get_param: pool_name }
            __node_address__: { get_attr: [ vm_node_1, fixed_ips, 0, ip_address] }
            __node_port__: { get_param: tcp_port }
            __vip_ip_address__: { get_attr: [ new_fixed_ip, ip_address ] }
          template:|
			#TMSH-VERSION: 11.6.0
			iapp::template start
			tmsh::create {
			  ltm pool __pool_name__
			  description "A pool of http servers"
			  load-balancing-mode least-connections-node
			  members replace-all-with {
				  __node_address__:__port__ {
					  address __node_address__
				  }
			  }
			}
			tmsh::create {
			   ltm virtual http_vs
			   destination __vip_ip_address__
			   ip-protocol tcp
			   mask 255.255.255.255
			   pool http_pool
			   profiles replace-all-with {
				   http { }
				   tcp { }
			   }
			   source 0.0.0.0/0
			   translate-address enabled
			   translate-port enabled
			}
			iapp::template stop
      presentation: |
        section say_hello {
          message intro "Saying hello."
        }
```

You can make your *full_template* or *implementation* properties use our old friend ` str_replace  ` exactly like we did with cloud-init or the TMOS VE user_data.  

Just to make you are getting it, if you wanted to create an *implementation* property on F5::Sys::iAppCompositeTemplate resource which had HOT parameters or resource attributes in it, you could do something like this:

```
  implementation:
    str_replace:
      params:
        __find_this__: { get_attr: [ resource1, attribute1 ] }
      template: |
        #TMSH-VERSION: 11.6.0
        iapp::template start
        tmsh::create {
          ltm pool http_pool
          description "A pool of http servers __find_this__"
  ...

```

Once we have iApp templates we still have to use the F5::iApp::Service Heat resource to deploy them as application objects in TMOS.

Let's use an old friend the *f5.http* iApp which is on every Big-IP to show how to use the F5::iApp::Service resource. 

This time rather then using ` str_replace ` on the iApp template (where not publishing a template, we are using the wonderful f5.http iApp template with all its options), we will use ` str_replace ` to fill in the right answers to the f5.http iApp and deploy an application. 

We'll only fill in a few answers, leaving other template values hard-coded. You can see what all we could replace however.

```

resources:
  
  bigip:
    type: F5::BigIP::Device
    properties:
      ip: { get_param: f5_bigip_mgmt_ip }
      username: { get_param: f5_bigip_username }
      password: { get_param: f5_bigip_password }
      
  partition:
    type: F5::Sys::Partition
    depends_on: bigip
    properties:
      name: Common
      bigip_server: { get_resource: bigip }
  
  mobile_http_adc_deployment:
    type: F5::Sys::iAppService
    properties:
      name: lb_service
      bigip_server: { get_resource: bigip }
      partition: { get_resource: partition }
      template_name: /Common/f5.http
      traffic_group: /Common/traffic-group-1
      variables:
        str_replace:
          params:
            __vip_address__: { get_param: vip_address }
            __client_ssl_profile__: { get_param: client_ssl_profile }
          template: |
            [
                {
                  "name": "client__http_compression",
                  "encrypted": "no",
                  "value": "/Common/wan-optimized-compression"
                },
                {
                  "name": "client__standard_caching_without_wa",
                  "encrypted": "no",
                  "value": "/Common/optimized-caching"
                },
                {
                  "name": "client__tcp_wan_opt",
                  "encrypted": "no",
                  "value": "/Common/tcp-mobile-optimized"
                },
                {
                  "name": "monitor__anonymous",
                  "encrypted": "no",
                  "value": "yes"
                },
                {
                  "name": "monitor__frequency",
                  "encrypted": "no",
                  "value": "30"
                },
                {
                  "name": "monitor__http_method",
                  "encrypted": "no",
                  "value": "GET"
                },
                {
                  "name": "monitor__http_version",
                  "encrypted": "no",
                  "value": "http11"
                },
                {
                  "name": "monitor__monitor",
                  "encrypted": "no",
                  "value": "/#create_new#"
                },
                {
                  "name": "monitor__response",
                  "encrypted": "no",
                  "value": "none"
                },
                {
                  "name": "monitor__uri",
                  "encrypted": "no",
                  "value": "/"
                },
                {
                  "name": "net__client_mode",
                  "encrypted": "no",
                  "value": "wan"
                },
                {
                  "name": "net__route_to_bigip",
                  "encrypted": "no",
                  "value": "no"
                },
                {
                  "name": "net__same_subnet",
                  "encrypted": "no",
                  "value": "no"
                },
                {
                  "name": "net__server_mode",
                  "encrypted": "no",
                  "value": "lan"
                },
                {
                  "name": "net__snat_type",
                  "encrypted": "no",
                  "value": "automap"
                },
                {
                  "name": "net__vlan_mode",
                  "encrypted": "no",
                  "value": "enabled"
                },
                {
                  "name": "pool__addr",
                  "encrypted": "no",
                  "value": "__vip_address__"
                },
                {
                  "name": "pool__http",
                  "encrypted": "no",
                  "value": "/Common/http"
                },
                {
                  "name": "pool__lb_method",
                  "encrypted": "no",
                  "value": "least-connections-member"
                },
                {
                  "name": "pool__mask",
                  "encrypted": "no",
                  "value": "255.255.255.255"
                },
                {
                  "name": "pool__persist",
                  "encrypted": "no",
                  "value": "/Common/cookie"
                },
                {
                  "name": "pool__pool_to_use",
                  "encrypted": "no",
                  "value": "/#create_new#"
                },
                {
                  "name": "pool__port_secure",
                  "encrypted": "no",
                  "value": "443"
                },
                {
                  "name": "pool__redirect_port",
                  "encrypted": "no",
                  "value": "80"
                },
                {
                  "name": "pool__redirect_to_https",
                  "encrypted": "no",
                  "value": "yes"
                },
                {
                  "name": "pool__use_pga",
                  "encrypted": "no",
                  "value": "no"
                },
                {
                  "name": "server__ntlm",
                  "encrypted": "no",
                  "value": "/#do_not_use#"
                },
                {
                  "name": "server__oneconnect",
                  "encrypted": "no",
                  "value": "/Common/oneconnect"
                },
                {
                  "name": "server__slow_ramp_setvalue",
                  "encrypted": "no",
                  "value": "300"
                },
                {
                  "name": "server__tcp_lan_opt",
                  "encrypted": "no",
                  "value": "/Common/tcp-lan-optimized"
                },
                {
                  "name": "server__tcp_req_queueing",
                  "encrypted": "no",
                  "value": "no"
                },
                {
                  "name": "server__use_slow_ramp",
                  "encrypted": "no",
                  "value": "yes"
                },
                {
                  "name": "ssl__client_ssl_profile",
                  "encrypted": "no",
                  "value": "__client_ssl_profile__"
                },
                {
                  "name": "ssl__mode",
                  "encrypted": "no",
                  "value": "client_ssl"
                },
                {
                  "name": "ssl_encryption_questions__advanced",
                  "encrypted": "no",
                  "value": "yes"
                },
                {
                  "name": "ssl_encryption_questions__help",
                  "encrypted": "no",
                  "value": "hide"
                },
                {
                  "name": "stats__request_logging",
                  "encrypted": "no",
                  "value": "/#do_not_use#"
                }
              ]   
      lists:
        str_replace:
          params:
            __vip_network_name__: { get_param: vip_network_name }
          template: |
            [
              {
                "name": "irules__irules",
                "encrypted": "no"
              },
              {
                "name": "net__client_vlan",
                "encrypted": "no",
                "value": [
                  "__vip_network_name__"
                ]
              }
             ]
      tables:
        str_replace:
          params:
            __pool_member_1_ip__: { get_param: pool_member_1_ip }
            __pool_member_1_protocol_port__: { get_param: pool_member_1_protocol_port }
            __pool_member_1_weight__: { get_param: pool_member_1_weight }
            __pool_member_2_ip__: { get_param: pool_member_2_ip }
            __pool_member_2_protocol_port__: { get_param: pool_member_2_protocol_port }
            __pool_member_2_weight__: { get_param: pool_member_2_weight }
            __pool_member_3_ip__: { get_param: pool_member_3_ip }
            __pool_member_3_protocol_port__: { get_param: pool_member_3_protocol_port }
            __pool_member_3_weight__: { get_param: pool_member_3_weight }
            __web_app_fqdn__: { get_param: web_app_fqdn }
          template: |
            [
              {
                "name": "basic__snatpool_members"
              },
              {
                "name": "net__snatpool_members"
              },
              {
                "name": "optimizations__hosts"
              },
              {
                "name": "pool__hosts",
                "columnNames": [
                  "name"
                ],
                "rows": [
                  {
                    "row": [
                      "__web_app_fqdn__"
                    ]
                  }
                ]
              },
              {
                "name": "pool__members",
                "columnNames": [
                  "addr",
                  "port",
                  "connection_limit"
                ],
                "rows": [
                  {
                    "row": [
                      "__pool_member_1_ip__",
                      "__pool_member_1_protocol_port__",
                      "__pool_member_1_weight__"
                    ]
                  },
                  {
                    "row": [
                      "__pool_member_2_ip__",
                      "__pool_member_2_protocol_port__",
                      "__pool_member_2_weight__"
                    ]
                  },
                  {
                    "row": [
                      "__pool_member_3_ip__",
                      "__pool_member_3_protocol_port__",
                      "__pool_member_3_weight__"
                    ]
                  }
                ]
              },
              {
                "name": "server_pools__servers"
              }
            ]

```
Hopefully your getting this by now. 

####You can basically do any L4-L7 configuration of your BIG-IP if you take the time to write a declarative iApp and then use the F5 Heat resources to integrate them into the infrastrucutre!

*Oh Heat.. is there nothing that can't be done with you?*

<sub>
[Table of Contents](01_TOC.md) - Next [Exercise 14 - A Full Application Deployment](20_Exercise14.md) 
</sub>

<sup>
<b>July 2016</b></br>
n.menant@f5.com</br>
j.gruber@f5.com
</sup>
