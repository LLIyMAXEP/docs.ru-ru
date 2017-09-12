---
title: "-refonly (параметры компилятора C#)"
ms.date: 2017-07-08
ms.prod: .net
ms.technology:
- devlang-csharp
ms.topic: article
f1_keywords:
- /refonly
dev_langs:
- CSharp
helpviewer_keywords:
- /refonly compiler option [C#]
- -refonly compiler option [C#]
author: BillWagner
ms.author: wiwagn
ms.translationtype: HT
ms.sourcegitcommit: 81a2252314ef51b5dc01fddc081eb881aa4431a7
ms.openlocfilehash: af99f7565a43dd28b6271611bc8690e7a2e51482
ms.contentlocale: ru-ru
ms.lasthandoff: 08/16/2017

---

# <a name="refonly-c-compiler-options"></a><span data-ttu-id="d7fbc-102">/refonly (параметры компилятора C#)</span><span class="sxs-lookup"><span data-stu-id="d7fbc-102">/refonly (C# Compiler Options)</span></span>

<span data-ttu-id="d7fbc-103">Параметр **/refonly** указывает на то, что в качестве основных выходных данных должна быть выведена базовая сборка, а не сборка реализации.</span><span class="sxs-lookup"><span data-stu-id="d7fbc-103">The **/refonly** The /refonly option indicates that a reference assembly should be output instead of an implementation assembly, as the primary output.</span></span> <span data-ttu-id="d7fbc-104">Параметр `/refonly` автоматически отключает вывод файлов PDB, так как базовые сборки не могут выполняться.</span><span class="sxs-lookup"><span data-stu-id="d7fbc-104">The `/refonly` parameter silently disables outputting PDBs, as reference assemblies cannot be executed.</span></span>

## <a name="syntax"></a><span data-ttu-id="d7fbc-105">Синтаксис</span><span class="sxs-lookup"><span data-stu-id="d7fbc-105">Syntax</span></span>

```console
/refonly
```

## <a name="remarks"></a><span data-ttu-id="d7fbc-106">Примечания</span><span class="sxs-lookup"><span data-stu-id="d7fbc-106">Remarks</span></span>

<span data-ttu-id="d7fbc-107">Тела методов в сборках, состоящих только из метаданных, заменяются одним телом `throw null`, но такие сборки включают все члены, кроме анонимных типов.</span><span class="sxs-lookup"><span data-stu-id="d7fbc-107">Metadata-only assemblies have their method bodies replaced with a single `throw null` body, but include all members except anonymous types.</span></span> <span data-ttu-id="d7fbc-108">Тело `throw null` используется для того, чтобы могло выполняться средство PEVerify для проверки полноты метаданных, что было бы невозможно при отсутствии тела.</span><span class="sxs-lookup"><span data-stu-id="d7fbc-108">The reason for using `throw null` bodies (as opposed to no bodies) is so that PEVerify could run and pass (thus validating the completeness of the metadata).</span></span>

<span data-ttu-id="d7fbc-109">Базовые сборки содержат атрибут `ReferenceAssembly` уровня сборки.</span><span class="sxs-lookup"><span data-stu-id="d7fbc-109">Reference assemblies include an assembly-level `ReferenceAssembly` attribute.</span></span> <span data-ttu-id="d7fbc-110">Этот атрибут может быть задан в исходном коде (в этом случае компилятору не требуется его синтезировать).</span><span class="sxs-lookup"><span data-stu-id="d7fbc-110">This attribute may be specified in source (then the compiler won't need to synthesize it).</span></span> <span data-ttu-id="d7fbc-111">Из-за этого атрибута среды выполнения не будут загружать базовые сборки для выполнения (однако они могут загружаться в режиме только для отражения).</span><span class="sxs-lookup"><span data-stu-id="d7fbc-111">Because of this attribute, runtimes will refuse to load reference assemblies for execution (but they can still be loaded in reflection-only mode).</span></span> <span data-ttu-id="d7fbc-112">Средства, выполняющие отражение сборок, должны загружать базовые сборки как доступные только для отражения. В противном случае от среды выполнения будет получена ошибка загрузки типа.</span><span class="sxs-lookup"><span data-stu-id="d7fbc-112">Tools that reflect on assemblies need to ensure they load reference assemblies as reflection-only, otherwise they will receive a typeload error from the runtime.</span></span>

<span data-ttu-id="d7fbc-113">Базовые сборки удаляют метаданные (закрытые члены) из сборок, содержащих только метаданные.</span><span class="sxs-lookup"><span data-stu-id="d7fbc-113">Reference assemblies further remove metadata (private members) from metadata-only assemblies:</span></span>

- <span data-ttu-id="d7fbc-114">Базовая сборка содержит ссылки только на необходимые компоненты в слое доступа API.</span><span class="sxs-lookup"><span data-stu-id="d7fbc-114">A reference assembly only has references for what it needs in the API surface.</span></span> <span data-ttu-id="d7fbc-115">Реальная сборка может иметь дополнительные ссылки, связанные с конкретной реализацией.</span><span class="sxs-lookup"><span data-stu-id="d7fbc-115">The real assembly may have additional references related to specific implementations.</span></span> <span data-ttu-id="d7fbc-116">Например, базовая сборка для `class C { private void M() { dynamic d = 1; ... } }` не ссылается на типы, требуемые для `dynamic`.</span><span class="sxs-lookup"><span data-stu-id="d7fbc-116">For instance, the reference assembly for `class C { private void M() { dynamic d = 1; ... } }` does not reference any types required for `dynamic`.</span></span>
- <span data-ttu-id="d7fbc-117">Закрытые функции-члены (методы, свойства и события) удаляются в случае, если их удаление не скажется заметно на компиляции.</span><span class="sxs-lookup"><span data-stu-id="d7fbc-117">Private function-members (methods, properties, and events) are removed in cases where their removal doesn't observably impact compilation.</span></span> <span data-ttu-id="d7fbc-118">Если нет атрибутов `InternalsVisibleTo`, то же самое выполняется для внутренних функций-членов.</span><span class="sxs-lookup"><span data-stu-id="d7fbc-118">If there are no `InternalsVisibleTo` attributes, do the same for internal function-members.</span></span>
- <span data-ttu-id="d7fbc-119">Однако в базовых сборках сохраняются все типы (включая закрытые и вложенные).</span><span class="sxs-lookup"><span data-stu-id="d7fbc-119">But all types (including private or nested types) are kept in reference assemblies.</span></span> <span data-ttu-id="d7fbc-120">Сохраняются все атрибуты (даже внутренние).</span><span class="sxs-lookup"><span data-stu-id="d7fbc-120">All attributes are kept (even internal ones).</span></span>
- <span data-ttu-id="d7fbc-121">Сохраняются все виртуальные методы.</span><span class="sxs-lookup"><span data-stu-id="d7fbc-121">All virtual methods are kept.</span></span> <span data-ttu-id="d7fbc-122">Сохраняются явные реализации интерфейса.</span><span class="sxs-lookup"><span data-stu-id="d7fbc-122">Explicit interface implementations are kept.</span></span> <span data-ttu-id="d7fbc-123">Явно реализованные свойства и события сохраняются, так как их методы доступа являются виртуальными (и, следовательно, сохраняются).</span><span class="sxs-lookup"><span data-stu-id="d7fbc-123">Explicitly implemented properties and events are kept, as their accessors are virtual (and are therefore kept).</span></span>
- <span data-ttu-id="d7fbc-124">Сохраняются все поля структуры.</span><span class="sxs-lookup"><span data-stu-id="d7fbc-124">All fields of a struct are kept.</span></span> <span data-ttu-id="d7fbc-125">(Возможно, это будет изменено в версиях после C# 7.1.)</span><span class="sxs-lookup"><span data-stu-id="d7fbc-125">(This is a candidate for post-C#-7.1 refinement)</span></span>

<span data-ttu-id="d7fbc-126">Параметры `/refonly` и [`/refout`](refout-compiler-option.md) являются взаимоисключающими.</span><span class="sxs-lookup"><span data-stu-id="d7fbc-126">The `/refonly` and [`/refout`](refout-compiler-option.md) options are mutually exclusive.</span></span>

## <a name="see-also"></a><span data-ttu-id="d7fbc-127">См. также</span><span class="sxs-lookup"><span data-stu-id="d7fbc-127">See also</span></span>
 <span data-ttu-id="d7fbc-128">[Параметры компилятора C#](../../../csharp/language-reference/compiler-options/index.md) </span><span class="sxs-lookup"><span data-stu-id="d7fbc-128">[C# Compiler Options](../../../csharp/language-reference/compiler-options/index.md) </span></span>  
 [<span data-ttu-id="d7fbc-129">Управление свойствами проектов и решений</span><span class="sxs-lookup"><span data-stu-id="d7fbc-129">Managing Project and Solution Properties</span></span>](/visualstudio/ide/managing-project-and-solution-properties)
