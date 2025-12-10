---
title: 從追蹤伺服器移轉至報表套裝層級的伺服器端轉送
description: 瞭解如何在報表套裝層級（而非追蹤伺服器層級）啟用Adobe Analytics資料的伺服器端轉送至Audience Manager。
product: audience manager
feature: Adobe Analytics Integration
topics: null
activity: implement
doc-type: technical video
team: Technical Marketing
kt: 1776
role: Developer
level: Intermediate
exl-id: 08b81e52-a28a-43e4-a284-df2460a43016
source-git-commit: d47848370e7bf7617f2b706041c911161a6479cd
workflow-type: tm+mt
source-wordcount: '582'
ht-degree: 0%

---

# 從追蹤伺服器移轉至報表套裝層級的伺服器端轉送 {#migrating-from-tracking-server-to-report-suite-level-server-side-forwarding}

本文和影片將說明如何在[!DNL Analytics]層級而非[!UICONTROL report suite]層級啟用[!UICONTROL tracking server]資料的伺服器端轉送至Audience Manager。

## 簡介 {#introduction}

如果您有Adobe Audience Manager和Adobe Analytics，則可以實作[!DNL Analytics]資料到Audience Manager的伺服器端轉送。 這表示與其讓您的頁面傳送兩個點選(一個至[!DNL Analytics]，一個至Audience Manager)，不如讓頁面將點選傳送至[!DNL Analytics]，[!DNL Analytics]會將該資料轉送至Audience Manager。

如果您已啟用且正在執行，且已在2017年10月前啟用/實作此專案，則您的伺服器端轉送可能會以必須由Adobe客戶服務或Adobe Consulting啟用的[!UICONTROL Tracking Server]為基礎。 自2017年10月起，您現在可以自行設定伺服器端轉送，並在報表套裝層級（每個報表套裝的轉送）進行。 這有許多好處，如下所述。

## [!UICONTROL Tracking server]轉寄 {#tracking-server-forwarding}

您的[!UICONTROL tracking server]是您傳送[!DNL Analytics]資料的位置，也是影像要求和Cookie寫入所在的網域。 應在DTM、[!DNL Experience Platform Launch]或[!DNL AppMeasurement.js]檔案中設定，且通常看起來會是這樣，您的網站或企業名稱會取代「mysite」：

`s.trackingServer = "mysite.sc.omtrdc.net";`

如果伺服器端轉送設定為在[!UICONTROL tracking server]層級轉送，則任何傳送至此[!UICONTROL tracking server]的點選(如果Experience Cloud ID服務也啟用)將會轉送至Audience Manager。 這必須由Adobe客戶服務或Adobe Consulting啟用。 在您切換至[!UICONTROL report suite]轉送後，他們也可以停用該功能，如下所述。

如果您不確定是否已為您啟用[!DNL tracking server forwarding]，請聯絡Adobe客戶服務或Adobe Consulting，他們應該能夠通知您。

## [!UICONTROL Report-suite]層級伺服器端轉送 {#report-suite-level-server-side-forwarding}

從[!UICONTROL report suite]轉送移至[!UICONTROL tracking server]轉送的最大優點之一，就是您現在可以使用「Audience Analytics」，也就是將Audience Manager [!UICONTROL segments]轉送回Adobe Analytics以進行詳細的區段分析。 如果您仍在[!UICONTROL tracking server]轉送而非[!UICONTROL report suite]轉送，則不支援這項絕佳功能。 請參閱[檔案](https://experienceleague.adobe.com/docs/analytics/integration/audience-analytics/mc-audiences-aam.html)中有關Audience Analytics的詳細資訊。

>[!VIDEO](https://video.tv.adobe.com/v/23701/?quality=12)

## 重要提示 {#additional-resources}

如上述影片所述，一旦您將所有[!UICONTROL report suites]設定為轉寄您要轉寄至Audience Manager，您應聯絡Adobe客戶服務或Adobe Consulting，讓他們停用[!UICONTROL tracking server]轉寄。 您不必急著這麼做，因為同時有[!UICONTROL tracking server]轉送和[!UICONTROL report suite]轉送不會產生重複的點選。 不過，最佳實務是隻開啟[!UICONTROL report suite]轉送。

若將[!UICONTROL tracking server]轉寄功能保留為開啟，不僅可能會轉寄您不想轉寄來自[!UICONTROL report suites]的資料，而且將來當您（以及貴公司的所有人員）忘記[!UICONTROL tracking server]轉寄功能已開啟後，您可能會認為資料並未轉寄給特定[!UICONTROL report suite]。 這是因為未在報表套裝層級開啟該功能，但由於[!UICONTROL tracking server]，資料仍會轉送。 然後，您將會浪費時間和金錢來查明它為何要轉送，以及為何要為您未曾預期的AAM伺服器呼叫付費。 因此，在您設定好所有[!UICONTROL tracking server]轉寄後，最好立即停用[!UICONTROL report suites]轉寄，以符合您的業務需求。
