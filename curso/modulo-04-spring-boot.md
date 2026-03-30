# Módulo 04 — Spring Boot: Tu Primer Servidor
## 🐱 Analogía: Abriendo el Michi Café al público

---

## 🏠 Introducción

Hasta ahora hemos aprendido Java y POO. Nuestro código corre en nuestra computadora y nadie más puede usarlo.

Es como tener el Michi Café completamente equipado, con recetas, empleados y menú... pero con la puerta cerrada. Nadie puede entrar.

**Spring Boot** es lo que nos permite abrir esa puerta al mundo. Con Spring Boot vamos a crear un **servidor**: un programa que escucha peticiones de internet y responde a ellas.

---

## 💡 ¿Qué es un servidor?

Un servidor es un programa que está siempre encendido, esperando que alguien le haga una petición, y cuando llega, responde.

### Analogía 🐾

El Michi Café tiene un número de teléfono para pedidos a domicilio. El gatito que contesta el teléfono está siempre disponible:

1. Suena el teléfono (llega una petición)
2. El gatito contesta (el servidor recibe la petición)
3. El gatito pregunta qué quieren (el servidor lee los datos)
4. El gatito procesa el pedido (el servidor ejecuta la lógica)
5. El gatito confirma el pedido (el servidor envía la respuesta)

Ese gatito que contesta el teléfono es tu servidor Spring Boot.

---

## 💡 ¿Qué es Spring Boot exactamente?

**Spring** es un framework (conjunto de herramientas) para Java que facilita crear aplicaciones empresariales.

**Spring Boot** es una versión de Spring que viene preconfigurada. En lugar de configurar todo desde cero, Spring Boot ya tiene valores por defecto inteligentes.

### Analogía 🐾

- **Java solo** = Construir el café desde los cimientos: poner ladrillos, instalar tuberías, cablear electricidad...
- **Spring** = Comprar un local ya construido, pero tienes que conectar todo tú mismo
- **Spring Boot** = Comprar una franquicia del Michi Café: ya viene con el diseño, los equipos y el manual de operaciones. Tú solo pones tu menú y abres.

---

## 🛠️ Creando tu primer proyecto Spring Boot

### Paso 1: Usar Spring Initializr

Spring Initializr es una página web que genera el esqueleto del proyecto por ti.

1. Ve a: **https://start.spring.io**
2. Configura lo siguiente:

| Campo | Valor |
|-------|-------|
| Project | Maven |
| Language | Java |
| Spring Boot | 3.2.x (la más reciente estable) |
| Group | `com.michicafe` |
| Artifact | `michicafe` |
| Packaging | Jar |
| Java | 17 |

3. En **Dependencies** (dependencias), agrega:
   - `Spring Web` (para crear APIs)

4. Haz clic en **GENERATE**
5. Descarga el ZIP y descomprímelo
6. Ábrelo en IntelliJ IDEA

### Paso 2: Estructura del proyecto

Cuando abras el proyecto verás esta estructura:

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
│       └── java/...
├── pom.xml                                       ← Lista de ingredientes
└── ...
```

### ¿Qué es el `pom.xml`?

El `pom.xml` es como la lista de compras del café. Le dice a Maven (el gestor de dependencias) qué "ingredientes" (librerías) necesita descargar para que el proyecto funcione.

```xml
<!-- pom.xml simplificado -->
<dependencies>
    <!-- Spring Web: para crear APIs REST -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

### Paso 3: El archivo principal

```java
// MichicafeApplication.java
package com.michicafe;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication  // Esta anotación activa toda la magia de Spring Boot
public class MichicafeApplication {

    public static void main(String[] args) {
        SpringApplication.run(MichicafeApplication.class, args);
        // Esto arranca el servidor, como encender el letrero de "ABIERTO"
    }
}
```

Cuando ejecutes este archivo, verás en la consola algo como:

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
...
Started MichicafeApplication in 2.345 seconds
Tomcat started on port(s): 8080
```

¡Tu servidor está corriendo en el puerto 8080! 🎉

---

## 💡 ¿Qué es un puerto?

Un puerto es como la puerta específica de un edificio. Tu computadora tiene miles de puertas (puertos), y cada servicio usa una diferente.

### Analogía 🐾

El edificio del Michi Café tiene varias puertas:
- Puerta 8080: entrada de clientes (tu API)
- Puerta 27017: entrada de proveedores (MongoDB)
- Puerta 443: entrada VIP con seguridad (HTTPS)

Cuando alguien quiere hablar con tu servidor, va a `localhost:8080` (tu computadora, puerta 8080).

---

## 💡 Anotaciones: las instrucciones mágicas de Spring

Las **anotaciones** en Spring Boot son instrucciones especiales que empiezan con `@`. Le dicen a Spring cómo debe tratar cada clase o método.

### Analogía 🐾

Son como las etiquetas que pones en las cajas del café:
- `@Cocina` → esta caja va a la cocina
- `@Caja` → esta caja va a la caja registradora
- `@Almacén` → esta caja va al almacén

Spring lee esas etiquetas y sabe exactamente qué hacer con cada cosa.

| Anotación | ¿Qué hace? | Analogía |
|-----------|-----------|----------|
| `@SpringBootApplication` | Activa todo Spring Boot | El interruptor principal del café |
| `@RestController` | Esta clase maneja peticiones HTTP | El gatito que contesta el teléfono |
| `@GetMapping` | Responde a peticiones GET | "Cuando pregunten por el menú, di esto" |
| `@PostMapping` | Responde a peticiones POST | "Cuando llegue un pedido nuevo, haz esto" |
| `@Service` | Clase con lógica de negocio | El manual de operaciones del café |
| `@Repository` | Clase que habla con la base de datos | El archivero de registros |
| `@Autowired` | Spring inyecta la dependencia automáticamente | El café ya viene con el equipo instalado |

---

## 💡 Tu primer Controller: el gatito que contesta

Un **Controller** es la clase que recibe las peticiones HTTP y devuelve respuestas.

### Analogía 🐾

Es el gatito recepcionista del Michi Café. Cuando alguien llama o llega, él es el primero en atender.

```java
// src/main/java/com/michicafe/controller/MichiController.java
package com.michicafe.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController  // Le dice a Spring: "esta clase maneja peticiones HTTP"
public class MichiController {

    @GetMapping("/")  // Cuando alguien vaya a localhost:8080/, ejecuta este método
    public String bienvenida() {
        return "¡Bienvenido al Michi Café! 🐱☕";
    }

    @GetMapping("/estado")  // localhost:8080/estado
    public String estado() {
        return "El Michi Café está ABIERTO y listo para servirte 🐾";
    }
}
```

Ahora si abres tu navegador y vas a `http://localhost:8080/` verás:
```
¡Bienvenido al Michi Café! 🐱☕
```

---

## 💡 Devolviendo objetos: el menú en formato JSON

Hasta ahora devolvemos texto simple. Pero en el mundo real, los servidores devuelven datos en formato **JSON**.

### ¿Qué es JSON?

JSON (JavaScript Object Notation) es un formato estándar para intercambiar datos. Se parece mucho a los objetos de Java pero en texto.

```json
{
  "nombre": "Latte de vainilla",
  "precio": 45.50,
  "categoria": "caliente",
  "disponible": true
}
```

### Analogía 🐾

JSON es como la ficha de un producto en el menú digital del Michi Café. Tiene campos con nombre y valor, y cualquier sistema puede leerla (una app móvil, un navegador, otro servidor).

### Creando el modelo Bebida

```java
// src/main/java/com/michicafe/model/Bebida.java
package com.michicafe.model;

public class Bebida {
    private String nombre;
    private double precio;
    private String categoria;
    private boolean disponible;

    // Constructor
    public Bebida(String nombre, double precio, String categoria, boolean disponible) {
        this.nombre = nombre;
        this.precio = precio;
        this.categoria = categoria;
        this.disponible = disponible;
    }

    // Getters (Spring Boot los necesita para convertir a JSON)
    public String getNombre()    { return nombre; }
    public double getPrecio()    { return precio; }
    public String getCategoria() { return categoria; }
    public boolean isDisponible(){ return disponible; }
}
```

### Controller que devuelve objetos

```java
// src/main/java/com/michicafe/controller/BebidaController.java
package com.michicafe.controller;

import com.michicafe.model.Bebida;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Arrays;
import java.util.List;

@RestController
@RequestMapping("/bebidas")  // Todas las rutas de este controller empiezan con /bebidas
public class BebidaController {

    // GET localhost:8080/bebidas
    @GetMapping
    public List<Bebida> obtenerMenu() {
        // Por ahora usamos datos de prueba (en módulos siguientes usaremos MongoDB)
        List<Bebida> menu = Arrays.asList(
            new Bebida("Espresso",          35.00, "caliente", true),
            new Bebida("Latte de vainilla", 45.50, "caliente", true),
            new Bebida("Cappuccino",        48.00, "caliente", true),
            new Bebida("Matcha Latte",      55.00, "caliente", true),
            new Bebida("Cold Brew",         50.00, "frio",     true),
            new Bebida("Chai Latte",        52.00, "caliente", false)
        );
        return menu;
    }
}
```

Spring Boot convierte automáticamente la lista de objetos a JSON. Si vas a `http://localhost:8080/bebidas` verás:

```json
[
  { "nombre": "Espresso",          "precio": 35.0,  "categoria": "caliente", "disponible": true  },
  { "nombre": "Latte de vainilla", "precio": 45.5,  "categoria": "caliente", "disponible": true  },
  { "nombre": "Cappuccino",        "precio": 48.0,  "categoria": "caliente", "disponible": true  },
  { "nombre": "Matcha Latte",      "precio": 55.0,  "categoria": "caliente", "disponible": true  },
  { "nombre": "Cold Brew",         "precio": 50.0,  "categoria": "frio",     "disponible": true  },
  { "nombre": "Chai Latte",        "precio": 52.0,  "categoria": "caliente", "disponible": false }
]
```

---

## 💡 Parámetros en la URL: buscar en el menú

Podemos recibir información del cliente a través de la URL.

### Path Variables: parte de la ruta

```java
import org.springframework.web.bind.annotation.PathVariable;

// GET localhost:8080/bebidas/Espresso
@GetMapping("/{nombre}")
public String buscarBebida(@PathVariable String nombre) {
    return "Buscando la bebida: " + nombre + " en el menú del Michi Café";
}
```

### Request Params: parámetros opcionales

```java
import org.springframework.web.bind.annotation.RequestParam;

// GET localhost:8080/bebidas/filtrar?categoria=caliente
@GetMapping("/filtrar")
public String filtrarPorCategoria(@RequestParam String categoria) {
    return "Mostrando bebidas de categoría: " + categoria;
}

// Con valor por defecto (el parámetro es opcional)
// GET localhost:8080/bebidas/filtrar  → muestra todas
// GET localhost:8080/bebidas/filtrar?categoria=frio  → solo frías
@GetMapping("/filtrar")
public String filtrarPorCategoria(
        @RequestParam(defaultValue = "todas") String categoria) {
    return "Categoría seleccionada: " + categoria;
}
```

---

## 💡 La capa de Servicio: el manual de operaciones

En Spring Boot es buena práctica separar el código en capas:

```
Controller  →  recibe la petición y devuelve la respuesta
    ↓
Service     →  contiene la lógica de negocio
    ↓
Repository  →  habla con la base de datos
```

### Analogía 🐾

```
Recepcionista (Controller)  →  recibe al cliente y toma el pedido
        ↓
Manual de operaciones (Service)  →  define cómo se prepara cada cosa
        ↓
Archivero (Repository)  →  guarda y consulta los registros
```

### Creando el Service

```java
// src/main/java/com/michicafe/service/BebidaService.java
package com.michicafe.service;

import com.michicafe.model.Bebida;
import org.springframework.stereotype.Service;

import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

@Service  // Le dice a Spring: "esta clase tiene lógica de negocio"
public class BebidaService {

    // Datos de prueba (luego vendrán de MongoDB)
    private List<Bebida> menu = Arrays.asList(
        new Bebida("Espresso",          35.00, "caliente", true),
        new Bebida("Latte de vainilla", 45.50, "caliente", true),
        new Bebida("Cappuccino",        48.00, "caliente", true),
        new Bebida("Matcha Latte",      55.00, "caliente", true),
        new Bebida("Cold Brew",         50.00, "frio",     true),
        new Bebida("Chai Latte",        52.00, "caliente", false)
    );

    public List<Bebida> obtenerTodas() {
        return menu;
    }

    public List<Bebida> obtenerPorCategoria(String categoria) {
        return menu.stream()
                   .filter(b -> b.getCategoria().equalsIgnoreCase(categoria))
                   .collect(Collectors.toList());
    }

    public List<Bebida> obtenerDisponibles() {
        return menu.stream()
                   .filter(Bebida::isDisponible)
                   .collect(Collectors.toList());
    }
}
```

### Actualizando el Controller para usar el Service

```java
// BebidaController.java actualizado
package com.michicafe.controller;

import com.michicafe.model.Bebida;
import com.michicafe.service.BebidaService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/bebidas")
public class BebidaController {

    @Autowired  // Spring inyecta automáticamente el servicio aquí
    private BebidaService bebidaService;

    // GET /bebidas → todas las bebidas
    @GetMapping
    public List<Bebida> obtenerTodas() {
        return bebidaService.obtenerTodas();
    }

    // GET /bebidas/disponibles → solo las disponibles
    @GetMapping("/disponibles")
    public List<Bebida> obtenerDisponibles() {
        return bebidaService.obtenerDisponibles();
    }

    // GET /bebidas/categoria/caliente → por categoría
    @GetMapping("/categoria/{categoria}")
    public List<Bebida> obtenerPorCategoria(@PathVariable String categoria) {
        return bebidaService.obtenerPorCategoria(categoria);
    }
}
```

---

## 💡 Configuración: el reglamento del café

El archivo `application.properties` es donde configuramos el comportamiento del servidor.

```properties
# src/main/resources/application.properties

# Puerto del servidor (por defecto es 8080)
server.port=8080

# Nombre de la aplicación
spring.application.name=michi-cafe

# Mostrar logs detallados (útil para aprender)
logging.level.com.michicafe=DEBUG
```

---

## 💡 Probando con Postman

**Postman** es una herramienta que nos permite hacer peticiones HTTP sin necesidad de un navegador o una app. Es como el inspector de calidad del Michi Café: prueba que todo funcione antes de abrir al público.

### Cómo hacer tu primera petición

1. Abre Postman
2. Crea una nueva petición (New → HTTP Request)
3. Selecciona el método `GET`
4. Escribe la URL: `http://localhost:8080/bebidas`
5. Haz clic en **Send**

Deberías ver la lista de bebidas en formato JSON en la parte inferior.

### Rutas disponibles en nuestro servidor

| Método | URL | ¿Qué hace? |
|--------|-----|-----------|
| GET | `localhost:8080/` | Mensaje de bienvenida |
| GET | `localhost:8080/estado` | Estado del café |
| GET | `localhost:8080/bebidas` | Todas las bebidas |
| GET | `localhost:8080/bebidas/disponibles` | Solo las disponibles |
| GET | `localhost:8080/bebidas/categoria/caliente` | Bebidas calientes |
| GET | `localhost:8080/bebidas/categoria/frio` | Bebidas frías |

---

## 💡 Estructura final del proyecto

Al terminar este módulo, tu proyecto debe verse así:

```
michicafe/
└── src/main/java/com/michicafe/
    ├── MichicafeApplication.java       ← Punto de entrada
    ├── controller/
    │   ├── MichiController.java        ← Rutas generales
    │   └── BebidaController.java       ← Rutas de bebidas
    ├── service/
    │   └── BebidaService.java          ← Lógica de negocio
    └── model/
        └── Bebida.java                 ← Modelo de datos
```

### Analogía de la estructura 🐾

```
MichicafeApplication  →  El interruptor que enciende todo el café
controller/           →  La recepción (primer contacto con el cliente)
service/              →  La cocina y el manual de operaciones
model/                →  Las fichas de los productos y clientes
```

---

## 🧪 Ejercicios del Módulo 4

### Ejercicio 1: El endpoint de bienvenida personalizado

Crea un endpoint `GET /bienvenida/{nombre}` que reciba el nombre del cliente y devuelva:
```
¡Hola [nombre]! Bienvenido al Michi Café 🐱☕
```

### Ejercicio 2: El modelo Cliente

Crea la clase `Cliente` en el paquete `model` con los atributos del módulo anterior (nombre, correo, puntosFidelidad).

Crea un `ClienteService` con datos de prueba y un `ClienteController` con estos endpoints:
- `GET /clientes` → lista todos los clientes
- `GET /clientes/{nombre}` → busca un cliente por nombre

### Ejercicio 3: Filtro de precio

Agrega al `BebidaService` un método `obtenerPorPrecioMaximo(double precioMax)` que devuelva las bebidas con precio menor o igual al máximo indicado.

Expónlo en el controller como:
`GET /bebidas/precio?max=50`

---

## ✅ Resumen del Módulo 4

| Concepto | ¿Qué es? | Analogía del Michi Café |
|----------|----------|------------------------|
| Servidor | Programa que escucha y responde peticiones | El gatito que contesta el teléfono |
| Spring Boot | Framework que facilita crear servidores Java | La franquicia lista para abrir |
| Puerto | Puerta específica de comunicación | La puerta de entrada del café |
| `@RestController` | Clase que maneja peticiones HTTP | El recepcionista |
| `@GetMapping` | Responde a peticiones GET | "Cuando pregunten esto, di aquello" |
| JSON | Formato estándar de datos | La ficha del producto en el menú digital |
| `@Service` | Clase con lógica de negocio | El manual de operaciones |
| `@Autowired` | Spring conecta las clases automáticamente | El café ya viene con el equipo instalado |
| Postman | Herramienta para probar APIs | El inspector de calidad |

---

## ➡️ Siguiente Módulo

En el **Módulo 05** aprenderemos a crear una **API REST completa**: no solo GET, sino también POST, PUT y DELETE. El Michi Café podrá recibir pedidos nuevos, actualizarlos y cancelarlos.

> 🐾 "Un servidor bien configurado es como un café bien organizado: siempre listo para atender."
