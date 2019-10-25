---

copyright:
  years: 2017, 2019
lastupdated: "2019-10-03"

keywords: getting started with schematics, schematics tutorial, get started with terraform

subcollection: schematics

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:pre: .pre}
{:table: .aria-labeledby="caption"}
{:codeblock: .codeblock}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:deprecated: .deprecated}
{:download: .download}
{:preview: .preview}
{:external: target="_blank" .external}

# Getting started with {{site.data.keyword.bplong_notm}}
{: #getting-started}

Enable Infrastructure as Code (IaC) with {{site.data.keyword.bplong_notm}}, and start templatizing, provisioning, and managing {{site.data.keyword.cloud_notm}} resources in your {{site.data.keyword.cloud_notm}} environment by using Terraform configuration files. 
{: shortdesc}

## Create your Terraform template
{: #create-config}

Create a Terraform configuration file that specifies the {{site.data.keyword.cloud_notm}} resources that you want to provision with {{site.data.keyword.bplong_notm}}, and store the file in a GitHub repository to build your Terraform template. 
{: shortdesc}

In this getting started tutorial, you create a Terraform template with one Terraform configuration file that deploys a virtual server instance in a [Virtual Private Cloud (VPC)](/docs/vpc-on-classic?topic=vpc-on-classic-getting-started). You can use an existing Terraform template as part of this tutorial, but make sure that this template is stored in a GitHub or GitLab repository. 

A virtual server instance in a VPC incurs costs. Be sure to review the available plans for [VPC virtual server instances ![External link icon](../icons/launch-glyph.svg "External link icon")](https://cloud.ibm.com/vpc/provision/vs)before you proceed.
{: important}

**What is a Virtual Private Cloud (VPC) and what resources do I need?** </br> 
With a VPC, you can create your own space in {{site.data.keyword.cloud_notm}} so that you can run an isolated environment in the public cloud with custom network policies. To provision a virtual server in a VPC, you must set up the following infrastructure resources: 
- 1 VPC, in which you provision your VPC virtual server instance
- 1 security group and a rule for this security group to allow SSH connections to your virtual server instance
- 1 subnet to enable networking in your VPC
- 1 VPC virtual server instance 
- 1 floating IP address that you use to access your VPC virtual server instance over the public network

**What credentials do I need to provision a virtual server in a VPC?**</br>
The credentials that you need depend on the type of resource that you want to provision. To create a virtual server instance in a VPC, make sure that you have the [required permissions](/docs/vpc-on-classic?topic=vpc-on-classic-managing-user-permissions-for-vpc-resources) to create and work with VPC infrastructure. 

To connect to your virtual server instance, you must have an SSH key. You set up the SSH key as part of this tutorial. 

**Where can I find an overview of other supported resources in {{site.data.keyword.cloud_notm}}?**</br>
{{site.data.keyword.bplong_notm}} supports all resources that are defined by the {{site.data.keyword.cloud_notm}} Provider plug-in for Terraform. To find a full list of supported {{site.data.keyword.cloud_notm}} resources, see the [{{site.data.keyword.cloud_notm}} Provider reference ![External link icon](../icons/launch-glyph.svg "External link icon")](https://ibm-cloud.github.io/tf-ibm-docs/).

To create a configuration file for your VPC resources: 

1. Make sure that you have the [required permissions](/docs/vpc-on-classic?topic=vpc-on-classic-managing-user-permissions-for-vpc-resources) to create and work with VPC infrastructure. 
2. [Generate an SSH key](/docs/vpc-on-classic-vsi?topic=vpc-on-classic-vsi-ssh-keys). The SSH key is required to provision the VPC virtual server instance and you can use the SSH key to access your instance via SSH. After you created your SSH key, make sure to [upload this SSH key to your {{site.data.keyword.cloud_notm}} account](/docs/vpc-on-classic-vsi?topic=vpc-on-classic-vsi-managing-ssh-keys#managing-ssh-keys-with-ibm-cloud-console). 
3. Create your Terraform configuration `vpc.tf` file that includes all the VPC infrastructure resources that you need to successfully run a virtual server instance in a VPC. For more information about how to structure a Terraform configuration file, see [Creating a Terraform configuration](/docs/schematics?topic=schematics-create-tf-config). 
   ```
   variable "ssh_key" {}

   provider "ibm" {
     generation = 1
   }

   locals {
     BASENAME = "schematics" 
     ZONE     = "us-south-1"
   }

   resource ibm_is_vpc "vpc" {
     name = "${local.BASENAME}-vpc"
   }

   resource ibm_is_security_group "sg1" {
     name = "${local.BASENAME}-sg1"
     vpc  = "${ibm_is_vpc.vpc.id}"
   }

   # allow all incoming network traffic on port 22
   resource "ibm_is_security_group_rule" "ingress_ssh_all" {
     group     = "${ibm_is_security_group.sg1.id}"
     direction = "inbound"
     remote    = "0.0.0.0/0"                       

     tcp = {
       port_min = 22
       port_max = 22
     }
   }

   resource ibm_is_subnet "subnet1" {
     name = "${local.BASENAME}-subnet1"
     vpc  = "${ibm_is_vpc.vpc.id}"
     zone = "${local.ZONE}"
     total_ipv4_address_count = 256
   }

   data ibm_is_image "ubuntu" {
     name = "ubuntu-18.04-amd64"
   }

   data ibm_is_ssh_key "ssh_key_id" {
     name = "${var.ssh_key}"
   }

   data ibm_resource_group "group" {
     name = "default"
   }

   resource ibm_is_instance "vsi1" {
     name    = "${local.BASENAME}-vsi1"
     resource_group = "${data.ibm_resource_group.group.id}"
     vpc     = "${ibm_is_vpc.vpc.id}"
     zone    = "${local.ZONE}"
     keys    = ["${data.ibm_is_ssh_key.ssh_key_id.id}"]
     image   = "${data.ibm_is_image.ubuntu.id}"
     profile = "cc1-2x4"

     primary_network_interface = {
       subnet          = "${ibm_is_subnet.subnet1.id}"
       security_groups = ["${ibm_is_security_group.sg1.id}"]
     }
   }

   resource ibm_is_floating_ip "fip1" {
     name   = "${local.BASENAME}-fip1"
     target = "${ibm_is_instance.vsi1.primary_network_interface.0.id}"
   }

   output sshcommand {
     value = "ssh root@${ibm_is_floating_ip.fip1.address}"
   }
   ```
   {: codeblock}
   
   <table>
   <caption>Understanding the configuration file components</caption>
   <thead>
   <th colspan=2><img src="images/idea.png" alt="Idea icon"/> Understanding the configuration file components</th>
   </thead>
   <tbody>
     <tr>
       <td><code>variable.ssh_key</code></td>
       <td>A variable declaration for the name of the SSH key that you uploaded to your {{site.data.keyword.cloud_notm}} account. You enter the value for your variable when you create your workspace in {{site.data.keyword.bplong_notm}}.</td>
     </tr>
     <tr>
       <td><code>provider.generation</code></td>
       <td>Enter <strong>1</strong> to provision VPC on Classic infrastructure resources. </td>
     </tr>
   <tr>
   <td><code>locals.BASENAME</code></td>
   <td>Enter a name that you want to append to the name of all VPC infrastructure resources that you create with this configuration file. </td>
   </tr>
     <tr>
       <td><code>locals.ZONE</code></td>
       <td>Enter a supported VPC zone where you want to create your resources. To find available zones, run <code>ibmcloud is zones</code>.  </td>
     </tr>
     <tr>
       <td><code>resource.ibm_is_vpc.name</code></td>
       <td>Enter a name for your VPC. In this example, you use <code>locals.BASENAME</code> to create part of the name. For example, if your base name is `schematics`, the name of your VPC is set to `schematics-vpc`. Keep in mind that the name of your VPC must be unique within your {{site.data.keyword.cloud_notm}} account. </td>
     </tr>
     <tr>
       <td><code>resource.ibm_is_security_group.name</code></td>
       <td>Enter a name for the security group that you create for your VPC. In this example, you use <code>locals.BASENAME</code> to create part of the name. For example, if your base name is `schematics`, the name of your security group is set to `schematics-sg1`.  </td>
     </tr>
     <tr>
       <td><code>resource.ibm_is_security_group.vpc</code></td>
       <td>Enter the ID of the VPC for which you want to create the security group. In this example, you reference the ID of the VPC that you create with the <code>ibm_is_vpc</code> resource in the same configuration file.  </td>
     </tr>
     <tr>
       <td><code>resource.ibm_is_security_group_rule.group</code></td>
       <td>Enter the ID of the security group for which you want to create a security group rule. In this example, you reference the ID of the security group that you create with the <code>ibm_is_security_group</code> resource in the same configuration file.  </td>
     </tr>
      <tr>
       <td><code>resource.ibm_is_security_group_rule.direction</code></td>
       <td>Specify if the security group rule is applied to incoming or outgoing network traffic. Choose <strong>inbound</strong> to specify a rule for incoming network traffic, and <strong>outbound</strong> to specify a rule for outgoing network traffic. </td>
     </tr>
     <tr>
       <td><code>resource.ibm_is_security_group_rule.remote</code></td>
       <td>Enter the IP address range, for which the security group rule is applied. In this example, <code>0.0.0.0/0</code> allows incoming network traffic from all IP addresses. </td>
     </tr>
     <tr>
       <td><code>resource.ibm_is_security_group_rule.tcp</code></td>
       <td>Enter the TCP port range that you want to open in your security group rule. If you want to open up a single port, enter the same port number in <code>port_min</code> and <code>port_max</code>. </td>
     </tr>
     <tr>
       <td><code>resource.ibm_is_subnet.name</code></td>
       <td>Enter a name for your subnet. In this example, you use <code>locals.BASENAME</code> to create part of the name. For example, if your base name is `schematics`, the name of your subnet is set to `schematics-subnet1`.  </td>
     </tr>
      <tr>
       <td><code>resource.ibm_is_subnet.vpc</code></td>
       <td>Enter the ID of the VPC for which you want to create the subnet. In this example, you reference the ID of the VPC that you create with the <code>ibm_is_vpc</code> resource in the same configuration file. </td>
     </tr>
     <tr>
       <td><code>resource.ibm_is_subnet.zone</code></td>
       <td>Enter the zone in which you want to create the subnet. In this example, you use <code>locals.ZONE</code> as the name for your zone. </td>
     </tr>
     <tr>
       <td><code>resource.ibm_is_subnet.</code></br><code>total_ipv4_address_count</code></td>
       <td>Enter the number of IPv4 IP addresses that you want to have in your subnet. </td>
     </tr>
     <tr>
       <td><code>data.ibm_is_image.name</code></td>
       <td>Enter the name of the operating system that you want to install on your VPC virtual server instance. In this example, you retrieve the ID of the operating system so that you can use the ID when you create the VPC virtual server instance. For supported image names, run <code>ibmcloud is images</code>. </td>
     </tr>
     <tr>
       <td><code>data.ibm_is_ssh_key.name</code></td>
       <td>Enter the name of the SSH key that you uploaded to your {{site.data.keyword.cloud_notm}} account. In this example, you reference a variable for the SSH key name so that you don't have to enter the SSH key name into your Terraform configuration file directly. You can enter the values for all variable declarations when you create your workspace in {{site.data.keyword.bpshort}}. The <code>data.ibm.is.ssh_key</code> Terraform resource is used to retrieve the ID of an existing SSH key so that you can use this ID when you create your VPC virtual server instance. </td>
     </tr>
      <tr>
       <td><code>data.ibm_resource_group.name</code></td>
       <td>Enter the name of the resource group in your {{site.data.keyword.cloud_notm}} account that you want to use for your resources. Make sure that you have the right [permissions to provision {{site.data.keyword.cloud_notm}} resources in a resource group](/docs/schematics?topic=schematics-access). </td>
     </tr>
     <tr>
       <td><code>resource.ibm_is_instance.name</code></td>
       <td>Enter the name of the VPC virtual server instance that you want to create. In this example, you use <code>locals.BASENAME</code> to create part of the name. For example, if your base name is `test`, the name of your VPC virtual server instance is set to `test-vsi1`.  </td>
     </tr>
     <tr>
       <td><code>resource.ibm_is_instance.resource_group</code></td>
       <td>Enter the ID of the resource group that you want to use for your VPC virtual server instance. In this example, you retrieve the ID from the <code>ibm_resource_group</code> data source of this configuration file. Terraform uses the name of the resource group that you define in your data source object to look up information about the resource group in your {{site.data.keyword.cloud_notm}} account. </td>
     </tr>
     <tr>
       <td><code>resource.ibm_is_instance.vpc</code></td>
       <td>Enter the ID of the VPC in which you want to create the VPC virtual server instance. In this example, you reference the ID of the VPC that you create with the <code>ibm_is_vpc</code> resource in the same configuration file.  </td>
     </tr>
     <tr>
       <td><code>resource.ibm_is_instance.zone</code></td>
       <td>Enter the zone in which you want to create the VPC virtual server instance. In this example, you use <code>locals.ZONE</code> as the name for your zone. </td>
     </tr>
     <tr>
       <td><code>resource.ibm_is_instance.keys</code></td>
       <td>Enter the UUID of the SSH key that you uploaded to your {{site.data.keyword.cloud_notm}} account. In this example, you retrieve the UUID from the <code>ibm_is_ssh_key</code> data source of this configuration file. Terraform uses the name of the SSH key that you define in your data source object to look up information about the SSH key in your {{site.data.keyword.cloud_notm}} account.</td>
     </tr>
     <tr>
       <td><code>resource.ibm_is_instance.image</code></td>
       <td>Enter the ID of the image that represents the operating system that you want to install on your VPC virtual server instance. In this example, you retrieve the ID from the <code>ibm_is_image</code> data source of this configuration file. Terraform uses the name of the image that you define in your data source object to look up information about the image in the {{site.data.keyword.cloud_notm}} infrastructure portfolio. </td>
     </tr>
     <tr>
       <td><code>resource.ibm_is_instance.profile</code></td>
       <td>Enter the name of the profile that you want to use for your VPC virtual server instance. For supported profiles, run <code>ibmcloud is instance-profiles</code>. </td>
     </tr>
     <tr>
       <td><code>resource.ibm_is_instance.</code></br><code>primary_network_interface.subnet</code></td>
       <td>Enter the ID of the subnet that you want to use for your VPC virtual server instance. In this example, you use the <code>ibm_is_subnet</code> resource in this configuration file to retrieve the ID of the subnet.   </td>
     </tr>
     <tr>
       <td><code>resource.ibm_is_instance.</code></br><code>primary_network_interface.security_groups</code></td>
       <td>Enter the ID of the security group that you want to apply to your VPC virtual server instance. In this example, you use the <code>ibm_is_security_group</code> resource in this configuration file to retrieve the ID of the security group.   </td>
     </tr>
     <tr>
       <td><code>resource.ibm_is_floating_ip.name</code></td>
       <td>Enter a name for your floating IP resource. In this example, you use <code>locals.BASENAME</code> to create part of the name. For example, if your base name is `schematics`, the name of your floating IP resource is set to `schematics-fip1`.   </td>
     </tr>
     <tr>
       <td><code>resource.ibm_is_floating_ip.target</code></td>
       <td>Enter the ID of the network interface where you want to allocate the floating IP addresses. In this example, you use the <code>ibm_is_instance</code> resource to retrieve the ID of the primary network interface.   </td>
     </tr>
     <tr>
       <td><code>output.ssh_command.value</code></td>
       <td>Build the SSH command that you need to run to connect to your VPC virtual server instance. In this example, you use the <code>ibm_is_floating_ip</code> resource to retrieve the floating IP address that is assigned to your VPC virtual server instance.  </td>
     </tr>
   </tbody>
   </table>

4. Store this configuration file in a GitHub repository and name it `vpc.tf` to build your Terraform template. 

## Setting up your workspace
{: #setup-workspace}

Create a workspace in {{site.data.keyword.bplong_notm}} that points to the GitHub repository that hosts your Terraform template. 
{: shortdesc}

**Before you begin** 
- Make sure that you have a [Terraform template in a GitHub repository](#create-config) that you can use for your workspace.
- Make sure that you are [assigned the correct permissions](/docs/schematics?topic=schematics-access) in {{site.data.keyword.cloud_notm}} Identity and Access Management to create the workspace and deploy resources. 

**To create a workspace**: 
1. From the {{site.data.keyword.cloud_notm}} menu, select [**{{site.data.keyword.bpshort}}** ![External link icon](../icons/launch-glyph.svg "External link icon")](https://cloud.ibm.com/schematics/overview). 
2. Click **Create a workspace**. 
3. Configure your workspace. 
   1. Enter a descriptive name for your workspace. When you create a workspace for your own Terraform template, consider including the microservice component that you set up with your Terraform template and the {{site.data.keyword.cloud_notm}} environment where you want to deploy your resources in your name. For more information about how to structure your workspaces, see [Designing your workspace structure](/docs/schematics?topic=schematics-workspace-setup#structure-workspace).
   2. Optional: Enter tags for your workspace. You can use the tags later to find your workspaces more easily. 
   3. Optional: Enter a description for your workspace.
   4. Enter the link to your GitHub repository. The link can point to the `master` branch, other branches, or subdirectories.
      - Example for `master` branch: https://github.com/myorg/myrepo
      - Example for other branches: https://github.com/myorg/myrepo/tree/mybranch
      - Example for subdirectory: https://github.com/mnorg/myrepo/tree/mybranch/mysubdirectory
   
   5. If you want to use a private GitHub repository, enter your personal access token. The personal access token is used to authenticate with your GitHub repository to access your Terraform template. For more information, see [Creating a personal access token for the command line ![External link icon](../icons/launch-glyph.svg "External link icon")](https://help.github.com/en/articles/creating-a-personal-access-token-for-the-command-line). 
   6. Click **Retrieve input variables**. {{site.data.keyword.bplong_notm}} automatically parses through your template to find variable declarations. 
   7. In the **Input variables** section, enter the name of the SSH key that you uploaded to your {{site.data.keyword.cloud_notm}} account. 
4. Click **Create** to create your workspace. When you create the workspace, all Terraform configuration files of your template are loaded into {{site.data.keyword.bplong_notm}}, but your resources are not yet deployed to {{site.data.keyword.cloud_notm}}. 


## Provision your {{site.data.keyword.cloud_notm}} resources
{: #provision-resources}

Use {{site.data.keyword.bplong_notm}} to run the infrastructure code in your Terraform configuration files, and to provision your {{site.data.keyword.cloud_notm}} resources. 
{: shortdesc}

Before you begin, set up your workspace in [{{site.data.keyword.bplong_notm}}](#setup-workspace). 

**To provision your resources**: 

1. From the workspace details page, click **Generate plan** to create a Terraform execution plan. This action equals the `terraform plan` command. After you click the button, the workspace **Activity** page is opened.  
2. Click **View log** to review the log files of your Terraform execution plan. The execution plan includes a summary of {{site.data.keyword.cloud_notm}} resources that must be created, modified, or deleted to achieve the state that you described in your Terraform template. If you have syntax errors in your configuration files, you can review the error message in the log file. 
3. Go back to the workspace details page and apply your Terraform template by clicking **Apply plan**. This action equals the `terraform apply` command. After you click the button, the workspace **Activity** page opens and {{site.data.keyword.bplong_notm}} starts provisioning, modifying, or deleting your {{site.data.keyword.cloud_notm}} resources based on what actions were identified in the execution plan. Depending on the type and number of resources that you want to provision, this process might take a few minutes, or even up to hours to complete. During this time, you cannot make changes to your workspace. 
4. Review the log file to ensure that no errors occurred during the provisioning, modification, or deletion process. 
5. From the menu, select **Resources** to find a summary of {{site.data.keyword.cloud_notm}} resources that are available in your {{site.data.keyword.cloud_notm}} account. Use the link to the resource dashboard to see more details about the individual resource.

## What's next? 
{: #whats-next}

After you provisioned your resources in {{site.data.keyword.cloud_notm}}, choose between the following options: 
- Learn more about [{{site.data.keyword.bplong_notm}}](/docs/schematics?topic=schematics-about-schematics) and the benefits of using the service. 
- Review tips and tricks for how to [structure your Terraform configuration file](/docs/schematics?topic=schematics-create-tf-config).
- Set up a [GitHub repository and workspace structure](/docs/schematics?topic=schematics-workspace-setup) for your microservices. 
- Explore other {{site.data.keyword.cloud_notm}} resources that you can provision by reviewing the [{{site.data.keyword.cloud_notm}} Provider plug-in reference ![External link icon](../icons/launch-glyph.svg "External link icon")](https://ibm-cloud.github.io/tf-ibm-docs/).
- [Install the {{site.data.keyword.bplong_notm}} CLI](/docs/schematics?topic=schematics-setup-cli) to automate the provisioning of your resources in {{site.data.keyword.cloud_notm}}. 