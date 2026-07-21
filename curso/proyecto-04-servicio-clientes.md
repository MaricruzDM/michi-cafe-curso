# Proyecto P04 — Servicio de Clientes
## Registrando a los amigos del Michi Café

> 📚 **Referencias teóricas:**
> - [Módulo 05 — APIs REST](modulo-05-apis-rest.md) → GET, POST, PUT, PATCH, DELETE
> - [Módulo 06 — MongoDB](modulo-06-mongodb.md) → documentos, colecciones, índices únicos
> - [Módulo 07 — Spring + MongoDB](modulo-07-spring-mongodb.md) → `@Document`, `@Indexed`, Repository

---

## 🏠 ¿Qué vamos a construir?

El **Servicio de Clientes** gestiona el registro de los clientes del Michi Café y su
programa de puntos de fidelidad. Corre en el puerto **8082** con su propia base de datos.

**Lo que construirás:**
- Modelo `Cliente` con correo único y puntos de fidelidad
- `ClienteRepository` con consultas personalizadas
- `ClienteService` que valida el correo antes de registrar y suma puntos
- `ClienteController` con todos los endpoints REST
- `DataLoader` con 3 clientes de prueba
- Pruebas completas en Postman

**Tiempo estimado:** 60-90 minutos

---

## 🛠️ Paso 1 — Crear el proyecto en Spring Initializr

### 1.1 — Configurar en https://start.spring.io

| Campo | Valor |
|-------|-------|
| Project | `Maven` |
| Language | `Java` |
| Spring Boot | `4.1.0` |
| Group | `com.michicafe` |
| Artifact | `servicio-clientes` |
| Name | `servicio-clientes` |
| Description | `Michi Café - Servicio de gestión de clientes` |
| Package name | `com.michicafe.servicioclientes` (se llena solo) |
| Packaging | `Jar` |
| Java | `17` |

### 1.2 — Agregar las dependencias (una por una)

1. **"ADD DEPENDENCIES"** → escribe `web` → selecciona **Spring Web**
2. **"ADD DEPENDENCIES"** → escribe `mongodb` → selecciona **Spring Data MongoDB**
3. **"ADD DEPENDENCIES"** → escribe `valid` → selecciona **Validation**
4. **"ADD DEPENDENCIES"** → escribe `actuator` → selecciona **Spring Boot Actuator**

### 1.3 — Generar, descargar y abrir

1. Clic en **"GENERATE"** → se descarga `servicio-clientes.zip`
2. Descomprime en `C:\Proyectos\servicio-clientes`
3. Abre IntelliJ → **"Open"** → selecciona la carpeta → **"OK"** → **"Trust Project"**
4. Espera que termine la descarga de dependencias (2-5 minutos)

---

## 🛠️ Paso 2 — Configurar application.properties

1. En el panel izquierdo expande: `src` → `main` → `resources`
2. Haz **doble clic** en `application.properties`
3. Borra cualquier contenido y escribe:

```properties
server.port=8082
spring.application.name=servicio-clientes
spring.data.mongodb.host=localhost
spring.data.mongodb.port=27017
spring.data.mongodb.database=michicafe_clientes
logging.level.com.michicafe=INFO
```

| Línea | Explicación |
|-------|-------------|
| `server.port=8082` | Puerto 8082 — diferente al servicio de bebidas (8081) para correr simultáneamente |
| `spring.data.mongodb.database=michicafe_clientes` | Base de datos exclusiva para clientes |

Guarda con **Ctrl + S**.

---

## 🛠️ Paso 3 — Crear el modelo Cliente

### 3.1 — Crear el paquete `model`

1. Haz **clic derecho** sobre `com.michicafe.servicioclientes`
2. **"New"** → **"Package"** → escribe `model` → **Enter**

### 3.2 — Crear la clase `Cliente`

1. **Clic derecho** en `model` → **"New"** → **"Java Class"**
2. Escribe `Cliente` → asegúrate que esté seleccionado **"Class"** → **Enter**

### 3.3 — Código de la clase Cliente

```java
package com.michicafe.servicioclientes.model;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.index.Indexed;
import org.springframework.data.mongodb.core.mapping.Document;

import java.time.LocalDateTime;

@Document(collection = "clientes")
public class Cliente {

    @Id
    private String id;

    private String nombre;

    @Indexed(unique = true)
    private String correo;

    private int puntosFidelidad;

    private LocalDateTime fechaRegistro;

    public Cliente() {}

    public Cliente(String nombre, String correo) {
        this.nombre = nombre;
        this.correo = correo;
        this.puntosFidelidad = 0;
        this.fechaRegistro = LocalDateTime.now();
    }

    public String getId()                   { return id; }
    public String getNombre()               { return nombre; }
    public String getCorreo()               { return correo; }
    public int getPuntosFidelidad()         { return puntosFidelidad; }
    public LocalDateTime getFechaRegistro() { return fechaRegistro; }

    public void setId(String id)                        { this.id = id; }
    public void setNombre(String nombre)                { this.nombre = nombre; }
    public void setCorreo(String correo)                { this.correo = correo; }
    public void setPuntosFidelidad(int puntosFidelidad) { this.puntosFidelidad = puntosFidelidad; }
    public void setFechaRegistro(LocalDateTime fecha)   { this.fechaRegistro = fecha; }
}
```

### 3.4 — Importar con Alt+Enter

Haz clic en cada elemento subrayado en rojo y presiona **Alt + Enter** → **Import class**:

- `Id` → `org.springframework.data.annotation.Id`
- `Indexed` → `org.springframework.data.mongodb.core.index.Indexed`
- `Document` → `org.springframework.data.mongodb.core.mapping.Document`
- `LocalDateTime` → `java.time.LocalDateTime`

### 3.5 — Explicación de las anotaciones

| Elemento | Explicación |
|----------|-------------|
| `@Document(collection = "clientes")` | Esta clase se guarda en la colección "clientes" de MongoDB |
| `@Id` | Identificador único del documento; MongoDB lo genera automáticamente |
| `@Indexed(unique = true)` | Crea un índice en MongoDB que impide que dos clientes tengan el mismo correo. Si intentas guardar un correo duplicado, el sistema lanzará un error automáticamente |
| `LocalDateTime.now()` | Captura la fecha y hora exacta del momento en que se registra el cliente |

> 📚 Más sobre índices: [Módulo 06 — Índices](modulo-06-mongodb.md#índices-el-índice-del-archivero)



---

## 🛠️ Paso 4 — Crear el Repository

### 4.1 — Crear el paquete `repository`

1. **Clic derecho** en `com.michicafe.servicioclientes` → **"New"** → **"Package"**
2. Escribe `repository` → **Enter**

### 4.2 — Crear la interfaz `ClienteRepository`

> ⚠️ Es una **interfaz**, no una clase. Asegúrate de seleccionar "Interface".

1. **Clic derecho** en `repository` → **"New"** → **"Java Class"**
2. Escribe `ClienteRepository`
3. En la lista de abajo, haz clic en **"Interface"**
4. **Enter**

### 4.3 — Código del Repository

```java
package com.michicafe.servicioclientes.repository;

import com.michicafe.servicioclientes.model.Cliente;
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Optional;

@Repository
public interface ClienteRepository extends MongoRepository<Cliente, String> {

    Optional<Cliente> findByCorreo(String correo);

    List<Cliente> findByNombreContainingIgnoreCase(String nombre);

    List<Cliente> findByPuntosFidelidadGreaterThanEqual(int puntos);

    boolean existsByCorreo(String correo);
}
```

### 4.4 — Importar con Alt+Enter

- `Cliente` → `com.michicafe.servicioclientes.model.Cliente`
- `MongoRepository` → `org.springframework.data.mongodb.repository.MongoRepository`
- `Repository` → `org.springframework.stereotype.Repository`
- `List` → `java.util.List`
- `Optional` → `java.util.Optional`

### 4.5 — Los métodos del Repository

| Método | Lo que hace |
|--------|-------------|
| `findByCorreo(correo)` | Busca el cliente con ese correo exacto. Devuelve `Optional` porque puede no existir |
| `findByNombreContainingIgnoreCase(texto)` | Busca clientes cuyo nombre contiene el texto (mayúsculas/minúsculas indistintas) |
| `findByPuntosFidelidadGreaterThanEqual(n)` | Busca clientes con `puntosFidelidad >= n` |
| `existsByCorreo(correo)` | Devuelve `true` si ya hay un cliente con ese correo, `false` si no |

---

## 🛠️ Paso 5 — Crear el Service

### 5.1 — Crear el paquete `service`

1. **Clic derecho** en `com.michicafe.servicioclientes` → **"New"** → **"Package"**
2. Escribe `service` → **Enter**

### 5.2 — Crear la clase `ClienteService`

1. **Clic derecho** en `service` → **"New"** → **"Java Class"**
2. Escribe `ClienteService` → asegúrate de que diga **"Class"** → **Enter**

### 5.3 — Código del Service

```java
package com.michicafe.servicioclientes.service;

import com.michicafe.servicioclientes.model.Cliente;
import com.michicafe.servicioclientes.repository.ClienteRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

@Service
public class ClienteService {

    @Autowired
    private ClienteRepository clienteRepository;

    // ── Registrar un cliente nuevo ───────────────────────────────────────────
    // Valida que el correo no esté ya en uso antes de guardar
    public Cliente registrar(Cliente cliente) {
        if (clienteRepository.existsByCorreo(cliente.getCorreo())) {
            throw new RuntimeException(
                "Ya existe un cliente registrado con el correo: " + cliente.getCorreo());
        }
        return clienteRepository.save(cliente);
    }

    // ── Obtener todos los clientes ───────────────────────────────────────────
    public List<Cliente> obtenerTodos() {
        return clienteRepository.findAll();
    }

    // ── Obtener cliente por id ───────────────────────────────────────────────
    public Optional<Cliente> obtenerPorId(String id) {
        return clienteRepository.findById(id);
    }

    // ── Obtener cliente por correo ───────────────────────────────────────────
    public Optional<Cliente> obtenerPorCorreo(String correo) {
        return clienteRepository.findByCorreo(correo);
    }

    // ── Agregar puntos de fidelidad ──────────────────────────────────────────
    // Busca el cliente, suma los puntos y guarda
    public Optional<Cliente> agregarPuntos(String id, int puntos) {
        return clienteRepository.findById(id).map(cliente -> {
            int puntosActuales = cliente.getPuntosFidelidad();
            cliente.setPuntosFidelidad(puntosActuales + puntos);
            return clienteRepository.save(cliente);
        });
    }

    // ── Actualizar datos del cliente (PUT) ───────────────────────────────────
    public Optional<Cliente> actualizar(String id, Cliente clienteActualizado) {
        if (!clienteRepository.existsById(id)) {
            return Optional.empty();
        }
        clienteActualizado.setId(id);
        return Optional.of(clienteRepository.save(clienteActualizado));
    }

    // ── Eliminar un cliente ──────────────────────────────────────────────────
    public boolean eliminar(String id) {
        if (!clienteRepository.existsById(id)) {
            return false;
        }
        clienteRepository.deleteById(id);
        return true;
    }
}
```

### 5.4 — Importar con Alt+Enter

- `Cliente` → `com.michicafe.servicioclientes.model.Cliente`
- `ClienteRepository` → `com.michicafe.servicioclientes.repository.ClienteRepository`
- `Autowired` → `org.springframework.beans.factory.annotation.Autowired`
- `Service` → `org.springframework.stereotype.Service`
- `List` → `java.util.List`
- `Optional` → `java.util.Optional`

### 5.5 — Puntos importantes del Service

**La validación del correo en `registrar`:**

```java
if (clienteRepository.existsByCorreo(cliente.getCorreo())) {
    throw new RuntimeException("Ya existe un cliente registrado con el correo: ...");
}
```

Antes de guardar, el service pregunta al repository si ya existe un cliente con ese correo.
Si existe, lanza una excepción con un mensaje claro. Si no existe, lo guarda normalmente.

**La lógica de `agregarPuntos`:**

```java
return clienteRepository.findById(id).map(cliente -> {
    cliente.setPuntosFidelidad(cliente.getPuntosFidelidad() + puntos);
    return clienteRepository.save(cliente);
});
```

1. Busca el cliente por id (si no existe, el `Optional` estará vacío y no hace nada)
2. Suma los puntos actuales más los nuevos
3. Guarda el cliente actualizado en MongoDB
4. Devuelve el cliente actualizado



---

## 🛠️ Paso 6 — Crear el Controller

### 6.1 — Crear el paquete `controller`

1. **Clic derecho** en `com.michicafe.servicioclientes` → **"New"** → **"Package"**
2. Escribe `controller` → **Enter**

### 6.2 — Crear la clase `ClienteController`

1. **Clic derecho** en `controller` → **"New"** → **"Java Class"**
2. Escribe `ClienteController` → **"Class"** → **Enter**

### 6.3 — Código del Controller

```java
package com.michicafe.servicioclientes.controller;

import com.michicafe.servicioclientes.model.Cliente;
import com.michicafe.servicioclientes.service.ClienteService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;
import java.util.Map;

@RestController
@RequestMapping("/api/clientes")
public class ClienteController {

    @Autowired
    private ClienteService clienteService;

    // ── GET /api/clientes ────────────────────────────────────────────────────
    @GetMapping
    public ResponseEntity<List<Cliente>> obtenerTodos() {
        return ResponseEntity.ok(clienteService.obtenerTodos());
    }

    // ── GET /api/clientes/{id} ───────────────────────────────────────────────
    @GetMapping("/{id}")
    public ResponseEntity<Cliente> obtenerPorId(@PathVariable String id) {
        return clienteService.obtenerPorId(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // ── GET /api/clientes/correo/{correo} ────────────────────────────────────
    @GetMapping("/correo/{correo}")
    public ResponseEntity<Cliente> obtenerPorCorreo(@PathVariable String correo) {
        return clienteService.obtenerPorCorreo(correo)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // ── POST /api/clientes ───────────────────────────────────────────────────
    // Registrar un cliente nuevo
    @PostMapping
    public ResponseEntity<?> registrar(@RequestBody Cliente cliente) {
        try {
            Cliente nuevo = clienteService.registrar(cliente);
            return ResponseEntity.status(HttpStatus.CREATED).body(nuevo);
        } catch (RuntimeException e) {
            // Si el correo ya existe, devolvemos 409 Conflict con mensaje de error
            return ResponseEntity.status(HttpStatus.CONFLICT)
                    .body(Map.of("error", e.getMessage()));
        }
    }

    // ── PATCH /api/clientes/{id}/puntos?cantidad=50 ──────────────────────────
    // Agregar puntos de fidelidad a un cliente
    @PatchMapping("/{id}/puntos")
    public ResponseEntity<Cliente> agregarPuntos(
            @PathVariable String id,
            @RequestParam int cantidad) {
        return clienteService.agregarPuntos(id, cantidad)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // ── PUT /api/clientes/{id} ───────────────────────────────────────────────
    // Actualizar datos completos del cliente
    @PutMapping("/{id}")
    public ResponseEntity<Cliente> actualizar(
            @PathVariable String id,
            @RequestBody Cliente cliente) {
        return clienteService.actualizar(id, cliente)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // ── DELETE /api/clientes/{id} ────────────────────────────────────────────
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> eliminar(@PathVariable String id) {
        if (clienteService.eliminar(id)) {
            return ResponseEntity.noContent().build();
        }
        return ResponseEntity.notFound().build();
    }
}
```

### 6.4 — Importar con Alt+Enter

- `Cliente` → `com.michicafe.servicioclientes.model.Cliente`
- `ClienteService` → `com.michicafe.servicioclientes.service.ClienteService`
- `Autowired` → `org.springframework.beans.factory.annotation.Autowired`
- `HttpStatus` → `org.springframework.http.HttpStatus`
- `ResponseEntity` → `org.springframework.http.ResponseEntity`
- Las anotaciones `@RestController`, `@RequestMapping`, `@GetMapping`, etc. → `org.springframework.web.bind.annotation.*`
- `List` → `java.util.List`
- `Map` → `java.util.Map`

### 6.5 — Tabla de endpoints

| Método | URL | Descripción | Respuesta |
|--------|-----|-------------|-----------|
| GET | `/api/clientes` | Todos los clientes | 200 + lista |
| GET | `/api/clientes/{id}` | Cliente por id | 200 o 404 |
| GET | `/api/clientes/correo/{correo}` | Cliente por correo | 200 o 404 |
| POST | `/api/clientes` | Registrar cliente nuevo | 201 o 409 si correo duplicado |
| PATCH | `/api/clientes/{id}/puntos?cantidad=50` | Agregar puntos | 200 o 404 |
| PUT | `/api/clientes/{id}` | Actualizar cliente completo | 200 o 404 |
| DELETE | `/api/clientes/{id}` | Eliminar cliente | 204 o 404 |

> 💡 El endpoint POST devuelve **409 Conflict** (en lugar de 500) cuando se intenta registrar
> un correo que ya existe. Esto le da al cliente de la API un mensaje claro y útil.



---

## 🛠️ Paso 7 — Crear el DataLoader

### 7.1 — Crear el paquete `config`

1. **Clic derecho** en `com.michicafe.servicioclientes` → **"New"** → **"Package"**
2. Escribe `config` → **Enter**

### 7.2 — Crear la clase `DataLoader`

1. **Clic derecho** en `config` → **"New"** → **"Java Class"**
2. Escribe `DataLoader` → **"Class"** → **Enter**

### 7.3 — Código del DataLoader

```java
package com.michicafe.servicioclientes.config;

import com.michicafe.servicioclientes.model.Cliente;
import com.michicafe.servicioclientes.repository.ClienteRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component
public class DataLoader implements CommandLineRunner {

    @Autowired
    private ClienteRepository clienteRepository;

    @Override
    public void run(String... args) {
        if (clienteRepository.count() == 0) {
            System.out.println("🐱 Cargando clientes de prueba del Michi Café...");

            clienteRepository.save(new Cliente("Sofía Martínez", "sofia@email.com"));
            clienteRepository.save(new Cliente("Carlos López", "carlos@email.com"));
            clienteRepository.save(new Cliente("Ana García", "ana@email.com"));

            // Agregamos algunos puntos a Sofía manualmente
            clienteRepository.findByCorreo("sofia@email.com").ifPresent(sofia -> {
                sofia.setPuntosFidelidad(150);
                clienteRepository.save(sofia);
            });

            System.out.println("✅ Clientes cargados: "
                + clienteRepository.count() + " clientes en la base de datos");
        } else {
            System.out.println("📋 Ya hay "
                + clienteRepository.count() + " clientes registrados");
        }
    }
}
```

### 7.4 — Importar con Alt+Enter

- `Cliente` → `com.michicafe.servicioclientes.model.Cliente`
- `ClienteRepository` → `com.michicafe.servicioclientes.repository.ClienteRepository`
- `Autowired` → `org.springframework.beans.factory.annotation.Autowired`
- `CommandLineRunner` → `org.springframework.boot.CommandLineRunner`
- `Component` → `org.springframework.stereotype.Component`



---

## 🛠️ Paso 8 — Ejecutar y probar

### 8.1 — Verificar que MongoDB está corriendo

1. Presiona **Windows + R** → escribe `services.msc` → **Enter**
2. Busca **MongoDB Server** en la lista
3. Verifica que el estado sea **"En ejecución"**

### 8.2 — Ejecutar la aplicación

1. En el panel izquierdo, expande `src` → `main` → `java` → `com.michicafe.servicioclientes`
2. Haz **doble clic** en `ServicioClientesApplication`
3. Haz clic en el triángulo verde ▶ junto al método `main`
4. Selecciona **"Run 'ServicioClientesApplication'"**

### 8.3 — Verificar en la consola

Espera ver:

```
🐱 Cargando clientes de prueba del Michi Café...
✅ Clientes cargados: 3 clientes en la base de datos
...
Started ServicioClientesApplication in X seconds
```

---

### 8.4 — Probar con Postman

**PRUEBA 1 — GET: Ver todos los clientes**

```
Método: GET
URL:    http://localhost:8082/api/clientes
```

Respuesta esperada (200 OK):
```json
[
  {
    "id": "64a1b2c3d4e5f6a7b8c9d0e1",
    "nombre": "Sofía Martínez",
    "correo": "sofia@email.com",
    "puntosFidelidad": 150,
    "fechaRegistro": "2024-01-15T10:00:00"
  },
  {
    "id": "64a1b2c3d4e5f6a7b8c9d0e2",
    "nombre": "Carlos López",
    "correo": "carlos@email.com",
    "puntosFidelidad": 0,
    "fechaRegistro": "2024-01-15T10:00:00"
  }
]
```

---

**PRUEBA 2 — POST: Registrar un cliente nuevo**

1. Método: **POST**
2. URL: `http://localhost:8082/api/clientes`
3. **Body** → **raw** → **JSON**:

```json
{
  "nombre": "Miguel Torres",
  "correo": "miguel@email.com"
}
```

Respuesta esperada (201 Created):
```json
{
  "id": "64a1b2c3d4e5f6a7b8c9d0e9",
  "nombre": "Miguel Torres",
  "correo": "miguel@email.com",
  "puntosFidelidad": 0,
  "fechaRegistro": "2024-01-15T11:30:00"
}
```

---

**PRUEBA 3 — POST: Intentar registrar un correo duplicado**

```
Método: POST
URL:    http://localhost:8082/api/clientes
Body → raw → JSON:
```
```json
{
  "nombre": "Otra Sofía",
  "correo": "sofia@email.com"
}
```

Respuesta esperada (409 Conflict):
```json
{
  "error": "Ya existe un cliente registrado con el correo: sofia@email.com"
}
```

> 💡 El servidor rechaza el registro porque el correo ya está en uso. Así funciona la validación.

---

**PRUEBA 4 — GET: Buscar cliente por correo**

```
Método: GET
URL:    http://localhost:8082/api/clientes/correo/sofia@email.com
```

Respuesta esperada (200 OK): el objeto de Sofía con sus 150 puntos.

---

**PRUEBA 5 — PATCH: Agregar puntos a un cliente**

Copia el `id` de Carlos de la prueba 1.

```
Método: PATCH
URL:    http://localhost:8082/api/clientes/PEGA_ID_DE_CARLOS/puntos?cantidad=75
```

No necesitas body. Respuesta esperada (200 OK):
```json
{
  "id": "...",
  "nombre": "Carlos López",
  "correo": "carlos@email.com",
  "puntosFidelidad": 75,
  "fechaRegistro": "..."
}
```

Llama de nuevo con `?cantidad=25` y verás que acumula: ahora tendrá 100 puntos.

---

**PRUEBA 6 — PUT: Actualizar datos del cliente**

```
Método:  PUT
URL:     http://localhost:8082/api/clientes/PEGA_ID_DE_MIGUEL
Body → raw → JSON:
```
```json
{
  "nombre": "Miguel Torres Ruiz",
  "correo": "miguel.torres@email.com",
  "puntosFidelidad": 0,
  "fechaRegistro": "2024-01-15T11:30:00"
}
```

> ⚠️ En un PUT debes enviar todos los campos, no solo los que cambian.

---

**PRUEBA 7 — DELETE: Eliminar un cliente**

```
Método: DELETE
URL:    http://localhost:8082/api/clientes/PEGA_AQUI_EL_ID
```

Respuesta esperada: **204 No Content** (se eliminó correctamente).

---

## 🛠️ Paso 9 — Ver los datos en MongoDB Compass

1. Abre **MongoDB Compass** y conéctate a `mongodb://localhost:27017`
2. En el panel izquierdo busca la base de datos **`michicafe_clientes`**
3. Haz clic en la colección **`clientes`**
4. Verás los documentos con todos los campos, incluyendo `fechaRegistro` en formato ISO

---

## 🎉 ¡Felicidades!

Construiste el Servicio de Clientes completo del Michi Café.

### ¿Qué construiste?

- ✅ Proyecto Spring Boot 4.1.0 en puerto 8082 con base de datos `michicafe_clientes`
- ✅ Modelo `Cliente` con correo único (`@Indexed(unique = true)`) y fecha de registro
- ✅ `ClienteRepository` con métodos de búsqueda por correo, nombre y puntos
- ✅ `ClienteService` que valida correos duplicados y suma puntos de fidelidad
- ✅ `ClienteController` con 7 endpoints REST completos
- ✅ `DataLoader` con 3 clientes de prueba iniciales
- ✅ Pruebas en Postman incluyendo el caso del correo duplicado

### Estructura final del proyecto

```
servicio-clientes/
└── src/main/java/com/michicafe/servicioclientes/
    ├── ServicioClientesApplication.java
    ├── config/
    │   └── DataLoader.java
    ├── controller/
    │   └── ClienteController.java
    ├── service/
    │   └── ClienteService.java
    ├── repository/
    │   └── ClienteRepository.java
    └── model/
        └── Cliente.java
```

---

## ➡️ Siguiente paso

En el **[Proyecto P05 — Servicio de Pedidos](proyecto-05-servicio-pedidos.md)**
construirás el tercer microservicio: la API de pedidos que conecta bebidas y clientes,
el corazón del Michi Café.

> 🐾 "El Michi Café ya conoce a sus clientes. Pronto también gestionará sus pedidos."
