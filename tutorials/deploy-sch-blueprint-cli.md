---

copyright:
  years: 2017, 2022
lastupdated: "2022-11-05"

keywords: deploy schematics blueprint, blueprint cli deployment, deploy schematics blueprint cli, 

subcollection: schematics

content-type: tutorial
services: schematics
account-plan:
completion-time: 30m

---

{{site.data.keyword.attribute-definition-list}}

{{site.data.keyword.bpshort}} blueprints is a [beta feature](/docs/schematics?topic=schematics-bp-beta-limitations) that is available for evaluation and testing purposes. It is not intended for production usage. Refer to the list of [limitations](/docs/schematics?topic=schematics-bp-beta-limitations) for the beta release.
{: beta}

# Deploying a {{site.data.keyword.bpshort}} blueprint using the command line
{: #deploy-schematics-blueprint-cli}
{: toc-content-type="tutorial"}
{: toc-services="schematics"}
{: toc-completion-time="30m"}

Use an {{site.data.keyword.IBM}} provided sample to deploy a scalable multi-module {{site.data.keyword.bpshort}} blueprint template from the command line.
{: shortdesc}

In this tutorial, you deploy a {{site.data.keyword.bpshort}} blueprint using the {{site.data.keyword.bpshort}} command line. The tutorial demonstrates the blueprints lifecycle of creating cloud resources with a blueprint-managed cloud environment in {{site.data.keyword.bpshort}}, through to deleting the environment. 

Before your begin

- Install and login to the [{{site.data.keyword.cloud_notm}} command line](/docs/schematics?topic=schematics-setup-cli#install-schematics-cli).
- Select the {{site.data.keyword.cloud_notm}} region that you want to use to manage your {{site.data.keyword.bpshort}} blueprint environment. Set the region by running the command `ibmcloud target -r <us-south>`.
- Install the [{{site.data.keyword.bpshort}} command-line](/docs/schematics?topic=schematics-setup-cli#install-schematics-plugin) plug-in, or [update the command line plug-in](/docs/schematics?topic=schematics-setup-cli#schematics-cli-update) to access the {{site.data.keyword.bpshort}} blueprints commands.
- Check that you have the right [permissions](/docs/schematics?topic=schematics-access#blueprint-permissions) to create blueprints.

## Select a blueprint template
{: #select-schematics-blueprint-cli}
{: step}

This tutorial uses a [sample blueprint](https://github.com/Cloud-Schematics/blueprint-basic-example){: external} to  create two blueprint modules referencing a resource group and creating an {{site.data.keyword.cos_full}} instance and bucket. No costs are incurred deploying this example. 

## Create the blueprint configuration
{: #create-schematics-blueprint-cli}
{: step}

Create your blueprint environment by using CLI command [`ibmcloud schematics blueprint config create`](/docs/schematics?topic=schematics-schematics-cli-reference#schematics-blueprint-create). Use the following parameters to create a blueprint.
{: shortdesc}

For {{site.data.keyword.bpshort}} blueprints, the [{{site.data.keyword.bpshort}} plug-in](/docs/schematics?topic=schematics-setup-cli#install-schematics-plugin) version must be greater than the `1.12.3`.
{: important}

- The Name of the blueprint: `Blueprint_basic`
- The {{site.data.keyword.bpshort}} management resource group: \<default-resource-group-name\>  
- The blueprint URL: `https://github.com/Cloud-Schematics/blueprint-basic-example`
- The blueprint file: `basic-blueprint.yaml`
- An Input file URL: `https://github.com/Cloud-Schematics/blueprint-basic-example`
- An Input file: `basic-input.yaml` 


For all the blueprints commands, syntax, and detailed option flags, refer to, [blueprints commands](/docs/schematics?topic=schematics-schematics-cli-reference#blueprints-cmd).
{: important}

### Syntax
{: #step1-syntax}
The name of default resource group for the account is required for the blueprint creation. The blueprint config will be created in this resource group. 

```sh
ibmcloud resource groups --default

Retrieving default resource group under account 12345678901234567890123456789012 as anon@anon.com...
OK
Name      ID                                 Default Group   State
Default   aac37f57b20142dba1a435c70aeb12df   true            ACTIVE
```
{: pre}

In the `blueprint config create` command, replace `<default-resource-group-name>` with the name of the default resource group.   

```sh
ibmcloud schematics blueprint config create -name Blueprint_Basic -resource-group \<default-resource-group-name\>  -bp-git-url https://github.com/Cloud-Schematics/blueprint-basicexample -bp-git-file basic-blueprint.yaml -input-git-url https://github.com/Cloud-Schematics/blueprint-basic-example -input-git-file basic-input.yaml 
```
{: pre}



### Output
{: #step1-output}

```text
Created Blueprint ID: Blueprint_Basic.eaB.5cd9

Modules to be created
SNO Type Name
1 terraform basic-resource-group
2 terraform basic-cos-storage

Blueprint job running us-east.JOB.Blueprint_Basic.bb553ac5
Waiting:0 Draft:0 Connecting:0 In Progress:0 Inactive:2 Active:0
Failed:0
Type Name Status Job ID
Blueprint Blueprint_Basic CREATE_SUCCESS useast.JOB.Blueprint_Basic.bb553ac5
terraform basic-resource-group INACTIVE
terraform basic-cos-storage INACTIVE

Blueprint ID Blueprint_Basic.eaB.5cd9 create_success at 2022-08-03
21:19:16

OK
```
{: screen}

The output shows that the modules `basic-resource-group` and `basic-cos-storage` are created and are in `Inactive` status to indicate that the modules are initialized and ready for Terraform operations to be performed.

Record the ID of the blueprint to use in the later commands.
{: important}

Refer to [troubleshooting blueprint config create failures](/docs/schematics?topic=schematics-bp-create-fails) for details on debugging create failures. 

## Apply blueprint to deploy environment 
{: #install-schematics-blueprint-cli}
{: step}

Use the [`ibmcloud schematics blueprint run apply`](/docs/schematics?topic=schematics-schematics-cli-reference#schematics-blueprint-create) command to run the Terraform Apply operations for the Terraform modules specified in the blueprint template. This operation creates cloud resources and may incur costs. Insert the ID saved from the [output of the create](#create-schematics-blueprint-cli) command.

```sh
ibmcloud schematics blueprint run apply --id Blueprint_Basic.eaB.5cd9
```
{: pre}

### Output
{: #step3-output}

```text
Modules to be installed
SNO Type Name Status
1 terraform basic-resource-group INACTIVE
2 terraform basic-cos-storage INACTIVE

Blueprint job running us-east.JOB.Blueprint_Basic.e4081308 
Waiting:0 Draft:0 Connecting:0 In Progress:0 Inactive:0 Active:2
Failed:0

Type Name Status Job ID
Blueprint Blueprint_Basic FULFILMENT_SUCCESS useast.JOB.Blueprint_Basic.e4081308
terraform basic-resource-group ACTIVE
terraform basic-cos-storage ACTIVE

Blueprint ID Blueprint_Basic.eaB.5cd9 fulfilment_success at 2022-08-03
21:24:26
OK
```
{: screen}

The output shows that the workspaces `basic-resource-group` and `basic-cos-storage` are now in `Active` status to indicate that the Terraform Apply run successfully and Terraform resources is created.

Refer to [troubleshooting blueprint apply failures](/docs/schematics?topic=schematics-bp-apply-fails) for details on debugging apply failures.

## View blueprint job logs
{: #view-schematics-blueprint-cli}
{: step} 

Optional: Run the [`ibmcloud schematics blueprint job logs`](/docs/schematics?topic=schematics-schematics-cli-reference#schematics-blueprint-job-logs) command to view the blueprint job logs.

```sh
ibmcloud schematics blueprint job logs --id useast.JOB.Blueprint_Basic.e4081308
```
{: pre}

### Output
{: #viewing-output}

```text
 2022/08/03 15:51:17 ----- New Environment Action -----
 2022/08/03 15:51:17 Request: environmentId=Blueprint_Basic.eaB.5cd9,
account=16a85b7b99a6622e7c186fb6503781a0,
owner=kgurudut@in.ibm.com, requestID=20c30a3b-fbb1-4f41-
aa71-5ed337047ec2
 2022/08/03 15:51:17 Related Job: jobID=useast.JOB.Blueprint_Basic.e4081308
 2022/08/03 15:51:32 --- Ready to execute the environment flow install
command ---
 2022/08/03 15:51:32 Processing Module Entry us-east.workspace.basicresource-group.9ea437b2
 2022/08/03 15:51:33 Module Status for ModuleID=us-east.workspace.basicresource-group.9ea437b2 is INACTIVE
 2022/08/03 15:52:31 Install activity completed for module ModuleID=useast.workspace.basic-resource-group.9ea437b2
 2022/08/03 15:52:33 Status for module ModuleID=us-east.workspace.basicresource-group.9ea437b2 is ACTIVE
 2022/08/03 15:52:35 Processing Module Entry us-east.workspace.basic-cosstorage.adb78a38
 2022/08/03 15:52:37 Module Status for ModuleID=us-east.workspace.basiccos-storage.adb78a38 is INACTIVE
 2022/08/03 15:54:06 Install activity completed for module ModuleID=useast.workspace.basic-cos-storage.adb78a38
 2022/08/03 15:54:08 Status for module ModuleID=us-east.workspace.basiccos-storage.adb78a38 is ACTIVE
 2022/08/03 15:54:10 ENVIRONMENT_INSTALL -
ENVIRONMENT_SYSTEM_STATEENUM_FULFILMENT_SUCCESS
 2022/08/03 15:54:11 Done with the environment install flow job
OK
```
{: screen}


## Access and test the blueprint environment resources
{: #review-schematics-blueprint}
{: step}

After a successful deployment of the blueprint, review the deployed blueprint through the CLI or UI.  

### Using the cloud UI 
{: #review-schematics-blueprint-ui}

From the [{{site.data.keyword.bpshort}} blueprint list](https://cloud.ibm.com/schematics/blueprints){: external}, select the provisioned blueprint to view the workspaces and cloud resources. Review UI tabs such as **Overview**, **Modules**, **Resources**, **Variables**, **Jobs history**, and **Settings** for the computed output values of the blueprint. 

### Using the CLI
{: #review-schematics-blueprint-cli}

Use the [`ibmcloud schematics blueprint get`](/docs/schematics?topic=schematics-schematics-cli-reference#schematics-blueprint-get) command to display the details about the blueprint and the deployed environment. Insert the ID saved from the [output of the create](#create-schematics-blueprint-cli) command.

```sh
ibmcloud schematics blueprint get -id blueprint_id
```
{: pre}

The computed output values of the blueprint can be retrieved by using the `--profile outputs` option.

```sh
ibmcloud schematics blueprint get --id Blueprint_Basic.eaB.5cd9 --level outputs
```
{: pre}

```text
BLUEPRINT

Name Blueprint_Basic
ID Blueprint_Basic.eaB.5cd9
Status Normal
Location us-east
Creator kgurudut@in.ibm.com
Last modified 2022-08-03T15:54:14.993Z

MODULES
SNO   Workspace Name        Workspace ID                                    Status  Location  Updated
1     basic-resource-group  us-east.workspace.basic-resourcegroup.9ea437b2  ACTIVE  us-east
2     basic-cos-storage     us-east.workspace.basic-cos-storage.adb78a38
ACTIVE us-east

BLUEPRINT OUTPUTS
Key Value
cos_id ce0b7693-26b4-4996-aed6-a8528ac100e0

OK
```
{: screen}

The full details of the blueprint can be retrieved by using the `--level detailed` option.

```sh
ibmcloud schematics blueprint get --id Blueprint_Basic.eaB.5cd9 --level detailed
```
{: pre}

#### Output
{: #step4-output}

```text
BLUEPRINT

Name Blueprint_Basic
Git Repo URL https://github.com/Cloud-Schematics/blueprint-basic-example
Git Repo File basic-blueprint.yaml
Release latest

INPUT FILE

Git Repo URL https://github.com/Cloud-Schematics/blueprint-basic-example
Git Repo File basic-input.yaml
Git Branch main

MODULES
SNO Workspace Name Workspace ID Status Location Updated
1 basic-resource-group us-east.workspace.basic-resourcegroup.9ea437b2 ACTIVE us-east
2 basic-cos-storage us-east.workspace.basic-cos-storage.adb78a38
ACTIVE us-east

OK
```
{: screen}


## Destroy blueprint cloud resources
{: #destroy-schematics-blueprint-cli}
{: step}

Optional: Run the [`ibmcloud schematics blueprint run destroy`](/docs/schematics?topic=schematics-schematics-cli-reference#schematics-blueprint-destroy) command for the Blueprint environment to destroy all the cloud resources that are created in earlier steps. 
* Insert the ID saved from the [output of the create](#create-schematics-blueprint-cli) command.
* When prompted reply `yes`, or `y` to destroy the cloud resources.

```sh
ibmcloud schematics blueprint run destroy --id Blueprint_Basic.eaB.5cd9
```
{: pre}

### Output
{: #step5-output}

```text
Modules to be destroyed 
SNO Type Name Status
1 terraform basic-resource-group ACTIVE
2 terraform basic-cos-storage ACTIVE

Do you really want to destroy all the resources of Blueprint
Blueprint_Basic.eaB.5cd9? [y/N]> y
Blueprint job running us-east.JOB.Blueprint_Basic.f69f5bf2
Waiting:0 Draft:0 Connecting:0 In Progress:0 Inactive:2 Active:0
Failed:0
Type Name Status Job ID
Blueprint Blueprint_Basic FULFILMENT_SUCCESS useast.JOB.Blueprint_Basic.f69f5bf2
terraform basic-resource-group INACTIVE
terraform basic-cos-storage INACTIVE

Blueprint ID Blueprint_Basic.eaB.5cd9 fulfilment_success at 2022-08-04
17:48:36
OK
```
{: screen}

## Delete the blueprint
{: #delete-schematics-blueprint-cli}
{: step}

Deletes the blueprint workspace. You need to run the `blueprint run destroy` command and then run `blueprint config delete` command. 

Optional: Run the [`ibmcloud schematics blueprint config delete`](/docs/schematics?topic=schematics-schematics-cli-reference#schematics-blueprint-delete) command to remove the blueprint config created in earlier steps. 

* Insert the ID saved from the output of the [blueprint config create](#create-schematics-blueprint-cli) command.
* When prompted reply `yes`, or `y` to delete the cloud resources.

```sh
ibmcloud schematics blueprint delete --id Blueprint_Basic.eaB.5cd9
```
{: pre}

### Output
{: #step6-output}

```text
Modules to be deleted
SNO Type Name Status
1 terraform basic-resource-group INACTIVE
2 terraform basic-cos-storage INACTIVE

Do you really want to delete the Blueprint ? [y/N]> y
Job : us-east.JOB.Blueprint_Basic.cbe0591e Created
Job Type: BLUEPRINT DELETE
OK
```
{: screen}

## Next steps
{: #nextsteps-schematics-blueprint-cli}

By completing this tutorial, you learned to deploy a cloud environment using {{site.data.keyword.bpshort}} blueprints.

- Looking for blueprint samples? Check out the [{{site.data.keyword.bplong_notm}} GitHub repository](https://github.com/orgs/Cloud-Schematics/repositories/?q=topic:blueprint){: external}. Check the example `Readme` files for further blueprint customization and usage scenarios for each sample. 
- Looking for blueprint template and input file configuration Check out the [Understanding blueprint templates and configuration](/docs/schematics?topic=schematics-blueprint-templates).
