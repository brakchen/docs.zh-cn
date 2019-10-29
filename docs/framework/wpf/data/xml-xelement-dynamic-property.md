---
title: Xml（XElement 动态属性）
ms.date: 10/22/2019
ms.topic: reference
apiname:
- XElement.Xml
ms.openlocfilehash: 9bac9797f397a0bea1dda36bf864f88293061893
ms.sourcegitcommit: 82f94a44ad5c64a399df2a03fa842db308185a76
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/25/2019
ms.locfileid: "72921034"
---
# <a name="xml-xelement-dynamic-property"></a><span data-ttu-id="ef92d-102">Xml（XElement 动态属性）</span><span class="sxs-lookup"><span data-stu-id="ef92d-102">Xml (XElement dynamic property)</span></span>

<span data-ttu-id="ef92d-103">获取元素的非格式化 XML 内容。</span><span class="sxs-lookup"><span data-stu-id="ef92d-103">Gets the unformatted XML content of the element.</span></span>

## <a name="syntax"></a><span data-ttu-id="ef92d-104">语法</span><span class="sxs-lookup"><span data-stu-id="ef92d-104">Syntax</span></span>

```xaml
elem.Xml
```

## <a name="property-valuereturn-value"></a><span data-ttu-id="ef92d-105">属性值/返回值</span><span class="sxs-lookup"><span data-stu-id="ef92d-105">Property value/return value</span></span>

<span data-ttu-id="ef92d-106">表示元素的非格式化 XML 内容的 <xref:System.String>。</span><span class="sxs-lookup"><span data-stu-id="ef92d-106">A <xref:System.String> that represents the unformatted XML content of the element.</span></span>

## <a name="remarks"></a><span data-ttu-id="ef92d-107">备注</span><span class="sxs-lookup"><span data-stu-id="ef92d-107">Remarks</span></span>

<span data-ttu-id="ef92d-108">此属性等效于 <xref:System.Xml.Linq.XNode.ToString(System.Xml.Linq.SaveOptions)> 类的 <xref:System.Xml.Linq.XNode?displayProperty=fullName> 方法，其 `SaveOptions` 参数设置为 <xref:System.Xml.Linq.SaveOptions>。</span><span class="sxs-lookup"><span data-stu-id="ef92d-108">This property is equivalent to the <xref:System.Xml.Linq.XNode.ToString(System.Xml.Linq.SaveOptions)> method of the <xref:System.Xml.Linq.XNode?displayProperty=fullName> class, with the `SaveOptions` parameter set to <xref:System.Xml.Linq.SaveOptions>.</span></span>

## <a name="see-also"></a><span data-ttu-id="ef92d-109">请参阅</span><span class="sxs-lookup"><span data-stu-id="ef92d-109">See also</span></span>

- [<span data-ttu-id="ef92d-110">XElement 类动态属性</span><span class="sxs-lookup"><span data-stu-id="ef92d-110">XElement class dynamic properties</span></span>](attribute-xelement-dynamic-property.md)
- [<span data-ttu-id="ef92d-111">“值”</span><span class="sxs-lookup"><span data-stu-id="ef92d-111">Value</span></span>](value-xelement-dynamic-property.md)