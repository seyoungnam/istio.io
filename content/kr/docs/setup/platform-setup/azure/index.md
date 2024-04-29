---
title: Azure
description: Instructions to set up an Azure cluster for Istio.
weight: 10
skip_seealso: true
aliases:
  - /docs/setup/kubernetes/prepare/platform-setup/azure/
  - /docs/setup/kubernetes/platform-setup/azure/
keywords: [platform-setup, azure]
owner: istio/wg-environments-maintainers
test: no
---

Istio가 포함된 Azure 클러스터를 준비하기 위해 아래 설명을 따르세요.

{{< tip >}}
Azure는 Azure Kubernetes Service (AKS)에서 {{< gloss >}}managed control plane{{< /gloss >}} add-on을 제공하며, 
이를 활용하면 Istio를 수동으로 설치할 필요가 없습니다.
더 자세한 내용은 [Deploy Istio-based service mesh add-on for Azure Kubernetes Service](https://learn.microsoft.com/azure/aks/istio-deploy-addon)을 참고하세요.
{{< /tip >}}

[AKS](https://azure.microsoft.com/en-us/services/kubernetes-service/) 혹은 Istio를 완벽히 지원하는 
[자체 관리형 쿠버네티스 혹은 AKS를 위한 Azure (CAPZ)용 클러스터 API 공급자](https://capz.sigs.k8s.io/)를 통해 쿠버네티스 클러스터를 Azure에 생성할 수 있습니다.

## AKS

[az cli](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough), [Azure 포털](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal), [az cli with Bicep](https://learn.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-deploy-bicep?tabs=azure-cli), 혹은 [Terraform](https://learn.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-deploy-terraform?tabs=bash) 등 다양한 방법을 통해 AKS 클러스터를 생성할 수 있습니다.

`az` cli 옵션 사용시, `az login` 인증을 마치거나 cloud shell을 사용하여 아래 명령어를 입력하세요.

1. AKS를 지원하는 region명을 고르세요

    {{< text bash >}}
    $ az provider list --query "[?namespace=='Microsoft.ContainerService'].resourceTypes[] | [?resourceType=='managedClusters'].locations[]" -o tsv
    {{< /text >}}

1. 선택한 region 내 지원 중인 쿠버네티스 버전을 확인하세요

    `my location`을 원하는 region명으로 바꾼 후 아래 명령어를 실행하세요:

    {{< text bash >}}
    $ az aks get-versions --location "my location" --query "orchestrators[].orchestratorVersion"
    {{< /text >}}

1. resource group을 만들고 AKS 클러스터를 배포하세요

    `myResourceGroup`과 `myAKSCluster`을 원하는 값으로 바꾸고, `my location`은 step 1의 값으로 대체하세요. 해당 region에서 지원하지 않을 경우 `1.28.3`을 입력하고 실행하세요:

    {{< text bash >}}
    $ az group create --name myResourceGroup --location "my location"
    $ az aks create --resource-group myResourceGroup --name myAKSCluster --node-count 3 --kubernetes-version 1.28.3 --generate-ssh-keys
    {{< /text >}}

1. AKS `kubeconfig` credentials을 획득하세요

   `myResourceGroup` and `myAKSCluster`은 위 단계에서 얻은 값으로 교체한 후 실행하세요:

    {{< text bash >}}
    $ az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
    {{< /text >}}
