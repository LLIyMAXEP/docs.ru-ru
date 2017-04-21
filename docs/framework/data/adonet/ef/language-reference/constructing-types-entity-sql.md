---
title: "Сборка типов (Entity SQL) | Microsoft Docs"
ms.custom: ""
ms.date: "03/30/2017"
ms.prod: ".net-framework-4.6"
ms.reviewer: ""
ms.suite: ""
ms.technology: 
  - "dotnet-ado"
ms.tgt_pltfrm: ""
ms.topic: "article"
dev_langs: 
  - "VB"
  - "CSharp"
  - "C++"
  - "ESQL"
ms.assetid: 41fa7bde-8d20-4a3f-a3d2-fb791e128010
caps.latest.revision: 2
author: "JennieHubbard"
ms.author: "jhubbard"
manager: "jhubbard"
caps.handback.revision: 2
---
# Сборка типов (Entity SQL)
[!INCLUDE[esql](../../../../../../includes/esql-md.md)] предоставляет три типа конструкторов: конструкторы строк, конструкторы коллекций и конструкторы именованных типов.  
  
## Конструкторы строк  
 В [!INCLUDE[esql](../../../../../../includes/esql-md.md)] конструкторы строк можно использовать для создания анонимных структурно типизированных записей из одного или нескольких значений.  Результирующий тип конструктора строк является типом строки, типы полей которого соответствуют типам значений, использовавшихся для создания этой строки.  Например, следующее выражение создает значение типа  `Record(a int, b string, c int)`.  
  
 `ROW(1 AS a, "abc" AS b, a + 34 AS c)`  
  
 Если в конструкторе строк не указан псевдоним для выражения, то платформа Entity Framework попытается сформировать его.  Дополнительные сведения см. в подразделе «Правила присвоения псевдонимов» раздела [Идентификаторы](../../../../../../docs/framework/data/adonet/ef/language-reference/identifiers-entity-sql.md).  
  
 В конструкторе строк псевдонимы присваиваются выражениям по следующим правилам.  
  
-   Выражение в конструкторе строк не может ссылаться на другие псевдонимы в этом же конструкторе.  
  
-   Два выражения в одном конструкторе строк не могут иметь одинаковый псевдоним.  
  
 Дополнительные сведения о конструкторах строк см. в разделе [ROW](../../../../../../docs/framework/data/adonet/ef/language-reference/row-entity-sql.md).  
  
## Конструкторы коллекций  
 В [!INCLUDE[esql](../../../../../../includes/esql-md.md)] конструкторы коллекций можно использовать для создания экземпляра мультинабора из списка значений.  Все значения в конструкторе должны иметь взаимно совместимый тип `T`. Конструктор формирует коллекцию типа `Multiset<T>`.  Например, следующее выражение создает коллекцию целых чисел.  
  
 `Multiset(1, 2, 3)`  
  
 `{1, 2, 3}`  
  
 Конструкторы пустых мультинаборов не разрешены, так как в этом случае нельзя определить тип элементов.  Недопустимый мультинабор:  
  
 `multiset() {}`  
  
 Для получения дополнительной информации см. [MULTISET](../../../../../../docs/framework/data/adonet/ef/language-reference/multiset-entity-sql.md).  
  
## Конструкторы именованных типов \(инициализаторы NamedType\)  
 Язык [!INCLUDE[esql](../../../../../../includes/esql-md.md)] позволяет конструкторам типов \(инициализаторам\) создавать экземпляры именованных сложных типов и типов сущностей.  Например, следующее выражение создает экземпляр типа `Person`.  
  
 `Person("abc", 12)`  
  
 Приведенное далее выражение создает экземпляр сложного типа.  
  
 `MyModel.ZipCode(‘98118’, ‘4567’)`  
  
 Приведенное далее выражение создает экземпляр вложенного сложного типа.  
  
 `MyModel.AddressInfo('My street address', 'Seattle', 'WA', MyModel.ZipCode('98118', '4567'))`  
  
 Приведенное далее выражение создает экземпляр сущности с вложенным сложным типом.  
  
 `MyModel.Person("Bill", MyModel.AddressInfo('My street address', 'Seattle', 'WA', MyModel.ZipCode('98118', '4567')))`  
  
 В следующем примере показано, как инициализировать свойство сложного типа значением null.  `MyModel.ZipCode(‘98118’, null)`  
  
 Предполагается, что аргументы конструктора находятся в порядке, соответствующем порядку декларации атрибутов типа.  
  
 Для получения дополнительной информации см. [Конструктор именованных типов](../../../../../../docs/framework/data/adonet/ef/language-reference/named-type-constructor-entity-sql.md).  
  
## См. также  
 [Справочник по Entity SQL](../../../../../../docs/framework/data/adonet/ef/language-reference/entity-sql-reference.md)   
 [Общие сведения о языке Entity SQL](../../../../../../docs/framework/data/adonet/ef/language-reference/entity-sql-overview.md)   
 [Система типов](../../../../../../docs/framework/data/adonet/ef/language-reference/type-system-entity-sql.md)