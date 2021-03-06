---
title: Ataques de evitar Cross-Site falsificación de solicitud (XSRF/CSRF) en ASP.NET Core
author: steve-smith
description: Descubra cómo impedir ataques contra las aplicaciones web en un sitio Web malintencionado puede influir en la interacción entre un explorador del cliente y la aplicación.
manager: wpickett
ms.author: riande
ms.custom: mvc
ms.date: 03/19/2018
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: security/anti-request-forgery
ms.openlocfilehash: ad50f8b261447d40ccc24c0ee006239aa976bf20
ms.sourcegitcommit: 7d02ca5f5ddc2ca3eb0258fdd6996fbf538c129a
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2018
---
# <a name="prevent-cross-site-request-forgery-xsrfcsrf-attacks-in-aspnet-core"></a>Ataques de evitar Cross-Site falsificación de solicitud (XSRF/CSRF) en ASP.NET Core

Por [Steve Smith](https://ardalis.com/), [Fiyaz Hasan](https://twitter.com/FiyazBinHasan), y [Rick Anderson](https://twitter.com/RickAndMSFT)

Falsificación de solicitudes entre sitios (también conocido como XSRF o CSRF, pronunciado *vea navegación*) es un ataque contra mediante el cual una aplicación web malintencionado puede influir en la interacción entre un explorador del cliente y una aplicación web que confía en que las aplicaciones hospedadas en web Explorador. Estos ataques son posibles porque los exploradores web envían algunos tipos de tokens de autenticación automáticamente con cada solicitud a un sitio Web. Esta forma de vulnerabilidad de seguridad es también conocido como un *ataque de un solo clic* o *sesión conducir* porque el ataque aprovecha las ventajas de sesión del autentica previamente en el usuario.

Un ejemplo de un ataque CSRF:

1. Un usuario inicia sesión en `www.good-banking-site.com` utilizando la autenticación de formularios. El servidor autentica al usuario y emite una respuesta que incluye una cookie de autenticación. El sitio es vulnerable a ataques porque confía en cualquier solicitud recibida con una cookie de autenticación válida.
1. El usuario visita un sitio malintencionado, `www.bad-crook-site.com`.

   El sitio malintencionado, `www.bad-crook-site.com`, contiene un formulario HTML similar al siguiente:

   ```html
   <h1>Congratulations! You're a Winner!</h1>
   <form action="http://good-banking-site.com/api/account" method="post">
       <input type="hidden" name="Transaction" value="withdraw">
       <input type="hidden" name="Amount" value="1000000">
       <input type="submit" value="Click to collect your prize!">
   </form>
   ```

   Tenga en cuenta que el formulario `action` entradas para el sitio sea vulnerable, no para el sitio malintencionado. Esta es la parte "cross-site" de CSRF.

1. El usuario selecciona el botón Enviar. El explorador realiza la solicitud y automáticamente incluye la cookie de autenticación para el dominio solicitado, `www.good-banking-site.com`.
1. La solicitud se ejecuta en el `www.good-banking-site.com` servidor con contexto de autenticación del usuario y pueden realizar cualquier acción que puede realizar un usuario autenticado.

Cuando el usuario selecciona el botón para enviar el formulario, el sitio malintencionado podría:

* Ejecutar un script que envía automáticamente el formulario.
* Envía un envío de formulario como una solicitud AJAX. 
* Usar un formulario oculto con CSS. 

Uso de HTTPS no impide que un ataque CSRF. El sitio malintencionado puede enviar un `https://www.good-banking-site.com/` solicitar de manera tan sencilla como puede enviar una solicitud insegura.

Algunos ataques de destino que respondan a las solicitudes GET, en cuyo caso se puede utilizar una etiqueta de imagen para realizar la acción. Esta forma de ataque es habitual en los sitios de foro que permiten imágenes pero bloquean JavaScript. Las aplicaciones que cambian el estado de las solicitudes GET, donde se modifican las variables o los recursos, son vulnerables a ataques malintencionados. **Las solicitudes GET que cambiarán el estado no son seguros. Una práctica recomendada consiste en no cambiar nunca el estado en una solicitud GET.**

Los ataques CSRF son posibles con aplicaciones web que usan cookies para la autenticación porque:

* Los exploradores almacenan las cookies emitidas por una aplicación web.
* Almacenado cookies incluyen cookies de sesión para los usuarios autenticados.
* Exploradores envían que todas las cookies asociadas con un dominio a la aplicación web de todas las solicitudes, independientemente de cómo se generó la solicitud de aplicación dentro del explorador.

Sin embargo, no están limitados CSRF ataques para aprovecharse de las cookies. Por ejemplo, la autenticación básica e implícita también son vulnerables. Una vez que un usuario inicia sesión con autenticación básica o implícita, el explorador envía automáticamente las credenciales hasta que la sesión&dagger; finaliza.

&dagger;En este contexto, *sesión* hace referencia a la sesión de cliente durante el cual el usuario está autenticado. Es no relacionados con las sesiones de servidor o [Middleware de sesión de ASP.NET Core](xref:fundamentals/app-state).

Los usuarios pueden protegerse contra vulnerabilidades CSRF, tome precauciones:

* Inicie sesión en las aplicaciones web cuando termine de utilizarlos.
* Borrar las cookies del explorador periódicamente.

Sin embargo, las vulnerabilidades CSRF son fundamentalmente un problema con la aplicación web, no el usuario final.

## <a name="authentication-fundamentals"></a>Conceptos básicos de autenticación

Autenticación basada en cookies es una forma popular de autenticación. Sistemas de autenticación basada en token son cada vez más popularidad, especialmente para las aplicaciones de página única (SPAs).

### <a name="cookie-based-authentication"></a>Autenticación basada en cookies

Cuando un usuario se autentica con su nombre de usuario y contraseña, le emite un token, que contiene un vale de autenticación que se puede usar para la autenticación y autorización. El token se almacena como hace que una cookie que acompaña a cada solicitud del cliente. Generar y validar esta cookie se realizan mediante el Middleware de autenticación de la Cookie. El [middleware](xref:fundamentals/middleware/index) serializa una entidad de seguridad de usuario en una cookie cifrada. En solicitudes posteriores, el middleware valida la cookie, vuelve a crear la entidad de seguridad y asigna la entidad de seguridad para la [usuario](/dotnet/api/microsoft.aspnetcore.http.httpcontext.user) propiedad de [HttpContext](/dotnet/api/microsoft.aspnetcore.http.httpcontext).

### <a name="token-based-authentication"></a>Autenticación basada en token

Cuando un usuario se autentica, le emite un token (no un token antiforgery). El token contiene información de usuario en forma de [notificaciones](/dotnet/framework/security/claims-based-identity-model) o un símbolo (token) de referencia que señala la aplicación al estado de usuario que se mantienen en la aplicación. Cuando un usuario intenta acceder a un recurso que requiere autenticación, el token se envía a la aplicación con un encabezado de autorización adicionales en forma de token de portador. Esto hace que la aplicación sin estado. En cada solicitud posterior, el token se pasa en la solicitud para la validación del lado servidor. Este token no es *cifrados*; tiene *codificado*. En el servidor, se descodifica el token para acceder a su información. Para enviar el token en las solicitudes subsiguientes, almacene el token en el almacenamiento local del explorador. No se preocupe acerca de vulnerabilidad CSRF si el token se almacena en almacenamiento local del explorador. CSRF es una preocupación cuando el token se almacena en una cookie.

### <a name="multiple-apps-hosted-at-one-domain"></a>Varias aplicaciones hospedadas en un dominio

Entornos de hospedaje compartidos son vulnerables a secuestro de sesión, inicio de sesión CSRF y otros ataques.

Aunque `example1.contoso.net` y `example2.contoso.net` son diferentes de los hosts, hay una relación de confianza implícita entre los hosts bajo la `*.contoso.net` dominio. Esta relación de confianza implícita permite a los hosts no sea de confianza influir en sus respectivas cookies (las directivas de mismo origen que rigen las solicitudes AJAX necesariamente no se aplican a las cookies HTTP).

Se pueden evitar ataques que aprovechan las cookies de confianza entre las aplicaciones hospedadas en el mismo dominio, como no compartir dominios. Cuando cada aplicación se hospeda en su propio dominio, no hay ninguna relación de confianza implícita cookie aprovechar.

## <a name="aspnet-core-antiforgery-configuration"></a>Configuración de ASP.NET Core antiforgery

> [!WARNING]
> ASP.NET Core implementa con antiforgery [protección de datos de ASP.NET Core](xref:security/data-protection/introduction). La pila de protección de datos debe configurarse para que funcione en una granja de servidores. Vea [configurando la protección de datos](xref:security/data-protection/configuration/overview) para obtener más información.

En el núcleo de ASP.NET 2.0 o posterior, el [FormTagHelper](xref:mvc/views/working-with-forms#the-form-tag-helper) inyecta antiforgery tokens en elementos de formulario HTML. El siguiente marcado en un archivo Razor automáticamente genera tokens antiforgery:

```cshtml
<form method="post">
    ...
</form>
```

De forma similar, [IHtmlHelper.BeginForm](/dotnet/api/microsoft.aspnetcore.mvc.rendering.ihtmlhelper.beginform) genera antiforgery símbolos (tokens) de forma predeterminada si el método del formulario no es GET.

La generación automática de símbolos (tokens) antiforgery para los elementos de formulario HTML se produce cuando el `<form>` etiqueta contiene el `method="post"` atributo y cualquiera de las siguientes son verdaderas:

  * El atributo de acción está vacío (`action=""`).
  * No se especifica el atributo de acción (`<form method="post">`).

Se puede deshabilitar la generación automática de símbolos (tokens) antiforgery para los elementos de formulario HTML:

* Deshabilitar explícitamente antiforgery tokens con el `asp-antiforgery` atributo:

  ```cshtml
  <form method="post" asp-antiforgery="false">
      ...
  </form>
  ```

* El elemento de formulario es darse de alta en horizontal de aplicaciones auxiliares de etiquetas mediante el uso de la aplicación auxiliar de etiqueta [! símbolos de desactivación](xref:mvc/views/tag-helpers/intro#opt-out):

  ```cshtml
  <!form method="post">
      ...
  </!form>
  ```

* Quitar el `FormTagHelper` de la vista. La `FormTagHelper` puede quitarse desde una vista agregando la siguiente directiva a la vista Razor:

  ```cshtml
  @removeTagHelper Microsoft.AspNetCore.Mvc.TagHelpers.FormTagHelper, Microsoft.AspNetCore.Mvc.TagHelpers
  ```

> [!NOTE]
> [Las páginas de Razor](xref:mvc/razor-pages/index) están protegidos automáticamente frente a XSRF/CSRF. Para obtener más información, consulte [XSRF/CSRF y páginas de Razor](xref:mvc/razor-pages/index#xsrf).

El enfoque más común para defenderse contra ataques CSRF consiste en usar la *patrón del Token Sincronizador* (STP). STP se utiliza cuando el usuario solicita una página con datos del formulario:

1. El servidor envía un token asociado con la identidad del usuario actual al cliente.
1. El cliente devuelve el token en el servidor para la comprobación.
1. Si el servidor recibe un token que no coincide con la identidad del usuario autenticado, se rechaza la solicitud.

El token es único y no previstos. El token puede usarse también para asegurarse de la secuencia correcta de una serie de solicitudes (por ejemplo, garantizar la secuencia de solicitud de: página 1 &ndash; página 2 &ndash; página 3). Todos los formularios en plantillas de MVC de ASP.NET Core y páginas de Razor generan tokens antiforgery. El siguiente par de ejemplos de vista genera tokens antiforgery:

```cshtml
<form asp-controller="Manage" asp-action="ChangePassword" method="post">
    ...
</form>

@using (Html.BeginForm("ChangePassword", "Manage"))
{
    ...
}
```

Agregar explícitamente un token antiforgery a una `<form>` elemento sin usar aplicaciones auxiliares de etiquetas con la aplicación auxiliar HTML [ @Html.AntiForgeryToken ](/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.htmlhelper.antiforgerytoken):

```cshtml
<form action="/" method="post">
    @Html.AntiForgeryToken()
</form>
```

En cada uno de los casos anteriores, ASP.NET Core agrega un campo de formulario oculto similar al siguiente:

```cshtml
<input name="__RequestVerificationToken" type="hidden" value="CfDJ8NrAkS ... s2-m9Yw">
```

ASP.NET Core incluye tres [filtros](xref:mvc/controllers/filters) para trabajar con tokens antiforgery:

* [ValidateAntiForgeryToken](/dotnet/api/microsoft.aspnetcore.mvc.validateantiforgerytokenattribute)
* [AutoValidateAntiforgeryToken](/dotnet/api/microsoft.aspnetcore.mvc.autovalidateantiforgerytokenattribute)
* [IgnoreAntiforgeryToken](/dotnet/api/microsoft.aspnetcore.mvc.ignoreantiforgerytokenattribute)

## <a name="antiforgery-options"></a>Opciones de antiforgery

Personalizar [opciones antiforgery](/dotnet/api/Microsoft.AspNetCore.Antiforgery.AntiforgeryOptions) en `Startup.ConfigureServices`:

```csharp
services.AddAntiforgery(options => 
{
    options.CookieDomain = "contoso.com";
    options.CookieName = "X-CSRF-TOKEN-COOKIENAME";
    options.CookiePath = "Path";
    options.FormFieldName = "AntiforgeryFieldname";
    options.HeaderName = "X-CSRF-TOKEN-HEADERNAME";
    options.RequireSsl = false;
    options.SuppressXFrameOptionsHeader = false;
});
```

| Opción | Descripción |
| ------ | ----------- |
| [Cookie](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.cookie) | Determina la configuración utilizada para crear la cookie antiforgery. |
| [CookieDomain](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.cookiedomain) | El dominio de la cookie. Tiene como valor predeterminado `null`. Esta propiedad está obsoleta y se quitará en una versión futura. La alternativa recomendada es Cookie.Domain. |
| [CookieName](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.cookiename) | El nombre de la cookie. Si no está establecida, el sistema genera un nombre único que comienza con la [DefaultCookiePrefix](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.defaultcookieprefix) (". AspNetCore.Antiforgery."). Esta propiedad está obsoleta y se quitará en una versión futura. La alternativa recomendada es Cookie.Name. |
| [CookiePath](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.cookiepath) | La ruta de acceso establecido en la cookie. Esta propiedad está obsoleta y se quitará en una versión futura. La alternativa recomendada es Cookie.Path. |
| [FormFieldName](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.formfieldname) | El nombre del campo oculto del formulario utilizado por el sistema antiforgery para representar antiforgery tokens en las vistas. |
| [HeaderName](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.headername) | El nombre del encabezado utilizado por el sistema antiforgery. Si `null`, el sistema considera que solo los datos de formulario. |
| [RequireSsl](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.requiressl) | Especifica si se requiere SSL por el sistema antiforgery. Si `true`, se producirá un error en las solicitudes sin SSL. Tiene como valor predeterminado `false`. Esta propiedad está obsoleta y se quitará en una versión futura. La alternativa recomendada es establecer Cookie.SecurePolicy. |
| [SuppressXFrameOptionsHeader](/dotnet/api/microsoft.aspnetcore.antiforgery.antiforgeryoptions.suppressxframeoptionsheader) | Especifica si se debe suprimir la generación de la `X-Frame-Options` encabezado. De forma predeterminada, el encabezado se genera con un valor de "SAMEORIGIN". Tiene como valor predeterminado `false`. |

Para obtener más información, consulte [CookieAuthenticationOptions](/dotnet/api/Microsoft.AspNetCore.Builder.CookieAuthenticationOptions).

## <a name="configure-antiforgery-features-with-iantiforgery"></a>Configurar características antiforgery con IAntiforgery

[IAntiforgery](/dotnet/api/microsoft.aspnetcore.antiforgery.iantiforgery) proporciona la API para configurar características antiforgery. `IAntiforgery` se puede solicitar en el `Configure` método de la `Startup` clase. En el ejemplo siguiente se usa el middleware de página principal de la aplicación para generar un token de antiforgery y enviarlo en la respuesta como una cookie (mediante la convención de nomenclatura de manera predeterminada Angular que se describe más adelante en este tema):

```csharp
public void Configure(IApplicationBuilder app, IAntiforgery antiforgery)
{
    app.Use(next => context =>
    {
        string path = context.Request.Path.Value;

        if (
            string.Equals(path, "/", StringComparison.OrdinalIgnoreCase) ||
            string.Equals(path, "/index.html", StringComparison.OrdinalIgnoreCase))
        {
            // The request token can be sent as a JavaScript-readable cookie, 
            // and Angular uses it by default.
            var tokens = antiforgery.GetAndStoreTokens(context);
            context.Response.Cookies.Append("XSRF-TOKEN", tokens.RequestToken, 
                new CookieOptions() { HttpOnly = false });
        }

        return next(context);
    });
}
```

### <a name="require-antiforgery-validation"></a>Requerir la validación antiforgery

[ValidateAntiForgeryToken](/dotnet/api/microsoft.aspnetcore.mvc.validateantiforgerytokenattribute) es un filtro de acción que se puede aplicar a una acción individual, un controlador, o de forma global. Las solicitudes realizadas a las acciones que se ha aplicado este filtro se bloquean a menos que la solicitud incluye un token de antiforgery válido.

```csharp
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> RemoveLogin(RemoveLoginViewModel account)
{
    ManageMessageId? message = ManageMessageId.Error;
    var user = await GetCurrentUserAsync();

    if (user != null)
    {
        var result = 
            await _userManager.RemoveLoginAsync(
                user, account.LoginProvider, account.ProviderKey);

        if (result.Succeeded)
        {
            await _signInManager.SignInAsync(user, isPersistent: false);
            message = ManageMessageId.RemoveLoginSuccess;
        }
    }

    return RedirectToAction(nameof(ManageLogins), new { Message = message });
}
```

El `ValidateAntiForgeryToken` atributo requiere un token para las solicitudes a los métodos de acción que decora, incluidas las solicitudes HTTP GET. Si el `ValidateAntiForgeryToken` atributo se aplica en todos los controladores de la aplicación, puede reemplazarse por el `IgnoreAntiforgeryToken` atributo.

> [!NOTE]
> ASP.NET Core no admite la adición de tokens antiforgery a solicitudes GET automáticamente.

### <a name="automatically-validate-antiforgery-tokens-for-unsafe-http-methods-only"></a>Validar automáticamente los tokens antiforgery para solo los métodos HTTP no seguros

Las aplicaciones ASP.NET Core no generan antiforgery tokens para los métodos HTTP seguros (GET, HEAD, opciones y seguimiento). En lugar de aplicar ampliamente el `ValidateAntiForgeryToken` atributo y, a continuación, reemplazar con `IgnoreAntiforgeryToken` atributos, los [AutoValidateAntiforgeryToken](/dotnet/api/microsoft.aspnetcore.mvc.autovalidateantiforgerytokenattribute) atributo se puede usar. Este atributo funciona de forma idéntica a la `ValidateAntiForgeryToken` de atributo, salvo que no requiere tokens para las solicitudes realizadas con los siguientes métodos HTTP:

* GET
* HEAD
* OPCIONES
* TRACE

Se recomienda usar `AutoValidateAntiforgeryToken` ampliamente para escenarios de API no. Esto garantiza que las acciones de entrada están protegidas de forma predeterminada. La alternativa es omitir antiforgery símbolos (tokens) de forma predeterminada, a menos que `ValidateAntiForgeryToken` se aplica a los métodos de acción individuales. No esté más probable es que en este escenario para un método de acción POST que desee dejar protegida por error, salir de la aplicación sea vulnerable a ataques CSRF. Todas las publicaciones deben enviar el token antiforgery.

Las API no tienen un mecanismo automático para el envío de la parte no cookie del token. La implementación probablemente depende de la implementación del código de cliente. A continuación se muestran algunos ejemplos:

Ejemplo de nivel de clase:

```csharp
[Authorize]
[AutoValidateAntiforgeryToken]
public class ManageController : Controller
{
```

Ejemplo global:

```csharp
services.AddMvc(options => 
    options.Filters.Add(new AutoValidateAntiforgeryTokenAttribute()));
```

### <a name="override-global-or-controller-antiforgery-attributes"></a>Reemplazo global o atributos antiforgery de controlador

El [IgnoreAntiforgeryToken](/dotnet/api/microsoft.aspnetcore.mvc.ignoreantiforgerytokenattribute) filtro se utiliza para eliminar la necesidad de un token antiforgery para una acción determinada (o controlador). Cuando se aplica, invalida este filtro `ValidateAntiForgeryToken` y `AutoValidateAntiforgeryToken` filtros especificados en un nivel más alto (global o en un controlador).

```csharp
[Authorize]
[AutoValidateAntiforgeryToken]
public class ManageController : Controller
{
    [HttpPost]
    [IgnoreAntiforgeryToken]
    public async Task<IActionResult> DoSomethingSafe(SomeViewModel model)
    {
        // no antiforgery token required
    }
}
```

## <a name="refresh-tokens-after-authentication"></a>Tokens de actualización después de la autenticación

Símbolos (tokens) se debe actualizar después de que el usuario se autenticó mediante redirige al usuario a una vista o página de las páginas de Razor.

## <a name="javascript-ajax-and-spas"></a>JavaScript, AJAX y SPAs

En las aplicaciones tradicionales basadas en HTML, antiforgery símbolos (tokens) se pasa al servidor mediante campos ocultos de formulario. En las aplicaciones modernas basadas en JavaScript y SPAs, muchas de las solicitudes se realizan mediante programación. Estas solicitudes de AJAX pueden usar otras técnicas (por ejemplo, los encabezados de solicitud o cookies) para enviar el token.

Si se usan cookies para almacenar los tokens de autenticación y para autenticar las solicitudes de API en el servidor, CSRF es un posible problema. Si se usa almacenamiento local para almacenar el token, podría ser mitigada vulnerabilidad CSRF porque valores desde el almacenamiento local no se envían automáticamente al servidor con cada solicitud. Por lo tanto, puede utilizar almacenamiento local para almacenar el token antiforgery en el cliente y enviar el token como un encabezado de solicitud es un enfoque recomendado.

### <a name="javascript"></a>JavaScript

Uso de JavaScript con vistas, el token de puede crearse con un servicio desde dentro de la vista. Insertar el [Microsoft.AspNetCore.Antiforgery.IAntiforgery](/dotnet/api/microsoft.aspnetcore.antiforgery.iantiforgery) servicio en la vista y llame a [GetAndStoreTokens](/dotnet/api/microsoft.aspnetcore.antiforgery.iantiforgery.getandstoretokens):

[!code-csharp[](anti-request-forgery/sample/MvcSample/Views/Home/Ajax.cshtml?highlight=4-10,12-13,35-36)]

Este enfoque elimina la necesidad de tratar directamente con la configuración de cookies del servidor o para leerlos desde el cliente.

El ejemplo anterior utiliza JavaScript para leer el valor del campo oculto para el encabezado de POST de AJAX.

JavaScript puede tokens de acceso en las cookies y usar el contenido de la cookie para crear un encabezado con el valor del token.

```csharp
context.Response.Cookies.Append("CSRF-TOKEN", tokens.RequestToken, 
    new Microsoft.AspNetCore.Http.CookieOptions { HttpOnly = false });
```

Suponiendo que la secuencia de comandos las solicitudes para enviar el token en un encabezado denominado `X-CSRF-TOKEN`, configurar el servicio antiforgery para buscar la `X-CSRF-TOKEN` encabezado:

```csharp
services.AddAntiforgery(options => options.HeaderName = "X-CSRF-TOKEN");
```

En el ejemplo siguiente se utiliza JavaScript para realizar una solicitud de AJAX con el encabezado adecuado:

```javascript
function getCookie(cname) {
    var name = cname + "=";
    var decodedCookie = decodeURIComponent(document.cookie);
    var ca = decodedCookie.split(';');
    for(var i = 0; i <ca.length; i++) {
        var c = ca[i];
        while (c.charAt(0) == ' ') {
            c = c.substring(1);
        }
        if (c.indexOf(name) == 0) {
            return c.substring(name.length, c.length);
        }
    }
    return "";
}

var csrfToken = getCookie("CSRF-TOKEN");

var xhttp = new XMLHttpRequest();
xhttp.onreadystatechange = function() {
    if (xhttp.readyState == XMLHttpRequest.DONE) {
        if (xhttp.status == 200) {
            alert(xhttp.responseText);
        } else {
            alert('There was an error processing the AJAX request.');
        }
    }
};
xhttp.open('POST', '/api/password/changepassword', true);
xhttp.setRequestHeader("Content-type", "application/json");
xhttp.setRequestHeader("X-CSRF-TOKEN", csrfToken);
xhttp.send(JSON.stringify({ "newPassword": "ReallySecurePassword999$$$" }));
```

### <a name="angularjs"></a>AngularJS

AngularJS usa una convención para dirección CSRF. Si el servidor envía una cookie con el nombre `XSRF-TOKEN`, AngularJS `$http` servicio agrega el valor de la cookie a un encabezado cuando envía una solicitud al servidor. Este proceso es automático. El encabezado no tiene que establecerse de forma explícita. El nombre de encabezado es `X-XSRF-TOKEN`. El servidor debe detectar este encabezado y validar su contenido.

Para la API de ASP.NET básica trabajar con esta convención:

* Configurar la aplicación para proporcionar un token en una cookie denominada `XSRF-TOKEN`.
* Configurar el servicio para que busque un encabezado denominado antiforgery `X-XSRF-TOKEN`.

```csharp
services.AddAntiforgery(options => options.HeaderName = "X-XSRF-TOKEN");
```

[Vea o descargue el código de ejemplo](https://github.com/aspnet/Docs/tree/master/aspnetcore/security/anti-request-forgery/sample/AngularSample) ([cómo descargarlo](xref:tutorials/index#how-to-download-a-sample))

## <a name="extend-antiforgery"></a>Extender antiforgery

El [IAntiForgeryAdditionalDataProvider](/dotnet/api/microsoft.aspnetcore.antiforgery.iantiforgeryadditionaldataprovider) tipo permite a los desarrolladores extender el comportamiento del sistema anti-CSRF por datos adicionales de ida y vuelta de cada token. El [GetAdditionalData](/dotnet/api/microsoft.aspnetcore.antiforgery.iantiforgeryadditionaldataprovider.getadditionaldata) método se llama cada vez que se genera un token de campo y el valor devuelto está incrustado en el token generado. Un implementador podría devolver una marca de tiempo, un nonce o cualquier otro valor y, a continuación, llame a [ValidateAdditionalData](/dotnet/api/microsoft.aspnetcore.antiforgery.iantiforgeryadditionaldataprovider.validateadditionaldata) para validar estos datos cuando se valida el token. Nombre de usuario del cliente ya está incrustada en los tokens generados, por lo que no es necesario incluir esta información. Si un token incluye datos suplementarios pero no `IAntiForgeryAdditionalDataProvider` está configurado, no validados los datos suplementarios.

## <a name="additional-resources"></a>Recursos adicionales

* [CSRF](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)) en [Abrir proyecto de seguridad de aplicación Web](https://www.owasp.org/index.php/Main_Page) (OWASP).
