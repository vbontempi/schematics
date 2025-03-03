---

copyright:
  years: 2017, 2022
lastupdated: "2022-11-05"

keywords: migrating terraform version, terraform version migration for schematics 

subcollection: schematics

---

{{site.data.keyword.attribute-definition-list}}


# Migrating Terraform version in {{site.data.keyword.bpshort}} Workspace
{: #migrating-terraform-version}

The driver to migrate might come from business factors such as cost reduction, or consolidation. You also might want to migrate to be more cloud native or adopt new technologies. Regardless of the reason, migration can be as simple as migrating a Terraform version in variable file. Or it can be as complex as migrating a piece of your application to a more complex environment where you need to migrate an entire template with all the underlying components.
{: shortdesc}

## Upgrading the Terraform template version 
{: #terraform-version-upgrade}

The table summarizes the recommendations to upgrade to the current Terraform version. {{site.data.keyword.bpshort}} supports `Terraform_v1.x` and also plans to make releases available after `30-45 days` of HashiCorp Configuration Language (HCL) release. The Terraform template must use the version constraint, such as, `>`, `>=`, or `~>` for the `required_version` parameter in the `versions.tf` of Terraform template, to automatically pick the current provider version.

{{site.data.keyword.bpshort}} supports all the releases of `terraform_v1.x`. {{site.data.keyword.bpshort}} plans to make all the releases of Terraform `n-1` versions available after 30 - 45 days after HCL release.
```terraform
terraform {
required_version = "~> 1.1"
}
```
{: pre}

{{site.data.keyword.bplong_notm}} suggests that use Terraform templates of `terraform_v1.0` and higher version.
{: note}

|Current Version|	Recommendation|
| ---| ---|
| `v0.11` | Use the [`Terraform v0.12 checklist`](https://developer.hashicorp.com/terraform/language/v1.1.x/upgrade-guides/0-12#pre-upgrade-checklist){: external} command to detect and fix to be addressed by referring [v0.12 Upgrade Guide](https://developer.hashicorp.com/terraform/language/v1.1.x/upgrade-guides/0-12){: external} before upgrade to `v0.12`. {{site.data.keyword.bpshort}} deprecated `Terraform v0.11` and `Terraform v0.12`.|
| `v0.12` | Use the [v0.13 upgrade guide](https://developer.hashicorp.com/terraform/language/v1.1.x/upgrade-guides/0-13){: external} your configuration file. The {{site.data.keyword.bpshort}} deprecated `Terraform v0.12`.|
| `v0.13` | To the `Terraform v0.14` upgrade, you must run the `terraform apply` with `Terraform v0.13` to complete its state format upgrades. If you get any errors, see the [v0.14 upgrade guide](https://developer.hashicorp.com/terraform/language/v1.1.x/upgrade-guides/0-14){: external}.|
| `v0.14` | You can upgrade directly to the [`Terraform v1.0`](https://developer.hashicorp.com/terraform/language/v1.1.x/upgrade-guides/1-1){: external} version and you must run the `terraform apply` with `Terraform v0.14`. If you get any errors, see the [v0.15 upgrade guide](https://developer.hashicorp.com/terraform/language/v1.1.x/upgrade-guides/0-15){: external}.|
| `v0.15` | You can upgrade directly to the [`Terraform v1.0`](https://developer.hashicorp.com/terraform/language/v1.1.x/upgrade-guides/1-0){: external} version, you must run the `terraform apply` with `terraform v0.15`. `Terraform v1.0` is a continuation of the `v0.15` series, hence `v1.0.0` and later are directly backward-compatible with an earlier version `Terraform v0.15.5`.|
{: caption="Terraform versions list" caption-side="bottom"}

## Upgrading `Terraform v0.12 to v0.13` in {{site.data.keyword.bpshort}} Workspace
{: #migrate-steps}

Upgrading the {{site.data.keyword.bpshort}} Workspace to use the version of the Terraform, can be required to use the current features in Terraform. You must carefully review the [Terraform upgrade guide](https://developer.hashicorp.com/terraform/language/upgrade-guides) by using the next version. 

Use the following steps to upgrade to the current Terraform version in the {{site.data.keyword.bpshort}} Workspace.

1. Upgrade the Terraform configuration files to use the newer syntax and semantics.
2. Migrate the Terraform state file to be compatible with the newer version. {{site.data.keyword.bpshort}} does not support built in upgrade of the Terraform version. Therefore, you need to follow these steps.

   1. Prepare the upgraded version of Terraform configuration files and [Terraform state file](/docs/schematics?topic=schematics-schematics-cli-reference#state), in your local machine.
   2. [Create the {{site.data.keyword.bpshort}} Workspaces](/docs/schematics?topic=schematics-schematics-cli-reference#schematics-workspace-new) with the new Terraform configuration files and Terraform state file.
   3. [Delete the older workspace](/docs/schematics?topic=schematics-schematics-cli-reference#schematics-workspace-delete) without destroying the resources.

The following are the detailed steps to upgrade:

1. A prerequisites, check whether {{site.data.keyword.bpshort}} Workspaces is created, plan is generated, and applied a job for your resources by using `Terraform v0.12`. Check the Terraform configuration files and Terraform state file, are in a consistent state for `Terraform v0.12`.
2. Download or clone the Git repository that is used by your `Terraform v0.12` {{site.data.keyword.bpshort}} Workspaces to your local machine.
3. Change directory to your cloned repository and upgrade your repository to `Terraform v0.13` by running `Terraform v0.13upgrade` command. For more information, see [Upgrading to `Terraform v0.13` documentation](https://developer.hashicorp.com/terraform/language/v1.1.x/upgrade-guides/0-13){: external}. The upgrade command generates a `versions.tf` file.
4. Edit `versions.tf` file to deselect the source parameter and add `source = "IBM-Cloud/ibm"` as shown in the code block.

    `versions.tf` file

    ```terraform
    terraform {
        required_providers {
        ibm = {
          # TF-UPGRADE-TODO
          #
          # No source detected for this provider. You must add a source address
          # in the following format:
          #
          source = "IBM-Cloud/ibm"
          #
          # For more information, see the provider source documentation:
          #
        }
        }
        required_version = ">= 0.13"
    } 
    ```
    {: codeblock}

5. Download the Terraform state file from the existing {{site.data.keyword.bpshort}} Workspace, by using [{{site.data.keyword.bpshort}} state pull](/docs/schematics?topic=schematics-schematics-cli-reference#state-pull) command.

    When, the workspace is created with the `tfstate` {{site.data.keyword.bpshort}} considers as a secure file. Also, you cannot pull the created `tfstate` file through UI. You need to use command line to [pull](/docs/schematics?topic=schematics-schematics-cli-reference#state-pull) the state file and [create workspace](/docs/schematics?topic=schematics-schematics-cli-reference#schematics-workspace-new).
    {: important}

6. Run the state replace provider command in command-line to update the Terraform version.
    ```sh
    terraform state replace-provider registry.terraform.io/-/ibm registry.terraform.io/ibm-cloud/ibm.
    ```
    {: pre}

7. Verify that the updates are made to the `terraform.tfstate` file with Terraform version update from `0.12` to `>= 0.13` and provider that is updated as `registry.terraform.io/ibm-cloud/ibm`.
8.	Push the upgraded `version.tf` to your Git repository.
9.	Create the {{site.data.keyword.bpshort}} Workspace](/docs/schematics?topic=schematics-schematics-cli-reference#schematics-workspace-new) through command-line from the updated Git repository.
    
    The Terraform files are present in Git repository, but the state file is present in your local file system.
    {: note}

10. Generate a [plan](/docs/schematics?topic=schematics-schematics-cli-reference#schematics-plan) and to view your workspace with the resources are now using the `Terraform v0.13`.
11. [Optional] you can delete the {{site.data.keyword.bpshort}} Workspaces that points to `Terraform v0.12`. 

    Do not destroy the resources that are used by your old workspace.
    {: note}

## Upgrading Terraform template from v0.12 to v0.13 
{: #upgrade-12-to13}

Use the following steps to upgrade from the `Terraform v0.12` to `Terraform v0.13`.
{: shortdesc}

Check that your Terraform template of the older version is provisioned without any errors before upgrade. For more information, see [Creating workspaces and importing your Terraform template](/docs/schematics?topic=schematics-workspace-setup#create-workspace).
{: note}

1. From the Terraform template `v0.12` Git repository, clone to a new GitHub repository.
2. From the new GitHub working directory, run `terraform v0.13upgrade` from command-line. The upgrade command generates a `versions.tf` file.
3. Edit `versions.tf` in the generated files, and add `source = "IBM-Cloud/ibm"` in the provider block as shown in the code block.
    ```terraform
      required_providers {
      ibm = {
        # TF-UPGRADE-TODO
        #
        # No source detected for this provider. You must add a source address
        # in the following format:
        #
        source = "IBM-Cloud/ibm"
        #
        # For more information, see the provider source documentation:
        #
      }
      }
      required_version = ">= 0.13"
    ```
    {: codeblock}

4. Push the `versions.tf` changes to the GitHub repository.
5. Pull the state file from the `Terraform v0.12` workspace by running `ibmcloud schematics state pull --id <WORKSPACE_ID> --template <TEMPLATE_ID>`.
6. Copy the content of state pull result in `state.json` file.
7. Create or update `workspace.json` as shown in the code block.

    ```json
    {
       "name": "gb",
       "type": [
           "terraform_v0.13"
       ],
       "description": "migration workspace",
       "template_repo": {
           "url": "https://github.com/xxxxxx/migration-testing"
       },
       "workspace_status" : {
           "frozen": false
       },
       "template_data": [{
           "folder": ".",
           "type": "terraform_v0.13"
       }]
     }
    ```
    {: codeblock}

8. Run these commands through command-line
   1. `ibmcloud schematics workspace new --file workspace.json --state state.json`.
   2. `ibmcloud schematics workspace get --id  <workspace-id> --json`.
        If your workspace status is not `inactive`, wait for few seconds and retry the command.
        {: note}

   3. `ibmcloud schematics plan id <workspace id>`.
   4. `ibmcloud schematics job get --id <job-id form plan> --json`.
        If your workspace plan status is not `success`, wait for few seconds and retry the command.
        {: note}
   
   5. `ibmcloud schematics apply --id <workspace id>`.
   6. `ibmcloud schematics job get --id <job-id from apply> --json`.

You completed the upgrade successfully. See [Terraform template from v0.13 to v0.14](#upgrade-13-to14) to upgrade from Terraform `v0.13` to `v0.14`.

## Upgrading Terraform template from v0.13 to v0.14 
{: #upgrade-13-to14}

Use the following steps to upgrade from the `Terraform v0.13` to `Terraform v0.14`.
{: shortdesc}

Check whether your Terraform template of the older version is provisioned perfectly without any errors before upgrade. For more information, see [Creating workspaces and importing your Terraform template](/docs/schematics?topic=schematics-workspace-setup#create-workspace).
{: note}

1. From your Terraform template `v0.13` Git repository, clone to a new GitHub repository.
2. Pull the state file from the Terraform `v0.13` workspace by running `ibmcloud schematics state pull --id <WORKSPACE_ID> --template <TEMPLATE_ID>`.
3. Copy the content of state pull result in `state.json` file.
4. Create or update `workspace.json` as shown in the code block.

    ```json
     {
       "name": "gb",
       "type": [
           "terraform_v0.14"
       ],
       "description": "migration workspace",
       "template_repo": {
           "url": "https://github.com/xxxxxx/migration-testing"
       },
       "workspace_status" : {
           "frozen": false
       },
       "template_data": [{
           "folder": ".",
           "type": "terraform_v0.14"
       }]
      }
     ```
     {: codeblock}

5. Run these commands through command-line
   1. `ibmcloud schematics workspace new --file workspace.json --state state.json`.
   2. `ibmcloud schematics workspace get --id  <workspace-id> --json`.
        If your workspace status is not `inactive`, wait for few seconds and retry the command.
        {: note}

   3. `ibmcloud schematics plan id <workspace id>`.
   4. `ibmcloud schematics job get --id <job-id form plan> --json`.
        If your workspace plan status is not `success`, wait for few seconds and retry the command.
        {: note}
   
   5. `ibmcloud schematics apply --id <workspace id>`.
   6. `ibmcloud schematics job get --id <job-id from apply> --json`.

## Upgrade Terraform template from `v0.14` and higher to `v1.0`
{: #upgrade-14-to10}

You can upgrade the [`Terraform v0.14`](https://developer.hashicorp.com/terraform/language/v1.1.x/upgrade-guides/0-14){: external} and [`Terraform v0.15`](https://developer.hashicorp.com/terraform/language/v1.1.x/upgrade-guides/0-15){: external} to `Terraform v1.0`, see [`Terraform v1.0` upgrade process](https://developer.hashicorp.com/terraform/language/v1.1.x/upgrade-guides/1-0){: external}.
{: shortdesc}

Check whether your Terraform template of the older version is provisioned perfectly without any errors before upgrade. For more information, see [Creating workspaces and importing your Terraform template](/docs/schematics?topic=schematics-workspace-setup#create-workspace).
{: note}
