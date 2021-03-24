---
title: 在 Azure 與 Azure Stack Hub 中部署可跨雲端調整的應用程式
description: 了解如何在 Azure 與 Azure Stack Hub 中部署可跨雲端調整的應用程式。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ed2ad5bed8f4bd80d4a40ab7600842d5544ff97d
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895409"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a>部署應用程式，並使用 Azure 與 Azure Stack Hub 來進行跨雲端規模調整

了解如何建立可提供手動觸發程序的跨雲端解決方案，以透過流量管理員使用自動縮放功能，從 Azure Stack Hub 裝載的 Web 應用程式切換到 Azure 裝載的 Web 應用程式。 此流程可確保雲端公用程式在處理負載時具有彈性且可調整。

透過此模式，您的租用戶可能還無法在公用雲端中執行應用程式。 不過，若讓企業在內部部署環境中維持用來處理應用程式需求高峰的容量，似乎不切實際。 您的租用戶可以透過其內部部署解決方案來使用公用雲端的靈活性。

在本解決方案中，您會建置環境範例，用以：

> [!div class="checklist"]
> - 建立多節點 Web 應用程式。
> - 設定和管理持續部署 (CD) 程序。
> - 將 Web 應用程式發佈至 Azure Stack Hub。
> - 建立發行。
> - 了解如何監視並追蹤您的部署。

> [!Tip]  
> ![混合式支柱圖](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub 是 Azure 的延伸模組。 Azure Stack Hub 可將雲端運算的靈活性和創新能力導入您的內部部署環境中，並啟用獨特的混合式雲端，讓您能夠隨處建置及部署混合式應用程式。  
> 
> [混合式應用程式設計考量](overview-app-design-considerations.md)一文檢閱了設計、部署和操作混合式應用程式時的軟體品質要素 (放置、延展性、可用性、復原、管理性和安全性)。 這些設計考量有助於您設計出最佳的混合式應用程式，減少生產環境可能會遇到的挑戰。

## <a name="prerequisites"></a>Prerequisites

- Azure 訂用帳戶。 如有需要，請在開始之前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。
- Azure Stack Hub 整合式系統，或 Azure Stack 開發套件 (ASDK) 的部署。
  - 如需如何安裝 Azure Stack Hub 的指示，請參閱[安裝 ASDK](/azure-stack/asdk/asdk-install)。
  - 如需 ASDK 部署後自動化指令碼，請移至：[https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)
  - 此安裝可能需要幾個小時才能完成。
- 將 [App Service](/azure-stack/operator/azure-stack-app-service-deploy) PaaS 服務部署至 Azure Stack Hub。
- 在 Azure Stack Hub 環境內[建立方案/供應項目](/azure-stack/operator/service-plan-offer-subscription-overview)。
- 在 Azure Stack Hub 環境內[建立租用戶訂用帳戶](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm)。
- 建立租用戶訂用帳戶中建立 Web 應用程式。 記下新的 Web 應用程式 URL，以供後續使用。
- 在租用戶訂用帳戶中部署 Azure Pipelines 虛擬機器 (VM)。
- 需要具有 .NET 3.5 的 Windows Server 2016 VM。 此 VM 將會建置在 Azure Stack Hub 上的租用戶訂用帳戶中，作為私人組建代理程式。
- 您可以在 Azure Stack Hub Marketplace 中取得[具有 SQL 2017 的 Windows Server 2016 VM 映像](/azure-stack/operator/azure-stack-add-vm-image)。 如果無法取得此映像，可與 Azure Stack Hub 操作員合作，以確實地將此映像新增至環境中。

## <a name="issues-and-considerations"></a>問題和考量

### <a name="scalability"></a>延展性

跨雲端縮放的關鍵要素是能視需求立即在公用和內部部署雲端基礎結構之間提供縮放功能，證明服務能保持一致且可靠。

### <a name="availability"></a>可用性

確定已透過內部部署硬體設定和軟體部署，針對本機部署應用程式的高可用性進行設定。

### <a name="manageability"></a>管理能力

跨雲端解決方案保證能在環境間提供無縫式管理和熟悉介面。 建議您使用 PowerShell 進行跨平台管理。

## <a name="cross-cloud-scaling"></a>跨雲端縮放

### <a name="get-a-custom-domain-and-configure-dns"></a>取得自訂網域並設定 DNS

更新網域的 DNS 區域檔案。 Azure AD 會驗證自訂網域名稱的擁有權。 對於 Azure 中的 Azure/Microsoft 365/外部 DNS 記錄使用 [Azure DNS](/azure/dns/dns-getstarted-portal)，或在[不同的 DNS 註冊機構](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider)新增 DNS 項目。

1. 向公用註冊機構註冊自訂網域。
2. 登入網域的網域名稱註冊機構。 已核准的系統管理員可能需要進行 DNS 更新。
3. 透過新增 Azure AD 提供的 DNS 項目來更新網域的 DNS 區域檔案。 (DNS 項目不會影響電子郵件路由或 Web 裝載行為。)

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a>在 Azure Stack Hub 中建立預設的多節點 Web 應用程式

設定混合式持續整合與持續部署 (CI/CD)，以將 Web 應用程式部署至 Azure 與 Azure Stack Hub，並自動將變更推送至這兩個雲端。

> [!Note]  
> 需要適當映像摘要整合執行的 Azure Stack Hub (Windows Server 和 SQL) 及 App Service 部署。 如需詳細資訊，請參閱 App Service 文件[部署 App Service on Azure Stack Hub 的必要條件](/azure-stack/operator/azure-stack-app-service-before-you-get-started)。

### <a name="add-code-to-azure-repos"></a>將程式碼新增至 Azure Repos

Azure Repos

1. 使用在 Azure Repos 上具有專案建立權限的帳戶登入 Azure Repos。

    混合式 CI/CD 可同時套用至應用程式程式碼和基礎結構程式碼。 使用 [Azure Resource Manager 範本](https://azure.microsoft.com/resources/templates/)進行私用與託管的雲端開發。

    ![連線到 Azure Repos 專案](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. 建立並開啟預設 Web 應用程式以 **複製存放庫**。

    ![在 Azure Web 應用程式中複製存放庫](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a>為這兩個雲端中的應用程式服務建立獨立的 Web 應用程式部署

1. 編輯 **WebApplication.csproj** 檔案。 選取 `Runtimeidentifier` 並新增 `win10-x64`。 (請參閱[獨立式部署](/dotnet/core/deploying/deploy-with-vs#simpleSelf)文件。)

    ![編輯 Web 應用程式專案檔](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. 使用 Team Explorer 將程式碼簽入 Azure Repos 中。

3. 確認應用程式程式碼已簽入 Azure Repos 中。

## <a name="create-the-build-definition"></a>建立組建定義

1. 登入 Azure Pipelines 以確認能夠建立組建定義。

2. 新增 **-r win10-x64** 程式碼。 新增的部分為觸發 .NNE Core 獨立部署所需的程式碼。

    ![將程式碼加入 Web 應用程式](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. 執行組建。 [獨立式部署組建](/dotnet/core/deploying/deploy-with-vs#simpleSelf)程序將會發佈可在 Azure 與 Azure Stack Hub 上執行的成品。

## <a name="use-an-azure-hosted-agent"></a>使用 Azure 託管的代理程式

在 Azure Pipelines 中使用託管的組建代理程式，是建置及部署 Web 應用程式的便利選項。 Microsoft Azure 會自動完成代理程式的維護和升級，以支援持續而不間斷的開發週期。

### <a name="manage-and-configure-the-cd-process"></a>管理和設定 CD 程序

Azure Pipelines 與 Azure DevOps Services 提供具有高度設定和管理能力的管線，可用於對多個環境 (例如開發、暫存、QA 和生產環境) 的發行；包括特定階段需要核准。

## <a name="create-release-definition"></a>建立發行定義

1. 在 Azure DevOps Services 的 [建置及發行]  區段的 [發行]  索引標籤下選取 [加號]  按鈕，以新增發行。

    ![建立發行定義](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. 套用 Azure App Service 部署範本。

   ![套用 Azure App Service 部署範本](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. 在 [新增成品]  下，為 Azure 雲端建置應用程式新增成品。

   ![將成品新增至 Azure 雲端組建](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. 在 [管線] 索引標籤下選取環境的 [階段]、[工作]  連結，並設定 Azure 雲端環境值。

   ![設定 Azure 雲端環境值](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. 設定 [環境名稱]  ，並選取 Azure 雲端端點的 **Azure訂用帳戶**。

      ![選取 Azure 訂用帳戶作為 Azure 雲端端點](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. 在 [應用程式服務名稱]  下設定所需的 Azure 應用程式服務名稱。

      ![設定 Azure 應用程式服務名稱](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. 在 Azure 雲端託管環境的 **代理程式佇列** 下輸入 "Hosted VS2017"。

      ![為 Azure 雲端託管環境設定代理程式佇列](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. 在 [部署 Azure App Service] 功能表中，為環境選取有效的 **套件或資料夾**。 對 **資料夾位置** 選取 [確定]  。
  
      ![選取適用於 Azure App Service 環境的套件或資料夾](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![資料夾選擇器對話方塊1](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. 儲存所有變更，並返回 **發行管線**。

    ![在發行管線中儲存變更](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. 選取 Azure Stack Hub 應用程式的組建，以新增成品。

    ![為 Azure Stack Hub 應用程式新增成品](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. 套用 Azure App Service 部署，以便再新增一個環境。

    ![將環境新增至 Azure App Service 部署](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. 將新環境命名為 "Azure Stack"。

    ![為 Azure App Service 部署中的環境命名](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. 在 [工作]  索引標籤下找出 Azure Stack 環境。

    ![Azure Stack 環境](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. 選取 Azure Stack 端點的訂用帳戶。

    ![選取 Azure Stack 端點的訂用帳戶](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. 將 Azure Stack Web 應用程式名稱設定為 App Service 名稱。
    ![設定 Azure Stack Web 應用程式名稱](media/solution-deployment-guide-cross-cloud-scaling/image20.png)

16. 選取 [Azure Stack 代理程式]。

    ![選取 Azure Stack 代理程式](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. 在 [部署 Azure App Service] 區段下，為環境選取有效的 **套件或資料夾**。 對資料夾位置選取 [確定]  。

    ![為 Azure App Service 部署選取資料夾](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![資料夾選擇器對話方塊2](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. 在 [變數] 索引標籤下新增名為 `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` 的變數，並將其值設定為 **true**，範圍設定為 Azure Stack。

    ![將變數新增至 Azure 應用程式部署](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. 選取兩個成品中的 [持續部署觸發程序]  圖示，並啟用 **持續** 部署觸發程序。

    ![選取持續部署觸發程序](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. 選取 Azure Stack 環境中的 [預先部署]  條件圖示，並將觸發程序設定為 [發行之後]  。

    ![選取部署前的條件](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. 儲存所有變更。

> [!Note]  
> 工作的某些設定可能已在從範本建立發行定義時自動定義為[環境變數](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables)。 這些設定無法在工作設定中修改；而是必須選取父環境項目才能編輯這些設定。

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a>透過 Visual Studio 發佈至 Azure Stack Hub

藉由建立端點，Azure DevOps Services 組建可以將 Azure 服務應用程式部署到 Azure Stack Hub。 Azure Pipelines 會連線至組建代理程式，後者再連線至 Azure Stack Hub。

1. 登入 Azure DevOps Services，並移至 [應用程式設定] 頁面。

2. 在 [設定]  上，選取 [安全性]  。

3. 在 [VSTS 群組]  中，選取 [端點建立者]  。

4. 在 [成員]  索引標籤上，選取 [新增]  。

5. 在 [新增使用者和群組]  中，輸入使用者名稱，然後從使用者清單中選取該使用者。

6. 選取 [儲存變更]  。

7. 在 [VSTS 群組]  清單中，選取 [端點管理員]  。

8. 在 [成員]  索引標籤上，選取 [新增]  。

9. 在 [新增使用者和群組]  中，輸入使用者名稱，然後從使用者清單中選取該使用者。

10. 選取 [儲存變更]  。

既然端點資訊已存在，所以 Azure Pipelines 對 Azure Stack Hub 的連線已可供使用。 Azure Stack Hub 中的組建代理程式會取得來自 Azure Pipelines 的指示，然後代理程式會傳達與 Azure Stack Hub 進行通訊所需的端點資訊。

## <a name="develop-the-app-build"></a>開發應用程式組建

> [!Note]  
> 需要適當映像摘要整合執行的 Azure Stack Hub (Windows Server 和 SQL) 及 App Service 部署。 如需詳細資訊，請參閱[部署 Azure Stack Hub 上的 App Service 必要條件](/azure-stack/operator/azure-stack-app-service-before-you-get-started)。

請使用 [Azure Resource Manager 範本](https://azure.microsoft.com/resources/templates/) (例如來自 Azure Repos 的 Web 應用程式程式碼) 以部署至這兩個雲端。

### <a name="add-code-to-an-azure-repos-project"></a>將程式碼新增至 Azure Repos 專案

1. 使用在 Azure Stack Hub 上具有專案建立權限的帳戶登入 Azure Repos。

2. 建立並開啟預設 Web 應用程式以 **複製存放庫**。

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a>為這兩個雲端中的應用程式服務建立獨立的 Web 應用程式部署

1. 編輯 **WebApplication.csproj** 檔案：選取 `Runtimeidentifier`，然後新增 `win10-x64`。 如需詳細資訊，請參閱[獨立式部署](/dotnet/core/deploying/deploy-with-vs#simpleSelf)文件。

2. 使用 Team Explorer 將程式碼簽入 Azure Repos 中。

3. 確認應用程式程式碼已簽入 Azure Repos 中。

### <a name="create-the-build-definition"></a>建立組建定義

1. 使用可建立組建定義的帳戶登入 Azure Pipelines。

2. 移至專案的 [建置 Web 應用程式]  頁面。

3. 在 [引數]  中，新增 **-r win10-x64** 程式碼。 想要透過 .NET Core 觸發獨立式部署就必須新增此項目。

4. 執行組建。 [獨立式部署組建](/dotnet/core/deploying/deploy-with-vs#simpleSelf)程序將會發佈可在 Azure 與 Azure Stack Hub 上執行的成品。

#### <a name="use-an-azure-hosted-build-agent"></a>使用 Azure 託管的組建代理程式

在 Azure Pipelines 中使用託管的組建代理程式，是建置及部署 Web 應用程式的便利選項。 Microsoft Azure 會自動完成代理程式的維護和升級，以支援持續而不間斷的開發週期。

### <a name="configure-the-continuous-deployment-cd-process"></a>設定持續部署 (CD) 程序

Azure Pipelines 與 Azure DevOps Services 提供具有高度設定和管理能力的管線，可用於對多個環境 (例如開發、暫存、品質保證 (QA) 和生產環境) 的發行。 此程序中可以包括在應用程式生命週期的特定階段要求核准。

#### <a name="create-release-definition"></a>建立發行定義

建立發行定義是應用程式建置程序的最後一個步驟。 這個發行定義可用來建立發行並部署組建。

1. 登入 Azure Pipelines，並移至專案的 [建置及發行]  。

2. 在 [發行]  索引標籤上，選取 [ + ]  ，然後挑選 [建立發行定義]  。

3. 在 [選取範本]  中，選擇 [Azure App Service 部署]  ，然後選取 [套用]  。

4. 在 [新增成品]  上，從 [來源 /(組建定義/)]  中選取 [Azure 雲端] 組建應用程式。

5. 在 [管線]  索引標籤上，選取 [檢視環境工作]  的 [1 階段 1 個工作]   連結。

6. 在 [工作]  索引標籤上，輸入「Azure」作為 [環境名稱]  ，然後從 [Azure 訂用帳戶]  清單選取 [AzureCloud Traders-Web EP]。

7. 輸入 **Azure App Service 名稱**，也就是下一個螢幕擷取畫面中的 `northwindtraders`。

8. 在 [代理程式階段] 中，從 [代理程式佇列]  清單中選取 [Hosted VS2017]  。

9. 在 [部署 Azure App Service]  中，為環境選取有效的 **套件或資料夾**。

10. 在 [選取檔案或資料夾]  中，對 [位置]  選取 [確定]  。

11. 儲存所有變更，並返回 [管線]  。

12. 在 [管線]  索引標籤上，選取 [新增成品]  ，然後從 [來源 (組建定義)]  清單中選擇 [NorthwindCloud Traders-Vessel]  。

13. 在 [選取範本]  上，新增另一個環境。 挑選 [Azure App Service 部署]  ，然後選取 [套用]  。

14. 輸入 `Azure Stack Hub` 作為 [環境名稱]  。

15. 在 [工作]  索引標籤上，尋找並選取 Azure Stack Hub。

16. 從 [Azure 訂用帳戶]  清單中，選取 [AzureStack Traders-Vessel EP]  作為 Azure Stack Hub 端點。

17. 輸入 Azure Stack Hub Web 應用程式名作為 **App Service 名稱**。

18. 在 [代理程式選擇]  底下，從 [代理程式佇列]  清單中挑選 **AzureStack -b Douglas Fir**。

19. 在 [部署 Azure App Service]  中，為環境選取有效的 **套件或資料夾**。 在 [選取檔案或資料夾]  中，對 [位置]  資料夾選取 [確定]  。

20. 在 [變數]  索引標籤上，尋找名為 `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` 的變數。 將變數值設定為 **true**，並將其範圍設定為 [Azure Stack Hub]  。

21. 在 [管線]  索引標籤上，選取 NorthwindCloud Traders-Web 成品的 [持續部署觸發程序]  圖示，並將 [持續部署觸發程序]  設定為 [啟用]  。 針對 **NorthwindCloud Traders-Vessel** 成品執行相同的動作。

22. 針對 Azure Stack Hub 環境，選取 [部署前的條件]  圖示，將觸發程序設定為 [發行之後]  。

23. 儲存所有變更。

> [!Note]  
> 發行工作的某些設定已在從範本建立發行定義時自動定義為[環境變數](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables)。 這些設定無法在工作設定中修改，但可在父環境項目中修改。

## <a name="create-a-release"></a>建立發行

1. 在 [管線]  索引標籤上，開啟 [發行]  清單，然後選取 [建立發行]  。

2. 輸入發行的說明，確認已選取正確的成品，然後選取 [建立]  。 幾分鐘之後將會出現一個橫幅，指出新的發行已建立，且發行名稱會顯示為連結。 選取連結以查看 [發行摘要] 頁面。

3. [發行摘要] 頁面會顯示有關發行的詳細資料。 在下列 "Release-2" 螢幕擷取畫面中，[環境]  區段顯示 Azure 的 [部署狀態]  為 [進行中]，Azure Stack Hub 的狀態為 [成功]。 當 Azure 環境的部署狀態變更為 [成功] 時，便會出現橫幅指出發行已可供核准。 對部署若擱置或失敗，將會顯示藍色的 **(i)** 資訊圖示。 將滑鼠暫留在圖示上，即可查看快顯，其中會包含延遲或失敗的原因。

4. 其他檢視 (例如發行清單) 也會顯示指出核准擱置中的圖示。 這個圖示的快顯會顯示環境名稱以及更多與部署相關的詳細資料。 管理員可輕鬆查看發行的整體進度，以及查看哪些版本正在等待核准。

## <a name="monitor-and-track-deployments"></a>監視和追蹤部署

1. 在 [Release-2]  摘要頁面上，選取 [記錄]  。 在部署期間，此頁面會顯示來自代理程式的即時記錄。 左窗格會顯示每個環境部署中每個作業的狀態。

2. 選取部署前或部署後核准 [動作]  資料行中的人形圖示，以查看哪些人已核准 (或拒絕) 部署，以及該人員提供的訊息。

3. 部署完成後，整個記錄檔會顯示在右窗格中。 在左窗格中選取任何 [步驟]  ，以查看單一步驟的記錄檔，例如「初始化作業」  。 能夠查看個別記錄，可讓您輕鬆地針對整體部署的各個部分進行追蹤和偵錯。 [儲存]  步驟的記錄，或是 [將所有記錄下載為 zip]  。

4. 開啟 [摘要]  索引標籤可查看發行的一般資訊。 此檢視會顯示組建的詳細資料、組件所部署到的環境、部署狀態，以及其他關於發行的資訊。

5. 選取環境連結 (**Azure** 或 **Azure Stack Hub**) 以查看特定環境現有與擱置中部署的資訊。 使用這些檢視可快速確認同一個組建已部署至兩個環境。

6. 在瀏覽器中開啟 **已部署的生產應用程式**。 例如，針對 Azure App Service 網站，請開啟 URL `https://[your-app-name\].azurewebsites.net`。

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a>Azure 與 Azure Stack Hub 的整合提供可調整的跨雲端解決方案

具彈性且完善的多雲端服務可提供資料安全性、備份和備援、一致且快速的可用性、可縮放的儲存體和散發，以及異地相容的路由。 此手動觸發程序可確實地在託管的 Web 應用程式之間提供可靠且有效率的負載切換，確保重要資料的立即可用性。

## <a name="next-steps"></a>後續步驟

- 若要深入了解 Azure 雲端模式，請參閱[雲端設計模式](/azure/architecture/patterns)。
