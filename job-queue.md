---

copyright:
  years: 2017, 2021
lastupdated: "2021-09-30"

keywords: schematics job queue, job queue process, pending queue, schematics pending queue

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
{:support: data-reuse='support'}
{:help: data-hd-content-type='help'}

# Executing process of the {{site.data.keyword.bpshort}} job queue
{: #job-queue-process}

## About job queue
{: #about-job-queue}

When a user generates plan, as a shared service {{site.data.keyword.bpshort}} queues all the user's jobs into a single job queue. Your jobs reside in a queue until scheduled to run in an environment. {{site.data.keyword.bpshort}} job queues use a scheduler to equitably available for the respective users.

## Functioning of job queue function
{: #functions-job-queue}

Depending upon the workload that is generated by the user and the time to run the jobs, the consumer might experience deplays. The job queue ensures that {{site.data.keyword.bpshort}} is equitably available to all its users irrespective of the load generated by the a user. 

**Example**

If an `user-1` has 20 jobs waiting in the queue, and `user-2` submits a new job. `User-2` job waits into a queue, ahead of `user-1's` 20 jobs to make {{site.data.keyword.bpshort}} equitably available for both `user-1` and `user-2`.

## When does the job enter into the pending queue?
{: #pending-job-queue}

Following are the tasks of the job when it enter into pending queue.
- your job requires more time to complete. Check that enough time is specified for the job to execute.
- your image that is used by your job run does not exist. Check that the provided image details exist and right name is specified.
- The environment variable parameters that are required by the job are not specified. Check that the environment variables are defined.
- The commands or arguments that are passed to the job are not valid. Check that the argument flags specified are correct.