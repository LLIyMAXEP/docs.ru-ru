---
title: "Ресурсы XAML"
ms.custom: 
ms.date: 03/30/2017
ms.prod: .net-framework
ms.reviewer: 
ms.suite: 
ms.technology: dotnet-wpf
ms.tgt_pltfrm: 
ms.topic: article
helpviewer_keywords:
- reusing resources [WPF]
- resources [WPF], reusing
- reusing commonly defined objects [WPF]
- XAML [WPF], reusing resources
ms.assetid: 91580b89-a0a8-4889-aecb-fddf8e63175f
caps.latest.revision: "33"
author: dotnet-bot
ms.author: dotnetcontent
manager: wpickett
ms.workload: dotnet
ms.openlocfilehash: a2d58802bcdfa57bb7689e7406651fcc9829a7e5
ms.sourcegitcommit: 16186c34a957fdd52e5db7294f291f7530ac9d24
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/22/2017
---
# <a name="xaml-resources"></a>Ресурсы XAML
Ресурс — это объект, который можно повторно использовать в разных местах приложения. Примерами ресурсов являются кисти и стили. В этом обзоре описывается использование ресурсов в [!INCLUDE[TLA2#tla_xaml](../../../../includes/tla2sharptla-xaml-md.md)]. Можно также создать и доступ к ресурсам с помощью кода или попеременно кода и [!INCLUDE[TLA#tla_xaml](../../../../includes/tlasharptla-xaml-md.md)]. Дополнительные сведения см. в разделе [ресурсы и код](../../../../docs/framework/wpf/advanced/resources-and-code.md).  
  
> [!NOTE]
>  Файлы ресурсов, описанные в этом разделе отличаются от описанных файлы ресурсов в [ресурса приложения WPF, содержимое и файлы данных](../../../../docs/framework/wpf/app-development/wpf-application-resource-content-and-data-files.md) и отличаются от внедренных или связанных ресурсов, приведенных в [ Управление ресурсами приложения (.NET)](http://msdn.microsoft.com/library/f2582734-8ada-4baa-8a7c-e2ef943ddf7e).  
  
  
<a name="usingresources"></a>   
## <a name="using-resources-in-xaml"></a>Использование ресурсов в XAML  
 В следующем примере определяется <xref:System.Windows.Media.SolidColorBrush> как ресурс в корневом элементе страницы. Затем ссылается на ресурс и использует его для задания свойств нескольких дочерних элементов, включая <xref:System.Windows.Shapes.Ellipse>, <xref:System.Windows.Controls.TextBlock>и <xref:System.Windows.Controls.Button>.  
  
 [!code-xaml[FEResourceSH_snip#XAML](../../../../samples/snippets/csharp/VS_Snippets_Wpf/FEResourceSH_snip/CS/page1.xaml#xaml)]  
  
 Каждый элемент уровня платформы (<xref:System.Windows.FrameworkElement> или <xref:System.Windows.FrameworkContentElement>) имеет <xref:System.Windows.FrameworkElement.Resources%2A> свойство, которое является свойство, которое содержит ресурсы (как <xref:System.Windows.ResourceDictionary>), определяющий ресурса. Вы можете определить ресурсы в любом элементе. Тем не менее, ресурсы, наиболее часто определяются в корневом элементе, который является <xref:System.Windows.Controls.Page> в примере.  
  
 Каждый ресурс в словаре ресурсов должен иметь уникальный ключ. При определении ресурсов в разметке, уникальный ключ через назначается [директива x: Key](../../../../docs/framework/xaml-services/x-key-directive.md). Как правило, ключ является строкой. Однако можно задать его как другой тип объекта с помощью соответствующих расширений разметки. Нестроковые ключи для ресурсов используются определенные функциональные области в [!INCLUDE[TLA2#tla_winclient](../../../../includes/tla2sharptla-winclient-md.md)], особенно для стилей, компонент ресурсов и стилей данных.  
  
 После определения ресурса можно ссылаться на ресурс, который должен быть использован для значения свойства, с помощью синтаксиса расширения разметки ресурсов, задающего имя ключа, например:  
  
 [!code-xaml[FEResourceSH_snip#KeyNameUsage](../../../../samples/snippets/csharp/VS_Snippets_Wpf/FEResourceSH_snip/CS/page2.xaml#keynameusage)]  
  
 В предыдущем примере когда [!INCLUDE[TLA2#tla_xaml](../../../../includes/tla2sharptla-xaml-md.md)] загрузчика обрабатывает значение `{StaticResource MyBrush}` для <xref:System.Windows.Controls.Control.Background%2A> свойство <xref:System.Windows.Controls.Button>, логика поиска ресурсов сначала проверяет наличие в словаре ресурсов <xref:System.Windows.Controls.Button> элемент. Если <xref:System.Windows.Controls.Button> нет определения ключа ресурса `MyBrush` (это не так, его коллекция ресурсов пуста), затем подстановка проверяет родительский элемент <xref:System.Windows.Controls.Button>, который является <xref:System.Windows.Controls.Page>. Таким образом, при определении ресурса в <xref:System.Windows.Controls.Page> корневой элемент, все элементы в логическом дереве <xref:System.Windows.Controls.Page> к нему возможен доступ, и можно повторно использовать тот же ресурс для задания значения любого свойства, принимает <xref:System.Type> , ресурс представляет. В предыдущем примере, то `MyBrush` ресурсов задает два различных свойства: <xref:System.Windows.Controls.Control.Background%2A> из <xref:System.Windows.Controls.Button>и <xref:System.Windows.Shapes.Shape.Fill%2A> из <xref:System.Windows.Shapes.Rectangle>.  
  
<a name="staticdynamic"></a>   
## <a name="static-and-dynamic-resources"></a>Статические и динамические ресурсы  
 Ресурс может использоваться как статический или динамический ресурс. Это делается с помощью [StaticResource Markup Extension](../../../../docs/framework/wpf/advanced/staticresource-markup-extension.md) или [DynamicResource Markup Extension](../../../../docs/framework/wpf/advanced/dynamicresource-markup-extension.md). Расширения разметки — это функция [!INCLUDE[TLA2#tla_xaml](../../../../includes/tla2sharptla-xaml-md.md)] , который предоставляет ссылку на объект можно указать, что расширение разметки обработки строки атрибута и возвращает объект для [!INCLUDE[TLA2#tla_xaml](../../../../includes/tla2sharptla-xaml-md.md)] загрузчика. Дополнительные сведения о поведении расширения разметки см. в разделе [расширения разметки и WPF XAML](../../../../docs/framework/wpf/advanced/markup-extensions-and-wpf-xaml.md).  
  
 При использовании расширения разметки обычно предоставляют один или несколько параметров в виде строки; они обрабатываются этим расширением разметки, а не в контексте задаваемого свойства. [StaticResource Markup Extension](../../../../docs/framework/wpf/advanced/staticresource-markup-extension.md) обрабатывает ключ путем поиска значения для этого ключа во всех доступных словарях ресурсов. Это происходит во время загрузки, которая является моментом времени, когда процессу загрузки необходимо присвоить значение свойству, которое принимает ссылку на статический ресурс. [DynamicResource Markup Extension](../../../../docs/framework/wpf/advanced/dynamicresource-markup-extension.md) вместо процессы ключ путем создания выражения и выражения остается необработанным пока приложение фактически не будет запущено, в этот момент выражение вычисляется и предоставляет значение.  
  
 Приведенные ниже соображения могут повлиять на выбор использования ссылки на статический или динамический ресурс.  
  
-   Общая схема способ создания ресурсов для приложения (постранично, в приложении, в свободном [!INCLUDE[TLA2#tla_xaml](../../../../includes/tla2sharptla-xaml-md.md)], в сборке только ресурсов).  
  
-   Является ли обновление ресурсов в режиме реального времени частью требований приложения.  
  
-   Соответствующее поведение подстановки этого ссылочного типа ресурса.  
  
-   Конкретное свойство или тип ресурса и собственное поведение этих типов.  
  
### <a name="static-resources"></a>Статические ресурсы  
 Ссылки на статические ресурсы лучше всего использовать в указанных ниже случаях.  
  
-   Когда структура приложения концентрирует большую часть его ресурсов в словарях ресурсов уровня страницы или приложения. Ссылки на статические ресурсы не обрабатываются повторно в зависимости от поведения во время выполнения, например при перезагрузке страницы. Таким образом, можно получить выигрыш в производительности благодаря отсутствию большого количества динамических ссылок на ресурсы, когда согласно структуре ресурсов и приложения в них нет необходимости.  
  
-   Значение свойства, не принадлежащем <xref:System.Windows.DependencyObject> или <xref:System.Windows.Freezable>.  
  
-   При создании словаря ресурсов, который будет скомпилирован в DLL и либо упакован как часть приложения, либо совместно использоваться приложениями.  
  
-   При создании темы для пользовательского элемента управления и определении ресурсов, используемых в темах. В этом случае обычно не требуется подстановка ссылок на динамические ресурсы. Вместо этого требуется подстановка ссылок на статические ресурсы, чтобы подстановка была прогнозируемой и самодостаточной для темы. При использовании ссылки на динамический ресурс даже ссылка в теме остается необработанной до времени выполнения и есть вероятность того, что при применении темы некоторые локальные элементы переопределят ключ, на который тема пытается сослаться, и локальный элемент окажется перед темой при подстановке. Если такое случится, тема будет работать не так, как ожидалось.  
  
-   При использовании ресурсов для задания большого количества свойств зависимостей. Свойства зависимостей обладают эффективным кэшированием значений, обеспечиваемым системой свойств, поэтому, если предоставить свойству зависимости значение, которое может быть вычислено во время загрузки, свойству не потребуется проверять пересчитанное выражение и оно сможет вернуть последнее фактическое значение. Этот метод может обеспечить выигрыш в производительности.  
  
-   Необходимо изменить основной ресурс для всех объектов-получателей, или вы хотите поддерживать отдельные записываемые экземпляры для каждого объекта-получателя с помощью [x: Shared Attribute](../../../../docs/framework/xaml-services/x-shared-attribute.md).  
  
#### <a name="static-resource-lookup-behavior"></a>Поведение подстановки статического ресурса  
  
1.  Процесс подстановки ищет запрошенный ключ в словаре ресурсов, определенном элементом, который устанавливает это свойство.  
  
2.  Затем процесс подстановки обходит логическое дерево снизу вверх до родительского элемента и его словаря ресурсов. Это продолжается до тех пор, пока не будет достигнут корневой элемент.  
  
3.  Далее проверяются ресурсы приложения. Ресурсы приложения — это ресурсы в словарь ресурсов, который определяется <xref:System.Windows.Application> объекта для вашего [!INCLUDE[TLA2#tla_winclient](../../../../includes/tla2sharptla-winclient-md.md)] приложения.  
  
 Ссылки на статический ресурс из словаря ресурсов должны указывать на ресурс, уже определенный лексически до ссылки на ресурс. Опережающие ссылки не могут быть разрешены ссылкой на статический ресурс. По этой причине, если используются ссылки на статический ресурс, нужно так разрабатывать структуру словаря ресурсов, чтобы ресурсы, предназначенные для использования ресурсами, определялись в начале каждого соответствующего словаря ресурсов или около него.  
  
 Подстановку статических ресурсов можно расширить в темы или в системные ресурсы, но это поддерживается только потому, что [!INCLUDE[TLA2#tla_xaml](../../../../includes/tla2sharptla-xaml-md.md)] загрузчик задерживает запрос. Задержка необходима, чтобы тема среды выполнения в момент загрузки страницы правильно применялась к приложению. Однако не рекомендуется использование ссылок на статические ресурсы для ключей, которые существуют только в теме или как системные ресурсы. Это происходит потому, что такие ссылки не будут повторно обработаны при изменении темы пользователем в режиме реального времени. Ссылки на динамические ресурсы более надежны при запросе темы или системных ресурсов. Исключением является случай, когда элемент темы сам запрашивает другой ресурс. Такие ссылки должны быть статическими по причинам, упомянутым выше.  
  
 Поведение исключения, если ссылка на статический ресурс не найдена, варьируется. Если ресурс был задержан, то исключение возникнет во время выполнения. Если ресурс не был задержан, исключение возникнет во время загрузки.  
  
### <a name="dynamic-resources"></a>Динамические ресурсы  
 Динамические ресурсы лучше всего подходят для указанных ниже случаев.  
  
-   Когда значение ресурса зависит от условий, не известных до времени выполнения. Сюда входят системные ресурсы или ресурсы, которые в противном случае задаются пользователем. Например, можно создать значения установщика, которые ссылаются на системные свойства, как представлено <xref:System.Windows.SystemColors>, <xref:System.Windows.SystemFonts>, или <xref:System.Windows.SystemParameters>. Эти значения являются действительно динамическими, так как они в конечном счете берутся из среды выполнения пользователя и операционной системы. Кроме того, возможны темы уровня приложения, которые могут изменяться, когда при доступе к ресурсу уровня страницы также необходимо отследить изменения.  
  
-   При создании стилей темы или ссылке на них для пользовательского элемента управления.  
  
-   Когда планируется настроить содержимое <xref:System.Windows.ResourceDictionary> во время существования приложения.  
  
-   Когда имеется сложная структура ресурсов с взаимозависимостями, где могут потребоваться опережающие ссылки. Ссылки на статические ресурсы не поддерживают дальнейшие ссылки, но ссылки на динамические ресурсы поддерживают их, так как ресурс не нужно пересчитывать до времени выполнения и ссылки вперед, следовательно, не релевантны.  
  
-   При ссылке на особенно большой с точки зрения компиляции или рабочего множества ресурс, если он может не использоваться немедленно при загрузке страницы. Ссылки на статические ресурсы всегда загружаются из [!INCLUDE[TLA2#tla_xaml](../../../../includes/tla2sharptla-xaml-md.md)] при загрузке страницы; однако ссылка на динамический ресурс не загружается до фактического использования.  
  
-   При создании стиля, где значения для метода задания могут браться из других значений, на которые влияют темы или другие пользовательские параметры.  
  
-   Когда ресурсы применяются к элементам, для которых во время существования приложения может быть изменен порядок наследования в логическом дереве. Изменение родительского элемента также потенциально изменяет область подстановки ресурса, так что, если необходим пересчет ресурса в новой области элемента с измененным порядком наследования, всегда следует использовать ссылку на динамический ресурс.  
  
#### <a name="dynamic-resource-lookup-behavior"></a>Поведение подстановки динамического ресурса  
 Поведение подстановки ресурса для ссылки на динамический ресурс аналогично поведению подстановки в коде при вызове метода <xref:System.Windows.FrameworkElement.FindResource%2A> или <xref:System.Windows.FrameworkElement.SetResourceReference%2A>.  
  
1.  Процесс подстановки ищет запрошенный ключ в словаре ресурсов, определенном элементом, который устанавливает это свойство.  
  
    -   Если элемент определяет <xref:System.Windows.FrameworkElement.Style%2A> свойства <xref:System.Windows.Style.Resources%2A> словаря в <xref:System.Windows.Style> проверяется.  
  
    -   Если элемент определяет <xref:System.Windows.Controls.Control.Template%2A> свойства <xref:System.Windows.FrameworkTemplate.Resources%2A> словаря в <xref:System.Windows.FrameworkTemplate> проверяется.  
  
2.  Затем процесс подстановки обходит логическое дерево снизу вверх до родительского элемента и его словаря ресурсов. Это продолжается до тех пор, пока не будет достигнут корневой элемент.  
  
3.  Далее проверяются ресурсы приложения. Ресурсы приложения — это ресурсы в словарь ресурсов, который определяется <xref:System.Windows.Application> объекта для вашего [!INCLUDE[TLA2#tla_winclient](../../../../includes/tla2sharptla-winclient-md.md)] приложения.  
  
4.  Для текущей активной темы проверяется словарь ресурсов темы. Если тема изменяется во время выполнения, значение будет повторно вычислено.  
  
5.  Проверяются системные ресурсы.  
  
 Поведение исключения (если таковое имеется) варьируется.  
  
-   Если ресурс запрошен по <xref:System.Windows.FrameworkElement.FindResource%2A> вызвать, а не найдена, создается исключение.  
  
-   Если ресурс запрошен по <xref:System.Windows.FrameworkElement.TryFindResource%2A> вызова и не найден, исключение не создается, но возвращенное значение является `null`. Если задаваемое свойство не принимает `null`, возможно, более глубокого исключение будет вызываться (это зависит от отдельного свойства).  
  
-   Если ресурс запрошен по ссылке на динамический ресурс в [!INCLUDE[TLA2#tla_xaml](../../../../includes/tla2sharptla-xaml-md.md)]и не найден, то поведение зависит от общей системы свойств, но общее поведение таково, как если бы не свойство произошло операции задания на уровне, где существует ресурс. Например, при попытке задать фон для отдельного элемента кнопки с помощью ресурса, который не может быть вычислен, не происходит задания значения, но фактическое значение по-прежнему может быть получено от других членов системы свойств и приоритета значения. Например, значение фона может по-прежнему браться из локально определенного стиля кнопки или из стиля темы. Для свойств, не определенных стилями темы, после неудавшейся попытки обработки ресурса фактическое значение может браться из значения по умолчанию в метаданных свойства.  
  
#### <a name="restrictions"></a>Ограничения  
 Для ссылок на динамические ресурсы есть некоторые важные ограничения. Должно выполняться по крайней мере одно из указанных ниже условий.  
  
-   Задаваемое свойство должно быть свойством <xref:System.Windows.FrameworkElement> или <xref:System.Windows.FrameworkContentElement>. Свойство должно поддерживаться <xref:System.Windows.DependencyProperty>.  
  
-   Ссылка указывает на значение в <xref:System.Windows.Style> <xref:System.Windows.Setter>.  
  
-   Задаваемое свойство должно быть свойством <xref:System.Windows.Freezable> , предоставленных в качестве значения любого <xref:System.Windows.FrameworkElement> или <xref:System.Windows.FrameworkContentElement> свойства, или <xref:System.Windows.Setter> значение.  
  
 Так как свойство должно быть <xref:System.Windows.DependencyProperty> или <xref:System.Windows.Freezable> свойства, большинство изменений свойств может распространяться на пользовательский Интерфейс поскольку изменение свойства (измененное значение динамического ресурса) подтверждается системой свойств. Большинство элементов управления включает логику, которая принудительно создаст новый макет элемента управления, если <xref:System.Windows.DependencyProperty> изменения и свойство может повлиять на макет. Однако не все свойства, имеющие [DynamicResource Markup Extension](../../../../docs/framework/wpf/advanced/dynamicresource-markup-extension.md) со своим значением, гарантированно предоставляют значение таким образом, что они обновляются в режиме реального времени в пользовательском Интерфейсе. Функциональные возможности по-прежнему могут отличаться в зависимости от свойства, типа, которому принадлежит свойство, или даже логической структуры приложения.  
  
<a name="stylesimplicitkeys"></a>   
## <a name="styles-datatemplates-and-implicit-keys"></a>Стили, DataTemplates и неявные ключи  
 Ранее было отмечено, что у всех элементов в <xref:System.Windows.ResourceDictionary> должна иметь ключ. Тем не менее, означает, что все ресурсы должны иметь явные `x:Key`. Некоторые типы объектов поддерживают неявный ключ, если он определен как ресурс, где значение ключа связано со значением другого свойства. Это называется неявным ключом, в то время как `x:Key` атрибута является явным ключом. Любой неявный ключ можно перезаписать, указав явный ключ.  
  
 Один из сценариев очень важны для ресурсов является определение <xref:System.Windows.Style>. На самом деле <xref:System.Windows.Style> почти всегда определяется как запись в словаре ресурсов, поскольку стили по своей природе предназначены для повторного использования. Дополнительные сведения о стилях см. в разделе [Стилизация и использование шаблонов](../../../../docs/framework/wpf/controls/styling-and-templating.md).  
  
 С помощью неявного ключа можно создавать стили для элементов управления либо ссылаться на них. Стили темы, определяющие внешний вид элемента управления по умолчанию, используют этот неявный ключ. Неявный ключ с точки зрения запроса представляет <xref:System.Type> самого элемента управления. Неявный ключ с точки зрения определения ресурса представляет <xref:System.Windows.Style.TargetType%2A> стиля. Таким образом, при создании темы для пользовательских элементов управления, стили, которые взаимодействуют с существующими стилями темы, необходимо указать [директива x: Key](../../../../docs/framework/xaml-services/x-key-directive.md) для этого <xref:System.Windows.Style>. А если требуется использовать стили из тем, то вообще не нужно задавать стиль. Например, следующее определение стиля работает, даже если <xref:System.Windows.Style> ресурса не имеет ключа:  
  
 [!code-xaml[FEResourceSH_snip#ImplicitStyle](../../../../samples/snippets/csharp/VS_Snippets_Wpf/FEResourceSH_snip/CS/page2.xaml#implicitstyle)]  
  
 Что действительно стилем ключ: неявный ключ `typeof(` <xref:System.Windows.Controls.Button> `)`. В разметке, можно указать <xref:System.Windows.Style.TargetType%2A> в качестве типа имени (или при необходимости можно использовать [{x: Type...}](../../../../docs/framework/xaml-services/x-type-markup-extension.md) Чтобы вернуть <xref:System.Type>.  
  
 Через механизмы стиля темы по умолчанию, используемые [!INCLUDE[TLA2#tla_winclient](../../../../includes/tla2sharptla-winclient-md.md)], что стиль будет применен как стиль среды выполнения для <xref:System.Windows.Controls.Button> на странице несмотря на то что <xref:System.Windows.Controls.Button> сама не пытается задать ее <xref:System.Windows.FrameworkElement.Style%2A> свойство или конкретный ресурс ссылка на стиль. Стиль, определенный на странице, находится в последовательности подстановки раньше, чем стиль словаря темы, и использует тот же ключ, что и стиль словаря темы. Можно просто задать `<Button>Hello</Button>` в любом месте страницы и стиль, определенный с <xref:System.Windows.Style.TargetType%2A> из `Button` будет применен к этой кнопке. Если требуется, можно по-прежнему явно ключ стиля тип совпадает со значением <xref:System.Windows.Style.TargetType%2A>для ясности в разметке, но это необязательно.  
  
 Неявные ключи для стилей не применяются для элемента управления, если <xref:System.Windows.FrameworkElement.OverridesDefaultStyle%2A> — `true` (также Обратите внимание, что <xref:System.Windows.FrameworkElement.OverridesDefaultStyle%2A> может быть задан как часть собственного поведения для класса элемента управления, а не явно на экземпляр элемента управления). Кроме того, чтобы поддерживать неявные ключи для сценариев в производном классе, элемент управления необходимо переопределить <xref:System.Windows.FrameworkElement.DefaultStyleKey%2A> (все существующие элементы управления, предоставляемые как часть [!INCLUDE[TLA2#tla_winclient](../../../../includes/tla2sharptla-winclient-md.md)] это сделать). Дополнительные сведения о стилях, темы и разработке элементов управления см. в разделе [рекомендации по разработке элементов](../../../../docs/framework/wpf/controls/guidelines-for-designing-stylable-controls.md).  
  
 <xref:System.Windows.DataTemplate>также имеет неявный ключ. Неявный ключ для <xref:System.Windows.DataTemplate> — <xref:System.Windows.DataTemplate.DataType%2A> значение свойства. <xref:System.Windows.DataTemplate.DataType%2A>Можно также задать имя типа, а не явным образом с помощью [{x: Type...} ](../../../../docs/framework/xaml-services/x-type-markup-extension.md). Дополнительные сведения см. в разделе [сведения о шаблонах данных](../../../../docs/framework/wpf/data/data-templating-overview.md).  
  
## <a name="see-also"></a>См. также  
 <xref:System.Windows.ResourceDictionary>  
 [Ресурсы приложений](../../../../docs/framework/wpf/advanced/optimizing-performance-application-resources.md)  
 [Ресурсы и код](../../../../docs/framework/wpf/advanced/resources-and-code.md)  
 [Определение и создание ссылки на ресурс](../../../../docs/framework/wpf/advanced/how-to-define-and-reference-a-resource.md)  
 [Общие сведения об управлении приложением](../../../../docs/framework/wpf/app-development/application-management-overview.md)  
 [Расширение разметки x:Type](../../../../docs/framework/xaml-services/x-type-markup-extension.md)  
 [Расширение разметки StaticResource](../../../../docs/framework/wpf/advanced/staticresource-markup-extension.md)  
 [Расширение разметки DynamicResource](../../../../docs/framework/wpf/advanced/dynamicresource-markup-extension.md)
