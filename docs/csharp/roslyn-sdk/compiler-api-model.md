---
title: "Концепции пакета SDK для .NET Compiler Platform и объектная модель"
description: "Этот обзор содержит сведения, необходимые для эффективной работы с пакетом SDK компилятора .NET. Вы узнаете о слоях API, основных используемых типах и общей объектной модели."
author: billwagner
ms.author: wiwagn
ms.date: 10/10/2017
ms.topic: conceptual
ms.prod: .net
ms.devlang: devlang-csharp
ms.custom: mvc
ms.openlocfilehash: 65a4695c6f4e5268582d83452246ed8262d6fe2f
ms.sourcegitcommit: d095094e942eedf09530ea5636fbaf9029853027
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/19/2017
---
# <a name="understand-the-net-compiler-platform-sdk-model"></a>Общие сведения о модели пакета SDK для .NET Compiler Platform

Компиляторы обрабатывают код, который вы пишете, следуя структурированным правилам, которые часто отличаются от того, как читают и воспринимают код люди. Понимание принципов работы модели, используемой компиляторами, важно для изучения API, используемых при создании средств на основе Roslyn. 

## <a name="compiler-pipeline-functional-areas"></a>Функциональные области конвейера компилятора

Пакет SDK для .NET Compiler Platform реализует функции анализа кода в компиляторах C# и Visual Basic, предоставляя слой API, отражающий традиционный конвейер компилятора.

![Этапы обработки исходного кода на конвейере компилятора для получения объектного кода](media/compiler-pipeline.png)

Каждый этап конвейера — это отдельный компонент. Сначала на этапе синтаксического анализа выполняется расстановка токенов и анализ текста исходного кода с учетом синтаксиса, а затем и грамматики языка. Потом на этапе объявления анализируется источник и импортированные метаданные для формирования именованных символов. Далее на этапе привязки идентификаторы в коде сопоставляются с символами. Наконец, на этапе порождения создается сборка со всеми сведениями, сформированными компилятором.

![API конвейера компилятора предоставляет доступ к каждому шагу, который является частью конвейера компилятора.](media/compiler-pipeline-api.png)

В соответствии с каждым из этих этапов пакет SDK для .NET Compiler Platform предоставляет объектную модель, обеспечивающую доступ к информации на этом этапе. Этап анализа предоставляет дерево синтаксиса, этап объявления предоставляет таблицу иерархических символов, этап привязки предоставляет результат семантического анализа компилятора, а этап порождения — это API, создающий коды байтов IL.

![Языковые службы, доступные из API компилятора на каждом этапе конвейера компилятора](media/compiler-pipeline-lang-svc.png)

Каждой компилятор сочетает в себе эти компоненты, образуя единый процесс.

Эти API также используются и в Visual Studio. Например, функции структурирования и форматирования кода используют деревья синтаксиса, обозреватель объектов и функции навигации — таблицу символов, рефакторинг и операции "Перейти к определению" — семантическую модель, а операция "Изменить и продолжить" использует их все, включая API порождения. 

## <a name="api-layers"></a>Слои API

Пакет SDK компилятора .NET состоит из двух основных слоев API: API компиляторов и API рабочих областей.

![Слои API, представленные API конвейера компилятора](media/api-layers.png)

### <a name="compiler-apis"></a>API компилятора

Слой компилятора содержит объектные модели, которые соответствуют сведениям, представляемым на каждом этапе конвейера компилятора, как синтаксическим, так и семантическим. Слой компилятора также содержит неизменяемый моментальный снимок однократного вызова компилятора, включая ссылки на сборки, параметры компилятора и файлы исходного кода. Существует два отдельных API, представляющих язык C# и Visual Basic. Эти два API похожи по форме, но для обеспечения высокой точности привязаны к конкретному языку. Этот слой не имеет зависимостей от компонентов Visual Studio.

### <a name="diagnostic-apis"></a>API диагностики

В процессе анализа компилятор может создать набор данных диагностики, охватывающий все — от синтаксиса, семантики и явных ошибок присваивания до различных предупреждений и информационных диагностических сообщений. Слой API компилятора предоставляет диагностику посредством расширяемого API, который разрешает подключать пользовательские анализаторы к процессу компиляции. Она позволяет создавать заданные пользователем диагностические данные, например, формируемые такими программами, как StyleCop или FxCop, параллельно с диагностическими данными, определенными компилятором. Такое формирование данных диагностики дает преимущество естественной интеграции со средствами, такими как MSBuild и Visual Studio, которые зависят от диагностики при применении таких процедур, как прерывание сборки на основе политики, отображение динамических волнистых линий в редакторе и предложение исправлений кода.

### <a name="scripting-apis"></a>API скриптов

API размещения и скриптов являются частью слоя компилятора. Их можно использовать для выполнения фрагментов кода и накопления контекста выполнения во время выполнения.
Интерактивная среда REPL (read-evaluate-print loop) C# использует эти API. REPL позволяет использовать C# в качестве скриптового языка C#, выполняя код в интерактивном режиме по мере его написания.

### <a name="workspaces-apis"></a>API рабочих областей

Слой рабочих областей содержит соответствующий API, который является отправной точкой для анализа кода и рефакторинга в рамках целых решений. Он помогает упорядочить всю информацию о проектах в решении в рамках одной объектной модели, предоставляя вам прямой доступ к объектным моделям слоя компилятора без необходимости анализировать файлы, настраивать параметры или управлять зависимостями между проектами.

Кроме того, слой рабочих областей предоставляет набор API, используемый реализацией средств анализа кода и рефакторинга, которые работают в среде размещения, таких как Visual Studio IDE. В качестве примеров можно привести API "Найти все ссылки", "Форматирование" и "Создание кода".

Этот слой не имеет зависимостей от компонентов Visual Studio.