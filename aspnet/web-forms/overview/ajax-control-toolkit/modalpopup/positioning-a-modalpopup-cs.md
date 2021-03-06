---
uid: web-forms/overview/ajax-control-toolkit/modalpopup/positioning-a-modalpopup-cs
title: Colocar un ModalPopup (C#) | Documentos de Microsoft
author: wenz
description: El control ModalPopup en el Kit de herramientas de Control de AJAX ofrece una manera sencilla de crear un elemento emergente modal mediante medios de lado cliente. Sin embargo el control no proporciona un...
ms.author: aspnetcontent
manager: wpickett
ms.date: 06/02/2008
ms.topic: article
ms.assetid: 1caac9d0-e21e-49d6-a8ff-e563a736d6ca
ms.technology: dotnet-webforms
ms.prod: .net-framework
msc.legacyurl: /web-forms/overview/ajax-control-toolkit/modalpopup/positioning-a-modalpopup-cs
msc.type: authoredcontent
ms.openlocfilehash: bee5be84259231d8cd5efde74b610d72f5e250cc
ms.sourcegitcommit: f8852267f463b62d7f975e56bea9aa3f68fbbdeb
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2018
---
<a name="positioning-a-modalpopup-c"></a>Colocar un ModalPopup (C#)
====================
por [Christian Wenz](https://github.com/wenz)

[Descargar código](http://download.microsoft.com/download/2/4/0/24052038-f942-4336-905b-b60ae56f0dd5/ModalPopup4.cs.zip) o [descarga de PDF](http://download.microsoft.com/download/b/6/a/b6ae89ee-df69-4c87-9bfb-ad1eb2b23373/modalpopup4CS.pdf)

> El control ModalPopup en el Kit de herramientas de Control de AJAX ofrece una manera sencilla de crear un elemento emergente modal mediante medios de lado cliente. Sin embargo el control proporcionan una funcionalidad integrada para colocar la ventana emergente.


## <a name="overview"></a>Información general

El control ModalPopup en el Kit de herramientas de Control de AJAX ofrece una manera sencilla de crear un elemento emergente modal mediante medios de lado cliente. Sin embargo el control proporcionan una funcionalidad integrada para colocar la ventana emergente.

## <a name="steps"></a>Pasos

Para activar la funcionalidad de AJAX de ASP.NET y el Kit de herramientas de Control, el `ScriptManager`. control se debe colocar en cualquier sitio en la página (pero en el `<form>` elemento):

[!code-aspx[Main](positioning-a-modalpopup-cs/samples/sample1.aspx)]

A continuación, agregue un panel que actúa como el elemento emergente modal. Se utiliza un botón para cerrar la ventana emergente:

[!code-aspx[Main](positioning-a-modalpopup-cs/samples/sample2.aspx)]

Cada vez que se muestra la ventana emergente, se deben colocarse en un lugar determinado en la página. Para esta tarea, se crea una función de JavaScript del lado cliente. Primero intenta tener acceso al panel. Si se realiza correctamente, la posición del panel se establece con CSS y JavaScript (cambiar la posición de la ventana emergente en le). Sin embargo, el `ModalPopupExtender` control también intenta colocar la ventana emergente. Por lo tanto, el código de JavaScript coloca varias veces en la ventana emergente, cada décima parte de un segundo.

[!code-html[Main](positioning-a-modalpopup-cs/samples/sample3.html)]

Como puede ver, el valor devuelto de la `setTimeout()` método JavaScript se guarda en una variable global. Esto permite detener la posición repetidos de la ventana emergente a petición, con el `clearTimeout()` método:

[!code-javascript[Main](positioning-a-modalpopup-cs/samples/sample4.js)]

Ahora todo lo que queda por hacer es hacer que el explorador llamar a estas funciones siempre que sea adecuado. El `movePanel()` debe llamar la función de JavaScript cuando se presiona el botón que desencadena el panel:

[!code-aspx[Main](positioning-a-modalpopup-cs/samples/sample5.aspx)]

Y el `stopMoving()` función entra en juego cuando la ventana emergente se cierra se pueden activar en el `ModalPopupExtender` control:

[!code-aspx[Main](positioning-a-modalpopup-cs/samples/sample6.aspx)]


[![La ventana emergente modal aparece en la posición designada](positioning-a-modalpopup-cs/_static/image2.png)](positioning-a-modalpopup-cs/_static/image1.png)

La ventana emergente modal aparece en la posición designada ([haga clic aquí para ver la imagen a tamaño completo](positioning-a-modalpopup-cs/_static/image3.png))

> [!div class="step-by-step"]
> [Anterior](handling-postbacks-from-a-modalpopup-cs.md)
> [Siguiente](launching-a-modal-popup-window-from-server-code-vb.md)
