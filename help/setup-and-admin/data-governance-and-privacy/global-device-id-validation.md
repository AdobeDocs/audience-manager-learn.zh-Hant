---
title: 全域裝置識別碼驗證
description: 瞭解如何驗證傳送至全域資料來源的裝置ID以使用正確格式，以及當ID格式錯誤時如何傳送錯誤訊息。
feature: Data Governance & Privacy
topics: mobile
activity: implement
doc-type: article
team: Technical Marketing
kt: 2977
role: Developer, Data Engineer, Architect
level: Experienced
exl-id: 0ff3f123-efb3-4124-bdf9-deac523ef8c9
source-git-commit: 2094d3bcf658913171afa848e4228653c71c41de
workflow-type: tm+mt
source-wordcount: '616'
ht-degree: 1%

---

# 全域裝置識別碼驗證 {#global-device-id-validation}

裝置Advertising識別碼（即iDFA、GAID、Roku ID）具有必須符合的格式標準，才能在數位廣告生態系統中使用。 現在，客戶和合作夥伴可以任何格式將ID上傳至我們的全球資料來源，而不需要通知ID的格式是否正確。 此功能將會驗證傳送至全域資料來源的裝置ID是否格式正確，並在ID格式錯誤時提供錯誤訊息。 啟動時，我們將支援[!DNL iDFA]、[!DNL Google Advertising]和[!DNL Roku IDs]的驗證。

## 格式標準概觀 {#overview-of-format-standards}

以下是AAM目前識別和支援的全域裝置Advertising ID集區。 這些皆實作為共用[!UICONTROL Data Sources]，可供任何與連結至這些平台使用者的資料搭配使用的客戶或資料合作夥伴使用。

<table>
  <tr>
   <td>平台 </td>
   <td>AAM Data Source ID </td>
   <td>ID格式 </td>
   <td>AAM PID </td>
   <td>附註 </td>
  </tr>
  <tr>
   <td>Google Android (GAID)</td>
   <td>20914</td>
   <td>32個十六進位數字，一般顯示為8-4-4-4-12<em>範例， 97987bca-ae59-4c7d-94ba-ee4f19ab8c21<br/> </em> </td>
   <td>1352</td>
   <td>此ID必須以原始/未雜湊/未變更的表單參考來收集 — <a href="https://play.google.com/about/monetization-ads/ads/ad-id/">https://play.google.com/about/monetization-ads/ads/ad-id/</a></td>
  </tr>
  <tr>
   <td>Apple iOS (IDFA)</td>
   <td>20915</td>
   <td>32個十六進位數字，一般顯示為8-4-4-4-12 <em>範例，6D92078A-8246-4BA4-AE5B-76104861E7DC<br /> </em> </td>
   <td>3560</td>
   <td>此ID必須以原始/未雜湊/未變更的表單參考來收集 — <a href="https://support.apple.com/en-us/HT205223">https://support.apple.com/en-us/HT205223</a></td>
  </tr>
  <tr>
   <td>Roku (RIDA)</td>
   <td>121963</td>
   <td>32個十六進位數字，一般顯示為8-4-4-4-12 <em>範例，</em> <em>fcb2a29c-315a-5e6b-bcfd-d889ba19aada</em></td>
   <td>11536</td>
   <td>此ID必須以原始/未雜湊/未變更的表單參考來收集 — <a href="https://sdkdocs.roku.com/display/sdkdoc/Roku+Advertising+Framework">https://sdkdocs.roku.com/display/sdkdoc/Roku+Advertising+Framework</a> </td>
  </tr>
  <tr>
   <td>Microsoft Advertising ID (MAID)</td>
   <td>389146</td>
   <td>Alpha數字字串</td>
   <td>14593</td>
   <td>此ID必須以原始/未雜湊/未變更的表單參考來收集 — <a href="https://docs.microsoft.com/en-us/uwp/api/windows.system.userprofile.advertisingmanager.advertisingid">https://docs.microsoft.com/en-us/uwp/api/windows.system.userprofile.advertisingmanager.advertisingid</a><br/><a href="https://msdn.microsoft.com/en-us/library/windows/apps/windows.system.userprofile.advertisingmanager.advertisingid.aspx">https://msdn.microsoft.com/en-us/library/windows/apps/windows.system.userprofile.advertisingmanager.advertisingid.aspx</a></td>
  </tr>
  <tr>
   <td>Samsung DUID</td>
   <td>404660</td>
   <td>Alpha數值字串範例，7XCBNROQJQPYW</td>
   <td>15950</td>
   <td>此ID必須以原始/未雜湊/未變更的表單參考來收集 — <a href="https://developer.samsung.com/tv/develop/api-references/samsung-product-api-references/productinfo-api">https://developer.samsung.com/tv/develop/api-references/samsung-product-api-references/productinfo-api</a> </td>
  </tr>
</table>

## 在應用程式中設定Advertising識別碼 {#setting-an-advertising-identifier-in-the-app}

在應用程式中設定廣告商ID其實是兩個步驟的程式，首先擷取廣告商ID，然後將其傳送至Experience Cloud。 執行上述步驟的連結如下所述。

1. 擷取ID
   1. 有關[!DNL advertising ID]的[!DNL Apple]資訊可在[此處](https://developer.apple.com/documentation/adsupport/asidentifiermanager)找到。
   1. 有關為[!DNL Android]開發人員設定[!DNL advertiser ID]的資訊，請參閱[這裡](http://android.cn-mirrors.com/google/play-services/id.html)。
1. 使用SDK中的[!DNL setAdvertisingIdentifier]方法將其傳送至Experience Cloud
   1. 使用`setAdvertisingIdentifier`的資訊位於[!DNL iOS]和[!DNL Android]的[檔案](https://aep-sdks.gitbook.io/docs/using-mobile-extensions/mobile-core/identity/identity-api-reference#set-an-advertising-identifier)中。

`// iOS (Swift) example for using setAdvertisingIdentifier:`
`ACPCore.setAdvertisingIdentifier([AdvertisingId]) // ...where [AdvertisingId] is replaced by the actual advertising ID`

## DCS錯誤訊息傳送不正確的ID  {#dcs-error-messaging-for-incorrect-ids}

當不正確的全域裝置ID （IDFA、GAID等）即時提交給Audience Manager時，點選時將傳回錯誤代碼。 以下為傳回錯誤的範例，因為識別碼是以[!DNL Apple IDFA]的形式傳送，其中只應包含大寫字母，但識別碼中卻有小寫的「x」。

![錯誤影像](assets/image_4_.png)

請參閱[檔案](https://experienceleague.adobe.com/docs/audience-manager/user-guide/api-and-sdk-code/dcs/dcs-api-reference/dcs-error-codes.html?lang=en#api-and-sdk-code)以取得錯誤代碼清單。

## 加入全球裝置ID {#onboarding-global-device-ids}

除了即時提交全域裝置ID之外，您還能依據ID &quot;[!DNL onboard]&quot; （上傳）資料。 此程式與您根據客戶ID （通常透過索引鍵/值配對）上線資料時相同，但您只需使用適當的資料Source ID，即可將資料指派給全域裝置ID。 您可以在[檔案](https://experienceleague.adobe.com/docs/audience-manager/user-guide/implementation-integration-guides/sending-audience-data/batch-data-transfer-process/batch-data-transfer-overview.html?lang=en#implementation-integration-guides)中找到有關上線流程的檔案。 請記得根據您使用的平台，使用全域資料來源ID。

如果透過上線程式提交不正確的全域裝置ID，則錯誤將顯示在[[!DNL Onboarding Status Report]](https://experienceleague.adobe.com/docs/audience-manager/user-guide/reporting/onboarding-status-report.html?lang=en#reporting)中。

以下為通過該報表發生的錯誤範例：

![錯誤影像](assets/image_5_.png)
