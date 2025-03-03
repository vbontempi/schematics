---

copyright:
  years: 2017, 2022
lastupdated: "2022-10-18"

keywords: blueprint apply, apply blueprint, blueprint

subcollection: schematics

---

{{site.data.keyword.attribute-definition-list}}

{{site.data.keyword.bpshort}} Blueprints is a [Beta feature](/docs/schematics?topic=schematics-bp-beta-limitations) that is available for evaluation and testing purposes. It is not intended for production usage. Refer to the list of [limitations](/docs/schematics?topic=schematics-bp-beta-limitations) for the Beta release.
{: beta}

# Apply changes to a blueprint environment
{: #apply-blueprint}

Blueprint run apply is the second step to create, or update resources in a blueprint environment. This step runs the IaC automation code for each module. For each module, {{site.data.keyword.bpshort}} runs a Terraform Apply to create or configure the cloud resources defined by the Terraform module code. 
{: shortdesc}

## Running a blueprint apply from the CLI  
{: #apply-blueprint-cli}
{: cli}

To deploy your blueprint environment from the CLI, use the `ibmcloud schematics blueprint run apply` command. 
{: shortdesc}

Before your begin

- Install and log in to the [{{site.data.keyword.cloud_notm}} CLI](/docs/schematics?topic=schematics-setup-cli#install-schematics-cli).

The following command executes a `blueprint run apply` operation for the environment with the ID `eu-de.BLUEPRINT.Blueprint-Basic-Example.21735936`

For all the blueprint commands, syntax, and detailed option flags, refer to, [blueprint commands](/docs/schematics?topic=schematics-schematics-cli-reference#blueprints-cmd).
{: important}

**`Syntax`**

```sh
ibmcloud schematics blueprint run apply --id Blueprint_Basic.eaB.5cd9
```
{: pre}

On successful completion, the apply command returns `fullfilment_success`. 

### Verify blueprint run apply success 
{: #bp-verify-apply-cli}

Verify that the blueprint configuration change has been applied successfully. When you apply the configuration from CLI, the command displays the details of the modules being applied, and a continuously updating status of the progress of the {{site.data.keyword.bpshort}} jobs that run the IaC automation code. The command returns on completion.

```text
Modules to be applied
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

On successful completion the Apply command returns `fullfilment_success`.  

If the operation fails, refer to the [troubleshooting](/docs/schematics?topic=schematics-bp-apply-fails) section.


## Creating a blueprint environment from the UI 
{: #create-blueprint-ui}
{: ui}

Currently, you can deploy a blueprint environment using the [blueprint run apply](#apply-blueprint-cli) command.

### Verify blueprint apply operation from the UI 
{: #bp-verify-apply-ui}

1. Click your environment that is listed in the [{{site.data.keyword.cloud_notm}} console](https://cloud.ibm.com/schematics/blueprints){: external} to view the results of the Apply operation. 
2. Click **Overview** tab to see the blueprint summary, including `Modules`, `Variables`, `Details`. The `Recent Job runs` must show the summary details of the blueprint job. 
3. Click **Modules** tab to see the status of the resource modules that are now in `Active` status.
4. Click **Resource** tab to view your provisioned resources list.
5. Click **Jobs history** tab view the result of the apply job and operations that are run against the resource modules to deploy the resources.  

## Next steps
{: #bp-apply-nextsteps}

The environment and cloud resources are now deployed. The resources can be viewed on the `Resources` tab in the [Console](https://cloud.ibm.com/schematics/blueprints){: external}, or through [Resource list](https://cloud.ibm.com/resources){: external}. 
{: shortdesc}

The blueprint configuration and outputs can be reviewed by using the `blueprint get` command. See section [displaying blueprints](/docs/schematics?topic=schematics-schematics-cli-reference&interface=cli#schematics-blueprint-get). 

To destroy or delete a blueprint environment, refer to, [destroy a blueprint environment](/docs/schematics?topic=schematics-destroy-blueprint&interface=cli), and [delete a blueprint config](/docs/schematics?topic=schematics-delete-blueprint&interface=cli#delete-blueprint-cli).

