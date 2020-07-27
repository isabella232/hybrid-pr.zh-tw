---
title: 為 Azure 和 Azure Stack Hub 應用程式設定混合式雲端身分識別
description: 了解如何為 Azure 和 Azure Stack Hub 應用程式設定混合式雲端身分識別。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 650eef0f144ecafab4586d93f72e1defdf4a61ce
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477247"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a><span data-ttu-id="71c52-103">為 Azure 和 Azure Stack Hub 應用程式設定混合式雲端身分識別</span><span class="sxs-lookup"><span data-stu-id="71c52-103">Configure hybrid cloud identity for Azure and Azure Stack Hub apps</span></span>

<span data-ttu-id="71c52-104">了解如何為 Azure 和 Azure Stack Hub 應用程式設定混合式雲端身分識別。</span><span class="sxs-lookup"><span data-stu-id="71c52-104">Learn how to configure a hybrid cloud identity for your Azure and Azure Stack Hub apps.</span></span>

<span data-ttu-id="71c52-105">在全域 Azure 和 Azure Stack Hub 中，您都有兩個用來授與應用程式存取權的選項。</span><span class="sxs-lookup"><span data-stu-id="71c52-105">You have two options for granting access to your apps in both global Azure and Azure Stack Hub.</span></span>

 * <span data-ttu-id="71c52-106">當 Azure Stack Hub 可持續連線至網際網路時，您可以使用 Azure Active Directory (Azure AD)。</span><span class="sxs-lookup"><span data-stu-id="71c52-106">When Azure Stack Hub has a continuous connection to the internet, you can use Azure Active Directory (Azure AD).</span></span>
 * <span data-ttu-id="71c52-107">當 Azure Stack Hub 的網際網路連線中斷時，您可以使用 Azure Directory 同盟服務 (AD FS)。</span><span class="sxs-lookup"><span data-stu-id="71c52-107">When Azure Stack Hub is disconnected from the internet, you can use Azure Directory Federated Services (AD FS).</span></span>

<span data-ttu-id="71c52-108">您可以使用服務主體來授與 Azure Stack Hub 應用程式存取權，以便在 Azure Stack Hub 中使用 Azure Resource Manager 進行部署或設定。</span><span class="sxs-lookup"><span data-stu-id="71c52-108">You use service principals to grant access to your Azure Stack Hub apps for deployment or configuration using the Azure Resource Manager in Azure Stack Hub.</span></span>

<span data-ttu-id="71c52-109">在本解決方案中，您會建置環境範例，用以：</span><span class="sxs-lookup"><span data-stu-id="71c52-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="71c52-110">在全域 Azure 與 Azure Stack Hub 中建立混合式身分識別</span><span class="sxs-lookup"><span data-stu-id="71c52-110">Establish a hybrid identity in global Azure and Azure Stack Hub</span></span>
> - <span data-ttu-id="71c52-111">擷取用以存取 Azure Stack Hub API 的權杖。</span><span class="sxs-lookup"><span data-stu-id="71c52-111">Retrieve a token to access the Azure Stack Hub API.</span></span>

<span data-ttu-id="71c52-112">您必須具有 Azure Stack Hub 操作員權限，才能執行本解決方案中的步驟。</span><span class="sxs-lookup"><span data-stu-id="71c52-112">You must have Azure Stack Hub operator permissions for the steps in this solution.</span></span>

> [!Tip]  
> <span data-ttu-id="71c52-113">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="71c52-113">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="71c52-114">Microsoft Azure Stack Hub 是 Azure 的延伸模組。</span><span class="sxs-lookup"><span data-stu-id="71c52-114">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="71c52-115">Azure Stack Hub 可將雲端運算的靈活性和創新能力導入您的內部部署環境中，並啟用獨特的混合式雲端，讓您能夠隨處建置及部署混合式應用程式。</span><span class="sxs-lookup"><span data-stu-id="71c52-115">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="71c52-116">[混合式應用程式設計考量](overview-app-design-considerations.md)一文檢閱了設計、部署和操作混合式應用程式時的軟體品質要素 (放置、延展性、可用性、復原、管理性和安全性)。</span><span class="sxs-lookup"><span data-stu-id="71c52-116">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="71c52-117">這些設計考量有助於您設計出最佳的混合式應用程式，減少生產環境可能會遇到的挑戰。</span><span class="sxs-lookup"><span data-stu-id="71c52-117">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a><span data-ttu-id="71c52-118">在入口網站中為 Azure AD 建立服務主體</span><span class="sxs-lookup"><span data-stu-id="71c52-118">Create a service principal for Azure AD in the portal</span></span>

<span data-ttu-id="71c52-119">如果您使用 Azure AD 作為身分識別存放區部署了 Azure Stack Hub，則可以如同對 Azure 般建立服務主體。</span><span class="sxs-lookup"><span data-stu-id="71c52-119">If you deployed Azure Stack Hub using Azure AD as the identity store, you can create service principals just like you do for Azure.</span></span> <span data-ttu-id="71c52-120">[使用應用程式身分識別來存取資源](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity)會說明如何透過入口網站逐步執行。</span><span class="sxs-lookup"><span data-stu-id="71c52-120">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity) shows you how to perform the steps through the portal.</span></span> <span data-ttu-id="71c52-121">請確認您具備[必要的 Azure AD 權限](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions)。</span><span class="sxs-lookup"><span data-stu-id="71c52-121">Be sure you have the [required Azure AD permissions](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) before beginning.</span></span>

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a><span data-ttu-id="71c52-122">使用 PowerShell 為 AD FS 建立服務主體</span><span class="sxs-lookup"><span data-stu-id="71c52-122">Create a service principal for AD FS using PowerShell</span></span>

<span data-ttu-id="71c52-123">如果您已使用 AD FS 部署 Azure Stack Hub，則可以使用 PowerShell 來建立服務主體、指派用於存取的角色，以及從 PowerShell 使用該身分識別登入。</span><span class="sxs-lookup"><span data-stu-id="71c52-123">If you deployed Azure Stack Hub with AD FS, you can use PowerShell to create a service principal, assign a role for access, and sign in from PowerShell using that identity.</span></span> <span data-ttu-id="71c52-124">[使用應用程式身分識別來存取資源](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity)會說明如何使用 PowerShell 逐步執行必要步驟。</span><span class="sxs-lookup"><span data-stu-id="71c52-124">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity) shows you how to perform the required steps using PowerShell.</span></span>

## <a name="using-the-azure-stack-hub-api"></a><span data-ttu-id="71c52-125">使用 Azure Stack Hub API</span><span class="sxs-lookup"><span data-stu-id="71c52-125">Using the Azure Stack Hub API</span></span>

<span data-ttu-id="71c52-126">[Azure Stack Hub API](/azure-stack/user/azure-stack-rest-api-use.md) 解決方案會逐步解說擷取權杖以存取 Azure Stack Hub API 的程序。</span><span class="sxs-lookup"><span data-stu-id="71c52-126">The [Azure Stack Hub API](/azure-stack/user/azure-stack-rest-api-use.md)  solution walks you through the process of retrieving a token to access the Azure Stack Hub API.</span></span>

## <a name="connect-to-azure-stack-hub-using-powershell"></a><span data-ttu-id="71c52-127">使用 PowerShell 連線至 Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="71c52-127">Connect to Azure Stack Hub using PowerShell</span></span>

<span data-ttu-id="71c52-128">[在 Azure Stack Hub 中使用 PowerShell 啟動並執行](/azure-stack/operator/azure-stack-powershell-install.md)快速入門會引導您完成安裝 Azure PowerShell 並連線至 Azure Stack Hub 安裝所需的步驟。</span><span class="sxs-lookup"><span data-stu-id="71c52-128">The quickstart [to get up and running with PowerShell in Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install.md) walks you through the steps needed to install Azure PowerShell and connect to your Azure Stack Hub installation.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="71c52-129">Prerequisites</span><span class="sxs-lookup"><span data-stu-id="71c52-129">Prerequisites</span></span>

<span data-ttu-id="71c52-130">您需要使用您可以存取的訂用帳戶連線至 Azure AD 的 Azure Stack 安裝。</span><span class="sxs-lookup"><span data-stu-id="71c52-130">You need an Azure Stack Hub installation connected to Azure AD with a subscription you can access.</span></span> <span data-ttu-id="71c52-131">如果您沒有 Azure Stack Hub 安裝，您可以依照這些指示設定 [Azure Stack 開發套件 (ASDK)](/azure-stack/asdk/asdk-install.md)。</span><span class="sxs-lookup"><span data-stu-id="71c52-131">If you don't have an Azure Stack Hub installation, you can use these instructions to set up an [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install.md).</span></span>

#### <a name="connect-to-azure-stack-hub-using-code"></a><span data-ttu-id="71c52-132">使用程式碼連線至 Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="71c52-132">Connect to Azure Stack Hub using code</span></span>

<span data-ttu-id="71c52-133">若要使用程式碼連線至 Azure Stack Hub，請使用 Azure Resource Manager 端點 API 取得 Azure Stack Hub 安裝的驗證和圖形端點。</span><span class="sxs-lookup"><span data-stu-id="71c52-133">To connect to Azure Stack Hub using code, use the Azure Resource Manager endpoints API to get the authentication and graph endpoints for your Azure Stack Hub installation.</span></span> <span data-ttu-id="71c52-134">然後，使用 REST 要求進行驗證。</span><span class="sxs-lookup"><span data-stu-id="71c52-134">Then authenticate using REST requests.</span></span> <span data-ttu-id="71c52-135">您可以在 [GitHub](https://github.com/shriramnat/HybridARMApplication) 找到範例用戶端應用程式。</span><span class="sxs-lookup"><span data-stu-id="71c52-135">You can find a sample client application on [GitHub](https://github.com/shriramnat/HybridARMApplication).</span></span>

>[!Note]
><span data-ttu-id="71c52-136">除非您所選語言的 Azure SDK 支援 Azure API 設定檔，否則 SDK 可能無法與 Azure Stack Hub 搭配使用。</span><span class="sxs-lookup"><span data-stu-id="71c52-136">Unless the Azure SDK for your language of choice supports Azure API Profiles, the SDK may not work with Azure Stack Hub.</span></span> <span data-ttu-id="71c52-137">若要深入了解 Azure API 設定檔，請參閱[管理 API 版本設定檔](/azure-stack/user/azure-stack-version-profiles.md)一文。</span><span class="sxs-lookup"><span data-stu-id="71c52-137">To learn more about Azure API Profiles, see the [manage API version profiles](/azure-stack/user/azure-stack-version-profiles.md) article.</span></span>

## <a name="next-steps"></a><span data-ttu-id="71c52-138">後續步驟</span><span class="sxs-lookup"><span data-stu-id="71c52-138">Next steps</span></span>

- <span data-ttu-id="71c52-139">若要深入了解在 Azure Stack Hub 處理身分識別的方式，請參閱 [Azure Stack Hub 的身分識別架構](/azure-stack/operator/azure-stack-identity-architecture.md)。</span><span class="sxs-lookup"><span data-stu-id="71c52-139">To learn more about how identity is handled in Azure Stack Hub, see [Identity architecture for Azure Stack Hub](/azure-stack/operator/azure-stack-identity-architecture.md).</span></span>
- <span data-ttu-id="71c52-140">若要深入了解 Azure 雲端模式，請參閱[雲端設計模式](/azure/architecture/patterns)。</span><span class="sxs-lookup"><span data-stu-id="71c52-140">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
