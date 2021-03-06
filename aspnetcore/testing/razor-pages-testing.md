---
title: Razor 頁面單位和整合測試中 ASP.NET Core
author: guardrex
description: 了解如何建立 Razor 網頁應用程式的單元和整合測試。
manager: wpickett
ms.author: riande
ms.custom: mvc
ms.date: 11/27/2017
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: testing/razor-pages-testing
ms.openlocfilehash: dc5e8651f873b8e86aaa8fdf2527e461bb065424
ms.sourcegitcommit: 48beecfe749ddac52bc79aa3eb246a2dcdaa1862
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/22/2018
---
# <a name="razor-pages-unit-and-integration-tests-in-aspnet-core"></a>Razor 頁面單位和整合測試中 ASP.NET Core

作者：[Luke Latham](https://github.com/guardrex)

ASP.NET Core 支援單位和整合測試的 Razor 網頁應用程式。 測試資料存取層 (DAL)、 頁面模型和整合式的網頁元件，可協助確保：

* Razor 網頁應用程式的組件的應用程式建構期間運作彼此獨立，而且一起當做一個單位。
* 類別和方法有限範圍的責任。
* 其他文件存在於應用程式的行為方式。
* 迴歸，也就是所更新的程式碼的錯誤，會在自動化的建置和部署中找到。

本主題假設您有基本了解 Razor 網頁應用程式、 單元測試，以及整合測試。 如果您熟悉 Razor 網頁應用程式或測試概念，請參閱下列主題：

* [Razor 頁面簡介](xref:mvc/razor-pages/index)
* [開始使用 Razor Pages](xref:tutorials/razor-pages/razor-pages-start)
* [單元測試 C# 中使用 dotnet 測試和 xUnit.NET Core](/dotnet/articles/core/testing/unit-testing-with-dotnet-test)
* [整合測試](xref:testing/integration-testing)

[檢視或下載範例程式碼](https://github.com/aspnet/Docs/tree/master/aspnetcore/testing/razor-pages-testing/sample/) \(英文\) ([如何下載](xref:tutorials/index#how-to-download-a-sample))

這個範例專案是由兩個應用程式所組成：

| 應用程式         | 專案資料夾                        | 描述 |
| ----------- | ------------------------------------- | ----------- |
| 訊息應用程式 | *src/RazorPagesTestingSample*         | 允許使用者加入、 刪除其中一個，刪除所有項目，以及分析訊息。 |
| 測試應用程式    | *tests/RazorPagesTestingSample.Tests* | 用來測試訊息的應用程式。<ul><li>單元測試： 資料存取層 (DAL)，索引頁面模型</li><li>整合測試： 索引頁</li></ul> |

可以使用內建的測試功能的 IDE，例如執行測試[Visual Studio](https://www.visualstudio.com/vs/)。 如果使用[Visual Studio Code](https://code.visualstudio.com/)或命令列中，執行下列命令，在命令提示字元中*tests/RazorPagesTestingSample.Tests*資料夾：

```console
dotnet test
```

## <a name="message-app-organization"></a>訊息應用程式的組織

訊息應用程式是簡單的 Razor 頁面訊息系統具有下列特性：

* 索引頁的應用程式 (*Pages/Index.cshtml*和*Pages/Index.cshtml.cs*) 提供 UI 和頁面來控制的新增、 刪除和訊息 （每個訊息的平均字） 的分析模型方法.
* 所描述的訊息`Message`類別 (*Data/Message.cs*) 使用兩個屬性： `Id` （金鑰） 和`Text`（訊息）。 `Text`屬性所需，且限制為 200 個字元。
* 訊息會儲存使用[Entity Framework 的記憶體中資料庫](/ef/core/providers/in-memory/)&#8224;。
* 應用程式在其資料庫內容類別，包含資料存取層 (DAL) `AppDbContext` (*Data/AppDbContext.cs*)。 DAL 方法標示為`virtual`，可讓模擬測試中使用的方法。
* 如果資料庫是空的應用程式啟動時，訊息存放區會使用三個訊息中初始化。 這些*植入訊息*也會用於測試。

&#8224;EF 主題[測試與 InMemory](/ef/core/miscellaneous/testing/in-memory)，說明如何使用記憶體中的資料庫使用 MSTest 進行測試。 本主題使用[xUnit](https://xunit.github.io/)測試架構。 測試概念與測試實作跨不同測試架構很類似的但不是完全相同。

雖然此應用程式不會使用[儲存機制模式](http://martinfowler.com/eaaCatalog/repository.html)並不是有效的範例[工作單位 (UoW) 模式](https://martinfowler.com/eaaCatalog/unitOfWork.html)，Razor 頁面支援這些模式開發。 如需詳細資訊，請參閱[設計基礎結構的持續性層級](/dotnet/standard/microservices-architecture/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design)， [ASP.NET MVC 應用程式中實作的儲存機制和工作單元模式](/aspnet/mvc/overview/older-versions/getting-started-with-ef-5-using-mvc-4/implementing-the-repository-and-unit-of-work-patterns-in-an-asp-net-mvc-application)，和[測試控制器邏輯](/aspnet/core/mvc/controllers/testing)（此範例會實作儲存機制模式）。

## <a name="test-app-organization"></a>測試應用程式的組織

測試應用程式是主控台應用程式內*tests/RazorPagesTestingSample.Tests*資料夾：

| 測試應用程式的資料夾    | 描述 |
| ------------------ | ----------- |
| *IntegrationTests* | <ul><li>*IndexPageTest.cs*包含索引頁的整合測試。</li><li>*TestFixture.cs*建立測試訊息的應用程式的測試主機。</li></ul> |
| *UnitTests*        | <ul><li>*DataAccessLayerTest.cs* DAL 包含單元測試。</li><li>*IndexPageTest.cs*包含單元測試，索引頁面模型。</li></ul> |
| *公用程式*        | *Utilities.cs*包含:<ul><li>`TestingDbContextOptions` 用來建立新的資料庫內容的每個 DAL 單元測試的選項，使資料庫重設為其基準條件，針對每個測試方法。</li><li>`GetRequestContentAsync` 方法可用來準備`HttpClient`和整合測試期間傳送至訊息的應用程式要求的內容。</li></ul>

測試架構是[xUnit](https://xunit.github.io/)。 模擬架構的物件是[Moq](https://github.com/moq/moq4)。 搜尋整合測試都會使用[ASP.NET Core 測試主機](xref:testing/integration-testing#the-test-host)。

## <a name="unit-testing-the-data-access-layer-dal"></a>單元測試資料存取層 (DAL)

訊息應用程式具有四種方法中所包含的 DAL`AppDbContext`類別 (*src/RazorPagesTestingSample/Data/AppDbContext.cs*)。 每個方法都有一個或兩個單元測試中測試應用程式。

| DAL 方法               | 功能                                                                   |
| ------------------------ | -------------------------------------------------------------------------- |
| `GetMessagesAsync`       | 取得`List<Message>`從依照資料庫`Text`屬性。 |
| `AddMessageAsync`        | 新增`Message`到資料庫。                                          |
| `DeleteAllMessagesAsync` | 刪除所有`Message`資料庫的項目。                           |
| `DeleteMessageAsync`     | 刪除單一`Message`從資料庫`Id`。                      |

DAL 的單元測試需要[DbContextOptions](/dotnet/api/microsoft.entityframeworkcore.dbcontextoptions)時建立新`AppDbContext`針對每個測試。 若要建立的其中一個方法`DbContextOptions`每項測試是使用[DbContextOptionsBuilder](/dotnet/api/microsoft.entityframeworkcore.dbcontextoptionsbuilder):

```csharp
var optionsBuilder = new DbContextOptionsBuilder<AppDbContext>()
    .UseInMemoryDatabase("InMemoryDb");

using (var db = new AppDbContext(optionsBuilder.Options))
{
    // Use the db here in the unit test.
}
```

這個方法的問題是，每項測試會接收資料庫中任何狀態先前測試所保留。 嘗試寫入，不會干擾彼此不可部分完成的單元測試時，這可能會造成問題。 若要強制`AppDbContext`若要使用新的資料庫內容，針對每個測試，提供`DbContextOptions`根據新的服務提供者的執行個體。 測試應用程式會顯示如何執行此動作使用其`Utilities`類別方法`TestingDbContextOptions`(*tests/RazorPagesTestingSample.Tests/Utilities/Utilities.cs*):

[!code-csharp[](razor-pages-testing/sample/tests/RazorPagesTestingSample.Tests/Utilities/Utilities.cs?name=snippet1)]

使用`DbContextOptions`DAL 單元測試可讓與全新資料庫的執行個體以不可分割方式執行每個測試：

```csharp
using (var db = new AppDbContext(Utilities.TestingDbContextOptions()))
{
    // Use the db here in the unit test.
}
```

在每個測試方法`DataAccessLayerTest`類別 (*UnitTests/DataAccessLayerTest.cs*) 遵循類似的排列 Act Assert 模式：

1. 排列： 將資料庫設定測試和/或定義獲得預期的結果。
1. Act： 執行測試。
1. 判斷提示： 若要判斷是否成功的測試結果進行判斷提示。

例如，`DeleteMessageAsync`方法負責移除單一訊息，由其`Id`(*src/RazorPagesTestingSample/Data/AppDbContext.cs*):

[!code-csharp[](razor-pages-testing/sample/src/RazorPagesTestingSample/Data/AppDbContext.cs?name=snippet4)]

有兩項測試，此方法。 一項測試會檢查方法在資料庫中有訊息時，會刪除訊息。 如果，不會變更資料庫的其他方法測試訊息`Id`刪除不存在。 `DeleteMessageAsync_MessageIsDeleted_WhenMessageIsFound`方法如下所示：

[!code-csharp[](razor-pages-testing/sample_snapshot/tests/RazorPagesTestingSample.Tests/UnitTests/DataAccessLayerTest.cs?name=snippet1)]

首先，此方法會執行排列步驟中，準備進行的動作步驟執行所在。 植入訊息會取得並保存在`seedMessages`。 植入訊息儲存到資料庫中。 與訊息`Id`的`1`設定為刪除。 當`DeleteMessageAsync`執行方法、 預期的郵件應具有所有除了有訊息`Id`的`1`。 `expectedMessages`變數代表這個預期的結果。

[!code-csharp[](razor-pages-testing/sample/tests/RazorPagesTestingSample.Tests/UnitTests/DataAccessLayerTest.cs?name=snippet1)]

方法會：`DeleteMessageAsync`執行方法中傳遞`recId`的`1`:

[!code-csharp[](razor-pages-testing/sample/tests/RazorPagesTestingSample.Tests/UnitTests/DataAccessLayerTest.cs?name=snippet2)]

最後，方法會取得`Messages`從內容，並比較它`expectedMessages`判斷提示兩個相等：

[!code-csharp[](razor-pages-testing/sample/tests/RazorPagesTestingSample.Tests/UnitTests/DataAccessLayerTest.cs?name=snippet3)]

若要比較的兩個`List<Message>`相同：

* 訊息都會按照`Id`。
* 訊息配對會比較上`Text`屬性。

類似的測試方法，`DeleteMessageAsync_NoMessageIsDeleted_WhenMessageIsNotFound`檢查結果的嘗試刪除不存在的訊息。 在此情況下，在資料庫中的預期的訊息應該會等於真正的訊息之後`DeleteMessageAsync`執行方法。 應該不會變更資料庫的內容：

[!code-csharp[](razor-pages-testing/sample/tests/RazorPagesTestingSample.Tests/UnitTests/DataAccessLayerTest.cs?name=snippet4)]

## <a name="unit-testing-the-page-model-methods"></a>單元測試的頁面模型方法

單元測試的另一組負責測試頁面模型方法。 在中找到訊息應用程式時，索引頁面模型`IndexModel`類別*src/RazorPagesTestingSample/Pages/Index.cshtml.cs*。

| 頁面模型方法 | 功能 |
| ----------------- | -------- | 
| `OnGetAsync` | 取得訊息的 UI 使用 DAL`GetMessagesAsync`方法。 |
| `OnPostAddMessageAsync` | 如果`ModelState`有效，但呼叫`AddMessageAsync`將訊息新增至資料庫。 | 
| `OnPostDeleteAllMessagesAsync` | 呼叫`DeleteAllMessagesAsync`刪除所有資料庫中的訊息。 |
| `OnPostDeleteMessageAsync` | 執行`DeleteMessageAsync`若要刪除的訊息`Id`指定。 |
| `OnPostAnalyzeMessagesAsync` | 如果一或多個訊息是在資料庫中，會計算平均每個訊息的字組數目。 |

使用七個測試中的測試網頁模型方法`IndexPageTest`類別 (*tests/RazorPagesTestingSample.Tests/UnitTests/IndexPageTest.cs*)。 測試可以使用熟悉的排列判斷提示動作模式。 這些測試著重於：

* 判斷方法是否遵循正確的行為時`ModelState`無效。
* 確認的方法產生正確`IActionResult`。
* 檢查屬性值的設定正確進行。

此群組的測試通常會模擬 DAL 以產生預期的資料頁面模型方法執行所在的動作步驟的方法。 例如，`GetMessagesAsync`方法`AppDbContext`模仿產生輸出。 當頁面模型方法執行這個方法時，模擬便會傳回結果。 資料並不是來自資料庫。 這會建立在頁面模型測試中使用 DAL 的可預測、 可靠的測試條件。

`OnGetAsync_PopulatesThePageModel_WithAListOfMessages`測試顯示如何`GetMessagesAsync`方法會模仿頁面模型：

[!code-csharp[](razor-pages-testing/sample/tests/RazorPagesTestingSample.Tests/UnitTests/IndexPageTest.cs?name=snippet1&highlight=3-4)]

當`OnGetAsync`方法執行的動作步驟中，它會呼叫頁面模型`GetMessagesAsync`方法。

單元測試的動作步驟 (*tests/RazorPagesTestingSample.Tests/UnitTests/IndexPageTest.cs*):

[!code-csharp[](razor-pages-testing/sample/tests/RazorPagesTestingSample.Tests/UnitTests/IndexPageTest.cs?name=snippet2)]

`IndexPage` 頁面模型`OnGetAsync`方法 (*src/RazorPagesTestingSample/Pages/Index.cshtml.cs*):

[!code-csharp[](razor-pages-testing/sample/src/RazorPagesTestingSample/Pages/Index.cshtml.cs?name=snippet1&highlight=3)]

`GetMessagesAsync` DAL 中的方法不會傳回這個方法呼叫的結果。 方法的模擬的版本傳回的結果。

在`Assert`步驟，實際的訊息 (`actualMessages`) 從指派的`Messages`頁面模型的屬性。 指派給訊息也執行類型檢查。 預期和實際的訊息會比較其`Text`屬性。 測試判斷提示，這兩個`List<Message>`執行個體包含相同的訊息。

[!code-csharp[](razor-pages-testing/sample/tests/RazorPagesTestingSample.Tests/UnitTests/IndexPageTest.cs?name=snippet3)]

此群組中的其他測試建立頁面包含模型物件`DefaultHttpContext`、 `ModelStateDictionary`、`ActionContext`建立`PageContext`、 `ViewDataDictionary`，和`PageContext`。 這些是適用於執行測試。 比方說，訊息應用程式會建立`ModelState`錯誤`AddModelError`檢查有效`PageResult`時，會傳回`OnPostAddMessageAsync`執行：

[!code-csharp[](razor-pages-testing/sample/tests/RazorPagesTestingSample.Tests/UnitTests/IndexPageTest.cs?name=snippet4&highlight=11,26,29,32)]

## <a name="integration-testing-the-app"></a>測試應用程式整合

整合測試上測試應用程式的元件一起運作的焦點。 搜尋整合測試都會使用[ASP.NET Core 測試主機](xref:testing/integration-testing#the-test-host)。 這被測試完整的要求-回應生命週期的處理。 這些測試判斷提示的頁面會產生正確的狀態碼和`Location`標頭，如果設定。

整合測試的範例的範例會檢查要求的訊息應用程式的索引頁面的結果 (*tests/RazorPagesTestingSample.Tests/IntegrationTests/IndexPageTest.cs*):

[!code-csharp[](razor-pages-testing/sample/tests/RazorPagesTestingSample.Tests/IntegrationTests/IndexPageTest.cs?name=snippet1)]

沒有任何排列步驟。 `GetAsync`上呼叫方法`HttpClient`GET 要求傳送至端點。 測試判斷提示，結果會是 200 OK 狀態碼。

訊息應用程式的所有 POST 要求都必須都滿足的應用程式會自動成為 antiforgery 核取[資料保護 antiforgery 系統](xref:security/data-protection/introduction)。 若要排列測試的 POST 要求，必須測試應用程式：

1. 提出要求的頁面。
1. 剖析 antiforgery cookie 和來自回應的要求驗證語彙基元。
1. 在位置進行 antiforgery cookie 和要求驗證的 POST 要求語彙基元。

`Post_AddMessageHandler_ReturnsRedirectToRoot`測試方法：

* 準備一個訊息和`HttpClient`。
* 對應用程式中的 POST 要求。
* 檢查回應重新導向至索引頁面。

`Post_AddMessageHandler_ReturnsRedirectToRoot ` 方法 (*tests/RazorPagesTestingSample.Tests/IntegrationTests/IndexPageTest.cs*):

[!code-csharp[](razor-pages-testing/sample/tests/RazorPagesTestingSample.Tests/IntegrationTests/IndexPageTest.cs?name=snippet2)]

`GetRequestContentAsync`公用程式方法會管理準備好要求驗證語彙基元 antiforgery cookie 與用戶端。 請注意此方法接收的方式`IDictionary`，允許呼叫測試方法，傳入要編碼以及要求驗證語彙基元要求的資料 (*tests/RazorPagesTestingSample.Tests/Utilities/Utilities.cs*):

[!code-csharp[](razor-pages-testing/sample/tests/RazorPagesTestingSample.Tests/Utilities/Utilities.cs?name=snippet2&highlight=1-2,8-9,29)]

整合測試也可以將應用程式來測試應用程式的回應行為不正確的資料。 訊息應用程式限制為 200 個字元的訊息長度 (*src/RazorPagesTestingSample/Data/Message.cs*):

[!code-csharp[](razor-pages-testing/sample/src/RazorPagesTestingSample/Data/Message.cs?name=snippet1&highlight=7)]

`Post_AddMessageHandler_ReturnsSuccess_WhenMessageTextTooLong`測試`Message`明確傳入 201"X"字元的文字。 這會導致`ModelState`錯誤。 Post 要求不會重新導向回索引頁面。 它會傳回具有 200 OK `null` `Location`標頭 (*tests/RazorPagesTestingSample.Tests/IntegrationTests/IndexPageTest.cs*):

[!code-csharp[](razor-pages-testing/sample/tests/RazorPagesTestingSample.Tests/IntegrationTests/IndexPageTest.cs?name=snippet3&highlight=7,16-17)]

## <a name="see-also"></a>另請參閱

* [單元測試 C# 中使用 dotnet 測試和 xUnit.NET Core](/dotnet/articles/core/testing/unit-testing-with-dotnet-test)
* [整合測試](xref:testing/integration-testing)
* [測試控制器](xref:mvc/controllers/testing)
* [單元測試程式碼](/visualstudio/test/unit-test-your-code)(Visual Studio)
* [xUnit.net](https://xunit.github.io/)
* [開始使用 xUnit.net （.NET Core/ASP.NET 核心）](https://xunit.github.io/docs/getting-started-dotnet-core)
* [Moq](https://github.com/moq/moq4)
* [Moq 快速入門](https://github.com/Moq/moq4/wiki/Quickstart)
