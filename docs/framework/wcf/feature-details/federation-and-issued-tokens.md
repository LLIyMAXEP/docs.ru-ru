---
title: "Федерация и выданные маркеры"
ms.custom: 
ms.date: 03/30/2017
ms.prod: .net-framework
ms.reviewer: 
ms.suite: 
ms.technology: dotnet-clr
ms.tgt_pltfrm: 
ms.topic: article
helpviewer_keywords:
- WCF, federation
- issued tokens [WCF]
- federation [WCF], issued tokens
ms.assetid: 4c31ee7d-a820-4067-8b84-a83049021bb6
caps.latest.revision: "16"
author: dotnet-bot
ms.author: dotnetcontent
manager: wpickett
ms.workload: dotnet
ms.openlocfilehash: 017d3e51022ad9980dc8f058415697c80a2a6b35
ms.sourcegitcommit: 16186c34a957fdd52e5db7294f291f7530ac9d24
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/22/2017
---
# <a name="federation-and-issued-tokens"></a>Федерация и выданные маркеры
С помощью [!INCLUDE[indigo1](../../../../includes/indigo1-md.md)] можно создавать клиенты, обеспечивающие безопасный обмен данными со службами, реализующими спецификации WS-Federation и WS-Trust. В этих спецификациях XML, протокол SOAP и язык WSDL используются для предоставления механизмов, позволяющих производить проверку подлинности и авторизацию в различных областях доверия.  
  
## <a name="in-this-section"></a>В этом разделе  
 [Федерация](../../../../docs/framework/wcf/feature-details/federation.md)  
 Общие сведения о федерации.  
  
 [Федерация и доверие](../../../../docs/framework/wcf/feature-details/federation-and-trust.md)  
 Список вопросов архитектуры, о которых необходимо помнить при создании федеративных служб или клиентов.  
  
 [Практическое руководство. Создание федеративного клиента](../../../../docs/framework/wcf/feature-details/how-to-create-a-federated-client.md)  
 Основы создания федеративного клиента с помощью [!INCLUDE[indigo2](../../../../includes/indigo2-md.md)].  
  
 [Практическое руководство. Настройка учетных данных службы федерации](../../../../docs/framework/wcf/feature-details/how-to-configure-credentials-on-a-federation-service.md)  
 Основные этапы создания федеративной службы.  
  
 [Практическое руководство. Создание WSFederationHttpBinding](../../../../docs/framework/wcf/feature-details/how-to-create-a-wsfederationhttpbinding.md)  
 Порядок настройки клиентов и служб, использующих `WSFederationHttpBinding`.  
  
 [Практическое руководство. Создание службы маркеров безопасности](../../../../docs/framework/wcf/feature-details/how-to-create-a-security-token-service.md)  
 Этапы создания службы маркеров безопасности.  
  
 [Утверждения и маркеры языка SAML (Security Assertions Markup Language)](../../../../docs/framework/wcf/feature-details/saml-tokens-and-claims.md)  
 Маркеры языка Security Assertions Markup Language (SAML), допускающие расширение и позволяющие создавать типы утверждений с широкими функциональными возможностями.  
  
 [Практическое руководство. Настройка локального издателя](../../../../docs/framework/wcf/feature-details/how-to-configure-a-local-issuer.md)  
 Порядок создания локального издателя маркеров безопасности.  
  
 [Практическое руководство. Порядок отключения безопасных сеансов в WSFederationHttpBinding](../../../../docs/framework/wcf/feature-details/how-to-disable-secure-sessions-on-a-wsfederationhttpbinding.md)  
 Порядок отключения безопасных сеансов в `WSFederationHttpBinding`. Отключение безопасных сеансов необходимо при создании веб-фермы, требующей сеансы для каждого клиента.  
  
## <a name="reference"></a>Ссылка  
 <xref:System.IdentityModel.Claims>  
  
 <xref:System.ServiceModel.ServiceAuthorizationManager>  
  
 <xref:System.IdentityModel.Tokens.SamlSecurityToken>  
  
 <xref:System.ServiceModel.Security.IssuedTokenClientCredential>  
  
 <xref:System.ServiceModel.Security.IssuedTokenServiceCredential>  
  
 <xref:System.ServiceModel.Security.Tokens.IssuedSecurityTokenParameters>  
  
 <xref:System.ServiceModel.Security.Tokens.IssuedSecurityTokenProvider>  
  
 <xref:System.ServiceModel.WSFederationHttpBinding>  
  
## <a name="see-also"></a>См. также  
 [Авторизация](../../../../docs/framework/wcf/feature-details/authorization-in-wcf.md)  
 [Пользовательские маркеры](../../../../docs/framework/wcf/extending/custom-tokens.md)  
 [Модель безопасности для Windows Server App Fabric](http://go.microsoft.com/fwlink/?LinkID=201279&clcid=0x409)
