---
title: 將資料傳送至SPA時，請在AAM頁面上使用最佳實務
description: 瞭解將資料從單頁應用程式(SPA)傳送至Adobe Audience Manager (AAM)的最佳實務。 本文章著重於使用Experience Platform標籤，此為建議的實作方法。
feature: Implementation Basics
topics: spa
activity: implement
doc-type: technical video
team: Technical Marketing
kt: 1390
topic: SPA
role: Developer, Data Engineer
level: Experienced
exl-id: 99ec723a-dd56-4355-a29f-bd6d2356b402
source-git-commit: d4874d9f6d7a36bb81ac183eb8b853d893822ae0
workflow-type: tm+mt
source-wordcount: '547'
ht-degree: 0%

---

# 將資料傳送至SPA時，請在AAM頁面上使用最佳實務 {#using-best-practices-on-spa-pages-when-sending-data-to-aam}

本檔案說明從單頁應用程式(SPA)傳送資料至Adobe Audience Manager (AAM)的幾種最佳作法。 本文著重於使用[!UICONTROL Experience Platform tags] （建議的實作方法）。

## 初始附註

* 以下專案假設您正使用Platform標籤在您的網站上實作。 如果您未使用Platform標籤，但可能需要根據您的實作方法調整，可能仍適用這些考量。
* 所有SPA並不相同，因此您可能需要調整部分以下專案，以最符合您的需求，但Adobe想要分享一些您從SPA頁面傳送資料至Audience Manager時需要考慮的最佳實務。

## 在Experience Platform標籤（前身為Launch）中使用SPA和AAM的簡圖{#simple-diagram-of-working-with-spas-and-aam-in-experience-platform-launch}

標籤中aam的![spa](assets/spa_for_aam_in_launch.png)

>[!NOTE]
>如前所述，這是如何在Adobe Audience Manager實施(不含Adobe Analytics)中使用Platform標籤處理SPA頁面的簡圖。 如您所見，這是相當直截了當的決定，其中最重大的決定是如何將檢視變更（或動作）傳達給Platform標籤。

## 從SPA頁面觸發標籤 {#triggering-launch-from-the-spa-page}

在Platform標籤中觸發規則(因而將資料傳送至Audience Manager)的兩個較常見方法為：

* 設定JavaScript自訂事件(使用Adobe Analytics參閱範例[這裡](https://helpx.adobe.com/analytics/kt/using/spa-analytics-best-practices-feature-video-use.html))
* 使用[!UICONTROL Direct Call Rule]

在此Audience Manager範例中，您會在Platform標籤中使用[!UICONTROL Direct Call rule]來觸發進入Audience Manager的點選。 您將在下一節中看到，將[!UICONTROL Data Layer]設定為新值可讓其Platform標籤中的[!UICONTROL Data Element]擷取，如此會很有用。

## 示範頁面 {#demo-page}

以下是一個小頁面，示範如何變更資料層中的值並將其傳送到Audience Manager中(您可能在SPA頁面上這麼做)。 此功能可以模型化，以因應更複雜的必要變更。 您可以在[這裡](https://aam.enablementadobe.com/SPA-Launch.html)找到此示範頁面。

## 設定資料層 {#setting-the-data-layer}

如前所述，當新內容載入頁面上，或有人在網站上執行動作時，必須在頁面標題中動態設定資料層，之後才會呼叫Platform標籤並執行[!UICONTROL rules]，這樣Platform標籤就能從資料層擷取新值，並將它們推入Audience Manager。

如果您前往上述示範網站並檢視頁面來源，您將會看到：

* 在呼叫Platform標籤之前，資料層位於頁面前端
* 模擬SPA連結中的JavaScript會變更[!UICONTROL Data Layer]，然後呼叫Platform標籤（`_satellite.track()`呼叫）。 如果您使用JavaScript自訂事件而非此[!UICONTROL Direct Call Rule]，則課程會相同。 請先變更[!DNL data layer]，然後呼叫Platform標籤。

>[!VIDEO](https://video.tv.adobe.com/v/23322/?quality=12)

## 其他資源 {#additional-resources}

* Adobe論壇上的[SPA討論](https://forums.adobe.com/thread/2451022)
* [參考架構網站，說明如何在Platform標籤中實作SPA](https://helpx.adobe.com/experience-manager/kt/integration/using/launch-reference-architecture-SPA-tutorial-implement.html)
* [在Adobe Analytics中追蹤SPA時使用最佳實務](https://helpx.adobe.com/analytics/kt/using/spa-analytics-best-practices-feature-video-use.html)
* [用於本文章的示範網站](https://aam.enablementadobe.com/SPA-Launch.html)
