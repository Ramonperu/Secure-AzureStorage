# Protección de Azure Storage

Bienvenidos a la guía de características de seguridad de Azure Storage, este resumen esta hecho gracias a la documentación oficial de Microsoft  https://learn.microsoft.com/en-us/training/modules/secure-azure-storage-account/ para estudiar y repasar para la renovación de la certificación **AZ-104** a **Marzo de 2024**.

## Revisión de las opciones para almacenar datos en Azure

| Característica                                   | Descripción                                                  | Cuándo se usa                                                |
| :----------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **Azure Files**                                  | Recursos compartidos de archivos en la nube totalmente administrados, a los que se puede acceder mediante el [protocolo Bloque de mensajes del servidor (SMB)](https://learn.microsoft.com/es-es/windows/win32/fileio/microsoft-smb-protocol-and-cifs-protocol-overview) estándar, el [protocolo Network File System (NFS)](https://en.wikipedia.org/wiki/Network_File_System) y la [API REST de Azure Files](https://learn.microsoft.com/es-es/rest/api/storageservices/file-service-rest-api).  Los recursos compartidos de archivos de Azure se pueden montar desde implementaciones de Windows, Linux y macOS en la nube o locales. | Cuando desee migrar mediante lift-and-shift una aplicación a la nube que ya usa las API del sistema de archivos nativo para compartir datos entre ella y otras aplicaciones que se ejecutan en Azure.  Cuando quiera reemplazar o complementar los servidores de archivos locales o los dispositivos NAS.  Cuando desee almacenar herramientas de desarrollo y depuración a las que es necesario acceder desde muchas máquinas virtuales. |
| **Azure Blobs**                                  | Permite que los datos no estructurados se almacenen y accedan a una escala masiva en blobs en bloques.  También admite [Azure Data Lake Storage Gen2](https://learn.microsoft.com/es-es/azure/storage/blobs/data-lake-storage-introduction) para soluciones de análisis de macrodatos empresariales. | Cuando desea que la aplicación admita escenarios de streaming y de acceso aleatorio.  Cuando desea poder tener acceso a datos de aplicación desde cualquier lugar.  Desea crear una instancia empresarial de Data Lake en Azure y realizar análisis de macrodatos. |
| **Azure Elastic SAN**                            | Azure Elastic SAN es una solución totalmente integrada que simplifica la implementación, el escalado, la administración y la configuración de una SAN, al mismo tiempo que ofrece capacidades integradas en la nube, como la alta disponibilidad. | Quiere un almacenamiento a gran escala que sea interoperable con varios tipos de recursos de proceso (como SQL, MariaDB, máquinas virtuales de Azure y Azure Kubernetes Services) a los que se accede mediante el protocolo [Internet Small Computer Systems Interface](https://en.wikipedia.org/wiki/ISCSI) (iSCSI). |
| **Azure Disks**                                  | Permite que los datos se almacenen y se acceda a ellos desde un disco duro virtual conectado de manera persistente. | Cuando desea migrar mediante lift-and-shift aplicaciones que usan las API del sistema de archivos nativo para leer y escribir datos en discos persistentes.  Quiere almacenar los datos a los que no es necesario tener acceso desde fuera de la máquina virtual a la que está conectado el disco. |
| **Azure Container Storage** (versión preliminar) | Azure Container Storage (versión preliminar) es un servicio de administración, implementación y orquestación de volúmenes que se integra con Kubernetes y se compila de forma nativa para contenedores. | Quiere aprovisionar volúmenes persistentes de forma dinámica y automática para almacenar datos para aplicaciones con estado que se ejecutan en clústeres de Kubernetes. |
| **Colas de Azure**                               | Permite la puesta en cola de mensajes asincrónica entre los componentes de la aplicación. | Cuando quiere desacoplar los componentes de la aplicación y usar la mensajería asincrónica para comunicarse entre ellos.  En caso de dudas visitar  [Colas de Storage y de Service Bus: comparación y diferencias](https://learn.microsoft.com/es-es/azure/service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted). |
| **Tablas de Azure**                              | Permite almacenar datos NoSQL estructurados en la nube, lo que proporciona un almacén de claves y atributos con un diseño sin esquema. | Cuando quiere almacenar conjuntos de datos flexibles, como datos de usuarios para aplicaciones web, libretas de direcciones, información de dispositivos u otros tipos de metadatos que el servicio requiera.  En caso de dudas visitar [Desarrollo con Azure Cosmos DB for Table y Azure Table Storage](https://learn.microsoft.com/es-es/azure/cosmos-db/table-support). |
| **Azure NetApp Files**                           | Ofrece un servicio NAS (almacenamiento conectado a la red) de nivel empresarial, de alta disponibilidad y totalmente administrado, que puede administrar las cargas de trabajo más exigentes, de alto rendimiento y baja latencia, que requieren funcionalidades avanzadas de administración de datos. | Tiene una carga de trabajo difícil de migrar, como aplicaciones Linux y Windows compatibles con POSIX, SAP HANA, bases de datos, infraestructura y aplicaciones de proceso de alto rendimiento (HPC) y aplicaciones web empresariales.  Necesita compatibilidad con varios protocolos de almacenamiento de archivos en un único servicio, incluidos NFSv3, NFSv4.1 y SMB3.1.x, lo que permite una amplia gama de escenarios de lift-and-shift de aplicaciones, sin necesidad de cambios en el código. |



<img align="center" src="/img/1ºimagenn.PNG"  />

## Protección

### Cifrado en reposo 

Azure Storage garantiza la seguridad de los datos mediante el cifrado automático mediante **Storage Service Encryption (SSE)**, utilizando el estándar de cifrado avanzado **(AES) de 256 bits** y **cumpliendo con FIPS 140-2**. Este proceso **encripta** automáticamente los datos **al escribirlos** en Azure Storage y los **descifra al leerlos**, sin generar cargos adicionales ni afectar el rendimiento, y no puede ser desactivado.

Para las máquinas virtuales (**VM**), Azure proporciona **Azure Disk Encryption**, que **cifra** los discos duros virtuales (**VHD**) utilizando BitLocker para sistemas **Windows** y **dm-crypt** para **Linux**. **Azure Key Vault gestiona automáticamente las claves** de cifrado del disco, asegurando que incluso si se accede a la imagen VHD, los datos dentro de esta permanezcan inaccesibles.

### Cifrado en tránsito 

Para garantizar la seguridad durante la comunicación entre Azure y el cliente, **se recomienda utilizar HTTPS** en todas las interacciones **a través de Internet pública**. Además, al acceder a objetos en cuentas de almacenamiento a través de **API REST**, **se puede imponer el uso de HTTPS** para garantizar una transferencia segura. Una vez habilitada esta opción, se rechazarán las conexiones HTTP, y también se aplicará la transferencia segura a través de SMB al requerir SMB 3.0 para todos los montajes de archivos compartidos.

### Soporte CORS

 Azure Storage permite el acceso entre dominios mediante el uso compartido de recursos entre orígenes (CORS), lo que **permite a las aplicaciones web cargar contenido solo desde fuentes autorizadas**. Esto se logra mediante el uso de encabezados HTTP para permitir que una aplicación web en un dominio acceda a recursos desde un servidor en otro dominio. El soporte CORS es opcional y se puede habilitar en las cuentas de almacenamiento para agregar los encabezados adecuados en las solicitudes HTTP GET que recuperan recursos de la cuenta de almacenamiento.

### Control de acceso basado en roles 

Azure Storage ofrece control de acceso basado en roles (**RBAC**) para garantizar que cada solicitud a un recurso seguro esté autorizada. Este control se puede gestionar mediante **Microsoft Azure Active Directory**, **permitiendo asignar** **roles** RBAC a entidades de seguridad **cuyo ámbito abarque desde la cuenta de almacenamiento hasta contenedores individuales**. Esto proporciona flexibilidad para autorizar operaciones de administración de recursos y operaciones de datos en blobs y colas de almacenamiento.

### Acceso de auditoría 

Azure Storage facilita la auditoría del acceso a través del servicio integrado Storage Analytics.

Storage Analytics registra cada operación en tiempo real y puede buscar solicitudes específicas en sus registros, permite filtrar según el mecanismo de autenticación, el éxito de la operación o el recurso al que se accedió

## Storage Key accounts

### Claves de la cuenta de almacenamiento

Las cuentas de almacenamiento en Azure están protegidas mediante claves compartidas conocidas como claves de cuenta de almacenamiento. Cada cuenta de almacenamiento creada en Azure recibe dos de estas claves, una principal y otra secundaria. Estas claves son esenciales para acceder a todo el contenido almacenado en la cuenta.

Las claves de la cuenta de almacenamiento se pueden encontrar en la interfaz del Azure Portal, específicamente navegando a la sección de Seguridad + redes > Claves de acceso en el panel de menú izquierdo de la cuenta de almacenamiento. 

### Proteger claves compartidas

Es crucial proteger estas claves compartidas, ya que cada una de ellas otorga acceso completo a la cuenta de almacenamiento. Por lo tanto, se recomienda utilizarlas solo con aplicaciones internas de confianza sobre las cuales se tenga control total.

En caso de que las claves estén en riesgo, se sugiere cambiar sus valores en el Azure Portal, estos son algunos motivos para regenerar las claves:

- Por motivos de seguridad, es posible que vuelvas a generar claves periódicamente.
- Si alguien piratea una aplicación y obtiene la clave codificada o guardada en un archivo de configuración, vuelva a generar la clave. La clave comprometida puede darle al hacker acceso completo a su cuenta de almacenamiento.
- Si su equipo utiliza una aplicación Storage Explorer que conserva la clave de la cuenta de almacenamiento y uno de los miembros del equipo se va, vuelva a generar la clave. De lo contrario, la aplicación seguirá funcionando y le dará al ex miembro del equipo acceso a su cuenta de almacenamiento.

Para actualizar claves

- Cambie cada aplicación confiable para usar la clave secundaria.
- Actualice la clave principal en Azure Portal. Este será el nuevo valor de clave secundaria.

***NOTA IMPORTANTE:***

***Después de actualizar las claves, cualquier cliente que intente utilizar el valor de clave anterior será rechazado. Asegúrese de identificar a todos los clientes que utilizan la clave compartida y actualícelos para mantenerlos operativos.***

## Shared access signatures o firmas de acceso compartido

Como medida recomendada, se aconseja evitar compartir las claves de las cuentas de almacenamiento con aplicaciones externas de terceros. En su lugar, si estas aplicaciones requieren acceso a los datos, se debe proteger sus conexiones sin utilizar las claves de las cuentas de almacenamiento.

Para clientes que no sean de confianza, se sugiere emplear una firma de acceso compartido (SAS). Una SAS consiste en una cadena que contiene un token de seguridad que puede adjuntarse a un URI. Mediante una SAS, se puede delegar el acceso a los objetos de almacenamiento y especificar restricciones, como permisos y el período de tiempo de acceso.

### Tipos de firma de acceso

**SAS de nivel de servicio**: Esta firma se emplea para permitir el acceso a recursos específicos dentro de una cuenta de almacenamiento. Por ejemplo, se puede utilizar una SAS de nivel de servicio para que una aplicación pueda recuperar una lista de archivos en un sistema de archivos o descargar un archivo en particular.

**SAS de nivel de cuenta**: Este tipo de firma permite todo lo que una SAS de nivel de servicio permite, además de ofrecer acceso a recursos y capacidades adicionales. Por ejemplo, mediante una SAS de nivel de cuenta se puede otorgar la capacidad de crear sistemas de archivos dentro de la cuenta de almacenamiento.

En general, se utiliza una SAS en servicios donde los usuarios necesitan leer y escribir datos en la cuenta de almacenamiento. Para este propósito, hay dos enfoques comunes:

- Un servicio proxy de interfaz de usuario, que realiza la autenticación, se encarga de cargar y descargar datos por parte de los clientes. Aunque este servicio frontal ofrece la ventaja de validar las reglas comerciales, puede resultar complicado o costoso escalarlo.

<img align="center" src="/img/2ºimagenn.PNG"  />

- Un servicio más liviano autentica al cliente según sea necesario y luego genera una SAS. Con esta firma, el cliente puede acceder directamente a los recursos de la cuenta de almacenamiento, especificando sus permisos y el intervalo de acceso. Este enfoque reduce la carga en el servicio frontal al evitar enrutar todos los datos a través de él.

<img align="center" src="/img/3ºimagenn.PNG"  />

## Control de acceso a la red de Azure Storage

Por defecto, las cuentas de almacenamiento están configuradas para aceptar conexiones de clientes desde cualquier red. Sin embargo, si desea limitar el acceso a redes específicas, primero debe modificar esta configuración predeterminada. Puede restringir el acceso a direcciones IP, rangos de IP o redes virtuales concretas.

*Es importante tener en cuenta que cambiar las reglas de red puede impactar en la capacidad de su aplicación para conectarse al Azure Storage. Si establece la regla de red predeterminada en "denegar", bloqueará todo el acceso a los datos a menos que se definan reglas de red específicas para otorgar acceso. Antes de modificar la regla predeterminada para denegar el acceso, asegúrese de configurar reglas de red que permitan el acceso a las redes autorizadas.*

Para gestionar las reglas de acceso a la red predeterminadas de las cuentas de almacenamiento, puede utilizar el Azure Portal, PowerShell o la CLI de Azure. Siga estos pasos para cambiar la configuración de acceso a la red predeterminada a través del Azure Portal:

1. Diríjase a la cuenta de almacenamiento que desea proteger.
2. Seleccione la opción "Redes" en el panel izquierdo.
3. Para limitar el acceso solo a redes e IP seleccionadas, elija "Habilitado desde redes virtuales y direcciones IP seleccionadas". Si desea permitir el acceso desde cualquier red, incluida Internet, seleccione "Habilitado desde todas las redes".
4. Una vez realizados los cambios deseados, seleccione "Guardar" para aplicar la nueva configuración.

<img align="center" src="/img/4ºimagenn.PNG"  />

## Advanced Threat Protection for Azure Storage o Proteccion contra amenazas de Azure Storage

Microsoft Defender para Almacenamiento ofrece una capa de seguridad adicional al detectar intentos potencialmente dañinos o inusuales de acceder o explotar cuentas de almacenamiento. Esta protección adicional permite abordar las amenazas sin requerir conocimientos profundos en seguridad ni la gestión de sistemas de monitoreo de seguridad.

Cuando se detectan anomalías en la actividad, se activan alertas de seguridad. Estas alertas están integradas con Microsoft Defender para la Nube y también se envían por correo electrónico a los administradores de la suscripción.

*Microsoft Defender para Almacenamiento actualmente está disponible para Blob Storage, Azure Files y Azure Data Lake Storage Gen2. Los tipos de cuentas compatibles incluyen cuentas de uso general v2, blobs en bloques y almacenamiento de blobs. Esta solución está disponible en todas las nubes públicas y en las nubes del gobierno de EE. UU., pero no en otras regiones de nubes soberanas o en Azure Government.*

Como activar Microsoft Defender para Almacenamiento en Azure:

1. 
   Inicie sesión en el portal de Azure.
2. Vaya a su cuenta de almacenamiento.
3. En la sección de "Seguridad + Redes", seleccione "Microsoft Defender para la Nube".
4. Seleccione la opción para habilitar Microsoft Defender para Almacenamiento.



Este es el tipo de alerta que se recibe en el correo con los siguientes datos:

<img align="center" src="/img/5ºimagenn.PNG"  />

## Características de Seguridad de Azure Data Lake Storage

Azure Data Lake Storage Gen2 ofrece una solución de almacenamiento de datos de alto rendimiento que permite a las empresas consolidar y gestionar sus datos de manera eficiente. Construido sobre Azure Blob Storage, hereda todas las características de seguridad revisadas en este módulo.

Con el control de acceso basado en roles (RBAC), Azure Data Lake Storage Gen2 proporciona listas de control de acceso (ACL) compatibles con POSIX que limitan el acceso a usuarios, grupos o entidades de servicio autorizados. Estas ACL permiten una gestión flexible y detallada de restricciones de acceso. La autenticación en Azure Data Lake Storage Gen2 se realiza mediante tokens de portador Microsoft OAuth 2.0, lo que facilita esquemas de autenticación flexibles, como la federación con Microsoft Azure AD y la autenticación multifactor.

Es importante destacar que estos esquemas de autenticación están integrados con los principales servicios de análisis de datos, como Azure Databricks, HDInsight y Azure Synapse Analytics, así como con herramientas de administración como Azure Storage Explorer. Una vez autenticado, se aplican permisos con granularidad para garantizar el nivel adecuado de autorización para los activos de big data de una empresa.

Azure Storage ofrece cifrado de datos de extremo a extremo y protecciones de capa de transporte, lo que proporciona una seguridad robusta para los datos almacenados. Esta infraestructura de seguridad se extiende a través de todas las herramientas y motores de análisis, asegurando una protección completa a lo largo de los canales de análisis de datos.
