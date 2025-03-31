# Stateful Workloads in Kubernetes: A Knowledge Base

This project aims to serve as a comprehensive knowledge base, gathering best practices for running stateful workloads
in Kubernetes.

See Proposal here: https://docs.google.com/document/d/13Wcn6yN8hQmsHogpUE85sFD5wBNsRl25Y1XcIb0RapA/edit?usp=sharing

## Why?

Regardless of whether an application runs on bare-metal, virtual machines, or containers, it requires the same
fundamental resources:

* CPU
* RAM
* Network Interface
* **Storage**

However, performance can vary significantly across different environments, even for the same application. To prevent
unexpected issues, it’s crucial to test performance before deploying to production.

This is especially true for stateful workloads, which rely heavily on storage. Throughout computing history, storage
has often been the primary bottleneck when applications run slower than expected.

## How?

This project aims to share knowledge on testing stateful workloads in Kubernetes, helping users evaluate performance
across different storage configurations, and collect baselines.

To support this, the project suggests tools and best practices for benchmarking stateful applications. This allows
users to compare various storage architectures, such as different storage classes or Container Storage Interfaces
(CSIs), to identify the best fit for their specific use case.

## Who?

The Data on Kubernetes Community is built by volunteers who collaborate and share their knowledge. We actively
welcome and appreciate contributions from anyone interested in improving and expanding this knowledge base. Your
insights, experiences, and best practices can make a difference!

## Where?

This project organizes directories based on different types of stateful workloads that can run on Kubernetes. This
categorization makes it easier to find the most relevant guide for any specific use case.

## When?

Better before than later. ;)

