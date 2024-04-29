---
title: Google Kubernetes Engine
description: Instructions to set up a Google Kubernetes Engine cluster for Istio.
weight: 20
skip_seealso: true
aliases:
    - /docs/setup/kubernetes/prepare/platform-setup/gke/
    - /docs/setup/kubernetes/platform-setup/gke/
keywords: [platform-setup,kubernetes,gke,google]
owner: istio/wg-environments-maintainers
test: no
---

Istio가 구동되는 GKE 클러스터 준비를 위해 아래 설명을 따르세요.

1. 새로운 클러스터를 생성합니다.

    {{< text bash >}}
    $ export PROJECT_ID=`gcloud config get-value project` && \
      export M_TYPE=n1-standard-2 && \
      export ZONE=us-west2-a && \
      export CLUSTER_NAME=${PROJECT_ID}-${RANDOM} && \
      gcloud services enable container.googleapis.com && \
      gcloud container clusters create $CLUSTER_NAME \
      --cluster-version latest \
      --machine-type=$M_TYPE \
      --num-nodes 4 \
      --zone $ZONE \
      --project $PROJECT_ID
    {{< /text >}}

    {{< tip >}}
    Istio의 디폴트 설치는 최소 한개 초과(>1)의 vCPU가 포함된 노드를 요구합니다. 만약 [demo 설정 프로파일](/docs/setup/additional-setup/config-profiles/)에 따라 설치를 진행하는 경우, `--machine-type` 인수를 제거하여 더 작은 `n1-standard-1` 머신 사이즈를 사용할 수 있습니다.
    {{< /tip >}}

    {{< warning >}}
    GKE에서 Istio CNI 기능을 사용하고자 한다면 [CNI 설치 가이드](/docs/setup/additional-setup/cni/#prerequisites)를 통해 클러스터의 필수 구성요소 설정 방법을 확인하세요.
    {{< /warning >}}

    {{< warning >}}
    **비공개(private) GKE 클러스터의 경우**

    자동으로 생성된 방화벽 규칙은 15017 포트를 열지 못합니다. 이 포트는 Pilot discovery validation webhook에 필요합니다.

    마스터 액세스를 위해 이 방화벽 규칙을 검토하려면:

    {{< text bash >}}
    $ gcloud compute firewall-rules list --filter="name~gke-${CLUSTER_NAME}-[0-9a-z]*-master"
    {{< /text >}}

    기존 규칙을 바꿔 마스터 액세스를 허용하려면:

    {{< text bash >}}
    $ gcloud compute firewall-rules update <firewall-rule-name> --allow tcp:10250,tcp:443,tcp:15017
    {{< /text >}}

    {{< /warning >}}

1. `kubectl`에 대한 자격 증명을 얻으세요.

    {{< text bash >}}
    $ gcloud container clusters get-credentials $CLUSTER_NAME \
        --zone $ZONE \
        --project $PROJECT_ID
    {{< /text >}}

1. 현재 사용자에게 클러스터 관리자 (admin) 권한을 부여하세요. 현재 사용자는 Istio를 위해 필요한 RBAC 규칙을 생성해야 하기에
   관리자 권한이 필요합니다.

    {{< text bash >}}
    $ kubectl create clusterrolebinding cluster-admin-binding \
        --clusterrole=cluster-admin \
        --user=$(gcloud config get-value core/account)
    {{< /text >}}

## 다중 클러스터 통신

경우에 따라 클러스터 간 트래픽 전송을 위해 방화벽 규칙을 명시적으로 생성해야 합니다.

{{< warning >}}
아래 설명은 당신의 프로젝트 내 *모든* 클러스터 간 통신을 허용해 줄 것입니다. 필요에 따라 명령어를 수정하세요.
{{< /warning >}}

1. 클러스터 네트워크에 대한 정보를 수집합니다.

    {{< text bash >}}
    $ function join_by { local IFS="$1"; shift; echo "$*"; }
    $ ALL_CLUSTER_CIDRS=$(gcloud --project $PROJECT_ID container clusters list --format='value(clusterIpv4Cidr)' | sort | uniq)
    $ ALL_CLUSTER_CIDRS=$(join_by , $(echo "${ALL_CLUSTER_CIDRS}"))
    $ ALL_CLUSTER_NETTAGS=$(gcloud --project $PROJECT_ID compute instances list --format='value(tags.items.[0])' | sort | uniq)
    $ ALL_CLUSTER_NETTAGS=$(join_by , $(echo "${ALL_CLUSTER_NETTAGS}"))
    {{< /text >}}

1. 방화벽 규칙을 생성합니다.

    {{< text bash >}}
    $ gcloud compute firewall-rules create istio-multicluster-pods \
        --allow=tcp,udp,icmp,esp,ah,sctp \
        --direction=INGRESS \
        --priority=900 \
        --source-ranges="${ALL_CLUSTER_CIDRS}" \
        --target-tags="${ALL_CLUSTER_NETTAGS}" --quiet
    {{< /text >}}
