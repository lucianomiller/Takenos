# Takenos - Prueba técnica

# Primera parte

- **Lenguaje y Framework**:
    - **Node.js** con **Express.js** para crear una API RESTful.
- **Base de Datos**:
    - Postgresql para almacenar información del usuario y transacciones, debido a su flexibilidad y escalabilidad.
- **Autenticación**:
    - **Auth0** para la autenticación y autorización, ya que proporciona una integración fácil y segura.
- **Servicios de Proveedores**:
    - **Mint API**: Para la creación de cuentas virtuales en EE.UU. y recepción de depósitos.
    - **Blockflow API**: Para la gestión de wallets crypto y envío de fondos.
- **Infraestructura**:
    - **AWS** para el despliegue y servicios en la nube, utilizando:
        - **EC2** para instancias del servidor.
        - **S3** para almacenamiento de documentos de usuarios.

### Sistema de Autenticación

- **Auth0** se encargará de la autenticación de usuarios. Los usuarios podrán registrarse y acceder utilizando correo electrónico y contraseña o con google.
- **Flujo**:
    1. El usuario se registra/inicia sesión en la aplicación móvil.
    2. Auth0 maneja el flujo de autenticación y devuelve un token JWT.
    3. El token JWT se utiliza para autenticar todas las solicitudes posteriores al backend.

### Mint

- **Mint** se encargará de la creación de las cuentas bancarias receptas en USD, este servicio debe poseer una consolidada experiencia en el rubro, capacidad de respuesta a nivel servicios y a nivel soporte para solución de issues. Debe poseer un dashboard claro donde se pueda ver el flujos de todas las transacciones de los usuarios para el manejos del admin.
- **Mint API**:
    1. Solicitud para enviar documentación de los usuarios a mint para la creación de la cuenta.
    2. Endpoints para recibir notificaciones de mint de apertura de cuenta, tanto si es exitoso como si hay errores en la apertura de la cuenta. 
    3. Endpoints para recibir notificaciones la hora de recibir transacciones.

### **Blockflow API**:

- **Blockflow** se encargará de los envíos de tokens crypto a las wallets de los usuarios.
Debe poseer un sistema solido seguro, rápido y con soporte para atender problemas.
- **Blockflow API**:
    1. Solicitud para envío de tokens.
    2. Endpoints para recibir notificaciones de **Blockflow** del estado de las transacciones.

### **Base de Datos**:

- Se utilizara para almacenar las cuentas de los usuarios con sus datos y sus documentación subida a S3.
- Datos almacenados:
    1. Usuarios con su email, datos personales, wallet de retiro.
    2. Balances: números de cuenta receptoras creadas por mint, balance actual y transacciones realizadas, estado de la cuenta.

### Flujo del Usuario

1. **Registro e Inicio de Sesión**:
    - El usuario se registra o inicia sesión a través de la app.
    - Auth0 maneja la autenticación y proporciona un token JWT si el acceso es exitoso.
    - Solicitud de datos personales si no los tenemos aun o si se necesitan mas, (wallet de retiro)
    - Enviar data para creación de cuenta a Mint y esperar webhook para habilitar cuenta o pedir mas data al usuario.
2. **Ingreso de Dinero**:
    - El usuario solicita ingresar dinero.
    - La aplicación móvil realiza una solicitud al backend para solicitar los datos bancarios par recibir dinero y se los proporciona al usuario.
    - Mint API notifica al backend mediante webhooks una vez que el depósito es exitoso.
    - El backend actualiza el saldo del usuario en la base de datos y crea la transacción de dinero recibido.
    - Notificación al usuario vía email y vía websocket a la app que llego la plata para actualizar el saldo en pantalla.
3. **Retiro de Dinero**:
    - El usuario solicita retirar dinero.
    - La aplicación móvil realiza una solicitud al backend con las validaciones correspondientes.
    - El backend interactúa con Blockflow API para realizar la transacción.
    - Blockflow API notifica al backend mediante webhooks una vez que la transacción es confirmada o rechazada.
    - El backend actualiza el saldo del usuario en la base de datos y crea transacción de retiro si es exitosa.
    - Notificación al usuario vía email y vía websocket a la app que llego la plata para actualizar el saldo en pantalla.
4. **Ver Saldo**:
    - El usuario solicita ver su saldo.
    - La aplicación móvil realiza una solicitud al backend.
    - El backend consulta el saldo y lo devuelve a la aplicación móvil.
    - Lo mismo para ver transacciones realizadas.
5. **Soporte**:
    - El usuario solicita ayuda o pide solución de errores, puede dejar consultas que se guardaran en la base de datos.

# Segunda parte

Para lograr un rendimiento óptimo y escalabilidad se propone un enfoque basado en microservicios. Este enfoque permitirá escalar componentes individuales según la demanda y mejorar la eficiencia general del sistema. A continuación, se detalla el plan:

### Arquitectura del Sistema

1. **Microservicio Independiente**
    - Crear un microservicio independiente para gestionar las queries de visa. Esto permitirá escalar este componente de manera independiente según sea necesario.
2. **Uso de Redis para Cache**
    - Implementar Redis para almacenar en caché los datos de la base de datos y acceder a ellos rápidamente.
    - Actualizar el caché en tiempo real cuando haya cambios en el balance del usuario para mantener la información siempre actualizada.
3. **Autoescalado y Balanceo de Carga**
    - Configurar autoescalado para el microservicio de recompensas en caso de tráfico elevado.
    - Implementar un balanceador de carga para distribuir las solicitudes de manera uniforme entre las instancias del microservicio.
4. **Despliegue Geográficamente Cercano**
    - Desplegar el microservicio en regiones geográficas cercanas a los usuarios para minimizar la latencia.
5. **Framework de Bajo Nivel**
    - Utilizar un framework rápido y de bajo nivel, como Gin en Golang, para aumentar la eficiencia de los recursos y mejorar el rendimiento del microservicio.

# Tercera parte

Desarrollo de Funcionalidades de Ahorro para Takenos

Se propone implementar un sistema de recompensas para motivar a los usuarios a dejar su dinero en Takenos y obtener rendimientos sobre su capital. Las recompensas pueden ser generadas en criptomonedas o en dinero fiat, dependiendo de cuál presente el mejor rendimiento en el servicio. Además, los usuarios pueden optar por diferentes tipos de servicios: rendimientos diarios o periodos de bloqueo, los cuales ofrecen mayores intereses.

- **Rendimientos Diarios:** Los usuarios pueden activar una funcionalidad que les permite generar intereses diarios sobre sus depósitos.
- **Rendimientos Bloqueados:** Los usuarios pueden optar por periodos de bloqueo (30, 60, 90 días, etc.) para obtener mayores tasas de interés.

- **Worker Diario**
    - Un worker se ejecutará diariamente y se comunicará con el banco encargado de los intereses para obtener la información del rendimiento de ese día.
    - El worker generará en la base de datos las transacciones de intereses generadas por el banco, ajustando los montos de acuerdo con la estrategia de Takenos para descontar sus ganancias.
- **Actualización del Balance**
    - El balance de los usuarios se actualizará automáticamente con los intereses generados, ya sea diariamente o al final del periodo de bloqueo.

**Interfaz de Usuario**

- Se diseñará una interfaz intuitiva que permita a los usuarios activar la funcionalidad de rendimientos diarios o seleccionar periodos de bloqueo.
- Los usuarios podrán ver sus balances actualizados y los intereses generados en tiempo real.