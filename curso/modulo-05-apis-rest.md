# Módulo 05 — APIs REST: El Menú Completo del Michi Café
## 🐱 Analogía: El sistema de pedidos en línea

---

## 🏠 Introducción

En el módulo anterior creamos nuestro primer servidor y aprendimos a responder peticiones `GET`.
Pero un café no solo muestra el menú: también recibe pedidos nuevos, actualiza pedidos existentes y los cancela.

En este módulo aprenderemos a construir una **API REST completa** con todas las operaciones posibles.

---

## 💡 ¿Qué es una API REST?

**API** significa *Application Programming Interface* (Interfaz de Programación de Aplicaciones).
Es un conjunto de reglas que define cómo dos sistemas se comunican entre sí.

**REST** (*Representational State Transfer*) es un estilo de arquitectura para diseñar esas APIs usando HTTP.

### Analogía 🐾

El Michi Café tiene un sistema de pedidos en línea. Los clientes pueden:
- **Ver el menú** (consultar información)
- **Hacer un pedido** (crear algo nuevo)
- **Modificar su pedido** (actualizar algo existente)
- **Cancelar su pedido** (eliminar algo)

Una API REST es exactamente eso: un sistema que permite hacer esas 4 operaciones sobre los datos de tu aplicación.

---

## 💡 Los métodos HTTP: los tipos de petición

HTTP tiene diferentes tipos de petición, cada uno con un propósito específico:

| Método HTTP | ¿Qué hace? | Analogía del Michi Café |
|-------------|-----------|------------------------|
| `GET` | Obtener/consultar datos | "¿Qué tienen en el menú?" |
| `POST` | Crear algo nuevo | "Quiero hacer un pedido nuevo" |
| `PUT` | Reemplazar algo completo | "Quiero cambiar todo mi pedido" |
| `PATCH` | Actualizar algo parcialmente | "Solo quiero cambiar el tamaño" |
| `DELETE` | Eliminar algo | "Quiero cancelar mi pedido" |

### El acrónimo CRUD

Estas operaciones se conocen como **CRUD**:

| CRUD | HTTP | Acción |
|------|------|--------|
| **C**reate | POST | Crear |
| **R**ead | GET | Leer |
| **U**pdate | PUT / PATCH | Actualizar |
| **D**elete | DELETE | Eliminar |

---

## 💡 Códigos de respuesta HTTP: lo que el gatito responde

Cuando el servidor responde, además de los datos, envía un **código de estado** que indica si todo salió bien o si hubo algún problema.

### Analogía 🐾

Es como la respuesta del gatito cajero:
- "Aquí está tu pedido" ✅ → todo bien
- "No encontré ese producto" ❌ → no existe
- "No puedo procesar eso" ⚠️ → error del cliente
- "Se me cayó el café" 💥 → error del servidor

| Código | Significado | Analogía |
|--------|-------------|----------|
| `200 OK` | Todo salió bien | "Aquí está tu pedido" |
| `201 Created` | Se creó algo nuevo | "Tu pedido fue registrado" |
| `204 No Content` | OK pero sin datos que devolver | "Tu pedido fue cancelado" |
| `400 Bad Request` | La petición tiene errores | "No entendí tu pedido" |
| `404 Not Found` | No se encontró lo que buscabas | "Ese producto no está en el menú" |
| `500 Internal Server Error` | Error en el servidor | "Se nos cayó el sistema" |

---

## 💡 ResponseEntity: el control total de la respuesta

`ResponseEntity` es una clase de Spring que nos permite controlar exactamente qué código de estado y qué datos devolvemos.

```java
// Devolver 200 OK con datos
return ResponseEntity.ok(bebida);

// Devolver 201 Created con datos
return ResponseEntity.status(HttpStatus.CREATED).body(nuevaBebida);

// Devolver 404 Not Found sin datos
return ResponseEntity.notFound().build();

// Devolver 204 No Content
return ResponseEntity.noContent().build();
```

---

## 💡 El modelo Bebida

```java
// src/main/java/com/michicafe/model/Bebida.java
package com.michicafe.model;

public class Bebida {
    private String id;
    private String nombre;
    private double precio;
    private String categoria;   // "caliente", "frio", "especial"
    private String descripcion;
    private boolean disponible;

    public Bebida() {} // Constructor vacío necesario para deserializar JSON

    public Bebida(String id, String nombre, double precio,
                  String categoria, String descripcion, boolean disponible) {
        this.id = id;
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
    public void setId(String id)               { this.id = id; }
    public void setNombre(String nombre)       { this.nombre = nombre; }
    public void setPrecio(double precio)       { this.precio = precio; }
    public void setCategoria(String categoria) { this.categoria = categoria; }
    public void setDescripcion(String desc)    { this.descripcion = desc; }
    public void setDisponible(boolean disp)    { this.disponible = disp; }
}
```

---

## 💡 El Service con todas las operaciones CRUD

```java
// src/main/java/com/michicafe/service/BebidaService.java
package com.michicafe.service;

import com.michicafe.model.Bebida;
import org.springframework.stereotype.Service;
import java.util.*;
import java.util.stream.Collectors;

@Service
public class BebidaService {

    // Simulamos una base de datos con un Map (clave: id, valor: bebida)
    // En el módulo 07 esto será reemplazado por MongoDB
    private Map<String, Bebida> baseDeDatos = new HashMap<>();
    private int contadorId = 1;

    public BebidaService() {
        agregarDatosDePrueba();
    }

    private void agregarDatosDePrueba() {
        guardar(new Bebida(null, "Espresso",
                35.00, "caliente", "Café concentrado y puro", true));
        guardar(new Bebida(null, "Latte de vainilla",
                45.50, "caliente", "Espresso con leche vaporizada y vainilla", true));
        guardar(new Bebida(null, "Cappuccino",
                48.00, "caliente", "Espresso con leche y espuma cremosa", true));
        guardar(new Bebida(null, "Matcha Latte",
                55.00, "caliente", "Té matcha japonés con leche de avena", true));
        guardar(new Bebida(null, "Cold Brew",
                50.00, "frio", "Café infusionado en frío por 12 horas", true));
        guardar(new Bebida(null, "Chai Latte",
                52.00, "caliente", "Té chai especiado con leche", false));
    }

    // CREATE
    public Bebida guardar(Bebida bebida) {
        String id = "BEB-" + contadorId++;
        bebida.setId(id);
        baseDeDatos.put(id, bebida);
        return bebida;
    }

    // READ: todas
    public List<Bebida> obtenerTodas() {
        return new ArrayList<>(baseDeDatos.values());
    }

    // READ: por id — Optional evita NullPointerException
    public Optional<Bebida> obtenerPorId(String id) {
        return Optional.ofNullable(baseDeDatos.get(id));
    }

    // READ: por categoría
    public List<Bebida> obtenerPorCategoria(String categoria) {
        return baseDeDatos.values().stream()
                .filter(b -> b.getCategoria().equalsIgnoreCase(categoria))
                .collect(Collectors.toList());
    }

    // READ: solo disponibles
    public List<Bebida> obtenerDisponibles() {
        return baseDeDatos.values().stream()
                .filter(Bebida::isDisponible)
                .collect(Collectors.toList());
    }

    // READ: por rango de precio
    public List<Bebida> obtenerPorRangoPrecio(double min, double max) {
        return baseDeDatos.values().stream()
                .filter(b -> b.getPrecio() >= min && b.getPrecio() <= max)
                .collect(Collectors.toList());
    }

    // UPDATE: reemplazar completo
    public Optional<Bebida> actualizar(String id, Bebida bebidaActualizada) {
        if (!baseDeDatos.containsKey(id)) return Optional.empty();
        bebidaActualizada.setId(id);
        baseDeDatos.put(id, bebidaActualizada);
        return Optional.of(bebidaActualizada);
    }

    // PATCH: solo disponibilidad
    public Optional<Bebida> cambiarDisponibilidad(String id, boolean disponible) {
        Bebida bebida = baseDeDatos.get(id);
        if (bebida == null) return Optional.empty();
        bebida.setDisponible(disponible);
        return Optional.of(bebida);
    }

    // DELETE
    public boolean eliminar(String id) {
        if (!baseDeDatos.containsKey(id)) return false;
        baseDeDatos.remove(id);
        return true;
    }
}
```

---

## 💡 El Controller completo con todos los endpoints

```java
// src/main/java/com/michicafe/controller/BebidaController.java
package com.michicafe.controller;

import com.michicafe.model.Bebida;
import com.michicafe.service.BebidaService;
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

    // ── GET ──────────────────────────────────────────────────────────────────

    // GET /api/bebidas → todas las bebidas
    @GetMapping
    public ResponseEntity<List<Bebida>> obtenerTodas() {
        return ResponseEntity.ok(bebidaService.obtenerTodas());
    }

    // GET /api/bebidas/BEB-1 → una bebida por id
    @GetMapping("/{id}")
    public ResponseEntity<Bebida> obtenerPorId(@PathVariable String id) {
        return bebidaService.obtenerPorId(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // GET /api/bebidas/disponibles → solo disponibles
    @GetMapping("/disponibles")
    public ResponseEntity<List<Bebida>> obtenerDisponibles() {
        return ResponseEntity.ok(bebidaService.obtenerDisponibles());
    }

    // GET /api/bebidas/categoria/caliente → por categoría
    @GetMapping("/categoria/{categoria}")
    public ResponseEntity<List<Bebida>> obtenerPorCategoria(@PathVariable String categoria) {
        return ResponseEntity.ok(bebidaService.obtenerPorCategoria(categoria));
    }

    // GET /api/bebidas/precio?min=40&max=55 → por rango de precio
    @GetMapping("/precio")
    public ResponseEntity<List<Bebida>> obtenerPorPrecio(
            @RequestParam(defaultValue = "0") double min,
            @RequestParam(defaultValue = "9999") double max) {
        return ResponseEntity.ok(bebidaService.obtenerPorRangoPrecio(min, max));
    }

    // ── POST ─────────────────────────────────────────────────────────────────

    // POST /api/bebidas → crear bebida nueva
    @PostMapping
    public ResponseEntity<Bebida> crear(@RequestBody Bebida bebida) {
        Bebida nueva = bebidaService.guardar(bebida);
        return ResponseEntity.status(HttpStatus.CREATED).body(nueva); // 201 Created
    }

    // ── PUT ──────────────────────────────────────────────────────────────────

    // PUT /api/bebidas/BEB-1 → reemplazar bebida completa
    @PutMapping("/{id}")
    public ResponseEntity<Bebida> actualizar(
            @PathVariable String id,
            @RequestBody Bebida bebida) {
        return bebidaService.actualizar(id, bebida)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // ── PATCH ────────────────────────────────────────────────────────────────

    // PATCH /api/bebidas/BEB-1/disponibilidad?valor=false
    @PatchMapping("/{id}/disponibilidad")
    public ResponseEntity<Bebida> cambiarDisponibilidad(
            @PathVariable String id,
            @RequestParam boolean valor) {
        return bebidaService.cambiarDisponibilidad(id, valor)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // ── DELETE ───────────────────────────────────────────────────────────────

    // DELETE /api/bebidas/BEB-1 → eliminar bebida
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> eliminar(@PathVariable String id) {
        if (bebidaService.eliminar(id)) {
            return ResponseEntity.noContent().build(); // 204 No Content
        }
        return ResponseEntity.notFound().build();      // 404 Not Found
    }
}
```

---

## 💡 Probando la API completa con Postman

### GET — Ver el menú completo

```
Método: GET
URL:    http://localhost:8080/api/bebidas
```

Respuesta (200 OK):
```json
[
  {
    "id": "BEB-1",
    "nombre": "Espresso",
    "precio": 35.0,
    "categoria": "caliente",
    "descripcion": "Café concentrado y puro",
    "disponible": true
  }
]
```

### POST — Agregar una bebida nueva

```
Método:  POST
URL:     http://localhost:8080/api/bebidas
Headers: Content-Type: application/json
Body:
```
```json
{
  "nombre": "Frappuccino de Caramelo",
  "precio": 65.00,
  "categoria": "frio",
  "descripcion": "Café frío con caramelo y crema batida",
  "disponible": true
}
```

Respuesta (201 Created):
```json
{
  "id": "BEB-7",
  "nombre": "Frappuccino de Caramelo",
  "precio": 65.0,
  "categoria": "frio",
  "descripcion": "Café frío con caramelo y crema batida",
  "disponible": true
}
```

### PUT — Actualizar una bebida completa

```
Método:  PUT
URL:     http://localhost:8080/api/bebidas/BEB-1
Headers: Content-Type: application/json
Body:
```
```json
{
  "nombre": "Espresso Doble",
  "precio": 42.00,
  "categoria": "caliente",
  "descripcion": "Doble shot de espresso para los más valientes",
  "disponible": true
}
```

Respuesta (200 OK): devuelve la bebida actualizada.

### PATCH — Cambiar disponibilidad

```
Método: PATCH
URL:    http://localhost:8080/api/bebidas/BEB-6/disponibilidad?valor=true
```

Respuesta (200 OK): devuelve la bebida con `disponible: true`.

### DELETE — Eliminar una bebida

```
Método: DELETE
URL:    http://localhost:8080/api/bebidas/BEB-7
```

Respuesta: `204 No Content` (sin cuerpo, solo confirma que se eliminó).

---

## 💡 Manejo de errores: cuando algo sale mal

Es importante manejar los errores de forma elegante y devolver mensajes claros.

### Modelo de respuesta de error

```java
// src/main/java/com/michicafe/model/ErrorResponse.java
package com.michicafe.model;

import java.time.LocalDateTime;

public class ErrorResponse {
    private int codigo;
    private String mensaje;
    private LocalDateTime timestamp;

    public ErrorResponse(int codigo, String mensaje) {
        this.codigo = codigo;
        this.mensaje = mensaje;
        this.timestamp = LocalDateTime.now();
    }

    public int getCodigo()              { return codigo; }
    public String getMensaje()          { return mensaje; }
    public LocalDateTime getTimestamp() { return timestamp; }
}
```

### Manejador global de excepciones

```java
// src/main/java/com/michicafe/exception/GlobalExceptionHandler.java
package com.michicafe.exception;

import com.michicafe.model.ErrorResponse;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice // Intercepta excepciones de todos los controllers
public class GlobalExceptionHandler {

    @ExceptionHandler(RuntimeException.class)
    public ResponseEntity<ErrorResponse> manejarRuntimeException(RuntimeException ex) {
        ErrorResponse error = new ErrorResponse(404, ex.getMessage());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> manejarExcepcionGeneral(Exception ex) {
        ErrorResponse error = new ErrorResponse(500,
                "Error interno en el Michi Café: " + ex.getMessage());
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

Ahora cuando algo falla, el cliente recibe un JSON claro:
```json
{
  "codigo": 404,
  "mensaje": "Bebida no encontrada en el menú del Michi Café",
  "timestamp": "2024-01-15T10:30:00"
}
```

---

## 💡 Tabla resumen de endpoints

| Método | Endpoint | Descripción | Código |
|--------|----------|-------------|--------|
| GET | `/api/bebidas` | Todas las bebidas | 200 |
| GET | `/api/bebidas/{id}` | Una bebida por ID | 200 / 404 |
| GET | `/api/bebidas/disponibles` | Solo disponibles | 200 |
| GET | `/api/bebidas/categoria/{cat}` | Por categoría | 200 |
| GET | `/api/bebidas/precio?min=40&max=55` | Por rango de precio | 200 |
| POST | `/api/bebidas` | Crear bebida nueva | 201 |
| PUT | `/api/bebidas/{id}` | Reemplazar bebida completa | 200 / 404 |
| PATCH | `/api/bebidas/{id}/disponibilidad` | Cambiar disponibilidad | 200 / 404 |
| DELETE | `/api/bebidas/{id}` | Eliminar bebida | 204 / 404 |

---

## 💡 Estructura del proyecto actualizada

```
michicafe/
└── src/main/java/com/michicafe/
    ├── MichicafeApplication.java
    ├── controller/
    │   ├── MichiController.java
    │   └── BebidaController.java
    ├── service/
    │   └── BebidaService.java
    ├── model/
    │   ├── Bebida.java
    │   └── ErrorResponse.java
    └── exception/
        └── GlobalExceptionHandler.java
```

---

## 🧪 Ejercicios del Módulo 5

### Ejercicio 1: API de Clientes

Crea la API REST completa para clientes con:
- Modelo `Cliente` (id, nombre, correo, puntosFidelidad)
- `ClienteService` con operaciones CRUD
- `ClienteController` con todos los endpoints
- Endpoint especial: `PATCH /api/clientes/{id}/puntos?cantidad=50`

### Ejercicio 2: Validaciones en el Service

Agrega validaciones en `BebidaService.guardar()`:
- El precio no puede ser negativo ni cero
- El nombre no puede estar vacío
- La categoría solo puede ser "caliente", "frio" o "especial"

Si alguna validación falla, lanza una `IllegalArgumentException` con un mensaje claro.

### Ejercicio 3: Búsqueda por nombre

Agrega el endpoint:
`GET /api/bebidas/buscar?nombre=latte`

Que devuelva todas las bebidas cuyo nombre contenga el texto buscado (sin importar mayúsculas/minúsculas).

---

## ✅ Resumen del Módulo 5

| Concepto | ¿Qué es? | Analogía del Michi Café |
|----------|----------|------------------------|
| API REST | Sistema de comunicación entre aplicaciones | El sistema de pedidos en línea |
| GET | Consultar datos | Ver el menú |
| POST | Crear datos nuevos | Hacer un pedido nuevo |
| PUT | Reemplazar datos completos | Cambiar todo el pedido |
| PATCH | Actualizar datos parciales | Solo cambiar el tamaño |
| DELETE | Eliminar datos | Cancelar el pedido |
| `@RequestBody` | Lee el JSON del cuerpo de la petición | El formulario de pedido |
| `ResponseEntity` | Control total de la respuesta HTTP | El gatito decide qué decir y cómo |
| Códigos HTTP | Indican el resultado de la operación | La respuesta del cajero |
| `Optional` | Valor que puede o no existir | "Puede que tengamos eso, puede que no" |

---

## ➡️ Siguiente Módulo

En el **Módulo 06** aprenderemos **MongoDB**: cómo funciona esta base de datos, qué son los documentos y colecciones, y cómo instalarla. Dejaremos de usar datos en memoria y empezaremos a guardar información de verdad.

> 🐾 "Una buena API es como un buen mesero: sabe exactamente qué preguntar, qué hacer con la respuesta y cómo decirte si algo salió mal."
