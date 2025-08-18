---
title: 更新至Adobe Audience Manager DIL 8.0版（或更新版本）
description: 本文會提供將Adobe Audience Manager (AAM) Data Integration Library (DIL)程式碼更新至8.0版或更新版本的步驟和建議。 這指的是「使用者端」DIL實施，而非Adobe Analytics資料的伺服器端轉送，範圍涵蓋DTM、Adobe的Launch，以及不含Adobe標籤管理程式解決方案的實施。
feature: DIL Implementation
topics: null
activity: implement
doc-type: technical video
team: Technical Marketing
kt: 1841
role: Developer, Data Engineer
level: Intermediate
exl-id: 8c1e6ed5-0f21-427b-a681-0ecb020a0e60
source-git-commit: 62b43b5627dabf754cf821f974a56c60989ef7ef
workflow-type: tm+mt
source-wordcount: '1074'
ht-degree: 0%

---

# 更新至Adobe Audience Manager的DIL 8.0版（或更新版本） {#updating-to-adobe-audience-manager-s-dil-version-or-greater}

本文會提供將Adobe Audience Manager (AAM) [!DNL Data Integration Library] (DIL)程式碼更新至8.0版或更新版本的步驟和建議。 這指的是「使用者端」DIL實施，而非Adobe Analytics資料的伺服器端轉送，範圍涵蓋DTM、Adobe的Launch，以及不含Adobe標籤管理程式解決方案的實施。

## 概述 {#overview}

Audience Manager的[!DNL Data Integration Library] (DIL)程式碼可讓您在網站上實作AAM*。 實作舊版DIL時，不需要同時實作Adobe的Experience Cloud ID Service (ECID) （不過這是個好主意）。 從DIL 8.0版開始，ECID 3.3版或更新版本有硬性相依性。 如果您在沒有ECID 3.3的情況下實作DIL 8.0或更新版本，或使用較舊的版本，則會發生錯誤而無法正常運作。 由於您有多種方式可實施AAM，因此我們建立了此頁面以提供您一些應逐步執行的步驟以及部分建議。 在下方，您會找到依平台/實作方法劃分的這些步驟和建議。 有關DIL的詳細資訊，請參閱[檔案](https://experienceleague.adobe.com/docs/audience-manager/user-guide/dil-api/dil-overview.html?lang=zh-Hant)。

* 如本頁面的說明所述，這僅涵蓋「使用者端」DIL實作，可供沒有Adobe Analytics的AAM客戶使用。 如果您有Adobe Analytics，則應該使用實作AAM的伺服器端轉送方法。 此方法的說明請參閱[檔案](https://experienceleague.adobe.com/docs/analytics/admin/admin-tools/server-side-forwarding/ssf.html?lang=zh-Hant)。

## 重複和過時的元素和方法 {#duplicate-and-deprecated-elements-and-methods}

在舊版DIL和ECID中，有重複方法(在DIL和ECID中執行相同功能的方法)，這會造成混淆，不知道該使用哪一種。 通常您需要同時使用兩者並搭配使用，而該訊息未與客戶妥善溝通。 從DIL 8.0開始，DIL已棄用這些重複的方法和元素，建議您使用ECID版本。

例如：

* 使用[!DNL DIL.create]時，某些元素已被取代，您應該改用ECID元素。 這些元素會在[[!DNL DIL.create] 檔案](https://experienceleague.adobe.com/docs/audience-manager/user-guide/dil-api/class-level-dil-methods/dil-create.html?lang=zh-Hant)中呼叫。
* [!DNL idSync]執行個體層級方法也已被取代，方法的[檔案](https://experienceleague.adobe.com/docs/audience-manager/user-guide/dil-api/dil-instance-methods.html?lang=zh-Hant)中已呼叫此方法。

## ID與客戶ID同步 {#id-syncing-with-a-customer-id}

在AAM中，您可以在電腦上將UUID （匿名不重複使用者ID）與客戶ID同步，以便您可以上傳有關該客戶的離線資料，並將其與其線上行為繫結，藉此深入瞭解您的客戶。 在過去，這是以兩種方式之一完成的：

* [!DNL idSync]執行個體層級方法
* [!DNL declaredId]中的[!DNL DIL.create]專案

如果您使用其中一個舊方法來與客戶ID同步，強烈建議您使用[!DNL setCustomerIDs]方法來更新為，此方法是ECID服務的一部分。 方法的[!DNL setCustomerIDs]檔案[中有更多有關](https://experienceleague.adobe.com/docs/id-service/using/id-service-api/methods/setcustomerids.html?lang=zh-Hant)的資訊。

**快速提示：**&#x200B;先前使用上述任一方法時，您參考了AAM [!UICONTROL Data Source]的[!UICONTROL Data Source] ID （亦即「DPID」）。 更新至[!DNL setCustomerIDs]時，您需要改用AAM [!UICONTROL Data Source]的&quot;[!UICONTROL Integration Code]&quot;。 它仍指向相同的[!UICONTROL Data Source]，但只是不同的識別碼。 這如下影片所示。

>[!VIDEO](https://video.tv.adobe.com/v/23873/?quality=12)

以下小節根據您的實作方法，列出更新至DIL 8.0的步驟和建議：

## 在Adobe Experience Platform標籤中更新至DIL 8.0 {#updating-to-dil-in-experience-platform-launch}

更新至DIL 8.0的基本步驟

1. 如果您使用的是8.0之前的DIL，在升級之前，請前往AAM擴充功能中的DIL設定，並記下您正在使用的任何進階選項（以便在後續步驟中使用）。
1. 將您的AAM擴充功能更新至8.0版或更新版本。
1. 確認您的Experience Cloud ID服務擴充功能為3.3.0版或更新版本。
1. 對於您8.0之前AAM擴充功能中或DIL自訂程式碼中的任何已棄用方法/元素（例如`disableIDSyncs`），請在ECID擴充功能中啟用ECID方法。

   1. (DIL) `disableDestinationPublishingIframe` -> (ECID) `disableIdSyncs`
   1. (DIL) `disableIDSyncs` -> (ECID) `disableIdSyncs`
   1. (DIL) `iframeAkamaiHTTPS` -> (ECID) `dSyncSSLUseAkamai`
   1. (DIL) `declaredId` -> (ECID) `setCustomerIDs`

1. 發佈變更。

>[!VIDEO](https://video.tv.adobe.com/v/23874/?quality=12)

## 在Adobe DTM中更新至DIL 8.0 {#updating-to-dil-in-adobe-dtm}

1. 將您的AAM工具更新至8.0版或更新版本。 此版本設定位於AAM工具的「一般」區段下。
1. 對於任何8.0之前AAM工具DIL自訂程式碼中已棄用的方法/元素（例如`disableIDSyncs`），請記下它們（以便將其新增至ECID工具），然後從AAM工具的自訂[!DNL DIL code]中移除它們。
1. 將您的Experience Cloud ID Service擴充功能更新至3.3.0版或更新版本
1. 新增進階選項至您從AAM工具自訂程式碼移除的ECID工具。
1. 發佈變更

## 正在更新至DIL 8.0，其中沒有Adobe Tag Management解決方案 {#additional-resources}

如果您直接在頁面上更新程式碼，則只能以較新的專案取代較舊的專案，除非您需要將方法/元素從DIL移動到ECID，如上所述。 在此情況下，您只需以ECID位置的ECID方法/元素，取代DIL位置的舊方法/元素即可。

非Adobe標籤管理員的情況也一樣。 只要您有該標籤管理解決方案中的舊版本，請依照以下步驟所述以新程式碼取代。

1. 將您的DIL程式庫更新至最新版本（8.0或更新版本） — 您需要從Adobe Consulting或Adobe客戶服務取得最新的DIL程式碼，因為目前無法在公共位置使用。 接著，只要以新的DIL程式庫程式碼取代舊的DIL程式庫程式碼，並繼續進行下一個步驟（請勿現在停止，否則您會遇到問題，ha）。
1. 安裝[!DNL ECID Service]或將現有版本更新至3.3.0或更新版本。 您可以從我們的GitHub頁面[下載最新的Experience Cloud ID服務版本](https://github.com/Adobe-Marketing-Cloud/id-service/releases)。 若您需要相關協助，請參閱[檔案](https://experienceleague.adobe.com/docs/id-service/using/home.html?lang=zh-Hant)或諮詢Adobe顧問。

1. 確認您的DIL自訂程式碼中的任何已棄用方法或元素已移至ECID方法：

   1. (DIL) `disableDestinationPublishingIframe` -> (ECID) `disableIdSyncs`

      [文件](https://experienceleague.adobe.com/docs/id-service/using/id-service-api/configurations/disableidsync.html?lang=zh-Hant)

   1. (DIL) `disableIDSyncs` -> (ECID) `disableIdSyncs`

      [文件](https://experienceleague.adobe.com/docs/id-service/using/id-service-api/configurations/disableidsync.html?lang=zh-Hant)

   1. (DIL) `iframeAkamaiHTTPS` -> (ECID) `idSyncSSLUseAkamai`

      [文件](https://experienceleague.adobe.com/docs/audience-manager/user-guide/dil-api/class-level-dil-methods/dil-create.html?lang=zh-Hant)

   1. (DIL) `declaredId` -> (ECID) `setCustomerIDs`

      [文件](https://experienceleague.adobe.com/docs/id-service/using/id-service-api/methods/setcustomerids.html?lang=zh-Hant)
