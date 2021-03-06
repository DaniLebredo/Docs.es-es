---
title: Migrar la configuración a ASP.NET Core
author: ardalis
description: Obtenga información acerca de cómo migrar la configuración de un proyecto de MVC de ASP.NET a un proyecto de MVC de ASP.NET Core.
manager: wpickett
ms.author: riande
ms.date: 10/14/2016
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: migration/configuration
ms.openlocfilehash: ead4f96aa0041cd919caa972d3bb05bd94a857b3
ms.sourcegitcommit: 477d38e33530a305405eaf19faa29c6d805273aa
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2018
---
# <a name="migrate-configuration-to-aspnet-core"></a>Migrar la configuración a ASP.NET Core

Por [Steve Smith](https://ardalis.com/) y [Scott Addie](https://scottaddie.com)

En el artículo anterior, se comenzó a [migrar un proyecto de MVC de ASP.NET a ASP.NET MVC de núcleo](xref:migration/mvc). En este artículo, se migra la configuración.

[Vea o descargue el código de ejemplo](https://github.com/aspnet/Docs/tree/master/aspnetcore/migration/configuration/samples) ([cómo descargarlo](xref:tutorials/index#how-to-download-a-sample))

## <a name="setup-configuration"></a>Parámetros de configuración

ASP.NET Core ya no utiliza la *Global.asax* y *web.config* archivos que utilizan versiones anteriores de ASP.NET. En versiones anteriores de ASP.NET, la lógica de inicio de la aplicación se ha puesto en un `Application_StartUp` método dentro de *Global.asax*. Más adelante, en ASP.NET MVC, un *Startup.cs* archivo se incluyó en la raíz del proyecto; y, se llamó cuando se inició la aplicación. ASP.NET Core ha adoptado este enfoque por completo mediante la colocación de toda la lógica de inicio en el *Startup.cs* archivo.

El *web.config* archivo también se ha sustituido en ASP.NET Core. Configuración en Sí ahora se puede configurar, como parte del procedimiento de inicio de aplicación se describen en *Startup.cs*. Configuración todavía puede usar los archivos XML, pero normalmente los proyectos de ASP.NET Core colocará los valores de configuración en un archivo con formato JSON, como *appSettings.JSON que se*. Sistema de configuración de ASP.NET Core también es posible tener acceso a las variables de entorno, lo que pueden proporcionar un [ubicación más segura y sólida](xref:security/app-secrets) para valores específicos del entorno. Esto es especialmente cierto para los secretos como cadenas de conexión y las claves de API que no deben protegerse en el control de código fuente. Vea [configuración](xref:fundamentals/configuration/index) para obtener más información sobre la configuración de ASP.NET Core.

En este artículo, estamos comenzando con el proyecto de ASP.NET Core parcialmente migrado desde [el artículo anterior](xref:migration/mvc). Para la configuración de la instalación, agregue el siguiente constructor y la propiedad a la *Startup.cs* archivo se encuentra en la raíz del proyecto:

[!code-csharp[](configuration/samples/WebApp1/src/WebApp1/Startup.cs?range=11-16)]

Tenga en cuenta que en este momento, el *Startup.cs* archivo no se compilará, tal y como necesitamos agregar lo siguiente `using` instrucción:

```csharp
using Microsoft.Extensions.Configuration;
```

Agregar un *appSettings.JSON que se* archivo a la raíz del proyecto mediante la plantilla de elemento apropiado:

![Agregar AppSettings JSON](configuration/_static/add-appsettings-json.png)

## <a name="migrate-configuration-settings-from-webconfig"></a>Migrar la configuración de web.config

Nuestro proyecto de ASP.NET MVC incluye la cadena de conexión de base de datos necesarios en *web.config*, en la `<connectionStrings>` elemento. En el proyecto de ASP.NET Core, vamos a almacenar esta información en el *appSettings.JSON que se* archivo. Abra *appSettings.JSON que se*y tenga en cuenta que ya incluye lo siguiente:

[!code-json[](../migration/configuration/samples/WebApp1/src/WebApp1/appsettings.json?highlight=4)]

En la línea resaltada descrita anteriormente, cambie el nombre de la base de datos de **_CHANGE_ME** en el nombre de la base de datos.

## <a name="summary"></a>Resumen

ASP.NET Core coloca toda la lógica de inicio de la aplicación en un único archivo, en el que los servicios necesarios y las dependencias pueden definirse y configuradas. Reemplaza el *web.config* archivo con una característica de configuración flexible que puede aprovechar una variedad de formatos de archivo, como JSON, así como las variables de entorno.
