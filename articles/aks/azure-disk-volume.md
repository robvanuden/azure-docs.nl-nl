---
title: Azure-schijven met AKS gebruiken
description: Azure-schijven met AKS gebruiken
services: container-service
author: neilpeterson
manager: timlt
ms.service: container-service
ms.topic: article
ms.date: 03/08/2018
ms.author: nepeters
ms.custom: mvc
ms.openlocfilehash: a2f46aba80ad47335b7cd9b5e8d615c1d895cccb
ms.sourcegitcommit: 8aab1aab0135fad24987a311b42a1c25a839e9f3
ms.translationtype: MT
ms.contentlocale: nl-NL
ms.lasthandoff: 03/16/2018
---
# <a name="volumes-with-azure-disks"></a>Volumes met Azure-schijven

Container gebaseerde toepassingen moeten vaak voor toegang tot en het behouden van gegevens in een externe gegevensbron-volume. Azure-schijven kunnen worden gebruikt als deze externe gegevensarchief. Dit artikel gegevens met een Azure-schijf als een volume Kubernetes in een Azure Container Service (AKS)-cluster.

Zie voor meer informatie over Kubernetes volumes [Kubernetes volumes][kubernetes-volumes].

## <a name="create-an-azure-disk"></a>Een Azure-schijf maken

Voordat het koppelen van een Azure beheerd schijf als een volume Kubernetes de schijf moet aanwezig zijn in dezelfde resourcegroep bevinden als de clusterbronnen AKS. Als deze resourcegroep zoekt, volgt u de [az groepslijst] [ az-group-list] opdracht.

```azurecli-interactive
az group list --output table
```

Zoekt u naar een resourcegroep met een naam die lijkt op `MC_clustername_clustername_locaton`, waarbij clustername de naam van uw cluster AKS en locatie van de Azure-regio waar het cluster is geïmplementeerd.

```console
Name                                 Location    Status
-----------------------------------  ----------  ---------
MC_myAKSCluster_myAKSCluster_eastus  eastus      Succeeded
myAKSCluster                         eastus      Succeeded
```

Gebruik de [az schijf maken] [ az-disk-create] opdracht voor het maken van de schijf van Azure. 

Het volgende voorbeeld bijwerken `--resource-group` met de naam van de resourcegroep en `--name` in een naam van uw keuze.

```azurecli-interactive
az disk create \
  --resource-group MC_myAKSCluster_myAKSCluster_eastus \
  --name myAKSDisk  \
  --size-gb 20 \
  --query id --output tsv
```

Als de schijf is gemaakt, ziet u uitvoer ziet er als volgt. Deze waarde is de schijf-ID wordt gebruikt bij het koppelen van de schijf naar een schil Kubernetes.

```console
/subscriptions/<subscriptionID>/resourceGroups/MC_myAKSCluster_myAKSCluster_eastus/providers/Microsoft.Compute/disks/myAKSDisk
```

## <a name="mount-disk-as-volume"></a>Schijf als een volume koppelen

Koppel de Azure-schijf in uw schil door het configureren van het volume in de container-specificaties. 

Maak een nieuw bestand met de naam `azure-disk-pod.yaml` met de volgende inhoud. Update `diskName` met de naam van de nieuwe schijf en `diskURI` met de id van de schijf. Let ook op de `mountPath`, dit is het pad waar de Azure-schijf is gekoppeld in de schil.

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: azure-disk-pod
spec:
 containers:
  - image: microsoft/sample-aks-helloworld
    name: azure
    volumeMounts:
      - name: azure
        mountPath: /mnt/azure
 volumes:
      - name: azure
        azureDisk:
          kind: Managed
          diskName: myAKSDisk
          diskURI: /subscriptions/<subscriptionID>/resourceGroups/MC_myAKSCluster_myAKSCluster_eastus/providers/Microsoft.Compute/disks/myAKSDisk
```

Gebruik kubectl schil maken.

```azurecli-interactive
kubectl apply -f azure-disk-pod.yaml
```

U hebt nu een actieve schil met een Azure schijf die is gekoppeld aan de `/mnt/azure`.

## <a name="next-steps"></a>Volgende stappen

Meer informatie over Kubernetes volumes met behulp van Azure-schijven.

> [!div class="nextstepaction"]
> [Kubernetes-invoegtoepassing voor Azure-schijven][kubernetes-disks]

<!-- LINKS - external -->
[kubernetes-disks]: https://github.com/kubernetes/examples/blob/master/staging/volumes/azure_disk/README.md
[kubernetes-volumes]: https://kubernetes.io/docs/concepts/storage/volumes/

<!-- LINKS - internal -->
[az-disk-list]: /cli/azure/disk#az_disk_list
[az-disk-create]: /cli/azure/disk#az_disk_create
[az-group-list]: /cli/azure/group#az_group_list
