---
title: 异步限制
ms.date: 12/13/2019
description: 说明异步 Api 的限制和一些可改用的替代项。
ms.openlocfilehash: 350237dc5c03023f60e9680e8b9c94aabb62606f
ms.sourcegitcommit: 30a558d23e3ac5a52071121a52c305c85fe15726
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/25/2019
ms.locfileid: "75450332"
---
# <a name="async-limitations"></a><span data-ttu-id="4ae07-103">异步限制</span><span class="sxs-lookup"><span data-stu-id="4ae07-103">Async limitations</span></span>

<span data-ttu-id="4ae07-104">SQLite 不支持异步 i/o。</span><span class="sxs-lookup"><span data-stu-id="4ae07-104">SQLite doesn't support asynchronous I/O.</span></span> <span data-ttu-id="4ae07-105">异步 ADO.NET 方法将在中同步执行。</span><span class="sxs-lookup"><span data-stu-id="4ae07-105">Async ADO.NET methods will execute synchronously in Microsoft.Data.Sqlite.</span></span> <span data-ttu-id="4ae07-106">避免调用它们。</span><span class="sxs-lookup"><span data-stu-id="4ae07-106">Avoid calling them.</span></span>

<span data-ttu-id="4ae07-107">相反，请使用[预写日志记录](https://www.sqlite.org/wal.html)来提高性能和并发性。</span><span class="sxs-lookup"><span data-stu-id="4ae07-107">Instead, use [write-ahead logging](https://www.sqlite.org/wal.html) to improve performance and concurrency.</span></span>

[!code-csharp[](../../../../samples/snippets/standard/data/sqlite/AsyncSample/Program.cs?name=snippet_WAL)]

> [!TIP]
> <span data-ttu-id="4ae07-108">默认情况下，在使用[Entity Framework Core](/ef/core/)创建的数据库上启用预写日志记录。</span><span class="sxs-lookup"><span data-stu-id="4ae07-108">Write-ahead logging is enabled by default on databases created using [Entity Framework Core](/ef/core/).</span></span>