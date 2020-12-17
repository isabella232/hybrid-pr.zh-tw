---
title: 在 Azure Stack Hub 上部署高可用性 Kubernetes 叢集
description: 了解如何使用 Azure 和 Azure Stack Hub 部署 Kubernetes 叢集來提供高可用性。
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: 91f5856aa670bf3810baa5e5f07dbb7dafc9e3f3
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/09/2020
ms.locfileid: "96911919"
---
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a>在 Azure Stack Hub 上部署高可用性 Kubernetes 叢集

本文將說明如何在不同的實體位置中，建置高可用性且部署在多個 Azure Stack Hub 執行個體上的 Kubernetes 叢集環境。

在此解決方案部署指南中，您將了解如何：

> [!div class="checklist"]
> - 下載並準備 AKS 引擎
> - 連線到 AKS 引擎協助程式 VM
> - 部署 Kubernetes 叢集
> - 連線至 Kubernetes 叢集
> - 將 Azure Pipelines 連線至 Kubernetes 叢集
> - 設定監視
> - 部署應用程式
> - 自動調整應用程式
> - 設定流量管理員
> - 升級 Kubernetes
> - 調整 Kubernetes

> [!Tip]  
> ![混合式支柱](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub 是 Azure 的延伸模組。 Azure Stack Hub 可將雲端運算的靈活性與創新能力導入您的內部部署環境中，並啟用獨特的混合式雲端，讓您能夠隨處建置及部署混合式應用程式。  
> 
> [混合式應用程式設計考量](overview-app-design-considerations.md)一文檢閱了設計、部署和操作混合式應用程式時的軟體品質要素 (放置、延展性、可用性、復原、管理性和安全性)。 這些設計考量有助於您設計出最佳的混合式應用程式，減少生產環境可能會遇到的挑戰。

## <a name="prerequisites"></a>Prerequisites

開始使用此部署指南之前，請務必先：

- 請參閱[高可用性 Kubernetes 叢集模式](pattern-highly-available-kubernetes.md)一文。
- 請參閱[附屬 GitHub 存放庫](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub)的內容，其中包含本文中所參考的其他資產。
- 擁有可存取 [Azure Stack Hub 使用者入口網站](/azure-stack/user/azure-stack-use-portal)的帳戶，且至少具有[「參與者」權限](/azure-stack/user/azure-stack-manage-permissions)。

## <a name="download-and-prepare-aks-engine"></a>下載並準備 AKS 引擎

AKS 引擎是一個二進位檔，可以從任何可連線至 Azure Stack Hub Azure Resource Manager 端點的 Windows 或 Linux 主機中使用。 本指南說明如何在 Azure Stack Hub 上部署新的 Linux (或 Windows) VM。 稍後會在 AKS 引擎部署 Kubernetes 叢集時用到。

> [!NOTE]
> 您也可以在現有的 Windows 或 Linux VM 上，使用 AKS 引擎在 Azure Stack Hub 上部署 Kubernetes 叢集。

AKS 引擎的逐步程序和需求記載於此處：

* [在 Azure Stack Hub 中的 Linux 上安裝 AKS 引擎](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (或使用 [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))

AKS 引擎是一個協助程式工具，可用於部署和操作 (非受控) Kubernetes 叢集 (在 Azure 和 Azure Stack Hub 中)。

如需 Azure Stack Hub 上的 AKS 引擎詳細資料和差異，請參閱：

* [Azure Stack Hub 上的 AKS 引擎是什麼？](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* [Azure Stack Hub 上的 AKS 引擎](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (GitHub 上)

範例環境會使用 Terraform 將 AKS 引擎 VM 的部署自動化。 您可以在[附屬的 GitHub 存放庫中找到詳細資料和程式碼](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md)。

此步驟會在 Azure Stack Hub 上產生新的資源群組，且資源群組中會包含 AKS 引擎協助程式 VM 和相關資源：

![Azure Stack Hub 中的 AKS 引擎 VM 資源](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> 如果您必須在中斷連線的網路隔絕環境中部署 AKS 引擎，請參閱[中斷連線的 Azure Stack Hub 執行個體](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances)以深入了解。

在下一個步驟中，我們將使用新部署的 AKS 引擎 VM 來部署 Kubernetes 叢集。

## <a name="connect-to-the-aks-engine-helper-vm"></a>連線到 AKS 引擎協助程式 VM

首先，您必須連線到先前建立的 AKS 引擎協助程式 VM。

VM 應具有公用 IP 位址，而且應該可透過 SSH (連接埠 22/TCP) 存取。

![AKS 引擎 VM 概觀頁面](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> 您可以使用您選擇的工具 (例如 Windows 10 中的 MobaXterm、puTTY 或 PowerShell)，以使用 SSH 連線到 Linux VM。

```console
ssh <username>@<ipaddress>
```

連線之後，請執行 `aks-engine` 命令。 若要深入了解 AKS 引擎和 Kubernetes 版本，請移至[支援的 AKS 引擎版本](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions)。

![aks-engine 命令列範例](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a>部署 Kubernetes 叢集

AKS 引擎協助程式 VM 本身尚未在我們的 Azure Stack Hub 上建立 Kubernetes 叢集。 建立叢集是要在 AKS 引擎協助程式 VM 中採取的第一個動作。

逐步程序記載於此處：

* [在 Azure Stack Hub 上使用 AKS 引擎部署 Kubernetes 叢集](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

`aks-engine deploy` 命令和先前步驟中準備工作的最終結果是功能完整的 Kubernetes 叢集，並且會部署到第一個 Azure Stack Hub 執行個體的租用戶空間。 叢集本身包含 Azure IaaS 元件，例如 VM、負載平衡器、Vnet、磁碟等等。

![Azure Stack Hub 入口網站的叢集 IaaS 元件](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) Azure 負載平衡器 (K8s API 端點)
2) 背景工作節點 (代理程式集區)
3) 主要節點

叢集現在已啟動且正在執行，而且在下一個步驟中，我們將連線到該叢集。

## <a name="connect-to-the-kubernetes-cluster"></a>連線至 Kubernetes 叢集

您現在可以透過 SSH 連線到先前建立的 Kubernetes 叢集 (使用部署中指定的 SSH 金鑰) 或透過 `kubectl` (建議)。 若要了解適用於 Windows、Linux 和 macOS 的 Kubernetes 命令列工具，請參閱`kubectl`[此處](https://kubernetes.io/docs/tasks/tools/install-kubectl/)。 其已在叢集的主要節點上預先安裝並設定。

```console
ssh azureuser@<k8s-master-lb-ip>
```

![在主要節點上執行 kubectl](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

不建議使用主要節點作為系統管理工作的 jumpbox。 `kubectl` 設定會儲存在主要節點和 AKS 引擎 VM 上的 `.kube/config` 中。 您可以將設定複製到具有 Kubernetes 叢集連線的管理員機器，並在該處使用 `kubectl` 命令。 `.kube/config` 檔案稍後也會用來設定 Azure Pipelines 中的服務連線。

> [!IMPORTANT]
> 保護這些檔案的安全，因為其中包含 Kubernetes 叢集的認證。 具有檔案存取權的攻擊者會具有足夠的資訊來取得其管理員存取權。 使用初始 `.kube/config` 檔案完成的所有動作都是使用叢集管理員帳戶完成。

您現在可以使用 `kubectl` 來嘗試各種命令，以檢查叢集的狀態。

```bash
kubectl get nodes
```

```console
NAME                       STATUS   ROLE     VERSION
k8s-linuxpool-35064155-0   Ready    agent    v1.14.8
k8s-linuxpool-35064155-1   Ready    agent    v1.14.8
k8s-linuxpool-35064155-2   Ready    agent    v1.14.8
k8s-master-35064155-0      Ready    master   v1.14.8
```

```bash
kubectl cluster-info
```

```console
Kubernetes master is running at https://aks.***
CoreDNS is running at https://aks.**_/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
kubernetes-dashboard is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy
Metrics-server is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
```

> [!IMPORTANT]
> Kubernetes 有自己的「角色型存取控制 (RBAC)」模型，可讓您建立更細緻的角色定義和角色繫結。 比起給予叢集管理員權限，這是較理想的叢集存取控制方式。

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a>將 Azure Pipelines 連線至 Kubernetes 叢集

若要將 Azure Pipelines 連線到新部署的 Kubernetes 叢集，我們需要其 kube config (`.kube/config`) 檔案，如上一個步驟所述。

* 連線到 Kubernetes 叢集的其中一個主要節點。
* 複製 `.kube/config` 檔案的內容。
* 移至 [Azure DevOps] > [專案設定] > [服務連線]，以建立新的 "Kubernetes" 服務連線 (使用 KubeConfig 作為驗證方法)

> [!IMPORTANT]
> Azure Pipelines (或其組建代理程式) 必須具有 Kubernetes API 的存取權。 如果存在從 Azure Pipelines 到 Azure Stack Hub Kubernetes 叢集的網際網路連線，您將需要部署自我裝載的 Azure Pipelines 組建代理程式。

為 Azure Pipelines 部署自我裝載式代理程式時，您可以在 Azure Stack Hub 上或在具有網路連線的機器上，對所有必要的管理端點進行部署。 請在此參閱詳細資料：

* [Windows](/azure/devops/pipelines/agents/v2-windows) 或 [Linux](/azure/devops/pipelines/agents/v2-linux) 上的 [Azure Pipelines 代理程式](/azure/devops/pipelines/agents/agents)

[部署 (CI/CD) 考量](pattern-highly-available-kubernetes.md#deployment-cicd-considerations)一節包含的決策流程可協助您了解要使用 Microsoft 裝載的代理程式或自我裝載的代理程式：

[![決策流程自我裝載式代理程式](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)

此範例解決方案中的拓撲包含每個 Azure Stack Hub 執行個體上的自我裝載式組建代理程式。 代理程式可以存取 Azure Stack Hub 管理端點和 Kubernetes 叢集 API 端點。

[![僅輸出流量](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)

這項設計可滿足一般的法規需求，也就是要有來自應用程式解決方案的輸出連線。

## <a name="configure-monitoring"></a>設定監視

您可以使用適用於容器的 [Azure 監視器](/azure/azure-monitor/) 來監視解決方案中的容器。 這會將 Azure 監視器指向 Azure Stack Hub 上由 AKS 引擎部署的 Kubernetes 叢集。

有兩種方式可以為您的叢集啟用 Azure 監視器。 這兩種方式都需要您在 Azure 中設定 Log Analytics 工作區。

* [方法一](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one)是使用 Helm 圖表
* [方法二](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two)是作為 AKS 引擎叢集規格的一部分

範例拓撲會使用「方法一」，這可允許程序自動化並輕鬆地安裝更新。

在下一個步驟中，您的機器上需要有 Azure LogAnalytics 工作區 (識別碼和金鑰)、`Helm` (第3版) 和 `kubectl`。

Helm 是 Kubernetes 套件管理員，可作為在 macOS、Windows 和 Linux 上執行的二進位檔。 可以在此下載：[helm.sh](https://helm.sh/docs/intro/quickstart/) Helm 會依賴用於 `kubectl` 命令的 Kubernetes 設定檔。

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

此命令會在您的 Kubernetes 叢集上安裝 Azure 監視器代理程式：

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

Kubernetes 叢集上的 Operations Management Suite (OMS) 代理程式會將監視資料傳送至您的 Azure Log Analytics 工作區 (使用輸出 HTTPS)。 您現在可以使用 Azure 監視器，在 Azure Stack Hub 上取得更深入的 Kubernetes 叢集深入解析。 此設計有利於說明透過應用程式叢集自動部署的分析有多強大。

[![在 Azure 監視器中 Azure Stack Hub 叢集](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)

[![Azure 監視器叢集詳細資料](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)

> [!IMPORTANT]
> 如果 Azure 監視器未顯示任何 Azure Stack Hub 資料，請確定您已遵循＜[如何將 AzureMonitor-Containers 解決方案新增至 Azure Loganalytics 工作區](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md)＞的指示進行。

## <a name="deploy-the-application"></a>部署應用程式

在安裝範例應用程式之前，還有另一個在 Kubernetes 叢集上設定 nginx 型輸入控制器的步驟。 輸入控制器會當做第 7 層負載平衡器使用，以根據主機、路徑或通訊協定來路由叢集中的流量。 Nginx 輸入是以 Helm 圖表的形式提供。 如需詳細指示，請參閱 [Helm 圖表的 GitHub 存放庫](https://github.com/helm/charts/tree/master/stable/nginx-ingress)。

我們的範例應用程式也會封裝為 Helm 圖表，例如上一個步驟中的 [Azure 監視代理程式](#configure-monitoring)。 因此，可直接將應用程式部署到我們的 Kubernetes 叢集上。 您可以在[附屬的 GitHub 存放庫中找到 Helm 圖表檔案](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)

範例應用程式是三層應用程式，並且已部署到兩個 Azure Stack Hub 執行個體的 Kubernetes 叢集上。 應用程式會使用 MongoDB 資料庫。 若要深入了解如何在多個執行個體之間複寫資料，請參閱[資料和儲存體考量](pattern-highly-available-kubernetes.md#data-and-storage-considerations)一節。

部署應用程式的 Helm 圖表之後，您會看到應用程式的三個層級以單一 Pod 表示為部署和 StatefulSet (適用於資料庫)：

```kubectl
kubectl get pod,deployment,statefulset
```

```console
NAME                                         READY   STATUS
pod/ratings-api-569d7f7b54-mrv5d             1/1     Running
pod/ratings-mongodb-0                        1/1     Running
pod/ratings-web-85667bfb86-l6vxz             1/1     Running

NAME                                         READY
deployment.extensions/ratings-api            1/1
deployment.extensions/ratings-web            1/1

NAME                                         READY
statefulset.apps/ratings-mongodb             1/1
```

在服務端上，您會找到 nginx 型輸入控制器及其公用 IP 位址：

```kubectl
kubectl get service
```

```console
NAME                                         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)
kubernetes                                   ClusterIP      10.0.0.1       <none>        443/TCP
nginx-ingress-1588931383-controller          LoadBalancer   10.0.114.180   *public-ip*   443:30667/TCP
nginx-ingress-1588931383-default-backend     ClusterIP      10.0.76.54     <none>        80/TCP
ratings-api                                  ClusterIP      10.0.46.69     <none>        80/TCP
ratings-web                                  ClusterIP      10.0.161.124   <none>        80/TCP
```

「外部 IP」位址是我們的「應用程式端點」。 這是使用者連線並開啟應用程式的方式，而且也會作為下一個步驟＜[設定流量管理員](#configure-traffic-manager)＞的端點。

## <a name="autoscale-the-application"></a>自動調整應用程式
您可以選擇性地設定[水平 Pod 自動調整程式](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)，以根據特定的計量 (例如 CPU 使用率) 來擴大或縮小規模。 下列命令會建立水平 Pod 自動調整程式，以維護 1 到 10 個由 ratings-web 部署控制的 Pod 複本。 HPA 會增加及減少複本的數目 (透過部署)，將所有 Pod 的平均 CPU 使用率維持在 80%。

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
您可以執行下列動作來檢查自動調整程式的目前狀態：

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a>設定流量管理員

為了在應用程式的兩個 (或多個) 部署之間分散流量，我們將使用 [Azure 流量管理員](/azure/traffic-manager/traffic-manager-overview)。 Azure 流量管理員是 Azure 中的 DNS 型流量負載平衡器。

> [!NOTE]
> 流量管理員會使用 DNS，根據流量路由方法和端點的健康情況，將用戶端要求導向最適當的服務端點。

您也可以使用裝載於內部部署環境的其他全域負載平衡解決方案，而不是使用 Azure 流量管理員。 在範例案例中，我們會使用 Azure 流量管理員在應用程式的兩個執行個體之間分散流量。 其可以在相同或不同位置的 Azure Stack Hub 執行個體上執行：

![內部部署流量管理員](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

在 Azure 中，我們會將流量管理員設定為指向應用程式的兩個不同執行個體：

[![TM 端點設定檔](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)

如您所見，這兩個端點會指向[上一節](#deploy-the-application)中已部署的兩個應用程式執行個體。

至此：
- 您已建立 Kubernetes 基礎結構，包括輸入控制器。
- 叢集已部署到兩個 Azure Stack Hub 執行個體。
- 已設定監視。
- Azure 流量管理員會將流量負載平衡到兩個 Azure Stack Hub 執行個體。
- 在此基礎結構之上，三層式應用程式範例已使用 Helm 圖表自動地進行部署。 

解決方案現在應該已啟動並可供使用者存取！

此外，還有一些部署後的操作考量值得討論，這會在下面兩節中討論。

## <a name="upgrade-kubernetes"></a>升級 Kubernetes

升級 Kubernetes 叢集時，請考慮下列主題：

- 升級 Kubernetes 叢集是可使用 AKS 引擎來完成但複雜的第 2 天 (Day 2) 作業。 如需詳細資訊，請參閱[升級 Azure Stack Hub 上的 Kubernetes 叢集](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade)。
- AKS 引擎可讓您將叢集升級為較新的 Kubernetes 和基礎 OS 映像版本。 如需詳細資訊，請參閱[升級至較新 Kubernetes 版本的步驟](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version)。 
- 您也可以只將基礎節點升級為較新的基礎 OS 映像版本。 如需詳細資訊，請參閱[僅升級 OS 映像的步驟](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image)。

較新的基礎 OS 映像包含安全性和核心更新。 叢集操作員必須負責監視較新的 Kubernetes 版本和 OS 映像的可用性。 操作員應該使用 AKS 引擎來規劃和執行這些升級。 基礎 OS 映像必須由 Azure Stack Hub 操作員從 Azure Stack Hub Marketplace 下載。

## <a name="scale-kubernetes"></a>調整 Kubernetes

調整是另一個第 2 天 (Day 2) 作業，可以使用 AKS 引擎進行協調。

調整命令會重複使用輸出目錄內的叢集設定檔 (apimodel.json)，以作為新 Azure Resource Manager 部署的輸入。 AKS 引擎會針對特定的代理程式集區執行調整作業。 當調整作業完成時，AKS 引擎就會更新該相同 apimodel.json 檔案中的叢集定義。 叢集定義會反映新的節點計數，以反映更新後的目前叢集設定。

- [在 Azure Stack Hub 上調整 Kubernetes 叢集](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a>後續步驟

- 深入了解[混合式應用程式設計考量](overview-app-design-considerations.md)
- 檢閱 [GitHub 上此範例的程式碼](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub)並提出改進建議。