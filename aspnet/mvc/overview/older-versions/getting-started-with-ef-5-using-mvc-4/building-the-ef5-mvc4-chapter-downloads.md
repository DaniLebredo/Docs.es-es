---
uid: mvc/overview/older-versions/getting-started-with-ef-5-using-mvc-4/building-the-ef5-mvc4-chapter-downloads
title: Compilar el capítulo descargas para MVC EF 5 4 tutoriales | Documentos de Microsoft
author: Rick-Anderson
description: La aplicación web de ejemplo de la Universidad de Contoso muestra cómo crear aplicaciones de ASP.NET MVC 4 mediante Code First de Entity Framework 5 y Visual Studio...
ms.author: aspnetcontent
manager: wpickett
ms.date: 07/30/2013
ms.topic: article
ms.assetid: d0a89089-eed8-4f61-a478-c5ffa30186f5
ms.technology: dotnet-mvc
ms.prod: .net-framework
msc.legacyurl: /mvc/overview/older-versions/getting-started-with-ef-5-using-mvc-4/building-the-ef5-mvc4-chapter-downloads
msc.type: authoredcontent
ms.openlocfilehash: bda7effabd715e4658d2472e1f0a66d7bba18cab
ms.sourcegitcommit: f8852267f463b62d7f975e56bea9aa3f68fbbdeb
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2018
---
<a name="building-the-chapter-downloads-for-the-ef-5-mvc-4-tutorials"></a>Compilar el capítulo descargas para MVC EF 5 4 tutoriales
====================
por [Rick Anderson](https://github.com/Rick-Anderson)

[Descargar el proyecto completado](http://code.msdn.microsoft.com/Getting-Started-with-dd0e2ed8)

> La aplicación web de ejemplo de la Universidad de Contoso muestra cómo crear aplicaciones de ASP.NET MVC 4 con Code First de Entity Framework 5 y Visual Studio 2012. Para obtener información sobre la serie de tutoriales, consulte [el primer tutorial de la serie](creating-an-entity-framework-data-model-for-an-asp-net-mvc-application.md).


## <a name="building-the-chapter-downloads"></a>Creación de las descargas de capítulo

1. Descargue y descomprima el archivo zip de ejemplo de proyecto. En el paquete de descarga descomprimida, encontrará los archivos zip adicionales, uno para la finalización de cada capítulo.
2. Haga clic con el botón secundario en el archivo zip deseado, haga clic en **propiedades**y haga clic en el **Unblock** botón.  
  
    ![](building-the-ef5-mvc4-chapter-downloads/_static/image1.png)
3. Descomprima el archivo.
4. Haga doble clic en el *CUx.sln* archivo para iniciar Visual Studio.
5. Desde el **herramientas** menú, haga clic en **Administrador de paquetes de biblioteca**, a continuación, **Package Manager Console**.  
  
    ![](building-the-ef5-mvc4-chapter-downloads/_static/image2.png)
6. En la consola de administrador de paquete (PMC), haga clic en **restaurar**.  
  
    ![](building-the-ef5-mvc4-chapter-downloads/_static/image3.png)
7. Salga de Visual Studio.
8. Reinicie Visual Studio, abra el archivo de solución que se cerró en el paso anterior.
9. En la consola de administrador de paquete (PMC), escriba la `Update-Database` comando:  
  
    ![](building-the-ef5-mvc4-chapter-downloads/_static/image4.png)  

    > [!NOTE]
    > Si se produce el error siguiente:  
    >   
    >  *El término 'Update-Database' no se reconoce como el nombre de un cmdlet, función, archivo de script o un programa ejecutable. Compruebe la ortografía del nombre, o si incluyó una ruta de acceso, compruebe que la ruta de acceso es correcta e inténtelo de nuevo.*  
    > Salga y reinicie Visual Studio.

    Se ejecutará cada migración y, a continuación, se ejecutará el método de inicialización. Ahora puede ejecutar la aplicación.

    ![](building-the-ef5-mvc4-chapter-downloads/_static/image5.png)

> [!div class="step-by-step"]
> [Anterior](advanced-entity-framework-scenarios-for-an-mvc-web-application.md)
