---
title: Empezar a trabajar con SignalR en ASP.NET Core
author: rachelappel
description: En este tutorial, creará una aplicación con SignalR para ASP.NET Core.
manager: wpickett
monikerRange: '>= aspnetcore-2.1'
ms.author: rachelap
ms.custom: mvc
ms.date: 05/22/2018
ms.prod: aspnet-core
ms.topic: tutorial
ms.technology: aspnet
uid: signalr/get-started
ms.openlocfilehash: ba1db640e5608fd9f5e7fa024283a651bf7772c2
ms.sourcegitcommit: 6784510cfb589308c3875ccb5113eb31031766b4
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/06/2018
ms.locfileid: "34819063"
---
# <a name="get-started-with-signalr-on-aspnet-core"></a>Empezar a trabajar con SignalR en ASP.NET Core

Por [Rachel Appel](https://twitter.com/rachelappel)

Este tutorial le enseña los fundamentos de la creación de una aplicación en tiempo real con SignalR para ASP.NET Core.

   ![Soluciones](get-started/_static/signalr-get-started-finished.png)

Este tutorial muestra las tareas de desarrollo de SignalR siguientes:

> [!div class="checklist"]
> * Crear un SignalR en la aplicación web de ASP.NET Core.
> * Cree un concentrador de SignalR para insertar contenido en los clientes.
> * Modificar el `Startup` clase y configurar la aplicación.

[Vea o descargue el código de ejemplo](https://github.com/aspnet/Docs/tree/master/aspnetcore/signalr/get-started/sample/) ([cómo descargarlo](xref:tutorials/index#how-to-download-a-sample))

# <a name="prerequisites"></a>Requisitos previos

Instalar el software siguiente:

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* [.NET core SDK 2.1 o posterior](https://www.microsoft.com/net/download/all)
* [Visual Studio de 2017](https://www.visualstudio.com/downloads/) versión 15.7 o posterior con el **ASP.NET y desarrollo web** carga de trabajo
* [NPM](https://www.npmjs.com/get-npm)

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* [.NET core SDK 2.1 o posterior](https://www.microsoft.com/net/download/all)
* [Visual Studio Code](https://code.visualstudio.com/download)
* [C# para el código de Visual Studio](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp)
* [NPM](https://www.npmjs.com/get-npm)

-----

## <a name="create-an-aspnet-core-project-that-hosts-signalr-client-and-server"></a>Cree un proyecto de ASP.NET Core que hospeda el servidor y cliente de SignalR

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio/)

1. Use la **archivo** > **nuevo proyecto** menú opción y elija **aplicación Web de ASP.NET Core**. Denomine el proyecto *SignalRChat*.

   ![Cuadro de diálogo nuevo proyecto en Visual Studio](get-started/_static/signalr-new-project-dialog.png)

2. Seleccione **aplicación Web** para crear un proyecto con páginas de Razor. A continuación, seleccione **Aceptar**. Asegúrese de que **ASP.NET Core 2.1** está seleccionada en el selector de framework, aunque se ejecute SignalR en versiones anteriores de .NET.

   ![Cuadro de diálogo nuevo proyecto en Visual Studio](get-started/_static/signalr-new-project-choose-type.png)

Visual Studio incluye la `Microsoft.AspNetCore.SignalR` paquete que contiene sus bibliotecas de servidor como parte de su **aplicación Web de ASP.NET Core** plantilla. Sin embargo, debe instalarse la biblioteca de cliente de JavaScript para SignalR con *npm*.

3. Ejecute los siguientes comandos el **Package Manager Console** ventana, desde la raíz del proyecto:

    ```console
    npm init -y
    npm install @aspnet/signalr
    ```     

4. Cree una carpeta nueva denominada "signalr" dentro de la *lib* carpeta del proyecto. Copia la *signalr.js* archivo *node_modules\\ @aspnet\signalr\dist\browser*  en esta carpeta.

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code/)

1. Desde el **Terminal integrado**, ejecute el siguiente comando:

    ```console
    dotnet new webapp -o SignalRChat
    ```

2. Instalar la biblioteca de cliente de JavaScript con *npm*.

    ```console
    npm init -y
    npm install @aspnet/signalr
    ```

3. Cree una carpeta nueva denominada "signalr" dentro de la *lib* carpeta del proyecto. Copia la *signalr.js* archivo *node_modules\\ @aspnet\signalr\dist\browser*  en esta carpeta.

-----

## <a name="create-the-signalr-hub"></a>Crear el concentrador de SignalR

Un concentrador es una clase que actúa como una canalización de alto nivel que permite al cliente y servidor llamar a métodos en entre sí.

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio/)

1. Agregar una clase al proyecto eligiendo **archivo** > **New** > **archivo** y seleccionando **clase de Visual C#**. Asignar nombre al archivo *ChatHub*. 

2. Heredar de `Microsoft.AspNetCore.SignalR.Hub`. La `Hub` clase contiene propiedades y eventos para administrar las conexiones y grupos, así como enviar y recibir datos.

3. Crear el `SendMessage` método que envía un mensaje a todos los clientes de chat conectado. Tenga en cuenta devuelve un [tarea](https://msdn.microsoft.com/library/system.threading.tasks.task(v=vs.110).aspx), porque SignalR es asincrónica. Código asincrónico se escala mejor.

   [!code-csharp[Startup](get-started/sample/Hubs/ChatHub.cs)]

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code/)

1. Abra la *SignalRChat* carpeta en el código de Visual Studio.

2. Agregar una clase al proyecto seleccionando **archivo** > **nuevo archivo** en el menú.

3. Heredar de `Microsoft.AspNetCore.SignalR.Hub`. La `Hub` clase contiene propiedades y eventos para administrar las conexiones y grupos, así como enviar y recibir datos a los clientes.

4. Agregue un método `SendMessage` a la clase. El `SendMessage` método envía un mensaje a todos los clientes de chat conectado. Tenga en cuenta devuelve un [tarea](/dotnet/api/system.threading.tasks.task), porque SignalR es asincrónica. Código asincrónico se escala mejor.

   [!code-csharp[Startup](get-started/sample/Hubs/ChatHub.cs?range=6-12)]

-----

## <a name="configure-the-project-to-use-signalr"></a>Configurar el proyecto para que use SignalR

El servidor de SignalR debe configurarse para que sepa para pasar las solicitudes para SignalR.

1. Para configurar un proyecto de SignalR, modifique el proyecto `Startup.ConfigureServices` método.

   `services.AddSignalR` Agrega SignalR como parte de la [middleware](xref:fundamentals/middleware/index) canalización.

2. Configurar las rutas a los concentradores con `UseSignalR`.


   [!code-csharp[Startup](get-started/sample/Startup.cs?highlight=37,57-60)]


## <a name="create-the-signalr-client-code"></a>Crear el código de cliente de SignalR

1. Agregue un archivo de JavaScript denominado *chat.js*, a la *wwwroot\js* carpeta. Agregue el código siguiente a él:

   [!code-javascript[Index](get-started/sample/wwwroot/js/chat.js)]

2. Reemplace el contenido de *Pages\Index.cshtml* con el código siguiente:

   [!code-cshtml[Index](get-started/sample/Pages/Index.cshtml)]

   El código HTML anterior muestra el nombre y campos del mensaje y un botón de envío. Tenga en cuenta las referencias de script en la parte inferior: una referencia a SignalR y *chat.js*.


## <a name="run-the-app"></a>Ejecutar la aplicación

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

1. Seleccione **depurar** > **iniciar sin depurar** para iniciar un explorador y cargar el sitio Web de forma local. Copie la dirección URL de la barra de direcciones.

1. Abra otra instancia del explorador (cualquier explorador) y pegue la dirección URL en la barra de direcciones.

1. Elija cualquiera de estos exploradores, escriba un nombre y un mensaje y haga clic en el **enviar** botón. El nombre y el mensaje se muestran en ambas páginas al instante.

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

1. Presione **Depurar** (F5) para compilar y ejecutar el programa. Ejecutar el programa, abre una ventana del explorador.

1. Abrir otra ventana del explorador y cargar el sitio Web de forma local en ella.

1. Elija cualquiera de estos exploradores, escriba un nombre y un mensaje y haga clic en el **enviar** botón. El nombre y el mensaje se muestran en ambas páginas al instante.

---

  ![Soluciones](get-started/_static/signalr-get-started-finished.png)

## <a name="related-resources"></a>Recursos relacionados

[Introducción a ASP.NET Core SignalR](introduction.md)
