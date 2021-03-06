---
uid: mvc/overview/getting-started/database-first-development/changing-the-database
title: 第一個使用 ASP.NET MVC 的 EF 資料庫： 變更的資料庫 |Microsoft 文件
author: tfitzmac
description: 使用 MVC、 Entity Framework 和 ASP.NET Scaffolding，您可以建立 web 應用程式提供的介面到現有的資料庫。 此教學課程里...
ms.author: aspnetcontent
manager: wpickett
ms.date: 10/01/2014
ms.topic: article
ms.assetid: cfd5c083-a319-482e-8f25-5b38caa93954
ms.technology: dotnet-mvc
ms.prod: .net-framework
msc.legacyurl: /mvc/overview/getting-started/database-first-development/changing-the-database
msc.type: authoredcontent
ms.openlocfilehash: 63ee8768a43dbdac80922e3adbedd3378c10da73
ms.sourcegitcommit: f8852267f463b62d7f975e56bea9aa3f68fbbdeb
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/06/2018
---
<a name="ef-database-first-with-aspnet-mvc-changing-the-database"></a>第一個使用 ASP.NET MVC 的 EF 資料庫： 資料庫進行變更
====================
由[Tom FitzMacken](https://github.com/tfitzmac)

> 使用 MVC、 Entity Framework 和 ASP.NET Scaffolding，您可以建立 web 應用程式提供的介面到現有的資料庫。 此教學課程的數列會顯示如何自動產生程式碼，可讓使用者顯示、 編輯、 建立和刪除存在於資料庫資料表中的資料。 產生的程式碼會對應至資料庫資料表中的資料行。
> 
> 數列的這個部分著重在對資料庫結構的更新及傳播該項變更整個 web 應用程式。


## <a name="add-a-column"></a>加入資料行

如果您在資料庫中更新之資料表的結構，您需要確保您的變更會傳播至資料模型、 檢視和控制器。

此教學課程中，您會將新的資料行，加入學生資料表記錄的學生的中間名。 若要加入此資料行，開啟資料庫專案，然後開啟 Student.sql 檔案。 透過設計工具或 T-SQL 程式碼中，加入名為資料行**MiddleName**它是 nvarchar （50），而且允許 NULL 值。

![加入 「 中間名](changing-the-database/_static/image1.png)

透過啟動您的資料庫專案 （或 F5），將這項變更部署到您的本機資料庫。 新的欄位加入至資料表。 如果您看不到 SQL Server 物件總管 中，按一下重新整理按鈕在窗格中。

![顯示新的資料行](changing-the-database/_static/image2.png)

新的資料行存在於資料庫資料表中，但是它目前不存在的資料模型類別中。 您必須更新模型，以包含新的資料行。 在**模型**資料夾中，開啟**ContosoModel.edmx**檔案，以顯示模型圖表。 請注意學生模型不包含 [MiddleName] 屬性。 以滑鼠右鍵按一下設計介面，然後選取**從資料庫更新模型**。

![更新模型](changing-the-database/_static/image3.png)

在 更新精靈 中，選取**重新整理** 索引標籤和**學生**資料表。

![更新精靈](changing-the-database/_static/image4.png)

按一下 [ **完成**]。

在更新程序完成之後，資料庫圖表包含新**MiddleName**屬性。 儲存**ContosoModel.edmx**檔案。 您必須將儲存為新的屬性，會傳播到這個檔案**Student.cs**類別。 您現在已更新資料庫和模型。

建置方案。

不幸的是，在檢視還是不包含新的屬性。 若要更新的檢視有兩個選項-您可以重新產生檢視再次加入 scaffold Student 類別，或您可以手動將新屬性新增至現有的檢視。 在本教學課程中，您會加入樣板再次因為您已不變更任何自訂的自動產生的檢視。 您可以考慮以手動方式將屬性加入您所做的變更來檢視和不想要捨棄這些變更時。

若要確保重新建立檢視，請刪除**學生**下的資料夾**檢視**，並刪除**StudentsController**。 然後，以滑鼠右鍵按一下**控制器**資料夾和樣板加入**學生**模型。 同樣地，命名為控制器**StudentsController**。 選取 [確定]。

在檢視現在會包含 [MiddleName] 屬性。

![顯示中間名](changing-the-database/_static/image5.png)

在下一步 區段中，您將加入程式碼來自訂顯示學生記錄的詳細檢視。

> [!div class="step-by-step"]
> [上一頁](generating-views.md)
> [下一頁](customizing-a-view.md)
