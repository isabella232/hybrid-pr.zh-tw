---
title: 在 Azure 和 Azure Stack Hub 中部署 AI 型客流量偵測解決方案
description: 了解如何使用 Azure 和 Azure Stack Hub 部署以 AI 型客流量偵測解決方案，以分析零售商店中的訪客流量。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 2177b32474dea695967e197acbd4bc1e18422d7b
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/09/2020
ms.locfileid: "96901485"
---
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a>使用 Azure 和 Azure Stack Hub 來部署 AI 型客流量偵測解決方案

本文說明如何使用 Azure、Azure Stack Hub 和自訂視覺 AI 開發套件部署 AI 型解決方案，從真實世界的動作產生深入解析。

在本解決方案中，您將了解如何：

> [!div class="checklist"]
> - 在邊緣部署雲端原生應用程式套件組合 (CNAB)。 
> - 部署跨越雲端界限的應用程式。
> - 在邊緣使用自訂視覺 AI 開發套件進行推斷。

> [!Tip]  
> ![混合式支柱圖](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub 是 Azure 的延伸模組。 Azure Stack Hub 可將雲端運算的靈活性與創新能力導入您的內部部署環境中，並啟用獨特的混合式雲端，讓您能夠隨處建置及部署混合式應用程式。  
> 
> [混合式應用程式設計考量](overview-app-design-considerations.md)一文檢閱了設計、部署和操作混合式應用程式時的軟體品質要素 (放置、延展性、可用性、復原、管理性和安全性)。 這些設計考量有助於您設計出最佳的混合式應用程式，減少生產環境可能會遇到的挑戰。

## <a name="prerequisites"></a>Prerequisites

開始使用此部署指南之前，請務必先：

- 檢閱[客流量偵測解決方案](pattern-retail-footfall-detection.md)主題。
- 透過以下方式，取得對 Azure Stack 開發套件 (ASDK) 或 Azure Stack Hub 整合式系統執行個體的使用者存取權：
  - 安裝 [Azure App Service on Azure Stack Hub 資源提供者](/azure-stack/operator/azure-stack-app-service-overview.md)。 您需要 Azure Stack Hub 執行個體的操作員存取權，或與您的系統管理員合作以進行安裝。
  - 一個可提供 App Service 和儲存體配額的訂用帳戶。 您需要操作員存取權才能建立供應專案。
- 取得 Azure 訂用帳戶的存取權。
  - 如果您沒有 Azure 訂用帳戶，請在開始前先註冊一個[免費試用帳戶](https://azure.microsoft.com/free/)。
- 在您的目錄中建立兩個服務主體：
  - 一個設定為與 Azure 資源搭配使用，並具備 Azure 訂用帳戶範圍的存取權。
  - 一個設定為與 Azure Stack Hub 資源搭配使用，並具備 Azure Stack Hub 訂用帳戶範圍的存取權。
  - 若要深入了解建立服務主體及授權存取的相關資訊，請參閱[使用應用程式身分識別來存取資源](/azure-stack/operator/azure-stack-create-service-principals.md)。 如果您偏好使用 Azure CLI，請參閱[使用 Azure CLI 建立 Azure 服務主體](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest&preserve-view=true)。
- 在 Azure 或 Azure Stack Hub 中部署 Azure 認知服務。
  - 首先，請[深入了解認知服務](https://azure.microsoft.com/services/cognitive-services/)。
  - 接著請造訪[將 Azure 認知服務部署至 Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services.md)，以在 Azure Stack Hub 上部署認知服務。 您必須先註冊才能存取預覽版。
- 複製或下載未設定的 Azure 自訂視覺 AI 開發套件。 如需詳細資訊，請參閱[視覺 AI 開發套件](https://azure.github.io/Vision-AI-DevKit-Pages/) \(英文\)。
- 註冊 Power BI 帳戶。
- Azure 認知服務臉部 API 訂用帳戶金鑰與端點 URL。 您可以透過[試用認知服務](https://azure.microsoft.com/try/cognitive-services/?api=face-api)免費試用取得這兩個項目。 或者，依照[建立認知服務帳戶](/azure/cognitive-services/cognitive-services-apis-create-account)中的指示取得免費試用。
- 安裝下列開發資源：
  - [Azure CLI 2.0](/azure-stack/user/azure-stack-version-profiles-azurecli2.md)
  - [Docker CE](https://hub.docker.com/search/?type=edition&offering=community)
  - [Porter](https://porter.sh/)。 您可以使用 Porter 來部署雲端應用程式，並使用為您提供的 CNAB 組合資訊清單。
  - [Visual Studio Code](https://code.visualstudio.com/)
  - [適用於 Visual Studio Code 的 Azure IoT 工具](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [適用於 Visual Studio Code 的 Python 擴充功能](https://marketplace.visualstudio.com/items?itemName=ms-python.python) \(英文\)
  - [Python](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a>部署混合式雲端應用程式

首先，您可以使用 Porter CLI 來產生認證集，然後部署雲端應用程式。  

1. 從 https://github.com/azure-samples/azure-intelligent-edge-patterns 複製或下載解決方案範例程式碼。 

1. Porter 將會產生一組將應用程式部署自動化的認證。 執行認證產生命令之前，請先確定有下列項目可用：

    - 用來存取 Azure 資源的服務主體，包括服務主體識別碼、金鑰和租用戶 DNS。
    - Azure 訂用帳戶的訂用帳戶識別碼。
    - 用來存取 Azure Stack Hub 資源的服務主體，包括服務主體識別碼、金鑰和租用戶 DNS。
    - Azure Stack Hub 訂用帳戶的訂用帳戶識別碼。
    - 您的 Azure 認知服務臉部 API 金鑰與資源端點 URL。

1. 執行 Porter 認證產生程序並遵循提示進行：

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. Porter 也需要一組參數來執行。 建立參數文字檔，並輸入下列名稱/值組。 如果您需要任何和必要值有關的協助，請洽詢您的 Azure Stack Hub 系統管理員。

   > [!NOTE] 
   > `resource suffix` 值是用來確保部署的資源具有在整個 Azure 上唯一的名稱。 它必須是字母和數字的唯一字串組合，且不能超過 8 個字元。

    ```porter
    azure_stack_tenant_arm="Your Azure Stack Hub tenant endpoint"
    azure_stack_storage_suffix="Your Azure Stack Hub storage suffix"
    azure_stack_keyvault_suffix="Your Azure Stack Hub keyVault suffix"
    resource_suffix="A unique string to identify your deployment"
    azure_location="A valid Azure region"
    azure_stack_location="Your Azure Stack Hub location identifier"
    powerbi_display_name="Your first and last name"
    powerbi_principal_name="Your Power BI account email address"
    ```
   儲存文字檔並記下其路徑。

1. 您現在已經準備好使用 Porter 部署混合式雲端應用程式。 執行 install 命令並監看，因為資源已部署至 Azure 和 Azure Stack Hub：

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. 完成部署之後，請記下下列值：
    - 觀景窗的連接字串。
    - 映像儲存體帳戶連接字串。
    - 資源群組名稱。

## <a name="prepare-the-custom-vision-ai-devkit"></a>準備自訂視覺 AI 開發套件

接下來，設定自訂視覺 AI 開發套件，如[視覺 AI DevKit 快速入門](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/)中所示。 您也可以使用在上一個步驟中提供的連接字串，來設定及測試您的觀景窗。

## <a name="deploy-the-camera-app"></a>部署觀景窗應用程式

請使用 Porter CLI 來產生認證集，然後部署觀景窗應用程式。

1. Porter 將會產生一組將應用程式部署自動化的認證。 執行認證產生命令之前，請先確定有下列項目可用：

    - 用來存取 Azure 資源的服務主體，包括服務主體識別碼、金鑰和租用戶 DNS。
    - Azure 訂用帳戶的訂用帳戶識別碼。
    - 您在部署雲端應用程式時提供的映像儲存體帳戶連接字串。

1. 執行 Porter 認證產生程序並遵循提示進行：

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. Porter 也需要一組參數來執行。 建立參數文字檔並輸入以下文字。 如果您不知道部分必要值，請洽詢您的 Azure Stack Hub 系統管理員。

    > [!NOTE]
    > `deployment suffix` 值是用來確保部署的資源具有在整個 Azure 上唯一的名稱。 它必須是字母和數字的唯一字串組合，且不能超過 8 個字元。

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    儲存文字檔並記下其路徑。

4. 您現在已經準備好使用 Porter 部署觀景窗應用程式。 執行安裝命令，並在建立 IoT Edge 部署時監看。

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. 藉由在 `https://<camera-ip>:3000/` 檢視觀景窗摘要，以確認相機的部署已完成，其中 `<camara-ip>` 是觀景窗 IP 位址。 此步驟最多可能需要 10 分鐘。

## <a name="configure-azure-stream-analytics"></a>設定 Azure 串流分析

現在資料已從觀景窗進入 Azure 串流分析，我們需要手動授權以與 Power BI 通訊。

1. 從 Azure 入口網站開啟 [所有資源]  ，然後尋找 *process-footfall\[yoursuffix\]* 作業。

2. 在串流分析作業窗格的 [作業拓撲]  區段中，選取 [輸出]  選項。

3. 選取 **traffic-output** 輸出接收。

4. 選取 [更新授權]  ，然後登入您的 Power BI 帳戶。
  
    ![Power BI 中的更新授權提示](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. 儲存輸出設定。

6. 前往 [概觀]  窗格，然後選取 [啟動]  即可開始傳送資料至 Power BI。

7. 針對作業輸出開始時間選取 [現在]  並選取 [啟動]  。 您可以在通知列中檢視作業狀態。

## <a name="create-a-power-bi-dashboard"></a>建立 Power BI 儀表板

1. 作業成功之後，請移至 [Power BI](https://powerbi.com/)，然後使用公司或學校帳戶登入。 如果串流分析作業查詢會輸出結果，您所建立的 *footfall-dataset* 資料集會存在於 [資料集]  索引標籤下。

2. 從 Power BI 工作區中選取 [+ 建立]  ，建立名為「客流量分析」  的新儀表板。

3. 在視窗頂端，選取 [新增圖格]  。 接著選取 [自訂串流資料]  ，然後選取 [下一步]  。 在 [您的資料集]  下方選擇 **footfall-dataset**。 從 [視覺效果類型]  下拉式清單中選取 [卡]  ，並將 [年齡]  新增至 [欄位]  。 選取 [下一步]  以輸入圖格的名稱，然後選取 [套用]  以建立圖格。

4. 您可以視需要新增其他欄位和卡片。

## <a name="test-your-solution"></a>測試解決方案

觀察您在 Power BI 中所建立卡片包含的資料如何隨著不同的人在觀景窗前面走動而改變。 在記錄之後，可能需要最多 20 秒的時間才會出現推斷。

## <a name="remove-your-solution"></a>移除您的解決方案

如果想要移除解決方案，請使用您為部署所建立的相同參數檔案，並使用 Porter 執行下列命令：

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a>後續步驟

- 深入了解[混合式應用程式設計考量](overview-app-design-considerations.md)
- 檢閱 [GitHub 上此範例的程式碼](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis)並提出改進建議。
