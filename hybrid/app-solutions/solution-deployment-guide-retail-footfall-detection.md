---
title: 在 Azure 和 Azure Stack Hub 中部署 AI 型客流量偵測解決方案
description: 了解如何使用 Azure 和 Azure Stack Hub 部署以 AI 型客流量偵測解決方案，以分析零售商店中的訪客流量。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: caedbd4758b9ae8c93cf9bb625ed9aac68bfa196
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895355"
---
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a><span data-ttu-id="84266-103">使用 Azure 和 Azure Stack Hub 來部署 AI 型客流量偵測解決方案</span><span class="sxs-lookup"><span data-stu-id="84266-103">Deploy an AI-based footfall detection solution using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="84266-104">本文說明如何使用 Azure、Azure Stack Hub 和自訂視覺 AI 開發套件部署 AI 型解決方案，從真實世界的動作產生深入解析。</span><span class="sxs-lookup"><span data-stu-id="84266-104">This article describes how to deploy an AI-based solution that generates insights from real world actions by using Azure, Azure Stack Hub, and the Custom Vision AI Dev Kit.</span></span>

<span data-ttu-id="84266-105">在本解決方案中，您將了解如何：</span><span class="sxs-lookup"><span data-stu-id="84266-105">In this solution, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="84266-106">在邊緣部署雲端原生應用程式套件組合 (CNAB)。</span><span class="sxs-lookup"><span data-stu-id="84266-106">Deploy Cloud Native Application Bundles (CNAB) at the edge.</span></span> 
> - <span data-ttu-id="84266-107">部署跨越雲端界限的應用程式。</span><span class="sxs-lookup"><span data-stu-id="84266-107">Deploy an app that spans cloud boundaries.</span></span>
> - <span data-ttu-id="84266-108">在邊緣使用自訂視覺 AI 開發套件進行推斷。</span><span class="sxs-lookup"><span data-stu-id="84266-108">Use the Custom Vision AI Dev Kit for inference at the edge.</span></span>

> [!Tip]  
> <span data-ttu-id="84266-109">![混合式支柱圖](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="84266-109">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="84266-110">Microsoft Azure Stack Hub 是 Azure 的延伸模組。</span><span class="sxs-lookup"><span data-stu-id="84266-110">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="84266-111">Azure Stack Hub 可將雲端運算的靈活性與創新能力導入您的內部部署環境中，並啟用獨特的混合式雲端，讓您能夠隨處建置及部署混合式應用程式。</span><span class="sxs-lookup"><span data-stu-id="84266-111">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="84266-112">[混合式應用程式設計考量](overview-app-design-considerations.md)一文檢閱了設計、部署和操作混合式應用程式時的軟體品質要素 (放置、延展性、可用性、復原、管理性和安全性)。</span><span class="sxs-lookup"><span data-stu-id="84266-112">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="84266-113">這些設計考量有助於您設計出最佳的混合式應用程式，減少生產環境可能會遇到的挑戰。</span><span class="sxs-lookup"><span data-stu-id="84266-113">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="84266-114">Prerequisites</span><span class="sxs-lookup"><span data-stu-id="84266-114">Prerequisites</span></span>

<span data-ttu-id="84266-115">開始使用此部署指南之前，請務必先：</span><span class="sxs-lookup"><span data-stu-id="84266-115">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="84266-116">檢閱[客流量偵測解決方案](pattern-retail-footfall-detection.md)主題。</span><span class="sxs-lookup"><span data-stu-id="84266-116">Review the [Footfall detection pattern](pattern-retail-footfall-detection.md) topic.</span></span>
- <span data-ttu-id="84266-117">透過以下方式，取得對 Azure Stack 開發套件 (ASDK) 或 Azure Stack Hub 整合式系統執行個體的使用者存取權：</span><span class="sxs-lookup"><span data-stu-id="84266-117">Obtain user access to an Azure Stack Development Kit (ASDK) or Azure Stack Hub integrated system instance, with:</span></span>
  - <span data-ttu-id="84266-118">安裝 [Azure App Service on Azure Stack Hub 資源提供者](/azure-stack/operator/azure-stack-app-service-overview)。</span><span class="sxs-lookup"><span data-stu-id="84266-118">The [Azure App Service on Azure Stack Hub resource provider](/azure-stack/operator/azure-stack-app-service-overview) installed.</span></span> <span data-ttu-id="84266-119">您需要 Azure Stack Hub 執行個體的操作員存取權，或與您的系統管理員合作以進行安裝。</span><span class="sxs-lookup"><span data-stu-id="84266-119">You need operator access to your Azure Stack Hub instance, or work with your administrator to install.</span></span>
  - <span data-ttu-id="84266-120">一個可提供 App Service 和儲存體配額的訂用帳戶。</span><span class="sxs-lookup"><span data-stu-id="84266-120">A subscription to an offer that provides App Service and Storage quota.</span></span> <span data-ttu-id="84266-121">您需要操作員存取權才能建立供應專案。</span><span class="sxs-lookup"><span data-stu-id="84266-121">You need operator access to create an offer.</span></span>
- <span data-ttu-id="84266-122">取得 Azure 訂用帳戶的存取權。</span><span class="sxs-lookup"><span data-stu-id="84266-122">Obtain access to an Azure subscription.</span></span>
  - <span data-ttu-id="84266-123">如果您沒有 Azure 訂用帳戶，請在開始前先註冊一個[免費試用帳戶](https://azure.microsoft.com/free/)。</span><span class="sxs-lookup"><span data-stu-id="84266-123">If you don't have an Azure subscription, sign up for a [free trial account](https://azure.microsoft.com/free/) before you begin.</span></span>
- <span data-ttu-id="84266-124">在您的目錄中建立兩個服務主體：</span><span class="sxs-lookup"><span data-stu-id="84266-124">Create two service principals in your directory:</span></span>
  - <span data-ttu-id="84266-125">一個設定為與 Azure 資源搭配使用，並具備 Azure 訂用帳戶範圍的存取權。</span><span class="sxs-lookup"><span data-stu-id="84266-125">One set up for use with Azure resources, with access at the Azure subscription scope.</span></span>
  - <span data-ttu-id="84266-126">一個設定為與 Azure Stack Hub 資源搭配使用，並具備 Azure Stack Hub 訂用帳戶範圍的存取權。</span><span class="sxs-lookup"><span data-stu-id="84266-126">One set up for use with Azure Stack Hub resources, with access at the Azure Stack Hub subscription scope.</span></span>
  - <span data-ttu-id="84266-127">若要深入了解建立服務主體及授權存取的相關資訊，請參閱[使用應用程式身分識別來存取資源](/azure-stack/operator/azure-stack-create-service-principals)。</span><span class="sxs-lookup"><span data-stu-id="84266-127">To learn more about creating service principals and authorizing access, see [Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals).</span></span> <span data-ttu-id="84266-128">如果您偏好使用 Azure CLI，請參閱[使用 Azure CLI 建立 Azure 服務主體](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest&preserve-view=true)。</span><span class="sxs-lookup"><span data-stu-id="84266-128">If you prefer to use Azure CLI, see [Create an Azure service principal with Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest&preserve-view=true).</span></span>
- <span data-ttu-id="84266-129">在 Azure 或 Azure Stack Hub 中部署 Azure 認知服務。</span><span class="sxs-lookup"><span data-stu-id="84266-129">Deploy Azure Cognitive Services in Azure or Azure Stack Hub.</span></span>
  - <span data-ttu-id="84266-130">首先，請[深入了解認知服務](https://azure.microsoft.com/services/cognitive-services/)。</span><span class="sxs-lookup"><span data-stu-id="84266-130">First, [learn more about Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).</span></span>
  - <span data-ttu-id="84266-131">接著請造訪[將 Azure 認知服務部署至 Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services)，以在 Azure Stack Hub 上部署認知服務。</span><span class="sxs-lookup"><span data-stu-id="84266-131">Then visit [Deploy Azure Cognitive Services to Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services) to deploy Cognitive Services on Azure Stack Hub.</span></span> <span data-ttu-id="84266-132">您必須先註冊才能存取預覽版。</span><span class="sxs-lookup"><span data-stu-id="84266-132">You first need to sign up for access to the preview.</span></span>
- <span data-ttu-id="84266-133">複製或下載未設定的 Azure 自訂視覺 AI 開發套件。</span><span class="sxs-lookup"><span data-stu-id="84266-133">Clone or download an unconfigured Azure Custom Vision AI Dev Kit.</span></span> <span data-ttu-id="84266-134">如需詳細資訊，請參閱[視覺 AI 開發套件](https://azure.github.io/Vision-AI-DevKit-Pages/) \(英文\)。</span><span class="sxs-lookup"><span data-stu-id="84266-134">For details, see the [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).</span></span>
- <span data-ttu-id="84266-135">註冊 Power BI 帳戶。</span><span class="sxs-lookup"><span data-stu-id="84266-135">Sign up for a Power BI account.</span></span>
- <span data-ttu-id="84266-136">Azure 認知服務臉部 API 訂用帳戶金鑰與端點 URL。</span><span class="sxs-lookup"><span data-stu-id="84266-136">An Azure Cognitive Services Face API subscription key and endpoint URL.</span></span> <span data-ttu-id="84266-137">您可以透過[試用認知服務](https://azure.microsoft.com/try/cognitive-services/?api=face-api)免費試用取得這兩個項目。</span><span class="sxs-lookup"><span data-stu-id="84266-137">You can get both with the [Try Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api) free trial.</span></span> <span data-ttu-id="84266-138">或者，依照[建立認知服務帳戶](/azure/cognitive-services/cognitive-services-apis-create-account)中的指示取得免費試用。</span><span class="sxs-lookup"><span data-stu-id="84266-138">Or, follow the instructions in [Create a Cognitive Services account](/azure/cognitive-services/cognitive-services-apis-create-account).</span></span>
- <span data-ttu-id="84266-139">安裝下列開發資源：</span><span class="sxs-lookup"><span data-stu-id="84266-139">Install the following development resources:</span></span>
  - [<span data-ttu-id="84266-140">Azure CLI 2.0</span><span class="sxs-lookup"><span data-stu-id="84266-140">Azure CLI 2.0</span></span>](/azure-stack/user/azure-stack-version-profiles-azurecli2)
  - [<span data-ttu-id="84266-141">Docker CE</span><span class="sxs-lookup"><span data-stu-id="84266-141">Docker CE</span></span>](https://hub.docker.com/search/?type=edition&offering=community)
  - <span data-ttu-id="84266-142">[Porter](https://porter.sh/)。</span><span class="sxs-lookup"><span data-stu-id="84266-142">[Porter](https://porter.sh/).</span></span> <span data-ttu-id="84266-143">您可以使用 Porter 來部署雲端應用程式，並使用為您提供的 CNAB 組合資訊清單。</span><span class="sxs-lookup"><span data-stu-id="84266-143">You use Porter to deploy cloud apps using CNAB bundle manifests that are provided for you.</span></span>
  - [<span data-ttu-id="84266-144">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="84266-144">Visual Studio Code</span></span>](https://code.visualstudio.com/)
  - [<span data-ttu-id="84266-145">適用於 Visual Studio Code 的 Azure IoT 工具</span><span class="sxs-lookup"><span data-stu-id="84266-145">Azure IoT Tools for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - <span data-ttu-id="84266-146">[適用於 Visual Studio Code 的 Python 擴充功能](https://marketplace.visualstudio.com/items?itemName=ms-python.python) \(英文\)</span><span class="sxs-lookup"><span data-stu-id="84266-146">[Python extension for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-python.python)</span></span>
  - [<span data-ttu-id="84266-147">Python</span><span class="sxs-lookup"><span data-stu-id="84266-147">Python</span></span>](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a><span data-ttu-id="84266-148">部署混合式雲端應用程式</span><span class="sxs-lookup"><span data-stu-id="84266-148">Deploy the hybrid cloud app</span></span>

<span data-ttu-id="84266-149">首先，您可以使用 Porter CLI 來產生認證集，然後部署雲端應用程式。</span><span class="sxs-lookup"><span data-stu-id="84266-149">First you use the Porter CLI to generate a credential set, then deploy the cloud app.</span></span>  

1. <span data-ttu-id="84266-150">從 https://github.com/azure-samples/azure-intelligent-edge-patterns 複製或下載解決方案範例程式碼。</span><span class="sxs-lookup"><span data-stu-id="84266-150">Clone or download the solution sample code from https://github.com/azure-samples/azure-intelligent-edge-patterns.</span></span> 

1. <span data-ttu-id="84266-151">Porter 將會產生一組將應用程式部署自動化的認證。</span><span class="sxs-lookup"><span data-stu-id="84266-151">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="84266-152">執行認證產生命令之前，請先確定有下列項目可用：</span><span class="sxs-lookup"><span data-stu-id="84266-152">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="84266-153">用來存取 Azure 資源的服務主體，包括服務主體識別碼、金鑰和租用戶 DNS。</span><span class="sxs-lookup"><span data-stu-id="84266-153">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="84266-154">Azure 訂用帳戶的訂用帳戶識別碼。</span><span class="sxs-lookup"><span data-stu-id="84266-154">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="84266-155">用來存取 Azure Stack Hub 資源的服務主體，包括服務主體識別碼、金鑰和租用戶 DNS。</span><span class="sxs-lookup"><span data-stu-id="84266-155">A service principal for accessing Azure Stack Hub resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="84266-156">Azure Stack Hub 訂用帳戶的訂用帳戶識別碼。</span><span class="sxs-lookup"><span data-stu-id="84266-156">The subscription ID for your Azure Stack Hub subscription.</span></span>
    - <span data-ttu-id="84266-157">您的 Azure 認知服務臉部 API 金鑰與資源端點 URL。</span><span class="sxs-lookup"><span data-stu-id="84266-157">Your Azure Cognitive Services Face API key and resource endpoint URL.</span></span>

1. <span data-ttu-id="84266-158">執行 Porter 認證產生程序並遵循提示進行：</span><span class="sxs-lookup"><span data-stu-id="84266-158">Run the Porter credential generation process and follow the prompts:</span></span>

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. <span data-ttu-id="84266-159">Porter 也需要一組參數來執行。</span><span class="sxs-lookup"><span data-stu-id="84266-159">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="84266-160">建立參數文字檔，並輸入下列名稱/值組。</span><span class="sxs-lookup"><span data-stu-id="84266-160">Create a parameter text file and enter the following name/value pairs.</span></span> <span data-ttu-id="84266-161">如果您需要任何和必要值有關的協助，請洽詢您的 Azure Stack Hub 系統管理員。</span><span class="sxs-lookup"><span data-stu-id="84266-161">Ask your Azure Stack Hub administrator if you need assistance with any of the required values.</span></span>

   > [!NOTE] 
   > <span data-ttu-id="84266-162">`resource suffix` 值是用來確保部署的資源具有在整個 Azure 上唯一的名稱。</span><span class="sxs-lookup"><span data-stu-id="84266-162">The `resource suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="84266-163">它必須是字母和數字的唯一字串組合，且不能超過 8 個字元。</span><span class="sxs-lookup"><span data-stu-id="84266-163">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

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
   <span data-ttu-id="84266-164">儲存文字檔並記下其路徑。</span><span class="sxs-lookup"><span data-stu-id="84266-164">Save the text file and make a note of its path.</span></span>

1. <span data-ttu-id="84266-165">您現在已經準備好使用 Porter 部署混合式雲端應用程式。</span><span class="sxs-lookup"><span data-stu-id="84266-165">You're now ready to deploy the hybrid cloud app using Porter.</span></span> <span data-ttu-id="84266-166">執行 install 命令並監看，因為資源已部署至 Azure 和 Azure Stack Hub：</span><span class="sxs-lookup"><span data-stu-id="84266-166">Run the install command and watch as resources are deployed to Azure and Azure Stack Hub:</span></span>

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. <span data-ttu-id="84266-167">完成部署之後，請記下下列值：</span><span class="sxs-lookup"><span data-stu-id="84266-167">Once deployment is complete, make note of the following values:</span></span>
    - <span data-ttu-id="84266-168">觀景窗的連接字串。</span><span class="sxs-lookup"><span data-stu-id="84266-168">The camera's connection string.</span></span>
    - <span data-ttu-id="84266-169">映像儲存體帳戶連接字串。</span><span class="sxs-lookup"><span data-stu-id="84266-169">The image storage account connection string.</span></span>
    - <span data-ttu-id="84266-170">資源群組名稱。</span><span class="sxs-lookup"><span data-stu-id="84266-170">The resource group names.</span></span>

## <a name="prepare-the-custom-vision-ai-devkit"></a><span data-ttu-id="84266-171">準備自訂視覺 AI 開發套件</span><span class="sxs-lookup"><span data-stu-id="84266-171">Prepare the Custom Vision AI DevKit</span></span>

<span data-ttu-id="84266-172">接下來，設定自訂視覺 AI 開發套件，如[視覺 AI DevKit 快速入門](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/)中所示。</span><span class="sxs-lookup"><span data-stu-id="84266-172">Next, set up the Custom Vision AI Dev Kit as shown in the [Vision AI DevKit quickstart](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span></span> <span data-ttu-id="84266-173">您也可以使用在上一個步驟中提供的連接字串，來設定及測試您的觀景窗。</span><span class="sxs-lookup"><span data-stu-id="84266-173">You also set up and test your camera, using the connection string provided in the previous step.</span></span>

## <a name="deploy-the-camera-app"></a><span data-ttu-id="84266-174">部署觀景窗應用程式</span><span class="sxs-lookup"><span data-stu-id="84266-174">Deploy the camera app</span></span>

<span data-ttu-id="84266-175">請使用 Porter CLI 來產生認證集，然後部署觀景窗應用程式。</span><span class="sxs-lookup"><span data-stu-id="84266-175">Use the Porter CLI to generate a credential set, then deploy the camera app.</span></span>

1. <span data-ttu-id="84266-176">Porter 將會產生一組將應用程式部署自動化的認證。</span><span class="sxs-lookup"><span data-stu-id="84266-176">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="84266-177">執行認證產生命令之前，請先確定有下列項目可用：</span><span class="sxs-lookup"><span data-stu-id="84266-177">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="84266-178">用來存取 Azure 資源的服務主體，包括服務主體識別碼、金鑰和租用戶 DNS。</span><span class="sxs-lookup"><span data-stu-id="84266-178">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="84266-179">Azure 訂用帳戶的訂用帳戶識別碼。</span><span class="sxs-lookup"><span data-stu-id="84266-179">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="84266-180">您在部署雲端應用程式時提供的映像儲存體帳戶連接字串。</span><span class="sxs-lookup"><span data-stu-id="84266-180">The image storage account connection string provided when you deployed the cloud app.</span></span>

1. <span data-ttu-id="84266-181">執行 Porter 認證產生程序並遵循提示進行：</span><span class="sxs-lookup"><span data-stu-id="84266-181">Run the Porter credential generation process and follow the prompts:</span></span>

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. <span data-ttu-id="84266-182">Porter 也需要一組參數來執行。</span><span class="sxs-lookup"><span data-stu-id="84266-182">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="84266-183">建立參數文字檔並輸入以下文字。</span><span class="sxs-lookup"><span data-stu-id="84266-183">Create a parameter text file and enter the following text.</span></span> <span data-ttu-id="84266-184">如果您不知道部分必要值，請洽詢您的 Azure Stack Hub 系統管理員。</span><span class="sxs-lookup"><span data-stu-id="84266-184">Ask your Azure Stack Hub administrator if you don't know some of the required values.</span></span>

    > [!NOTE]
    > <span data-ttu-id="84266-185">`deployment suffix` 值是用來確保部署的資源具有在整個 Azure 上唯一的名稱。</span><span class="sxs-lookup"><span data-stu-id="84266-185">The `deployment suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="84266-186">它必須是字母和數字的唯一字串組合，且不能超過 8 個字元。</span><span class="sxs-lookup"><span data-stu-id="84266-186">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    <span data-ttu-id="84266-187">儲存文字檔並記下其路徑。</span><span class="sxs-lookup"><span data-stu-id="84266-187">Save the text file and make a note of its path.</span></span>

4. <span data-ttu-id="84266-188">您現在已經準備好使用 Porter 部署觀景窗應用程式。</span><span class="sxs-lookup"><span data-stu-id="84266-188">You're now ready to deploy the camera app using Porter.</span></span> <span data-ttu-id="84266-189">執行安裝命令，並在建立 IoT Edge 部署時監看。</span><span class="sxs-lookup"><span data-stu-id="84266-189">Run the install command and watch as the IoT Edge deployment is created.</span></span>

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. <span data-ttu-id="84266-190">藉由在 `https://<camera-ip>:3000/` 檢視觀景窗摘要，以確認相機的部署已完成，其中 `<camara-ip>` 是觀景窗 IP 位址。</span><span class="sxs-lookup"><span data-stu-id="84266-190">Verify that the camera's deployment is complete by viewing the camera feed at `https://<camera-ip>:3000/`, where `<camara-ip>` is the camera IP address.</span></span> <span data-ttu-id="84266-191">此步驟最多可能需要 10 分鐘。</span><span class="sxs-lookup"><span data-stu-id="84266-191">This step may take up to 10 minutes.</span></span>

## <a name="configure-azure-stream-analytics"></a><span data-ttu-id="84266-192">設定 Azure 串流分析</span><span class="sxs-lookup"><span data-stu-id="84266-192">Configure Azure Stream Analytics</span></span>

<span data-ttu-id="84266-193">現在資料已從觀景窗進入 Azure 串流分析，我們需要手動授權以與 Power BI 通訊。</span><span class="sxs-lookup"><span data-stu-id="84266-193">Now that data is flowing to Azure Stream Analytics from the camera, we need to manually authorize it to communicate with Power BI.</span></span>

1. <span data-ttu-id="84266-194">從 Azure 入口網站開啟 [所有資源]  ，然後尋找 *process-footfall\[yoursuffix\]* 作業。</span><span class="sxs-lookup"><span data-stu-id="84266-194">From the Azure portal, open **All Resources**, and the *process-footfall\[yoursuffix\]* job.</span></span>

2. <span data-ttu-id="84266-195">在串流分析作業窗格的 [作業拓撲]  區段中，選取 [輸出]  選項。</span><span class="sxs-lookup"><span data-stu-id="84266-195">In the **Job Topology** section of the Stream Analytics job pane, select the **Outputs** option.</span></span>

3. <span data-ttu-id="84266-196">選取 **traffic-output** 輸出接收。</span><span class="sxs-lookup"><span data-stu-id="84266-196">Select the **traffic-output** output sink.</span></span>

4. <span data-ttu-id="84266-197">選取 [更新授權]  ，然後登入您的 Power BI 帳戶。</span><span class="sxs-lookup"><span data-stu-id="84266-197">Select **Renew authorization** and sign in to your Power BI account.</span></span>
  
    ![Power BI 中的更新授權提示](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. <span data-ttu-id="84266-199">儲存輸出設定。</span><span class="sxs-lookup"><span data-stu-id="84266-199">Save the output settings.</span></span>

6. <span data-ttu-id="84266-200">前往 [概觀]  窗格，然後選取 [啟動]  即可開始傳送資料至 Power BI。</span><span class="sxs-lookup"><span data-stu-id="84266-200">Go to the **Overview** pane and select **Start** to start sending data to Power BI.</span></span>

7. <span data-ttu-id="84266-201">針對作業輸出開始時間選取 [現在]  並選取 [啟動]  。</span><span class="sxs-lookup"><span data-stu-id="84266-201">Select **Now** for job output start time and select **Start**.</span></span> <span data-ttu-id="84266-202">您可以在通知列中檢視作業狀態。</span><span class="sxs-lookup"><span data-stu-id="84266-202">You can view the job status in the notification bar.</span></span>

## <a name="create-a-power-bi-dashboard"></a><span data-ttu-id="84266-203">建立 Power BI 儀表板</span><span class="sxs-lookup"><span data-stu-id="84266-203">Create a Power BI Dashboard</span></span>

1. <span data-ttu-id="84266-204">作業成功之後，請移至 [Power BI](https://powerbi.com/)，然後使用公司或學校帳戶登入。</span><span class="sxs-lookup"><span data-stu-id="84266-204">Once the job succeeds, go to [Power BI](https://powerbi.com/) and sign in with your work or school account.</span></span> <span data-ttu-id="84266-205">如果串流分析作業查詢會輸出結果，您所建立的 *footfall-dataset* 資料集會存在於 [資料集]  索引標籤下。</span><span class="sxs-lookup"><span data-stu-id="84266-205">If the Stream Analytics job query is outputting results, the *footfall-dataset* dataset you created exists under the **Datasets** tab.</span></span>

2. <span data-ttu-id="84266-206">從 Power BI 工作區中選取 [+ 建立]  ，建立名為「客流量分析」  的新儀表板。</span><span class="sxs-lookup"><span data-stu-id="84266-206">From your Power BI workspace, select **+ Create** to create a new dashboard named *Footfall Analysis.*</span></span>

3. <span data-ttu-id="84266-207">在視窗頂端，選取 [新增圖格]  。</span><span class="sxs-lookup"><span data-stu-id="84266-207">At the top of the window, select **Add tile**.</span></span> <span data-ttu-id="84266-208">接著選取 [自訂串流資料]  ，然後選取 [下一步]  。</span><span class="sxs-lookup"><span data-stu-id="84266-208">Then select **Custom Streaming Data** and **Next**.</span></span> <span data-ttu-id="84266-209">在 [您的資料集]  下方選擇 **footfall-dataset**。</span><span class="sxs-lookup"><span data-stu-id="84266-209">Choose the **footfall-dataset** under **Your Datasets**.</span></span> <span data-ttu-id="84266-210">從 [視覺效果類型]  下拉式清單中選取 [卡]  ，並將 [年齡]  新增至 [欄位]  。</span><span class="sxs-lookup"><span data-stu-id="84266-210">Select **Card** from the **Visualization type** dropdown, and add **age** to **Fields**.</span></span> <span data-ttu-id="84266-211">選取 [下一步]  以輸入圖格的名稱，然後選取 [套用]  以建立圖格。</span><span class="sxs-lookup"><span data-stu-id="84266-211">Select **Next** to enter a name for the tile, and then select **Apply** to create the tile.</span></span>

4. <span data-ttu-id="84266-212">您可以視需要新增其他欄位和卡片。</span><span class="sxs-lookup"><span data-stu-id="84266-212">You can add additional fields and cards as desired.</span></span>

## <a name="test-your-solution"></a><span data-ttu-id="84266-213">測試解決方案</span><span class="sxs-lookup"><span data-stu-id="84266-213">Test Your Solution</span></span>

<span data-ttu-id="84266-214">觀察您在 Power BI 中所建立卡片包含的資料如何隨著不同的人在觀景窗前面走動而改變。</span><span class="sxs-lookup"><span data-stu-id="84266-214">Observe how the data in the cards you created in Power BI changes as different people walk in front of the camera.</span></span> <span data-ttu-id="84266-215">在記錄之後，可能需要最多 20 秒的時間才會出現推斷。</span><span class="sxs-lookup"><span data-stu-id="84266-215">Inferences may take up to 20 seconds to appear once recorded.</span></span>

## <a name="remove-your-solution"></a><span data-ttu-id="84266-216">移除您的解決方案</span><span class="sxs-lookup"><span data-stu-id="84266-216">Remove Your Solution</span></span>

<span data-ttu-id="84266-217">如果想要移除解決方案，請使用您為部署所建立的相同參數檔案，並使用 Porter 執行下列命令：</span><span class="sxs-lookup"><span data-stu-id="84266-217">If you'd like to remove your solution, run the following commands using Porter, using the same parameter files that you created for deployment:</span></span>

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a><span data-ttu-id="84266-218">後續步驟</span><span class="sxs-lookup"><span data-stu-id="84266-218">Next steps</span></span>

- <span data-ttu-id="84266-219">深入了解[混合式應用程式設計考量](overview-app-design-considerations.md)</span><span class="sxs-lookup"><span data-stu-id="84266-219">Learn more about [Hybrid app design considerations](overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="84266-220">檢閱 [GitHub 上此範例的程式碼](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis)並提出改進建議。</span><span class="sxs-lookup"><span data-stu-id="84266-220">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span></span>
