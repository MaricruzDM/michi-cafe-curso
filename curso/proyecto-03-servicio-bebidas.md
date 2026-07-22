# Proyecto P03 — Servicio de Bebidas
## Construyendo el catálogo del Michi Café

> 📚 **Referencias teóricas — lee esto antes o consúltalo cuando necesites:**
> - [Módulo 05 — APIs REST](modulo-05-apis-rest.md) → GET, POST, PUT, PATCH, DELETE, ResponseEntity
> - [Módulo 06 — MongoDB](modulo-06-mongodb.md) → documentos, colecciones, el campo `_id`
> - [Módulo 07 — Spring + MongoDB](modulo-07-spring-mongodb.md) → `@Document`, `@Id`, Repository, DataLoader

---

## 🏠 ¿Qué vamos a construir?

En este proyecto construirás el **Servicio de Bebidas** del Michi Café: un microservicio
Spring Boot completo con base de datos MongoDB real. Al terminar tendrás una API REST
funcional en el puerto **8081** que guarda y gestiona todas las bebidas del menú.

**Lo que construirás:**
- Un proyecto Spring Boot 4.1.0 con Java 17
- Un modelo `Bebida` guardado en MongoDB
- Un `BebidaRepository` que habla con la base de datos
- Un `BebidaService` con toda la lógica de negocio
- Un `BebidaController` con endpoints GET, POST, PUT, PATCH y DELETE
- Un `DataLoader` que carga 6 bebidas de ejemplo al arrancar
- Pruebas completas con Postman

**Tiempo estimado:** 60-90 minutos

---

## 🛠️ Paso 1 — Crear el proyecto en Spring Initializr

### 1.1 — Abrir Spring Initializr

1. Abre tu navegador (Chrome, Firefox o Edge)
2. Ve a la dirección: **https://start.spring.io**
3. Verás una página con un formulario de configuración

### 1.2 — Configurar el proyecto

Rellena cada campo **exactamente** como se indica:

**Sección izquierda:**

| Campo | Valor |
|-------|-------|
| Project | `Maven` |
| Language | `Java` |
| Spring Boot | `4.1.0` |

> ⚠️ Asegúrate de seleccionar exactamente `4.1.0`. Evita versiones con `(SNAPSHOT)` o `(M1)` — son versiones inestables.

**Sección "Project Metadata":**

| Campo | Valor |
|-------|-------|
| Group | `com.michicafe` |
| Artifact | `servicio-bebidas` |
| Name | `servicio-bebidas` |
| Description | `Michi Café - Servicio de gestión de bebidas` |
| Package name | se llena solo: `com.michicafe.servicebebidas` |
| Packaging | `Jar` |
| Java | `17` |



### 1.3
### 1.3 — Agregar las dependencias

Las dependencias son las librerías que tu proyecto necesita. Vamos a agregar cuatro, una por una.

**Dependencia 1 — Spring Web:**
1. Haz clic en el botón azul **"ADD DEPENDENCIES"** (esquina superior derecha)
2. Se abre un buscador. Escribe: `web`
3. Verás `Spring Web` en la lista — haz clic en él
4. El buscador se cierra y verás "Spring Web" en el panel derecho bajo "Dependencies"

**Dependencia 2 — Spring Data MongoDB:**
1. Haz clic de nuevo en **"ADD DEPENDENCIES"**
2. Escribe: `mongodb`
3. Verás `Spring Data MongoDB` — haz clic en él

**Dependencia 3 — Validation:**
1. Haz clic de nuevo en **"ADD DEPENDENCIES"**
2. Escribe: `valid`
3. Verás `Validation` — haz clic en él

**Dependencia 4 — Spring Boot Actuator:**
1. Haz clic de nuevo en **"ADD DEPENDENCIES"**
2. Escribe: `actuator`
3. Verás `Spring Boot Actuator` — haz clic en él

Al terminar, el panel derecho debe mostrar estas cuatro dependencias:
- ✅ Spring Web
- ✅ Spring Data MongoDB
- ✅ Validation
- ✅ Spring Boot Actuator

### 1.4 — Generar y descargar el proyecto

1. Haz clic en el botón verde **"GENERATE"** (abajo al centro)
2. Se descargará un archivo llamado `servicio-bebidas.zip` en tu carpeta de Descargas

### 1.5 — Descomprimir el proyecto

1. Abre tu carpeta de **Descargas**
2. Haz clic derecho sobre `servicio-bebidas.zip`
3. Selecciona **"Extraer todo..."**
4. Haz clic en **Examinar** y navega hasta `C:\Proyectos`
5. Si la carpeta `Proyectos` no existe, créala haciendo clic en **Nueva carpeta**
6. Selecciona `C:\Proyectos` y haz clic en **Aceptar**
7. Haz clic en **Extraer**
8. Quedará la carpeta: `C:\Proyectos\servicio-bebidas`

### 1.6 — Abrir en IntelliJ IDEA

1. Abre **IntelliJ IDEA**
2. En la pantalla de bienvenida, haz clic en **"Open"**
3. Navega hasta `C:\Proyectos\servicio-bebidas`
4. Haz clic **una vez** en la carpeta `servicio-bebidas` para seleccionarla
5. Haz clic en **"OK"**
6. Si aparece un diálogo que dice "Trust and Open Project?", haz clic en **"Trust Project"**

### 1.7 — Esperar la descarga de dependencias

La primera vez que abres el proyecto, IntelliJ descarga todas las dependencias de Maven.
Esto puede tardar **2 a 5 minutos**.

Sabrás que terminó cuando:
- La barra de progreso en la parte inferior de la pantalla desaparece
- Ya no hay mensajes de "Downloading..." en la esquina inferior derecha

> ⚠️ No cierres IntelliJ mientras descarga. Si la conexión es lenta, ten paciencia.



---

## 🛠️ Paso 2 — Configurar application.properties

El archivo `application.properties` le dice a Spring Boot cómo debe comportarse y dónde está la base de datos.

### 2.1 — Abrir el archivo

1. En el panel izquierdo (**Project**), expande las carpetas haciendo clic en la flecha:
   `src` → `main` → `resources`
2. Haz **doble clic** en `application.properties` para abrirlo
3. El archivo probablemente está vacío o tiene solo una línea

### 2.2 — Escribir la configuración

Borra cualquier contenido existente y escribe exactamente esto:

```properties
server.port=8081
spring.application.name=servicio-bebidas
spring.data.mongodb.host=localhost
spring.data.mongodb.port=27017
spring.data.mongodb.database=michicafe_bebidas
logging.level.com.michicafe=INFO
```

### 2.3 — ¿Qué hace cada línea?

| Línea | Explicación |
|-------|-------------|
| `server.port=8081` | El servidor arrancará en el puerto 8081 (no el 8080 por defecto, porque luego tendremos varios servicios corriendo a la vez) |
| `spring.application.name=servicio-bebidas` | El nombre de este servicio (aparece en los logs) |
| `spring.data.mongodb.host=localhost` | MongoDB está en tu misma computadora |
| `spring.data.mongodb.port=27017` | El puerto estándar de MongoDB |
| `spring.data.mongodb.database=michicafe_bebidas` | El nombre de la base de datos que Spring usará (si no existe, MongoDB la crea automáticamente) |
| `logging.level.com.michicafe=INFO` | Muestra mensajes de nivel INFO en la consola para las clases de nuestro paquete |

> 💡 Guarda el archivo con **Ctrl + S** después de escribirlo.

---

## 🛠️ Paso 3 — Crear el modelo Bebida

El modelo es la clase Java que representa una bebida. Cada objeto `Bebida` se guardará como un documento en MongoDB.

> 📚 Si quieres entender qué es `@Document` y `@Id`, consulta:
> [Módulo 07 — Anotar el modelo con @Document](modulo-07-spring-mongodb.md#paso-3-anotar-el-modelo-con-document)

### 3.1 — Crear el paquete `model`

Los paquetes son carpetas que organizan las clases por su función.

1. En el panel izquierdo, expande: `src` → `main` → `java` → `com.michicafe.servicebebidas`
2. Haz **clic derecho** sobre `com.michicafe.servicebebidas`
3. Pasa el mouse sobre **"New"**
4. En el submenú, haz clic en **"Package"**
5. En el campo de texto que aparece, escribe exactamente: `model`
6. Presiona **Enter**

Verás que se creó una nueva carpeta `model` dentro de `com.michicafe.servicebebidas`.

> 💡 El nombre completo del paquete queda: `com.michicafe.servicebebidas.model`

### 3.2 — Crear la clase `Bebida`

1. Haz **clic derecho** sobre el paquete `model` que acabas de crear
2. Pasa el mouse sobre **"New"**
3. Haz clic en **"Java Class"**
4. En el campo de texto, escribe exactamente: `Bebida`
5. Asegúrate de que esté seleccionada la opción **"Class"** (no Interface, Enum ni Record)
6. Presiona **Enter**

Se abrirá un archivo con este contenido inicial:

```java
package com.michicafe.servicebebidas.model;

public class Bebida {
}
```

### 3.3 — Escribir el código de la clase Bebida

Reemplaza todo el contenido del archivo con el siguiente código:

```java
package com.michicafe.servicebebidas.model;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document(collection = "bebidas")
public class Bebida {

    @Id
    private String id;
    private String nombre;
    private double precio;
    private String categoria;
    private String descripcion;
    private boolean disponible;

    // Constructor vacío — requerido por Spring Data MongoDB
    public Bebida() {}

    // Constructor con datos (sin id, MongoDB lo genera automáticamente)
    public Bebida(String nombre, double precio, String categoria,
                  String descripcion, boolean disponible) {
        this.nombre = nombre;
        this.precio = precio;
        this.categoria = categoria;
        this.descripcion = descripcion;
        this.disponible = disponible;
    }

    // Getters
    public String getId()          { return id; }
    public String getNombre()      { return nombre; }
    public double getPrecio()      { return precio; }
    public String getCategoria()   { return categoria; }
    public String getDescripcion() { return descripcion; }
    public boolean isDisponible()  { return disponible; }

    // Setters
    public void setId(String id)                     { this.id = id; }
    public void setNombre(String nombre)             { this.nombre = nombre; }
    public void setPrecio(double precio)             { this.precio = precio; }
    public void setCategoria(String categoria)       { this.categoria = categoria; }
    public void setDescripcion(String descripcion)   { this.descripcion = descripcion; }
    public void setDisponible(boolean disponible)    { this.disponible = disponible; }
}
```

### 3.4 — Importar las anotaciones con Alt+Enter

Cuando escribas `@Document` y `@Id`, IntelliJ los subrayará en rojo porque no sabe de dónde vienen. Necesitas **importarlos**.

**Para importar `@Document`:**
1. Haz clic sobre la palabra `Document` (donde aparece el subrayado rojo)
2. Presiona **Alt + Enter** al mismo tiempo
3. Aparece un menú emergente con la sugerencia `Import class`
4. Haz clic en `Import class`
5. Si aparece más de una opción, elige la que diga `org.springframework.data.mongodb.core.mapping.Document`

**Para importar `@Id`:**
1. Haz clic sobre la palabra `Id` (donde aparece el subrayado rojo)
2. Presiona **Alt + Enter**
3. Haz clic en `Import class`
4. Elige `org.springframework.data.annotation.Id`

Después de importar, verás que IntelliJ agrega estas líneas al inicio del archivo:
```java
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;
```

Y los subrayados rojos desaparecen. ✅

### 3.5 — ¿Qué hace cada parte del código?

| Elemento | Explicación |
|----------|-------------|
| `@Document(collection = "bebidas")` | Le dice a MongoDB que esta clase se guarda en la colección llamada "bebidas" |
| `@Id` | Este campo (`id`) es el identificador único del documento en MongoDB. MongoDB lo genera automáticamente como un string de 24 caracteres |
| `private String id` | El campo id. Spring lo dejará en `null` cuando creamos un objeto nuevo; MongoDB lo rellenará al guardarlo |
| `public Bebida() {}` | Constructor vacío — obligatorio para que Spring pueda crear objetos Bebida al leerlos de MongoDB |
| Getters/Setters | Métodos para leer y modificar cada campo. Spring los necesita para convertir el JSON que llega en objetos Java, y viceversa |

> 💡 Guarda el archivo con **Ctrl + S**.



---

## 🛠️ Paso 4 — Crear el Repository

El Repository es la interfaz que habla directamente con MongoDB. Es quien sabe cómo guardar, buscar y eliminar documentos.

> 📚 Para entender cómo funciona la magia de los nombres de métodos, consulta:
> [Módulo 07 — Crear el Repository](modulo-07-spring-mongodb.md#paso-4-crear-el-repository)

### 4.1 — Crear el paquete `repository`

1. Haz **clic derecho** sobre `com.michicafe.servicebebidas` (el paquete raíz)
2. Pasa el mouse sobre **"New"** → haz clic en **"Package"**
3. Escribe exactamente: `repository`
4. Presiona **Enter**

### 4.2 — Crear la interfaz `BebidaRepository`

> ⚠️ Importante: vamos a crear una **interfaz**, no una clase. Son cosas diferentes en Java.

1. Haz **clic derecho** sobre el paquete `repository`
2. Pasa el mouse sobre **"New"** → haz clic en **"Java Class"**
3. En el campo de texto escribe: `BebidaRepository`
4. **Antes de presionar Enter**, haz clic en la opción **"Interface"** en la lista de abajo (no "Class")
5. Presiona **Enter**

Se abrirá un archivo con este contenido inicial:

```java
package com.michicafe.servicebebidas.repository;

public interface BebidaRepository {
}
```

> 💡 ¿Cuál es la diferencia entre clase e interfaz? Una **interfaz** define *qué* métodos existen pero no *cómo* funcionan. Spring Data lee nuestra interfaz y genera el código de las consultas automáticamente. Nunca tendrás que escribir el código de `findByCategoria` — Spring lo hace por ti.

### 4.3 — Escribir el código del Repository

Reemplaza todo el contenido con:

```java
package com.michicafe.servicebebidas.repository;

import com.michicafe.servicebebidas.model.Bebida;
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public interface BebidaRepository extends MongoRepository<Bebida, String> {

    // Buscar todas las bebidas de una categoría
    List<Bebida> findByCategoria(String categoria);

    // Buscar solo las bebidas que están disponibles
    List<Bebida> findByDisponibleTrue();

    // Buscar bebidas cuyo precio esté entre un mínimo y un máximo
    List<Bebida> findByPrecioBetween(double min, double max);

    // Buscar bebidas cuyo nombre contenga un texto (sin importar mayúsculas)
    List<Bebida> findByNombreContainingIgnoreCase(String texto);
}
```

### 4.4 — Importar con Alt+Enter

Importa cada elemento subrayado en rojo:

- Haz clic sobre `Bebida` → **Alt + Enter** → `Import class` → elige `com.michicafe.servicebebidas.model.Bebida`
- Haz clic sobre `MongoRepository` → **Alt + Enter** → `Import class` → elige `org.springframework.data.mongodb.repository.MongoRepository`
- Haz clic sobre `Repository` → **Alt + Enter** → `Import class` → elige `org.springframework.stereotype.Repository`
- Haz clic sobre `List` → **Alt + Enter** → `Import class` → elige `java.util.List`

### 4.5 — La magia de los nombres de métodos

Spring Data lee el nombre del método y genera la consulta MongoDB automáticamente:

| Nombre del método | Consulta que genera en MongoDB |
|-------------------|-------------------------------|
| `findByCategoria("caliente")` | Busca todos los documentos donde `categoria == "caliente"` |
| `findByDisponibleTrue()` | Busca todos los documentos donde `disponible == true` |
| `findByPrecioBetween(30, 50)` | Busca donde `precio >= 30 AND precio <= 50` |
| `findByNombreContainingIgnoreCase("latte")` | Busca donde el nombre contiene "latte" sin importar si es mayúscula o minúscula |

Y los métodos que ya tienes gratis por extender `MongoRepository`:

| Método | ¿Qué hace? |
|--------|-----------|
| `save(bebida)` | Guarda o actualiza la bebida en MongoDB |
| `findById(id)` | Busca una bebida por su `_id` |
| `findAll()` | Devuelve todas las bebidas |
| `deleteById(id)` | Elimina una bebida por su `_id` |
| `existsById(id)` | Devuelve `true` si la bebida existe |
| `count()` | Cuenta cuántas bebidas hay |

> 💡 Guarda el archivo con **Ctrl + S**.



---

## 🛠️ Paso 5 — Crear el Service

El Service contiene la lógica de negocio. Es quien decide qué se puede hacer y cómo, y habla con el Repository para acceder a la base de datos.

### 5.1 — Crear el paquete `service`

1. Haz **clic derecho** sobre `com.michicafe.servicebebidas`
2. **"New"** → **"Package"** → escribe `service` → **Enter**

### 5.2 — Crear la clase `BebidaService`

1. Haz **clic derecho** sobre el paquete `service`
2. **"New"** → **"Java Class"** → escribe `BebidaService`
3. Asegúrate de que esté seleccionada **"Class"** → **Enter**

### 5.3 — Escribir el código del Service

```java
package com.michicafe.servicebebidas.service;

import com.michicafe.servicebebidas.model.Bebida;
import com.michicafe.servicebebidas.repository.BebidaRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

@Service
public class BebidaService {

    @Autowired
    private BebidaRepository bebidaRepository;

    // ── Guardar (crear o actualizar) ─────────────────────────────────────────
    public Bebida guardar(Bebida bebida) {
        return bebidaRepository.save(bebida);
    }

    // ── Obtener todas ────────────────────────────────────────────────────────
    public List<Bebida> obtenerTodas() {
        return bebidaRepository.findAll();
    }

    // ── Obtener por id ───────────────────────────────────────────────────────
    // Devuelve Optional: puede contener la bebida o estar vacío (si no existe)
    public Optional<Bebida> obtenerPorId(String id) {
        return bebidaRepository.findById(id);
    }

    // ── Obtener por categoría ────────────────────────────────────────────────
    public List<Bebida> obtenerPorCategoria(String categoria) {
        return bebidaRepository.findByCategoria(categoria);
    }

    // ── Obtener solo las disponibles ─────────────────────────────────────────
    public List<Bebida> obtenerDisponibles() {
        return bebidaRepository.findByDisponibleTrue();
    }

    // ── Obtener por rango de precio ──────────────────────────────────────────
    public List<Bebida> obtenerPorRangoPrecio(double min, double max) {
        return bebidaRepository.findByPrecioBetween(min, max);
    }

    // ── Buscar por nombre ────────────────────────────────────────────────────
    public List<Bebida> buscarPorNombre(String texto) {
        return bebidaRepository.findByNombreContainingIgnoreCase(texto);
    }

    // ── Actualizar una bebida completa (PUT) ─────────────────────────────────
    public Optional<Bebida> actualizar(String id, Bebida bebidaActualizada) {
        // Primero verificamos que la bebida existe
        if (!bebidaRepository.existsById(id)) {
            return Optional.empty(); // No existe → devolvemos vacío
        }
        bebidaActualizada.setId(id); // Conservamos el mismo id
        return Optional.of(bebidaRepository.save(bebidaActualizada));
    }

    // ── Cambiar disponibilidad (PATCH) ───────────────────────────────────────
    public Optional<Bebida> cambiarDisponibilidad(String id, boolean disponible) {
        return bebidaRepository.findById(id).map(bebida -> {
            bebida.setDisponible(disponible);
            return bebidaRepository.save(bebida);
        });
    }

    // ── Eliminar ─────────────────────────────────────────────────────────────
    public boolean eliminar(String id) {
        if (!bebidaRepository.existsById(id)) {
            return false; // No existe → no se puede eliminar
        }
        bebidaRepository.deleteById(id);
        return true; // Se eliminó correctamente
    }
}
```

### 5.4 — Importar con Alt+Enter

Importa todo lo que esté subrayado en rojo:
- `Bebida` → `com.michicafe.servicebebidas.model.Bebida`
- `BebidaRepository` → `com.michicafe.servicebebidas.repository.BebidaRepository`
- `Autowired` → `org.springframework.beans.factory.annotation.Autowired`
- `Service` → `org.springframework.stereotype.Service`
- `List` → `java.util.List`
- `Optional` → `java.util.Optional`

### 5.5 — ¿Qué hace cada parte?

| Elemento | Explicación |
|----------|-------------|
| `@Service` | Le dice a Spring que esta clase es un servicio. Spring la crea automáticamente y la deja disponible para que otros la usen |
| `@Autowired` | Le pide a Spring que inyecte automáticamente el `BebidaRepository`. Tú no necesitas hacer `new BebidaRepository()` — Spring lo hace |
| `Optional<Bebida>` | Es un contenedor que puede tener una `Bebida` o estar vacío. Evita los errores de `NullPointerException`. [Ver más en Módulo 05](modulo-05-apis-rest.md) |
| `.map(bebida -> {...})` | Si el `Optional` tiene un valor, ejecuta el código dentro. Si está vacío, no hace nada |



---

## 🛠️ Paso 6 — Crear el Controller

El Controller es quien recibe las peticiones HTTP del mundo exterior y devuelve las respuestas.

> 📚 Para repasar GET, POST, PUT, PATCH, DELETE y ResponseEntity:
> [Módulo 05 — APIs REST](modulo-05-apis-rest.md)

### 6.1 — Crear el paquete `controller`

1. Haz **clic derecho** sobre `com.michicafe.servicebebidas`
2. **"New"** → **"Package"** → escribe `controller` → **Enter**

### 6.2 — Crear la clase `BebidaController`

1. Haz **clic derecho** sobre el paquete `controller`
2. **"New"** → **"Java Class"** → escribe `BebidaController` → **Enter**

### 6.3 — Escribir el código del Controller

```java
package com.michicafe.servicebebidas.controller;

import com.michicafe.servicebebidas.model.Bebida;
import com.michicafe.servicebebidas.service.BebidaService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/bebidas")
public class BebidaController {

    @Autowired
    private BebidaService bebidaService;

    // ── GET /api/bebidas ─────────────────────────────────────────────────────
    // Devuelve todas las bebidas
    @GetMapping
    public ResponseEntity<List<Bebida>> obtenerTodas() {
        return ResponseEntity.ok(bebidaService.obtenerTodas());
    }

    // ── GET /api/bebidas/{id} ────────────────────────────────────────────────
    // Devuelve una bebida por su id
    @GetMapping("/{id}")
    public ResponseEntity<Bebida> obtenerPorId(@PathVariable String id) {
        return bebidaService.obtenerPorId(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // ── GET /api/bebidas/disponibles ─────────────────────────────────────────
    // Devuelve solo las bebidas disponibles
    @GetMapping("/disponibles")
    public ResponseEntity<List<Bebida>> obtenerDisponibles() {
        return ResponseEntity.ok(bebidaService.obtenerDisponibles());
    }

    // ── GET /api/bebidas/categoria/{categoria} ───────────────────────────────
    // Devuelve las bebidas de una categoría (ej: caliente, frio, especial)
    @GetMapping("/categoria/{categoria}")
    public ResponseEntity<List<Bebida>> obtenerPorCategoria(
            @PathVariable String categoria) {
        return ResponseEntity.ok(bebidaService.obtenerPorCategoria(categoria));
    }

    // ── GET /api/bebidas/precio?min=30&max=60 ────────────────────────────────
    // Devuelve bebidas dentro de un rango de precio
    @GetMapping("/precio")
    public ResponseEntity<List<Bebida>> obtenerPorPrecio(
            @RequestParam(defaultValue = "0") double min,
            @RequestParam(defaultValue = "9999") double max) {
        return ResponseEntity.ok(bebidaService.obtenerPorRangoPrecio(min, max));
    }

    // ── GET /api/bebidas/buscar?nombre=latte ─────────────────────────────────
    // Busca bebidas cuyo nombre contenga el texto indicado
    @GetMapping("/buscar")
    public ResponseEntity<List<Bebida>> buscarPorNombre(
            @RequestParam String nombre) {
        return ResponseEntity.ok(bebidaService.buscarPorNombre(nombre));
    }

    // ── POST /api/bebidas ────────────────────────────────────────────────────
    // Crea una bebida nueva
    @PostMapping
    public ResponseEntity<Bebida> crear(@RequestBody Bebida bebida) {
        Bebida nueva = bebidaService.guardar(bebida);
        return ResponseEntity.status(HttpStatus.CREATED).body(nueva); // 201 Created
    }

    // ── PUT /api/bebidas/{id} ────────────────────────────────────────────────
    // Reemplaza una bebida completa
    @PutMapping("/{id}")
    public ResponseEntity<Bebida> actualizar(
            @PathVariable String id,
            @RequestBody Bebida bebida) {
        return bebidaService.actualizar(id, bebida)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // ── PATCH /api/bebidas/{id}/disponibilidad?valor=false ───────────────────
    // Cambia solo la disponibilidad de una bebida
    @PatchMapping("/{id}/disponibilidad")
    public ResponseEntity<Bebida> cambiarDisponibilidad(
            @PathVariable String id,
            @RequestParam boolean valor) {
        return bebidaService.cambiarDisponibilidad(id, valor)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // ── DELETE /api/bebidas/{id} ─────────────────────────────────────────────
    // Elimina una bebida
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> eliminar(@PathVariable String id) {
        if (bebidaService.eliminar(id)) {
            return ResponseEntity.noContent().build(); // 204 No Content
        }
        return ResponseEntity.notFound().build();       // 404 Not Found
    }
}
```

### 6.4 — Importar con Alt+Enter

Importa todo lo que esté subrayado en rojo usando **Alt + Enter**:
- `Bebida` → `com.michicafe.servicebebidas.model.Bebida`
- `BebidaService` → `com.michicafe.servicebebidas.service.BebidaService`
- `Autowired` → `org.springframework.beans.factory.annotation.Autowired`
- `HttpStatus` → `org.springframework.http.HttpStatus`
- `ResponseEntity` → `org.springframework.http.ResponseEntity`
- Para las anotaciones `@RestController`, `@RequestMapping`, `@GetMapping`, etc. → haz **Alt + Enter** en cada una y elige `org.springframework.web.bind.annotation.*`
- `List` → `java.util.List`

> 💡 Si IntelliJ muestra el aviso `@RestController`, `@GetMapping`, etc. en el mismo import (`org.springframework.web.bind.annotation.*`), puedes importar todos juntos con un solo import usando el asterisco.

### 6.5 — Tabla de endpoints creados

| Método | URL | Descripción | Respuesta |
|--------|-----|-------------|-----------|
| GET | `/api/bebidas` | Todas las bebidas | 200 + lista |
| GET | `/api/bebidas/{id}` | Una bebida por id | 200 o 404 |
| GET | `/api/bebidas/disponibles` | Solo disponibles | 200 + lista |
| GET | `/api/bebidas/categoria/{cat}` | Por categoría | 200 + lista |
| GET | `/api/bebidas/precio?min=30&max=60` | Por rango de precio | 200 + lista |
| GET | `/api/bebidas/buscar?nombre=latte` | Buscar por nombre | 200 + lista |
| POST | `/api/bebidas` | Crear bebida nueva | 201 + bebida creada |
| PUT | `/api/bebidas/{id}` | Reemplazar bebida | 200 o 404 |
| PATCH | `/api/bebidas/{id}/disponibilidad?valor=false` | Cambiar disponibilidad | 200 o 404 |
| DELETE | `/api/bebidas/{id}` | Eliminar bebida | 204 o 404 |



---

## 🛠️ Paso 7 — Crear el DataLoader (datos iniciales)

El DataLoader carga automáticamente algunas bebidas al arrancar el servidor, para que no tengas que crearlas manualmente cada vez.

> 📚 Para entender cómo funciona `CommandLineRunner`, consulta:
> [Módulo 07 — Cargando datos iniciales](modulo-07-spring-mongodb.md#cargando-datos-iniciales)

### 7.1 — Crear el paquete `config`

1. Haz **clic derecho** sobre `com.michicafe.servicebebidas`
2. **"New"** → **"Package"** → escribe `config` → **Enter**

### 7.2 — Crear la clase `DataLoader`

1. Haz **clic derecho** sobre el paquete `config`
2. **"New"** → **"Java Class"** → escribe `DataLoader` → **Enter**

### 7.3 — Escribir el código del DataLoader

```java
package com.michicafe.servicebebidas.config;

import com.michicafe.servicebebidas.model.Bebida;
import com.michicafe.servicebebidas.repository.BebidaRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component
public class DataLoader implements CommandLineRunner {

    @Autowired
    private BebidaRepository bebidaRepository;

    @Override
    public void run(String... args) {
        // Solo cargamos datos si la colección está vacía
        // Así evitamos duplicar datos al reiniciar el servidor
        if (bebidaRepository.count() == 0) {
            System.out.println("🐱 Cargando menú inicial del Michi Café...");

            bebidaRepository.save(new Bebida(
                "Espresso", 35.00, "caliente",
                "Café concentrado y puro", true));

            bebidaRepository.save(new Bebida(
                "Latte de Vainilla", 45.50, "caliente",
                "Espresso con leche vaporizada y jarabe de vainilla", true));

            bebidaRepository.save(new Bebida(
                "Cappuccino", 48.00, "caliente",
                "Espresso con leche vaporizada y espuma cremosa", true));

            bebidaRepository.save(new Bebida(
                "Matcha Latte", 55.00, "caliente",
                "Té matcha japonés con leche de avena", true));

            bebidaRepository.save(new Bebida(
                "Cold Brew", 50.00, "frio",
                "Café infusionado en frío durante 12 horas", true));

            bebidaRepository.save(new Bebida(
                "Chai Latte", 52.00, "caliente",
                "Té chai especiado con leche y canela", false));

            System.out.println("✅ Menú cargado: "
                + bebidaRepository.count() + " bebidas en la base de datos");
        } else {
            System.out.println("📋 El menú ya tiene "
                + bebidaRepository.count() + " bebidas — no se cargan datos nuevos");
        }
    }
}
```

### 7.4 — Importar con Alt+Enter

- `Bebida` → `com.michicafe.servicebebidas.model.Bebida`
- `BebidaRepository` → `com.michicafe.servicebebidas.repository.BebidaRepository`
- `Autowired` → `org.springframework.beans.factory.annotation.Autowired`
- `CommandLineRunner` → `org.springframework.boot.CommandLineRunner`
- `Component` → `org.springframework.stereotype.Component`

### 7.5 — ¿Qué hace este código?

| Elemento | Explicación |
|----------|-------------|
| `@Component` | Le dice a Spring que cree esta clase y la gestione automáticamente |
| `implements CommandLineRunner` | Indica que esta clase tiene un método `run` que Spring ejecutará al arrancar |
| `if (bebidaRepository.count() == 0)` | Verifica si la colección está vacía. Si ya hay datos (porque el servidor ya arrancó antes), no carga nada |
| `bebidaRepository.save(...)` | Guarda cada bebida en MongoDB |



---

## 🛠️ Paso 8 — Ejecutar y probar

### 8.1 — Verificar que MongoDB está corriendo

Antes de arrancar el servidor, asegúrate de que MongoDB esté activo.

**En Windows:**
1. Presiona **Windows + R** al mismo tiempo
2. En el cuadro que aparece, escribe: `services.msc`
3. Presiona **Enter**
4. Se abrirá la ventana de Servicios de Windows
5. Busca en la lista: **MongoDB Server** (puede aparecer como `MongoDB Server (MongoDB)`)
6. Verifica que en la columna **"Estado"** diga **"En ejecución"**
7. Si no está en ejecución, haz clic derecho sobre él → **"Iniciar"**

> 💡 Si no ves MongoDB Server en la lista, revisa la instalación siguiendo el
> [Proyecto P01 — Instalación del entorno](proyecto-01-instalacion.md).

### 8.2 — Ejecutar la aplicación

1. En el panel izquierdo, expande `src` → `main` → `java` → `com.michicafe.servicebebidas`
2. Haz **doble clic** en `ServicioBedidasApplication` para abrirlo

> 💡 El nombre exacto del archivo es `ServicioBedidasApplication.java` (Spring Initializr quita los guiones del artifact name).

3. Busca el método `main` (la línea que dice `public static void main`)
4. Verás un triángulo verde ▶ a la izquierda de esa línea
5. Haz **clic en ese triángulo verde ▶**
6. Selecciona **"Run 'ServicioBedidasApplication'"**

### 8.3 — Verificar en la consola

En la parte inferior de IntelliJ se abrirá la consola. Espera ver mensajes como:

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
...
 :: Spring Boot ::               (v4.1.0)

🐱 Cargando menú inicial del Michi Café...
✅ Menú cargado: 6 bebidas en la base de datos
...
Started ServicioBedidasApplication in 3.2 seconds (process running for 3.8)
```

La línea más importante: `Started ServicioBedidasApplication in X seconds`

Si en lugar de eso ves un error en rojo, lo más probable es una de estas causas:
- **MongoDB no está corriendo** → regresa al paso 8.1
- **El puerto 8081 ya está en uso** → hay otra aplicación usando ese puerto, ciérrala

✅ Cuando veas el mensaje de inicio, tu servidor está listo.

### 8.4 — Probar con Postman

Abre **Postman**. En cada prueba: elige el método, escribe la URL y haz clic en **"Send"**.

---

**PRUEBA 1 — GET: Ver todas las bebidas**

```
Método: GET
URL:    http://localhost:8081/api/bebidas
```

Respuesta esperada (200 OK):
```json
[
  {
    "id": "64a1b2c3d4e5f6a7b8c9d0e1",
    "nombre": "Espresso",
    "precio": 35.0,
    "categoria": "caliente",
    "descripcion": "Café concentrado y puro",
    "disponible": true
  },
  {
    "id": "64a1b2c3d4e5f6a7b8c9d0e2",
    "nombre": "Latte de Vainilla",
    "precio": 45.5,
    "categoria": "caliente",
    "descripcion": "Espresso con leche vaporizada y jarabe de vainilla",
    "disponible": true
  }
]
```

> 💡 Los `id` que verás serán diferentes a los del ejemplo — MongoDB los genera únicos.

---

**PRUEBA 2 — GET: Ver una bebida por id**

Copia uno de los `id` que aparecieron en la prueba anterior (un string largo como `64a1b2c3d4e5f6a7b8c9d0e1`).

```
Método: GET
URL:    http://localhost:8081/api/bebidas/PEGA_AQUI_EL_ID
```

Respuesta esperada (200 OK): el objeto de esa bebida.
Si el id no existe: respuesta 404.

---

**PRUEBA 3 — POST: Crear una bebida nueva**

1. Método: **POST**
2. URL: `http://localhost:8081/api/bebidas`
3. Haz clic en la pestaña **"Body"** (debajo de la URL)
4. Selecciona la opción **"raw"**
5. En el desplegable que aparece a la derecha (dice "Text"), cámbialo a **"JSON"**
6. En el campo grande de texto, escribe este JSON:

```json
{
  "nombre": "Frappuccino de Caramelo",
  "precio": 65.00,
  "categoria": "frio",
  "descripcion": "Café frío mezclado con caramelo y crema batida",
  "disponible": true
}
```

7. Haz clic en **"Send"**

Respuesta esperada (201 Created):
```json
{
  "id": "64a1b2c3d4e5f6a7b8c9d0e9",
  "nombre": "Frappuccino de Caramelo",
  "precio": 65.0,
  "categoria": "frio",
  "descripcion": "Café frío mezclado con caramelo y crema batida",
  "disponible": true
}
```

---

**PRUEBA 4 — PUT: Actualizar una bebida completa**

Usa el id del Frappuccino que acabas de crear.

```
Método:  PUT
URL:     http://localhost:8081/api/bebidas/PEGA_AQUI_EL_ID
Body → raw → JSON:
```
```json
{
  "nombre": "Frappuccino de Caramelo Grande",
  "precio": 72.00,
  "categoria": "frio",
  "descripcion": "Versión grande del café frío con caramelo y crema batida",
  "disponible": true
}
```

Respuesta esperada (200 OK): la bebida actualizada.

---

**PRUEBA 5 — PATCH: Cambiar disponibilidad**

```
Método: PATCH
URL:    http://localhost:8081/api/bebidas/PEGA_AQUI_EL_ID/disponibilidad?valor=false
```

No necesitas body. Respuesta esperada (200 OK): la bebida con `disponible: false`.

---

**PRUEBA 6 — GET: Ver solo disponibles**

```
Método: GET
URL:    http://localhost:8081/api/bebidas/disponibles
```

Verás que el Frappuccino ya no aparece (lo marcamos como no disponible).

---

**PRUEBA 7 — DELETE: Eliminar una bebida**

```
Método: DELETE
URL:    http://localhost:8081/api/bebidas/PEGA_AQUI_EL_ID
```

Respuesta esperada: **204 No Content** (sin cuerpo — solo confirma que se eliminó).
Si el id no existe: 404.



---

## 🛠️ Paso 9 — Ver los datos en MongoDB Compass

MongoDB Compass te permite ver visualmente los documentos guardados en la base de datos.

### 9.1 — Abrir MongoDB Compass

1. Busca **MongoDB Compass** en el menú de inicio de Windows y ábrelo
2. Si no lo tienes instalado, descárgalo desde [mongodb.com/products/compass](https://www.mongodb.com/products/compass) e instálalo

### 9.2 — Conectar a tu base de datos local

1. En la pantalla inicial de Compass verás un campo de conexión
2. Debe decir `mongodb://localhost:27017` (ya viene por defecto)
3. Haz clic en el botón verde **"Connect"**

### 9.3 — Explorar la base de datos

1. En el panel izquierdo verás la lista de bases de datos
2. Busca y haz clic en **`michicafe_bebidas`**
3. Verás las colecciones de esa base de datos
4. Haz clic en **`bebidas`**
5. Se mostrarán todos los documentos guardados

### 9.4 — Lo que verás

Cada documento se ve así en Compass:

```json
{
  "_id": ObjectId("64a1b2c3d4e5f6a7b8c9d0e1"),
  "nombre": "Espresso",
  "precio": 35.0,
  "categoria": "caliente",
  "descripcion": "Café concentrado y puro",
  "disponible": true,
  "_class": "com.michicafe.servicebebidas.model.Bebida"
}
```

> 💡 El campo `_class` lo agrega Spring Data automáticamente para saber a qué clase Java corresponde el documento. No te preocupes por él.

### 9.5 — Explorar con filtros en Compass

En la barra de búsqueda de Compass puedes filtrar documentos:

- Para ver solo las bebidas disponibles, escribe en el filtro:
  ```json
  { "disponible": true }
  ```
- Para ver solo las bebidas calientes:
  ```json
  { "categoria": "caliente" }
  ```
- Haz clic en **"Find"** para aplicar el filtro

---

## 🎉 ¡Felicidades!

Construiste el Servicio de Bebidas completo del Michi Café.

### ¿Qué construiste en este proyecto?

- ✅ Un proyecto Spring Boot 4.1.0 con Java 17 configurado desde cero
- ✅ Conexión real a MongoDB en la base de datos `michicafe_bebidas`
- ✅ Modelo `Bebida` con `@Document` e `@Id`
- ✅ `BebidaRepository` que habla con MongoDB usando métodos por nombre
- ✅ `BebidaService` con lógica de negocio y operaciones CRUD completas
- ✅ `BebidaController` con 10 endpoints REST (GET, POST, PUT, PATCH, DELETE)
- ✅ `DataLoader` que carga 6 bebidas iniciales al arrancar
- ✅ Probaste todos los endpoints con Postman
- ✅ Visualizaste los datos en MongoDB Compass

### Estructura del proyecto

```
servicio-bebidas/
└── src/main/java/com/michic

### Estructura final del proyecto

```
servicio-bebidas/
└── src/main/java/com/michicafe/servicebebidas/
    ├── ServicioBedidasApplication.java   ← punto de entrada
    ├── config/
    │   └── DataLoader.java               ← carga datos iniciales
    ├── controller/
    │   └── BebidaController.java         ← recibe peticiones HTTP
    ├── service/
    │   └── BebidaService.java            ← lógica de negocio
    ├── repository/
    │   └── BebidaRepository.java         ← habla con MongoDB
    └── model/
        └── Bebida.java                   ← estructura del documento
```

---

## ➡️ Siguiente paso

En el **[Proyecto P04 — Servicio de Clientes](proyecto-04-servicio-clientes.md)** construirás el segundo microservicio: la gestión de clientes con registro, puntos de fidelidad y validación de correos duplicados.

> 🐾 "El menú del Michi Café ya está en línea. Ahora le damos la bienvenida a los clientes."

# Proyecto P03 — Servicio de Bebidas
## Construyendo el primer microservicio real del Michi Café

> 📚 Referencias teóricas:
> - [Módulo 04 — Spring Boot](modulo-04-spring-boot.md) → Capas, anotaciones, `@RestController`
> - [Módulo 05 — APIs REST](modulo-05-apis-rest.md) → GET, POST, PUT, DELETE, códigos HTTP
> - [Módulo 06 — MongoDB](modulo-06-mongodb.md) → Documentos, colecciones
> - [Módulo 07 — Spring + MongoDB](modulo-07-spring-mongodb.md) → `@Document`, `@Id`, Repository

---

## 🏠 ¿Qué vamos a construir?

El **Servicio de Bebidas** es el microservicio que gestiona el menú del Michi Café.
Permitirá:
- Ver todas las bebidas del menú
- Buscar una bebida por su ID
- Filtrar por categoría o disponibilidad
- Agregar bebidas nuevas al menú
- Actualizar una bebida existente
- Marcar una bebida como no disponible
- Eliminar una bebida del menú

Todo conectado a **MongoDB** para que los datos sean permanentes.

**Tiempo estimado:** 90-120 minutos

---

## 🗺️ Estructura que vamos a crear

```
servicio-bebidas/
└── src/main/java/com/michicafe/bebidas/
    ├── BebidasApplication.java          ← Punto de entrada
    ├── model/
    │   └── Bebida.java                  ← Representa una bebida
    ├── repository/
    │   └── BebidaRepository.java        ← Habla con MongoDB
    ├── service/
    │   └── BebidaService.java           ← Lógica de negocio
    ├── controller/
    │   └── BebidaController.java        ← Recibe peticiones HTTP
    └── config/
        └── DataLoader.java              ← Carga datos iniciales
```


---

## 🛠️ Paso 1: Crear el proyecto en Spring Initializr

Este microservicio es un proyecto Spring Boot **independiente**. Vamos a crearlo
desde cero igual que hicimos en el P02, pero con diferentes dependencias.

### 1.1 — Ir a Spring Initializr

1. Abre tu navegador y ve a: **https://start.spring.io**

### 1.2 — Configurar el proyecto

Configura cada campo **exactamente** como se muestra:

| Campo | Valor |
|-------|-------|
| Project | `Maven` |
| Language | `Java` |
| Spring Boot | `4.1.0` |
| Group | `com.michicafe` |
| Artifact | `servicio-bebidas` |
| Name | `servicio-bebidas` |
| Description | `Michi Café - Servicio de Bebidas` |
| Package name | `com.michicafe.bebidas` |
| Packaging | `Jar` |
| Java | `17` |

### 1.3 — Agregar las dependencias

Haz clic en **"ADD DEPENDENCIES"** y agrega estas tres:

1. Escribe `web` → selecciona **Spring Web**
2. Escribe `mongodb` → selecciona **Spring Data MongoDB**
3. Escribe `actuator` → selecciona **Spring Boot Actuator**

Verifica que en el panel de dependencias aparezcan las tres:
- ✅ Spring Web
- ✅ Spring Data MongoDB
- ✅ Spring Boot Actuator

### 1.4 — Generar y descomprimir

1. Haz clic en **"GENERATE"**
2. Se descarga `servicio-bebidas.zip`
3. Clic derecho → **"Extraer todo..."**
4. Extrae en: `C:\Proyectos\servicio-bebidas`

---

## 🛠️ Paso 2: Abrir en IntelliJ IDEA

1. Abre IntelliJ IDEA
2. En la pantalla de bienvenida, haz clic en **"Open"**
3. Navega a `C:\Proyectos\servicio-bebidas`
4. Selecciona la carpeta y haz clic en **"OK"**
5. Espera a que IntelliJ descargue las dependencias (2-5 minutos)

> ⚠️ Sabrás que terminó cuando desaparezca la barra de progreso en la parte inferior.

---

## 🛠️ Paso 3: Configurar el puerto y MongoDB

### 3.1 — Abrir application.properties

1. En el panel izquierdo, expande: `src` → `main` → `resources`
2. Haz **doble clic** en `application.properties`

### 3.2 — Escribir la configuración

El archivo estará vacío. Escribe exactamente esto:

```properties
# Puerto de este microservicio
server.port=8081

# Nombre del servicio
spring.application.name=servicio-bebidas

# Conexión a MongoDB
spring.data.mongodb.host=localhost
spring.data.mongodb.port=27017
spring.data.mongodb.database=michicafe_bebidas

# Logs
logging.level.com.michicafe=INFO
```

Guarda el archivo con **Ctrl + S**.

> 📚 ¿Qué significa cada línea?
> Consulta [Módulo 04 — application.properties](modulo-04-spring-boot.md#el-archivo-applicationproperties)


---

## 🛠️ Paso 4: Crear el modelo Bebida

El modelo es la clase que representa una bebida en nuestro sistema.
Cada bebida en MongoDB será un **documento** basado en esta clase.

### 4.1 — Crear el paquete `model`

1. En el panel izquierdo, busca el paquete `com.michicafe.bebidas`
   (está en `src` → `main` → `java` → `com.michicafe.bebidas`)
2. Haz **clic derecho** sobre `com.michicafe.bebidas`
3. Pasa el mouse sobre **"New"** → haz clic en **"Package"**
4. Escribe: `model`
5. Presiona **Enter**

### 4.2 — Crear la clase `Bebida`

1. Haz **clic derecho** sobre el paquete `model`
2. **"New"** → **"Java Class"**
3. Escribe: `Bebida`
4. Asegúrate de que esté seleccionado **"Class"**
5. Presiona **Enter**

### 4.3 — Escribir el código de la clase Bebida

El archivo se abre con el código vacío. Reemplaza **todo** el contenido con esto:

```java
package com.michicafe.bebidas.model;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document(collection = "bebidas")
public class Bebida {

    @Id
    private String id;

    private String nombre;
    private double precio;
    private String categoria;
    private String descripcion;
    private boolean disponible;

    // Constructor vacío — Spring Data lo necesita para leer datos de MongoDB
    public Bebida() {}

    // Constructor con parámetros — para crear bebidas en el código
    public Bebida(String nombre, double precio,
                  String categoria, String descripcion, boolean disponible) {
        this.nombre = nombre;
        this.precio = precio;
        this.categoria = categoria;
        this.descripcion = descripcion;
        this.disponible = disponible;
    }

    // Getters — permiten leer los valores
    public String getId()          { return id; }
    public String getNombre()      { return nombre; }
    public double getPrecio()      { return precio; }
    public String getCategoria()   { return categoria; }
    public String getDescripcion() { return descripcion; }
    public boolean isDisponible()  { return disponible; }

    // Setters — permiten modificar los valores
    public void setId(String id)               { this.id = id; }
    public void setNombre(String nombre)       { this.nombre = nombre; }
    public void setPrecio(double precio)       { this.precio = precio; }
    public void setCategoria(String categoria) { this.categoria = categoria; }
    public void setDescripcion(String desc)    { this.descripcion = desc; }
    public void setDisponible(boolean disp)    { this.disponible = disp; }
}
```

### 4.4 — Importar las anotaciones

Verás que `@Document` e `@Id` están subrayados en rojo.

1. Haz clic sobre `@Document`
2. Presiona **Alt + Enter**
3. Selecciona **"Import class"**
4. Repite para `@Id`

Las líneas de import aparecerán automáticamente arriba del archivo:
```java
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;
```

Guarda con **Ctrl + S**.

> 📚 ¿Qué hace `@Document` y `@Id`?
> Consulta [Módulo 07 — Anotar el modelo](modulo-07-spring-mongodb.md#paso-3-anotar-el-modelo-con-document)


---

## 🛠️ Paso 5: Crear el Repository

El Repository es la interfaz que se comunica directamente con MongoDB.
Spring genera el código de las consultas automáticamente solo con leer el nombre del método.

### 5.1 — Crear el paquete `repository`

1. Haz **clic derecho** sobre `com.michicafe.bebidas`
2. **"New"** → **"Package"**
3. Escribe: `repository`
4. Presiona **Enter**

### 5.2 — Crear la interfaz `BebidaRepository`

1. Haz **clic derecho** sobre el paquete `repository`
2. **"New"** → **"Java Class"**
3. Escribe: `BebidaRepository`
4. ⚠️ Esta vez selecciona **"Interface"** (no Class)
5. Presiona **Enter**

### 5.3 — Escribir el código del Repository

```java
package com.michicafe.bebidas.repository;

import com.michicafe.bebidas.model.Bebida;
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public interface BebidaRepository extends MongoRepository<Bebida, String> {
    // MongoRepository<Bebida, String> significa:
    // - Bebida  → el tipo de documento que maneja
    // - String  → el tipo del campo @Id

    // Spring genera estas consultas automáticamente por el nombre del método:

    // Busca todas las bebidas de una categoría (ej: "caliente")
    List<Bebida> findByCategoria(String categoria);

    // Busca solo las bebidas disponibles (disponible = true)
    List<Bebida> findByDisponibleTrue();

    // Busca bebidas disponibles de una categoría específica
    List<Bebida> findByCategoriaAndDisponibleTrue(String categoria);

    // Busca bebidas con precio entre dos valores
    List<Bebida> findByPrecioBetween(double min, double max);

    // Busca bebidas cuyo nombre contenga un texto (sin importar mayúsculas)
    List<Bebida> findByNombreContainingIgnoreCase(String texto);
}
```

Importa `MongoRepository` y `Repository` con **Alt + Enter** donde estén subrayados en rojo.

Guarda con **Ctrl + S**.

> 📚 ¿Cómo funciona MongoRepository y los métodos por nombre?
> Consulta [Módulo 07 — Crear el Repository](modulo-07-spring-mongodb.md#paso-4-crear-el-repository)

---

## 🛠️ Paso 6: Crear el Service

El Service contiene la lógica de negocio. Es el intermediario entre el Controller y el Repository.

### 6.1 — Crear el paquete `service`

1. Haz **clic derecho** sobre `com.michicafe.bebidas`
2. **"New"** → **"Package"**
3. Escribe: `service`
4. Presiona **Enter**

### 6.2 — Crear la clase `BebidaService`

1. Haz **clic derecho** sobre el paquete `service`
2. **"New"** → **"Java Class"**
3. Escribe: `BebidaService`
4. Selecciona **"Class"**
5. Presiona **Enter**

### 6.3 — Escribir el código del Service

```java
package com.michicafe.bebidas.service;

import com.michicafe.bebidas.model.Bebida;
import com.michicafe.bebidas.repository.BebidaRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

@Service
public class BebidaService {

    @Autowired
    private BebidaRepository bebidaRepository;
    // Spring inyecta automáticamente el repository aquí
    // No necesitas escribir: bebidaRepository = new BebidaRepository()

    // Guarda una bebida nueva en MongoDB
    public Bebida guardar(Bebida bebida) {
        return bebidaRepository.save(bebida);
    }

    // Devuelve todas las bebidas
    public List<Bebida> obtenerTodas() {
        return bebidaRepository.findAll();
    }

    // Busca una bebida por su ID — devuelve Optional porque puede no existir
    public Optional<Bebida> obtenerPorId(String id) {
        return bebidaRepository.findById(id);
    }

    // Devuelve solo las bebidas disponibles
    public List<Bebida> obtenerDisponibles() {
        return bebidaRepository.findByDisponibleTrue();
    }

    // Devuelve bebidas de una categoría
    public List<Bebida> obtenerPorCategoria(String categoria) {
        return bebidaRepository.findByCategoria(categoria);
    }

    // Devuelve bebidas dentro de un rango de precio
    public List<Bebida> obtenerPorRangoPrecio(double min, double max) {
        return bebidaRepository.findByPrecioBetween(min, max);
    }

    // Busca bebidas por nombre (parcial, sin importar mayúsculas)
    public List<Bebida> buscarPorNombre(String texto) {
        return bebidaRepository.findByNombreContainingIgnoreCase(texto);
    }

    // Actualiza una bebida existente — si no existe, devuelve Optional vacío
    public Optional<Bebida> actualizar(String id, Bebida bebidaActualizada) {
        if (!bebidaRepository.existsById(id)) {
            return Optional.empty();
        }
        bebidaActualizada.setId(id);
        return Optional.of(bebidaRepository.save(bebidaActualizada));
    }

    // Cambia solo la disponibilidad de una bebida
    public Optional<Bebida> cambiarDisponibilidad(String id, boolean disponible) {
        return bebidaRepository.findById(id).map(bebida -> {
            bebida.setDisponible(disponible);
            return bebidaRepository.save(bebida);
        });
    }

    // Elimina una bebida — devuelve true si existía, false si no
    public boolean eliminar(String id) {
        if (!bebidaRepository.existsById(id)) return false;
        bebidaRepository.deleteById(id);
        return true;
    }
}
```

Importa todo con **Alt + Enter** sobre cada subrayado rojo. Guarda con **Ctrl + S**.


---

## 🛠️ Paso 7: Crear el Controller

El Controller recibe las peticiones HTTP y llama al Service para procesarlas.

### 7.1 — Crear el paquete `controller`

1. Haz **clic derecho** sobre `com.michicafe.bebidas`
2. **"New"** → **"Package"**
3. Escribe: `controller`
4. Presiona **Enter**

### 7.2 — Crear la clase `BebidaController`

1. Haz **clic derecho** sobre el paquete `controller`
2. **"New"** → **"Java Class"**
3. Escribe: `BebidaController`
4. Selecciona **"Class"**
5. Presiona **Enter**

### 7.3 — Escribir el código del Controller

```java
package com.michicafe.bebidas.controller;

import com.michicafe.bebidas.model.Bebida;
import com.michicafe.bebidas.service.BebidaService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/bebidas")
public class BebidaController {

    @Autowired
    private BebidaService bebidaService;

    // GET /api/bebidas → devuelve todas las bebidas
    @GetMapping
    public ResponseEntity<List<Bebida>> obtenerTodas() {
        return ResponseEntity.ok(bebidaService.obtenerTodas());
    }

    // GET /api/bebidas/{id} → devuelve una bebida por ID
    @GetMapping("/{id}")
    public ResponseEntity<Bebida> obtenerPorId(@PathVariable String id) {
        return bebidaService.obtenerPorId(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // GET /api/bebidas/disponibles → solo bebidas disponibles
    @GetMapping("/disponibles")
    public ResponseEntity<List<Bebida>> obtenerDisponibles() {
        return ResponseEntity.ok(bebidaService.obtenerDisponibles());
    }

    // GET /api/bebidas/categoria/caliente → bebidas por categoría
    @GetMapping("/categoria/{categoria}")
    public ResponseEntity<List<Bebida>> obtenerPorCategoria(
            @PathVariable String categoria) {
        return ResponseEntity.ok(bebidaService.obtenerPorCategoria(categoria));
    }

    // GET /api/bebidas/precio?min=40&max=55 → bebidas por rango de precio
    @GetMapping("/precio")
    public ResponseEntity<List<Bebida>> obtenerPorPrecio(
            @RequestParam(defaultValue = "0") double min,
            @RequestParam(defaultValue = "9999") double max) {
        return ResponseEntity.ok(bebidaService.obtenerPorRangoPrecio(min, max));
    }

    // GET /api/bebidas/buscar?nombre=latte → buscar por nombre
    @GetMapping("/buscar")
    public ResponseEntity<List<Bebida>> buscarPorNombre(
            @RequestParam String nombre) {
        return ResponseEntity.ok(bebidaService.buscarPorNombre(nombre));
    }

    // POST /api/bebidas → crear bebida nueva
    @PostMapping
    public ResponseEntity<Bebida> crear(@RequestBody Bebida bebida) {
        Bebida nueva = bebidaService.guardar(bebida);
        return ResponseEntity.status(HttpStatus.CREATED).body(nueva);
    }

    // PUT /api/bebidas/{id} → reemplazar bebida completa
    @PutMapping("/{id}")
    public ResponseEntity<Bebida> actualizar(
            @PathVariable String id,
            @RequestBody Bebida bebida) {
        return bebidaService.actualizar(id, bebida)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // PATCH /api/bebidas/{id}/disponibilidad?valor=false → cambiar disponibilidad
    @PatchMapping("/{id}/disponibilidad")
    public ResponseEntity<Bebida> cambiarDisponibilidad(
            @PathVariable String id,
            @RequestParam boolean valor) {
        return bebidaService.cambiarDisponibilidad(id, valor)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // DELETE /api/bebidas/{id} → eliminar bebida
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> eliminar(@PathVariable String id) {
        if (bebidaService.eliminar(id)) {
            return ResponseEntity.noContent().build();
        }
        return ResponseEntity.notFound().build();
    }
}
```

Importa todo con **Alt + Enter**. Guarda con **Ctrl + S**.

> 📚 ¿Qué hace cada anotación (`@GetMapping`, `@PostMapping`, `@RequestBody`...)?
> Consulta [Módulo 05 — APIs REST](modulo-05-apis-rest.md)


---

## 🛠️ Paso 8: Crear el DataLoader (datos iniciales)

Para no tener que agregar bebidas manualmente cada vez que arranques el servidor,
vamos a crear una clase que cargue el menú inicial automáticamente.

### 8.1 — Crear el paquete `config`

1. Haz **clic derecho** sobre `com.michicafe.bebidas`
2. **"New"** → **"Package"**
3. Escribe: `config`
4. Presiona **Enter**

### 8.2 — Crear la clase `DataLoader`

1. Haz **clic derecho** sobre el paquete `config`
2. **"New"** → **"Java Class"**
3. Escribe: `DataLoader`
4. Selecciona **"Class"**
5. Presiona **Enter**

### 8.3 — Escribir el código del DataLoader

```java
package com.michicafe.bebidas.config;

import com.michicafe.bebidas.model.Bebida;
import com.michicafe.bebidas.repository.BebidaRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component
public class DataLoader implements CommandLineRunner {
    // CommandLineRunner hace que el método run() se ejecute al arrancar la app

    @Autowired
    private BebidaRepository bebidaRepository;

    @Override
    public void run(String... args) {
        // Solo carga datos si la colección está vacía
        // Así no duplica datos cada vez que reinicias
        if (bebidaRepository.count() == 0) {
            System.out.println("🐱 Cargando menú inicial del Michi Café...");

            bebidaRepository.save(new Bebida(
                "Espresso", 35.00, "caliente",
                "Café concentrado y puro", true));

            bebidaRepository.save(new Bebida(
                "Latte de vainilla", 45.50, "caliente",
                "Espresso con leche vaporizada y jarabe de vainilla", true));

            bebidaRepository.save(new Bebida(
                "Cappuccino", 48.00, "caliente",
                "Espresso con leche y espuma cremosa", true));

            bebidaRepository.save(new Bebida(
                "Matcha Latte", 55.00, "caliente",
                "Té matcha japonés con leche de avena", true));

            bebidaRepository.save(new Bebida(
                "Cold Brew", 50.00, "frio",
                "Café infusionado en frío durante 12 horas", true));

            bebidaRepository.save(new Bebida(
                "Chai Latte", 52.00, "caliente",
                "Té chai especiado con leche", false));

            System.out.println("✅ Menú cargado con "
                + bebidaRepository.count() + " bebidas");
        } else {
            System.out.println("📋 El menú ya tiene "
                + bebidaRepository.count() + " bebidas guardadas");
        }
    }
}
```

Importa todo con **Alt + Enter**. Guarda con **Ctrl + S**.

---

## 🛠️ Paso 9: Ejecutar el servicio

### 9.1 — Verificar que MongoDB está corriendo

Antes de arrancar, asegúrate de que MongoDB esté activo:
1. Presiona **Windows + R** → escribe `services.msc` → **Enter**
2. Busca **"MongoDB Server"** → debe decir **"Running"**
3. Si no está corriendo: clic derecho → **"Start"**

### 9.2 — Arrancar el servicio en IntelliJ

1. En el panel izquierdo, abre `BebidasApplication.java`
   (está en `com.michicafe.bebidas`)
2. Haz clic en el triángulo verde ▶ junto al método `main`
3. Selecciona **"Run 'BebidasApplication'"**

### 9.3 — Verificar que arrancó correctamente

En la consola de IntelliJ busca estas líneas:

```
🐱 Cargando menú inicial del Michi Café...
✅ Menú cargado con 6 bebidas
Started BebidasApplication in 3.x seconds
Tomcat started on port 8081
```

✅ Si ves eso, el servicio está corriendo en `http://localhost:8081`

❌ Si ves un error en rojo con `Connection refused` o `MongoException`:
- Verifica que MongoDB esté corriendo (paso 9.1)
- Verifica que `application.properties` tenga `spring.data.mongodb.host=localhost`

---

## 🛠️ Paso 10: Probar la API con Postman

### 10.1 — Crear una colección en Postman

Una colección agrupa peticiones relacionadas. Vamos a crear una para el servicio de bebidas.

1. Abre **Postman**
2. En el panel izquierdo, haz clic en **"Collections"**
3. Haz clic en el botón **"+"** para crear una colección nueva
4. Escribe el nombre: `Michi Café - Bebidas`
5. Presiona **Enter**

### 10.2 — Probar GET todas las bebidas

1. Dentro de la colección, haz clic en **"Add a request"**
2. Ponle el nombre: `GET todas las bebidas`
3. Asegúrate de que el método sea **GET**
4. Escribe la URL: `http://localhost:8081/api/bebidas`
5. Haz clic en **"Send"**

Respuesta esperada (200 OK):
```json
[
  {
    "id": "64a1b2c3...",
    "nombre": "Espresso",
    "precio": 35.0,
    "categoria": "caliente",
    "descripcion": "Café concentrado y puro",
    "disponible": true
  },
  ...
]
```

### 10.3 — Probar GET bebidas disponibles

1. Nueva petición: `GET bebidas disponibles`
2. Método: **GET**
3. URL: `http://localhost:8081/api/bebidas/disponibles`
4. **Send**

Devuelve solo 5 bebidas (el Chai Latte tiene `disponible: false`).

### 10.4 — Probar POST crear bebida nueva

1. Nueva petición: `POST crear bebida`
2. Método: **POST**
3. URL: `http://localhost:8081/api/bebidas`
4. Haz clic en la pestaña **"Body"**
5. Selecciona **"raw"**
6. En el desplegable de la derecha, selecciona **"JSON"**
7. Escribe este cuerpo:

```json
{
  "nombre": "Frappuccino de Caramelo",
  "precio": 65.00,
  "categoria": "frio",
  "descripcion": "Café frío batido con caramelo y crema",
  "disponible": true
}
```

8. Haz clic en **"Send"**

Respuesta esperada (201 Created):
```json
{
  "id": "64a1b2c3d4e5f6a7b8c9d0e7",
  "nombre": "Frappuccino de Caramelo",
  "precio": 65.0,
  "categoria": "frio",
  "descripcion": "Café frío batido con caramelo y crema",
  "disponible": true
}
```

Guarda el `id` que devolvió — lo necesitarás para las siguientes pruebas.

### 10.5 — Probar PUT actualizar bebida

1. Nueva petición: `PUT actualizar bebida`
2. Método: **PUT**
3. URL: `http://localhost:8081/api/bebidas/` + el id que copiaste
   - Ejemplo: `http://localhost:8081/api/bebidas/64a1b2c3d4e5f6a7b8c9d0e7`
4. Body → raw → JSON:

```json
{
  "nombre": "Frappuccino de Caramelo Grande",
  "precio": 70.00,
  "categoria": "frio",
  "descripcion": "Versión grande del frappuccino de caramelo",
  "disponible": true
}
```

5. **Send** → debe devolver la bebida actualizada con 200 OK

### 10.6 — Probar PATCH cambiar disponibilidad

1. Nueva petición: `PATCH disponibilidad`
2. Método: **PATCH**
3. URL: `http://localhost:8081/api/bebidas/` + el id + `/disponibilidad?valor=false`
4. **Send** → debe devolver la bebida con `"disponible": false`

### 10.7 — Probar DELETE eliminar bebida

1. Nueva petición: `DELETE bebida`
2. Método: **DELETE**
3. URL: `http://localhost:8081/api/bebidas/` + el id
4. **Send** → debe responder `204 No Content` (sin cuerpo)

### 10.8 — Verificar la persistencia

1. Detén el servidor en IntelliJ (botón cuadrado rojo ⏹)
2. Vuelve a arrancarlo (triángulo verde ▶)
3. Haz GET a `http://localhost:8081/api/bebidas`
4. Las bebidas que creaste siguen ahí ✅

Esto confirma que MongoDB está guardando los datos de forma permanente.


---

## 🛠️ Paso 11: Ver los datos en MongoDB Compass

MongoDB Compass te permite ver visualmente los documentos guardados en MongoDB.

1. Abre **MongoDB Compass** (lo instalamos en el P01)
2. En el campo de conexión verás la URL: `mongodb://localhost:27017`
3. Haz clic en **"Connect"**
4. En el panel izquierdo verás las bases de datos disponibles
5. Busca y haz clic en **`michicafe_bebidas`**
6. Haz clic en la colección **`bebidas`**
7. Verás todos los documentos guardados como tarjetas JSON

Desde aquí puedes:
- Ver cada documento expandido
- Filtrar documentos usando el campo "Filter"
- Editar un documento haciendo clic en el ícono de lápiz
- Eliminar un documento haciendo clic en el ícono de papelera

---

## ✅ Resumen: lo que construiste en este módulo

- ✅ Proyecto Spring Boot 4.1.0 independiente en el puerto 8081
- ✅ Modelo `Bebida` anotado con `@Document` e `@Id`
- ✅ `BebidaRepository` con consultas automáticas por nombre de método
- ✅ `BebidaService` con todas las operaciones CRUD y lógica de negocio
- ✅ `BebidaController` con 10 endpoints REST completos
- ✅ `DataLoader` que carga 6 bebidas de ejemplo al arrancar
- ✅ Datos persistentes en MongoDB — sobreviven al reinicio del servidor
- ✅ Pruebas completas con Postman

---

## ➡️ Siguiente paso

En el **[Proyecto P04 — Servicio de Clientes](proyecto-04-servicio-clientes.md)**
construirás el segundo microservicio: la gestión de clientes con registro,
puntos de fidelidad y validación de correos duplicados.

> 🐾 "El menú del Michi Café ya está en línea. Ahora le damos la bienvenida a los clientes."