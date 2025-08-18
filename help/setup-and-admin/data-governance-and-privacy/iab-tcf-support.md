---
title: IAB TCF 2.2支援
description: 瞭解適用於IAB TCF的Audience Manager外掛程式，以及它如何與Adobe的選擇加入物件和您的同意管理提供者(CMP)搭配運作。
feature: Data Governance & Privacy
thumbnail: 26434.jpg
kt: 5027
role: Developer, Data Engineer, Architect
level: Experienced
exl-id: 04b4e786-0457-4dcc-bcf9-a79eda67bb2e
source-git-commit: f9708e705d95b43084ff11e342dc54ff11d6326c
workflow-type: tm+mt
source-wordcount: '1059'
ht-degree: 0%

---

# Audience Manager中的IAB TCF 2.2支援 {#iab-tcf-support-in-audience-manager}

Adobe可讓您透過選擇加入功能和IAB透明與同意架構2.2 (TCF 2.2)支援的Audience Manager外掛程式，管理使用者的隱私權選擇，並與使用者針對該選擇溝通。 本文會與檔案搭配使用，協助您瞭解IAB TCF的Audience Manager外掛程式，以及它如何與Adobe的選擇加入物件和您的同意管理提供者(CMP)搭配運作。 若要深入瞭解IAB，請參閱其網站： [https://www.iabeurope.eu/](https://www.iabeurope.eu/)。

## 第一步：瞭解Experience Cloud ID選擇加入 {#first-step-understand-ecid-s-opt-in}

若要瞭解如何使用IAB TCF，您必須先瞭解[!DNL Opt-in]功能，此功能屬於Experience Cloud ID服務(ECID)程式庫的一部分。 如果您不熟悉選擇加入的運作方式，請先參閱[這篇實用文章](https://experienceleague.adobe.com/docs/core-services-learn/tutorials/id-service/use-opt-in-to-control-experience-cloud-activities-based-on-user-consent.html?lang=zh-Hant)。 您也應該檢閱選擇加入[檔案](https://experienceleague.adobe.com/docs/id-service/using/implementation/opt-in-service/optin-overview.html?lang=zh-Hant)。 瀏覽完這些資源後，請返回此頁面並繼續。

## 適用於IAB TCF的Audience Manager外掛程式 {#the-audience-manager-plug-in-for-iab-tcf}

現在您已至少基本瞭解選擇加入服務的運作方式，Audience Manager可為其提供[!DNL IAB Transparency and Consent Framework (TCF)]支援，此支援是透過選擇加入物件的外掛程式完成的。

適用於IAB TCF的Audience Manager外掛程式擴充了「選擇加入」功能，可讓AAM客戶根據IAB TCF評估、尊重使用者隱私權選擇，並將其轉呈給下游合作夥伴。 它提供資料控管單位(即您身為Adobe客戶)和廠商（DMP、DSP、SSP、廣告伺服器等）可用來跨同意範圍瞭解同意的標準。

## 啟用IAB TCF {#enabling-iab-tcf}

如果您使用Adobe Experience Platform Launch，啟用適用IAB TCF的Audience Manager外掛程式相當容易，因為這是一個簡單的核取方塊，如下面的短片所示：

>[!VIDEO](https://video.tv.adobe.com/v/26433/?quality=12)

或者，如果您沒有使用Launch，可以在例項化Experience Cloud訪客時使用`isIabContext=true`啟用它。 這會起始IAB TCF流程，亦即新增另一個同意收集步驟，使用IAB TCF來查詢IAB TC字串，並將其提供回選擇加入，接著選擇加入再與Experience Cloud解決方案通訊。

## IAB TC字串 {#iab-tcf-consent-string}

IAB提供的其中一項標準是「同意字串」（也稱為「DaisyBit」），實際上是兩個清單彙整在一起：

1. 目的： **同意做什麼**？
1. 廠商：**同意給**&#x200B;誰？

### 用途 {#purposes}

使用IAB TCF 2.2時，有10種「目的」可收集同意（廠商可處理訪客的資料）。 Adobe Audience Manager不要求全部十項，而是除了廠商同意外，僅要求針對下列用途取得同意：

* **用途1：**&#x200B;儲存和/或存取裝置上的資訊；
* **用途10：**&#x200B;開發和改進產品；
* **特殊用途1：**&#x200B;確保安全性、防止欺詐和偵錯。

這是IAB TC字串的第一個部分，僅記錄為1和0，並指定是否核准該用途/活動。

>[!NOTE]
>
>根據IAB法規，特殊目的1 （確保安全性、防止欺詐和除錯）一律獲得同意，使用者無法反對。

### 廠商 {#vendors}

IAB TC字串的另一個部分是數百家廠商的長清單，如此一來，訪客就可以看到網站上具有標籤的適用廠商清單，並可選擇要使用的廠商。 廠商維持在清單中的地位。 例如，此清單中的Adobe Audience Manager廠商編號為565。 如果清單中該數字為「1」，則Audience Manager可從清單前端執行已核准目的。 如果AAM的點有「0」，則無法對資料執行任何動作。

**若要Audience Manager提供使用者介面，讓客戶使用IAB TCF來選擇這些用途和廠商，或是核准/不核准所有活動，您必須使用已在IAB TCF註冊的CMP合作夥伴，或建置支援IAB TCF並在IAB TCF註冊的CMP合作夥伴。**

## 選擇加入：在IAB和Adobe應用程式之間轉換 {#opt-in-translating-between-iab-and-adobe-solutions}

使用IAB TCF的優點之一，是上述標準用途可能讓一般使用者比Adobe解決方案清單更能瞭解他們要核准的內容。 一般使用者可能不知道「核准」Audience Manager或[!DNL Target]的意義，但「在裝置上儲存和/或存取資訊」或「開發和改善產品」可能更容易讓使用者理解和同意。

為了核准Audience Manager (即為了針對選擇加入以獲得「是」投票的AAM而翻譯IAB，目的1和10 （如前所列）必須獲得一般使用者的同意。 如果任一專案未核准，或交易者未核准，AAM將不會執行畫素觸發或設定Cookie。 同樣很高興知道的是，許多客戶只要選擇為一般使用者提供「完整或無」的UI，這當然會允許或禁止使用Audience Manager (及其他Experience Cloud解決方案)。

[檔案](https://experienceleague.adobe.com/docs/audience-manager/user-guide/overview/data-privacy/consent-management/aam-iab-plugin.html?lang=zh-Hant)中有一些關於IAB TCF流程的Audience Manager外掛程式如何套用至發佈者和廣告商使用案例的實用資訊。

## IAB：傳送下游同意 {#iab-sending-consent-downstream}

當使用適用於IAB TCF的Audience Manager外掛程式時，使用者的同意選擇也會傳送至存在於全球廠商清單中的合作夥伴的平台層級（第三方） ID同步，讓合作夥伴擁有使用者同意資訊，並可據以採取行動。 此資訊會以兩個變數傳送：

* gdpr = 1
* gdpr_consent = [編碼的同意字串]

請注意，如果使用者在IAB內容中，但未提供同意（或提供負面同意），則Audience Manager完全不會收集IAB TC字串，因此會捨棄呼叫。 因此，在該情況下……不會向下游傳遞同意。

## 示範 {#demo}

在以下影片中，瞭解ECID和解決方案的Cookie和信標如何受到IAB使用者選擇的影響。

>[!VIDEO](https://video.tv.adobe.com/v/26434/?quality=12)

如需有關適用於IAB TCF 2.2的Audience Manager外掛程式的詳細資訊，包括如何實作和測試、使用案例和工作流程，請參閱[檔案](https://experienceleague.adobe.com/docs/audience-manager/user-guide/overview/data-privacy/consent-management/aam-iab-plugin.html?lang=zh-Hant)。
