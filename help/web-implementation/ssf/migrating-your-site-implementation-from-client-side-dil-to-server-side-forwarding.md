---
title: 將網站的Audience Manager實作從使用者端DIL移轉至伺服器端轉送
description: 瞭解如何將網站的Audience Manager(AAM)實作從使用者端DIL移轉至伺服器端轉送。 如果您同時擁有AAM和Adobe Analytics，而且您使用DIL(Data Integration Library)代碼將頁面上的點選傳送至AAM，也已將頁面上的點選傳送至Adobe Analytics，則本教學課程適用。
product: audience manager
feature: Adobe Analytics Integration
topics: null
activity: implement
doc-type: tutorial
team: Technical Marketing
kt: 1778
role: Developer, Data Engineer
level: Intermediate
exl-id: bcb968fb-4290-4f10-b1bb-e9f41f182115
source-git-commit: 2094d3bcf658913171afa848e4228653c71c41de
workflow-type: tm+mt
source-wordcount: '2333'
ht-degree: 0%

---

# 將網站的Audience Manager實作從使用者端DIL移轉至伺服器端轉送 {#migrating-your-site-s-aam-implementation-from-client-side-dil-to-server-side-forwarding}

如果您同時擁有Adobe Audience Manager (AAM)和Adobe Analytics，而且您目前正在使用DIL([!DNL Data Integration Library])程式碼從頁面傳送點選至AAM，以及從頁面傳送點選至Adobe Analytics，則此教學課程適用於您。 由於您擁有這兩個解決方案，而且它們都是Adobe Experience Cloud的一部分，因此您有機會遵循開啟伺服器端轉送的最佳做法，這可讓[!DNL Analytics]資料收集伺服器即時轉送網站分析資料至Audience Manager，而不是讓使用者端程式碼從頁面傳送額外點選至AAM。 本教學課程將逐步帶您瞭解從舊版使用者端DIL實作切換至新版伺服器端轉送方法的步驟。

## 使用者端(DIL)與伺服器端 {#client-side-dil-vs-server-side}

比較和比較這兩種將Adobe Analytics資料匯入AAM的方法時，先將差異以視覺效果顯示在下列影像中可能有所助益：

![使用者端到伺服器端](assets/client-side_vs_server-side_aam_implementation.png)

### 使用者端DIL實施 {#client-side-dil-implementation}

如果您使用此方法將Adobe Analytics資料匯入AAM，您會有兩個點選來自您的網頁：一個會前往[!DNL Analytics]，另一個會前往AAM （在網頁上複製[!DNL Analytics]資料後）。 [!UICONTROL Segments]會從AAM傳回至頁面，以便用於個人化等。 這將視為舊版實作，不再建議使用。

除了這並非遵循最佳實務之外，使用此方法的缺點包括：

* 來自頁面的兩個點選，而非只有一個
* 需要伺服器端轉送才能將AAM對象即時分享到[!DNL Analytics]，因此使用者端實作不允許此功能（以及未來可能的其他功能）

建議您改用AAM實作的伺服器端轉送方法。

### 伺服器端轉送實作 {#server-side-forwarding-implementation}

如上圖所示，點選來自指向Adobe Analytics的網頁。 [!DNL Analytics]然後即時將該資料轉送到AAM，系統會將訪客評估為AAM特徵和[!UICONTROL segments]，就像點選直接來自頁面一樣。

[!UICONTROL Segments]會在相同的即時點選上傳回[!DNL Analytics]，這會將其上的回應轉送至網頁以進行個人化等。

移至伺服器端轉送沒有時間上的問題。 Adobe強烈建議同時擁有Audience Manager和[!DNL Analytics]的任何人使用此實作方法。

## 您有兩個主要任務 {#you-have-two-main-tasks}

本頁有不少資訊，當然也很重要。 不過，它&#x200B;**可歸結為您需要做的兩項主要工作**：

1. 將您的程式碼從使用者端DIL程式碼變更為伺服器端轉送程式碼
1. 在[!DNL Analytics] [!DNL Admin Console]中反向切換以開始實際轉送資料（根據[!UICONTROL report suite]）

如果您略過其中一項工作，伺服器端轉送將無法正常運作。 已新增步驟和其他資料至本檔案，協助您在設定中正確執行這兩個步驟。

## 實作選項 {#implementation-options}

當您從使用者端轉送移至伺服器端轉送時，其中一個工作是將程式碼變更為新的伺服器端轉送程式碼。 使用下列其中一個選項來完成：

* Adobe Experience Platform標籤 — Adobe建議的Web屬性實作選項。 您會發現這是一項輕鬆的工作，因為Platform標籤已為您完成所有艱難的工作。
* 在頁面上 — 您也可以將新的SSF程式碼直接放入`appMeasurement.js`檔案內的`doPlugins`函式中(如果尚未使用Adobe啟動)
* 其他標籤管理員 — 這些處理方式與上一個（在頁面上）選項相同，因為無論其他標籤管理員儲存[!DNL AppMeasurement]程式碼，您仍會將SSF程式碼放入`doPlugins`

我們將在&#x200B;_更新程式碼_&#x200B;區段中逐一檢視這些專案。

## 實作步驟 {#implementation-steps}

下列步驟說明實作。

### 步驟0：先決條件：Experience CloudID服務(ECID) {#step-prerequisite-experience-cloud-id-service-ecid}

移至伺服器端轉送的主要先決條件為實作Experience CloudID服務。 如果您使用Experience Platform Launch，這可輕鬆完成這項工作，此時您只需安裝ECID擴充功能，其他動作就會執行。

如果您使用非AdobeTMS或完全沒有TMS，請實作ECID以執行&#x200B;**之前**&#x200B;任何其他Adobe解決方案。 如需詳細資訊，請參閱[ECID檔案](https://experienceleague.adobe.com/docs/id-service/using/home.html?lang=zh-Hant)。 唯一的其他先決條件與程式碼版本有關，因此您只需在下列步驟中套用程式碼的最新版本，就不會有問題。

>[!NOTE]
>
>實作前請先閱讀本整份檔案。 下面的「計時」區段有有關&#x200B;*當*&#x200B;您應該實作每個專案的重要資訊，包括ECID （如果尚未實作）。

### 步驟1：從DIL程式碼記錄目前使用的選項 {#step-record-currently-used-options-from-dil-code}

當您準備好從使用者端DIL代碼移至伺服器端轉送時，第一步是識別您使用DIL代碼執行的所有操作，包括自訂設定和傳送至AAM的資料。 需注意及考慮的事項包括：

* 一般[!DNL Analytics]變數，使用`siteCatalyst.init`DIL模組 — 您不需要擔心此變數，因為其工作只是傳送一般[!DNL Analytics]變數，只要啟用伺服器端轉送即可完成。
* 夥伴子網域 — 在`DIL.create`函式中，記下`partner`引數。 這稱為您的「合作夥伴子網域」，或有時稱為「合作夥伴ID」，當您放置新的伺服器端轉送程式碼時，將需使用此專案。
* [!DNL Visitor Service Namespace] — 也稱為您的&quot;[!DNL Org ID]&quot;或&quot;[!DNL IMS Org ID]&quot;，當您設定新的伺服器端轉送程式碼時，也會需要此專案。 記下它。
* containerNSID、uuidCookie和其他進階選項 — 記下您正在使用的其他進階選項，以便在伺服器端轉送程式碼中設定它們。
* 其他頁面變數 — 如果從頁面將其他變數傳送到AAM （除了siteCatalyst.init處理的一般[!DNL Analytics]變數之外），您將需要記下這些變數，以便透過伺服器端轉送來傳送它們（破壞程式警示：透過[!DNL contextData]變數）。

### 步驟2：更新程式碼 {#step-updating-the-code}

在[實作選項](#implementation-options) （以上）中，會針對您實作伺服器端轉送的方式和位置提供多個選項。 為了讓此區段生效，我們需要將其分成這些區段（其中兩個區段合併在一起）。 移至本節最能描述您需求的方法。

#### Adobe Experience Platform標籤 {#launch-by-adobe}

觀看以下影片，瞭解如何在Experience Platform Launch中將實作選項從使用者端DIL程式碼移至伺服器端轉送。

>[!VIDEO](https://video.tv.adobe.com/v/26310/?quality=12)

#### 「在頁面上」或非Adobe標籤管理員 {#on-the-page-or-non-adobe-tag-manager}

觀看以下影片，瞭解如何將實作選項從使用者端DIL程式碼移至伺服器端轉送(使用[!DNL AppMeasurement]程式碼，位於檔案或非Adobe標籤管理系統中)。

>[!VIDEO](https://video.tv.adobe.com/v/26312/?quality=12)

### 步驟3：啟用轉送（根據[!UICONTROL Report Suite]） {#step-enabling-the-forwarding-per-report-suite}

在本教學課程中，我們目前花所有時間將程式碼從使用者端DIL程式碼切換至伺服器端轉送。 沒關係，因為這是比較困難的部分。 雖然您會看到本節非常簡單，但與更新程式碼一樣重要。 在本影片中，您將會瞭解如何切換開關，以實際將資料從Analytics轉送至Audience Manager。

>[!VIDEO](https://video.tv.adobe.com/v/26355/?quality-12)

**注意：**&#x200B;如影片中所述，最多需要4小時才能在Experience Cloud後端完全實作轉送。

## 時間 {#timing}

提醒您，從使用者端DIL移至伺服器端轉送有兩個主要工作：

1. 更新程式碼
1. 在[!DNL Analytics] [!DNL Admin Console]中反向切換

但問題是，您首先要做什麼？ 這重要嗎？ 好的，很抱歉，有兩個問題。 但答案是……視情況而定，是的，它&#x200B;*可以*&#x200B;很重要。 這表示模糊嗎？ 讓我們來加以劃分。 但首先，如果您是擁有許多網站的大型組織，會提出另一個問題：我是否必須一次完成所有工作？ 那個比較簡單。 沒有。 您可以逐一執行。

### 再深入一點 {#a-little-deeper-dive}

時間與順序之所以重要，是因為轉送&#x200B;_實際上_&#x200B;的運作方式，其摘要可歸納為下列幾個技術事實：

* 如果您已實作Experience CloudID服務(ECID)，而且[!DNL Analytics] [!DNL Admin Console]中的開關（「開關」）已開啟，即使您尚未更新程式碼，資料仍會從[!DNL Analytics]轉送至AAM。
* 如果您尚未實作ECID，則不會轉送資料，即使您已開啟開關，且伺服器端轉送程式碼亦然。
* 伺服器端轉送程式碼（無論在Platform標籤中或頁面上）會真正處理回應，且是完成移轉的必要專案。
* 請記住，伺服器端轉送交換器是由[!UICONTROL report suite]啟用，但程式碼是由Platform標籤中的屬性處理，或是由[!DNL AppMeasurement]檔案處理（如果您未使用Platform標籤）。

### 最佳實務 {#best-practices}

根據這些技術細節，以下提供何時該做什麼的建議：

#### 如果您尚未實施ECID {#if-you-do-not-have-ecid-yet-implemented}

1. 在[!DNL Analytics]中，針對您為伺服器端轉送啟用的每個[!UICONTROL report suite]翻轉開關。

   1. 因為您沒有ECID，所以轉送尚未開始。

1. 針對每個網站，將您的程式碼從使用者端DIL更新至伺服器端轉送（這可能位於Platform標籤中）或頁面上，如上方另一節所述。

   1. 轉送現在會流程（因為您已新增ECID），且您也應該會收到對[!DNL Analytics]信標的正確JSON回應（請參閱下方的「驗證和疑難排解」一節，以取得詳細資訊）。

#### 如果您已實施ECID {#if-you-do-have-ecid-implemented}

1. 準備並規劃，好讓您可以根據[!UICONTROL report suite]將程式碼從DIL更新至伺服器端轉送，以便啟用伺服器端轉送：

   1. 在[!DNL Analytics]中翻轉交換器以啟用伺服器端轉送。

      1. 因為您已啟用ECID，轉送功能將會啟動。

   1. 請儘快將您的程式碼從使用者端DIL更新為單一端轉送（這可以在Platform標籤中或頁面上，如上方另一節所述）。

      1. 您應該會收到[!DNL Analytics]信標的正確JSON回應（如需詳細資訊，請參閱下方的[驗證和疑難排解](#validation-and-troubleshooting)區段）。

>[!NOTE]
>
>這兩個步驟務必要儘可能靠近，因為在上述步驟1和2之間，您會發現進入AAM的資料重複。 換言之，單端轉送已開始從[!DNL Analytics]傳送資料至AAM，而且由於DIL程式碼仍在頁面上，因此也會有直接從頁面進入AAM的點選，因此資料會加倍。 一旦您將程式碼從DIL更新到伺服器端轉送，就會緩解此問題。

>[!NOTE]
>
>若您想讓資料有小差異，而不是資料有小重複，您可以切換上述步驟1和2的順序。 將程式碼從DIL移至伺服器端轉送會停止資料流入AAM，直到您能夠翻轉交換器以開啟[!UICONTROL report suite]的伺服器端轉送為止。 客戶通常寧願將資料翻一番，也不願錯過將訪客帶入特徵和[!UICONTROL segments]。

#### 有許多網站和[!UICONTROL report suites]時的移轉時間 {#migration-timing-when-you-have-many-sites-and-report-suites}

本主題在前幾節中會簡單介紹，主要策略可歸納如下：

一次移轉一個網站/[!UICONTROL report suite] （或一組網站/[!UICONTROL report suites]）。

不過，根據幾種可能的情況，這可能會有點棘手：

* 您有一個包含數個不同[!UICONTROL report suites]的網站
* 您有一個包含數個網站的[!UICONTROL report suite] （例如全域[!UICONTROL report suite]）
* 您使用一個Platform標籤屬性來涵蓋多個網站
* 您擁有不同網站的不同開發團隊

因為這些專案，事情可能會變得有點複雜。 最好的建議是：

* 請花點時間，根據先前說明的內容，制定移轉至伺服器端轉送的策略
* 根據Platform標籤中的單一屬性（或單一[!DNL AppMeasurement]檔案）通常對應至1或2個相異[!UICONTROL report suites]的事實，您可能會制定一個計畫，逐個處理這些相異群組，將您的企業更新至伺服器端轉送
* 如果您使用Adobe Consulting，請和他們討論您的移轉計畫，讓他們視需要提供協助

## 驗證和疑難排解 {#validation-and-troubleshooting}

驗證伺服器端轉送是否正常運作的主要方法是，檢視來自應用程式的任何Adobe Analytics點選回應。

如果您沒有執行從[!DNL Analytics]到Audience Manager的資料伺服器端轉送，則不會有任何對[!DNL Analytics]信標的回應（除2x2畫素以外）。 不過，如果您正在執行伺服器端轉送，[!DNL Analytics]要求與回應中就會有可驗證的專案，讓您知道[!DNL Analytics]正在與Audience Manager正確通訊、轉送點選及取得回應。

>[!VIDEO](https://video.tv.adobe.com/v/26359/?quality=12)

>[!WARNING]
>
>請注意誤報的「Success」訊息。 如果有回應，且一切似乎運作正常，請確定您有`stuff`物件在回應中。 如果沒有該訊息，您可能會看到顯示`"status":"SUCCESS"`的訊息。 雖然聽起來不合理，但這實際上是運作「不」正常的證明。
>
>如果您看到這個訊息，表示您已完成Platform標籤或[!DNL AppMeasurement]中的程式碼更新，但[!DNL Analytics] [!DNL Admin Console]中的轉送尚未完成。 在此情況下，您需要確認已在[!DNL Analytics] [!DNL Admin Console]中為您的[!UICONTROL report suite]啟用伺服器端轉送。 如果您已啟用且尚未經過4個小時，請耐心等候，因為在後端進行所有必要的變更可能需要花很長的時間。


![false成功](assets/falsesuccess.png)

如需伺服器端轉送的詳細資訊，請參閱[檔案](https://experienceleague.adobe.com/docs/analytics/admin/admin-tools/server-side-forwarding/ssf.html?lang=zh-Hant)。
