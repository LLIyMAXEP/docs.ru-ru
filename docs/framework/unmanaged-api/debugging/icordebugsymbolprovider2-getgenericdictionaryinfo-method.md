---
title: "Метод ICorDebugSymbolProvider2::GetGenericDictionaryInfo"
ms.custom: 
ms.date: 03/30/2017
ms.prod: .net-framework
ms.reviewer: 
ms.suite: 
ms.technology: dotnet-clr
ms.tgt_pltfrm: 
ms.topic: reference
ms.assetid: ba28fe4e-5491-4670-bff7-7fde572d7593
caps.latest.revision: "4"
author: rpetrusha
ms.author: ronpet
manager: wpickett
ms.openlocfilehash: e0c4192c94d70bd9406607d645716e4dd6f8b957
ms.sourcegitcommit: 4f3fef493080a43e70e951223894768d36ce430a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/21/2017
---
# <a name="icordebugsymbolprovider2getgenericdictionaryinfo-method"></a>Метод ICorDebugSymbolProvider2::GetGenericDictionaryInfo
Получает универсальную карту словаря  
  
## <a name="syntax"></a>Синтаксис  
  
```  
HRESULT GetGenericDictionaryInfo(  
   [out] ICorDebugMemoryBuffer** ppMemoryBuffer  
);  
```  
  
#### <a name="parameters"></a>Параметры  
 `ppMemoryBuffer`  
 [out] Указатель на адрес [ICorDebugMemoryBuffer](../../../../docs/framework/unmanaged-api/debugging/icordebugmemorybuffer-interface.md) объекта, содержащего универсальную карту словаря. Дополнительные сведения см. в разделе "Примечания".  
  
## <a name="remarks"></a>Примечания  
  
> [!NOTE]
>  Этот метод доступен только в машинном коде .NET.  
  
 Карта состоит из двух разделов верхнего уровня.  
  
-   Объект [каталога](#Directory) содержащий относительные виртуальные адреса (RVA) всех словарей, включенных в эту карту.  
  
-   Байт синхронизированная [кучи](#Heap) , содержащий сведения о создании экземпляра объекта. Она запускается сразу после последней записи каталога.  
  
<a name="Directory"></a>   
## <a name="the-directory"></a>Каталог  
 Каждая запись в каталоге ссылается на смещение в куче; то есть это смещение относительно начала кучи. Значения отдельных записей, не обязательно должны быть уникальными; несколько записей каталога могут указывать на одно и то же смещение в куче.  
  
 Часть каталога универсальной карты словаря имеет следующую структуру.  
  
-   Первые 4 байта содержит число записей словаря (т. е. число относительных виртуальных адресов в словаре). Мы будем называть это значение как *N*. Если старший бит установлен, записи сортируются по относительному виртуальному адресу в порядке возрастания.  
  
-   *N* записей каталога. Каждая запись состоит из 8 байт в двух 4-байтовых сегментах.  
  
    -   Байты с 0 по 3: RVA; относительный виртуальный адрес словаря.  
  
    -   Байты с 4 по 7: смещение; смещение относительно начала кучи.  
  
<a name="Heap"></a>   
## <a name="the-heap"></a>Куча  
 Размер кучи может быть вычислен модулем чтения потока путем вычитания длины потока из размера каталога + 4. Другими словами:  
  
```  
Heap Size = Stream.Length – (Directory Size + 4)  
```  
  
 где размер каталога равен `N * 8`.  
  
 Формат каждого элемента сведений о создании экземпляра в куче выглядит следующим образом.  
  
-   Длина этого элемента сведений о создании экземпляра в байтах в сжатом формате ECMA метаданных. Значение исключает эти сведения о длине.  
  
-   Количество типов универсального создания экземпляра или *T*, в сжатом формате ECMA метаданных.  
  
-   *T* типов, каждый из которых представлен в формате ECMA сигнатуры типа.  
  
 Включение длины для каждого элемента кучи позволяет простую сортировку раздела каталога без влияния на кучу.  
  
## <a name="requirements"></a>Требования  
 **Платформы:** разделе [требования к системе для](../../../../docs/framework/get-started/system-requirements.md).  
  
 **Заголовок:** CorDebug.idl, CorDebug.h  
  
 **Библиотека:** CorGuids.lib  
  
 **Версии платформы .NET framework:**[!INCLUDE[net_46_native](../../../../includes/net-46-native-md.md)]  
  
## <a name="see-also"></a>См. также  
 [Интерфейс ICorDebugSymbolProvider2](../../../../docs/framework/unmanaged-api/debugging/icordebugsymbolprovider2-interface.md)  
 [Интерфейсы отладки](../../../../docs/framework/unmanaged-api/debugging/debugging-interfaces.md)