---
title: Extensibilidad de criptografía de núcleo de ASP.NET Core
author: rick-anderson
description: Obtenga información acerca de IAuthenticatedEncryptor, IAuthenticatedEncryptorDescriptor, IAuthenticatedEncryptorDescriptorDeserializer y el generador de nivel superior.
manager: wpickett
ms.author: riande
ms.date: 8/11/2017
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: security/data-protection/extensibility/core-crypto
ms.openlocfilehash: b5a0dbc9120a8032dbb8d8eee74684495a982ac1
ms.sourcegitcommit: f8852267f463b62d7f975e56bea9aa3f68fbbdeb
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2018
---
# <a name="core-cryptography-extensibility-in-aspnet-core"></a>Extensibilidad de criptografía de núcleo de ASP.NET Core

<a name="data-protection-extensibility-core-crypto"></a>

>[!WARNING]
> Los tipos que implementan cualquiera de las interfaces siguientes deben ser seguro para subprocesos para distintos llamadores.

<a name="data-protection-extensibility-core-crypto-iauthenticatedencryptor"></a>

## <a name="iauthenticatedencryptor"></a>IAuthenticatedEncryptor

El **IAuthenticatedEncryptor** interfaz es el bloque de creación básico del subsistema criptográfico. Por lo general, hay un IAuthenticatedEncryptor por clave y la instancia de IAuthenticatedEncryptor contiene todos los material de clave de cifrado y algoritmo información necesaria para realizar operaciones criptográficas.

Como sugiere su nombre, el tipo es responsable de proporcionar servicios de cifrado y descifrado autenticados. Expone las siguientes API de dos.

* Descifrar (ArraySegment<byte> texto cifrado, ArraySegment<byte> additionalAuthenticatedData): byte]

* Cifrar (ArraySegment<byte> texto simple, ArraySegment<byte> additionalAuthenticatedData): byte]

El método Encrypt devuelve un blob que incluye el texto sin formato descifra y una etiqueta de autenticación. La etiqueta de autenticación debe incluir los datos adicionales autenticados (AAD), aunque el AAD propio no necesita poder recuperarse desde la carga final. El método Decrypt valida la etiqueta de autenticación y devuelve la carga deciphered. Todos los errores (excepto ArgumentNullException y similar) deben homogeneizarse a CryptographicException.

> [!NOTE]
> La propia instancia IAuthenticatedEncryptor realmente no debe contener el material de clave. Por ejemplo, la implementación puede delegar a un HSM para todas las operaciones.

<a name="data-protection-extensibility-core-crypto-iauthenticatedencryptorfactory"></a>
<a name="data-protection-extensibility-core-crypto-iauthenticatedencryptordescriptor"></a>

## <a name="how-to-create-an-iauthenticatedencryptor"></a>Cómo crear un IAuthenticatedEncryptor

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

El **IAuthenticatedEncryptorFactory** interfaz representa un tipo que sabe cómo crear un [IAuthenticatedEncryptor](xref:security/data-protection/extensibility/core-crypto#data-protection-extensibility-core-crypto-iauthenticatedencryptor) instancia. La API es como sigue.

* CreateEncryptorInstance (clave IKey): IAuthenticatedEncryptor

Para cualquier instancia de IKey determinada, los sistemas de cifrado autenticados creados por el método CreateEncryptorInstance deben considerarse equivalentes, como en el ejemplo de código siguiente.

```csharp
// we have an IAuthenticatedEncryptorFactory instance and an IKey instance
IAuthenticatedEncryptorFactory factory = ...;
IKey key = ...;

// get an encryptor instance and perform an authenticated encryption operation
ArraySegment<byte> plaintext = new ArraySegment<byte>(Encoding.UTF8.GetBytes("plaintext"));
ArraySegment<byte> aad = new ArraySegment<byte>(Encoding.UTF8.GetBytes("AAD"));
var encryptor1 = factory.CreateEncryptorInstance(key);
byte[] ciphertext = encryptor1.Encrypt(plaintext, aad);

// get another encryptor instance and perform an authenticated decryption operation
var encryptor2 = factory.CreateEncryptorInstance(key);
byte[] roundTripped = encryptor2.Decrypt(new ArraySegment<byte>(ciphertext), aad);


// the 'roundTripped' and 'plaintext' buffers should be equivalent
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

El **IAuthenticatedEncryptorDescriptor** interfaz representa un tipo que sabe cómo crear un [IAuthenticatedEncryptor](xref:security/data-protection/extensibility/core-crypto#data-protection-extensibility-core-crypto-iauthenticatedencryptor) instancia. La API es como sigue.

* CreateEncryptorInstance(): IAuthenticatedEncryptor

* ExportToXml(): XmlSerializedDescriptorInfo

Al igual que IAuthenticatedEncryptor, una instancia de IAuthenticatedEncryptorDescriptor se supone que va a contener una clave específica. Esto significa que para cualquier instancia de IAuthenticatedEncryptorDescriptor determinada, los sistemas de cifrado autenticados creados por el método CreateEncryptorInstance deben considerarse equivalentes, como en el ejemplo de código siguiente.

```csharp
// we have an IAuthenticatedEncryptorDescriptor instance
IAuthenticatedEncryptorDescriptor descriptor = ...;

// get an encryptor instance and perform an authenticated encryption operation
ArraySegment<byte> plaintext = new ArraySegment<byte>(Encoding.UTF8.GetBytes("plaintext"));
ArraySegment<byte> aad = new ArraySegment<byte>(Encoding.UTF8.GetBytes("AAD"));
var encryptor1 = descriptor.CreateEncryptorInstance();
byte[] ciphertext = encryptor1.Encrypt(plaintext, aad);

// get another encryptor instance and perform an authenticated decryption operation
var encryptor2 = descriptor.CreateEncryptorInstance();
byte[] roundTripped = encryptor2.Decrypt(new ArraySegment<byte>(ciphertext), aad);


// the 'roundTripped' and 'plaintext' buffers should be equivalent
```

---

<a name="data-protection-extensibility-core-crypto-iauthenticatedencryptordescriptor"></a>

## <a name="iauthenticatedencryptordescriptor-aspnet-core-2x-only"></a>IAuthenticatedEncryptorDescriptor (ASP.NET Core solo 2.x)

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

El **IAuthenticatedEncryptorDescriptor** interfaz representa un tipo que sabe cómo exportar a XML. La API es como sigue.

* ExportToXml(): XmlSerializedDescriptorInfo

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

---

## <a name="xml-serialization"></a>Serialización XML

La diferencia principal entre IAuthenticatedEncryptor y IAuthenticatedEncryptorDescriptor es que el descriptor sabe cómo crear el sistema de cifrado y proporcionarle argumentos válidos. Tenga en cuenta un IAuthenticatedEncryptor cuya implementación se basa en SymmetricAlgorithm y KeyedHashAlgorithm. Trabajo del sistema de cifrado es consumen estos tipos, pero no conoce necesariamente estos tipos de proceden, por lo que realmente no se puede escribir una descripción de cómo volver a sí mismo si se reinicia la aplicación adecuada. El descriptor de actúa como un nivel más alto a partir de esto. Puesto que el descriptor sabe cómo crear la instancia de sistema de cifrado (p. ej., sabe cómo crear los algoritmos necesarios), puede serializar esa información en forma de XML para que la instancia de sistema de cifrado se puede volver a crear después de restablece una aplicación.

<a name="data-protection-extensibility-core-crypto-exporttoxml"></a>

El descriptor de se puede serializar a través de su rutina de ExportToXml. Esta rutina devuelve un XmlSerializedDescriptorInfo que contiene dos propiedades: la representación de XElement de descriptor y el tipo que representa un [IAuthenticatedEncryptorDescriptorDeserializer](xref:security/data-protection/extensibility/core-crypto#data-protection-extensibility-core-crypto-iauthenticatedencryptordescriptordeserializer) que puede ser se usa para restablecerse este descriptor dada la XElement correspondiente.

El descriptor serializado puede contener información confidencial como material de clave de cifrado. El sistema de protección de datos tiene compatibilidad integrada para cifrar la información antes de que se conservan en el almacenamiento. Para aprovechar estas características, el descriptor debería marcar el elemento que contiene información confidencial con el nombre de atributo "requiresEncryption" (xmlns "<http://schemas.asp.net/2015/03/dataProtection>"), valor "true".

>[!TIP]
> Hay una API auxiliar para establecer este atributo. Llame al método de extensión que XElement.markasrequiresencryption() ubicado en el espacio de nombres Microsoft.AspNetCore.DataProtection.AuthenticatedEncryption.ConfigurationModel.

También puede haber casos donde el descriptor serializado no contiene información confidencial. Considere la posibilidad de nuevo el caso de una clave criptográfica que se almacenan en un HSM. El material de clave no se puede escribir el descriptor al serializar a sí mismo porque el HSM no expone el material en formato de texto simple. En su lugar, puede escribir el descriptor de la versión de clave ajusta de la clave (si el HSM permite la exportación de este modo) o el identificador único del HSM para la clave.

<a name="data-protection-extensibility-core-crypto-iauthenticatedencryptordescriptordeserializer"></a>

## <a name="iauthenticatedencryptordescriptordeserializer"></a>IAuthenticatedEncryptorDescriptorDeserializer

El **IAuthenticatedEncryptorDescriptorDeserializer** interfaz representa un tipo que sabe cómo deserializar una instancia de IAuthenticatedEncryptorDescriptor desde un XElement. Expone un único método:

* ImportFromXml (elemento de XElement): IAuthenticatedEncryptorDescriptor

El método ImportFromXml toma el XElement que devolvió [IAuthenticatedEncryptorDescriptor.ExportToXml](xref:security/data-protection/extensibility/core-crypto#data-protection-extensibility-core-crypto-exporttoxml) y crea un equivalente de la IAuthenticatedEncryptorDescriptor original.

Tipos que implementan IAuthenticatedEncryptorDescriptorDeserializer deben tener uno de los dos constructores públicos siguientes:

* .ctor(IServiceProvider)

* .ctor()

> [!NOTE]
> IServiceProvider pasado al constructor puede ser null.

## <a name="the-top-level-factory"></a>El generador de nivel superior

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

El **AlgorithmConfiguration** clase representa un tipo que sabe cómo crear [IAuthenticatedEncryptorDescriptor](xref:security/data-protection/extensibility/core-crypto#data-protection-extensibility-core-crypto-iauthenticatedencryptordescriptor) instancias. Expone una sola API.

* CreateNewDescriptor(): IAuthenticatedEncryptorDescriptor

Considerar AlgorithmConfiguration como el generador de nivel superior. La configuración actúa como una plantilla. Encapsula información algorítmica (p. ej., esta configuración produce descriptores con una clave maestra de AES-128-GCM), pero aún no está asociada a una clave específica.

Cuando se llama a CreateNewDescriptor, material de clave nueva se crea únicamente para esta llamada y se genera un nuevo IAuthenticatedEncryptorDescriptor que ajusta este material de clave y la información algorítmica necesarios para consumir el material. El material de clave podría creó en software (y se mantienen en la memoria), podría ser crea y mantiene dentro de un HSM y así sucesivamente. El punto fundamental es que las dos llamadas a CreateNewDescriptor nunca deben crearse instancias de IAuthenticatedEncryptorDescriptor equivalente.

El tipo de AlgorithmConfiguration actúa como punto de entrada para las rutinas de creación de claves como [reversión de clave automática](xref:security/data-protection/implementation/key-management#key-expiration-and-rolling). Para cambiar la implementación de todas las claves futuras, establezca la propiedad AuthenticatedEncryptorConfiguration en KeyManagementOptions.

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

El **IAuthenticatedEncryptorConfiguration** interfaz representa un tipo que sabe cómo crear [IAuthenticatedEncryptorDescriptor](xref:security/data-protection/extensibility/core-crypto#data-protection-extensibility-core-crypto-iauthenticatedencryptordescriptor) instancias. Expone una sola API.

* CreateNewDescriptor(): IAuthenticatedEncryptorDescriptor

Considerar IAuthenticatedEncryptorConfiguration como el generador de nivel superior. La configuración actúa como una plantilla. Encapsula información algorítmica (p. ej., esta configuración produce descriptores con una clave maestra de AES-128-GCM), pero aún no está asociada a una clave específica.

Cuando se llama a CreateNewDescriptor, material de clave nueva se crea únicamente para esta llamada y se genera un nuevo IAuthenticatedEncryptorDescriptor que ajusta este material de clave y la información algorítmica necesarios para consumir el material. El material de clave podría creó en software (y se mantienen en la memoria), podría ser crea y mantiene dentro de un HSM y así sucesivamente. El punto fundamental es que las dos llamadas a CreateNewDescriptor nunca deben crearse instancias de IAuthenticatedEncryptorDescriptor equivalente.

El tipo de IAuthenticatedEncryptorConfiguration actúa como punto de entrada para las rutinas de creación de claves como [reversión de clave automática](xref:security/data-protection/implementation/key-management#key-expiration-and-rolling). Para cambiar la implementación de todas las claves futuras, registrar un singleton IAuthenticatedEncryptorConfiguration en el contenedor de servicios.

---
