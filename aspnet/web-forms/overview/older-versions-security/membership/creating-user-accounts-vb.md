---
uid: web-forms/overview/older-versions-security/membership/creating-user-accounts-vb
title: Crear cuentas de usuario (VB) | Documentos de Microsoft
author: rick-anderson
description: En este tutorial se explorará mediante el marco de trabajo de pertenencia (a través de SqlMembershipProvider) para crear nuevas cuentas de usuario. Veremos cómo crear nuevos us...
ms.author: aspnetcontent
manager: wpickett
ms.date: 01/18/2008
ms.topic: article
ms.assetid: 9ef3e893-bebe-4b13-9fe5-8b71720dd85e
ms.technology: dotnet-webforms
ms.prod: .net-framework
msc.legacyurl: /web-forms/overview/older-versions-security/membership/creating-user-accounts-vb
msc.type: authoredcontent
ms.openlocfilehash: d665e7ba43401da76a88a904c10a587aa4576d4b
ms.sourcegitcommit: f8852267f463b62d7f975e56bea9aa3f68fbbdeb
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2018
---
<a name="creating-user-accounts-vb"></a>Crear cuentas de usuario (VB)
====================
por [Scott Mitchell](https://twitter.com/ScottOnWriting)

[Descargar código](http://download.microsoft.com/download/3/f/5/3f5a8605-c526-4b34-b3fd-a34167117633/ASPNET_Security_Tutorial_05_VB.zip) o [descarga de PDF](http://download.microsoft.com/download/3/f/5/3f5a8605-c526-4b34-b3fd-a34167117633/aspnet_tutorial05_CreatingUsers_vb.pdf)

> En este tutorial se explorará mediante el marco de trabajo de pertenencia (a través de SqlMembershipProvider) para crear nuevas cuentas de usuario. Veremos cómo crear nuevos usuarios mediante programación y a través de ASP. Control de CreateUserWizard integradas de NET.


## <a name="introduction"></a>Introducción

En el <a id="_msoanchor_1"> </a> [tutorial anterior](creating-the-membership-schema-in-sql-server-vb.md) se instala el esquema de servicios de aplicación en una base de datos, que agrega las tablas, vistas y almacena los procedimientos necesarios para la `SqlMembershipProvider` y `SqlRoleProvider`. Esto crea la infraestructura que necesitamos para el resto de los tutoriales de esta serie. En este tutorial se explorará mediante el marco de pertenencia (a través de la `SqlMembershipProvider`) para crear nuevas cuentas de usuario. Veremos cómo crear nuevos usuarios mediante programación y a través de ASP. Control de CreateUserWizard integradas de NET.

Además de obtener información sobre cómo crear nuevas cuentas de usuario, también se necesitará actualizar el sitio Web de demostración se crea por primera vez en el *<a id="_msoanchor_2"> </a> [una visión general de autenticación mediante formularios](../introduction/an-overview-of-forms-authentication-vb.md)* tutorial y, a continuación, ha mejorado en la *<a id="_msoanchor_3"> </a> [configuración de autenticación de formularios y temas avanzados](../introduction/forms-authentication-configuration-and-advanced-topics-vb.md)* tutorial. Nuestra aplicación de demostración web tiene una página de inicio de sesión que valida las credenciales de los usuarios con pares de nombre de usuario/contraseña codificados de forma rígida. Además, `Global.asax` incluye código que crea personalizado `IPrincipal` y `IIdentity` objetos para los usuarios autenticados. Se actualizará la página de inicio de sesión para validar las credenciales de los usuarios con el marco de trabajo de pertenencia y quite la lógica personalizada de principal e identity.

Comencemos.

## <a name="the-forms-authentication-and-membership-checklist"></a>La autenticación de formularios y la lista de comprobación de pertenencia

Antes de comenzar trabajar con el marco de trabajo de pertenencia, dedique un momento para revisar los pasos importantes que se le hemos llevado para llegar a este punto. Cuando se utiliza el marco de trabajo de pertenencia con el `SqlMembershipProvider` en un escenario de autenticación basada en formularios, los pasos siguientes deben realizarse antes de implementar la funcionalidad de pertenencia en la aplicación web:

1. **Habilitar la autenticación basada en formularios.** Como se explicó en  *<a id="_msoanchor_4"> </a> [una visión general de autenticación mediante formularios](../introduction/an-overview-of-forms-authentication-vb.md)*, está habilitada la autenticación de formularios mediante la edición de `Web.config` y estableciendo el `<authentication>` del elemento `mode` atribuir a `Forms`. Con la autenticación de formularios habilitada, se examina cada solicitud entrante para una *vale de autenticación de formularios*, que, si está presente, identifica el solicitante.
2. **Agregar el esquema de servicios de aplicación a la base de datos adecuado.** Cuando se usa el `SqlMembershipProvider` es necesario instalar el esquema de servicios de aplicación a una base de datos. Normalmente, este esquema se agrega a la misma base de datos que contiene el modelo de datos de la aplicación. El *<a id="_msoanchor_5"> </a> [crear el esquema de pertenencia en SQL Server](creating-the-membership-schema-in-sql-server-vb.md)* tutorial examinando utilizando el `aspnet_regsql.exe` herramienta para lograr esto.
3. **Personalizar la configuración de la aplicación Web para hacer referencia a la base de datos del paso 2.** El *crear el esquema de pertenencia en SQL Server* tutorial se han mostrado dos formas de configurar la aplicación web para que la `SqlMembershipProvider` utilizaría la base de datos seleccionada en el paso 2: modificando el `LocalSqlServer` nombre de la cadena de conexión; o agregando un nuevo proveedor registrado a la lista de proveedores de pertenencia de framework y personalizar ese nuevo proveedor para usar la base de datos desde el paso 2.

Cuando cree una aplicación web que utiliza el `SqlMembershipProvider` y autenticación basada en formularios, deberá realizar estos tres pasos antes de usar la `Membership` clase o los controles Web de inicio de sesión de ASP.NET. Puesto que ya se realizan estos pasos en los tutoriales anteriores, estamos preparados para empezar a usar el marco de trabajo de pertenencia.

## <a name="step-1-adding-new-aspnet-pages"></a>Paso 1: Agregar nuevas páginas ASP.NET

En este tutorial y los tres siguientes analizaremos varios relacionados con la pertenencia a funciones y capacidades. Necesitamos una serie de páginas ASP.NET que se va a implementar en los temas que se examinan a lo largo de estos tutoriales. Vamos a crear dichas páginas y, a continuación, en un archivo de asignación de sitio `(Web.sitemap)`.

Empiece por crear una nueva carpeta en el proyecto denominado `Membership`. A continuación, agregue cinco nuevas páginas ASP.NET para la `Membership` carpeta, vincular cada página con el `Site.master` página maestra. Nombre de las páginas:

- `CreatingUserAccounts.aspx`
- `UserBasedAuthorization.aspx`
- `EnhancedCreateUserWizard.aspx`
- `AdditionalUserInfo.aspx`
- `Guestbook.aspx`

En este momento, el Explorador de soluciones del proyecto debe ser similar a la pantalla se muestra en la figura 1.


[![Se agregaron cinco nuevas páginas a la carpeta de pertenencia](creating-user-accounts-vb/_static/image2.png)](creating-user-accounts-vb/_static/image1.png)

**Figura 1**: cinco nuevas páginas se han agregado a la `Membership` carpeta ([haga clic aquí para ver la imagen a tamaño completo](creating-user-accounts-vb/_static/image3.png))


Cada página en este punto, debe tener los dos controles de contenido, uno para cada uno de ContentPlaceHolders la página maestra: `MainContent` y `LoginContent`.

[!code-aspx[Main](creating-user-accounts-vb/samples/sample1.aspx)]

Recuerde que el `LoginContent` marcado de forma predeterminada del ContentPlaceHolder muestra un vínculo para iniciar sesión o cerrar la sesión del sitio, dependiendo de si el usuario está autenticado. La presencia de la `Content2` control de contenido, sin embargo, reemplaza el marcado de la página maestra predeterminada. Como se explicó en *<a id="_msoanchor_6"> </a> [una visión general de autenticación mediante formularios](../introduction/an-overview-of-forms-authentication-vb.md)* tutorial, esto es útil en las páginas donde no queremos mostrar opciones relacionadas con el inicio de sesión en la columna izquierda.

Para estos cinco páginas, sin embargo, queremos que aparezca marcado de la página maestra predeterminada para el `LoginContent` ContentPlaceHolder. Por lo tanto, quite el marcado declarativo para la `Content2` control de contenido. Una vez hecho esto, cada uno de los cinco marcado de la página debe contener un solo control de contenido.

## <a name="step-2-creating-the-site-map"></a>Paso 2: Crear el mapa del sitio

Salvo los sitios Web más triviales, todas deben implementar alguna forma de una interfaz de usuario de navegación. La interfaz de usuario de navegación puede ser una simple lista de vínculos a las distintas secciones del sitio. Como alternativa, pueden organizar estos vínculos en los menús o vistas de árbol. Como los desarrolladores de páginas, la creación de la interfaz de usuario de navegación es solo la mitad del proceso. También se necesita algún medio para definir la estructura del sitio lógico de una manera fácil de mantener y actualizable. Como páginas nuevas se agregan o quitan las páginas existentes, queremos que pueda actualizar un único origen - el mapa del sitio - y reflejarán las modificaciones a través de la interfaz de usuario de navegación del sitio.

Estas dos tareas - definir el mapa del sitio e implementar una interfaz de usuario de navegación basada en el mapa del sitio - son fáciles de realizar gracias a la plataforma de mapa del sitio y la exploración Web controla agregado en ASP.NET versión 2.0. El marco de mapa del sitio permite que un desarrollador definir un mapa del sitio y, a continuación, tener acceso a ella a través de una API de programación (el [ `SiteMap` clase](https://msdn.microsoft.com/library/system.web.sitemap.aspx)). Integrado en la exploración Web controla incluyen un [control de menú](https://msdn.microsoft.com/library/bz09dy46.aspx), [TreeView (control)](https://msdn.microsoft.com/library/3eafky27.aspx)y el [control SiteMapPath](https://msdn.microsoft.com/library/3eafky27.aspx).

Como los marcos de pertenencia y los Roles, el marco de mapa del sitio se basa sobre la [modelo de proveedor](http://aspnet.4guysfromrolla.com/articles/101905-1.aspx). El trabajo de la clase de proveedor de mapa del sitio consiste en generar la estructura en la memoria utilizada por la `SiteMap` clase a partir de un almacén de datos persistente, como un archivo XML o una tabla de base de datos. .NET Framework se suministra con un proveedor de mapa del sitio predeterminado que lee los datos del mapa del sitio desde un archivo XML ([`XmlSiteMapProvider`](https://msdn.microsoft.com/library/system.web.xmlsitemapprovider.aspx)), y éste es el proveedor que se va a utilizar en este tutorial. Para algunas implementaciones de proveedor de mapa del sitio alternativos, consulte la sección de lecturas adicionales al final de este tutorial.

El proveedor de mapa del sitio predeterminado espera un archivo XML con un formato correcto denominado `Web.sitemap` que existe el directorio raíz. Puesto que estamos usando este proveedor de forma predeterminada, es necesario agregar un archivo y definir la estructura del mapa del sitio con el formato XML adecuado. Para agregar el archivo, haga doble clic en el nombre del proyecto en el Explorador de soluciones y elija Agregar nuevo elemento. En el cuadro de diálogo, optar por agregar un archivo de mapa del sitio con el nombre de tipo `Web.sitemap`.


[![Agregue un archivo denominado Web.sitemap al directorio raíz del proyecto](creating-user-accounts-vb/_static/image5.png)](creating-user-accounts-vb/_static/image4.png)

**Figura 2**: agregue un archivo denominado `Web.sitemap` al directorio raíz del proyecto ([haga clic aquí para ver la imagen a tamaño completo](creating-user-accounts-vb/_static/image6.png))


El archivo de asignación de sitio XML define la estructura del sitio Web como una jerarquía. Esta relación jerárquica se modela en el archivo XML a través del linaje de los `<siteMapNode>` elementos. El `Web.sitemap` debe comenzar con un `<siteMap>` nodo primario que tiene exactamente una `<siteMapNode>` secundarios. Este nivel superior `<siteMapNode>` elemento representa la raíz de la jerarquía y puede tener un número arbitrario de los nodos descendientes. Cada `<siteMapNode>` elemento debe incluir un `title` de atributo y, opcionalmente, puede incluir `url` y `description` atributos, entre otros; cada usuario que no esté vacía `url` atributo debe ser único.

Escriba el siguiente código XML en el `Web.sitemap` archivo:

[!code-xml[Main](creating-user-accounts-vb/samples/sample2.xml)]

El marcado de asignación de sitio anterior define la jerarquía que se muestra en la figura 3.


[![El mapa del sitio representa una estructura jerárquica de navegación](creating-user-accounts-vb/_static/image8.png)](creating-user-accounts-vb/_static/image7.png)

**Figura 3**: el mapa del sitio representa una estructura jerárquica de navegación ([haga clic aquí para ver la imagen a tamaño completo](creating-user-accounts-vb/_static/image9.png))


## <a name="step-3-updating-the-master-page-to-include-a-navigational-user-interface"></a>Paso 3: Actualizar la página maestra para incluir una interfaz de usuario de navegación

ASP.NET incluye una serie de controles de Web relacionados con la navegación para diseñar una interfaz de usuario. Estos incluyen el menú, vista de árbol y los controles SiteMapPath. Los controles de menú y la vista de árbol representan la estructura de mapa del sitio en un menú o un árbol, respectivamente, mientras que la SiteMapPath muestra una ruta de navegación que muestra el nodo actual que se está visitando, así como sus antecesores. Los datos del mapa del sitio se puede enlazar a otros datos de Web se controla mediante el SiteMapDataSource y puede obtenerse acceso mediante programación a través de la `SiteMap` clase.

Puesto que una discusión detallada de la plataforma de mapa del sitio y los controles de navegación está fuera del ámbito de esta serie de tutoriales, en lugar de dedicar tiempo a elaborar vamos a nuestra propia interfaz de usuario de navegación en su lugar pedir prestado el utilizado en mi *[ Trabajar con datos en ASP.NET 2.0](../../data-access/index.md)* serie de tutoriales, que utiliza un control de repetidor para mostrar una lista con viñetas de dos profunda de los vínculos de navegación, como se muestra en la figura 4.

### <a name="adding-a-two-level-list-of-links-in-the-left-column"></a>Agregar una lista de dos niveles de vínculos en la columna izquierda

Para crear esta interfaz, agregue el siguiente marcado declarativo para la `Site.master` columna izquierda de la página maestra donde el texto de la lista de tareas: menú va aquí... reside actualmente.

[!code-aspx[Main](creating-user-accounts-vb/samples/sample3.aspx)]

El marcado anterior enlaza un control de repetidor denominado `menu` a un SiteMapDataSource, que devuelve la jerarquía de mapa del sitio definida en `Web.sitemap`. Desde el control SiteMapDataSource [ `ShowStartingNode` propiedad](https://msdn.microsoft.com/library/system.web.ui.webcontrols.sitemapdatasource.showstartingnode.aspx) está establecida en False se inicia devolver la jerarquía del mapa del sitio a partir de los descendientes del nodo principal. Muestra el repetidor cada uno de estos nodos (actualmente solo pertenencia) en un `<li>` elemento. Repetidor interna y otro, a continuación, muestra los elementos secundarios del nodo actual en una lista sin ordenar anidada.

La figura 4 muestra la salida representada del marcado anterior con la estructura de mapa del sitio que se creó en el paso 2. Repetidor representa el marcado de vainilla lista sin ordenar; las reglas de hojas de estilos en cascada definen en `Styles.css` son responsables del diseño estéticamente agradable. Para obtener una descripción más detallada de cómo funciona el marcado anterior, consulte la [páginas maestras y navegación del sitio](https://asp.net/learn/data-access/tutorial-03-vb.aspx) tutorial.


[![La interfaz de usuario de navegación es representar utilizando anidados sin ordenar listas](creating-user-accounts-vb/_static/image11.png)](creating-user-accounts-vb/_static/image10.png)

**Figura 4**: la interfaz de usuario de navegación es representar utilizando anidados sin ordenar listas ([haga clic aquí para ver la imagen a tamaño completo](creating-user-accounts-vb/_static/image12.png))


### <a name="adding-breadcrumb-navigation"></a>Agregar ruta de navegación

Además de la lista de vínculos en la columna izquierda, vamos a también que cada página muestre un [ruta de navegación](http://en.wikipedia.org/wiki/Breadcrumb_%28navigation%29). Una ruta de navegación es un elemento de la interfaz de usuario de navegación que muestra rápidamente a los usuarios de su posición actual dentro de la jerarquía de sitios. El control SiteMapPath usa el marco de mapa del sitio para determinar la ubicación de la página actual en el mapa del sitio y, a continuación, muestra una ruta de navegación en función de esta información.

En concreto, agregar un `<span>` elemento de encabezado de la página maestra `<div>` elemento y establecer la nueva `<span>` del elemento `class` atribuir a la ruta de navegación. (El `Styles.css` clase contiene una regla para una clase de ruta de navegación.) A continuación, agregue un SiteMapPath a este nuevo `<span>` elemento.

[!code-aspx[Main](creating-user-accounts-vb/samples/sample4.aspx)]

Figura 5 se muestra la salida de SiteMapPath al visitar `~/Membership/CreatingUserAccounts.aspx`.


[![La ruta de navegación muestra la página actual y sus antecesores en el sitio de mapa](creating-user-accounts-vb/_static/image14.png)](creating-user-accounts-vb/_static/image13.png)

**Figura 5**: la ruta de navegación muestra la página actual y sus antecesores en el mapa del sitio ([haga clic aquí para ver la imagen a tamaño completo](creating-user-accounts-vb/_static/image15.png))


## <a name="step-4-removing-the-custom-principal-and-identity-logic"></a>Paso 4: Quitar la entidad de seguridad personalizada y la lógica de identidad

En el *<a id="_msoanchor_7"> </a> [configuración de autenticación de formularios y temas avanzados](../introduction/forms-authentication-configuration-and-advanced-topics-vb.md)* tutorial hemos visto cómo asociar objetos principal e identity personalizados al usuario autenticado. Esto se logra mediante la creación de un controlador de eventos en `Global.asax` para la aplicación `PostAuthenticateRequest` evento, que se activa tras la `FormsAuthenticationModule` ha autenticado al usuario. En este controlador de eventos se ha reemplazado el `GenericPrincipal` y `FormsIdentity` objetos agregados por el `FormsAuthenticationModule` con el `CustomPrincipal` y `CustomIdentity` objetos hemos creado en este tutorial.

Mientras que los objetos principal e identity personalizados son útiles en determinados escenarios, en la mayoría de los casos la `GenericPrincipal` y `FormsIdentity` objetos son suficientes. Por lo tanto, creo que sería merece la pena volver al comportamiento predeterminado. Realice este cambio quitando o marcando como comentario el `PostAuthenticateRequest` controlador de eventos o eliminando el `Global.asax` archivo completamente.

> [!NOTE]
> Una vez que haya marcado como comentario o quita el código de `Global.asax`, tendrá que marcar con comentarios el código en `Default.aspx's` clase de código subyacente que convierte el `User.Identity` propiedad a un `CustomIdentity` instancia.


## <a name="step-5-programmatically-creating-a-new-user"></a>Paso 5: Crear mediante programación un nuevo usuario

Para crear una nueva cuenta de usuario mediante el uso de framework de pertenencia a la `Membership` la clase [ `CreateUser` método](https://msdn.microsoft.com/library/system.web.security.membership.createuser.aspx). Este método tiene parámetros para el nombre de usuario, contraseña y otros campos relacionados con el usuario de entrada. En la invocación, se delega la creación de la nueva cuenta de usuario para el proveedor de pertenencia configurado y, a continuación, devuelve un [ `MembershipUser` objeto](https://msdn.microsoft.com/library/system.web.security.membershipuser.aspx) que representa la cuenta de usuario recién creado.

El `CreateUser` método tiene cuatro sobrecargas, cada uno de ellos acepta un número diferente de parámetros de entrada:

- [`CreateUser(username, password)`](https://msdn.microsoft.com/library/d8t4h2es.aspx)
- [`CreateUser(username, password, email)`](https://msdn.microsoft.com/library/t8yy6w3h.aspx)
- [`CreateUser(username, password, email, passwordQuestion, passwordAnswer, isApproved, MembershipCreateStatus)`](https://msdn.microsoft.com/library/82xx2e62.aspx)
- [`CreateUser(username, password, email, passwordQuestion, passwordAnswer, isApproved, providerUserKey, MembershipCreateStatus)`](https://msdn.microsoft.com/library/ms152012.aspx)

Estos cuatro sobrecargas difieren en la cantidad de información que se recopila. La primera sobrecarga, por ejemplo, requiere solo el nombre de usuario y la contraseña para la nueva cuenta de usuario, mientras que la segunda requiere también la dirección de correo electrónico del usuario.

Estas sobrecargas existen porque la información necesaria para crear una nueva cuenta de usuario depende de los valores de configuración del proveedor de pertenencia. En el *<a id="_msoanchor_8"> </a> [crear el esquema de pertenencia en SQL Server](creating-the-membership-schema-in-sql-server-vb.md)* tutorial examinamos especificando opciones de configuración de proveedor de pertenencia en `Web.config`. Tabla 2 incluye una lista completa de los valores de configuración.

Una tal configuración del proveedor de pertenencia a configuración que afecta a lo que `CreateUser` se pueden usar las sobrecargas es la `requiresQuestionAndAnswer` configuración. Si `requiresQuestionAndAnswer` se establece en `true` (valor predeterminado), a continuación, al crear una nueva cuenta de usuario se debe especificar una pregunta de seguridad y la respuesta. Esta información se utiliza posteriormente si el usuario tiene que restablecer o cambiar su contraseña. En concreto, en ese momento se muestran la pregunta de seguridad y debe escribir la respuesta correcta para poder restablecer o cambiar su contraseña. Por lo tanto, si la `requiresQuestionAndAnswer` está establecido en `true` , a continuación, llamar a cualquiera de las dos primeras `CreateUser` sobrecargas producen una excepción porque faltan la pregunta de seguridad y la respuesta. Puesto que nuestra aplicación actualmente está configurado para requerir una pregunta de seguridad y la respuesta, se deberá usar una de las dos sobrecargas este últimas al crear mediante programación del usuario.

Para ilustrar con el `CreateUser` método, vamos a crear una interfaz de usuario donde se pedirá al usuario su nombre, contraseña, correo electrónico y una respuesta a una pregunta de seguridad predefinidos. Abra la `CreatingUserAccounts.aspx` página en el `Membership` carpeta y agregue los siguientes controles Web para el control de contenido:

- Un cuadro de texto denominado `Username`
- Un cuadro de texto denominado `Password`, cuyo `TextMode` propiedad está establecida en `Password`
- Un cuadro de texto denominado `Email`
- Una etiqueta denominada `SecurityQuestion` con su `Text` propiedad se ha retirado
- Un cuadro de texto denominado `SecurityAnswer`
- Un botón denominado `CreateAccountButton` cuyo `Text` propiedad está establecida en crear la cuenta de usuario
- Un control Label denominado `CreateAccountResults` con su `Text` propiedad se ha retirado

En este momento la pantalla debe ser similar a la pantalla se muestra en la figura 6.


[![Agregue los distintos controles Web a la página CreatingUserAccounts.aspx](creating-user-accounts-vb/_static/image17.png)](creating-user-accounts-vb/_static/image16.png)

**Figura 6**: agregar los distintos controles Web a la `CreatingUserAccounts.aspx Page` ([haga clic aquí para ver la imagen a tamaño completo](creating-user-accounts-vb/_static/image18.png))


El `SecurityQuestion` etiqueta y `SecurityAnswer` cuadro de texto están diseñados para mostrar una pregunta de seguridad predefinidos y recopilar la respuesta del usuario. Tenga en cuenta que la pregunta de seguridad y la respuesta se almacenan en una base por usuario, por lo que es posible permitir que cada usuario definir su propia pregunta de seguridad. Sin embargo, en este ejemplo he decidido volver a utilizar una pregunta de seguridad universal, a saber: ¿cuál es su color favorito?

Para implementar esta pregunta de seguridad predefinidos, agregue una constante a la clase de código subyacente de la página denominada `passwordQuestion`, asignar la pregunta de seguridad. A continuación, en la `Page_Load` controlador de eventos, asignar esta constante para el `SecurityQuestion` etiqueta `Text` propiedad:

[!code-vb[Main](creating-user-accounts-vb/samples/sample5.vb)]

A continuación, cree un controlador de eventos para el `CreateAccountButton'` s `Click` eventos y agregue el código siguiente:

[!code-vb[Main](creating-user-accounts-vb/samples/sample6.vb)]

El `Click` controlador de eventos se inicia mediante la definición de una variable denominada `createStatus` de tipo [ `MembershipCreateStatus` ](https://msdn.microsoft.com/library/system.web.security.membershipcreatestatus.aspx). `MembershipCreateStatus` es una enumeración que indica el estado de la `CreateUser` operación. Por ejemplo, si la cuenta de usuario se ha creado correctamente, resultante `MembershipCreateStatus` instancia se establecerá en un valor de `Success;` por otro lado, si se produce un error en la operación porque ya existe un usuario con el mismo nombre de usuario, se establecerá en un valor de `DuplicateUserName`. En el `CreateUser` sobrecarga se utiliza, es necesario pasar un `MembershipCreateStatus` instancia en el método. Este parámetro se establece en el valor adecuado en el `CreateUser` método y se puede examinar su valor después de la llamada de método para determinar si la cuenta de usuario se creó correctamente.

Después de llamar a `CreateUser`, pasando `createStatus`, `Select Case` instrucción se utiliza para generar un mensaje adecuado según el valor asignado a `createStatus`. Las figuras 7 muestra el resultado cuando se ha creado correctamente un nuevo usuario. Las figuras 8 y 9 se muestra la salida cuando no se crea la cuenta de usuario. En la figura 8, el visitante introducido una contraseña cinco letras, que no cumple los requisitos de seguridad de contraseña establece en los valores de configuración del proveedor de pertenencia. En la figura 9, el visitante está intentando crear una cuenta de usuario con un nombre de usuario existente (el uno creado en la figura 7).


[![Es de una cuenta de usuario se creó correctamente](creating-user-accounts-vb/_static/image20.png)](creating-user-accounts-vb/_static/image19.png)

**Figura 7**: una cuenta de usuario nueva es creado correctamente ([haga clic aquí para ver la imagen a tamaño completo](creating-user-accounts-vb/_static/image21.png))


[![No se creará la cuenta de usuario porque la contraseña proporcionada es demasiado débil](creating-user-accounts-vb/_static/image23.png)](creating-user-accounts-vb/_static/image22.png)

**Figura 8**: no se creará la cuenta de usuario porque la contraseña proporcionada es demasiado débil ([haga clic aquí para ver la imagen a tamaño completo](creating-user-accounts-vb/_static/image24.png))


[![La cuenta de usuario no es que crear porque el nombre de usuario está ya en uso](creating-user-accounts-vb/_static/image26.png)](creating-user-accounts-vb/_static/image25.png)

**Figura 9**: la cuenta de usuario no es crear porque el nombre de usuario está ya en uso ([haga clic aquí para ver la imagen a tamaño completo](creating-user-accounts-vb/_static/image27.png))


> [!NOTE]
> Quizás se pregunte cómo determinar el éxito o error al utilizar uno de los dos primeros `CreateUser` sobrecargas de método, ninguno de ellos tiene un parámetro de tipo `MembershipCreateStatus`. Estas sobrecargas de dos primeras lanzan una [ `MembershipCreateUserException` excepción](https://msdn.microsoft.com/library/system.web.security.membershipcreateuserexception.aspx) en caso de un error, lo que incluye un [ `StatusCode` propiedad](https://msdn.microsoft.com/library/system.web.security.membershipcreateuserexception.statuscode.aspx) de tipo `MembershipCreateStatus`.


Después de crear algunas cuentas de usuario, compruebe que se han creado las cuentas al enumerar el contenido de la `aspnet_Users` y `aspnet_Membership` tablas en el `SecurityTutorials.mdf` base de datos. Como se muestra en la figura 10, he agregado dos usuarios a través de la `CreatingUserAccounts.aspx` página: Tito y Bruce.


[![Hay dos usuarios en el almacén de usuario de pertenencia: Tito y Bruce](creating-user-accounts-vb/_static/image29.png)](creating-user-accounts-vb/_static/image28.png)

**Figura 10**: hay dos usuarios en el almacén de usuario de pertenencia: Tito y Bruce ([haga clic aquí para ver la imagen a tamaño completo](creating-user-accounts-vb/_static/image30.png))


Mientras que el almacén del usuario de pertenencia incluye ahora información de cuenta Bruce y de Tito, todavía tenemos que implementan funcionalidad que permite Bruce o Tito iniciar sesión en el sitio. Actualmente, `Login.aspx` valida las credenciales del usuario con un conjunto codificado de forma rígida de pares de nombre de usuario/contraseña - hace *no* validar las credenciales proporcionadas en el marco de trabajo de pertenencia. Para ver ahora las nuevas cuentas de usuario en el `aspnet_Users` y `aspnet_Membership` tablas debe ser suficiente. En el siguiente tutorial,  *<a id="_msoanchor_9"> </a> [validar el usuario credenciales en la pertenencia al usuario almacenar](validating-user-credentials-against-the-membership-user-store-vb.md)*, actualizaremos la página de inicio de sesión para validar con el almacén de pertenencia.

> [!NOTE]
> Si no ve ninguno de los usuarios la `SecurityTutorials.mdf` base de datos, puede deberse a que la aplicación web utiliza el proveedor de pertenencia predeterminado, `AspNetSqlMembershipProvider`, que usa el `ASPNETDB.mdf` base de datos como su almacén de usuario. Para determinar si éste es el problema, haga clic en el botón Actualizar en el Explorador de soluciones. Si una base de datos denominada `ASPNETDB.mdf` se ha agregado a la `App_Data` carpeta, éste es el problema. Volver al paso 4 de la *<a id="_msoanchor_10"> </a> [crear el esquema de pertenencia en SQL Server](creating-the-membership-schema-in-sql-server-vb.md)* tutorial para obtener instrucciones sobre cómo configurar correctamente el proveedor de pertenencia.


En la mayoría crear escenarios de la cuenta de usuario, se presenta el visitante con alguna interfaz para escribir su nombre de usuario, contraseña, correo electrónico y otra información esencial, momento en que se crea una nueva cuenta. En este paso se examinando compilar dicha interfaz manualmente y, a continuación, hemos visto cómo utilizar la `Membership.CreateUser` método para agregar mediante programación la nueva cuenta de usuario basada en entradas del usuario. El código, sin embargo, acaba de crear la nueva cuenta de usuario. No realizó ningún seguimiento de las acciones, como inicio de sesión del usuario para el sitio, en la cuenta de usuario recién creado, o enviar un correo electrónico de confirmación al usuario. Estos pasos adicionales requiere código adicional en el botón `Click` controlador de eventos.

ASP.NET se distribuye con el control CreateUserWizard, que está diseñado para controlar el proceso de creación de cuenta de usuario, desde una interfaz de usuario para crear nuevas cuentas de usuario, a crear la cuenta en el marco de trabajo de pertenencia y realizar posteriores a la cuenta de representación tareas de creación, como enviar un correo electrónico de confirmación e iniciando sesión el usuario recién creado en el sitio. Uso del control CreateUserWizard es tan sencillo como arrastrar el control CreateUserWizard del cuadro de herramientas en una página y, a continuación, establecer algunas propiedades. En la mayoría de los casos, no tiene que escribir una sola línea de código. Exploraremos este control formidables en detalle en el paso 6.

Si solo se crean nuevas cuentas de usuario a través de una página web típica de crear una cuenta, es probable que tenga que escribir código que usa el `CreateUser` método, como el control CreateUserWizard probablemente satisfará sus necesidades. Sin embargo, la `CreateUser` método es útil en escenarios en los que tenga una experiencia de usuario muy personalizada de crear una cuenta o cuando necesite crear mediante programación las nuevas cuentas de usuario a través de una interfaz alternativa. Por ejemplo, podría tener una página que permite a un usuario cargar un archivo XML que contiene información de usuario de otra aplicación. La página podría analiza el contenido del XML cargado de archivos y crea una nueva cuenta para cada usuario representado en el código XML mediante una llamada a la `CreateUser` método.

## <a name="step-6-creating-a-new-user-with-the-createuserwizard-control"></a>Paso 6: Crear un nuevo usuario con el Control CreateUserWizard

ASP.NET incluye una serie de controles Web de inicio de sesión. Estos controles ayudan a muchos escenarios comunes de usuario relacionadas con el inicio de sesión y la cuenta. El [control CreateUserWizard](https://quickstarts.asp.net/QuickStartv20/aspnet/doc/ctrlref/login/createuserwizard.aspx) es uno de estos controles está diseñado para presentar una interfaz de usuario para agregar una nueva cuenta de usuario para el marco de trabajo de pertenencia.

Al igual que muchos de los otros controles relacionados con el inicio de sesión Web, CreateUserWizard puede utilizarse sin escribir una sola línea de código. Proporciona una interfaz de usuario basada en valores de configuración del proveedor de pertenencia e internamente llama intuitivamente la `Membership` la clase `CreateUser` método después de que el usuario escribe la información necesaria y haga clic en el botón de Create User. El control CreateUserWizard es muy personalizable. Hay una gran cantidad de eventos que se desencadenan durante varias fases del proceso de creación de cuenta. Podemos crear controladores de eventos, según sea necesario, para insertar lógica personalizada en el flujo de trabajo de creación de cuenta. Además, la apariencia del control CreateUserWizard es muy flexible. Hay una serie de propiedades que definen la apariencia de la interfaz predeterminada; Si es necesario, el control se puede convertir en una plantilla o se pueden agregar pasos de registro de usuario adicionales.

Comencemos con un vistazo al uso de la interfaz predeterminada y el comportamiento del control CreateUserWizard. A continuación, exploraremos cómo personalizar la apariencia a través de eventos y propiedades del control.

### <a name="examining-the-createuserwizards-default-interface-and-behavior"></a>Examen de la interfaz predeterminada y el comportamiento del control CreateUserWizard

Vuelva a la `CreatingUserAccounts.aspx` página en el `Membership` carpeta, ir al modo de diseño o de división y, a continuación, agregue un control CreateUserWizard a la parte superior de la página. El control CreateUserWizard se archiva en la sección de controles de inicio de sesión del cuadro de herramientas. Después de agregar el control, establezca su `ID` propiedad `RegisterUser`. Como la captura de pantalla en la figura 11 muestra, CreateUserWizard representa una interfaz con cuadros de texto para el nuevo usuario nombre de usuario, contraseña, dirección de correo electrónico y pregunta de seguridad y respuesta.


[![Control de CreateUserWizard representa un tipo genérico crea interfaz de usuario](creating-user-accounts-vb/_static/image32.png)](creating-user-accounts-vb/_static/image31.png)

**Figura 11**: el Control CreateUserWizard representa una interfaz de usuario genérica crear ([haga clic aquí para ver la imagen a tamaño completo](creating-user-accounts-vb/_static/image33.png))


Dedique un momento para comparar la interfaz de usuario predeterminado generada por el control CreateUserWizard con la interfaz que hemos creado en el paso 5. Para empezar, el control CreateUserWizard permite el visitante especificar la pregunta de seguridad y la respuesta, mientras que la interfaz creados manualmente usa una pregunta de seguridad predefinidos. Interfaz del control CreateUserWizard también incluye controles de validación, mientras que todavía tuvimos que implementar la validación de campos del formulario de la interfaz. Y la interfaz de control CreateUserWizard incluye un cuadro de texto Confirmar contraseña (junto con un control CompareValidator para asegurarse de que el texto escrito la contraseña y cuadros de texto contraseña comparar son iguales).

¿Qué es interesante es que el control CreateUserWizard consulta valores de configuración del proveedor de pertenencia al representar la interfaz de usuario. Por ejemplo, solo se muestran los cuadros de texto de pregunta y respuesta de seguridad si `requiresQuestionAndAnswer` está establecida en True. Del mismo modo, CreateUserWizard agrega automáticamente un control RegularExpressionValidator para asegurarse de que se cumplen los requisitos de seguridad de contraseña y establece su `ErrorMessage` y `ValidationExpression` propiedades basadas en el `minRequiredPasswordLength`, `minRequiredNonalphanumericCharacters`y `passwordStrengthRegularExpression` opciones de configuración.

El control CreateUserWizard, como su nombre implica, se deriva de la [control Wizard](https://msdn.microsoft.com/library/s2etd1ek.aspx). Asistente para controles están diseñados para proporcionar una interfaz para completar las tareas de varios pasos. Un control de asistente puede tener un número arbitrario de `WizardSteps`, cada uno de los cuales es una plantilla que define el código HTML y controles Web para ese paso. El control Wizard muestra inicialmente la primera `WizardStep`, junto con los controles de navegación que permiten al usuario para continuar desde un paso posterior, o para volver a pasos anteriores.

Como se muestra en el marcado declarativo en la figura 11, interfaz predeterminada del control CreateUserWizard incluye dos `WizardStep` s:

- [`CreateUserWizardStep`](https://msdn.microsoft.com/library/system.web.ui.webcontrols.createuserwizardstep.aspx) ? representa la interfaz para recopilar información para crear la nueva cuenta de usuario. Este es el paso que se muestra en la figura 11.
- [`CompleteWizardStep`](https://msdn.microsoft.com/library/system.web.ui.webcontrols.completewizardstep.aspx) ? Representa un mensaje que indica que la cuenta se ha creado correctamente.

Apariencia y el comportamiento del control CreateUserWizard pueden modificarse mediante la conversión de cualquiera de estos pasos para plantillas o agregando su propia `WizardStep` s. Examinaremos agregando un `WizardStep` a la interfaz de registro en el *almacenar información de usuario adicional* tutorial.

Veamos el control CreateUserWizard en acción. Visite la `CreatingUserAccounts.aspx` página a través de un explorador. Primero escriba algunos valores no válidos en la interfaz del control CreateUserWizard. Intente escribir una contraseña que no se ajusta a los requisitos de seguridad de contraseña, o si deja el cuadro de texto Nombre de usuario vacío. CreateUserWizard mostrará un mensaje de error adecuado. Figura 12 muestra el resultado al intentar crear un usuario con una contraseña suficientemente segura.


[![CreateUserWizard inserta automáticamente los controles de validación](creating-user-accounts-vb/_static/image35.png)](creating-user-accounts-vb/_static/image34.png)

**Figura 12**: The CreateUserWizard automáticamente inserta los controles de validación ([haga clic aquí para ver la imagen a tamaño completo](creating-user-accounts-vb/_static/image36.png))


A continuación, escriba los valores adecuados en CreateUserWizard y haga clic en el botón Crear usuario. Suponiendo que se han especificado los campos obligatorios y seguridad de la contraseña es suficiente, CreateUserWizard creará una nueva cuenta de usuario a través del marco de pertenencia y, a continuación, mostrar el `CompleteWizardStep`de la interfaz (Véase la figura 13). En segundo plano, se llama CreateUserWizard el `Membership.CreateUser` método, al igual que hicimos en el paso 5.


[![Una cuenta de usuario tiene que ha creado correctamente](creating-user-accounts-vb/_static/image38.png)](creating-user-accounts-vb/_static/image37.png)

**Figura 13**: una nueva cuenta de usuario se ha creado correctamente ([haga clic aquí para ver la imagen a tamaño completo](creating-user-accounts-vb/_static/image39.png))


> [!NOTE]
> Como se muestra en la figura 13, la `CompleteWizardStep`de interfaz incluye un botón Continuar. Sin embargo, en este momento al hacer clic en él solo realiza una devolución de datos, dejando el visitante en la misma página. En la personalización del CreateUserWizard apariencia y comportamiento a través de sus propiedades de la sección se examinará cómo puede tener este botón Enviar al visitante a `Default.aspx` (o de otra página).


Después de crear una nueva cuenta de usuario, vuelva a Visual Studio y examine el `aspnet_Users` y `aspnet_Membership` tablas como hicimos en la figura 10 para comprobar que la cuenta se creó correctamente.

### <a name="customizing-the-createuserwizards-behavior-and-appearance-through-its-properties"></a>Personalizar el CreateUserWizard comportamiento y apariencia a través de sus propiedades

CreateUserWizard puede personalizarse en una variedad de formas, mediante las propiedades, `WizardStep` s y controladores de eventos. En esta sección, veremos cómo personalizar la apariencia del control a través de sus propiedades; la siguiente sección se examina extender el comportamiento del control a través de los controladores de eventos.

Prácticamente todo el texto que se muestra en la interfaz de usuario predeterminada del control CreateUserWizard puede personalizarse mediante su gran cantidad de propiedades. Por ejemplo, se pueden personalizar las etiquetas de nombre de usuario, contraseña, Confirmar contraseña, correo electrónico, pregunta de seguridad y respuesta de seguridad que aparece a la izquierda de los cuadros de texto el [ `UserNameLabelText` ](https://msdn.microsoft.com/library/system.web.ui.webcontrols.createuserwizard.usernamelabeltext.aspx), [ `PasswordLabelText` ](https://msdn.microsoft.com/library/system.web.ui.webcontrols.createuserwizard.passwordlabeltext.aspx), [ `ConfirmPasswordLabelText` ](https://msdn.microsoft.com/library/system.web.ui.webcontrols.createuserwizard.confirmpasswordlabeltext.aspx), [ `EmailLabelText` ](https://msdn.microsoft.com/library/system.web.ui.webcontrols.createuserwizard.emaillabeltext.aspx), [ `QuestionLabelText` ](https://msdn.microsoft.com/library/system.web.ui.webcontrols.createuserwizard.questionlabeltext.aspx), y [ `AnswerLabelText` ](https://msdn.microsoft.com/library/system.web.ui.webcontrols.createuserwizard.answerlabeltext.aspx) propiedades, respectivamente. Asimismo, existen propiedades para especificar el texto de los botones de Create User y continuar en la `CreateUserWizardStep` y `CompleteWizardStep`, así como si estos botones se representan como botones, LinkButton o ImageButtons.

Los colores, bordes, fuentes y otros elementos visuales son configurables a través de un host de propiedades de estilo. El propio control CreateUserWizard tiene las propiedades de estilo de control Web comunes - `BackColor`, `BorderStyle`, `CssClass`, `Font`, y así sucesivamente - y hay una serie de propiedades de estilo para definir la apariencia de secciones específicas de la Interfaz del CreateUserWizard. El [ `TextBoxStyle` propiedad](https://msdn.microsoft.com/library/system.web.ui.webcontrols.createuserwizard.textboxstyle.aspx), por ejemplo, define los estilos para los cuadros de texto en el `CreateUserWizardStep`, mientras que la [ `TitleTextStyle` propiedad](https://msdn.microsoft.com/library/system.web.ui.webcontrols.createuserwizard.titletextstyle.aspx) define el estilo del título (suscribirse a la nueva (Cuenta).

Además de las propiedades relacionadas con la apariencia, hay una serie de propiedades que afectan al comportamiento del control CreateUserWizard. El [ `DisplayCancelButton` propiedad](https://msdn.microsoft.com/library/system.web.ui.webcontrols.wizard.displaycancelbutton.aspx), si se establece en True, muestra un botón Cancelar situada junto al botón de Create User (el valor predeterminado es False). Si se muestra el botón Cancelar, asegúrese de establecer el [ `CancelDestinationPageUrl` propiedad](https://msdn.microsoft.com/library/system.web.ui.webcontrols.createuserwizard.continuedestinationpageurl.aspx), que especifica la página que se envía al usuario a tras hacer clic en Cancelar. Como se indicó en la sección anterior, el botón Continuar en la `CompleteWizardStep`de interfaz provoca una devolución de datos, pero deja el visitante en la misma página. Para enviar al visitante a otra página después de hacer clic en el botón Continuar, simplemente hay que especificar la dirección URL en el [ `ContinueDestinationPageUrl` propiedad](https://msdn.microsoft.com/library/system.web.ui.webcontrols.createuserwizard.continuedestinationpageurl.aspx).

Vamos a actualizar el `RegisterUser` CreateUserWizard control para mostrar un botón de cancelación y enviar al visitante a `Default.aspx` cuando se hace clic en los botones Cancelar o continuar. Para ello, establezca el `DisplayCancelButton` propiedad en True y tanto el `CancelDestinationPageUrl` y `ContinueDestinationPageUrl` propiedades para ~ / Default.aspx. La figura 14 muestra CreateUserWizard actualizado cuando se ven a través de un explorador.


[![El CreateUserWizardStep incluye un botón Cancelar](creating-user-accounts-vb/_static/image41.png)](creating-user-accounts-vb/_static/image40.png)

**Figura 14**: el `CreateUserWizardStep` incluye un botón Cancelar ([haga clic aquí para ver la imagen a tamaño completo](creating-user-accounts-vb/_static/image42.png))


Cuando un visitante entra en un nombre de usuario, contraseña, dirección de correo electrónico y pregunta de seguridad y respuesta y hace clic en Create User, se crea una nueva cuenta de usuario y el visitante es iniciado sesión como ese usuario recién creado. Suponiendo que la persona que visita la página es crear una nueva cuenta de por sí mismos, probablemente es el comportamiento deseado. Sin embargo, puede que desee permitir que los administradores agregar nuevas cuentas de usuario. Si lo hace, se creará la cuenta de usuario, pero el administrador se mantendría ha iniciado sesión como administrador (y no como la cuenta recién creada). Este comportamiento puede modificarse mediante el valor booleano [ `LoginCreatedUser` propiedad](https://msdn.microsoft.com/library/system.web.ui.webcontrols.createuserwizard.logincreateduser.aspx).

Cuentas de usuario en el marco de trabajo de pertenencia contienen una marca aprobada; los usuarios que no estén aprobados son no se ha podido iniciar sesión en el sitio. De forma predeterminada, una cuenta recién creada se marca como aprobado, lo que permite al usuario que inicie sesión en el sitio inmediatamente. Sin embargo, es posible, tienen cuentas de usuario marcadas como no aprobados. Quizás desee que un administrador apruebe manualmente los nuevos usuarios antes de poder iniciar sesión en; o bien, quizá desee comprobar que la dirección de correo electrónico que especificó durante el registro es válida antes de permitir que un usuario que inició sesión. Lo que puede ser el caso, puede tener la cuenta de usuario recién creado marcada como no aprobados estableciendo el control CreateUserWizard [ `DisableCreatedUser` propiedad](https://msdn.microsoft.com/library/system.web.ui.webcontrols.createuserwizard.disablecreateduser.aspx) en True (el valor predeterminado es False).

Incluyen otras propiedades relacionadas con el comportamiento de nota `AutoGeneratePassword` y `MailDefinition`. Si el [ `AutoGeneratePassword` propiedad](https://msdn.microsoft.com/library/system.web.ui.webcontrols.createuserwizard.autogeneratepassword.aspx) está establecida en True, el `CreateUserWizardStep` no muestra los cuadros de texto contraseña y Confirmar contraseña; en su lugar, contraseña del usuario recién creado se genera automáticamente con el `Membership` la clase [ `GeneratePassword` método](https://msdn.microsoft.com/library/system.web.security.membership.generatepassword.aspx). El `GeneratePassword` método crea una contraseña de una longitud especificada y con un número suficiente de caracteres no alfanuméricos para satisfacer los requisitos de seguridad de la contraseña configurada.

El [ `MailDefinition` propiedad](https://msdn.microsoft.com/library/system.web.ui.webcontrols.createuserwizard.maildefinition.aspx) es útil si desea enviar un correo electrónico a la dirección de correo electrónico especificada durante el proceso de creación de cuenta. El `MailDefinition` esta propiedad incluye una serie de subpropiedades para definir la información sobre el mensaje de correo electrónico construido. Estas subpropiedades incluyen opciones como `Subject`, `Priority`, `IsBodyHtml`, `From`, `CC`, y `BodyFileName`. El [ `BodyFileName` propiedad](https://msdn.microsoft.com/library/system.web.ui.webcontrols.maildefinition.bodyfilename.aspx) señala a un texto o archivo HTML que contiene el cuerpo del mensaje de correo electrónico. El cuerpo es compatible con dos marcadores de posición predefinidos: `<%UserName%>` y `<%Password%>`. Estos marcadores de posición, si está presente en el `BodyFileName` de archivo, se reemplazará por el nombre y la contraseña del usuario recién creado.

> [!NOTE]
> El `CreateUserWizard` del control `MailDefinition` propiedad solo especifica los detalles sobre el mensaje de correo electrónico que se envía cuando se crea una nueva cuenta. No incluye todos los datos sobre cómo se envía realmente el mensaje de correo electrónico (es decir, si se utiliza un directorio del servidor o la entrega de correo SMTP, cualquier información de autenticación y así sucesivamente). Estos detalles de bajo nivel se deben definir en el `<system.net>` sección `Web.config`. Para obtener más información sobre estas opciones de configuración y enviar correo electrónico de ASP.NET 2.0 en general, consulte el [preguntas más frecuentes en SystemNetMail.com](http://www.systemnetmail.com/) y mi artículo [enviar correo electrónico en ASP.NET 2.0](http://aspnet.4guysfromrolla.com/articles/072606-1.aspx).


### <a name="extending-the-createuserwizards-behavior-using-event-handlers"></a>Extender el comportamiento del control CreateUserWizard mediante controladores de eventos

El control CreateUserWizard eleva un número de eventos durante su flujo de trabajo. Por ejemplo, después de un visitante entra en su nombre de usuario, contraseña y otra información relevante y hace clic en el botón Crear usuario, el control CreateUserWizard genera su [ `CreatingUser` eventos](https://msdn.microsoft.com/library/system.web.ui.webcontrols.createuserwizard.creatinguser.aspx). Si hay un problema durante el proceso de creación, el [ `CreateUserError` eventos](https://msdn.microsoft.com/library/system.web.ui.webcontrols.createuserwizard.createusererror.aspx) se activa; sin embargo, si el usuario se creó correctamente, la [ `CreatedUser` evento](https://msdn.microsoft.com/library/system.web.ui.webcontrols.createuserwizard.createduser.aspx) se genera. Hay eventos de control CreateUserWizard adicionales que se produzca, pero son tres las más relevante.

En determinados escenarios que podemos desear puntee en el flujo de trabajo CreateUserWizard, lo que podemos hacer mediante la creación de un controlador de eventos para el evento adecuado. Para ilustrar esto, vamos a mejorar la `RegisterUser` control CreateUserWizard para incluir alguna validación personalizada en el nombre de usuario y contraseña. En concreto, vamos a mejorar nuestro CreateUserWizard para que los nombres de usuario no pueden contener espacios iniciales ni finales y el nombre de usuario no puede aparecer en cualquier parte en la contraseña. En resumen, queremos evitar que alguien crea un nombre de usuario como "Scott" o tener una combinación de nombre de usuario/contraseña como Scott y Scott.1234.

Para lograr esto se creará un controlador de eventos para el `CreatingUser` eventos para realizar nuestro comprobaciones de validación adicional. Si los datos proporcionados no están válidos, es necesario cancelar el proceso de creación. También es necesario agregar un control Web Label a la página para mostrar un mensaje que explica que el nombre de usuario o la contraseña no es válido. Empiece agregando un control Label debajo del control CreateUserWizard, establecer su `ID` propiedad `InvalidUserNameOrPasswordMessage` y su `ForeColor` propiedad `Red`. Borrar su `Text` propiedad y establezca su `EnableViewState` y `Visible` propiedades en False.

[!code-aspx[Main](creating-user-accounts-vb/samples/sample7.aspx)]

A continuación, cree un controlador de eventos para el control CreateUserWizard `CreatingUser` eventos. Para crear un controlador de eventos, seleccione el control en el diseñador y, a continuación, vaya a la ventana Propiedades. Desde allí, haga clic en el icono de rayo y, a continuación, haga doble clic en el evento adecuado para crear el controlador de eventos.

Agregue el código siguiente al controlador de eventos `CreatingUser`:

[!code-vb[Main](creating-user-accounts-vb/samples/sample8.vb)]

Tenga en cuenta que el nombre de usuario y una contraseña escritos en el control CreateUserWizard están disponibles a través de su [ `UserName` ](https://msdn.microsoft.com/library/system.web.ui.webcontrols.createuserwizard.username.aspx) y [ `Password` propiedades](https://msdn.microsoft.com/library/system.web.ui.webcontrols.createuserwizard.password.aspx), respectivamente. Usamos estas propiedades en el controlador de eventos anteriores para determinar si el nombre de usuario proporcionado contiene espacios iniciales o finales y si el nombre de usuario se encuentra dentro de la contraseña. Si se cumple alguna de estas condiciones, se muestra un mensaje de error en la `InvalidUserNameOrPasswordMessage` etiqueta y el controlador de eventos `e.Cancel` propiedad está establecida en `True`. Si `e.Cancel` se establece en `True`, CreateUserWizard cortocircuita su flujo de trabajo, eficazmente si cancela el proceso de creación de la cuenta de usuario.

La figura 15 muestra una captura de pantalla de `CreatingUserAccounts.aspx` cuando el usuario escribe un nombre de usuario con los espacios iniciales.


[![No se permiten nombres de usuario con iniciales o espacios finales](creating-user-accounts-vb/_static/image44.png)](creating-user-accounts-vb/_static/image43.png)

**Figura 15**: no se permiten nombres de usuario con iniciales o espacios finales ([haga clic aquí para ver la imagen a tamaño completo](creating-user-accounts-vb/_static/image45.png))


> [!NOTE]
> Vemos un ejemplo del uso del control CreateUserWizard `CreatedUser` evento en el *<a id="_msoanchor_11"> </a> [almacenar información de usuario adicional](storing-additional-user-information-vb.md)* tutorial.


## <a name="summary"></a>Resumen

El `Membership` la clase `CreateUser` método crea una nueva cuenta de usuario en el marco de trabajo de pertenencia. Para ello, delegar la llamada al proveedor de pertenencia configurado. En el caso de los `SqlMembershipProvider`, `CreateUser` método agrega un registro a la `aspnet_Users` y `aspnet_Membership` tablas de base de datos.

Aunque las cuentas de usuario se pueden crear mediante programación (como vimos en el paso 5), un enfoque más rápido y sencillo es usar el control CreateUserWizard. Este control representa una interfaz de usuario de varios pasos para recopilar información de usuario y crear un nuevo usuario en el marco de trabajo de pertenencia. Interiormente, este control utiliza la misma `Membership.CreateUser` método como examinar en el paso 5, pero el control crea la interfaz de usuario, los controles de validación y responde a los errores de creación de cuentas de usuario sin tener que escribir ni una sola línea de código.

En este momento tenemos la funcionalidad en su lugar para crear nuevas cuentas de usuario. Sin embargo, la página de inicio de sesión todavía se está validando contra esas credenciales codificadas de forma rígida, especificado en el segundo tutorial. En el <a id="_msoanchor_12"> </a> [tutorial siguiente](validating-user-credentials-against-the-membership-user-store-vb.md) actualizaremos `Login.aspx` validar el usuario proporcionado las credenciales con el marco de trabajo de pertenencia.

Feliz programación.

### <a name="further-reading"></a>Información adicional

Para obtener más información sobre los temas tratados en este tutorial, consulte los siguientes recursos:

- [`CreateUser` Documentación técnica](https://msdn.microsoft.com/library/system.web.security.membershipprovider.createuser.aspx)
- [Información general del Control CreateUserWizard](https://quickstarts.asp.net/QuickStartv20/aspnet/doc/ctrlref/login/createuserwizard.aspx)
- [Crear un proveedor de mapas de sitio basado en el sistema de archivos](http://aspnet.4guysfromrolla.com/articles/020106-1.aspx)
- [Crear una interfaz de usuario de paso a paso con el Control de asistente 2.0 de ASP.NET](http://aspnet.4guysfromrolla.com/articles/061406-1.aspx)
- [Examen de ASP.NET 2.0 de la navegación del sitio](http://aspnet.4guysfromrolla.com/articles/111605-1.aspx)
- [Páginas maestras y navegación del sitio](https://asp.net/learn/data-access/tutorial-03-vb.aspx)
- [El proveedor de mapas de sitio SQL que ha estado esperando](https://msdn.microsoft.com/msdnmag/issues/06/02/WickedCode/default.aspx)

### <a name="about-the-author"></a>Acerca del autor

Scott Mitchell, autor de varios libros sobre ASP/ASP.NET y fundador de 4GuysFromRolla.com, ha trabajado con las tecnologías Web de Microsoft desde 1998. Scott funciona como un consultor independiente, instructor y escritor. Su último libro es *[SAM enseñar a usted mismo ASP.NET 2.0 en 24 horas](https://www.amazon.com/exec/obidos/ASIN/0672327384/4guysfromrollaco)*. Puede ponerse en contacto Scott [ mitchell@4guysfromrolla.com ](mailto:mitchell@4guysfromrolla.com) o a través de su blog en [ http://ScottOnWriting.NET ](http://scottonwriting.net/).

### <a name="special-thanks-to"></a>Agradecimientos especiales a

Esta serie de tutoriales se revisó por varios revisores útiles. Revisor inicial para este tutorial era Teresa Murphy. ¿Está interesado en revisar mi próximos artículos MSDN? Si es así, me quitar una línea en [ mitchell@4GuysFromRolla.com ](mailto:mitchell@4guysfromrolla.com).

> [!div class="step-by-step"]
> [Anterior](creating-the-membership-schema-in-sql-server-vb.md)
> [Siguiente](validating-user-credentials-against-the-membership-user-store-vb.md)
