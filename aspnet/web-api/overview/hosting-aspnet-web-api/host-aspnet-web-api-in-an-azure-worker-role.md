---
uid: web-api/overview/hosting-aspnet-web-api/host-aspnet-web-api-in-an-azure-worker-role
title: Hospedar ASP.NET Web API 2 en un rol de trabajador de Azure | Documentos de Microsoft
author: MikeWasson
description: Este tutorial muestra cómo hospedar ASP.NET Web API en un rol de trabajador de Azure, mediante OWIN para probar internamente el marco Web API. Abrir la interfaz Web de .NET (OWIN)...
ms.author: aspnetcontent
manager: wpickett
ms.date: 04/02/2014
ms.topic: article
ms.assetid: 6980ee2e-d6b0-4a08-8fb6-ab96362dd0e3
ms.technology: dotnet-webapi
ms.prod: .net-framework
msc.legacyurl: /web-api/overview/hosting-aspnet-web-api/host-aspnet-web-api-in-an-azure-worker-role
msc.type: authoredcontent
ms.openlocfilehash: 7ba1dc850e2f9d9c88e6ddf263a796e1867a98be
ms.sourcegitcommit: f8852267f463b62d7f975e56bea9aa3f68fbbdeb
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2018
---
<a name="host-aspnet-web-api-2-in-an-azure-worker-role"></a>Hospedar ASP.NET Web API 2 en un rol de trabajador de Azure
====================
por [Mike Wasson](https://github.com/MikeWasson)

> Este tutorial muestra cómo hospedar ASP.NET Web API en un rol de trabajador de Azure, mediante OWIN para probar internamente el marco Web API.
> 
> [Abrir la interfaz Web para .NET](http://owin.org/) (OWIN) define una abstracción entre servidores web de .NET y las aplicaciones web. OWIN separa la aplicación web desde el servidor, lo que hace OWIN ideal para el autohospedaje una aplicación web en su propio proceso, fuera de IIS: por ejemplo, dentro de un rol de trabajador de Azure.
> 
> En este tutorial, usará el paquete Microsoft.Owin.Host.HttpListener, lo que proporciona un servidor HTTP que se usa para auto-hospedar aplicaciones OWIN.
> 
> ## <a name="software-versions-used-in-the-tutorial"></a>Versiones de software que se usa en el tutorial
> 
> 
> - [Visual Studio 2013](https://www.microsoft.com/visualstudio/eng/2013-downloads)
> - Web API 2
> - [Azure SDK para .NET 2.3](https://azure.microsoft.com/downloads/)


## <a name="create-a-microsoft-azure-project"></a>Crear un proyecto de Microsoft Azure

Inicie Visual Studio con privilegios de administrador. Se necesitan privilegios de administrador para depurar la aplicación localmente, mediante el emulador de proceso de Azure.

En el **archivo** menú, haga clic en **New**, a continuación, haga clic en **proyecto**. De **plantillas instaladas**, en Visual C#, haga clic en **nube** y, a continuación, haga clic en **servicio de nube de Windows Azure**. Denomine el proyecto "AzureApp" y haga clic en **Aceptar**.

[![](host-aspnet-web-api-in-an-azure-worker-role/_static/image2.png)](host-aspnet-web-api-in-an-azure-worker-role/_static/image1.png)

En el **nuevo servicio de nube de Windows Azure** cuadro de diálogo, haga doble clic en **rol de trabajo**. Deje el nombre predeterminado ("Roldetrabajo1"). Este paso agrega un rol de trabajo a la solución. Haga clic en **Aceptar**.

[![](host-aspnet-web-api-in-an-azure-worker-role/_static/image4.png)](host-aspnet-web-api-in-an-azure-worker-role/_static/image3.png)

La solución de Visual Studio que se crea contiene dos proyectos:

- &quot;AzureApp&quot; define los roles y la configuración de la aplicación de Azure.
- &quot;WorkerRole1&quot; contiene el código para el rol de trabajo.

En general, una aplicación de Azure puede contener varios roles, aunque en este tutorial usa un solo rol.

![](host-aspnet-web-api-in-an-azure-worker-role/_static/image5.png)

## <a name="add-the-web-api-and-owin-packages"></a>Agregar la API Web y paquetes OWIN

Desde el **herramientas** menú, haga clic en **Administrador de paquetes de biblioteca**, a continuación, haga clic en **Package Manager Console**.

En la ventana de la consola de administrador de paquetes, escriba el siguiente comando:

[!code-console[Main](host-aspnet-web-api-in-an-azure-worker-role/samples/sample1.cmd)]

## <a name="add-an-http-endpoint"></a>Agregar un extremo HTTP

En el Explorador de soluciones, expanda el proyecto AzureApp. Expanda el nodo Roles, haga clic en WorkerRole1 y seleccione **propiedades**.

![](host-aspnet-web-api-in-an-azure-worker-role/_static/image6.png)

Haga clic en **extremos**y, a continuación, haga clic en **Agregar extremo**.

En el **protocolo** lista desplegable, seleccione "http". En **puerto público** y **puerto privado**, escriba 80. Estos números de puerto pueden ser diferentes. El puerto público es lo que los clientes utilizan cuando envía una solicitud a la función.

[![](host-aspnet-web-api-in-an-azure-worker-role/_static/image8.png)](host-aspnet-web-api-in-an-azure-worker-role/_static/image7.png)

## <a name="configure-web-api-for-self-host"></a>Configure Web API de autohospedaje

En el Explorador de soluciones, haga clic en el proyecto de WorkerRole1 y seleccione **agregar** / **clase** para agregar una nueva clase. Asigne a la clase el nombre `Startup`.

![](host-aspnet-web-api-in-an-azure-worker-role/_static/image9.png)

Reemplace todo el código reutilizable en este archivo con lo siguiente:

[!code-csharp[Main](host-aspnet-web-api-in-an-azure-worker-role/samples/sample2.cs)]

## <a name="add-a-web-api-controller"></a>Agregar un controlador de API Web

A continuación, agregue una clase de controlador de API Web. Haga clic en el proyecto WorkerRole1 y seleccione **agregar** / **clase**. Nombre de la clase TestController. Reemplace todo el código reutilizable en este archivo con lo siguiente:

[!code-csharp[Main](host-aspnet-web-api-in-an-azure-worker-role/samples/sample3.cs)]

Para simplificar, este controlador simplemente define dos métodos GET que devuelven texto sin formato.

## <a name="start-the-owin-host"></a>Iniciar el Host OWIN

Abra el archivo WorkerRole.cs. Esta clase define el código que se ejecuta cuando se inicia y detiene el rol de trabajo.

Agregue la siguiente instrucción using:

[!code-csharp[Main](host-aspnet-web-api-in-an-azure-worker-role/samples/sample4.cs)]

Agregar un **IDisposable** miembro a la `WorkerRole` clase:

[!code-csharp[Main](host-aspnet-web-api-in-an-azure-worker-role/samples/sample5.cs)]

En el `OnStart` método, agregue el código siguiente para iniciar el host:

[!code-csharp[Main](host-aspnet-web-api-in-an-azure-worker-role/samples/sample6.cs?highlight=5)]

El **WebApp.Start** método inicia el host OWIN. El nombre de la `Startup` clase es un parámetro de tipo para el método. Por convención, el host puede llamar el `Configure` método de esta clase.

Invalidar el `OnStop` para desechar el  *\_aplicación* instancia:

[!code-csharp[Main](host-aspnet-web-api-in-an-azure-worker-role/samples/sample7.cs)]

Este es el código completo para WorkerRole.cs:

[!code-csharp[Main](host-aspnet-web-api-in-an-azure-worker-role/samples/sample8.cs)]

Compile la solución y presione F5 para ejecutar la aplicación localmente en el emulador de proceso de Azure. Dependiendo de la configuración del firewall, deberá permitir que el emulador a través del firewall.

> [!NOTE]
> Si se produce una excepción similar al siguiente, vea [esta entrada de blog](https://blogs.msdn.com/b/praburaj/archive/2013/11/20/fileloadexception-on-microsoft-owin-when-running-on-worker-role.aspx) para encontrar una solución. "No se pudo cargar el archivo o ensamblado ' Microsoft.Owin, Version = 2.0.2.0, Culture = neutral, PublicKeyToken = 31bf3856ad364e35' o una de sus dependencias. Definición del manifiesto del ensamblado encontrada no coincide con la referencia de ensamblado. (Excepción de HRESULT: 0x80131040) "


El emulador de proceso asigna una dirección IP local para el punto de conexión. Puede encontrar la dirección IP mediante la visualización de la interfaz de usuario del emulador de proceso. Haga clic en el icono del emulador en la barra del área de notificación de tareas y seleccione **Mostrar IU del emulador de proceso**.

[![](host-aspnet-web-api-in-an-azure-worker-role/_static/image11.png)](host-aspnet-web-api-in-an-azure-worker-role/_static/image10.png)

Buscar la dirección IP en implementaciones de servicios de implementación [id], detalles del servicio. Abra un explorador web y vaya a http://<em>dirección</em>/test/1, donde <em>dirección</em> es la dirección IP asignada por el emulador de proceso; por ejemplo, `http://127.0.0.1:80/test/1`. Debería ver la respuesta desde el controlador de API Web:

![](host-aspnet-web-api-in-an-azure-worker-role/_static/image12.png)

## <a name="deploy-to-azure"></a>Implementar en Azure

Para este paso, debe tener una cuenta de Azure. Si aún no tiene uno, puede crear una cuenta de prueba gratuita en tan solo unos minutos. Para obtener más información, consulte [evaluación gratuita de Microsoft Azure](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A261C142F).

En el Explorador de soluciones, haga clic en el proyecto AzureApp. Seleccione **Publicar**.

![](host-aspnet-web-api-in-an-azure-worker-role/_static/image13.png)

Si no inició sesión en su cuenta de Azure, haga clic en **inicio de sesión**.

[![](host-aspnet-web-api-in-an-azure-worker-role/_static/image15.png)](host-aspnet-web-api-in-an-azure-worker-role/_static/image14.png)

Una vez que haya iniciado sesión, elija una suscripción y haga clic en **siguiente**.

[![](host-aspnet-web-api-in-an-azure-worker-role/_static/image17.png)](host-aspnet-web-api-in-an-azure-worker-role/_static/image16.png)

Escriba un nombre para el servicio de nube y elija una región. Haga clic en **Crear**.

![](host-aspnet-web-api-in-an-azure-worker-role/_static/image18.png)

Haga clic en **Publicar**.

[![](host-aspnet-web-api-in-an-azure-worker-role/_static/image20.png)](host-aspnet-web-api-in-an-azure-worker-role/_static/image19.png)

La ventana de registro de actividad de Azure muestra el progreso de la implementación. Cuando se implementa la aplicación, vaya a http://appname.cloudapp.net/test/1.

![](host-aspnet-web-api-in-an-azure-worker-role/_static/image21.png)

## <a name="additional-resources"></a>Recursos adicionales

- [Información general del proyecto Katana](../../../aspnet/overview/owin-and-katana/an-overview-of-project-katana.md)
- [Proyecto de Katana en GitHub](https://github.com/aspnet/AspNetKatana)
