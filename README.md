# GRADEX - Prototipo 3



![logo unal](https://github.com/user-attachments/assets/c1925586-3eaa-43e5-9a0e-a25c4a5093aa)



## 1. **Team** 
- name: 2F  
- Integrantes: 
    - David Stiven Martinez Triana 
    - Carlos Sebastian Gomez Fernandez 
    - Nestor Steven Piraquive Garzon 
    - Luis Alfonso Pedraos Suarez 
    - Cesar Fabian Rincon Robayo 
## 2. **Software System** 
- name: Gradex 
- Logo:
  
![logo gradex](https://github.com/user-attachments/assets/b3657f68-ea46-4990-8fa8-a0c4b5c26e13)


- Description: Sistema de Gestión de calificaciones para colegios.

## 3. Architectural Structures


### Component-and Connector (C&C) Structure

![Copy of Diagrama C C](https://github.com/user-attachments/assets/f30b778b-27ed-40d6-999a-5e5e165c3197)


#### **Description of architectural styles and patterns used**

**Microservicios:**  
 Los servicios backend (por ejemplo `GX_BE_EstCur` y `GX_BE_ProAsig`) son independientes:

* Usan diferentes tecnologías: Django, Spring Boot, Go.

* Se comunican a través de HTTP (REST o GraphQL) con un API Gateway.

* Cada uno mantiene su propia base de datos (PostgreSQL o MongoDB).

* No existe acoplamiento directo entre ellos.

**Estilo Arquitectónico Cliente-Servidor:**
El sistema presenta una separación clara entre el cliente (frontend) y los servidores (backends):
* Cliente: GX_FE_Gradex. Ubicado en la Capa de Presentación. Consume APIs expuestas por los servicios del backend usando HTTPS GraphQL.
* Servidor: GX_BE_EstCur y GX_BE_ProAsigCal.Ubicados en la Capa de Lógica, se encargan del procesamiento de datos y lógica de negocio.

**Patrones utilizados:**

* **API Gateway Pattern**: Punto único de entrada (`gx_api_gateway`) gestionado por NGINX (`gx_nginx_proxy`), ocultando la complejidad del backend a los clientes frontend y Electron.

* **Event-driven architecture**: `gx_be_calif` interactúa directamente con `gx_be_rabbitmq` mediante AMQP.

* **Database per Service**: Cada microservicio tiene su propia base de datos dedicada.

* **Load Balancing Frontend**: Uso de `gx_nginx_balancer` para distribuir tráfico entre múltiples instancias frontend (`gx_fe_gradex_1`, `2`, `3`).

---

#### **Description of architectural elements and relations**

**gx\_nginx\_balancer**

* Función: Balanceador de carga HTTPS hacia frontends.

* Tecnología: NGINX

* Propósito: Distribuir peticiones entrantes entre instancias React/Next.js.

**gx\_fe\_gradex\_1, gx\_fe\_gradex\_2, gx\_fe\_gradex\_3**

* Tecnología: React \+ Next.js

* Función: Frontend web de usuarios finales.

* Comunicación: HTTPS \+ GraphQL hacia `gx_nginx_proxy`.

**GX\_WD\_Gradex (Desktop App)**

* Tecnología: Electron.js

* Función: Interfaz de escritorio local.

* Comunicación: HTTPS \+ GraphQL hacia `gx_nginx_proxy`.

**gx\_nginx\_proxy**

* Función: Reverso proxy que expone el API Gateway.

* Tecnología: NGINX

* Comunicación: HTTP GraphQL hacia `gx_api_gateway`.

**gx\_api\_gateway**

* Tecnología: Express.js \+ Apollo Server

* Función: API Gateway para orquestar microservicios.

* Comunicación: HTTP REST y GraphQL con todos los microservicios.

**gx\_be\_rabbitmq**

* Tecnología: RabbitMQ

* Función: Sistema de mensajería asíncrona.

* Comunicación: AMQP con `GX_BE_Calif`.

**GX\_BE\_Auth**

* Tecnología: Go

* Función: Autenticación y emisión de JWT

* DB: PostgreSQL (`GX_DB_Auth`) vía lib pgx.

**GX\_BE\_EstCur**

* Tecnología: Django

* Función: Gestión de estudiantes y cursos

* DB: PostgreSQL (`GX_DB_EstCur`) vía ORM de Django.

**GX\_BE\_ProAsig**

* Tecnología: Spring Boot

* Función: Gestión de profesores y asignaturas

* DB: MongoDB (`GX_DB_ProAsig`) con Spring Data.

**GX\_BE\_Calif**

* Tecnología: Spring Boot

* Función: Gestión de calificaciones

* DB: MongoDB (`GX_DB_Calif`)

* Comunicación: HTTP GraphQL con API Gateway y AMQP con RabbitMQ.

🔌 **Conectores (Puertos y protocolos)**

| Origen | Destino | Protocolo | Puerto | Observaciones |
| ----- | ----- | ----- | ----- | ----- |
| **`gx_nginx_balancer`** | **`gx_fe_gradex`** | **HTTPS** | **3000** | **Balanceo hacia frontend Next.js** |
| **`gx_nginx_balancer`** | **Usuario externo** | **HTTPS** | **443 / 80** | **Entrada desde navegador** |
| **`gx_fe_gradex`** | **`gx_nginx_proxy`** | **HTTPS** | **444** | **Proxy reverso para GraphQL** |
| **`GX_WD_Gradex`** | **`gx_nginx_proxy`** | **HTTPS** | **444** | **Conexión desde app de escritorio** |
| **`gx_nginx_proxy`** | **`gx_api_gateway`** | **HTTP GraphQL** | **4000** | **Entrada única a microservicios** |
| **`gx_api_gateway`** | **`GX_BE_Auth`** | **HTTP REST** | **8082** | **Servicio de autenticación** |
| **`gx_api_gateway`** | **`GX_BE_EstCur`** | **HTTP REST** | **8000** | **Estudiantes y cursos** |
| **`gx_api_gateway`** | **`GX_BE_ProAsig`** | **HTTP GraphQL** | **8080** | **Profesores y asignaturas** |
| **`gx_api_gateway`** | **`GX_BE_Calif`** | **HTTP GraphQL** | **8081** | **Calificaciones** |
| **`GX_BE_Calif`** | **`gx_be_rabbitmq`** | **AMQP** | **5672 / 5673** | **Eventos asincrónicos** |
| **`GX_BE_Auth`** | **`GX_DB_Auth`** | **TCP/IP** | **5432** | **PostgreSQL** |
| **`GX_BE_EstCur`** | **`GX_DB_EstCur`** | **TCP/IP** | **5433** | **PostgreSQL** |
| **`GX_BE_ProAsig`** | **`GX_DB_ProAsig`** | **TCP/IP** | **27018** | **MongoDB** |
| **`GX_BE_Calif`** | **`GX_DB_Calif`** | **TCP/IP** | **27017** | **MongoDB** |


### Layered Structure
#### Layered View:
![cesar Copia de Copy of Diagrama C C](https://github.com/user-attachments/assets/5fc98764-773c-499a-a50d-9eeafc757116)





####  Description:

  **Capa de Presentación**
  Permite la interacción del usuario a través de:
  Web (Next.js) y Escritorio (Electron).
  Se comunica con el sistema mediante GraphQL hacia el API Gateway.

  **Capa de Comunicación**
  Gestiona la entrada de solicitudes y coordina servicios.
  API Gateway (Apollo \+ Express.js) para centralizar peticiones.
  Broker asíncrono (Express.js) con RabbitMQ para procesamiento de eventos.

  **Capa Lógica**
  Contiene la lógica de negocio:
  Autenticación (Go)
  Gestión académica (Django)
  Profesores, asignaturas y calificaciones (Spring Boot)

  **Capa de Datos**
  Responsable de persistir la información:
  PostgreSQL para autenticación y datos académicos generales.
  MongoDB para profesores, asignaturas y calificaciones.

### Deployment Structure
#### Deployment View

![Copy of Diagrama de Despliegue](https://github.com/user-attachments/assets/d1159354-c255-487b-a1c1-e49460d87460)





Este sistema está compuesto por múltiples microservicios y componentes backend que se ejecutan dentro de una red interna Docker, y un componente cliente desarrollado con Electron.js, que opera desde el dispositivo del usuario final como aplicación de escritorio. Todos los servicios están expuestos por distintos puertos y protocolos, y se comunican mediante HTTP, AMQP o TCP/IP.

 **Cliente de Escritorio \- Electron.js**

* **Tipo**: Aplicación de escritorio.  
* **Tecnología**: Electron.js (Node.js \+ Chromium)  
* **Ubicación**: Fuera del entorno Docker, en el equipo del. usuario.  
* **Función**:  
  * Interfaz gráfica de usuario del sistema.  
  * Se conecta a múltiples servicios del backend mediante HTTP, AMQP o TCP/IP.  
  * Sirve como punto de entrada principal del sistema para los usuarios finales.

 **Red Interna Docker**

Contiene todos los contenedores y servicios backend. Se encuentra encapsulada en una red interna aislada que permite la comunicación entre servicios por nombre y puerto.

 **1\. GX\_BE\_Auth (Go)**

**Nombre del contenedor:** gx\_be\_auth  
 **Tecnología:** Go

**Puerto expuesto:** 8082  
 **Rol:** Microservicio de Autenticación  
 **Responsabilidad:** Gestiona autenticación, registro, validación de credenciales, emisión de tokens JWT, y mantiene la seguridad del sistema mediante el control de acceso.

 **2\. GX\_BE\_EstCur (Django/Python)**

**Nombre del contenedor:** gx\_be\_estcur  
 **Tecnología:** Django (Python)

**Puerto expuesto:** 8083  
 **Rol:** Microservicio de Gestión de Estudiantes y Cursos  
 **Responsabilidad:** Manejo de operaciones CRUD relacionadas con estudiantes y cursos, administración de perfiles estudiantiles y organización académica básica.

 **3\. GX\_BE\_ProAsig (Spring Boot/Java)**

**Nombre del contenedor:** gx\_be\_proasig  
 **Tecnología:** Spring Boot (Java)

**Puerto expuesto:** 8080  
 **Rol:** Microservicio de Gestión de Profesores y Asignaturas  
 **Responsabilidad:** Administración de los docentes y sus asignaturas asociadas. Permite crear, modificar, consultar y eliminar la información académica del personal docente.

 **4\. GX\_BE\_Calif (Spring Boot/Java)**

**Nombre del contenedor:** gx\_be\_calif  
 **Tecnología:** Spring Boot (Java)

**Puerto expuesto:** 8081  
 **Rol:** Microservicio de Calificaciones  
 **Responsabilidad:** Control de las notas de los estudiantes: creación, modificación, consulta y almacenamiento seguro de calificaciones académicas.

 **5\. GX\_FE\_Gradex (Next.js/React)**

**Nombre del contenedor:** gx\_fe\_gradex  
 **Tecnología:** Next.js (React)

**Puerto expuesto:** 3000  
 **Rol:** Cliente Web  
 **Responsabilidad:** Proporciona la interfaz gráfica del sistema GRADEX accesible desde navegadores web para estudiantes, profesores y administradores.

 **6\. GX\_FE\_Gradex\_Desktop (Electron.js)**

**Nombre del contenedor:** GX\_FE\_Gradex\_Desktop  
 **Tecnología:** Electron.js

**Puerto expuesto:** 4000  
 **Rol:** Cliente de Escritorio  
 **Responsabilidad:** Interfaz nativa multiplataforma instalada en el dispositivo del usuario final. Permite interacción con el sistema mediante GUI conectándose al API Gateway.

 **7\. gx\_api\_gateway (Node.js/Apollo Server)**

**Nombre del contenedor:** gx\_api\_gateway  
 **Tecnología:** Node.js \+ Apollo Server

**Puerto expuesto:** 9000  
 **Rol:** API Gateway  
 **Responsabilidad:** Punto único de entrada al backend. Orquesta peticiones de frontends a microservicios. Expone GraphQL y maneja autenticación, validación y delegación de llamadas.

 **8\. gx\_be\_comun\_async (Node.js)**

**Nombre del contenedor:** gx\_be\_comun\_async  
 **Tecnología:** Node.js (Express.js)

**Puerto expuesto:** 8081  
 **Rol:** Broker HTTP Asíncrono  
 **Responsabilidad:** Recibe eventos desde RabbitMQ y los retransmite al microservicio gx\_be\_calif por HTTP, facilitando una arquitectura basada en eventos.

 **9\. gx\_be\_rabbitmq (RabbitMQ)**

**Nombre del contenedor:** gx\_be\_rabbitmq  
 **Tecnología:** RabbitMQ

**Puerto expuesto:** 5673  
 **Rol:** Sistema de Mensajería Asíncrona  
 **Responsabilidad:** Actúa como cola de mensajes AMQP para desacoplar procesos. Recibe, mantiene y entrega mensajes entre servicios de manera fiable y asincrónica.

 **10\. GX\_DB\_Auth (PostgreSQL)**

**Nombre del contenedor:** gx\_db\_auth  
 **Tecnología:** PostgreSQL

**Puerto expuesto:** 5432  
 **Rol:** Base de Datos de Autenticación  
 **Responsabilidad:** Almacena usuarios, credenciales, roles y tokens de sesión relacionados con el servicio de autenticación.

 **11\. GX\_DB\_EstCur (PostgreSQL)**

**Nombre del contenedor:** gx\_db\_estcur  
 **Tecnología:** PostgreSQL

**Puerto expuesto:** 5433  
 **Rol:** Base de Datos de Estudiantes y Cursos  
 **Responsabilidad:** Persistencia estructurada de la información académica básica como estudiantes y cursos.

 **12\. GX\_DB\_ProAsig (MongoDB)**

**Nombre del contenedor:** gx\_db\_proasig  
 **Tecnología:** MongoDB

**Puerto expuesto:** 27018  
 **Rol:** Base de Datos de Profesores y Asignaturas  
 **Responsabilidad:** Almacena datos académicos del personal docente y sus materias de forma flexible usando documentos JSON.

 **13\. GX\_DB\_Calif (MongoDB)**

**Nombre del contenedor:** gx\_db\_calif  
 **Tecnología:** MongoDB

**Puerto expuesto:** 27019  
 **Rol:** Base de Datos de Calificaciones  
 **Responsabilidad:** Gestión de las calificaciones estudiantiles. Permite un almacenamiento eficiente y escalable de las evaluaciones.

 **Nodo de Despliegue (Servidor on-premise)**

* **Procesador:** Intel Core i5 @ 2.27GHz  
* **Memoria RAM:** 8 GB  
* **Disco Duro:** 1 TB SSD  
* **Tipo de Despliegue:** Servidor físico local (on-premise)  
* **S.O. y Contenedores:** Se asume sistema Linux con servicios desplegados como contenedores Docker
  

### Decomposition Structure
#### Decomposition View

![Diagrama de Descompocicion2](https://github.com/user-attachments/assets/0a7d19bb-a6c7-454d-8308-0dd226006d09)




#### Description of architectural elements and relations:

* **GX\_FE\_Gradex**: Es el componente frontend del sistema GRADEX para una web app, encargado de la interfaz de usuario donde estudiantes, profesores y administradores interactúan con la plataforma. Proporciona formularios, listados y visualización de datos, comunicándose con el API Gateway para realizar todas las operaciones.


* **GX\_WD\_Gradex**: Es el componente frontend del sistema GRADEX para una aplicación de escritorio, encargado de la interfaz de usuario donde estudiantes, profesores y administradores interactúan con la plataforma. Proporciona formularios, listados y visualización de datos, comunicándose con el API Gateway para realizar todas las operaciones.


* **gx\_api\_gateway**: Actúa como intermediario entre el frontend y los microservicios backend. Centraliza todas las solicitudes, enrutándolas al servicio correspondiente y manejando aspectos como autenticación y unificación de respuestas para simplificar la comunicación.


* **GX\_BE\_Auth**: Se encarga de la autenticación y gestión de usuarios, incluyendo registro e inicio de sesión para estudiantes, profesores y administradores. Garantiza el acceso seguro al sistema mediante validación de credenciales.


* **GX\_BE\_EstCur**: Gestiona la información relacionada con estudiantes y cursos, permitiendo operaciones como creación, consulta, modificación y eliminación de registros. Es fundamental para la administración académica básica.


* **GX\_BE\_Calif**: Administra todo lo referente a calificaciones, incluyendo su registro, consulta y actualización. Este componente asegura que los datos de evaluación de los estudiantes sean almacenados y accesibles de manera confiable.


* **Registro**: Proceso específico para dar de alta nuevos usuarios en el sistema, ya sean profesores o administradores, validando sus datos antes de permitirles acceso.


* **Login**: Módulo que autentica a los usuarios (profesores y administradores) mediante credenciales, permitiéndoles ingresar a sus respectivas áreas dentro de la plataforma.


* **Estudiantes**: Área dedicada a la gestión de perfiles estudiantiles, donde se pueden realizar acciones como agregar, consultar, modificar o eliminar estudiantes del sistema.


* **Cursos**: Componente que maneja la creación, consulta, actualización y eliminación de cursos, facilitando la organización de la oferta académica.


* **Calificaciones**: Sección especializada en el manejo de las notas y evaluaciones de los estudiantes, permitiendo su registro y modificación según sea necesario.


* **Broker**: Elemento que actúa como cola de mensajes en la comunicación entre el servicio de calificaciones, asegurando que los mensajes y datos fluyan correctamente entre los componentes.


* **Profesor**: Rol dentro del sistema que tiene permisos para gestionar calificaciones de estusiantes, además de acceder a información específica de los estudiantes que enseña.


* **Administrador**: Rol con privilegios elevados para gestionar usuarios, cursos y configuraciones globales del sistema, asegurando su correcto funcionamiento y crear y administrar profesores asignaturas y cursos.


* **Crear/Consultar/Eliminar/Modificar**: Operaciones básicas disponibles en distintos módulos del sistema, permitiendo la administración completa de estudiantes, cursos y calificaciones.



## 4. Quality Attributes

En la tercera entrega del prototipo se incorporaron diversas tácticas y técnicas orientadas a mejorar la seguridad, el rendimiento y la escalabilidad del sistema. Estas dimensiones son fundamentales para garantizar la confiabilidad, eficiencia y capacidad de crecimiento del software frente a un entorno dinámico y exigente.

A continuación, se describen las tácticas y técnicas implementadas para abordar estos atributos de calidad, así como los resultados obtenidos a partir de las pruebas realizadas de rendimiento y escalabilidad.

### Security

#### Security scenarios

#### Applied architectural tactics
* **Encrypt data** es una táctica arquitectónica de seguridad que consiste en cifrar la información sensible tanto en tránsito como en reposo. Esto garantiza que, incluso si los datos son interceptados o accedidos de forma no autorizada, no puedan ser leídos sin la clave de descifrado correspondiente. Su implementación protege la confidencialidad de la información crítica, como credenciales de usuarios o datos académicos, y es esencial para cumplir con normativas de protección de datos y prevenir brechas de seguridad.

* **Limit Access** es una táctica arquitectónica de seguridad que restringe el acceso a recursos del sistema únicamente a usuarios o servicios autorizados, siguiendo el principio de mínimo privilegio. . La táctica reduce superficies de ataque al garantizar que cada actor solo pueda interactuar con los componentes y datos estrictamente necesarios para su función, previniendo accesos maliciosos o errores humanos que comprometan la integridad del sistema.

* **Detect Service Denial** es una táctica arquitectónica de seguridad enfocada en identificar y responder ante intentos de denegación de servicio (DoS/DDoS) que buscan afectar la disponibilidad del sistema. Esta táctica permite detectar actividades sospechosas -como intentos de saturación de APIs- y activar contramedidas automáticas (rate limiting, bloqueo de IPs maliciosas) o manuales para mitigar el impacto. En GRADEX, sería clave para proteger endpoints críticos como autenticación (GX_BE_Auth) o consultas masivas de calificaciones (GX_BE_Calif), asegurando que el sistema permanezca accesible para usuarios legítimos.

#### Applied architectural patterns


* **Secure Channel Pattern** es un patrón arquitectónico de seguridad que garantiza la confidencialidad e integridad de los datos transmitidos entre componentes del sistema, mediante el establecimiento de canales de comunicación cifrados. Usa la tactica arquitectonica **Encrypt Data**

* **Reverse Proxy Pattern** es un patrón de seguridad que actúa como intermediario entre clientes y servidores backend, protegiendo la infraestructura al ocultar los servidores internos, filtrar tráfico malicioso (como ataques DDoS o inyecciones SQL), gestionar el cifrado SSL/TLS, y balancear la carga entre instancias para optimizar el rendimiento. Usa la tactica arquitectonica **Limit Access**

* **Network Segmentation Pattern** es un patrón de seguridad que divide la red en segmentos o zonas lógicas (subredes, VLANs, dominios de seguridad) para aislar componentes críticos y limitar el movimiento lateral de amenazas. Usa la tactica arquitectonica **Limit Access**

* **WAF Pattern (Web Application Firewall)** es un patrón de seguridad que implementa un firewall especializado en proteger aplicaciones web al filtrar y monitorear el tráfico HTTP/HTTPS, bloqueando ataques comunes como inyecciones SQL, XSS (Cross-Site Scripting) o solicitudes maliciosas. Usa la tactica arquitectonica **Detect Service Denial**


### Performance and Scalability 

#### Performance scenarios

#### Applied architectural tactics

* **Maintain Multiple Copies of Computations** es una táctica de rendimiento y escalabilidad que consiste en ejecutar múltiples instancias redundantes de un mismo componente o servicio (como microservicios o procesos críticos) para distribuir la carga de trabajo y garantizar alta disponibilidad. La táctica mejora la capacidad de respuesta del sistema y se complementa con herramientas como Kubernetes para orquestación automática o bases de datos replicadas (PostgreSQL/MongoDB) para consistencia en los datos.

* **Improve Efficiency** es una táctica arquitectónica de rendimiento y escalabilidad que optimiza el uso de recursos para maximizar la capacidad del sistema y reducir tiempos de respuesta. Se implementa mediante técnicas como caching (almacenar resultados frecuentes en memoria) o pooling de conexiones (reutilizar recursos costosos como conexiones a BD).  Es clave para sistemas con cargas variables o limitaciones de hardware.
 
#### Applied architectural patterns

* **Load Balancer Pattern** es un patrón de rendimiento y escalabilidad que distribuye el tráfico entrante entre múltiples instancias de un servicio para optimizar el uso de recursos, evitar la sobrecarga en nodos individuales y garantizar alta disponibilidad. Esto permite escalar horizontalmente y tolerar fallos, asegurando que picos de tráfico no afecten la capacidad de respuesta del sistema. Usa la tactica arquitectonica **Maintain Multiple Copies of Computations**

#### Performance testing analysis and results

## 5. Repositorio del proyecto

Para utilizar el proyecto simplemente clona el repositorio principal, el cual ya incluye todos los submódulos necesarios:

```bash
git clone --recursive https://github.com/Swarch2F/prototipo3.git
cd prototipo3
```

### Información sobre submódulos (Solo informativo)

Los submódulos fueron agregados inicialmente con estos comandos, pero no necesitas ejecutarlos nuevamente:

```bash
git submodule add https://github.com/Swarch2F/component-1.git components/component-1
git submodule add https://github.com/Swarch2F/component-2-1.git components/component-2-1
git submodule add https://github.com/Swarch2F/component-2-2.git components/component-2-2
git submodule add https://github.com/Swarch2F/component-3.git components/component-3
git submodule add https://github.com/Swarch2F/component-4.git components/component-4
git submodule add https://github.com/Swarch2F/broker.git components/broker
git submodule add https://github.com/Swarch2F/api-gateway.git components/api-gateway
```

### Actualización de submódulos recursivamente (por primer vez, una vez clonado el proyecto. Si realizo el clonado con "git clone --recursive" no es necesario):

```bash
git submodule update --init --recursive
git submodule update --remote --merge --recursive
```

### Levantar el prototipo con Docker Compose

El proyecto utiliza Docker Compose para gestionar la ejecución de todos los servicios.

### Ejecución rápida

Una vez clonado el proyecto, ejecuta:

```bash
docker compose up --build
```

---
**© 2025 Swarch2F. GRADEX Prototipo 3** 
