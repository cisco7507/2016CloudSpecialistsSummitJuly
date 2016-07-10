# 2016 Global Cloud Specialists Summit July

Summit Presentations


#Exercise 1 - Discovering Your Resources

HOT with resources to orchestrate is useless. So let's see what your working with.

####Query Heat for a List of Resources

#####Through the Horizon Web Interface Heat Client

**Step 1. Navigate to Orchestration -> Resouce Types**
**Step 2. Select the resource `OS::Glance::Image`**

#####Through the CLI Heat Client

**Step 1. Get a list of Resource Types**

```
# heat resource-type-list
```

Step 2. Show OS::Glance::Image

```
# heat resource-type-show OS::Glance::Image
{
  "support_status": {
    "status": "SUPPORTED", 
    "message": null, 
    "version": "2014.2", 
    "previous_status": null
  }, 
  "attributes": {
    "show": {
      "type": "map", 
      "description": "Detailed information about resource."
    }
  }, 
  "properties": {
    "name": {
      "type": "string", 
      "required": false, 
      "update_allowed": false, 
      "description": "Name for the image. The name of an image is not unique to a Image Service node.", 
      "immutable": false
    }, 
    "container_format": {
      "description": "Container format of image.", 
      "required": true, 
      "update_allowed": false, 
      "type": "string", 
      "immutable": false, 
      "constraints": [
        {
          "allowed_values": [
            "ami", 
            "ari", 
            "aki", 
            "bare", 
            "ova", 
            "ovf"
          ]
        }
      ]
    }, 
    "min_ram": {
      "description": "Amount of ram (in MB) required to boot image. Default value is 0 if not specified and means no limit on the ram size.", 
      "required": false, 
      "update_allowed": false, 
      "type": "integer", 
      "immutable": false, 
      "constraints": [
        {
          "range": {
            "min": 0
          }
        }
      ]
    }, 
    "disk_format": {
      "description": "Disk format of image.", 
      "required": true, 
      "update_allowed": false, 
      "type": "string", 
      "immutable": false, 
      "constraints": [
        {
          "allowed_values": [
            "ami", 
            "ari", 
            "aki", 
            "vhd", 
            "vmdk", 
            "raw", 
            "qcow2", 
            "vdi", 
            "iso"
          ]
        }
      ]
    }, 
    "protected": {
      "type": "boolean", 
      "required": false, 
      "update_allowed": false, 
      "description": "Whether the image can be deleted. If the value is True, the image is protected and cannot be deleted.", 
      "immutable": false
    }, 
    "location": {
      "type": "string", 
      "required": true, 
      "update_allowed": false, 
      "description": "URL where the data for this image already resides. For example, if the image data is stored in swift, you could specify \"swift://example.com/container/obj\".", 
      "immutable": false
    }, 
    "min_disk": {
      "description": "Amount of disk space (in GB) required to boot image. Default value is 0 if not specified and means no limit on the disk size.", 
      "required": false, 
      "update_allowed": false, 
      "type": "integer", 
      "immutable": false, 
      "constraints": [
        {
          "range": {
            "min": 0
          }
        }
      ]
    }, 
    "is_public": {
      "description": "Scope of image accessibility. Public or private. Default value is False means private.", 
      "default": false, 
      "required": false, 
      "update_allowed": false, 
      "type": "boolean", 
      "immutable": false
    }, 
    "id": {
      "type": "string", 
      "required": false, 
      "update_allowed": false, 
      "description": "The image ID. Glance will generate a UUID if not specified.", 
      "immutable": false
    }
  }, 
  "resource_type": "OS::Glance::Image"
}



```

You will notice the resouce has *properties* (inputs) and *attributes* (outputs)

If we wanted to use this resource in a HOT, then we could declare it according to it's attributes and properties:

```
resources:
  
  my_ubuntu_image:
    type: OS::Glance::Image
    properties:
        name: MyNewImage
        container_format: bare
        disk_format: qcow2
        is_public: true
        location: https://cloud-images.ubuntu.com/xenial/current/xenial-server-cloudimg-amd64-disk1.img
        min_disk: 20
        min_ram: 512

outputs:

  my_ubuntu_image_details:
    description: Glance Image details
    value: { get_attr: [my_ubuntu_image, show] }
    
        
``` 

You will notice the use of the ` get_attr ` HOT function to get the attribute from our named resource. The syntax for the ` get_attr ` function is:

```
{ get_attr: [<resource name>, <attribute name> ]}
```


<sub>
[Table of Contents](01_TOC.md) - Next [Exercise 2 - Your First Heat Orchestration](07_Exercise2.md) 
</sub>

<sup>
<b>July 2016</b></br>
n.menant@f5.com</br>
j.gruber@f5.com
</sup>
