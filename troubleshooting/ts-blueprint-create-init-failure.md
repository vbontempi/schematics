---

copyright:
  years: 2017, 2022
lastupdated: "2022-07-11"

keywords: blueprint create init failure, blueprint init error, create init fails,

subcollection: schematics

content-type: troubleshoot

---

{{site.data.keyword.attribute-definition-list}}

{{site.data.keyword.bpshort}} Blueprints is a [Beta feature](/docs/schematics?topic=schematics-bp-beta-limitations) that is available for evaluation and testing purposes. It is not intended for production usage. Refer to the list of [limitations](/docs/schematics?topic=schematics-bp-beta-limitations) for the Beta release.
{: beta}

# Blueprint create fails in the Blueprint `create_init` step
{: #bp-create-init-fails}

When you create a Blueprint, the create fails during initialization of the {{site.data.keyword.bpshort}} Workspaces for the modules. 
{: tsSymptoms}

Blueprints are created in two steps. The first step retrieves, validates the Blueprint definition and creates the configuration in {{site.data.keyword.bpshort}}. The second step creates the required module Workspaces based on the module statements in the definition. This step clones the specified module source repos. An incorrectly specified repo URL will result in a Workspace initialisation failure. Other possible causes are the repository is private and an access token is not specified, or the access token is invalid. 
{: tsCauses}

Sample error

```text
Created Blueprint   blueprints-starter-example   eu-gb.BLUEPRINT.blueprints-starter-example.deee1f6f

Modules to be created
SNO   Type        Name   
1     Terraform   terraform_module2   
2     Terraform   terraform_module1   
      
Blueprint job running eu-gb.JOB.blueprints-starter-example.e44adbab

Waiting:2    Draft:0    Connecting:0    InProgress:0    Inactive:0    Active:0    Failed:0   

Type        Name                         Status                     Job ID   
Blueprint   blueprints-starter-example   CREATE_FAILED   eu-gb.JOB.blueprints-starter-example.e44adbab   
Terraform   terraform_module2            NA                 
Terraform   terraform_module1            NA                 
            
Blueprint create_failed at Tue Jun 14 13:21:49 BST 2022
OK
```
{: screen} 

The Blueprint job status shows as `create_failed`. Additionally the Workspace job status shows as `NA` indicating that {{site.data.keyword.bpshort}} failed to create the Workspaces and no Workspace logs are available.  


Review the error messages from the log of the failing Blueprint job, the Blueprint module definitions and Git repos to determine the cause of the initialisation failure.  
{: tsResolve}

Typical issues are:
- An incorrectly specified module repo URL 
- Repo is private and an access token is not specified
- Invalid Git access token 

Correct the cause of the failure. Most typically it is an invalid module repo URL and the Blueprint module statement will require updating.  
- Update the Blueprint module statements to specify the correct Git repo URL
- Push the new release of the Blueprint definition to its Git source repository. With an updated release tag if required.

If using relaxed Blueprint definition versioning (latest), run the `ibmcloud schematics blueprint update` command to refresh the Blueprint configuration stored by {{site.data.keyword.bpshort}} with the update to the Blueprint definition. 


```sh
ibmcloud schematics blueprint update -id <blueprint_id> 
```
{: pre}

If explicit Blueprint versioning is used with release tags for each Blueprint definition release, the Blueprint configuration must be updated in {{site.data.keyword.bpshort}} with the new Blueprint release tag.  

```sh
ibmcloud schematics blueprint update --id <blueprint_id> --bp-git-release x.y.z  
```
{: pre}


Verify that the Blueprint has been updated successfully. When you update the Blueprint from the CLI, the command displays details of the linked Workspaces to be updated and a continuously updating status of the progress of the {{site.data.keyword.bpshort}} jobs initalising the Workspaces. The command only returns on completion.

```text
Update Blueprint ID: eu-de.BLUEPRINT.Blueprint-Basic-Example.21735936
Modules to be updated
SNO   Type        Name   
1     Workspace   basic-resource-group   
2     Workspace   basic-cos-storage   
      
Blueprint job running eu-de.JOB.Blueprint-Basic-Example.da1b13ca
Waiting:0    Draft:0    Connecting:0    In Progress:0    Inactive:2    Active:0    Failed:0   

Type        Name                      Status           Job ID   
Blueprint   Blueprint Basic Example   CREATE_SUCCESS   eu-de.JOB.Blueprint-Basic-Example.da1b13ca   
Workspace   basic-resource-group      INACTIVE            
Workspace   basic-cos-storage         INACTIVE            
            
Blueprint ID eu-de.BLUEPRINT.Blueprint-Basic-Example.21735936 update_success at Mon Jun 27 16:18:47 BST 2022
OK
```
{: screen}

On successful completion the update command will return **update_success**. All Workspaces should now be initialised to `Inactive` state and deployment of the Blueprint can continue with [blueprint install](/docs/schematics?topic=schematics-install-blueprint). 

If the update job fails, repeat problem diagnosis and resolution until update commpletes successfully and all Workspaces are in `Inactive` state. 