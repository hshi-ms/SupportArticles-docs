---
title: Troubleshoot "PodDrainFailure" error code
description: Learn how to troubleshoot the "PodDrainFailure" error when you try to upgrade an Azure Kubernetes Service cluster.
ms.date: 7/18/2022
author: genlin
ms.author: genli
editor: v-jsitser
ms.reviewer: chiragpa
ms.service: container-service
#Customer intent: As an Azure Kubernetes Services (AKS) user, I want to troubleshoot an Azure Kubernetes Service cluster upgrade that failed because of a "PodDrainFailure" error so that I can upgrade the cluster successfully.
---

# Troubleshoot the "PodDrainFailure" error code

This article discusses how to identify and resolve the "PodDrainFailure" error that occurs when you try to upgrade an Azure Kubernetes Service (AKS) cluster.

## Prerequisites

This article requires Azure CLI version 2.0.65 or a later version. To find the version number, run `az --version`. If you have to install or upgrade Azure CLI, see [How to install the Azure CLI](/cli/azure/install-azure-cli).

For more detailed information about the upgrade process, see the "Upgrade an AKS cluster" section in [Upgrade an Azure Kubernetes Service (AKS) cluster](/azure/aks/upgrade-cluster#upgrade-an-aks-cluster).

## Symptoms

An AKS cluster upgrade fails, and you receive a "PodDrainFailure" error message.

## Cause

This error might occur if a pod is protected by the Pod Disruption Budget (PDB) policy. In this situation, the pod resists being drained.

To test this situation, run `kubelect get pdb -A`, and then check the **Allowed Disruption** value. The value should be **1** or greater. For more information, see [Plan for availability using pod disruption budgets](/azure/aks/operator-best-practices-scheduler#plan-for-availability-using-pod-disruption-budgets).

If the **Allowed Disruption** value is **0**, the node drain will fail during the upgrade process.

To resolve this issue, use one of the following solutions.

## Solution 1: Enable pods to drain

1. Adjust the PDB to enable pod draining. Generally, The allowed disruption is controlled by the `Min Available / Max unavailable` or `Running pods / Replicas` parameter. You can modify the `Min Available / Max unavailable` parameter at the PDB level or increase the number of `Running pods / Replicas` to push the Allowed Disruption value to **1** or greater.
2. Try again to upgrade the AKS cluster to the same version that you tried to upgrade to previously. This process will trigger a reconciliation.

## Solution 2: Back up, delete, and redeploy the PDB

1. Make a backup of the PDB `kubectl get pdb <pdb-name> -n <pdb-namespace> -o yaml > pdb_backup.yaml`, and then delete the PDB `kubectl delete pdb <pdb-name> -n /<pdb-namespace>`. After the upgrade is finished, you can redeploy the PDB `kubectl apply -f pdb_backup.yaml`.
1. Try again to upgrade the AKS cluster to the same version that you tried to upgrade to previously. This process will trigger a reconciliation.

## Solution 3: Delete the pods that can't be drained

1. Delete the pods that can’t be drained. **Note**: If the pods were created by a deployment or StatefulSet, they'll be controlled by a ReplicaSet. If that's the case, you might have to delete the deployment or StatefulSet. Before you do that, we recommend that you make a backup: `kubectl get <kubernetes-object> <name> -n <namespace> -o yaml > backup.yaml`.

2. Try again to upgrade the AKS cluster to the same version that you tried to upgrade to previously. This process will trigger a reconciliation.

[!INCLUDE [Azure Help Support](../../includes/azure-help-support.md)]
