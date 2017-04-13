---
title: "Деструкторы (руководство по программированию на C#) | Документы Майкрософт"
ms.date: 2015-07-20
ms.prod: .net
ms.technology:
- devlang-csharp
ms.topic: article
dev_langs:
- CSharp
helpviewer_keywords:
- ~ [C#], in destructors
- C# language, destructors
- destructors [C#]
ms.assetid: 1ae6e46d-a4b1-4a49-abe5-b97f53d9e049
caps.latest.revision: 24
author: BillWagner
ms.author: wiwagn
translation.priority.ht:
- cs-cz
- de-de
- es-es
- fr-fr
- it-it
- ja-jp
- ko-kr
- pl-pl
- pt-br
- ru-ru
- tr-tr
- zh-cn
- zh-tw
translationtype: Human Translation
ms.sourcegitcommit: a06bd2a17f1d6c7308fa6337c866c1ca2e7281c0
ms.openlocfilehash: 6940be34b6cc15f006901e6d14d2a38ebb5d012a
ms.lasthandoff: 03/13/2017

---
# <a name="destructors-c-programming-guide"></a>Деструкторы (Руководство по программированию в C#)
Деструкторы используются для уничтожения экземпляров классов.  
  
## <a name="remarks"></a>Примечания  
  
-   В структурах определение деструкторов невозможно. Они применяются только в классах.  
  
-   Класс может иметь только один деструктор.  
  
-   Деструкторы не могут быть унаследованы или перегружены.  
  
-   Деструкторы невозможно вызвать. Они запускаются автоматически.  
  
-   Деструктор не принимает модификаторов и не имеет параметров.  
  
 Например, ниже показано объявление деструктора для класса `Car`.  
  
 [!code-cs[csProgGuideObjects#86](../../../csharp/programming-guide/classes-and-structs/codesnippet/CSharp/destructors_1.cs)]  
  
 Деструктор неявным образом вызывает метод <xref:System.Object.Finalize%2A> для базового класса объекта. Поэтому предыдущий код деструктора неявным образом преобразуется в следующий код:  
  
```  
protected override void Finalize()  
{  
    try  
    {  
        // Cleanup statements...  
    }  
    finally  
    {  
        base.Finalize();  
    }  
}  
```  
  
 Это означает, что метод `Finalize` вызывается рекурсивно для всех экземпляров цепочки наследования начиная с самого дальнего и заканчивая самым первым.  
  
> [!NOTE]
>  Пустые деструкторы использовать не следует. Если класс содержит деструктор, то в очереди метода `Finalize` создается запись. При вызове деструктора вызывается сборщик мусора, выполняющий обработку очереди. Если деструктор пустой, это приводит только к ненужному снижению производительности.  
  
 Программист не может управлять моментом вызова деструктора, потому что этот момент определяется сборщиком мусора. Сборщик мусора проверяет наличие объектов, которые больше не используются приложением. Если он считает, что какой-либо объект требует уничтожения, то он вызывает деструктор (при наличии) и освобождает память, используемую для хранения этого объекта. Деструкторы также вызываются при выходе из программы.  
  
 Существует возможность принудительно выполнить сборку мусора, вызвав метод <xref:System.GC.Collect%2A>, но в большинстве случаев этого следует избегать, потому что это может привести к проблемам с производительностью.  
  
## <a name="using-destructors-to-release-resources"></a>Использование деструкторов для освобождения ресурсов  
 В целом язык C# не требует управления памятью в той степени, в какой это требуется в случае разработки кода на языке, не рассчитанном на среду выполнения со сборкой мусора. Это связано с тем, что сборщик мусора платформы .NET Framework неявным образом управляет выделением и высвобождением памяти для объектов. Однако при инкапсуляции приложением неуправляемых ресурсов, например окон, файлов и сетевых подключений, для высвобождения этих ресурсов следует использовать деструкторы. Если объект требует уничтожения, то сборщик мусора выполняет метод `Finalize` этого объекта.  
  
## <a name="explicit-release-of-resources"></a>Высвобождение ресурсов явным образом  
 В случае, когда приложением используется ценный внешний ресурс, также рекомендуется обеспечить способ высвобождения этого ресурса явным образом, прежде чем сборщик мусора освободит объект. Для этого реализуется метод `Dispose` интерфейса <xref:System.IDisposable>, который выполняет необходимую очистку для объекта. Это может значительно повысить производительность приложения. Даже в случае использования такого явного управления ресурсами деструктор становится резервным средством очистки ресурсов, если вызов метода `Dispose` выполнить не удастся.  
  
 Дополнительные сведения об очистке ресурсов см. в следующих разделах:  
  
-   [Очистка неуправляемых ресурсов](http://msdn.microsoft.com/library/a17b0066-71c2-4ba4-9822-8e19332fc213)  
  
-   [Реализация метода dispose](http://msdn.microsoft.com/library/eb4e1af0-3b48-4fbc-ad4e-fc2f64138bf9)  
  
-   [Оператор using](../../../csharp/language-reference/keywords/using-statement.md)  
  
## <a name="example"></a>Пример  
 В приведенном ниже примере создаются три класса, образующих цепочку наследования. Класс `First` является базовым, класс `Second` является производным от класса `First`, а класс `Third` является производным от класса `Second`. Все три класса имеют деструкторы. В методе `Main()` создается экземпляр самого дальнего в цепочке наследования класса. При выполнении программы обратите внимание, что происходит автоматический вызов деструкторов всех трех классов по порядку от самого дальнего до первого в цепочке наследования.  
  
 [!code-cs[csProgGuideObjects#85](../../../csharp/programming-guide/classes-and-structs/codesnippet/CSharp/destructors_2.cs)]  
  
## <a name="c-language-specification"></a>Спецификация языка C#  
 [!INCLUDE[CSharplangspec](../../../csharp/language-reference/keywords/includes/csharplangspec_md.md)]  
  
## <a name="see-also"></a>См. также  
 <xref:System.IDisposable>   
 [Руководство по программированию на C#](../../../csharp/programming-guide/index.md)   
 [Конструкторы](../../../csharp/programming-guide/classes-and-structs/constructors.md)   
 [Сборка мусора](../../../standard/garbagecollection/index.md)