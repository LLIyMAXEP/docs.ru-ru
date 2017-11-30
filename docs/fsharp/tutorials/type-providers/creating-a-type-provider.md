---
title: "Учебное руководство. Создание поставщика типов (F#)"
description: "Описание способов создания собственных поставщиков типов F # в F # 3.0, изучив несколько поставщиков простого типа, чтобы продемонстрировать основные понятия."
keywords: "visual f#, f#, функциональное программирование"
author: cartermp
ms.author: phcart
ms.date: 05/16/2016
ms.topic: language-reference
ms.prod: .net
ms.technology: devlang-fsharp
ms.devlang: fsharp
ms.assetid: 82bec076-19d4-470c-979f-6c3a14b7c70a
ms.openlocfilehash: a1d6315c2546de12e85efdd06cf2520605cb6e91
ms.sourcegitcommit: a19548e5167cbe7e9e58df4ffd8c3b23f17d5c7a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/02/2017
---
# <a name="tutorial-creating-a-type-provider"></a>Учебник: Создание поставщика типов

> [!NOTE]
В этом руководстве, была написана для F # 3.0 и будут обновлены.

Механизм поставщик типа F # 3.0 является значительная часть его поддержка сведения широкие возможности программирования. Этот учебник посвящен созданию собственных поставщиков типов посредством разработки из нескольких поставщиков простого типа, чтобы продемонстрировать основные понятия. Дополнительные сведения о механизме поставщик типа F # см. в разделе [поставщиков типов](index.md).

F # 3.0 содержит несколько встроенных поставщиков типов для часто используемых Интернет и enterprise служб данных. Эти поставщики типов предоставляют простой постоянный доступ к реляционным базам данных SQL и сетевых служб OData и WSDL. Эти поставщики также поддерживают использование запросов LINQ F # для этих источников данных.

При необходимости можно создать пользовательские поставщики типов или ссылаются на поставщики типов, созданных другими пользователями. Например ваша организация может иметь службу данных, которая предоставляет большое и возрастающее число именованных наборов данных, каждый из которых свой собственный стабильная схема данных. Можно создать поставщик типов, который считывает схемы и представляет текущее наборы данных программисту строго типизированным образом.


## <a name="before-you-start"></a>Прежде чем начать
Механизм поставщика типа в основном предназначен для вводится неизменных данных и службы информационных пространств опыт программирования F #.

Этот механизм не поддерживает внедрение информационных пространств, схема которой изменяется во время выполнения программы таким образом, относящиеся к программной логики. Кроме того механизм не поддерживает внутри языка метапрограммирования, несмотря на то, что этот домен содержит некоторые допустимые варианты. Этот механизм следует использовать только при необходимости и где разработки поставщик типа создает очень большое значение.

Следует создавать поставщик типа, где схема недоступна. Аналогичным образом, следует создавать поставщик типа там, где обычных (или даже существующего) Библиотека .NET было бы достаточно.

Прежде чем начать, можно задать следующие вопросы:


- У схемы для источника сведений? Если это так, что такое сопоставление в F # и системой типов .NET

- Можно ли использовать существующие интерфейсы API (динамического типа) в качестве отправной точки для реализации?

- Вы и ваша организация будет достаточно использует поставщик типа делает написание ее смысл? Обычные библиотеки .NET удовлетворят вашим потребностям?

- Сколько изменится схемы?

- Изменится во время кодирования?

- Изменится между кодирования сеансы?

- Изменится во время выполнения программы?

Поставщики типов являются наилучшим образом подходит для случаев, когда схема стабильный во время выполнения и в течение времени существования скомпилированного кода.


## <a name="a-simple-type-provider"></a>Поставщик простой тип
В этом примере рассматривается Samples.HelloWorldTypeProvider в `SampleProviders\Providers` каталог [F # 3.0 образец пакета](http://fsharp3sample.codeplex.com) на веб-сайте Codeplex. Поставщик делает доступными «тип space», содержащий 100 вместо удаленных типов, как показано в следующем коде, используя синтаксис подписи F # и пропуск подробные сведения для всех, кроме `Type1`. Дополнительные сведения об удаленных типах см. в разделе [сведения о стереть предоставленные типы](#details-about-erased-provided-types) далее в этом разделе.

```fsharp
namespace Samples.HelloWorldTypeProvider

type Type1 =
    /// This is a static property.
    static member StaticProperty : string

    /// This constructor takes no arguments.
    new : unit -> Type1

    /// This constructor takes one argument.
    new : data:string -> Type1

    /// This is an instance property.
    member InstanceProperty : int

    /// This is an instance method.
    member InstanceMethod : x:int -> char

    /// This is an instance property.
    nested type NestedType = 
        /// This is StaticProperty1 on NestedType.
        static member StaticProperty1 : string
        …
        /// This is StaticProperty100 on NestedType.
        static member StaticProperty100 : string

type Type2 =
…
…

type Type100 =
…
```

Обратите внимание, набор типов и членов, предоставляемых статически известен. В этом примере не используют возможность поставщики предоставляют типы, зависящие от схемы. В следующем коде представлен реализации поставщика типа и подробности описаны в следующих разделах этой статьи.


>[!WARNING] 
Возможно, некоторые небольшие именования различия между этим кодом и примеров в сети.

```fsharp
namespace Samples.FSharp.HelloWorldTypeProvider

open System
open System.Reflection
open Samples.FSharp.ProvidedTypes
open Microsoft.FSharp.Core.CompilerServices
open Microsoft.FSharp.Quotations

// This type defines the type provider. When compiled to a DLL, it can be added
// as a reference to an F# command-line compilation, script, or project.
[<TypeProvider>]
type SampleTypeProvider(config: TypeProviderConfig) as this = 

  // Inheriting from this type provides implementations of ITypeProvider 
  // in terms of the provided types below.
  inherit TypeProviderForNamespaces()

  let namespaceName = "Samples.HelloWorldTypeProvider"
  let thisAssembly = Assembly.GetExecutingAssembly()

  // Make one provided type, called TypeN.
  let makeOneProvidedType (n:int) = 
  …
  // Now generate 100 types
  let types = [ for i in 1 .. 100 -> makeOneProvidedType i ] 

  // And add them to the namespace
  do this.AddNamespace(namespaceName, types)

  [<assembly:TypeProviderAssembly>] 
  do()
```

Чтобы использовать этот поставщик, откройте отдельный экземпляр Visual Studio 2012, создание скрипта F # и затем добавьте ссылку на службу из скрипта с помощью #r, как показано в следующем коде:

```fsharp
#r @".\bin\Debug\Samples.HelloWorldTypeProvider.dll"

let obj1 = Samples.HelloWorldTypeProvider.Type1("some data")

let obj2 = Samples.HelloWorldTypeProvider.Type1("some other data")

obj1.InstanceProperty
obj2.InstanceProperty

[ for index in 0 .. obj1.InstanceProperty-1 -> obj1.InstanceMethod(index) ]
[ for index in 0 .. obj2.InstanceProperty-1 -> obj2.InstanceMethod(index) ]

let data1 = Samples.HelloWorldTypeProvider.Type1.NestedType.StaticProperty35
```

Выполните поиск типов в `Samples.HelloWorldTypeProvider` пространство имен, которое создается поставщик типа.

Перед перекомпиляцией поставщик убедитесь, что закрыты все экземпляры Visual Studio и F # Interactive, использующих DLL-Библиотеки поставщика. В противном случае возникает ошибка сборки будет возникать, так как выходные данные библиотеки DLL будет заблокирована.

Чтобы отладить этот поставщик с помощью инструкции print, скрипт, указывающий на проблему с поставщиком, а затем использовать следующий код:

```fsharp
fsc.exe -r:bin\Debug\HelloWorldTypeProvider.dll script.fsx
```

Чтобы отладить этот поставщик с помощью Visual Studio, откройте командную строку Visual Studio с правами администратора и выполните следующую команду:

```fsharp
devenv.exe /debugexe fsc.exe -r:bin\Debug\HelloWorldTypeProvider.dll script.fsx
```

Кроме того, откройте Visual Studio откройте меню "Отладка", выберите `Debug/Attach to process…`и присоединить к другому `devenv` процесса, где требуется изменить скрипт. С помощью этого метода, можно точнее указывать определенной логики у поставщика типов, введя интерактивно выражения в второй экземпляр (с технологией IntelliSense и других функций).

Можно отключить только мой код отладки, чтобы облегчить поиск ошибок в созданном коде. Сведения о том, как включить или отключить эту функцию, в разделе [Навигация по коду с помощью отладчика](https://msdn.microsoft.com/library/y740d9d3.aspx). Кроме того, можно также задать исключения первого шанса, перехват, открыв `Debug` меню и затем выбрав `Exceptions` или нажав клавиши Ctrl + Alt + E, чтобы открыть `Exceptions` диалоговое окно. В этом диалоговом окне в разделе `Common Language Runtime Exceptions`выберите `Thrown` флажок.


### <a name="implementation-of-the-type-provider"></a>Реализация поставщика типов
В этом разделе описывается основной части реализации поставщика типа. Во-первых определения типа для пользовательского типа самим поставщиком:

```fsharp
[<TypeProvider>]
type SampleTypeProvider(config: TypeProviderConfig) as this =
```

Этот тип должен быть открытым, и необходимо отметить его с [TypeProvider](https://msdn.microsoft.com/library/bdf7b036-7490-4ace-b79f-c5f1b1b37947) таким образом, компилятор распознает поставщика типов при отдельном проекте F # ссылается на сборку, содержащую тип. *Config* является необязательным и, если он имеется, содержит сведения о экземпляр типа поставщика, который компилятор F # создает контекстные конфигурации.

Далее необходимо реализовать [ITypeProvider](https://msdn.microsoft.com/library/2c2b0571-843d-4a7d-95d4-0a7510ed5e2f) интерфейса. В этом случае используется `TypeProviderForNamespaces` из тип `ProvidedTypes` API в качестве базового типа. Этот вспомогательный тип можно предоставить на конечную коллекцию заранее предоставлена пространства имен, непосредственно каждый из которых содержит конечное количество предопределенных, заранее типов. В этом контексте, поставщик *заранее* создает типы, даже если они не требуется или не используется.

```fsharp
inherit TypeProviderForNamespaces()
```

Затем определить локальной частной значения, которые определяют пространство имен для существующих и найти сборку поставщика в тип. Эта сборка используется как логический родительский тип удаленных типов, которые предоставляются для более поздней версии.

```fsharp
let namespaceName = "Samples.HelloWorldTypeProvider"
let thisAssembly = Assembly.GetExecutingAssembly()
```

Далее создайте функцию для предоставления каждого типа тип1... Type100. Эта функция является более подробно далее в этом разделе.

```fsharp
let makeOneProvidedType (n:int) = …
```

Теперь можно создать 100 предоставляемых типов:

```fsharp
let types = [ for i in 1 .. 100 -> makeOneProvidedType i ]
```

Добавьте типы указанного пространства имен.

```fsharp
do this.AddNamespace(namespaceName, types)
```

Наконец добавьте атрибут сборки, которое указывает, что вы создаете DLL-Библиотека поставщика типа:

```fsharp
[<assembly:TypeProviderAssembly>] 
do()
```

### <a name="providing-one-type-and-its-members"></a>Предоставление одного типа и его членов
`makeOneProvidedType` Функция не работает указать один из типов.

```fsharp
let makeOneProvidedType (n:int) = 
…
```

Этот шаг описывается реализация этой функции. Сначала создайте предоставленный тип (например, тип 1, если n = 1 или Type57, когда n = 57).

```fsharp
// This is the provided type. It is an erased provided type and, in compiled code, 
// will appear as type 'obj'.
let t = ProvidedTypeDefinition(thisAssembly,namespaceName,
"Type" + string n,
baseType = Some typeof<obj>)
```

Следует отметить следующие моменты:


- Это при условии, что тип удаляется.  Поскольку указывают, что базовый тип является `obj`, экземпляров будет отображаться в виде значения типа [obj](https://msdn.microsoft.com/library/dcf2430f-702b-40e5-a0a1-97518bf137f7) в скомпилированный код.
<br />

- При указании типа, не являющегося вложенным, необходимо указать сборку и пространство имен. Для удаленных типов сборки должно быть самой сборки типа поставщика.
<br />

Добавьте XML-документации для типа. В этой документации задерживается, то есть, вычисляются по требованию, если компилятор узла она нужна.

```fsharp
t.AddXmlDocDelayed (fun () -> sprintf "This provided type %s" ("Type" + string n))
```

Далее вы добавите предоставленный статическое свойство в тип:

```fsharp
let staticProp = ProvidedProperty(propertyName = "StaticProperty", 
propertyType = typeof<string>, 
IsStatic=true,
GetterCode= (fun args -> <@@ "Hello!" @@>))
```

При получении этого свойства всегда возвращают строку «Hello!». `GetterCode` Для свойство использует F # предложение, которое представляет код, который создает компилятор узла для получения свойства. Дополнительные сведения о предложениях см. в разделе [Цитирование кода (F #)](https://msdn.microsoft.com/library/6f055397-a1f0-4f9a-927c-f0d7c6951155).

Добавьте XML-документации в свойство.

```fsharp
staticProp.AddXmlDocDelayed(fun () -> "This is a static property")
```

Теперь подключиться предоставленного свойства в указанный тип. Необходимо добавить указанный элемент только один тип. В противном случае, значение элемента никогда не будут доступны.

```fsharp
t.AddMember staticProp
```

Теперь можно создайте указанный конструктор, который не принимает никаких параметров.

```fsharp
let ctor = ProvidedConstructor(parameters = [ ], 
InvokeCode= (fun args -> <@@ "The object data" :> obj @@>))
```

`InvokeCode` Для завершения работы конструктор возвращает F # предложение, которое представляет код, который компилятор узла приводит к возникновению ошибки при вызове конструктора. Например можно использовать следующий конструктор:

```fsharp
new Type10()
```

Экземпляр указанного типа создается с базовыми данными «Данные объекта». Заключенные в кавычки код включает преобразование в [obj](https://msdn.microsoft.com/library/dcf2430f-702b-40e5-a0a1-97518bf137f7) , так как этот тип является стирания это предоставленный тип (как при объявлении предоставленного типа указан).

Добавьте в конструктор XML-документации и добавьте указанный конструктор для предоставленного типа:

```fsharp
ctor.AddXmlDocDelayed(fun () -> "This is a constructor")

t.AddMember ctor
```

Создайте второй предоставленный конструктор, который принимает один параметр:

```fsharp
let ctor2 = 
ProvidedConstructor(parameters = [ ProvidedParameter("data",typeof<string>) ], 
InvokeCode= (fun args -> <@@ (%%(args.[0]) : string) :> obj @@>))
```

`InvokeCode` Для конструктора снова возвращает F # предложение, которое представляет код, созданный в компиляторе базовой среды для вызова метода. Например можно использовать следующий конструктор:

```fsharp
new Type10("ten")
```

Экземпляр указанного типа создается с базовыми данными «10». Возможно, вы уже заметили, `InvokeCode` функция возвращает предложения. Входные данные для этой функции — список выражений, по одной на каждый параметр конструктора. В этом случае выражение, представляющее значение одного параметра доступна в `args.[0]`. Код для вызова конструктора приводит возвращаемое значение в тип стертой `obj`. После добавления второй конструктор указанного типа, можно создать свойство предоставленный экземпляр:

```fsharp
let instanceProp = 
ProvidedProperty(propertyName = "InstanceProperty", 
propertyType = typeof<int>, 
GetterCode= (fun args -> 
<@@ ((%%(args.[0]) : obj) :?> string).Length @@>))
instanceProp.AddXmlDocDelayed(fun () -> "This is an instance property")
t.AddMember instanceProp
```

При получении этого свойства возвращается длина строки, которое является объектом представления. `GetterCode` Свойство возвращает F # предложение, которое указывает код, который создает компилятор узла для получения свойства. Как `InvokeCode`, `GetterCode` функция возвращает предложения. Компилятор узла вызывает эту функцию со списком аргументов. В этом случае аргументы включать только одно выражение, представляющее экземпляр, на котором вызывается метод получения значения, к которому можно получить с помощью `args.[0]`. Реализация `GetterCode` затем получает эту предложениям результат, введите в удаленных `obj`, и использовать приведение для удовлетворения компилятора механизм для проверки типов, что объект является строкой. В следующей части `makeOneProvidedType` предоставляет метод экземпляра с одним параметром.

```fsharp
let instanceMeth = 
ProvidedMethod(methodName = "InstanceMethod", 
parameters = [ProvidedParameter("x",typeof<int>)], 
returnType = typeof<char>, 
InvokeCode = (fun args -> 
<@@ ((%%(args.[0]) : obj) :?> string).Chars(%%(args.[1]) : int) @@>))

instanceMeth.AddXmlDocDelayed(fun () -> "This is an instance method")
// Add the instance method to the type.
t.AddMember instanceMeth
```

Наконец создайте вложенный тип, содержащий 100 вложенных свойств. Создание этого вложенные типа и его свойства задерживается, то есть, вычисляемые по требованию.

```fsharp
t.AddMembersDelayed(fun () -> 
let nestedType = ProvidedTypeDefinition("NestedType",
Some typeof<obj>

)

nestedType.AddMembersDelayed (fun () -> 
let staticPropsInNestedType = 
[ for i in 1 .. 100 do
let valueOfTheProperty = "I am string "  + string i

let p = ProvidedProperty(propertyName = "StaticProperty" + string i, 
propertyType = typeof<string>, 
IsStatic=true,
GetterCode= (fun args -> <@@ valueOfTheProperty @@>))

p.AddXmlDocDelayed(fun () -> 
sprintf "This is StaticProperty%d on NestedType" i)

yield p ]
staticPropsInNestedType)

[nestedType])

// The result of makeOneProvidedType is the type.
t
```

### <a name="details-about-erased-provided-types"></a>Сведения об удаленных предоставляемых типов
В примере в этом разделе приводится только *стереть предоставляемых типов*, что особенно полезны в следующих ситуациях:


- При написании поставщика для информационное пространство, который содержит только данные и методы.
<br />

- При написании поставщика, где семантику точных типов среды выполнения не являются критическими для практического использования пространства сведения.
<br />

- При написании поставщика для информационное пространство, который является слишком большим, соединенных друг с другом, чтобы оно не технически возможно, для создания реальных типов .NET для информации место.
<br />

В этом примере каждый предоставленный тип удаления данных в тип `obj`, и все случаи использования типа будут отображаться как тип `obj` в скомпилированный код. На самом деле базовых объектов в этих примерах являются строками, но тип будет отображаться как `System.Object` в .NET, скомпилированный код. Как все случаи использования усилий используется явная упаковка, распаковка-преобразование и приведение к разрушить удалены типов. В этом случае при использовании объекта, может привести приведения исключение, которое является недопустимым. Поставщик среды выполнения можно определить собственный тип закрытого представление для защиты от представления значение false. Не удается определить удаленных типов в F #. Только в том случае, если типы могут быть удалены. Необходимо понимать последствия обоих Практическое применение, а семантических одним удаленных типов для поставщика тип или поставщика, который предоставляет удалены типов. Удаленный тип имеет тип не реальными .NET. Таким образом точное отражение не поддерживается на тип элемента, и может разрушить удаленных типов при использовании приведений среды выполнения и другие методы, которые используют семантику типа точное время выполнения. Subversion удаленных типов часто приводит к исключениям приведения типа во время выполнения.


### <a name="choosing-representations-for-erased-provided-types"></a>Выбор представления для удаления предоставленных типов
Некоторые варианты использования удаленных предоставляемых типов нет представления не требуется. Например удаленный в тип может содержать только статические свойства и члены и нет конструкторов и методов и свойств возвращает экземпляр типа. Если требуется получить доступ из удаленных экземпляров предоставленного типа, необходимо учитывать следующие вопросы:


- Что такое стирания предоставленного типа
<br />
  - Стирания предоставленного типа является, как тип появится в скомпилированном коде .NET.
<br />

  - Стирания предоставленный класс стертой типа является всегда первый не удалены базовым типом в цепочке наследования типа.
<br />

  - Всегда является стирания предоставленный интерфейс стертой типа `System.Object`.
<br />

- Что такое представления предоставленного типа
<br />
  - Набор возможных объектов для удаленных предоставленного типа, называются его представления. В примере в этом документе представления всех удаленных предоставляемых типов `Type1..Type100` всегда являются объектами string.
<br />

Все представления предоставленного типа должны быть совместимы с стирания предоставленного типа. (В противном случае компилятор F # будет выдавать ошибку для использования поставщика типа, или создается непроверяемый код .NET, который является недопустимым. Поставщик типов не является допустимым, если он возвращает код, который дает представление, которое является недопустимым.)

Можно выбрать представление для указанных объектов с помощью любого из следующих подходов, которые являются очень часто.


- Если просто предоставляет строго типизированной оболочки на существующий тип .NET, часто имеет смысл для типа, чтобы стереть к этому типу, используйте экземпляры этого типа в качестве представления или оба. Этот подход используется, когда большинство существующих методов для этого типа по-прежнему имеет смысла при использовании строго типизированную версию.
<br />

- Если требуется значительно создать интерфейс API, который отличается от любого существующего API .NET, имеет смысл создать типы среды выполнения, которые будут усилий и представления для указанных типов.
<br />

В примере в этом документе используются строки как представления указанных объектов. Часто возможно, следует использовать для представления других объектов. Например вы можете использовать словарь, контейнер свойств:

```fsharp
ProvidedConstructor(parameters = [], 
InvokeCode= (fun args -> <@@ (new Dictionary<string,obj>()) :> obj @@>))
```

Кроме того могут определять тип в поставщике тип, который будет использоваться во время выполнения для формирования представление, вместе с одной или несколькими операциями среды выполнения:

```fsharp
type DataObject() =
let data = Dictionary<string,obj>()
member x.RuntimeOperation() = data.Count
```

Предоставляемые элементы затем можно создать экземпляры объектов данного типа:

```fsharp
ProvidedConstructor(parameters = [], 
InvokeCode= (fun args -> <@@ (new DataObject()) :> obj @@>))
```

В этом случае вы можете (необязательно) использовать этот тип, усилий, указав этот тип как `baseType` при создании `ProvidedTypeDefinition`:

```fsharp
ProvidedTypeDefinition(…, baseType = Some typeof<DataObject> )
…
ProvidedConstructor(…, InvokeCode = (fun args -> <@@ new DataObject() @@>), …)
```

`Key Lessons`

В предыдущем разделе описано создание простой стирания поставщик типов, который предоставляет широкий набор типов, свойств и методов. В этом разделе также объясняется понятие усилий, включая некоторые преимущества и недостатки предоставляет типы данных, удаленных из поставщика типов и обсуждается представлениями удаленных типов.


## <a name="a-type-provider-that-uses-static-parameters"></a>Тип поставщика, который использует статические параметры
Возможность параметризовать поставщиков типов, статические данные включает разнообразные сценарии, даже в случаях, когда поставщик не требуется доступ к любым локальным или удаленным данным. В этом разделе вы узнаете, как некоторые основные методы совместное размещение такой поставщик.


### <a name="type-checked-regex-provider"></a>Введите проверкой Regex-поставщик
Предположим, необходимо реализовать тип поставщика по использованию регулярных выражений, являющийся оболочкой для .NET `System.Text.RegularExpressions.Regex` библиотек в интерфейс, который предоставляет следующие гарантии компиляции:


- Проверка, является ли допустимым регулярным выражением.
<br />

- Предоставление именованных свойств на соответствий, которые основаны на любые имена групп в регулярном выражении.
<br />

В этом разделе показано, как использовать для создания поставщиков типов `RegExProviderType` введите параметризует, шаблон регулярного выражения, чтобы предоставить следующие преимущества. Компилятор выдает ошибку, если указанный шаблон не является допустимым, и поставщик типов можно извлечь группы из шаблона, чтобы вы могли обращаться к ним с помощью свойства совпадает с именем. При разработке поставщиков типов, можно как должен выглядеть без поддержки API-интерфейса для конечных пользователей и как преобразует такой подход в коде .NET. Приведенный ниже показано, как использовать такие API-Интерфейс для получения компонентов код области:

```fsharp
type T = RegexTyped< @"(?<AreaCode>^\d{3})-(?<PhoneNumber>\d{3}-\d{4}$)">
let reg = T()
let result = T.IsMatch("425-555-2345")
let r = reg.Match("425-555-2345").Group_AreaCode.Value //r equals "425"
```

В следующем примере показано, как поставщик типа преобразует эти вызовы:

```fsharp
let reg = new Regex(@"(?<AreaCode>^\d{3})-(?<PhoneNumber>\d{3}-\d{4}$)")
let result = reg.IsMatch("425-123-2345")
let r = reg.Match("425-123-2345").Groups.["AreaCode"].Value //r equals "425"
```

Обратите внимание на следующие моменты:


- Стандартный тип Regex представляет параметризованного `RegexTyped` типа.
<br />

- `RegexTyped` Конструктор приводит к вызову конструктора Regex, передавая статический тип аргумента для шаблона.
<br />

- Результаты `Match` метод представлены стандартные `System.Text.RegularExpressions.Match` типа.
<br />

- Каждой именованной группы приведет к предоставляемого свойства и обращение к свойству приводит к использованию индексатора на соответствие `Groups` коллекции.
<br />

В следующем коде показан основной логику для реализации такого поставщика, и в этом примере не включает добавление всех элементов в указанный тип. Сведения о каждом добавлен элемент см. Полный код, загрузите образец с [F # 3.0 образец пакета](http://fsharp3sample.codeplex.com) на веб-сайте Codeplex.

```fsharp
namespace Samples.FSharp.RegexTypeProvider

open System.Reflection
open Microsoft.FSharp.Core.CompilerServices
open Samples.FSharp.ProvidedTypes
open System.Text.RegularExpressions

[<TypeProvider>]
type public CheckedRegexProvider() as this =
inherit TypeProviderForNamespaces()

// Get the assembly and namespace used to house the provided types
let thisAssembly = Assembly.GetExecutingAssembly()
let rootNamespace = "Samples.FSharp.RegexTypeProvider"
let baseTy = typeof<obj>
let staticParams = [ProvidedStaticParameter("pattern", typeof<string>)]

let regexTy = ProvidedTypeDefinition(thisAssembly, rootNamespace, "RegexTyped", Some baseTy)

do regexTy.DefineStaticParameters(
parameters=staticParams, 
instantiationFunction=(fun typeName parameterValues ->

match parameterValues with 
| [| :? string as pattern|] -> 
// Create an instance of the regular expression. 
//
// This will fail with System.ArgumentException if the regular expression is not valid. 
// The exception will escape the type provider and be reported in client code.
let r = System.Text.RegularExpressions.Regex(pattern)            

// Declare the typed regex provided type.
// The type erasure of this type is 'obj', even though the representation will always be a Regex
// This, combined with hiding the object methods, makes the IntelliSense experience simpler.
let ty = ProvidedTypeDefinition(
thisAssembly, 
rootNamespace, 
typeName, 
baseType = Some baseTy)

...

ty
| _ -> failwith "unexpected parameter values")) 

do this.AddNamespace(rootNamespace, [regexTy])

[<TypeProviderAssembly>]
do ()
```

Обратите внимание на следующие моменты:


- Поставщик типа принимает два параметра статического: `pattern`, который является обязательным и `options`, которой являются необязательными (поскольку предоставляется значение по умолчанию).
<br />

- После статических аргументов, можно создать экземпляр регулярного выражения. Этот экземпляр вызовет исключение, если регулярное выражение имеет неправильный формат, и эта ошибка будет выводиться пользователям.
<br />

- В пределах `DefineStaticParameters` обратного вызова, определите тип, который будет возвращаться после аргументов.
<br />

- Этот код задает `HideObjectMethods` значение true, чтобы возможности IntelliSense будет по-прежнему упрощенное. Этот атрибут вызывает `Equals`, `GetHashCode`, `Finalize`, и `GetType` члены должны быть отключены из списка IntelliSense для предоставленного объекта.
<br />

- Вы используете `obj` как базовый тип метода, но вы будете использовать `Regex` объект как представление во время выполнения этого типа, как показано в следующем примере.
<br />

- Вызов `Regex` конструктор вызывает `System.ArgumentException` при регулярное выражение является недопустимым. Компилятор перехватывает это исключение и сообщение об ошибке для пользователя во время компиляции, а также в редакторе Visual Studio. Это исключение позволяет регулярные выражения для проверки без запуска приложения.
<br />

Так как она не содержит все значимые методов или свойств типа, определенного выше еще не полезно. Сначала добавьте статический `IsMatch` метод:

```fsharp
let isMatch = ProvidedMethod(
methodName = "IsMatch", 
parameters = [ProvidedParameter("input", typeof<string>)], 
returnType = typeof<bool>, 
IsStaticMethod = true,
InvokeCode = fun args -> <@@ Regex.IsMatch(%%args.[0], pattern) @@>) 

isMatch.AddXmlDoc "Indicates whether the regular expression finds a match in the specified input string." 
ty.AddMember isMatch
```

Предыдущий код определяет метод `IsMatch`, который принимает строку в качестве входных данных и возвращает `bool`. Единственной сложностью является использование класса `args` аргумент `InvokeCode` определения. В этом примере `args` является список предложений, который представляет аргументы для этого метода. Если метод является методом экземпляра, первый аргумент представляет `this` аргумент. Однако для статического метода, аргументы — все, что явно заданные аргументы в метод. Обратите внимание, соответствие тип значения, заключенные в кавычки указанный тип возвращаемого значения (в данном случае `bool`). Также Обратите внимание, что этот код использует `AddXmlDoc` метод, чтобы убедиться в том, что предоставленному методу также имеет полезные документации, доступной через IntelliSense можно указать.

Добавьте метод Match экземпляра. Тем не менее, этот метод должен возвращать значение предоставленного `Match` тип, так что группы можно получить доступ к строго типизированным образом. Таким образом, сначала объявить `Match` типа. Поскольку этот тип зависит от шаблон, который указан в качестве статического аргумента, этот тип должен быть вложен в определении параметризованного типа:

```fsharp
let matchTy = ProvidedTypeDefinition(
"MatchType", 
baseType = Some baseTy, 
HideObjectMethods = true)

ty.AddMember matchTy
```

Затем добавьте одно свойство в тип соответствия для каждой группы. Во время выполнения, соответствие представляется в виде `System.Text.RegularExpressions.Match` значения, поэтому необходимо использовать предложение, которое определяет свойство `System.Text.RegularExpressions.Match.Groups` индексированное свойство, чтобы получить соответствующую группу.

```fsharp
for group in r.GetGroupNames() do
// Ignore the group named 0, which represents all input.
if group <> "0" then
let prop = ProvidedProperty(
propertyName = group, 
propertyType = typeof<Group>, 
GetterCode = fun args -> <@@ ((%%args.[0]:obj) :?> Match).Groups.[group] @@>)
prop.AddXmlDoc(sprintf @"Gets the ""%s"" group from this match" group)
matchTy.AddMember prop
```

Обратите внимание, что вы добавляете XML-документации для указанного свойства. Также Обратите внимание, что свойство доступно для чтения, если `GetterCode` предоставляется функции и свойства могут записываться при `SetterCode` функция предоставляется, поэтому результирующее свойство только для чтения.

Теперь можно создать метод экземпляра, который возвращает значение этого `Match` типа:

```fsharp
let matchMethod = 
ProvidedMethod(
methodName = "Match", 
parameters = [ProvidedParameter("input", typeof<string>)], 
returnType = matchTy, 
InvokeCode = fun args -> <@@ ((%%args.[0]:obj) :?> Regex).Match(%%args.[1]) :> obj @@>)
matchMeth.AddXmlDoc "Searches the specified input string for the first ocurrence of this regular expression" 

ty.AddMember matchMeth
```

Поскольку создается методом экземпляра `args.[0]` представляет `RegexTyped` экземпляр, на котором вызывается метод, и `args.[1]` является входного аргумента.

Наконец предоставляет конструктор для создания экземпляров указанного типа.

```fsharp
let ctor = ProvidedConstructor(
parameters = [], 
InvokeCode = fun args -> <@@ Regex(pattern, options) :> obj @@>)
ctor.AddXmlDoc("Initializes a regular expression instance.")

ty.AddMember ctor
```

Просто стирает конструктор для создания стандартных экземпляров регулярных выражений .NET упаковывается в объект снова, поскольку `obj` является стирания предоставленного типа. С данным изменением пример использования API, указанном ранее в этом разделе работает надлежащим образом. Ниже приведен полный и последнем:

```fsharp
namespace Samples.FSharp.RegexTypeProvider

open System.Reflection
open Microsoft.FSharp.Core.CompilerServices
open Samples.FSharp.ProvidedTypes
open System.Text.RegularExpressions

[<TypeProvider>]
type public CheckedRegexProvider() as this =
inherit TypeProviderForNamespaces()

// Get the assembly and namespace used to house the provided types.
let thisAssembly = Assembly.GetExecutingAssembly()
let rootNamespace = "Samples.FSharp.RegexTypeProvider"
let baseTy = typeof<obj>
let staticParams = [ProvidedStaticParameter("pattern", typeof<string>)]

let regexTy = ProvidedTypeDefinition(thisAssembly, rootNamespace, "RegexTyped", Some baseTy)

do regexTy.DefineStaticParameters(
parameters=staticParams, 
instantiationFunction=(fun typeName parameterValues ->

match parameterValues with 
| [| :? string as pattern|] -> 
// Create an instance of the regular expression. 




let r = System.Text.RegularExpressions.Regex(pattern)            

// Declare the typed regex provided type.



let ty = ProvidedTypeDefinition(
thisAssembly, 
rootNamespace, 
typeName, 
baseType = Some baseTy)

ty.AddXmlDoc "A strongly typed interface to the regular expression '%s'"

// Provide strongly typed version of Regex.IsMatch static method.
let isMatch = ProvidedMethod(
methodName = "IsMatch", 
parameters = [ProvidedParameter("input", typeof<string>)], 
returnType = typeof<bool>, 
IsStaticMethod = true,
InvokeCode = fun args -> <@@ Regex.IsMatch(%%args.[0], pattern) @@>) 

isMatch.AddXmlDoc "Indicates whether the regular expression finds a match in the specified input string"

ty.AddMember isMatch

// Provided type for matches
// Again, erase to obj even though the representation will always be a Match
let matchTy = ProvidedTypeDefinition(
"MatchType", 
baseType = Some baseTy, 
HideObjectMethods = true)

// Nest the match type within parameterized Regex type.
ty.AddMember matchTy

// Add group properties to match type
for group in r.GetGroupNames() do
// Ignore the group named 0, which represents all input.
if group <> "0" then
let prop = ProvidedProperty(
propertyName = group, 
propertyType = typeof<Group>, 
GetterCode = fun args -> <@@ ((%%args.[0]:obj) :?> Match).Groups.[group] @@>)
prop.AddXmlDoc(sprintf @"Gets the ""%s"" group from this match" group)
matchTy.AddMember(prop)

// Provide strongly typed version of Regex.Match instance method.
let matchMeth = ProvidedMethod(
methodName = "Match", 
parameters = [ProvidedParameter("input", typeof<string>)], 
returnType = matchTy, 
InvokeCode = fun args -> <@@ ((%%args.[0]:obj) :?> Regex).Match(%%args.[1]) :> obj @@>)
matchMeth.AddXmlDoc "Searches the specified input string for the first occurence of this regular expression"

ty.AddMember matchMeth

// Declare a constructor.
let ctor = ProvidedConstructor(
parameters = [], 
InvokeCode = fun args -> <@@ Regex(pattern) :> obj @@>)

// Add documentation to the constructor.
ctor.AddXmlDoc "Initializes a regular expression instance"

ty.AddMember ctor

ty
| _ -> failwith "unexpected parameter values")) 

do this.AddNamespace(rootNamespace, [regexTy])

[<TypeProviderAssembly>]
do ()
```

`Key Lessons`

В этом разделе описано создание поставщика типов, который работает на его статических параметров. Поставщик проверяет параметр static и предоставляет операции, основанные на его значение.


## <a name="a-type-provider-that-is-backed-by-local-data"></a>Поставщик типов, поддерживаемый локальные данные
Часто можно поставщиков типов для представления API, основываясь на не только статические параметры, но данные из локальных или удаленных систем. В этом разделе рассматриваются поставщиков типов, основанных на локальных данных, например локальные файлы данных.


### <a name="simple-csv-file-provider"></a>Поставщик простой CSV-файла
В качестве простого примера рассмотрим поставщика типов для доступа к научных данных в формате с разделителями запятыми значения (CSV). В этом разделе предполагается, что CSV-файлы содержат строку заголовка, следуют данных с плавающей запятой, как показано в следующей таблице:



|Расстояние (счетчик)|Время (сек)|
|----------------|-------------|
|50.0|3.7|
|100.0|5.2|
|150.0|6.4|
В этом разделе показано, как указать тип, который можно использовать для получения строк с `Distance` свойство типа `float<meter>` и `Time` свойство типа `float<second>`. Для простоты предполагается следующее:


- Имена заголовков являются либо меньше единицы или иметь формат «Имя (единицы)» и не могут содержать запятые.
<br />

- Единицы измерения — это все единицы международной Systeme (SI) как [Microsoft.FSharp.Data.UnitSystems.SI.UnitNames модуль (F #)](https://msdn.microsoft.com/library/3cb43485-11f5-4aa7-a779-558f19d4013b) определяется модулем.
<br />

- Единицы просты (например, контролировать), а не составные (например, индикатор в секунду).
<br />

- Все столбцы содержат данных с плавающей запятой.
<br />

Более полный поставщик будет ослабить эти ограничения.

Снова первым делом следует учитывать, как должен выглядеть интерфейс API. Получает `info.csv` файла с содержимым из предыдущей таблицы (в формате с разделителями запятыми), пользователи поставщика должны иметь возможность писать код, аналогичный следующему примеру:

```fsharp
let info = new MiniCsv<"info.csv">()
for row in info.Data do
let time = row.Time
printfn "%f" (float time)
```

В этом случае компилятор должен преобразовать эти вызовы в нечто похожее на следующий пример:

```fsharp
let info = new MiniCsvFile("info.csv")
for row in info.Data do
let (time:float) = row.[1]
printfn "%f" (float time)
```

Оптимальной преобразования требуется поставщик типов для определения реальную `CsvFile` типа в сборке поставщика типов. Поставщики типов часто используют несколько вспомогательные типы и методы программы-оболочки для важных логических. Поскольку меры будут удалены во время выполнения, можно использовать `float[]` как уничтоженные тип строки. Как разные измерения типа компилятор будет рассматривать различных столбцов. Например, первый столбец в нашем примере имеет тип `float<meter>`, а второй — `float<second>`. Однако вместо удаленных представление может оставаться довольно просто.

В следующем коде показано ядро реализации.

```fsharp
// Simple type wrapping CSV data
type CsvFile(filename) =
// Cache the sequence of all data lines (all lines but the first)
let data = 
seq { for line in File.ReadAllLines(filename) |> Seq.skip 1 do
yield line.Split(',') |> Array.map float }
|> Seq.cache
member __.Data = data

[<TypeProvider>]
type public MiniCsvProvider(cfg:TypeProviderConfig) as this =
inherit TypeProviderForNamespaces()

// Get the assembly and namespace used to house the provided types.
let asm = System.Reflection.Assembly.GetExecutingAssembly()
let ns = "Samples.FSharp.MiniCsvProvider"

// Create the main provided type.
let csvTy = ProvidedTypeDefinition(asm, ns, "MiniCsv", Some(typeof<obj>))

// Parameterize the type by the file to use as a template.
let filename = ProvidedStaticParameter("filename", typeof<string>)
do csvTy.DefineStaticParameters([filename], fun tyName [| :? string as filename |] ->

// Resolve the filename relative to the resolution folder.
let resolvedFilename = Path.Combine(cfg.ResolutionFolder, filename)

// Get the first line from the file.
let headerLine = File.ReadLines(resolvedFilename) |> Seq.head

// Define a provided type for each row, erasing to a float[].
let rowTy = ProvidedTypeDefinition("Row", Some(typeof<float[]>))

// Extract header names from the file, splitting on commas.
// use Regex matching to get the position in the row at which the field occurs
let headers = Regex.Matches(headerLine, "[^,]+")

// Add one property per CSV field.
for i in 0 .. headers.Count - 1 do
let headerText = headers.[i].Value

// Try to decompose this header into a name and unit.
let fieldName, fieldTy =
let m = Regex.Match(headerText, @"(?<field>.+) \((?<unit>.+)\)")
if m.Success then


let unitName = m.Groups.["unit"].Value
let units = ProvidedMeasureBuilder.Default.SI unitName
m.Groups.["field"].Value, ProvidedMeasureBuilder.Default.AnnotateType(typeof<float>,[units])


else
// no units, just treat it as a normal float
headerText, typeof<float>

let prop = ProvidedProperty(fieldName, fieldTy, 
GetterCode = fun [row] -> <@@ (%%row:float[]).[i] @@>)

// Add metadata that defines the property's location in the referenced file.
prop.AddDefinitionLocation(1, headers.[i].Index + 1, filename)
rowTy.AddMember(prop) 

// Define the provided type, erasing to CsvFile.
let ty = ProvidedTypeDefinition(asm, ns, tyName, Some(typeof<CsvFile>))

// Add a parameterless constructor that loads the file that was used to define the schema.
let ctor0 = ProvidedConstructor([], 
InvokeCode = fun [] -> <@@ CsvFile(resolvedFilename) @@>)
ty.AddMember ctor0

// Add a constructor that takes the file name to load.
let ctor1 = ProvidedConstructor([ProvidedParameter("filename", typeof<string>)], 
InvokeCode = fun [filename] -> <@@ CsvFile(%%filename) @@>)
ty.AddMember ctor1

// Add a more strongly typed Data property, which uses the existing property at runtime.
let prop = ProvidedProperty("Data", typedefof<seq<_>>.MakeGenericType(rowTy), 
GetterCode = fun [csvFile] -> <@@ (%%csvFile:CsvFile).Data @@>)
ty.AddMember prop

// Add the row type as a nested type.
ty.AddMember rowTy
ty)

// Add the type to the namespace.
do this.AddNamespace(ns, [csvTy])
```

Обратите внимание на следующие аспекты реализации:


- Перегруженные конструкторы позволяют исходный файл или ту, которая имеет идентичные схемы для чтения. Этот вариант является стандартным при написании поставщика типа для источников данных на локальном или удаленном, и этот шаблон позволяет локальный файл для использования в качестве шаблона для удаленных данных.
<br />  Можно использовать [TypeProviderConfig](https://msdn.microsoft.com/library/1cda7b9a-3d07-475d-9315-d65e1c97eb44) значение, которое передается в конструктор поставщика типа для разрешения имен с относительным путем к файлу.
<br />

- Можно использовать `AddDefinitionLocation` метод для определения местоположения указанные свойства. Таким образом Если вы используете `Go To Definition` предоставленное свойство CSV-файл открывается в Visual Studio.
<br />

- Можно использовать `ProvidedMeasureBuilder` типов для поиска SI единицы и создания соответствующего `float<_>` типов.
<br />

`Key Lessons`

В этом разделе описано создание поставщика типов для локальный источник данных с простой схемой, которая содержится в самого источника данных.


## <a name="going-further"></a>Если продолжить
В следующих разделах приведены рекомендации для дальнейшего изучения.


### <a name="a-look-at-the-compiled-code-for-erased-types"></a>Рассмотрим скомпилированный код для удаленных типов
Чтобы получить представление об как использование поставщика типов соответствует коду, который создается, рассмотрим следующую функцию с помощью `HelloWorldTypeProvider` , используемый этой статьи.

```fsharp
let function1 () = 
let obj1 = Samples.HelloWorldTypeProvider.Type1("some data")
obj1.InstanceProperty
```

Ниже приведен результирующий код, с помощью ildasm.exe decompiled изображения.

```
.class public abstract auto ansi sealed Module1
extends [mscorlib]System.Object
{
.custom instance void [FSharp.Core]Microsoft.FSharp.Core.CompilationMappingAtt
ribute::.ctor(valuetype [FSharp.Core]Microsoft.FSharp.Core.SourceConstructFlags)
= ( 01 00 07 00 00 00 00 00 )
.method public static int32  function1() cil managed
{
// Code size       24 (0x18)
.maxstack  3
.locals init ([0] object obj1)
IL_0000:  nop
IL_0001:  ldstr      "some data"
IL_0006:  unbox.any  [mscorlib]System.Object
IL_000b:  stloc.0
IL_000c:  ldloc.0
IL_000d:  call       !!0 [FSharp.Core_2]Microsoft.FSharp.Core.LanguagePrimit
ives/IntrinsicFunctions::UnboxGeneric<string>(object)
IL_0012:  callvirt   instance int32 [mscorlib_3]System.String::get_Length()
IL_0017:  ret
} // end of method Module1::function1

} // end of class Module1
```

Как показано в примере, все упоминания типа `Type1` и `InstanceProperty` удалены свойство, оставляя участвующих только операции с типами среды выполнения.


### <a name="design-and-naming-conventions-for-type-providers"></a>Проектирование и соглашения об именовании для поставщиков типов
Соблюдайте следующие соглашения для создания поставщиков типов.


- `Providers for Connectivity Protocols`
<br />  Как правило, имена для большинства DLL-библиотеки поставщика для протоколов подключения данных и служб, например соединения OData или SQL, должна завершиться в `TypeProvider` или `TypeProviders`. Например используйте имя библиотеки DLL, примерно следующего вида:
<br />

```
  Fabrikam.Management.BasicTypeProviders.dll
```

  Убедитесь, что ваш предоставляемых типов являются членами соответствующее пространство имен и указать протокол связи, реализованной:
<br />

```
  Fabrikam.Management.BasicTypeProviders.WmiConnection<…>
  Fabrikam.Management.BasicTypeProviders.DataProtocolConnection<…>
```

- `Utility Providers for General Coding`
<br />  Для программы поставщика типов, например для регулярных выражений поставщик типов может быть частью базовой библиотеки, как показано в следующем примере:
<br />

```fsharp
  #r "Fabrikam.Core.Text.Utilities.dll"
```

  В этом случае предоставленный тип будет выглядеть в соответствующий момент согласно обычным соглашения разработки .NET:
<br />

```fsharp
  open Fabrikam.Core.Text.RegexTyped
  
  let regex = new RegexTyped<"a+b+a+b+">()
```

- `Singleton Data Sources`
<br />  Некоторые поставщики типов подключения для одного выделенного источника данных и для предоставления только данные. В этом случае следует удалить `TypeProvider` суффикс и используйте обычный соглашения для .NET:
<br />

```fsharp
  #r "Fabrikam.Data.Freebase.dll"
  
  let data = Fabrikam.Data.Freebase.Astronomy.Asteroids
```

  Дополнительные сведения см. в разделе `GetConnection` разработать соглашение, которое описано далее в этом разделе.
<br />


### <a name="design-patterns-for-type-providers"></a>Шаблоны разработки для поставщиков типов
В следующих разделах описаны принципы разработки, которые можно использовать при разработке поставщиков типов.


#### <a name="the-getconnection-design-pattern"></a>Шаблон разработки GetConnection
Большинство поставщиков типов должны быть написаны для использования `GetConnection` шаблон, используемый поставщиками типов в FSharp.Data.TypeProviders.dll, как показано в следующем примере:

```fsharp
#r "Fabrikam.Data.WebDataStore.dll"

type Service = Fabrikam.Data.WebDataStore<…static connection parameters…>

let connection = Service.GetConnection(…dynamic connection parameters…)

let data = connection.Astronomy.Asteroids
```

#### <a name="type-providers-backed-by-remote-data-and-services"></a>Поставщики типов удаленных данных и служб
Прежде чем создать поставщик типов, который содержится в удаленных данных и служб, необходимо учитывать ряд проблем, которые принадлежат подключенных программирования. Эти проблемы включают следующее:


- Сопоставление схем
<br />

- liveness и недействительности при наличии изменений схемы
<br />

- Кэширование схем
<br />

- реализации асинхронных операций доступа к данным
<br />

- Поддержка запросов, включая запросы LINQ
<br />

- учетные данные и проверки подлинности
<br />

В этом разделе не исследовать эти дополнительные проблемы.


### <a name="additional-authoring-techniques"></a>Дополнительные методы разработки
При написании собственных поставщиков типов, можно использовать следующие дополнительные методы.


- `Creating Types and Members On-Demand`
<br />  ProvidedType API откладывала версий AddMember.
<br />

```fsharp
  type ProvidedType =
  member AddMemberDelayed  : (unit -> MemberInfo)      -> unit
  member AddMembersDelayed : (unit -> MemberInfo list) -> unit
```

  Эти версии используются для создания пространства по требованию типов.
<br />

- `Providing Array, ByRef, and Pointer types`
<br />  Сделать указанных членов, (, подписи включают типы массивов, типы byref и создание экземпляров универсальных типов) с помощью нормали `MakeArrayType`, `MakePointerType`, и `MakeGenericType` на любом экземпляре System.Type, включая `ProvidedTypeDefinitions`.
<br />

- `Providing Unit of Measure Annotations`
<br />  ProvidedTypes API предоставляет вспомогательные методы для предоставления мер заметок. Например, чтобы указать тип `float<kg>`, используйте следующий код:
<br />

```fsharp
  let measures = ProvidedMeasureBuilder.Default
  let kg = measures.SI "kilogram"
  let m = measures.SI "meter"
  let float_kg = measures.AnnotateType(typeof<float>,[kg])
```

  Чтобы указать тип `Nullable<decimal<kg/m^2>>`, используйте следующий код:
<br />

```fsharp
  let kgpm2 = measures.Ratio(kg, measures.Square m)
  let dkgpm2 = measures.AnnotateType(typeof<decimal>,[kgpm2])
  let nullableDecimal_kgpm2 = typedefof<System.Nullable<_>>.MakeGenericType [|dkgpm2 |]
```

- `Accessing Project-Local or Script-Local Resources`
<br />  Каждый экземпляр поставщика типа может быть задан `TypeProviderConfig` значение во время построения. Это значение содержит «разрешения папки» для поставщика (то есть папка проекта для компиляции или каталог, содержащий сценарий), список сборок, на которые имеются ссылки и другие сведения.
<br />

- `Invalidation`
<br />  Поставщики могут вызывать сигналы о недействительности уведомить службу F #, которые могли изменить предположения схемы. При возникновении недействительности typecheck Повторено, если поставщик находится в Visual Studio. Этот сигнал будет игнорироваться, если поставщик размещается в F # Interactive или компилятором F # (fsc.exe).
<br />

- `Caching Schema Information`
<br />  Поставщики часто необходимо кэшировать доступ к сведениям о схеме. Кэшированные данные должны храниться с помощью имени файла, которое задано как статического параметра или пользовательские данные. Кэширование схем является примером `LocalSchemaFile` параметра в тип поставщики `FSharp.Data.TypeProviders` сборки. В реализации этих поставщиков этот параметр статических направляет поставщик типов, чтобы использовать сведения о схеме в указанный локальный файл вместо обращения к источнику данных по сети. Для использования кэшированных сведений о схеме, необходимо задать параметр static `ForceUpdate` для `false`. Подобный прием можно использовать для доступа к данным и отключенные от сети.
<br />

- `Backing Assembly`
<br />  При компиляции файл .dll или .exe, резервное копирование DLL-файла для создаваемых типов статически связанная с результирующей сборки. Эта ссылка создается путем копирования определения типов промежуточного языка (IL) и все управляемые ресурсы из сборки резервное копирование в окончательную сборку. При использовании F # Interactive, резервное копирование DLL-файл не копируется и вместо него загружается непосредственно в F # Interactive процесса.
<br />

- `Exceptions and Diagnostics from Type Providers`
<br />  Все случаи использования все элементы из предоставляемых типов могут вызывать исключения. Во всех случаях если поставщик типа вызывает исключение, компилятор узла атрибутов ошибки поставщику определенного типа.
<br />
  - Тип поставщика исключения никогда не должны появиться внутренних ошибок компилятора.
<br />

  - Поставщики типов не может сообщить предупреждения.
<br />

  - Поставщик типа размещается в компиляторе F # среды разработки F # и F # Interactive, перехватывает все исключения из этого поставщика. Свойство сообщения всегда является текст ошибки, и отображается не трассировку стека. Если исключение, можно вызывать следующие примеры:
<br />
    - `System.NotSupportedException`
<br />

    - `System.IO.IOException`
<br />

    - `System.Exception`
<br />


#### <a name="providing-generated-types"></a>Предоставляя типы, созданные
Таким образом, в этом документе содержатся как обеспечить удаленных типов. Также можно использовать механизм поставщика типов в языке F # для предоставления созданных типов, которые добавляются в виде реального определений типов .NET в программу пользователей. Вы должны ссылаться на созданный типы, предоставляемые с помощью определения типа.

```fsharp
open Microsoft.FSharp.TypeProviders 

type Service = ODataService<" http://services.odata.org/Northwind/Northwind.svc/">
```

0,2 ProvidedTypes вспомогательный код, который является частью версии F # 3.0 имеется только ограниченная поддержка для предоставления создаваемых типов. Для определения создаваемого типа должны выполняться следующие инструкции:


- Должно быть присвоено IsErased `false`.
<br />

- Поставщик должен иметь сборку, которая содержит фактические резервного .NET DLL-файл, соответствующий файл DLL-файл на диске.
<br />

Кроме того, необходимо вызвать метод `ConvertToGenerated` на указанный корневой тип, вложенные типы формируют закрытых набор создаваемых типов. Этот вызов создает определение данного типа, предоставленный и его вложенные определения типов в сборку, а также корректируются `Assembly` свойства всех определений предоставленного типа, чтобы вернуться на эту сборку. Сборка создается только в том случае, когда свойство сборки в корневом типе осуществляется в первый раз. При обработке объявление generative типа для типа, компилятор F # узла доступ к этому свойству.


## <a name="rules-and-limitations"></a>Правила и ограничения
При написании поставщиков типов, примите во внимание следующие правила и ограничения.


- `Provided types must be reachable.`
<br />  Все входящие типы должны быть доступны из типов-nested. Типы, не являющегося вложенным, получают в вызове `TypeProviderForNamespaces` конструктор или вызов `AddNamespace`. Например, если поставщик предоставляет тип `StaticClass.P : T`, необходимо убедиться в том, что T невложенном типе или вложенными в один.
<br />  Например, некоторые поставщики иметь статического класса, такие как `DataTypes` , содержащие такие `T1, T2, T3, ...` типов. В противном случае ошибка сообщает, что была обнаружена ссылка на тип "T" в сборке A, но не удалось найти тип в этой сборке. При появлении этой ошибки, убедитесь, что все подтипы можно получить от поставщика типов. Примечание: Эти `T1, T2, T3...` типы называются *на лету* типов. Не забудьте поместить их в доступного пространства имен или родительского типа.
<br />

- `Limitations of the Type Provider Mechanism`
<br />  Механизм поставщика типов в языке F # имеет следующие ограничения:
<br />
  - Базовой инфраструктуры для поставщиков типов F # не поддерживает универсальные типы или универсальные методы.
<br />

  - Механизм не поддерживает вложенные типы с указанием статических параметров.
<br />

- `Limitations of the ProvidedTypes Support Code`
<br />  `ProvidedTypes` Код поддержки действуют следующие правила и ограничения:
<br />
  1. Указанные свойства с индексированным методы get и Set не реализованы.
<br />

  2. Если события не реализованы.
<br />

  3. Предоставляемых типов и сведений об объектах следует использовать только для механизм поставщика типов в языке F #. Они не являются более пригодной как объектов System.Type.
<br />

  4. Конструкции, которые можно использовать в кавычках, определяющие способ реализации применяется несколько ограничений. Изучите исходный код для ProvidedTypes -*версии* чтобы увидеть, какие конструкции поддерживаются в кавычках.
<br />

- `Type providers must generate output assemblies that are .dll files, not .exe files.`
<br />


## <a name="development-tips"></a>Советы по разработке
Следующие советы могут оказаться полезными в процессе разработки.


- `Run Two Instances of Visual Studio.`Можно разрабатывать в одном экземпляре типа поставщика и проверки поставщика в другом, поскольку интегрированная среда разработки тест будет иметь блокировку на DLL-файл, предотвращает перестраивает поставщика типов. Таким образом необходимо закрыть второй экземпляр Visual Studio, пока поставщик встроен в первом экземпляре, а затем необходимо повторно открыть второй экземпляр после построения поставщика.
<br />

- `Debug type providers by using invocations of fsc.exe.`Поставщики типов можно вызвать с помощью следующих средств:
<br />
  - fsc.exe (компилятор командной строки F #)
<br />

  - fsi.exe (F # Interactive компилятор)
<br />

  - devenv.exe (Visual Studio)
<br />

  Поставщики типов часто можно отлаживать проще всего с помощью fsc.exe на файл скрипта теста (например, файл script.fsx). Вы можете запустить отладчик из командной строки.
<br />

```
  devenv /debugexe fsc.exe script.fsx
```

  Можно использовать ведение журнала печать в stdout.
<br />


## <a name="see-also"></a>См. также
[Поставщики типов](index.md)