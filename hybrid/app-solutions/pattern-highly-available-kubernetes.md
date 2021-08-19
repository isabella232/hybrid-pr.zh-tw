---
title: 使用 Azure 和 Azure Stack Hub 的高可用性 Kubernetes 模式
description: 了解 Kubernetes Cluster 解決方案如何使用 Azure 和 Azure Stack Hub 提供高可用性。
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: f8a733bcdab871695e552ec687d42e3ff4230490
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281307"
---
# <a name="high-availability-kubernetes-cluster-pattern"></a>高可用性 Kubernetes 叢集模式

本文說明如何使用 Azure Stack Hub 上的 Azure Kubernetes Service (AKS) 引擎來建構和操作高可用性的 Kubernetes 式基礎結構。 適用此案例的組織通常具有極重要的工作負載，且處在高度受限及受管制的環境中。 例如財務、國防和政府領域中的組織。

## <a name="context-and-problem"></a>內容和問題

許多組織都正在開發雲端原生解決方案，以運用最先進的服務和技術，例如 Kubernetes。 雖然 Azure 在全球大多數區域都提供資料中心，但有時候會有商務關鍵性應用程式必須在特定位置執行的邊緣使用案例和情況。 考量項目包括：

- 位置敏感度
- 應用程式與內部部署系統之間的延遲
- 頻寬節約
- 連線能力
- 法規或法定需求

Azure 與 Azure Stack Hub 結合可解決大部分的疑慮。 以下說明在 Azure Stack Hub 上成功實作 Kubernetes 的一組廣泛選項、決策及考量。

## <a name="solution"></a>解決方案

此模式假設我們必須處理一組嚴格的條件約束。 應用程式必須在內部部署環境中執行，而且所有個人資料都不得連線到公用雲端服務。 監視資料和其他非 PII 資料可以傳送至 Azure，並在該處進行處理。 可以存取外部服務 (例如公用 Container Registry 或其他服務)，但可能會透過防火牆或 Proxy 伺服器進行篩選。

此處顯示的範例應用程式 (以 [Azure Kubernetes Service 工作坊](/learn/modules/aks-workshop/)為基礎) 已設計為盡可能使用 Kubernetes 原生解決方案。 比起使用平台原生服務，此設計可避免廠商鎖定。 例如，應用程式會使用自我裝載的 MongoDB 資料庫後端，而不是 PaaS 服務或外部資料庫服務。

[![混合的應用程式模式](media/pattern-highly-available-kubernetes/application-architecture.png)](media/pattern-highly-available-kubernetes/application-architecture.png#lightbox)

上圖說明 Azure Stack Hub 中範例應用程式執行於 Kubernetes 時的應用程式架構。 應用程式是由數個元件所組成，包括：

 1) Azure Stack Hub 上以 AKS 引擎為基礎的 Kubernetes 叢集。
 2) [cert-manager](https://www.jetstack.io/cert-manager/)，這會在 Kubernetes 中提供一套憑證管理工具套件，用來自動向 Let's Encrypt 要求憑證。
 3) Kubernetes 命名空間，其中包含前端 (ratings-web)、API (ratings-api) 和資料庫 (ratings-mongodb) 的應用程式元件。
 4) 將 HTTP/HTTPS 流量路由至 Kubernetes 叢集中端點的輸入控制器。

範例應用程式的用途是說明應用程式架構。 所有元件都是範例。 此架構僅包含單一應用程式部署。 為了達到高可用性 (HA)，我們會在兩個不同的 Azure Stack Hub 執行個體上執行部署至少兩次 (可以在相同位置或兩個以上不同的網站中執行)：

![基礎結構架構](media/pattern-highly-available-kubernetes/aks-azure-architecture.png)

Azure Container Registry、Azure 監視器及其他等服務都會裝載在 Azure 或內部部署的 Azure Stack Hub 外部。 此混合式設計可防止解決方案發生單一 Azure Stack Hub 執行個體中斷的情況。

## <a name="components"></a>元件

整體架構是由下列元件組成：

**Azure Stack Hub** 是 Azure 的一項擴充功能，可在資料中心提供 Azure 服務，以便您在內部部署環境中執行應用程式。 請移至 [Azure Stack Hub 概觀](/azure-stack/operator/azure-stack-overview)以深入了解。

**Azure Kubernetes Service 引擎 (AKS 引擎)** 是受控 Kubernetes 服務供應項目 Azure Kubernetes Service (AKS) 背後的引擎，目前已可在 Azure 中使用。 針對 Azure Stack Hub，AKS 引擎可讓我們使用 Azure Stack Hub 的 IaaS 功能來部署、調整和升級功能完整且自我管理的 Kubernetes 叢集。 若要深入了解，請移至 [AKS 引擎概觀](https://github.com/Azure/aks-engine)。

若要深入了解 Azure 上的 AKS 引擎與 Azure Stack Hub 上的 AKS Engine 之間有何差異，請移至[已知問題和限制](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#known-issues-and-limitations)。

針對裝載 Kubernetes 叢集基礎結構的虛擬機器 (VM)，**Azure 虛擬網路 (VNet)** 可用來為其中每個 Azure Stack Hub 提供網路基礎結構。

Kubernetes API 端點和 Nginx 輸入控制器會使用 **Azure Load Balancer**。 此負載平衡器會將外部 (例如網際網路) 流量路由至提供特定服務的節點和 VM。

**Azure Container Registry (ACR)** 會用來儲存私人 Docker 映像，以及部署至叢集的 Helm 圖表。 AKS 引擎可以使用 Azure AD 身分識別向 Container Registry 進行驗證。 Kubernetes 不需要 ACR。 您可以使用其他容器登錄，例如 Docker Hub。

**Azure Repos** 是一組版本控制工具，可用來管理程式碼。 您也可以使用 GitHub 或其他以 git 為基礎的存放庫。 若要深入了解，請移至 [Azure Repos 概觀](/azure/devops/repos/get-started/what-is-repos)。

**Azure Pipelines** 是 Azure DevOps Services 的一部分，可執行自動化組建、測試及部署。 您也可以使用 Jenkins 等第三方 CI/CD 解決方案。 若要深入了解，請移至 [Azure Pipeline 概觀](/azure/devops/pipelines/get-started/what-is-azure-pipelines)。

**Azure 監視器** 會收集並儲存計量和記錄，包括解決方案中 Azure 服務的平台計量和應用程式遙測。 您可以使用此資料來監視應用程式、設定警示和儀表板，以及對失敗執行根本原因分析。 Azure 監視器會與 Kubernetes 整合，以收集控制器、節點和容器中的計量，以及容器記錄和主要節點記錄。 若要深入了解，請移至 [Azure 監視器概觀](/azure/azure-monitor/overview)。

**Azure 流量管理員** 是以 DNS 為基礎的流量負載平衡器，可讓您以最佳方式將流量分散到不同 Azure 區域或 Azure Stack Hub 部署上的服務。 流量管理員也會提供高可用性和回應性。 應用程式端點必須可從外部存取。 還有其他內部部署解決方案也可供使用。

**Kubernetes 輸入控制器** 會將 HTTP (S) 路由公開給 Kubernetes 叢集中的服務。 Nginx 或任何適當的輸入控制器都可以用於此用途。

**Helm** 是 Kubernetes 部署的套件管理員，可將不同的 Kubernetes 物件 (例如部署、服務、祕密) 組合成單一「圖表」。 您可以發行、部署、控制版本管理並更新圖表物件。 Azure Container Registry 可當作存放庫來儲存已封裝的 Helm 圖表。

## <a name="design-considerations"></a>設計考量

此模式會遵循一些高階考量，本文的後續章節中會有更詳細的說明：

- 應用程式會使用 Kubernetes 原生解決方案，以避免廠商鎖定。
- 應用程式會使用微服務架構。
- Azure Stack Hub 不需要輸入連線，但允許輸出網際網路連線。

這些建議的作法也適用於真實世界的工作負載和案例。

## <a name="scalability-considerations"></a>延展性考量

可擴縮性非常重要，因為要為使用者提供一致、可靠且高效能的應用程式存取。

範例案例涵蓋應用程式堆疊的多層次可擴縮性。 以下是不同等級的高階概觀：

| 架構層級 | 影響 | 怎麼做？ |
| --- | --- | ---
| 應用程式 | 應用程式 | 根據 Pod/複本/容器實執行個體的數目進行水平調整* |
| 叢集 | Kubernetes 叢集 | 節點數目 (介於 1 到 50)、VM SKU 大小和節點集區 (Azure Stack Hub 的 AKS 引擎目前僅支援單一節點集區)；使用 AKS 引擎的調整命令 (手動) |
| 基礎結構 | Azure Stack Hub | Azure Stack Hub 部署內的節點數目、容量和縮放單位數目 |

\* 使用 Kubernetes 的水平 Pod 自動調整程式 (HPA)；調整容器執行個體的大小 (CPU/記憶體) 可進行自動化的計量式調整或垂直調整。

**Azure Stack Hub (基礎結構層級)**

Azure Stack Hub 基礎結構是此實作的基礎，因為 Azure Stack Hub 是在資料中心內的實體硬體上執行。 選取您的 Hub 硬體時，您需要選擇 CPU、記憶體密度、儲存體設定和伺服器數目。 若要深入了解 Azure Stack Hub 的可擴縮性，請參閱下列資源：

- [Azure Stack Hub 的容量規劃概觀](/azure-stack/operator/azure-stack-capacity-planning-overview)
- [在 Azure Stack Hub 中新增更多縮放單位節點](/azure-stack/operator/azure-stack-add-scale-node)

**Kubernetes 叢集 (叢集層級)**

Kubernetes 叢集本身包含並以其作為建置基礎的 Azure (Stack) IaaS 元件包括計算、儲存體和網路資源。 Kubernetes 解決方案包含主要和背景工作節點，這些都會部署為 Azure (和 Azure Stack Hub) 中的 VM。

- [控制平面節點](/azure/aks/concepts-clusters-workloads#control-plane) (主要) 提供核心 Kubernetes 服務和應用程式工作負載的協調流程。
- [背景工作節點](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools) (背景工作角色) 會執行您的應用程式工作負載。

選取初始部署的 VM 大小時，有幾個考量點：  

- **成本** - 規劃背景工作節點時，請注意每個 VM 產生的整體成本。 例如，如果您的應用程式工作負載需要的資源有限，您應該規劃部署大小較小的 VM。 Azure Stack Hub (例如 Azure) 通常會以耗用量計費，因此適當地調整 Kubernetes 角色的 VM 大小，對於最佳化耗用量成本非常重要。 

- **可擴縮性** - 縮減和擴增主要和背景工作節點的數目，或藉由新增其他節點集區，都可完成叢集的擴縮 (目前不適用於 Azure Stack Hub)。 調整叢集規模可以根據以容器深入解析 (Azure 監視器 + Log Analytics) 所收集的效能資料來完成。 

    如果您的應用程式需要更多 (或更少) 資源，您可以水平擴增 (或縮減) 您目前的節點 (介於 1 到 50 個節點之間)。 如果您需要超過 50 個節點，您可以在個別的訂用帳戶中建立額外叢集。 若不重新部署叢集，您無法將實際的 VM 垂直擴大為另一個 VM 大小。

    您可以使用一開始用來部署 Kubernetes 叢集的 AKS 引擎協助程式 VM 來手動調整規模。 如需詳細資訊，請參閱[調整 Kubernetes 叢集](https://github.com/Azure/aks-engine/blob/master/docs/topics/scale.md)

- **配額** - 請考量您在 Azure Stack Hub 上規劃 AKS 部署時所設定的 [配額](/azure-stack/operator/azure-stack-quota-types)。 請確定每個[訂用帳戶](/azure-stack/operator/service-plan-offer-subscription-overview)都已設定適當的方案和配額。 擴增叢集時，訂用帳戶必須可容納您叢集所需的計算、儲存體和其他服務數量。

- **應用程式工作負載** - 請在 Azure Kubernetes Service 文件的 Kubernetes 核心概念中參閱＜[叢集和工作負載概念](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools)＞。 本文將協助您根據應用程式的計算和記憶體需求來界定適當的 VM 大小範圍。  

**應用程式 (應用程式層級)**

在應用程式層次中，我們使用了 Kubernetes [水平 Pod 自動調整程式 (HPA)](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)。 HPA 可以根據不同的計量 (例如 CPU 使用率) 來增加或減少部署中的複本數目 (Pod/容器執行個體)。

另一個選項是垂直調整容器執行個體的規模。 這可以藉由變更要求的 CPU 和記憶體數量，以及特定部署的可用空間來完成。 若要深入了解，請參閱 kubernetes.io 上的[管理容器資源](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)。

## <a name="networking-and-connectivity-considerations"></a>網路功能和連線能力的考量

針對 Azure Stack Hub 上的 Kubernetes，網路功能和連線能力也會影響先前所述的三個層級。 下表顯示各層次及其包含的服務：

| 層 | 影響 | 何事？ |
| --- | --- | ---
| 應用程式 | 應用程式 | 如何存取應用程式？ 是否會公開至網際網路？ |
| 叢集 | Kubernetes 叢集 | Kubernetes API、AKS 引擎 VM、提取容器映像 (輸出)、傳送監視資料和遙測 (輸出) |
| 基礎結構 | Azure Stack Hub | Azure Stack Hub 管理端點 (例如入口網站和 Azure Resource Manager 端點) 的可存取性。 |

**應用程式**

對於應用程式層，最重要的考量是應用程式是否公開，以及是否可從網際網路存取。 從 Kubernetes 的觀點來看，網際網路的可存取性代表著使用 Kubernetes 服務或輸入控制器來公開部署或 Pod。

> [!NOTE]
> 我們建議使用輸入控制器來公開 Kubernetes 服務，因為 Azure Stack Hub 上的前端公用 IP 數目會限制為 5 個。 此設計也會將 Kubernetes 服務 (類型為 LoadBalancer) 的數目限制為 5 個，而這對許多部署而言太小。 若要深入了解，請移至 [AKS 引擎文件](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#limited-number-of-frontend-public-ips)。

透過 Load Balancer 或輸入控制器使用公用 IP 來公開應用程式，並不一定表示應用程式現在可以透過網際網路存取。 Azure Stack Hub 具有的公用 IP 位址可能只會顯示在本地的內部網路上，並非所有公用 IP 都是真正地公開於網際網路。

先前的區塊考量到應用程式的輸入流量。 要成功部署 Kubernetes 的另一個必須考量的主題是輸出流量。 以下是幾個需要輸出流量的使用案例：

- 提取儲存 DockerHub 或 Azure Container Registry 上儲存的容器映像
- 擷取 Helm 圖表
- 發出 Application Insights 資料 (或其他監視資料)

某些企業環境可能需要使用「透明」或「非透明」的 Proxy 伺服器。 這些伺服器需要在叢集的各種元件上進行特定設定。 AKS 引擎文件包含有關如何配合網路 Proxy 的各種詳細資料。 如需詳細資訊，請參閱 [AKS 引擎和 Proxy 伺服器](https://github.com/Azure/aks-engine/blob/master/docs/topics/proxy-servers.md)

最後，跨叢集流量必須在 Azure Stack Hub 執行個體之間流動。 範例部署是由個別 Azure Stack Hub 執行個體上執行的個別 Kubernetes 叢集所組成。 其間的流量 (例如兩個資料庫之間的複寫流量) 是「外部流量」。 外部流量必須透過站對站 VPN 或 Azure Stack Hub 公用 IP 位址來進行路由，才能連結兩個 Azure Stack Hub 執行個體上的 Kubernetes：

![內部和之間的叢集流量](media/pattern-highly-available-kubernetes/aks-inter-and-intra-cluster-traffic.png)

**Cluster**  

Kubernetes 叢集不一定需要透過網際網路存取。 相關部分是用來操作叢集的 Kubernetes API，例如，使用 `kubectl`。 所有操作叢集或在其中部署應用程式和服務的人員都必須可存取 Kubernetes API 端點。 本主題將詳細說明[部署 (CI/CD) 考量](#deployment-cicd-considerations)一節中的 DevOps 觀點。

在叢集層級上，還有一些關於輸出流量的考量：

- 節點更新 (適用於 Ubuntu)
- 監視資料 (傳送至 Azure LogAnalytics)
- 需要輸出流量的其他代理程式 (專屬於每個部署人員的環境)

使用 AKS 引擎部署 Kubernetes 叢集之前，請先規劃最終的網路設計。 比起建立專用虛擬網路，將叢集部署到現有的網路可能會更有效率。 例如，您可以利用已在 Azure Stack Hub 環境中設定的現有站對站 VPN 連線。

**基礎結構**  

基礎結構是指存取 Azure Stack Hub 管理端點。 端點包括租用戶和管理員入口網站，以及 Azure Resource Manager 管理員和用戶端點。 您需要這些端點才能操作 Azure Stack Hub 及其核心服務。

## <a name="data-and-storage-considerations"></a>資料和儲存體考量

應用程式的兩個執行個體會部署在兩個 Azure Stack Hub 執行個體上的兩個個別 Kubernetes 叢集上。 這種設計會要求我們考慮如何複寫和同步兩者之間的資料。

我們可透過 Azure 內建功能在雲端內的多個區域之間複寫儲存體。 目前，Azure Stack Hub 中沒有任何原生方式可跨兩個不同的 Azure Stack Hub 執行個體複寫儲存體，兩個執行個體會形成兩個獨立的雲端，而且沒有完善的方式可將其當做集合來管理。 為跨 Azure Stack Hub 執行的應用程式規劃復原功能，一定會迫使您在應用程式設計和部署中考慮這種獨立性。

在大多數情況下，AKS 上部署的復原性和高可用性應用程式不需要進行儲存體複寫。 但是，您應該在應用程式設計中考量每個 Azure Stack Hub 執行個體的獨立儲存體。 如果此設計是在 Azure Stack Hub 上部署解決方案的顧慮或障礙，Microsoft 合作夥伴可能可藉由提供儲存體附件來解決此問題。 儲存體附件可為跨多個 Azure Stack Hub 和 Azure 的儲存體複寫提供解決方案。 如需詳細資訊，請參閱[合作夥伴解決方案](#partner-solutions)。

在我們的架構中，下列層次會列入考量：

**設定**

設定包括 Azure Stack Hub、AKS 引擎和 Kubernetes 叢集本身的設定。 設定應該盡可能自動化，並在 Git 型版本控制系統 (例如 Azure DevOps 或 GitHub) 中儲存為基礎結構即程式碼。 這些設定無法輕鬆地在多個部署之間同步。 因此，建議您從外部儲存和套用設定，並且使用 DevOps 管線。

**應用程式**

應用程式應該儲存在 Git 架構的存放庫中。 每當有新的部署、應用程式變更或災害復原時，都可以使用 Azure Pipelines 輕鬆地進行部署。

**資料**

在大部分的應用程式設計中，資料都是最重要的考量。 應用程式資料必須在不同的應用程式執行個體之間保持同步。 如果發生中斷情形，資料也需要備份和災害復原策略。

達成這項設計的關鍵在於技術選擇。 以下是以高可用性方式在 Azure Stack Hub 上實作資料庫的一些解決方案範例：

- [將 SQL Server 2016 可用性群組部署至 Azure 和 Azure Stack Hub](/azure-stack/hybrid/solution-deployment-guide-sql-ha)
- [將高度可用的 MongoDB 解決方案部署到 Azure 和 Azure Stack Hub](/azure-stack/hybrid/solution-deployment-guide-mongodb-ha)

針對跨多個位置的資料提供高可用性和復原性解決方案是更複雜的考量。 考量：

- Azure Stack Hub 之間的延遲和網路連線能力。
- 服務和權限的身分識別可用性。 每個 Azure Stack Hub 執行個體都會與外部目錄整合。 部署期間，您會選擇 Azure Active Directory (Azure AD) 或 Active Directory 同盟服務 (ADFS)。 因此，可能會使用可與多個獨立 Azure Stack Hub 執行個體互動的單一身分識別。

## <a name="business-continuity-and-disaster-recovery"></a>商務持續性和災害復原

商務持續性和災害復原 (BCDR) 是 Azure Stack Hub 和 Azure 中的重要主題。 主要差異在於，在 Azure Stack Hub 中，操作員必須管理整個 BCDR 程序。 在 Azure 中，BCDR 的某些部分會由 Microsoft 自動管理。

BCDR 會影響上一節＜[資料和儲存體考量](#data-and-storage-considerations)＞中所述的相同區域：

- 基礎結構/設定
- 應用程式可用性
- 應用程式資料

如上一節所述，這些區域是 Azure Stack Hub 操作員的責任，而且可能會因組織而異。 根據您可用的工具和程序來規劃 BCDR。

**基礎結構與設定**

本節涵蓋 Azure Stack Hub 的實體和邏輯基礎結構及設定。 其中涵蓋管理員和租用戶空間中的動作。

Azure Stack Hub 操作員 (或管理員) 需負責維護 Azure Stack Hub 執行個體。 包括網路、儲存體、身分識別等元件，以及本文範圍以外的其他主題。 若要深入了解 Azure Stack Hub 作業的詳細資訊，請參閱下列資源：

- [使用基礎結構備份服務來復原 Azure Stack Hub 中的資料](/azure-stack/operator/azure-stack-backup-infrastructure-backup)
- [從系統管理員入口網站啟用 Azure Stack Hub 的備份](/azure-stack/operator/azure-stack-backup-enable-backup-console)
- [從重大資料遺失的情況下復原](/azure-stack/operator/azure-stack-backup-recover-data)
- [基礎結構備份服務的最佳做法](/azure-stack/operator/azure-stack-backup-best-practices)

Azure Stack Hub 是將用來部署 Kubernetes 應用程式的平台和網狀架構。 Kubernetes 應用程式的應用程式擁有者將是 Azure Stack Hub 的使用者，並且有權部署解決方案所需的應用程式基礎結構。 在此情況下，應用程式基礎結構即代表使用 AKS 引擎部署的 Kubernetes 叢集及周圍的服務。 這些元件將會部署到 Azure Stack Hub，並受到 Azure Stack Hub 供應項目的限制。 請確定 Kubernetes 應用程式擁有者所接受的供應項目有足夠的容量 (以 Azure Stack Hub 配額表示)，可部署整個解決方案。 如前一節所建議，應用程式部署應該使用基礎結構即程式碼和部署管線 (如 Azure DevOps Azure Pipelines) 來自動化。

如需有關 Azure Stack Hub 供應項目和配額的詳細資訊，請參閱 [Azure Stack Hub 服務、方案、供應項目和訂用帳戶概觀](/azure-stack/operator/service-plan-offer-subscription-overview)

請務必安全地儲存 AKS 引擎設定，包括其輸出。 這些檔案包含用來存取 Kubernetes 叢集的機密資訊，因此必須受到保護，以免公開給非管理員知道。

**應用程式可用性**

應用程式不應該依賴已部署執行個體的備份。 標準做法應是遵循基礎結構即程式碼模式，來完全重新部署應用程式。 例如，使用 Azure DevOps Azure Pipelines 重新部署。 BCDR 程序應該牽涉到將應用程式重新部署到相同或另一個 Kubernetes 叢集。

**應用程式資料**

應用程式資料，是將資料遺失機率降到最低的重要部分。 在上一節中，我們已說明在應用程式的兩個 (或多個) 執行個體之間複寫和同步資料的技術。 由於儲存資料的資料庫基礎結構不同 (MySQL、MongoDB、MSSQL 或其他)，因此也會有不同的資料庫可用性和備份技術可供選擇。

若要達到完整性，建議使用下列其中一個方法：
- 適用於特定資料庫的原生備份方案。
- 正式支援備份和復原您應用程式所用資料庫類型的備份解決方案。

> [!IMPORTANT]
> 請勿將您的備份資料儲存在應用程式資料所在的相同 Azure Stack Hub 執行個體上。 若 Azure Stack Hub 執行個體完全中斷，也會危害您的備份。

## <a name="availability-considerations"></a>可用性考量

透過 AKS 引擎部署在 Azure Stack Hub 上的 Kubernetes 不是受控服務。 這是使用 Azure 基礎結構即服務 (IaaS) 的自動 Kubernetes 叢集部署和設定。 因此，其提供與基礎結構相同的可用性。

Azure Stack Hub 基礎結構已具備失敗復原能力，並提供可用性設定組之類的功能，將元件分散到多個[容錯和更新網域](/azure-stack/user/azure-stack-vm-considerations#high-availability)。 但在發生硬體失敗時，基礎技術 (容錯移轉叢集) 仍然會造成受影響實體伺服器上的 VM 產生一些停機時間。

將實際執行的 Kubernetes 叢集和工作負載部署至兩個 (或多個) 叢集是很好的作法。 這些叢集應裝載於不同的位置或資料中心，並使用 Azure 流量管理員之類的技術，根據叢集回應時間或根據地理位置來路由使用者。

![使用流量管理員控制流量](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager.png)

具有單一 Kubernetes 叢集的客戶通常會連線至指定應用程式的服務 IP 或 DNS 名稱。 在多個叢集的部署中，客戶應該連線至流量管理員 DNS 名稱，而這個名稱會指向每個 Kubernetes 叢集上的服務/輸入。

![使用流量管理員路由至內部部署叢集](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

> [!NOTE]
> 此模式也是 [Azure中適用於 (受控) AKS 叢集的最佳作法](/azure/aks/operator-best-practices-multi-region#plan-for-multiregion-deployment)。

透過 AKS 引擎部署的 Kubernetes 叢集本身應包含至少三個主要節點和兩個背景工作節點。

## <a name="identity-and-security-considerations"></a>身分識別和安全性考量

身分識別和安全性是很重要的主題。 特別是當解決方案跨越獨立 Azure Stack Hub 執行個體時。 Kubernetes 和 Azure (包含 Azure Stack Hub) 均有不同的角色型存取控制 (RBAC) 機制：

- Azure RBAC 會控制 Azure (以及 Azure Stack Hub) 中的資源存取權，包括建立新 Azure 資源的能力。 權限可以指派給使用者、群組或服務主體。 (服務主體是由應用程式所使用的安全性身分識別)。
- Kubernetes RBAC 會控制 Kubernetes API 的權限。 例如，建立 Pod 和列出 Pod 都是可透過 RBAC 來允許 (或拒絕) 使用者執行的動作。 若要將 Kubernetes 權限指派給使用者，您可以建立「角色」和「角色繫結」。

**Azure Stack Hub 身分識別和 RBAC**

Azure Stack Hub 提供兩個身分識別提供者的選項。 您所使用的提供者取決於環境，以及是否在已連線或已中斷連線的環境中執行：

- Azure AD - 只能在已連線的環境中使用。
- 傳統 Active Directory 樹系的 ADFS - 可以用於已連線或已中斷連線的環境。

識別提供者會管理使用者和群組，包括存取資源的驗證和授權。 您可以將存取權授與 Azure Stack Hub 資源，例如訂用帳戶、資源群組和個別資源 (例如 VM 或負載平衡器)。 若要擁有一致的存取模型，您應該考慮對所有 Azure Stack Hub 使用相同群組 (直接或巢狀)。 以下是設定範例：

![使用 Azure Stack Hub 的巢狀 AAD 群組](media/pattern-highly-available-kubernetes/azure-stack-azure-ad-nested-groups.png)

此範例包含適用於特定用途的專用群組 (使用 AAD 或 ADFS)。 例如，針對在特定 Azure Stack Hub 執行個體上包含 Kubernetes 叢集基礎結構的資源群組提供「參與者」權限 (在此為 "Seattle K8s 叢集參與者")。 這些群組接著會進到包含每個 Azure Stack Hub「子群組」的整體群組巢狀架構中。

我們的範例使用者現在在包含整組 Kubernetes 基礎結構資源的資源群組上都具有「參與者」權限。 因為兩個 Azure Stack Hub 執行個體共用相同的身分識別提供者，所以使用者可以存取這些執行個體上的資源。

> [!IMPORTANT]
> 這些權限只會影響 Azure Stack Hub 和以其為基礎部署的一些資源。 具有此存取層級的使用者可造成不少傷害，但無法存取 Kubernetes IaaS VM 或 Kubernetes API，除非有額外的 Kubernetes 部署存取權。

**Kubernetes 身分識別與 RBAC**

根據預設，Kubernetes 叢集不會使用與基礎 Azure Stack Hub 相同的識別提供者。 裝載 Kubernetes 叢集、主要節點和背景工作角色節點的 VM 會使用部署叢集期間所指定的 SSH 金鑰。 您需要此 SSH 金鑰，才能使用 SSH 連線到這些節點。

Kubernetes API (例如，使用 `kubectl` 存取) 也會受到服務帳戶的保護，包含預設的「叢集管理員」服務帳戶。 此服務帳戶的認證一開始會儲存在 Kubernetes 主要節點上的 `.kube/config` 檔案中。

**祕密管理和應用程式認證**

若要儲存連接字串或資料庫認證之類的秘密，則有數個選擇，包括：

- Azure 金鑰保存庫
- Kubernetes 秘密
- 第三方解決方案，例如 HashiCorp Vault (在 Kubernetes 上執行)

請勿將祕密或認證以純文字方式儲存在設定檔、應用程式代碼或指令碼中。 並且不要將其儲存在版本控制系統中。 相反地，部署自動化應視需要擷取祕密。

## <a name="patch-and-update"></a>修補和更新

Azure Kubernetes Service 中的 **修補和更新 (PNU)** 程序會部分自動化。 Kubernetes 版本升級會以手動方式觸發，而安全性更新會自動套用。 這些更新包括 OS 安全性修正或核心更新。 AKS 不會自動重新啟動這些 Linux 節點來完成更新程序。 

在 Azure Stack Hub 上使用 AKS 引擎部署的 Kubernetes 叢集 PNU 程序並不受管理，而且是叢集操作員的責任。 

AKS 引擎可協助進行兩項最重要的工作：

- [升級至較新的 Kubernetes 和基礎 OS 映像版本](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version)
- [僅升級基礎 OS 映像](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image)

較新的基礎 OS 映像包含最新的 OS 安全性修正和核心更新。 

[自動升級](https://wiki.debian.org/UnattendedUpgrades)機制會自動安裝安全性更新，此更新會在 Azure Stack Hub Marketplace 中有新的基礎 OS 映像版本可用之前發行。 自動升級會預設為啟用，而且會自動安裝安全性更新，但不會重新啟動 Kubernetes 叢集節點。 您可以使用開放原始碼的 [**K** Ubernetes **RE** boot **D** aemon (kured)](/azure/aks/node-updates-kured) 來重新啟動節點。 Kured 會監看需要重新啟動的 Linux 節點，然後自動處理執行中 Pod 和節點重新啟動程序的重新排程。

## <a name="deployment-cicd-considerations"></a>部署 (CI/CD) 考量︰

Azure 和 Azure Stack Hub 會公開相同的 Azure Resource Manager REST API。 這些 API 的定址方式與任何其他 Azure 雲端 (Azure、Azure China 21Vianet、Azure Government) 類似。 雲端之間的 API 版本可能會有差異，Azure Stack Hub 只提供服務的子集。 每個雲端和每個 Azure Stack Hub 執行個體的管理端點 URI 也會不同。

除了所述的細微差異之外，Azure Resource Manager REST API 會提供一致的方式來與 Azure 和 Azure Stack Hub 互動。 在這裡使用的工具組，也可以與任何其他 Azure 雲端搭配使用。 您可以使用 Azure DevOps、Jenkins 之類的工具或 PowerShell，將服務部署至 Azure Stack Hub 並進行協調。

**考量**

在 Azure Stack Hub 部署方面，其中一項主要的差異是網際網路存取性的問題。 網際網路存取性會決定要為您的 CI/CD 作業選取 Microsoft 裝載或自我裝載的組建代理程式。

自我裝載式代理程式可以在 Azure Stack Hub 上執行 (作為 IaaS VM)，或在可存取 Azure Stack Hub 的網路子網路中執行。 若要深入了解差異，請移至 [Azure Pipelines 代理程式](/azure/devops/pipelines/agents/agents)。

下圖可協助您決定是否需要自我裝載或 Microsoft 裝載的組建代理程式：

![自我裝載組建代理程式的是或否](media/pattern-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)

- Azure Stack Hub 管理端點是否可透過網際網路存取？
  - 是：我們可以搭配使用 Azure Pipelines 與 Microsoft 裝載的代理程式來連線到 Azure Stack Hub。
  - 否：我們需要可連線到 Azure Stack Hub 管理端點的自我裝載式代理程式。
- 我們的 Kubernetes 叢集是否可透過網際網路存取？
  - 是：我們可以搭配使用 Azure Pipelines 與 Microsoft 裝載的代理程式來直接與 Kubernetes API 端點互動。
  - 否：我們需要可連線到 Kubernetes 叢集 API 端點的自我裝載式代理程式。

在 Azure Stack Hub 管理端點和 Kubernetes API 可透過網際網路存取的案例中，部署可以使用 Microsoft 裝載的代理程式。 此部署會產生的應用程式架構如下所示：

[![公用架構概觀](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png#lightbox)

如果 Azure Resource Manager 端點、Kubernetes API 或兩者都無法直接透過網際網路存取，我們可以利用自我裝載的組建代理程式來執行管線步驟。 此設計需要的連線能力比較低，而且只能以內部部署網路到 Azure Resource Manager 端點和 Kubernetes API 的連線進行部署：

[![內部部署架構概觀](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png#lightbox)

> [!NOTE]
> **那麼中斷連線的情況呢？** 如果 Azure Stack Hub、Kubernetes 或兩者都沒有向網際網路公開的管理端點，您仍然可以針對您的部署使用 Azure DevOps。 您可以使用自我裝載的代理程式集區 (在內部部署或 Azure Stack Hub 本身執行的 DevOps 代理程式)，或是完全自我裝載 Azure DevOps Server 內部部署環境。 自我裝載的代理程式只需要輸出的 HTTPS (TCP/443) 網際網路連線。

該模式可以在每個 Azure Stack Hub 執行個體上使用 Kubernetes 叢集 (已透過 AKS 引擎部署與協調)。 其中包含由前端、中間層、後端服務 (例如 MongoDB) 和以 nginx 為基礎的輸入控制器所組成的應用程式。 您可以利用「外部資料存放區」，而不是使用裝載於 K8s 叢集上的資料庫。 資料庫選項包括 MySQL、SQL Server 或裝載於 Azure Stack Hub 或 IaaS 外部的任何資料庫類型。 這類設定並不在此範圍內。

## <a name="partner-solutions"></a>合作夥伴解決方案

Microsoft 合作夥伴解決方案可以擴充 Azure Stack Hub 的功能。 這些解決方案在 Kubernetes 叢集上執行的應用程式部署中很有用。  

## <a name="storage-and-data-solutions"></a>儲存體和資料解決方案

如[資料和儲存體考量](#data-and-storage-considerations)中所述，Azure Stack Hub 目前沒有可跨多個執行個體複寫儲存體的原生解決方案。 不同於 Azure，跨多個區域複寫儲存體的功能並不存在。 在 Azure Stack Hub 中，每個執行個體都是各自不同的雲端。 不過，您可以從 Microsoft 合作夥伴取得解決方案，讓您能夠跨 Azure Stack Hub 和 Azure 進行儲存體複寫。 

**SCALITY**

自 2009 起，[Scality](https://www.scality.com/) 即提供具有強大數位商務功能的 Web 規模儲存體。 Scality RING 是我們的軟體定義儲存體，可將商用 x86 伺服器轉換為任何資料類型 (檔案及物件) 的無限制存放集區 (以 PB 為單位)。

**CLOUDIAN**

[Cloudian](https://www.cloudian.com/) 可利用無限制的可擴充儲存體來簡化企業儲存體，讓大量資料可集合併至單一且易於管理的環境。

## <a name="next-steps"></a>後續步驟

深入了解此文章介紹的更多相關概念：

- Azure Stack Hub 中的[跨雲端調整](pattern-cross-cloud-scale.md)及[異地分散式應用程式模式](pattern-geo-distributed.md)。
- [Azure Kubernetes Service (AKS) 上的微服務架構](/azure/architecture/reference-architectures/microservices/aks)。

當您準備好要測試解決方案範例時，請繼續前往[高可用性 Kubernetes 叢集部署指南](/azure/architecture/hybrid/deployments/solution-deployment-guide-highly-available-kubernetes)。 部署指南提供部署及測試其元件的逐步指示。