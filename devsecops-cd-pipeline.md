---

copyright:
  years: 2021
lastupdated: "2021-12-08"

keywords: DevSecOps

subcollection: devsecops

---

{:shortdesc: .shortdesc}
{:table: .aria-labeledby="caption"}
{:external: target="_blank" .external}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:download: .download}
{:help: data-hd-content-type='help'}
{:support: data-reuse='support'}

# Continuous delivery pipeline
{: #cd-devsecops-cd-pipeline}

The continuous delivery pipeline generates all of the evidence and change request summary content. The pipeline deploys the build artifacts to a specific environment, such as staging or prod, and then collects, creates, and uploads all existing log files, evidence, and artifacts to the evidence locker.
{: shortdesc}

## Stages and tasks
{: #cd-devsecops-pipeline-stages}

Review the following table to learn more about the stages and tasks that are part of the continuous delivery pipeline.

|Task or stage |Short description	|Customizable in `.pipeline-config.yaml` |
|:----------|:------------------------------|:------------------|
|`start` 		|Set up the pipeline environment. 		|No		|
|`setup`		|Set up your build and test environment.		|Yes			|
|`change-request`		|Generate the change request and create the evidence summary. 		|Yes		|
|`deployment`		|Deploy the build artifacts to the environment, such as staging or production.		|Yes		|
|`acceptance-test` 		|Run acceptance and integration tests on the deployment.   	|Yes			|
|`finish` 		|Collect and upload log files, artifact, and evidence to the evidence locker. 		|Yes			|
{: caption="Table 1. Pipeline stages and tasks" caption-side="top"}

For more information about how to customize stages by using the `.pipeline-config.yaml` file, see [Custom scripts](/docs/devsecops?topic=devsecops-custom-scripts) and [Pipeline parameters](/docs/devsecops?topic=devsecops-cd-devsecops-pipeline-parm) lists.

## Deployment delta
{: #cd-devsecops-pipeline-delta}

When the inventory promotion is ready, the continuous delivery pipeline can start. The deployment delta is the difference between the contents of the last concluded deployment and the current deployment. The deployment delta lists the inventory items that are being deployed.

## Calculate deployment BOM
{: #cd-devsecops-pipeline-bom}

The deployment Bill of Material (BOM) represents all of the artifacts that are deployed as part of a single change request. After the deployment delta is calculated, the pipeline creates the deployment BOM based on these items. 

## Collect evidence summary
{: #cd-devsecops-pipeline-collect}

An evidence summary is created from all of the evidence that was created during the relevant build that led to a deployment. Evidence that is created during the deployment itself is also added to the summary. The evidence summary is added to the change request's description.

## Prepare and create change request
{: #cd-devsecops-pipeline-prepcr}

Everything that changes the [baseline](/docs/devsecops?topic=devsecops-cd-devsecops-inventory) must be traced by way of a change request. These changes include updates to the existing code level, changes to the configuration, and updates of the worker nodes. Collection of [peer review compliance](/docs/devsecops?topic=devsecops-cd-devsecops-peer-review) data is based on the data that is accessible in the inventory, the evidence locker, and the incident issue repo.

This step creates the change request by attaching the available compliance data based on the [promotion pull request](/docs/devsecops?topic=devsecops-cd-devsecops-promotion-pipeline#cd-devsecops-promotion-pipelinepr) fields. [Deployment readiness](/docs/devsecops?topic=devsecops-cd-devsecops-automate-changemgmt#cd-devsecops-cr-approve) is calculated based on the available evidence in the collected compliance status.

## Check change request approval
{: #cd-devsecops-pipeline-checkcr}

If every compliance check, such as unit tests, Code Risk Analyzer tasks, branch protection, and detect secrets, is successful, the change request is approved automatically, and the task runs successfully.

If a compliance check fails, the change request state is not approved. You can approve the change request manually and add the `change-request-id` to the environment properties to use the previously created change request in the next run. You can also approve the change request manually and add an emergency label.

## Deployment
{: #cd-devsecops-pipeline-deployment}

In the deploy stage, the pipeline deploys the built artifacts into an environment, such as staging or prod. The variables and credentials for these stages can be found in the following sources:

* Variables from the pipeline UI (`get_env`)

 ![Edit property](images/target-environment.png){: caption="Edit property" caption-side="bottom"}

* [Pipeline trigger webhook](/docs/devsecops?topic=devsecops-cd-devsecops-webhook-payloads)

For more information about how to access these variables, see [Custom scripts](/docs/devsecops?topic=devsecops-custom-scripts).

## Acceptance test
{: #cd-devsecops-pipeline-acctest}

You can run a set of automated tests to validate that the deployment was successful and is working as expected. For traceability purposes, make sure that the test log contains a reference to the code level or image that is being tested.

## Change request close
{: #cd-devsecops-pipeline-closecr}

The details for the deployment are uploaded to the closing summary change task and then the task closes the change request. The `close_category` is added to the close change request task, with the following values:

* Successful
* Successful with issues (if the summary has issues)

## Inventory conclude
{: #cd-devsecops-pipeline-inventory}

For more information about inventory conclude, see [Inventory](/docs/devsecops?topic=devsecops-cd-devsecops-inventory).
