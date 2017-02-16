# azure-template-add-node
This ARM template creates a set of worker nodes ready to be added to an existing CM + CDH cluster created by Cloudera's EDH Azure Marketplace offering or Cloudera Director.

This template assumes that the Virtual Network, Availability Set, and Network Security Group have all been created beforehand.

This template can be used to add nodes to Cloudera Director (CD) created clusters - all it requires is that CM is running.

## How to use this template to add nodes
Given an existing Cloudera Manager (CM) you can add nodes to it by running this template.

### Run the template with Azure CLI

1. Host the contents of the `scripts` directory, keeping the scripts in a `scripts` directory (for the template to work, the scripts must be in a directory named `scripts`).

1. Update your local `scriptsUri` field in `azuredeploy.json` to point to where the code is hosted.

1. Update your local `azuredeploy.parameters.json` file.

1. Run
```
azure group deployment create -f ~/<path-to>/cloudera-add-node/azuredeploy.json -e ~/<path-to>/cloudera-add-node/azuredeploy.parameters.json -g <resource-group-name>
```


### Fill in the required fields

Most of the fields are references to existing resources. For simplicity it is recommended to use the same resources as the other worker nodes in the cluster.


| Field name  | Description  |
|---|---|
| resource group | The Resource Group to deploy into. This can be a new or existing Resource Group. |
| admin username | The Username for the VM. |
| admin password | The Password for the VM. |
| availability set name | The name of the Availability Set to use. This must exist. Note that v1 and v2 instances can't share an Availability Set. |
| virtual machine size | The VM size. Note that v1 and v2 instances can't share an Availability Set. |
| storage account type | The type of Storage Account to use. |
| virtual machine count | The number of VMs to allocate. |
| network security group resource group name | The Resource Group holding the Network Security Group. This must exist and the Network Security Group must exist as well. |
| network security group | The Network Security Group to use. This must exist. |
| virtual network resource group name | The Resource Group holding the Virtual Network. This must exist and the Virtual Network must exist as well. |
| virtual network name | The Virtual Network to use. This must exist. No public IP addresses are created and so for simplicity the new node should be deployed into the same Virtual Network as CM, so that the node can be added to CM easily. |
| subnet name | The Subnet to use. This must exist and must be a subnet of the Virtual Network. For simplicity this subnet should be the same as the other worker nodes in the cluster. |
| dns label prefix | This becomes part of the VM name and must be unique across **all** runs. For example, each time running this template increment a counter: `add-node-batch-1`, `add-node-batch-2`, etc. If the DNS Label Prefix is re-used the build will "succeed" but no VMs or other resources will be created. |
| dns label suffix | The DNS Label Suffix must be the same as the DNS Label Suffix that the existing cluster nodes use. This is configured by the DNS Server configurations. One way to find it is to run the command `hostname -d` on CM or other worker nodes. By default the Cloudera on Azure docs set up a DNS Server with `cdh-cluster.internal` as the DNS Label Suffix. |

### Add the VMs to Cloudera Manager
For each VM created:

1. Go to portal.azure.com, find the RG used, and the VM name, select the VM, then click "settings" > Network interfaces. This will give you the private IP of the host, as the hostname won't resolve - yet.
1. `ssh` to the CM host.
1. `ssh` from the CM host to the private IP of the newly added host.
1. As root, run the same instance bootstrap script you used for creating the cluster via Cloudera Director. You can refer to the [os-generic-bootstrap script](https://github.com/cloudera/director-scripts/tree/master/azure-bootstrap-scripts) as an example.<br>
The command `hostname` will work on the host, however `hostname -f`, `hostname -i` wont.

Then go to CM and add the hosts.


## Other Notes

**Resource Naming Pattern**

* DNS prefix + copyIndex() is use to create a unique seed for naming resources to avoid name collisions.
* VirtualMachine name: `prefix-copyIndex()-suffix`
* StorageAccount: `uniqueString(seed+copyIndex()) + "sa"`
* NetworkInterface: `uniqueString(seed+copyIndex()) + "nic"`

If there is a naming collision (e.g. if the same DNS Label Prefix is used again) then the deployment will succeed but no resources will be created.
