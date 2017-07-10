---
title: "UriTemplate и UriTemplateTable | Microsoft Docs"
ms.custom: ""
ms.date: "03/30/2017"
ms.prod: ".net-framework-4.6"
ms.reviewer: ""
ms.suite: ""
ms.technology: 
  - "dotnet-clr"
ms.tgt_pltfrm: ""
ms.topic: "article"
ms.assetid: 5cbbe03f-4a9e-4d44-9e02-c5773239cf52
caps.latest.revision: 24
author: "Erikre"
ms.author: "erikre"
manager: "erikre"
caps.handback.revision: 24
---
# UriTemplate и UriTemplateTable
Интернет\-разработчикам требуется возможность описывать форму и структуру универсальных кодов ресурса \(URI\), на которые отвечают их службы.В [!INCLUDE[indigo1](../../../../includes/indigo1-md.md)] добавлены два новых класса, позволяющие разработчикам контролировать свои универсальные коды ресурса \(URI\).Классы <xref:System.UriTemplate> и <xref:System.UriTemplateTable> формируют основу модуля отправки данных на основе URI, который используется в [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)].Эти классы могут также использоваться сами по себе, позволяя разработчиками воспользоваться преимуществами шаблонов и механизма сопоставления универсального кода ресурса \(URI\) без реализации службы [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)].  
  
## Шаблоны  
 Шаблон — это способ описания набора относительных универсальных кодов ресурса \(URI\).Набор шаблонов универсальных кодов ресурсов \(URI\), приведенный в следующей таблице, показывает, как может быть определена система, извлекающая различные виды информации о погоде.  
  
|Данные|Шаблон|  
|------------|------------|  
|Прогноз для страны|weather\/national|  
|Прогноз для штата|weather\/{state}|  
|Прогноз для города|weather\/{state}\/{city}|  
|Прогноз для рода занятий|weather\/{state}\/{city}\/{activity}|  
  
 В этой таблице описывает набор, структурно аналогичный универсальным кодам ресурсов \(URI\).Каждая запись является шаблоном универсального кода ресурса \(URI\).Сегменты в фигурных скобках описывают переменные.Сегменты без фигурных скобок описывают строки литералов.Классы\-шаблоны [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)] позволяют разработчику сопоставить входящий универсальный код ресурса \(URI\) \(например, «\/weather\/wa\/seattle\/cycling»\) с описывающим его шаблоном \(«\/weather\/{state}\/{city}\/{activity}»\).  
  
## UriTemplate  
 Класс <xref:System.UriTemplate> является классом, инкапсулирующим шаблон универсального кода ресурса \(URI\).Конструктор принимает строковый параметр, задающий шаблон.Эта строка содержит шаблон в формате, приведенном в следующем разделе.Класс <xref:System.UriTemplate> предоставляет методы, позволяющие сопоставлять входящий URI с шаблоном, создавать URI из шаблона, получать коллекцию имен переменных в шаблоне, определять, эквивалентны ли два шаблона, и возвращать строку шаблона.  
  
 Метод <xref:System.UriTemplate.Match%28System.Uri%2CSystem.Uri%29> принимает базовый адрес и потенциальный универсальный код ресурса \(URI\), после чего пытается сопоставить универсальный код ресурса \(URI\) с шаблоном.Если сопоставление завершается успешно, возвращается экземпляр <xref:System.UriTemplateMatch>.Объект <xref:System.UriTemplateMatch> содержит базовый универсальный код ресурса \(URI\), потенциальный URI, коллекцию имя\/значение параметров запроса, массив сегментов относительного пути, коллекцию имя\/значение сопоставленных переменных, экземпляр <xref:System.UriTemplate>, использованный для сопоставления, строку, содержащую несопоставленную часть потенциального URI \(используется, если шаблон содержит подстановочный знак\) и объект, связанный с этим шаблоном.  
  
> [!NOTE]
>  При сопоставлении потенциального универсального кода ресурса \(URI\) класс <xref:System.UriTemplate> не учитывает схему и номер порта.  
  
 Есть два метода, позволяющие создавать универсальный код ресурса \(URI\) из шаблона — <xref:System.UriTemplate.BindByName%28System.Uri%2CSystem.Collections.Specialized.NameValueCollection%29> и [BindByPosition\(Uri, String\<xref:System.UriTemplate.BindByPosition%28System.Uri%2CSystem.String%5B%5D%29>.Метод <xref:System.UriTemplate.BindByName%28System.Uri%2CSystem.Collections.Specialized.NameValueCollection%29> принимает базовый адрес и коллекцию параметров вида имя\/значение.Эти параметры подставляются вместо переменных при использовании шаблона.Метод [BindByPosition\(Uri, String\<xref:System.UriTemplate.BindByPosition%28System.Uri%2CSystem.String%5B%5D%29> принимает пары имя\/значение и подставляет их в порядке слева направо.  
  
 Метод <xref:System.UriTemplate.ToString> возвращает строку шаблона.  
  
 Свойство <xref:System.UriTemplate.PathSegmentVariableNames%2A> содержит коллекцию имен переменных, используемых в сегментах пути в строке шаблона.  
  
 Метод <xref:System.UriTemplate.IsEquivalentTo%28System.UriTemplate%29> принимает <xref:System.UriTemplate> в качестве параметра и возвращает логическое значение, указывающее, являются ли два шаблона эквивалентными.[!INCLUDE[crdefault](../../../../includes/crdefault-md.md)] подраздел «Эквивалентность шаблонов» далее в этом разделе.  
  
 Класс <xref:System.UriTemplate> предназначен для работы с любой схемой URI, соблюдающей грамматику URI HTTP.Ниже приведены примеры поддерживаемых схем URI.  
  
-   http:\/\/  
  
-   https:\/\/  
  
-   net.tcp:\/\/  
  
-   net.pipe:\/\/  
  
-   sb:\/\/  
  
 Схемы, подобные file:\/\/ и urn:\/\/, не соответствуют грамматике URI HTTP и при использовании с шаблонами URI приведут к непредсказуемым результатам.  
  
### Синтаксис строки шаблона  
 Шаблон содержит три части: путь, необязательный запрос и необязательный фрагмент.Например, рассмотрим следующий шаблон:  
  
```  
"/weather/{state}/{city}?forecast={length)#frag1  
```  
  
 Путь состоит из «\/weather\/{state}\/{city}», запрос состоит из «?forecast\={length}», и фрагмент состоит из «\#frag1».  
  
 Начальные и конечные знаки косой черты в выражении пути необязательны.Как запрос, так и фрагмент могут быть полностью опущены.Путь состоит из последовательных сегментов, разделенных знаком "\/", каждый сегмент имеет литеральное значение, имя переменной \(указанное в {фигурных скобках}\) или подстановочный знак \(указанный как "\*"\).В предыдущем шаблоне сегмент "\\weather\\" является литеральным значением, а "{state}" и "{city}" — это переменные.Переменным присваивается имя, содержащееся в их фигурных скобках; впоследствии переменные могут заменяться конкретными значениями для создания *закрытого URI*.Подстановочный знак является необязательным, но может использоваться только в конце универсального кода ресурса \(URI\), где он логически означает «остальная часть пути».  
  
 Выражение запроса \(если оно указано\) задает неупорядоченную последовательность пар «имя\-значение», разделяемых знаком «&».Элементами выражения запроса могут быть либо литеральные пары \(x\=2\) или переменная пара \(x\={var}\).Переменное выражение может содержаться только в правой части запроса.Конструкции \({someName} \= {someValue}\) не допускаются.Непарные значения \(?x\) не допускаются.Пустое выражение запроса и выражение запроса, содержащее только знак вопроса "?", равнозначны \(оба означают «любой запрос»\).  
  
 Выражение фрагмента может содержать любое литеральное значение, переменные не допускаются.  
  
 Все имена переменных шаблона в строке шаблона должны быть уникальными.В именах переменных шаблона регистр не учитывается.  
  
 Ниже приведены допустимые строки шаблона:  
  
-   ""  
  
-   "\/shoe"  
  
-   "\/shoe\/\*"  
  
-   "{shoe}\/boat"  
  
-   “{shoe}\/{boat}\/bed\/{quilt}”  
  
-   “shoe\/{boat}”  
  
-   “shoe\/{boat}\/\*”  
  
-   “shoe\/boat?x\=2”  
  
-   “shoe\/{boat}?x\={bed}”  
  
-   "shoe\/{boat}?x\={bed}&y\=band"  
  
-   “?x\={shoe}”  
  
-   "shoe?x\=3&y\={var}  
  
 Ниже приведены недопустимые строки шаблона:  
  
-   “{shoe}\/{SHOE}\/x\=2” — повторяющиеся имена переменных.  
  
-   “{shoe}\/boat\/?bed\={shoe}” — повторяющиеся имена переменных.  
  
-   "?x\=2&x\=3" — пары «имя\-значение» в строке запроса должны быть уникальными, даже если они являются литералами.  
  
-   "?x\=2&" — строка запроса неправильного формата.  
  
-   "?2&x\={shoe}" — строка запроса должна состоять из пар «имя\-значение».  
  
-   "?y\=2&&&X\=3" — строка запроса должна состоять из пар «имя\-значение», а имена не могут начинаться с символа «».  
  
### Составные сегменты пути  
 Составные сегменты пути позволяют включать в один сегмент пути универсального кода ресурса \(URI\) несколько переменных, а также несколько переменных совместно с литералами.Ниже приведены примеры допустимых составных сегментов пути:  
  
-   \/имя\_файла.{ext}\/  
  
-   \/{filename}.jpg\/  
  
-   \/{filename}.{ext}\/  
  
-   \/{a}.{b}некоторый\_литерал{c}\({d}\)\/  
  
 Ниже приведены примеры недопустимых составных сегментов пути.  
  
-   \/{} — переменная должна иметь имя.  
  
-   \/{shoe}{boat} — переменные должны разделяться литералом.  
  
### Совпадающие и составные сегменты пути  
 Составные сегменты пути позволяют определять UriTemplate, который имеет несколько переменных в одном сегменте пути.Возьмем в качестве примера следующую строку шаблона: «Адреса\/{штат}.{город}». Две переменные \(«штат» и «город»\) определены в одном сегменте.Этот шаблон будет соответствовать такому URL\-адресу, как «http:\/\/example.com\/Washington.Redmond», но он также будет соответствовать такому URL\-адресу, как «http:\/\/example.com\/Washington.Redmond.Microsoft».В последнем случае переменная «штат» будет содержать название штата «Вашингтон», а переменная «город» будет содержать «Redmond.Microsoft».В данном случае любой текст \(за исключением символа «\/»\) будет соответствовать переменной {город}.Если требуется шаблон, который не будет соответствовать «дополнительному» тексту, поместите переменную в отдельный сегмент шаблона, например: «Адреса\/{штат}\/{город}.  
  
### Именованные сегменты с подстановочным знаком  
 Именованный сегмент с подстановочным знаком — это любой переменный сегмент, имя переменной которого начинается с символа\-шаблона "\*".Следующая строка шаблона содержит именованный сегмент с подстановочным знаком, имеющий имя «shoe».  
  
```  
"literal/{*shoe}"  
```  
  
 Сегменты с подстановочными знаками должны удовлетворять следующим правилам.  
  
-   В каждой строке шаблона должно быть не более одного именованного сегмента с подстановочным знаком.  
  
-   Именованный сегмент с подстановочным знаком должен быть самым правым сегментом пути.  
  
-   Именованный сегмент с подстановочным знаком и анонимный сегмент с подстановочным знаком не могут одновременно использоваться в одной строке шаблона.  
  
-   Имя именованного сегмента с подстановочным знаком должно быть уникальным.  
  
-   Именованные сегменты с подстановочным знаком не могут иметь значений по умолчанию.  
  
-   Именованные сегменты с подстановочным знаком не могут заканчиваться знаком "\/".  
  
### Значения переменных по умолчанию  
 Значения переменных по умолчанию позволяют задавать значения по умолчанию для переменных шаблона.Значения переменных по умолчанию могут быть заданы в фигурных скобках, объявляющих переменную, или в виде коллекции, передаваемой конструктору UriTemplate.В следующем шаблоне показаны два способа задания <xref:System.UriTemplate> с переменными, имеющими значения по умолчанию.  
  
```  
UriTemplate t = new UriTemplate("/test/{a=1}/{b=5}");  
```  
  
 В этом шаблоне объявляются переменная с именем `a` и значением по умолчанию `1` и переменная с именем `b` со значением по умолчанию `5`.  
  
> [!NOTE]
>  Значения по умолчанию могут иметь только переменные сегмента пути.Переменные строки запроса, переменные составного сегмента и именованные переменные с подстановочным знаком не могут иметь значений по умолчанию.  
  
 В следующем коде показано, как обрабатываются значения переменных по умолчанию при сопоставлении потенциального универсального кода ресурса \(URI\).  
  
```  
Uri baseAddress = new Uri("http://localhost:800   
Dictionary<string,string> defVals = new Dictionary<string,string> {{"a","1"}, {"b", "5"}};  
UriTemplate t = new UriTemplate("/test/{a}/{b}", defVals);0");  
UriTemplate t = new UriTemplate("/{state=WA}/{city=Redmond}/", true);  
Uri candidate = new Uri("http://localhost:8000/OR");  
  
UriTemplateMatch m1 = t.Match(baseAddress, candidate);  
  
// Display contents of BoundVariables  
foreach (string key in m1.BoundVariables.AllKeys)  
{  
    Console.WriteLine("\t\t{0}={1}", key, m1.BoundVariables[key]);  
}  
// The output of the above code is  
// Template: /{state=WA}/{city=Redmond}/  
// Candidate URI: http://localhost:8000/OR  
// BoundVariables:  
//      STATE=OR  
//       CITY=Redmond  
  
```  
  
> [!NOTE]
>  URI вида http:\/\/localhost:8000\/\/\/ не соответствует шаблону, указанному в приведенном выше коде, в отличие от URI вида http:\/\/localhost:8000\/.  
  
 В следующем коде показано, как обрабатываются значения переменных по умолчанию при создании универсального кода ресурса \(URI\) с помощью шаблона.  
  
```  
Uri baseAddress = new Uri("http://localhost:8000/");  
Dictionary<string,string> defVals = new Dictionary<string,string> {{"a","1"}, {"b", "5"}};  
UriTemplate t = new UriTemplate("/test/{a}/{b}", defVals);  
NameValueCollection vals = new NameValueCollection();  
vals.Add("a", "10");  
  
Uri boundUri = t.BindByName(baseAddress, vals);  
Console.WriteLine("BaseAddress: {0}", baseAddress);  
Console.WriteLine("Template: {0}", t.ToString());  
  
Console.WriteLine("Values: ");  
foreach (string key in vals.AllKeys)  
{  
    Console.WriteLine("\tKey = {0}, Value = {1}", key, vals[key]);  
}  
Console.WriteLine("Bound URI: {0}", boundUri);  
  
// The output of the preceding code is  
// BaseAddress: http://localhost:8000/  
// Template: /test/{a}/{b}  
// Values:  
//     Key = a, Value = 10  
// Bound URI: http://localhost:8000/test/10/5  
  
```  
  
 Если переменной присвоено значение по умолчанию `null`, существуют дополнительные ограничения.Переменная может иметь значение по умолчанию `null`, если эта переменная содержится в самом правом сегменте строки шаблона или если все сегменты справа от этого сегмента имеют значения по умолчанию `null`.Ниже приводятся допустимые строки шаблона со значениями по умолчанию `null`:  
  
-   ```  
    UriTemplate t = new UriTemplate("shoe/{boat=null}");  
    ```  
  
-   ```  
    UriTemplate t = new UriTemplate("{shoe=null}/{boat=null}");  
    ```  
  
-   ```  
    UriTemplate t = new UriTemplate("{shoe=1}/{boat=null}");  
    ```  
  
 Ниже приводятся недопустимые строки шаблона со значениями по умолчанию `null`:  
  
-   ```  
    UriTemplate t = new UriTemplate("{shoe=null}/boat"); // null default must be in the right most path segment  
    ```  
  
-   ```  
    UriTemplate t = new UriTemplate("{shoe=null}/{boat=x}/{bed=null}"); // shoe cannot have a null default because boat does not have a default null value  
    ```  
  
### Значения по умолчанию и сопоставление  
 При сопоставлении URI\-кандидата с шаблоном, имеющим значения по умолчанию, в коллекцию <xref:System.UriTemplateMatch.BoundVariables%2A> помещаются значения по умолчанию, если в URI\-кандидате не указаны значения.  
  
### Эквивалентность шаблонов  
 Два шаблона называются *структурно эквивалентными*, если все литералы в шаблоне совпадают, а переменные находятся в одинаковых сегментах.Например, указанные ниже шаблоны структурно эквивалентны.  
  
-   \/a\/{var1}\/b b\/{var2}?x\=1&y\=2  
  
-   a\/{x}\/b%20b\/{var1}?y\=2&x\=1  
  
-   a\/{y}\/B%20B\/{z}\/?y\=2&x\=1  
  
 Некоторые замечания.  
  
-   Если шаблон содержит начальные символы косой черты, не учитывается только первый из них.  
  
-   При сравнении строк шаблона для определения их структурной эквивалентности регистр пропускается для имен переменных и сегментов пути, в строках запроса регистр учитывается.  
  
-   Строки запроса неупорядочены.  
  
## UriTemplateTable  
 Класс <xref:System.UriTemplateTable> представляет ассоциативную таблицу объектов <xref:System.UriTemplate>, привязанных к объекту, выбранному разработчиком.Класс <xref:System.UriTemplateTable> должен содержать по крайней мере один объект <xref:System.UriTemplate>, чтобы можно было вызвать метод <xref:System.UriTemplateTable.MakeReadOnly%28System.Boolean%29>.Содержимое таблицы <xref:System.UriTemplateTable> можно изменять до тех пор, пока не будет вызван метод <xref:System.UriTemplateTable.MakeReadOnly%28System.Boolean%29>.Проверка выполняется при вызове метода <xref:System.UriTemplateTable.MakeReadOnly%28System.Boolean%29>.Тип выполняемой проверки зависит от значения параметра `allowMultiple` метода <xref:System.UriTemplateTable.MakeReadOnly%28System.Boolean%29>.  
  
 Если при вызове метода <xref:System.UriTemplateTable.MakeReadOnly%28System.Boolean%29> ему передается значение `false`, таблица <xref:System.UriTemplateTable> проверяется, чтобы убедиться, что в ней отсутствуют шаблоны.Если будут найдены какие\-либо структурно эквивалентные шаблоны, возникает исключение.Это используется совместно с методом <xref:System.UriTemplateTable.MatchSingle%28System.Uri%29>, если требуется обеспечить, чтобы только один шаблон соответствовал входящему универсальному коду ресурса \(URI\).  
  
 Если при выборе метода <xref:System.UriTemplateTable.MakeReadOnly%28System.Boolean%29> ему передается значение `true`, <xref:System.UriTemplateTable> допускает наличии нескольких структурно эквивалентных шаблонов в <xref:System.UriTemplateTable>.  
  
 Если набор объектов <xref:System.UriTemplate>, добавленных в таблицу <xref:System.UriTemplateTable>, содержит строки запроса, они не должны быть неоднозначными.Допускаются идентичные строки запросов.  
  
> [!NOTE]
>  Хотя <xref:System.UriTemplateTable> допускает базовые адреса, использующие схемы, отличные от HTTP, при сопоставлении потенциальных универсальных кодов ресурсов \(URI\) с шаблонами схема и номер порта пропускаются.  
  
### Неоднозначность строки запроса  
 Шаблоны, содержащие эквивалентные пути, содержат неоднозначные строки запроса, если существует универсальный код ресурса \(URI\), соответствующий нескольким шаблонам.  
  
 Приведенные ниже наборы строк запроса однозначны внутри себя.  
  
-   ?x\=1  
  
-   ?x\=2  
  
-   ?x\=3  
  
-   ?x\=1&y\={var}  
  
-   ?x\=2&z\={var}  
  
-   ?x\=3  
  
-   ?x\=1  
  
-   ?  
  
-   ?x\={var}  
  
-   ?  
  
-   ?m\=get&c\=rss  
  
-   ?m\=put&c\=rss  
  
-   ?m\=get&c\=atom  
  
-   ?m\=put&c\=atom  
  
 Приведенные ниже шаблоны строк запроса неоднозначны внутри себя.  
  
-   ?x\=1  
  
-   ?x\={var}  
  
 "x\=1" — соответствует обеим шаблонам.  
  
-   ?x\=1  
  
-   ?y\=2  
  
 Строка "x\=1&y\=2" соответствует обоим шаблонам.Это связано с тем, что строка запроса может содержать больше переменных стоки запроса, чем соответствующий ей шаблон.  
  
-   ?x\=1  
  
-   ?x\=1&y\={var}  
  
 Строка "x\=1&y\=3" соответствует обоим шаблонам.  
  
-   ?x\=3&y\=4  
  
-   ?x\=3&z\=5  
  
> [!NOTE]
>  Буквы "á" и "Á" считаются разными символами, если они используются в пути универсального кода ресурса \(URI\) или в литерале сегмента пути <xref:System.UriTemplate> \(но буквы "a" и "A" считаются одинаковыми\).Буквы "á" и "Á" считаются одинаковыми символами, если они используются в <xref:System.UriTemplate> {имя\_переменной} или строке запроса \(а буквы "a" и "A" также считаются одинаковыми\).  
  
## См. также  
 [Общие сведения о модели программирования WCF Web HTTP](../../../../docs/framework/wcf/feature-details/wcf-web-http-programming-model-overview.md)   
 [Объектная модель программирования WCF Web HTTP](../../../../docs/framework/wcf/feature-details/wcf-web-http-programming-object-model.md)   
 [UriTemplate](../../../../docs/framework/wcf/samples/uritemplate-sample.md)   
 [Таблица UriTemplate](../../../../docs/framework/wcf/samples/uritemplate-table-sample.md)   
 [Диспетчер таблицы UriTemplate](../../../../docs/framework/wcf/samples/uritemplate-table-dispatcher-sample.md)