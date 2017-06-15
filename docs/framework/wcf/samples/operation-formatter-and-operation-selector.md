---
title: "Модуль форматирования и селектор операции | Microsoft Docs"
ms.custom: ""
ms.date: "03/30/2017"
ms.prod: ".net-framework-4.6"
ms.reviewer: ""
ms.suite: ""
ms.technology: 
  - "dotnet-clr"
ms.tgt_pltfrm: ""
ms.topic: "article"
ms.assetid: 1c27e9fe-11f8-4377-8140-828207b98a0e
caps.latest.revision: 19
author: "Erikre"
ms.author: "erikre"
manager: "erikre"
caps.handback.revision: 19
---
# Модуль форматирования и селектор операции
В этом образце показано, как с помощью точек расширяемости [!INCLUDE[indigo1](../../../../includes/indigo1-md.md)] разрешить передачу в сообщениях данных в формате, отличном от ожидаемого средой [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)].По умолчанию модули форматирования [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)] ожидают, что параметры методов будут включаться в элемент `soap:body`.В этом образце показано, как реализовать пользовательский модуль форматирования операций, который анализирует параметры из строки HTTP\-запроса GET и вызывает методы с использованием этих данных.  
  
 Данный образец основан на образце [Начало работы](../../../../docs/framework/wcf/samples/getting-started-sample.md), который реализует контракт службы `ICalculator`.Он показывает, каким образом можно изменить сообщения Add, Subtract, Multiply и Divide, чтобы они использовали HTTP\-запросы GET в качестве запросов клиента серверу и HTTP\-запросы POST с сообщениями POX в качестве ответов сервера клиенту.  
  
 Для этого в образце имеются следующие элементы.  
  
-   Класс `QueryStringFormatter`, который реализует интерфейсы <xref:System.ServiceModel.Dispatcher.IClientMessageFormatter> и <xref:System.ServiceModel.Dispatcher.IDispatchMessageFormatter> для клиента и сервера соответственно и обрабатывает данные в строке запроса.  
  
-   Класс `UriOperationSelector`, который реализует интерфейс <xref:System.ServiceModel.Dispatcher.IDispatchOperationSelector> на сервере для выполнения диспетчеризации операций в зависимости от имени операции в запросе GET.  
  
-   Поведение конечной точки `EnableHttpGetRequestsBehavior` \(и соответствующая конфигурация\), которое добавляет в среду выполнения селектор нужной операции.  
  
-   Показано, как включить в среду выполнения новый модуль форматирования операций.  
  
-   В этом образце служба и клиент являются консольными приложениями \(EXE\).  
  
> [!NOTE]
>  Процедура настройки и инструкции по построению для данного образца приведены в конце этого раздела.  
  
## Ключевые понятия  
 `QueryStringFormatter`. Модуль форматирования — это компонент в [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)], который отвечает за преобразование сообщения в массив объектов параметров, а массив объектов параметров — в сообщение.Эта задача выполняется на клиенте с помощью интерфейса <xref:System.ServiceModel.Dispatcher.IClientMessageFormatter> и на сервере с помощью интерфейса <xref:System.ServiceModel.Dispatcher.IDispatchMessageFormatter>.Эти интерфейсы позволяют пользователям получать сообщения запросов и ответов из методов `Serialize` и `Deserialize`.  
  
 В этом образце класс `QueryStringFormatter` реализует оба эти интерфейса и реализуется на стороне клиента и сервера.  
  
 Запрос  
  
-   В этом образце класс <xref:System.ComponentModel.TypeConverter> используется для преобразования параметров в сообщении запроса в строки и обратно.Если объект для определенного типа <xref:System.ComponentModel.TypeConverter> недоступен, модуль форматирования в этом образце создает исключение.  
  
-   В методе `IClientMessageFormatter.SerializeRequest` на стороне клиента модуль форматирования создает код URI для соответствующего адреса назначения и добавляет имя операции в качестве суффикса.Это имя используется для перенаправления на соответствующую операцию на сервере.После этого массив объектов параметров сериализуется в строку запроса кода URI с использованием имен и значений параметров, преобразованных классом <xref:System.ComponentModel.TypeConverter>.Свойства <xref:System.ServiceModel.Channels.MessageHeaders.To%2A> и <xref:System.ServiceModel.Channels.MessageProperties.Via%2A> задаются данному URI.Значение <xref:System.ServiceModel.Channels.MessageProperties> доступно с помощью свойства <xref:System.ServiceModel.Channels.Message.Properties%2A>.  
  
-   В методе `IDispatchMessageFormatter.DeserializeRequest` на сервере модуль форматирования извлекает код URI `Via` в свойствах сообщения входящего запроса.Модуль форматирования преобразует пары "имя\-значение" в строке запроса URI в имена и значения параметров и подставляет эти имена и значения параметров в массив передаваемых методу параметров.Обратите внимание, что диспетчеризация по операциям уже произошла, поэтому в данном методе суффикс имени операции игнорируется.  
  
 Ответ  
  
-   В этом образце HTTP\-метод GET используется только для запросов.Модуль форматирования делегирует отправку ответа исходному модулю форматирования, который бы использовался для создания XML\-сообщения.Одна из целей этого образца заключается в том, чтобы показать, как можно реализовать такой делегирующий модуль форматирования.  
  
### Класс UriPathSuffixOperationSelector  
 Интерфейс <xref:System.ServiceModel.Dispatcher.IDispatchOperationSelector> позволяет пользователям реализовывать собственную логику определения операции, которой должно перенаправляться то или иное сообщение.  
  
 В этом образце необходимо реализовать на сервере класс `UriPathSuffixOperationSelector`, чтобы выбирать нужную операцию, поскольку имя операции включается в код URI HTTP\-запроса GET, а не в заголовок действия в сообщении.В этом образце разрешены только имена операций, указываемые с учетом регистра.  
  
 Метод `SelectOperation` принимает входящее сообщение и ищет в свойствах сообщения код URI `Via`.Он извлекает из кода URI суффикс имени операции, ищет во внутренней таблице имя операции, которой следует перенаправить сообщение, и возвращает имя операции.  
  
### Класс EnableHttpGetRequestsBehavior  
 Компонент `UriPathSuffixOperationSelector` можно настраивать программным образом или с помощью поведения конечной точки.В этом образце реализовано поведение `EnableHttpGetRequestsBehavior`, которое задано в файле конфигурации приложения службы.  
  
 На сервере:  
  
 Свойству <xref:System.ServiceModel.Dispatcher.DispatchRuntime.OperationSelector%2A> присвоена реализация <xref:System.ServiceModel.Dispatcher.IDispatchOperationSelector>.  
  
 По умолчанию [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)] использует фильтр адресов с точным совпадением.Код URI входящего сообщения содержит суффикс имени операции, за которым следует строка запроса, содержащая данные параметров, поэтому поведение также изменяет фильтр адресов, чтобы он срабатывал по совпадению префикса.Для этого используется фильтр [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)]<xref:System.ServiceModel.Dispatcher.PrefixEndpointAddressMessageFilter>.  
  
### Установка модулей форматирования операций  
 Поведения операций, которые задают модули форматирования, являются уникальными.Одно такое поведение всегда реализуется по умолчанию для каждой операции, чтобы создать нужный модуль форматирования операции.Однако такие поведения очень похожи на поведения других операций; их невозможно идентифицировать по каким\-либо другим атрибутам.Чтобы установить поведение замены, реализация должна искать конкретные поведения модулей форматирования, по умолчанию устанавливаемые загрузчиком типов [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)], и либо заменять его, либо добавлять совместимое поведение для выполнения после поведения по умолчанию.  
  
 Эти поведения модулей форматирования операций можно задать программным образом перед вызовом <xref:System.ServiceModel.Channels.CommunicationObject.Open%2A?displayProperty=fullName> или путем задания поведения операции, которое выполняется после поведения по умолчанию.Однако его невозможно легко настроить с помощью поведения конечной точки \(а следовательно с помощью конфигурации\), поскольку модель поведений не допускает замены поведения другими поведениями или иного изменения дерева описаний.  
  
 На клиенте:  
  
 Реализацию <xref:System.ServiceModel.Dispatcher.IClientMessageFormatter> нужно осуществлять таким образом, чтобы этот модуль форматирования мог преобразовывать запросы в HTTP\-запросы GET и делегировать создание ответов исходному модулю форматирования.Для этого вызывается вспомогательный метод `EnableHttpGetRequestsBehavior.ReplaceFormatterBehavior`.  
  
 Это необходимо сделать до вызова метода `CreateChannel`.  
  
```  
void ReplaceFormatterBehavior(OperationDescription operationDescription, EndpointAddress address)  
{  
    // Remove the DataContract behavior if it is present.  
    IOperationBehavior formatterBehavior = operationDescription.Behaviors.Remove<DataContractSerializerOperationBehavior>();  
    if (formatterBehavior == null)  
    {  
        // Remove the XmlSerializer behavior if it is present.  
        formatterBehavior = operationDescription.Behaviors.Remove<XmlSerializerOperationBehavior>();  
        ...  
    }  
  
    // Remember what the innerFormatterBehavior was.  
    DelegatingFormatterBehavior delegatingFormatterBehavior = new DelegatingFormatterBehavior(address);  
    delegatingFormatterBehavior.InnerFormatterBehavior = formatterBehavior;  
   operationDescription.Behaviors.Add(delegatingFormatterBehavior);  
}  
```  
  
 На сервере:  
  
-   Интерфейс <xref:System.ServiceModel.Dispatcher.IDispatchMessageFormatter> нужно реализовать таким образом, чтобы этот модуль форматирования мог считывать HTTP\-запросы GET и делегировать создание ответов исходному модулю форматирования.Это делается путем вызова того же вспомогательного метода `EnableHttpGetRequestsBehavior.ReplaceFormatterBehavior`, что и для клиента \(см. предыдущий пример кода\).  
  
-   Это необходимо сделать до вызова метода <xref:System.ServiceModel.Channels.CommunicationObject.Open%2A>.В этом образце показано, каким образом можно вручную изменить модуль форматирования перед вызовом метода <xref:System.ServiceModel.Channels.CommunicationObject.Open%2A>.Того же результата можно достичь, создав для класса <xref:System.ServiceModel.ServiceHost> производный класс, который вызывает `EnableHttpGetRequestsBehavior.ReplaceFormatterBehavior` перед открытием \(примеры см. в документации по размещению\).  
  
### Взаимодействие с пользователем  
 На сервере:  
  
-   Серверную реализацию `ICalculator` изменять не требуется.  
  
-   В файле App.config службы следует задавать пользовательскую привязку POX, которая устанавливает для атрибута `messageVersion` элемента `textMessageEncoding` значение `None`.  
  
    ```  
    <bindings>  
      <customBinding>  
        <binding name="poxBinding">  
          <textMessageEncoding messageVersion="None" />  
          <httpTransport />  
        </binding>  
      </customBinding>  
    </bindings>  
    ```  
  
-   Кроме того, в файле App.config службы необходимо задать пользовательское поведение `EnableHttpGetRequestsBehavior`, добавив его в раздел расширений поведения.  
  
    ```  
    <behaviors>  
      <endpointBehaviors>  
        <behavior name="enableHttpGetRequestsBehavior">  
          <enableHttpGetRequests />  
        </behavior>  
      </endpointBehaviors>  
    </behaviors>  
  
    <extensions>  
      <behaviorExtensions>  
        <!-- Enabling HTTP GET requests: Behavior Extension -->  
        <add   
          name="enableHttpGetRequests"           type="Microsoft.ServiceModel.Samples.EnableHttpGetRequestsBehaviorElement, QueryStringFormatter, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />  
      </behaviorExtensions>  
    </extensions>  
    ```  
  
-   Перед вызовом метода <xref:System.ServiceModel.Channels.CommunicationObject.Open%2A> необходимо добавить модули форматирования операций.  
  
 На клиенте:  
  
-   Изменять клиентскую реализацию не требуется.  
  
-   В файле App.config клиента следует задать пользовательскую привязку POX, которая устанавливает для атрибута `messageVersion` элемента `textMessageEncoding` значение `None`.Отличие от службы заключается в том, что клиент должен включить ручную адресацию, чтобы можно было изменять исходящий адрес назначения.  
  
    ```  
    <bindings>  
      <customBinding>  
        <binding name="poxBinding">  
          <textMessageEncoding messageVersion="None" />  
          <httpTransport manualAddressing="True" />  
        </binding>  
      </customBinding>  
    </bindings>  
    ```  
  
-   В файле App.config клиента должно быть задано то же пользовательское поведение `EnableHttpGetRequestsBehavior`, что и на сервере.  
  
-   Перед вызовом метода <xref:System.ServiceModel.ChannelFactory%601.CreateChannel> необходимо добавить модули форматирования операций.  
  
 При выполнении образца запросы и ответы операций отображаются в окне консоли клиента.Все четыре операции \(Add, Subtract, Multiply и Divide\) должны выполняться успешно.  
  
> [!IMPORTANT]
>  Образцы уже могут быть установлены на компьютере.Перед продолжением проверьте следующий каталог \(по умолчанию\).  
>   
>  `<диск_установки>:\WF_WCF_Samples`  
>   
>  Если этот каталог не существует, перейдите на страницу [Образцы Windows Communication Foundation \(WCF\) и Windows Workflow Foundation \(WF\) для .NET Framework 4](http://go.microsoft.com/fwlink/?LinkId=150780), чтобы загрузить все образцы [!INCLUDE[indigo1](../../../../includes/indigo1-md.md)] и [!INCLUDE[wf1](../../../../includes/wf1-md.md)].Этот образец расположен в следующем каталоге.  
>   
>  `<диск_установки>:\WF_WCF_Samples\WCF\Extensibility\Formatters\QuieryStringFormatter`  
  
##### Настройка, построение и выполнение образца  
  
1.  Убедитесь, что выполнены действия, описанные в разделе [Процедура однократной настройки образцов Windows Communication Foundation](../../../../docs/framework/wcf/samples/one-time-setup-procedure-for-the-wcf-samples.md).  
  
2.  Чтобы выполнить построение решения, следуйте инструкциям раздела [Построение образцов Windows Communication Foundation](../../../../docs/framework/wcf/samples/building-the-samples.md).  
  
3.  Чтобы выполнить образец на одном или нескольких компьютерах, следуйте инструкциям раздела [Выполнение примеров Windows Communication Foundation](../../../../docs/framework/wcf/samples/running-the-samples.md).  
  
## См. также