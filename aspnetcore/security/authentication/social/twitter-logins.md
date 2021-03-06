---
title: Programa de instalación de inicio de sesión externo de Twitter con ASP.NET Core
author: rick-anderson
description: Este tutorial muestra la integración de autenticación de usuario de cuenta de Twitter en una aplicación existente de ASP.NET Core.
manager: wpickett
ms.author: riande
ms.date: 11/01/2016
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: security/authentication/twitter-logins
ms.openlocfilehash: 3f59f7d1bf0280cef8f7757e8cd57d4872769b3d
ms.sourcegitcommit: 43bd79667bbdc8a07bd39fb4cd6f7ad3e70212fb
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2018
ms.locfileid: "34689001"
---
# <a name="twitter-external-login-setup-with-aspnet-core"></a>Programa de instalación de inicio de sesión externo de Twitter con ASP.NET Core

Por [Valeriy Novytskyy](https://github.com/01binary) y [Rick Anderson](https://twitter.com/RickAndMSFT)

Este tutorial muestra cómo permitir a los usuarios a [iniciar sesión con su cuenta de Twitter](https://dev.twitter.com/web/sign-in/desktop-browser) usando un proyecto de ASP.NET Core 2.0 de ejemplo creado en el [página anterior](xref:security/authentication/social/index).

## <a name="create-the-app-in-twitter"></a>Crear la aplicación en Twitter

* Vaya a [ https://apps.twitter.com/ ](https://apps.twitter.com/) e inicie sesión. Si ya no tiene una cuenta de Twitter, use la **[Regístrese ahora](https://twitter.com/signup)** vínculo para crear uno. Después de iniciar sesión, el **Application Management** se muestra la página:

![Administración de aplicaciones de Twitter abierta en Microsoft Edge](index/_static/TwitterAppManage.png)

* Pulse **crear una aplicación nueva** y rellene la aplicación **nombre**, **descripción** públicas y **sitio Web** (Esto puede ser temporal hasta que el URI Registre el nombre de dominio):

![Crear una página de aplicación](index/_static/TwitterCreate.png)

* Escriba el URI de desarrollo con */signin-twitter* anexan a la **válido URI de redireccionamiento de OAuth** campo (por ejemplo: `https://localhost:44320/signin-twitter`). El esquema de autenticación de Twitter configurado más adelante en este tutorial controlará automáticamente las solicitudes en */signin-twitter* ruta para implementar el flujo de OAuth.

* Rellene el resto del formulario y pulse **crear su aplicación de Twitter**. Se muestran los detalles de la nueva aplicación:

![Pestaña Detalles de la página de aplicación](index/_static/TwitterAppDetails.png)

* Al implementar el sitio que necesite volver a visitar el **Application Management** página y registrar un nuevo URI público.

## <a name="storing-twitter-consumerkey-and-consumersecret"></a>Almacenar Twitter ConsumerKey y ConsumerSecret

Vincular valores confidenciales como Twitter `Consumer Key` y `Consumer Secret` a su configuración de aplicación con el [secreto Manager](xref:security/app-secrets). Para los fines de este tutorial, nombre de los tokens `Authentication:Twitter:ConsumerKey` y `Authentication:Twitter:ConsumerSecret`.

Estos tokens se pueden encontrar en el **claves y Tokens de acceso** ficha después de crear la nueva aplicación de Twitter:

![Pestaña de claves y los Tokens de acceso](index/_static/TwitterKeys.png)

## <a name="configure-twitter-authentication"></a>Configurar la autenticación de Twitter

La plantilla de proyecto que se usan en este tutorial asegura de que [Microsoft.AspNetCore.Authentication.Twitter](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Twitter) paquete ya está instalado.

* Para instalar este paquete con 2017 de Visual Studio, haga doble clic en el proyecto y seleccione **administrar paquetes de NuGet**.
* Para instalar con CLI de .NET Core, ejecute lo siguiente en el directorio del proyecto:

   `dotnet add package Microsoft.AspNetCore.Authentication.Twitter`

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x/)

Agregue el servicio de Twitter en el `ConfigureServices` método *Startup.cs* archivo:

```csharp
services.AddIdentity<ApplicationUser, IdentityRole>()
        .AddEntityFrameworkStores<ApplicationDbContext>()
        .AddDefaultTokenProviders();

services.AddAuthentication().AddTwitter(twitterOptions =>
{
    twitterOptions.ConsumerKey = Configuration["Authentication:Twitter:ConsumerKey"];
    twitterOptions.ConsumerSecret = Configuration["Authentication:Twitter:ConsumerSecret"];
});
```

[!INCLUDE [default settings configuration](includes/default-settings.md)]

[!INCLUDE[](~/includes/chain-auth-providers.md)]

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x/)

Agregar el middleware de Twitter en el `Configure` método *Startup.cs* archivo:

```csharp
app.UseTwitterAuthentication(new TwitterOptions()
{
    ConsumerKey = Configuration["Authentication:Twitter:ConsumerKey"],
    ConsumerSecret = Configuration["Authentication:Twitter:ConsumerSecret"]
});
```

---

Consulte la [TwitterOptions](/dotnet/api/microsoft.aspnetcore.builder.twitteroptions) referencia de API para obtener más información sobre las opciones de configuración compatible con autenticación de Twitter. Esto se puede usar para solicitar información diferente sobre el usuario.

## <a name="sign-in-with-twitter"></a>Inicie sesión con Twitter

Ejecute la aplicación y haga clic en **sesión**. Aparece una opción para iniciar sesión con Twitter:

![Aplicación Web: usuario no autenticado](index/_static/DoneTwitter.png)

Al hacer clic en **Twitter** redirige a Twitter para la autenticación:

![Página de autenticación de Twitter](index/_static/TwitterLogin.png)

Después de escribir sus credenciales de Twitter, se le redirigirá al sitio web donde puede establecer el correo electrónico.

Ahora que haya iniciado sesión con sus credenciales de Twitter:

![Aplicación Web: usuario autenticado](index/_static/Done.png)

## <a name="troubleshooting"></a>Solución de problemas

* **ASP.NET Core solo 2.x:** identidad si no está configurado mediante una llamada a `services.AddIdentity` en `ConfigureServices`, intenta autenticar se producirá en *ArgumentException: se debe proporcionar la opción 'SignInScheme'*. La plantilla de proyecto que se usan en este tutorial se asegura de que esto se realiza.
* Si la base de datos de sitio no se ha creado mediante la aplicación de la migración inicial, obtendrá *error en una operación de base de datos al procesar la solicitud* error. Pulse **migraciones aplicar** para crear la base de datos y actualizar para continuar después del error.

## <a name="next-steps"></a>Pasos siguientes

* En este artículo se ha explicado cómo puede autenticar con Twitter. Puede seguir un enfoque similar para autenticar con otros proveedores que se enumeran en la [página anterior](xref:security/authentication/social/index).

* Una vez que se publica un sitio web a la aplicación web de Azure, debe restablecer la `ConsumerSecret` en el portal para desarrolladores de Twitter.

* Establecer el `Authentication:Twitter:ConsumerKey` y `Authentication:Twitter:ConsumerSecret` como configuración de la aplicación en el portal de Azure. El sistema de configuración está configurado para leer las claves de las variables de entorno.
