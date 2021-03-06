---
uid: web-forms/overview/ajax-control-toolkit/animation/executing-several-animations-at-the-same-time-cs
title: Ejecutar varias animaciones al mismo tiempo (C#) | Documentos de Microsoft
author: wenz
description: El control de animación en el Kit de herramientas de Control de AJAX de ASP.NET no es simplemente un control sino un marco completo para agregar animaciones a un control. Lo que permite para ejecutar severa...
ms.author: aspnetcontent
manager: wpickett
ms.date: 06/02/2008
ms.topic: article
ms.assetid: 219149e1-3ee9-4b79-8fe4-7433f6b7d15b
ms.technology: dotnet-webforms
ms.prod: .net-framework
msc.legacyurl: /web-forms/overview/ajax-control-toolkit/animation/executing-several-animations-at-the-same-time-cs
msc.type: authoredcontent
ms.openlocfilehash: 6ecd822f7fa00a24e97b9aa14888798825624617
ms.sourcegitcommit: f8852267f463b62d7f975e56bea9aa3f68fbbdeb
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2018
---
<a name="executing-several-animations-at-the-same-time-c"></a>Ejecutar varias animaciones al mismo tiempo (C#)
====================
por [Christian Wenz](https://github.com/wenz)

[Descargar código](http://download.microsoft.com/download/f/9/a/f9a26acd-8df4-4484-8a18-199e4598f411/Animation2.cs.zip) o [descarga de PDF](http://download.microsoft.com/download/6/7/1/6718d452-ff89-4d3f-a90e-c74ec2d636a3/animation2CS.pdf)

> El control de animación en el Kit de herramientas de Control de AJAX de ASP.NET no es simplemente un control sino un marco completo para agregar animaciones a un control. Lo que permite para ejecutar varias animaciones de forma paralela.


## <a name="overview"></a>Información general

El control de animación en el Kit de herramientas de Control de AJAX de ASP.NET no es simplemente un control sino un marco completo para agregar animaciones a un control. Lo que permite para ejecutar varias animaciones de forma paralela.

## <a name="steps"></a>Pasos

En primer lugar, se incluyen el `ScriptManager` en la página; a continuación, AJAX de ASP.NET se carga la biblioteca, lo que permite usar el Kit de herramientas de Control:

[!code-aspx[Main](executing-several-animations-at-the-same-time-cs/samples/sample1.aspx)]

La animación se aplicará a un panel de texto que el siguiente aspecto:

[!code-aspx[Main](executing-several-animations-at-the-same-time-cs/samples/sample2.aspx)]

En la clase CSS asociada para el panel, definir un color de fondo "nice" y también establece un ancho fijo para el panel:

[!code-css[Main](executing-several-animations-at-the-same-time-cs/samples/sample3.css)]

A continuación, agregue el `AnimationExtender` a la página, proporciona un `ID`, el `TargetControlID` atributo y el obligatoria `runat="server"`:

[!code-aspx[Main](executing-several-animations-at-the-same-time-cs/samples/sample4.aspx)]

En el `<Animations>` nodo, use `<OnLoad>` para ejecutar las animaciones una vez que la página se ha cargado completamente. Por lo general, `<OnLoad>` solo acepta una animación. El marco de trabajo de animación le permite unir varios animaciones en uno solo mediante la `<Parallel>` elemento. Todas las animaciones en `<Parallel>` se ejecutan al mismo tiempo.

Aquí es el marcado posibles para el `AnimationExtender` control desaparecer gradualmente y el tamaño del panel a la vez:

[!code-aspx[Main](executing-several-animations-at-the-same-time-cs/samples/sample5.aspx)]

Y realmente: al ejecutar este script, el panel se muestra, a continuación, se cambia el tamaño (más de threefolding su ancho y halfing su alto) y fundido de salida al mismo tiempo.


[![El panel se atenúa y cambiar el tamaño (incluido su contenido, gracias al motor de representación del explorador)](executing-several-animations-at-the-same-time-cs/_static/image2.png)](executing-several-animations-at-the-same-time-cs/_static/image1.png)

El panel se atenúa y cambiar el tamaño (incluido su contenido, gracias al motor de representación del explorador) ([haga clic aquí para ver la imagen a tamaño completo](executing-several-animations-at-the-same-time-cs/_static/image3.png))

> [!div class="step-by-step"]
> [Anterior](adding-animation-to-a-control-cs.md)
> [Siguiente](executing-several-animations-after-each-other-cs.md)
