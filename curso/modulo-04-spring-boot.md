# Módulo 04 — Spring Boot: Tu Primer Servidor
## 🐱 Abriendo el Michi Café al público

---

## 🏠 Introducción

Hasta ahora hemos aprendido Java y POO. Nuestro código corre en nuestra computadora
pero nadie más puede usarlo.

Es como tener el Michi Café completamente equipado — con recetas, empleados y menú —
pero con la puerta cerrada y sin teléfono. Nadie puede comunicarse con nosotros.

**Spring Boot** es lo que nos permite abrir esa puerta. Vamos a crear un **servidor**:
un programa que escucha peticiones que llegan por internet y responde a ellas.

---

## 💡 ¿Qué es un servidor?

Un servidor es un programa que está siempre encendido, esperando peticiones.
Cuando llega una, la procesa y devuelve una respuesta.

### Analogía 🐾

El Michi Café tiene un número de teléfono para pedidos a domicilio.
El gatito que contesta siempre está disponible:

1. Suena el teléfono → llega una petición HTTP
2. El gatito contesta → el servidor recibe la petición
3. El gatito pregunta qué quieren → el servidor lee los datos enviados
4. El gatito busca la información → el servidor ejecuta la lógica
5. El gatito responde → el servidor envía la respuesta

Ese gatito que contesta es tu servidor Spring Boot.

---

## 💡 ¿Qué es Spring Boot 4.1.0?

**Spring** es un framework (conjunto de herramientas) para Java que facilita
crear aplicaciones empresariales.

**Spring Boot** es una versión de Spring que viene preconfigurada y lista para usar.
La versión actual es **4.1.0** y requiere **Java 17** como mínimo.

### Cambios importantes vs versiones anteriores

Si has visto tutoriales más viejos, notarás diferencias. Las principales:

| Versión antigua (≤ 2.x) | Versión actual (4.1.0) |
|------------------------|------------------------|
| Java 8 o 11 | Java 17 mínimo |
| `javax.*` en los imports | `jakarta.*` en los imports |
| Spring Fox para Swagger | SpringDoc OpenAPI |
| `spring-boot-starter-webmvc` (Spring MVC) | Sigue igual ✅ |

> 💡 El cambio de `javax` a `jakarta` es el más importante. Si ves código antiguo
> con `import javax.persistence.*` o `import javax.validation.*`, en Spring Boot 4.1.0
> esos imports son `import jakarta.persistence.*` y `import jakarta.validation.*`.

---

## 💡 ¿Qué es Maven y el archivo `pom.xml`?

**Maven** es una herramienta que gestiona las **dependencias** de tu proyecto.

Una dependencia es una librería de código que alguien más ya escribió y que tú
puedes usar en tu proyecto sin tener que escribirla desde cero.

### Analogía 🐾

Imagina que vas a preparar un Latte de vainilla. No vas a cultivar el café, ordeñar
la vaca ni fabricar el jarabe de vainilla. **Los compras ya hechos**.

Maven es el sistema de compras del Michi Café:
- Tú dices qué necesitas en el archivo `pom.xml` (la lista de compras)
- Maven descarga automáticamente todo lo necesario de internet
- Cada vez que alguien más abra el proyecto, Maven descarga lo mismo

### ¿Cómo se ve una dependencia en el `pom.xml`?

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>    <!-- La marca -->
    <artifactId>spring-boot-starter-webmvc</artifactId> <!-- El producto -->
</dependency>
```

No necesitas escribir la versión porque el `pom.xml` hereda las versiones del
**Spring Boot Parent**, que las gestiona automáticamente:

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>4.1.0</version>  <!-- Versión de Spring Boot -->
</parent>
```

---

## 💡 Las dependencias principales que usaremos

Estos son los "ingredientes" que vamos a pedir en nuestra lista de compras:

| Dependencia (artifactId) | ¿Para qué sirve? |
|--------------------------|-----------------|
| `spring-boot-starter-webmvc` | Crear APIs REST (controladores, peticiones HTTP) |
| `spring-boot-starter-data-mongodb` | Conectar con MongoDB |
| `spring-boot-starter-validation` | Validar datos de entrada |
| `spring-boot-starter-actuator` | Endpoints de salud y métricas |
| `spring-cloud-starter-gateway` | Solo para el API Gateway |

---

## 💡 ¿Qué es Spring Initializr?

**Spring Initializr** es una página web oficial de Spring que genera el esqueleto
de tu proyecto automáticamente. Es el punto de partida de cualquier proyecto Spring Boot.

URL: **https://start.spring.io**

### ¿Qué genera?

- La estructura de carpetas correcta
- El `pom.xml` con las dependencias que elijas
- La clase principal con `@SpringBootApplication`
- El archivo `application.properties` vacío
- Un test básico

Todo esto en un archivo ZIP que descargas y abres en IntelliJ.

> 💡 En la Parte 2 del curso (módulos de proyecto) verás capturas exactas de
> cómo configurar Spring Initializr paso a paso. Aquí solo explicamos los conceptos.

---

## 💡 La estructura de un proyecto Spring Boot

Cuando abres el proyecto descargado en IntelliJ, verás esta estructura:

```
michicafe/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/michicafe/
│   │   │       └── MichicafeApplication.java   ← Punto de entrada
│   │   └── resources/
│   │       └── application.properties           ← Configuración
│   └── test/
│       └── java/
│           └── com/michicafe/
│               └── MichicafeApplicationTests.java
├── pom.xml                                       ← Lista de dependencias
└── mvnw / mvnw.cmd                              ← Maven wrapper
```

### ¿Qué es cada cosa?

| Archivo/Carpeta | Función | Analogía |
|----------------|---------|----------|
| `src/main/java` | Todo tu código Java | Las recetas del café |
| `src/main/resources` | Configuraciones, archivos estáticos | El manual de configuración |
| `src/test/java` | Pruebas automatizadas | El inspector de calidad |
| `pom.xml` | Lista de dependencias | La lista de compras |
| `application.properties` | Configuración del servidor | El panel de control del café |
| `mvnw` | Maven empaquetado (no necesitas instalarlo) | El asistente de compras incluido |

---

## 💡 La clase principal: el interruptor que enciende todo

```java
package com.michicafe;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication  // ← Esta anotación activa toda la magia de Spring Boot
public class MichicafeApplication {

    public static void main(String[] args) {
        // Esta línea arranca el servidor
        // Es como girar la llave para encender el café
        SpringApplication.run(MichicafeApplication.class, args);
    }
}
```

`@SpringBootApplication` es una sola anotación que en realidad activa tres cosas:
- `@SpringBootConfiguration`: esta clase es la configuración principal
- `@EnableAutoConfiguration`: Spring configura automáticamente lo que detecta
- `@ComponentScan`: Spring busca en este paquete todas las clases anotadas

### Analogía 🐾

Es el **interruptor principal del café**: cuando lo activas, se encienden las luces,
se calientan las máquinas, se conecta el sistema de pedidos. Todo arranca solo.

---

## 💡 ¿Qué es un puerto?

Un puerto es como la puerta específica de un edificio.
Tu computadora tiene miles de puertas (puertos), y cada servicio usa una diferente.

| Puerto | Servicio | Analogía |
|--------|---------|----------|
| 8080 | Tu API (Spring Boot por defecto) | Puerta principal del café |
| 8081 | servicio-bebidas | Puerta del área de bebidas |
| 8082 | servicio-clientes | Puerta del área de clientes |
| 8083 | servicio-pedidos | Puerta del área de pedidos |
| 27017 | MongoDB | Puerta del almacén |

Cuando el servidor arranca, verás en la consola de IntelliJ:

```
Tomcat started on port 8080 (http) with context path '/'
Started MichicafeApplication in 2.345 seconds
```

Eso significa que el café está abierto en `http://localhost:8080`.
`localhost` significa "esta misma computadora".

---

## 💡 Anotaciones de Spring: las etiquetas mágicas

Las **anotaciones** en Spring Boot son instrucciones especiales que empiezan con `@`.
Le dicen a Spring cómo debe tratar cada clase o método.

### Analogía 🐾

Son como las etiquetas que pones en las cajas del café:
- Una caja con etiqueta `@Cocina` va a la cocina
- Una caja con etiqueta `@Caja` va a la caja registradora
- Spring lee esas etiquetas y sabe exactamente qué hacer con cada cosa

### Las anotaciones más importantes

| Anotación | ¿Dónde se usa? | ¿Qué hace? |
|-----------|---------------|-----------|
| `@SpringBootApplication` | Clase principal | Activa todo Spring Boot |
| `@RestController` | Clase | Esta clase recibe y responde peticiones HTTP |
| `@RequestMapping("/ruta")` | Clase o método | Define la ruta base |
| `@GetMapping` | Método | Responde a peticiones GET |
| `@PostMapping` | Método | Responde a peticiones POST |
| `@PutMapping` | Método | Responde a peticiones PUT |
| `@DeleteMapping` | Método | Responde a peticiones DELETE |
| `@Service` | Clase | Esta clase contiene lógica de negocio |
| `@Repository` | Interfaz | Esta interfaz habla con la base de datos |
| `@Autowired` | Campo | Spring inyecta aquí la instancia de esa clase |
| `@Document` | Clase | Esta clase es un documento de MongoDB |
| `@Id` | Campo | Este campo es el identificador único |

---

## 💡 Las capas de Spring Boot

Un proyecto bien organizado en Spring Boot tiene **tres capas** bien definidas.
Cada capa tiene una responsabilidad clara y se comunica con la capa de abajo.

```
┌─────────────────────────────────────┐
│          Controller  (@RestController)│  ← Recibe peticiones, envía respuestas
├─────────────────────────────────────┤
│          Service     (@Service)      │  ← Contiene la lógica de negocio
├─────────────────────────────────────┤
│          Repository  (@Repository)   │  ← Habla con la base de datos
└─────────────────────────────────────┘
```

### Analogía 🐾

```
┌─────────────────────────────────────┐
│  Recepcionista (Controller)         │  ← Atiende al cliente, toma el pedido
├─────────────────────────────────────┤
│  Manual de operaciones (Service)    │  ← Define cómo se hace cada cosa
├─────────────────────────────────────┤
│  Archivero (Repository)             │  ← Busca y guarda en el libro de registros
└─────────────────────────────────────┘
```

### ¿Por qué separar en capas?

- Si cambias la base de datos de MongoDB a otra, solo tocas el Repository
- Si cambias la lógica de negocio, solo tocas el Service
- Si cambias la API (las rutas), solo tocas el Controller
- Las capas no se mezclan: el Controller nunca habla directamente con el Repository

---

## 💡 Tu primer Controller

Un **Controller** es la clase que recibe las peticiones HTTP y devuelve respuestas.

```java
package com.michicafe.controller;

// Importaciones: le decimos a Java qué clases necesitamos
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController  // Le dice a Spring: "esta clase maneja peticiones HTTP"
public class MichiController {

    // Cuando alguien haga GET a localhost:8080/ se ejecuta este método
    @GetMapping("/")
    public String bienvenida() {
        return "¡Bienvenido al Michi Café! 🐱☕";
    }

    // Cuando alguien haga GET a localhost:8080/estado se ejecuta este método
    @GetMapping("/estado")
    public String estado() {
        return "El Michi Café está ABIERTO 🐾";
    }
}
```

Si abres el navegador y vas a `http://localhost:8080/` verás:
```
¡Bienvenido al Michi Café! 🐱☕
```

---

## 💡 ¿Qué es JSON?

Nuestro servidor no solo devuelve texto. Devuelve datos estructurados en formato **JSON**.

**JSON** (JavaScript Object Notation) es el formato estándar que usan todas las
APIs del mundo para intercambiar datos. Se lee fácilmente y cualquier lenguaje
de programación puede procesarlo.

```json
{
  "id": "64a1b2c3d4e5f6",
  "nombre": "Latte de vainilla",
  "precio": 45.50,
  "categoria": "caliente",
  "disponible": true
}
```

### Las reglas de JSON

| Regla | Ejemplo |
|-------|---------|
| Los textos van entre comillas dobles | `"nombre": "Latte"` |
| Los números van sin comillas | `"precio": 45.50` |
| Los booleanos son `true` o `false` | `"disponible": true` |
| Las listas van entre corchetes | `"tags": ["café", "caliente"]` |
| Los objetos van entre llaves | `{ "nombre": "Sofía" }` |

### Spring Boot convierte automáticamente

Cuando un método del Controller devuelve un objeto Java, Spring Boot lo convierte
automáticamente a JSON. No tienes que hacer nada extra.

```java
// Devuelves un objeto Java...
@GetMapping("/bebida")
public Bebida obtenerBebida() {
    return new Bebida("Latte de vainilla", 45.50, "caliente");
}

// ...y Spring lo convierte automáticamente a JSON:
// { "nombre": "Latte de vainilla", "precio": 45.50, "categoria": "caliente" }
```

---

## 💡 El archivo `application.properties`

Este archivo controla el comportamiento del servidor. Usa el formato `clave=valor`.

```properties
# Puerto del servidor
server.port=8081

# Nombre de la aplicación (aparece en los logs)
spring.application.name=servicio-bebidas

# Conexión a MongoDB
spring.data.mongodb.host=localhost
spring.data.mongodb.port=27017
spring.data.mongodb.database=michicafe_bebidas

# Nivel de detalle de los logs (DEBUG muestra mucho, INFO muestra lo esencial)
logging.level.com.michicafe=INFO
```

Los comentarios en este archivo empiezan con `#`.

---

## 💡 Inyección de dependencias: la magia del `@Autowired`

Este es uno de los conceptos más importantes de Spring Boot.

Normalmente en Java, si una clase necesita usar otra, la creas tú:

```java
// Sin Spring: tienes que crear el objeto manualmente
BebidaService bebidaService = new BebidaService();
```

Con Spring, tú declaras que la necesitas y Spring la crea y la inyecta:

```java
// Con Spring: Spring crea el objeto y lo inyecta aquí automáticamente
@Autowired
private BebidaService bebidaService;
```

### Analogía 🐾

Es como si el Michi Café ya viniera con todos los empleados asignados.
No tienes que contratar a nadie: el sistema detecta que necesitas un barista
y lo asigna automáticamente a tu turno.

### ¿Por qué es mejor?

- Spring gestiona el ciclo de vida de los objetos (los crea y los destruye cuando ya no se necesitan)
- Hace el código más fácil de probar y de cambiar
- Evita crear múltiples instancias del mismo objeto innecesariamente

---

## 💡 ResponseEntity: controlar exactamente la respuesta

`ResponseEntity` es una clase de Spring que nos permite controlar:
- Los **datos** que devolvemos
- El **código de estado HTTP** (200 OK, 404 Not Found, etc.)
- Los **headers** (metadatos de la respuesta)

```java
import org.springframework.http.ResponseEntity;
import org.springframework.http.HttpStatus;

// Devolver 200 OK con datos
return ResponseEntity.ok(bebida);

// Devolver 201 Created con los datos del nuevo recurso
return ResponseEntity.status(HttpStatus.CREATED).body(nuevaBebida);

// Devolver 404 Not Found sin datos
return ResponseEntity.notFound().build();

// Devolver 204 No Content (éxito pero sin datos que devolver)
return ResponseEntity.noContent().build();
```

---

## 🧪 Ejercicios de comprensión — Módulo 04

1. ¿Por qué usamos Spring Boot en lugar de Java puro para crear el servidor?

2. ¿Cuál es la diferencia entre el Controller, el Service y el Repository?
   Descríbelos con la analogía del Michi Café.

3. Si quisieras crear un endpoint que responda a `GET /menu`, ¿qué anotaciones usarías?

4. ¿Qué significa que Spring Boot "convierte automáticamente a JSON"?
   ¿Qué ventaja tiene eso?

---

## ✅ Resumen del Módulo 04

| Concepto | ¿Qué es? | Analogía del Michi Café |
|----------|----------|------------------------|
| Servidor | Programa que escucha y responde peticiones | El gatito que contesta el teléfono |
| Spring Boot 4.1.0 | Framework para crear servidores con Java | La franquicia lista para abrir |
| Maven / `pom.xml` | Gestor de dependencias | Lista de compras del café |
| Puerto | Puerta de comunicación del servidor | La puerta de entrada del café |
| `@RestController` | Clase que maneja peticiones HTTP | El recepcionista |
| `@GetMapping` | Responde a peticiones GET | "Cuando pregunten X, di Y" |
| JSON | Formato de datos estándar | La ficha del producto |
| `@Autowired` | Spring inyecta la dependencia | El empleado ya viene asignado |
| `ResponseEntity` | Control total de la respuesta | El gatito decide qué decir y cómo |
| `application.properties` | Configuración del servidor | El panel de control del café |

---

## ➡️ Siguiente módulo

En el **[Módulo 05 — APIs REST](modulo-05-apis-rest.md)** aprenderemos las operaciones REST completas: GET, POST, PUT, PATCH y DELETE.

> 🐾 "Un servidor bien configurado es como un café bien organizado: siempre listo para atender."
