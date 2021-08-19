---
title: Azure Stack Hub 中的跨雲端調整 (內部部署資料) 模式
description: 瞭解如何建立可調整的跨雲端應用程式，以使用 Azure 中的內部內部部署資料和 Azure Stack Hub。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 5c8e3adb621ae4322bf6d60792fc307dbb24ff90
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281239"
---
# <a name="cross-cloud-scaling-on-premises-data-pattern"></a>跨雲端調整 (內部部署資料) 模式

瞭解如何建立橫跨 Azure 和 Azure Stack Hub 的混合式應用程式。 此模式也示範如何使用單一內部部署資料來源以遵守合規性。

## <a name="context-and-problem"></a>內容和問題

許多組織會收集及儲存大量的機密客戶資料。 由於公司法規或政府政策，通常會防止他們將機密資料儲存在公用雲端中。 這些組織也想要妥善利用公用雲端的延展性。 公用雲端可以處理季節性的流量尖峰，讓客戶在需要的時候將費用花在其真正需要的硬體上。

## <a name="solution"></a>解決方法

此解決方案善用私人雲端的合規性優勢，並將這些優勢與公用雲端的延展性互相結合。 Azure 和 Azure Stack Hub 混合式雲端為開發人員提供一致的體驗。 這項一致性可讓他們將其技能同時套用至公用雲端和內部部署環境。

解決方案部署指南可讓您將相同的 web 應用程式部署至公用和私人雲端。 您也可以存取裝載于私人雲端上的非網際網路可路由網路。 系統會監視 web 應用程式的負載。 當流量大幅增加時，程序會處理 DNS 記錄以將流量重新導向至公用雲端。 當流量不再大幅增加時，將更新 DNS 記錄，以將流量導向回到私人雲端。

[![使用內部內部部署資料模式進行跨雲端調整](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)

## <a name="components"></a>元件

此解決方案使用下列元件：

| 階層 | 元件 | 描述 |
|----------|-----------|-------------|
| Azure | Azure App Service | [Azure App Service](/azure/app-service/) 可讓您組建和裝載 Web 應用程式、RESTful API 應用程式，以及 Azure Functions。 全部皆以您自己選擇的程式設計語言編寫，而不必管理基礎結構。 |
| | Azure 虛擬網路| [Azure 虛擬網路 (VNet)](/azure/virtual-network/virtual-networks-overview) 是私人網路在 Azure 中的基本建置組塊。 VNet 可讓多個 Azure 資源類型（例如虛擬機器 (VM) ）安全地彼此通訊，以及與網際網路和內部部署網路通訊。 該解決方案也示範其他網路元件的使用方法：<br>-應用程式和閘道子網。<br>-本機內部部署網路閘道。<br>-虛擬網路閘道，可作為站對站 VPN 閘道連線。<br>-公用 IP 位址。<br>-點對站 VPN 連線。<br>-用於裝載 DNS 網域和提供名稱解析的 Azure DNS。 |
| | Azure 流量管理員 | [Azure 流量管理員](/azure/traffic-manager/traffic-manager-overview)是以 DNS 為基礎的流量負載平衡器。 其可讓您控制使用者流量，將流量分散到不同資料中心的服務端點。 |
| | Azure Application Insights | [Application Insights](/azure/azure-monitor/app/app-insights-overview)是可延伸的應用程式效能管理服務，可供 網頁程式開發人員在多個平臺上建立和管理應用程式。|
| | Azure Functions | [Azure Functions](/azure/azure-functions/) 可讓您在無伺服器環境中執行程式碼，而不需要先建立 VM 或發佈 web 應用程式。 |
| | Azure 自動調整 | [自動](/azure/azure-monitor/platform/autoscale-overview) 調整是雲端服務、vm 和 web 應用程式的內建功能。 這項功能可讓應用程式在需求變更時執行其最佳效果。 應用程式會針對流量尖峰進行調整，視需要在計量變更和調整時通知您。 |
| Azure Stack Hub | IaaS 計算 | Azure Stack Hub 可讓您使用 Azure 所啟用的相同應用程式模型、自助入口網站和 Api。 Azure Stack Hub IaaS 可提供各種開放原始碼技術，以進行一致的混合式雲端部署。 例如，解決方案範例使用 Windows Server VM 到 SQL Server。|
| | Azure App Service | 就像 Azure Web 應用程式一樣，解決方案會使用 [Azure Stack Hub 上的 Azure App Service](/azure-stack/operator/azure-stack-app-service-overview) 來裝載 Web 應用程式。 |
| | 網路 | Azure Stack Hub 的虛擬網路與 Azure 虛擬網路的運作方式完全相同。 它使用許多相同的網路元件，包括自訂的主機名稱。
| Azure DevOps Services | 註冊 | 快速設定用於建置、測試和部署的持續整合。 如需詳細資訊，請參閱[註冊，登入 Azure DevOps](/azure/devops/user-guide/sign-up-invite-teammates?view=azure-devops)。 |
| | Azure Pipelines | 使用 [Azure Pipelines](/azure/devops/pipelines/agents/agents?view=azure-devops) 以進行持續整合/持續傳遞。 Azure Pipelines 可讓您管理裝載的組建和版本代理程式及定義。 |
| | 程式碼存放庫 | 利用多個程式碼存放庫以簡化您的開發管線。 使用 GitHub、Bitbucket、Dropbox、OneDrive 和 Azure Repos 中現有的程式碼存放庫。 |

## <a name="issues-and-considerations"></a>問題和考量

決定如何實作此解決方案時，請考慮下列幾點：

### <a name="scalability"></a>延展性

Azure 和 Azure Stack Hub 的獨特之處在于支援現今全球分散業務的需求。

#### <a name="hybrid-cloud-without-the-hassle"></a>混合式雲端，一點也不麻煩

Microsoft 運用單一的解決方案，為內部部署資產與 Azure Stack Hub 和 Azure 提供絕佳的整合。 這項整合消弭了管理多點解決方案和混合雲端提供者的麻煩。 透過跨雲端調整，Azure 的強大功能只需按幾下即可。 只需將您的 Azure Stack Hub 連線到 Azure，並使用雲端負載平衡，您的資料和應用程式就會在需要時于 Azure 中提供。

- 消除了建立和維護次要災害復原網站的需求。
- 藉由消除磁帶備份，並在 Azure 中存放最多99年的備份資料，節省時間和金錢。
- 輕鬆地將執行中的 Hyper-V、實體 (預覽)，以及 VMware (預覽) 工作負載移轉至 Azure，以妥善運用雲端的經濟效益和彈性。
- 針對 Azure 中的內部部署資產複寫複本執行大量計算報告或分析，而不會影響生產工作負載。
- 高載至雲端，並在 Azure 中執行內部部署工作負載，並視需要使用較大的計算範本。 混合式的解決方案可在需要時為您提供所需的功能。
- 只要按幾下，就能在 Azure 中建立多層式開發環境-甚至會將即時生產資料複寫到開發/測試環境，以保持近乎即時的同步。

#### <a name="economy-of-cross-cloud-scaling-with-azure-stack-hub"></a>透過 Azure Stack Hub 進行跨雲端調整的經濟效益

雲端負載平衡的主要優點是經濟實惠。 只有在需要這些額外資源時，才需要支付費用。 不再需要花費額外的容量，或嘗試預測需求尖峰和波動。

#### <a name="reduce-high-demand-loads-into-the-cloud"></a>減少載入雲端的高度需求

跨雲端調整可以用來處理處理負擔。 負載是藉由將基本應用程式移至公用雲端，釋出本機資源給商務關鍵應用程式來散發。 應用程式可以套用至私人雲端，然後只在需要符合需求時才高載至公用雲端。

### <a name="availability"></a>可用性

全域部署有自己的挑戰，例如變數連線能力和不同的政府法規（依區域）。 開發人員只能開發一個應用程式，然後根據不同的原因和需求，部署該應用程式。 將您的應用程式部署至 Azure 公用雲端，然後將其他實例或元件部署在本機。 您可以使用 Azure 管理所有執行個體之間的流量。

### <a name="manageability"></a>管理能力

#### <a name="a-single-consistent-development-approach"></a>單一、一致的開發方法

Azure 和 Azure Stack Hub 可讓您在整個組織中使用一組一致的開發工具。 這種一致性可讓您更輕鬆地執行持續整合和持續開發 (CI/CD)。 許多部署在 Azure 或 Azure Stack Hub 中的應用程式和服務都是可互換的，且可以在任一位置順暢地執行。

混合式 CI/CD 管線有助於：

- 根據程式碼存放庫認可的程式碼起始新的組建。
- 自動將您新建置的程式碼部署至 Azure 以進行使用者驗收測試。
- 一旦您的程式碼已通過測試，便會自動部署至 Azure Stack Hub。

### <a name="a-single-consistent-identity-management-solution"></a>單一、一致的身分識別管理解決方案

Azure Stack Hub 適用于 Azure Active Directory (Azure AD) 和 Active Directory 同盟服務 (ADFS) 。 Azure Stack Hub 適用于連接案例中的 Azure AD。 針對沒有連線能力的環境，您可以使用 ADFS 做為中斷連線時的解決方案。 服務主體會用來授與應用程式的存取權，讓他們可以透過 Azure Resource Manager 部署或設定資源。

### <a name="security"></a>安全性

#### <a name="ensure-compliance-and-data-sovereignty"></a>確保合規性和資料主權

Azure Stack Hub 可讓您在多個國家/地區執行相同的服務，就像使用公用雲端一樣。 在每個國家/地區的資料中心部署相同的應用程式，可符合資料主權需求。 這項功能可確保個人資料會保留在每個國家/地區的邊界內。

#### <a name="azure-stack-hub---security-posture"></a>Azure Stack Hub - 安全性狀態

安全性狀態來自於穩定、持續的服務處理程序。 因此，Microsoft 投資一個協調流程引擎，可在整個基礎結構中順暢地套用修補程式和更新。

由於與 Azure Stack Hub OEM 合作夥伴合作，Microsoft 會將相同的安全性狀態延伸至 OEM 專屬的元件，例如硬體生命週期主機和在其上執行的軟體。 這種合作關係可確保 Azure Stack Hub 在整個基礎結構上具有一致且穩固的安全性狀態。 接著，客戶可以建立和保護其應用程式工作負載。

#### <a name="use-of-service-principals-via-powershell-cli-and-azure-portal"></a>透過 PowerShell、CLI 和 Azure 入口網站來使用服務主體

若要為資源提供指令碼或應用程式的存取權，請設定應用程式的身分識別，並使用應用程式本身的認證進行驗證。 此身分識別稱為「服務主體」，可讓您：

- 將許可權指派給與您自己的許可權不同的應用程式身分識別，並且限制為應用程式的需求。
- 使用憑證在執行無人看管的指令碼時進行驗證。

如需有關服務主體建立和使用認證憑證的詳細資訊，請參閱 [使用應用程式識別來存取資源](/azure-stack/operator/azure-stack-create-service-principals)。

## <a name="when-to-use-this-pattern"></a>使用此模式的時機

- 我的組織使用 DevOps 方法，或具有一個打算在不久的將來使用的方法。
- 我想要跨 Azure Stack Hub 實作和公用雲端來實作 CI/CD 做法。
- 我想要跨雲端和內部部署環境來合併 CI/CD 管線。
- 我想要能夠使用雲端或內部部署服務順暢地開發應用程式。
- 我想要跨雲端和內部部署應用程式來運用一致的開發人員技能。
- 我使用的是 Azure，但我的開發人員是在內部部署 Azure Stack Hub 雲端中工作。
- 我的內部部署應用程式在季節性、迴圈或無法預測的波動期間遇到需求尖峰。
- 我有內部部署元件，而我想要使用雲端以順暢地進行調整。
- 我需要雲端擴充性，但我希望應用程式盡可能在內部部署執行。

## <a name="next-steps"></a>後續步驟

深入了解此文章介紹的更多相關主題：

- 觀看[資料中心和公用雲端之間的動態調整應用程式](https://www.youtube.com/watch?v=2lw8zOpJTn0)，以了解如何使用此模式。
- 若要深入瞭解最佳作法，並回答您可能遇到的其他問題，請參閱 [混合式應用程式設計考慮](overview-app-design-considerations.md) 。
- 此模式會使用 Azure Stack 系列產品，包括 Azure Stack Hub。 若要深入了解產品與解決方案的完整組合，請參閱 [Azure Stack 系列產品和解決方案](/azure-stack)。

當您準備好測試解決方案範例時，請繼續進行 [跨雲端調整 (內部部署資料) 解決方案部署指南](/azure/architecture/hybrid/deployments/solution-deployment-guide-cross-cloud-scaling-onprem-data)。 部署指南提供部署及測試其元件的逐步指示。