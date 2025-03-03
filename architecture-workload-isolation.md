---

copyright:
  years: 2017, 2022
lastupdated: "2022-10-17"

keywords: schematics architecture, schematics compliance, schematics workload isolation, schematics dependencies

subcollection: schematics

---

{{site.data.keyword.attribute-definition-list}}

# {{site.data.keyword.bpshort}} architecture
{: #compute-isolation}

Learn about the {{site.data.keyword.bplong}} service architecture, the service dependencies, and how customer workloads are isolated from each other in {{site.data.keyword.bplong_notm}}?
{: shortdesc}

## Architectural flow
{: #basic-architecture}

{{site.data.keyword.bpshort}} is a shared service. On the initial use, a new service instance is automatically provisioned for each user account by using the provisioning method.

The following {{site.data.keyword.bpshort}} architecture image depicts the 
- Main {{site.data.keyword.bpshort}} components.
- How cloud service interacts each other with the components for an operation?
- What type of encryption are applied to secure your services?
- Usage of the observability services.
- Role of runtime jobs to interact with {{site.data.keyword.cloud_notm}} `APIs`, private cloud such as `vSphere`, `Kubernetes`, and other public cloud providers such as `AWS`, `Google`, so on.
{: shortdesc}

![{{site.data.keyword.bpshort}} architecture](images/schematics-enduser-architecture.png){: caption="Figure 1. {{site.data.keyword.bpshort}} architecture" caption-side="bottom"}

## Workload isolation
{: #workload-isolation}

Review how your workloads are isolated in {{site.data.keyword.bplong_notm}}.
{: shortdesc}

### How are API requests to the service isolated from other API requests?
{: #workload-api-isolation}

All API requests that to the {{site.data.keyword.bpshort}} apiserver are queued in {{site.data.keyword.bpshort}} internal messages. {{site.data.keyword.bpshort}} job queue manager forwards the requests with the job ID and health check messages. At any particular time, a maximum of n API requests are processed by the {{site.data.keyword.bpshort}} engine. By default, n equals 20, but this number is manually adjusted by the {{site.data.keyword.bpshort}} operator based on the current API workload.

### How are the workloads isolated from other tenants? 
{: #workload-tenant-isolation}

When you use {{site.data.keyword.bpshort}} to provision {{site.data.keyword.cloud_notm}} resources, these resources are created in your personal {{site.data.keyword.cloud_notm}} account. You are responsible to manage these resources and to keep them up to date to avoid security vulnerabilities or downtime for your workloads. {{site.data.keyword.cloud_notm}} resources are provisioned, updated, and deleted as defined in the Terraform template and requested by the user. Because all resources are created in your personal account, resources are not shared with or reused by other {{site.data.keyword.cloud_notm}} tenants.

