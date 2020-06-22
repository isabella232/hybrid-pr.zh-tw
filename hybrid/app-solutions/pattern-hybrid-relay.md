---
title: Azure 和 Azure Stack Hub 中的混合式轉送模式
description: 使用 Azure 與 Azure Stack Hub 中的混合式轉送模式來連線到受防火牆保護的邊緣資源。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 03b20a20a04f620c977fb20e1ea26f5982e42721
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910127"
---
# <a name="hybrid-relay-pattern"></a><span data-ttu-id="53cd0-103">Hybrid 轉送模式</span><span class="sxs-lookup"><span data-stu-id="53cd0-103">Hybrid relay pattern</span></span>

<span data-ttu-id="53cd0-104">瞭解如何使用混合式轉送模式和 Azure 轉送來連線到邊緣資源或受防火牆保護的裝置。</span><span class="sxs-lookup"><span data-stu-id="53cd0-104">Learn how to connect to edge resources or devices protected by firewalls using the hybrid relay pattern and Azure Relay.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="53cd0-105">內容和問題</span><span class="sxs-lookup"><span data-stu-id="53cd0-105">Context and problem</span></span>

<span data-ttu-id="53cd0-106">邊緣裝置通常位於公司防火牆或 NAT 裝置後方。</span><span class="sxs-lookup"><span data-stu-id="53cd0-106">Edge devices are often behind a corporate firewall or NAT device.</span></span> <span data-ttu-id="53cd0-107">雖然這些裝置是安全的，但可能無法與公用雲端或其他公司網路上的邊緣裝置通訊。</span><span class="sxs-lookup"><span data-stu-id="53cd0-107">Although they're secure, they may be unable to communicate with the public cloud or edge devices on other corporate networks.</span></span> <span data-ttu-id="53cd0-108">可能必須以安全的方式，向公用雲端中的使用者公開特定連接埠與功能。</span><span class="sxs-lookup"><span data-stu-id="53cd0-108">It may be necessary to expose certain ports and functionality to users in the public cloud in a secure manner.</span></span>

## <a name="solution"></a><span data-ttu-id="53cd0-109">解決方法</span><span class="sxs-lookup"><span data-stu-id="53cd0-109">Solution</span></span>

<span data-ttu-id="53cd0-110">混合式轉送模式會使用 Azure 轉送，在無法直接通訊的兩個端點之間建立 Websocket 通道。</span><span class="sxs-lookup"><span data-stu-id="53cd0-110">The hybrid relay pattern uses Azure Relay to establish a WebSockets tunnel between two endpoints that can't directly communicate.</span></span> <span data-ttu-id="53cd0-111">不是內部部署但需要連線到內部部署端點的裝置，將會連線至公用雲端中的端點。</span><span class="sxs-lookup"><span data-stu-id="53cd0-111">Devices that aren't on-premises but need to connect to an on-premises endpoint will connect to an endpoint in the public cloud.</span></span> <span data-ttu-id="53cd0-112">此端點會在安全通道上，透過預先定義的路由將流量重新導向。</span><span class="sxs-lookup"><span data-stu-id="53cd0-112">This endpoint will redirect the traffic on predefined routes over a secure channel.</span></span> <span data-ttu-id="53cd0-113">內部部署環境內部的端點會接收流量，並將其路由傳送到正確的目的地。</span><span class="sxs-lookup"><span data-stu-id="53cd0-113">An endpoint inside the on-premises environment receives the traffic and routes it to the correct destination.</span></span>

![混合式轉送模式解決方案架構](media/pattern-hybrid-relay/solution-architecture.png)

<span data-ttu-id="53cd0-115">以下是混合式轉送模式的運作方式：</span><span class="sxs-lookup"><span data-stu-id="53cd0-115">Here's how the hybrid relay pattern works:</span></span>

1. <span data-ttu-id="53cd0-116">裝置透過預先定義的連接埠連線至 Azure 中的虛擬機器 (VM)。</span><span class="sxs-lookup"><span data-stu-id="53cd0-116">A device connects to the virtual machine (VM) in Azure, on a predefined port.</span></span>
2. <span data-ttu-id="53cd0-117">流量會轉送到 Azure 中的 Azure 轉送。</span><span class="sxs-lookup"><span data-stu-id="53cd0-117">Traffic is forwarded to the Azure Relay in Azure.</span></span>
3. <span data-ttu-id="53cd0-118">Azure Stack 中樞上的 VM 已建立與 Azure 轉送的長時間連線，會接收流量並將其轉送至目的地。</span><span class="sxs-lookup"><span data-stu-id="53cd0-118">The VM on Azure Stack Hub, which has already established a long-lived connection to the Azure Relay, receives the traffic and forwards it on to the destination.</span></span>
4. <span data-ttu-id="53cd0-119">內部部署服務或端點會處理要求。</span><span class="sxs-lookup"><span data-stu-id="53cd0-119">The on-premises service or endpoint processes the request.</span></span>

## <a name="components"></a><span data-ttu-id="53cd0-120">元件</span><span class="sxs-lookup"><span data-stu-id="53cd0-120">Components</span></span>

<span data-ttu-id="53cd0-121">此解決方案使用下列元件：</span><span class="sxs-lookup"><span data-stu-id="53cd0-121">This solution uses the following components:</span></span>

| <span data-ttu-id="53cd0-122">階層</span><span class="sxs-lookup"><span data-stu-id="53cd0-122">Layer</span></span> | <span data-ttu-id="53cd0-123">元件</span><span class="sxs-lookup"><span data-stu-id="53cd0-123">Component</span></span> | <span data-ttu-id="53cd0-124">描述</span><span class="sxs-lookup"><span data-stu-id="53cd0-124">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="53cd0-125">Azure</span><span class="sxs-lookup"><span data-stu-id="53cd0-125">Azure</span></span> | <span data-ttu-id="53cd0-126">Azure VM</span><span class="sxs-lookup"><span data-stu-id="53cd0-126">Azure VM</span></span> | <span data-ttu-id="53cd0-127">Azure VM 提供內部部署資源的可公開存取端點。</span><span class="sxs-lookup"><span data-stu-id="53cd0-127">An Azure VM provides a publicly accessible endpoint for the on-premises resource.</span></span> |
| | <span data-ttu-id="53cd0-128">Azure 轉送</span><span class="sxs-lookup"><span data-stu-id="53cd0-128">Azure Relay</span></span> | <span data-ttu-id="53cd0-129">[Azure 轉送](/azure/azure-relay/)提供基礎結構來維護 Azure vm 與 AZURE STACK Hub vm 之間的通道和連線。</span><span class="sxs-lookup"><span data-stu-id="53cd0-129">An [Azure Relay](/azure/azure-relay/) provides the infrastructure for maintaining the tunnel and connection between the Azure VM and Azure Stack Hub VM.</span></span>|
| <span data-ttu-id="53cd0-130">Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="53cd0-130">Azure Stack Hub</span></span> | <span data-ttu-id="53cd0-131">計算</span><span class="sxs-lookup"><span data-stu-id="53cd0-131">Compute</span></span> | <span data-ttu-id="53cd0-132">Azure Stack Hub VM 提供混合式轉送通道的伺服器端。</span><span class="sxs-lookup"><span data-stu-id="53cd0-132">An Azure Stack Hub VM provides the server-side of the Hybrid Relay tunnel.</span></span> |
| | <span data-ttu-id="53cd0-133">儲存體</span><span class="sxs-lookup"><span data-stu-id="53cd0-133">Storage</span></span> | <span data-ttu-id="53cd0-134">已部署到 Azure Stack Hub 中的 AKS 引擎叢集會提供可調整的彈性引擎以執行臉部 API 容器。</span><span class="sxs-lookup"><span data-stu-id="53cd0-134">The AKS engine cluster deployed into Azure Stack Hub provides a scalable, resilient engine to run the Face API container.</span></span>|

## <a name="issues-and-considerations"></a><span data-ttu-id="53cd0-135">問題和考量</span><span class="sxs-lookup"><span data-stu-id="53cd0-135">Issues and considerations</span></span>

<span data-ttu-id="53cd0-136">決定如何實作此解決方案時，請考慮下列幾點：</span><span class="sxs-lookup"><span data-stu-id="53cd0-136">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="53cd0-137">延展性</span><span class="sxs-lookup"><span data-stu-id="53cd0-137">Scalability</span></span>

<span data-ttu-id="53cd0-138">此模式只允許在用戶端與伺服器上進行 1:1 連接埠對應。</span><span class="sxs-lookup"><span data-stu-id="53cd0-138">This pattern only allows for 1:1 port mappings on the client and server.</span></span> <span data-ttu-id="53cd0-139">例如，如果 Azure 端點上的一個服務在連接埠 80 建立通道，它就無法供另一個服務使用。</span><span class="sxs-lookup"><span data-stu-id="53cd0-139">For example, if port 80 is tunneled for one service on the Azure endpoint, it can't be used for another service.</span></span> <span data-ttu-id="53cd0-140">應據以規劃連接埠對應。</span><span class="sxs-lookup"><span data-stu-id="53cd0-140">Port mappings should be planned accordingly.</span></span> <span data-ttu-id="53cd0-141">Azure 轉送和 Vm 應該適當地調整來處理流量。</span><span class="sxs-lookup"><span data-stu-id="53cd0-141">The Azure Relay and VMs should be appropriately scaled to handle traffic.</span></span>

### <a name="availability"></a><span data-ttu-id="53cd0-142">可用性</span><span class="sxs-lookup"><span data-stu-id="53cd0-142">Availability</span></span>

<span data-ttu-id="53cd0-143">這些通道與連線並不是多餘的。</span><span class="sxs-lookup"><span data-stu-id="53cd0-143">These tunnels and connections aren't redundant.</span></span> <span data-ttu-id="53cd0-144">若要確保高可用性，您可以實作錯誤檢查程式碼。</span><span class="sxs-lookup"><span data-stu-id="53cd0-144">To ensure high-availability, you may want to implement error checking code.</span></span> <span data-ttu-id="53cd0-145">另一個選項是在負載平衡器後方有 Azure 轉送連接的 Vm 集區。</span><span class="sxs-lookup"><span data-stu-id="53cd0-145">Another option is to have a pool of Azure Relay-connected VMs behind a load balancer.</span></span>

### <a name="manageability"></a><span data-ttu-id="53cd0-146">管理能力</span><span class="sxs-lookup"><span data-stu-id="53cd0-146">Manageability</span></span>

<span data-ttu-id="53cd0-147">此解決方案可能會跨越許多裝置和位置而變得不好管理。</span><span class="sxs-lookup"><span data-stu-id="53cd0-147">This solution can span many devices and locations, which could get unwieldy.</span></span> <span data-ttu-id="53cd0-148">Azure 的 IoT 服務可讓新的位置與裝置自動上線，並讓其保持最新狀態。</span><span class="sxs-lookup"><span data-stu-id="53cd0-148">Azure's IoT services can automatically bring new locations and devices online and keep them up to date.</span></span>

### <a name="security"></a><span data-ttu-id="53cd0-149">安全性</span><span class="sxs-lookup"><span data-stu-id="53cd0-149">Security</span></span>

<span data-ttu-id="53cd0-150">這種模式可讓您從邊緣對內部裝置上的連接埠進行自由存取。</span><span class="sxs-lookup"><span data-stu-id="53cd0-150">This pattern as shown allows for unfettered access to a port on an internal device from the edge.</span></span> <span data-ttu-id="53cd0-151">請考慮為內部裝置上的服務，或在混合式轉送端點前方新增驗證機制。</span><span class="sxs-lookup"><span data-stu-id="53cd0-151">Consider adding an authentication mechanism to the service on the internal device, or in front of the hybrid relay endpoint.</span></span>

## <a name="next-steps"></a><span data-ttu-id="53cd0-152">後續步驟</span><span class="sxs-lookup"><span data-stu-id="53cd0-152">Next steps</span></span>

<span data-ttu-id="53cd0-153">深入了解此文章介紹的更多相關主題：</span><span class="sxs-lookup"><span data-stu-id="53cd0-153">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="53cd0-154">此模式會使用 Azure 轉送。</span><span class="sxs-lookup"><span data-stu-id="53cd0-154">This pattern uses Azure Relay.</span></span> <span data-ttu-id="53cd0-155">如需詳細資訊，請參閱[Azure 轉送檔](/azure/azure-relay/)。</span><span class="sxs-lookup"><span data-stu-id="53cd0-155">For more information, see the [Azure Relay documentation](/azure/azure-relay/).</span></span>
- <span data-ttu-id="53cd0-156">若要深入了解最佳做法並獲得任何其他問題的解答，請參閱[混合式應用程式設計考量](overview-app-design-considerations.md)。</span><span class="sxs-lookup"><span data-stu-id="53cd0-156">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and get answers to any additional questions.</span></span>
- <span data-ttu-id="53cd0-157">若要深入了解產品與解決方案的完整組合，請參閱 [Azure Stack 系列產品和解決方案](/azure-stack)。</span><span class="sxs-lookup"><span data-stu-id="53cd0-157">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="53cd0-158">當您準備好測試解決方案範例時，請繼續參閱[混合式轉送解決方案部署指南](https://aka.ms/hybridrelaydeployment)。</span><span class="sxs-lookup"><span data-stu-id="53cd0-158">When you're ready to test the solution example, continue with the [Hybrid relay solution deployment guide](https://aka.ms/hybridrelaydeployment).</span></span> <span data-ttu-id="53cd0-159">部署指南提供部署及測試其元件的逐步指示。</span><span class="sxs-lookup"><span data-stu-id="53cd0-159">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>