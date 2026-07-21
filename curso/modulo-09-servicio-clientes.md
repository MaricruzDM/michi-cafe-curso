# Módulo 09 — Microservicio de Clientes
## 🐱 Analogía: El registro de clientes fieles del Michi Café

---

## 🏠 Introducción

Es momento de construir nuestro primer microservicio real: el **Servicio de Clientes**.

Este servicio es responsable de todo lo relacionado con los clientes del Michi Café:
- Registrar clientes nuevos
- Consultar información de clientes
- Gestionar sus puntos de fidelidad
- Actualizar sus datos

Nada más. No sabe nada de bebidas ni de pedidos. Solo clientes.

### Analogía 🐾

En el Michi Café, el área de **Registro de Clientes Fieles** tiene su propio escritorio, su propio archivero y su propia persona encargada. Si quieres saber algo de un cliente, vas a ese escritorio. Si quieres hacer un pedido, vas a otro lado.

---

## 🛠️ Creando el proyecto

### Paso 1: Spring Initializr

Ve a **https://start.spring.io** y configura:

| Campo | Valor |
|-------|-------|
| Project | Maven |
| Language | Java |
| Spring Boot | 4.1.0 |
| Group | `com.michicafe` |
| Artifact | `servicio-clientes` |
| Packaging | Jar |
| Java | 17 |

Dependencias:
- `Spring Web`
- `Spring Data MongoDB`

Descarga, descomprime y abre en IntelliJ.

### Paso 2: Configuración

```properties
# src/main/resources/application.properties

spring.application.name=servicio-clientes
server.port=8082

# MongoDB — base de datos PROPIA del servicio de clientes
spring.data.mongodb.host=localhost
spring.data.mongodb.port=27017
spring.data.mongodb.database=michicafe-clientes
```

> 💡 Nota que la base de datos se llama `michicafe-clientes`, no `michicafe`. Cada microservicio tiene su propia base de datos.

---

## 💡 El modelo Cliente

```java
// src/main/java/com/michicafe/clientes/model/Cliente.java
package com.michicafe.clientes.model;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.index.Indexed;
import org.springframework.data.mongodb.core.mapping.Document;
import java.time.LocalDateTime;

@Document(collection = "clientes")
public class Cliente {

    @Id
    private String id;

    private String nombre;
    private String apellido;

    @Indexed(unique = true)
    private String correo;

    private String telefono;
    private int puntosFidelidad;
    private String nivel;           // "bronce", "plata", "oro", "platino"
    private LocalDateTime fechaRegistro;
    private boolean activo;

    public Cliente() {}

    public Cliente(String nombre, String apellido, String correo, String telefono) {
        this.nombre = nombre;
        this.apellido = apellido;
        this.correo = correo;
        this.telefono = telefono;
        this.puntosFidelidad = 0;
        this.nivel = "bronce";
        this.fechaRegistro = LocalDateTime.now();
        this.activo = true;
    }

    // Getters
    public String getId()                  { return id; }
    public String getNombre()              { return nombre; }
    public String getApellido()            { return apellido; }
    public String getCorreo()              { return correo; }
    public String getTelefono()            { return telefono; }
    public int getPuntosFidelidad()        { return puntosFidelidad; }
    public String getNivel()               { return nivel; }
    public LocalDateTime getFechaRegistro(){ return fechaRegistro; }
    public boolean isActivo()              { return activo; }

    // Setters
    public void setId(String id)                      { this.id = id; }
    public void setNombre(String nombre)              { this.nombre = nombre; }
    public void setApellido(String apellido)          { this.apellido = apellido; }
    public void setCorreo(String correo)              { this.correo = correo; }
    public void setTelefono(String telefono)          { this.telefono = telefono; }
    public void setPuntosFidelidad(int puntos)        { this.puntosFidelidad = puntos; }
    public void setNivel(String nivel)                { this.nivel = nivel; }
    public void setFechaRegistro(LocalDateTime fecha) { this.fechaRegistro = fecha; }
    public void setActivo(boolean activo)             { this.activo = activo; }
}
```

---

## 💡 El Repository

```java
// src/main/java/com/michicafe/clientes/repository/ClienteRepository.java
package com.michicafe.clientes.repository;

import com.michicafe.clientes.model.Cliente;
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.stereotype.Repository;
import java.util.List;
import java.util.Optional;

@Repository
public interface ClienteRepository extends MongoRepository<Cliente, String> {

    Optional<Cliente> findByCorreo(String correo);

    List<Cliente> findByActivoTrue();

    List<Cliente> findByNivel(String nivel);

    List<Cliente> findByNombreContainingIgnoreCaseOrApellidoContainingIgnoreCase(
            String nombre, String apellido);

    boolean existsByCorreo(String correo);

    List<Cliente> findByPuntosFidelidadGreaterThanEqual(int puntos);
}
```

---

## 💡 DTOs: objetos de transferencia de datos

Un **DTO** (Data Transfer Object) es un objeto que usamos para recibir o enviar datos en la API, separado del modelo de base de datos.

### Analogía 🐾

Cuando un cliente nuevo llega al Michi Café, llena un **formulario de registro** (DTO de entrada). Ese formulario no es igual a la ficha completa del archivero (modelo), porque la ficha tiene campos que el cliente no llena (como la fecha de registro o los puntos, que empiezan en 0).

```java
// src/main/java/com/michicafe/clientes/dto/ClienteRegistroDTO.java
package com.michicafe.clientes.dto;

// DTO para RECIBIR datos al registrar un cliente nuevo
public class ClienteRegistroDTO {
    private String nombre;
    private String apellido;
    private String correo;
    private String telefono;

    public ClienteRegistroDTO() {}

    // Getters y Setters
    public String getNombre()   { return nombre; }
    public String getApellido() { return apellido; }
    public String getCorreo()   { return correo; }
    public String getTelefono() { return telefono; }

    public void setNombre(String nombre)     { this.nombre = nombre; }
    public void setApellido(String apellido) { this.apellido = apellido; }
    public void setCorreo(String correo)     { this.correo = correo; }
    public void setTelefono(String telefono) { this.telefono = telefono; }
}
```

```java
// src/main/java/com/michicafe/clientes/dto/ClienteRespuestaDTO.java
package com.michicafe.clientes.dto;

// DTO para ENVIAR datos del cliente (no exponemos todos los campos internos)
public class ClienteRespuestaDTO {
    private String id;
    private String nombre;
    private String apellido;
    private String correo;
    private int puntosFidelidad;
    private String nivel;

    public ClienteRespuestaDTO() {}

    public ClienteRespuestaDTO(String id, String nombre, String apellido,
                                String correo, int puntos, String nivel) {
        this.id = id;
        this.nombre = nombre;
        this.apellido = apellido;
        this.correo = correo;
        this.puntosFidelidad = puntos;
        this.nivel = nivel;
    }

    // Getters
    public String getId()           { return id; }
    public String getNombre()       { return nombre; }
    public String getApellido()     { return apellido; }
    public String getCorreo()       { return correo; }
    public int getPuntosFidelidad() { return puntosFidelidad; }
    public String getNivel()        { return nivel; }
}
```

---

## 💡 El Service con lógica de negocio

```java
// src/main/java/com/michicafe/clientes/service/ClienteService.java
package com.michicafe.clientes.service;

import com.michicafe.clientes.dto.ClienteRegistroDTO;
import com.michicafe.clientes.dto.ClienteRespuestaDTO;
import com.michicafe.clientes.model.Cliente;
import com.michicafe.clientes.repository.ClienteRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;
import java.util.stream.Collectors;

@Service
public class ClienteService {

    @Autowired
    private ClienteRepository clienteRepository;

    // Registrar cliente nuevo
    public ClienteRespuestaDTO registrar(ClienteRegistroDTO dto) {
        if (clienteRepository.existsByCorreo(dto.getCorreo())) {
            throw new RuntimeException(
                "Ya existe un cliente con el correo: " + dto.getCorreo());
        }

        Cliente cliente = new Cliente(
            dto.getNombre(),
            dto.getApellido(),
            dto.getCorreo(),
            dto.getTelefono()
        );

        Cliente guardado = clienteRepository.save(cliente);
        return convertirADTO(guardado);
    }

    // Obtener todos los clientes activos
    public List<ClienteRespuestaDTO> obtenerTodos() {
        return clienteRepository.findByActivoTrue()
                .stream()
                .map(this::convertirADTO)
                .collect(Collectors.toList());
    }

    // Obtener por id
    public Optional<ClienteRespuestaDTO> obtenerPorId(String id) {
        return clienteRepository.findById(id)
                .map(this::convertirADTO);
    }

    // Obtener por correo
    public Optional<ClienteRespuestaDTO> obtenerPorCorreo(String correo) {
        return clienteRepository.findByCorreo(correo)
                .map(this::convertirADTO);
    }

    // Buscar por nombre o apellido
    public List<ClienteRespuestaDTO> buscar(String texto) {
        return clienteRepository
                .findByNombreContainingIgnoreCaseOrApellidoContainingIgnoreCase(texto, texto)
                .stream()
                .map(this::convertirADTO)
                .collect(Collectors.toList());
    }

    // Agregar puntos y recalcular nivel
    public Optional<ClienteRespuestaDTO> agregarPuntos(String id, int puntos) {
        return clienteRepository.findById(id).map(cliente -> {
            int nuevosPuntos = cliente.getPuntosFidelidad() + puntos;
            cliente.setPuntosFidelidad(nuevosPuntos);
            cliente.setNivel(calcularNivel(nuevosPuntos));
            return convertirADTO(clienteRepository.save(cliente));
        });
    }

    // Desactivar cliente (baja lógica, no se borra de la BD)
    public boolean desactivar(String id) {
        return clienteRepository.findById(id).map(cliente -> {
            cliente.setActivo(false);
            clienteRepository.save(cliente);
            return true;
        }).orElse(false);
    }

    // Calcular nivel según puntos
    private String calcularNivel(int puntos) {
        if (puntos >= 1000) return "platino";
        if (puntos >= 500)  return "oro";
        if (puntos >= 200)  return "plata";
        return "bronce";
    }

    // Convertir modelo a DTO de respuesta
    private ClienteRespuestaDTO convertirADTO(Cliente cliente) {
        return new ClienteRespuestaDTO(
            cliente.getId(),
            cliente.getNombre(),
            cliente.getApellido(),
            cliente.getCorreo(),
            cliente.getPuntosFidelidad(),
            cliente.getNivel()
        );
    }
}
```

---

## 💡 El Controller

```java
// src/main/java/com/michicafe/clientes/controller/ClienteController.java
package com.michicafe.clientes.controller;

import com.michicafe.clientes.dto.ClienteRegistroDTO;
import com.michicafe.clientes.dto.ClienteRespuestaDTO;
import com.michicafe.clientes.service.ClienteService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/clientes")
public class ClienteController {

    @Autowired
    private ClienteService clienteService;

    // GET /api/clientes
    @GetMapping
    public ResponseEntity<List<ClienteRespuestaDTO>> obtenerTodos() {
        return ResponseEntity.ok(clienteService.obtenerTodos());
    }

    // GET /api/clientes/{id}
    @GetMapping("/{id}")
    public ResponseEntity<ClienteRespuestaDTO> obtenerPorId(@PathVariable String id) {
        return clienteService.obtenerPorId(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // GET /api/clientes/correo/{correo}
    @GetMapping("/correo/{correo}")
    public ResponseEntity<ClienteRespuestaDTO> obtenerPorCorreo(@PathVariable String correo) {
        return clienteService.obtenerPorCorreo(correo)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // GET /api/clientes/buscar?texto=sofia
    @GetMapping("/buscar")
    public ResponseEntity<List<ClienteRespuestaDTO>> buscar(@RequestParam String texto) {
        return ResponseEntity.ok(clienteService.buscar(texto));
    }

    // POST /api/clientes
    @PostMapping
    public ResponseEntity<ClienteRespuestaDTO> registrar(@RequestBody ClienteRegistroDTO dto) {
        ClienteRespuestaDTO nuevo = clienteService.registrar(dto);
        return ResponseEntity.status(HttpStatus.CREATED).body(nuevo);
    }

    // PATCH /api/clientes/{id}/puntos?cantidad=50
    @PatchMapping("/{id}/puntos")
    public ResponseEntity<ClienteRespuestaDTO> agregarPuntos(
            @PathVariable String id,
            @RequestParam int cantidad) {
        return clienteService.agregarPuntos(id, cantidad)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // DELETE /api/clientes/{id}
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> desactivar(@PathVariable String id) {
        if (clienteService.desactivar(id)) {
            return ResponseEntity.noContent().build();
        }
        return ResponseEntity.notFound().build();
    }
}
```

---

## 💡 Manejo de errores del servicio

```java
// src/main/java/com/michicafe/clientes/exception/GlobalExceptionHandler.java
package com.michicafe.clientes.exception;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.time.LocalDateTime;
import java.util.HashMap;
import java.util.Map;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(RuntimeException.class)
    public ResponseEntity<Map<String, Object>> manejarRuntimeException(RuntimeException ex) {
        Map<String, Object> error = new HashMap<>();
        error.put("codigo", 400);
        error.put("mensaje", ex.getMessage());
        error.put("timestamp", LocalDateTime.now().toString());
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<Map<String, Object>> manejarExcepcion(Exception ex) {
        Map<String, Object> error = new HashMap<>();
        error.put("codigo", 500);
        error.put("mensaje", "Error interno del servicio de clientes");
        error.put("timestamp", LocalDateTime.now().toString());
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

---

## 💡 Estructura final del servicio

```
servicio-clientes/
└── src/main/java/com/michicafe/clientes/
    ├── ServicioClientesApplication.java
    ├── controller/
    │   └── ClienteController.java
    ├── service/
    │   └── ClienteService.java
    ├── repository/
    │   └── ClienteRepository.java
    ├── model/
    │   └── Cliente.java
    ├── dto/
    │   ├── ClienteRegistroDTO.java
    │   └── ClienteRespuestaDTO.java
    └── exception/
        └── GlobalExceptionHandler.java
```

---

## 💡 Probando el servicio

Con el servicio corriendo en el puerto 8082:

### Registrar un cliente nuevo

```
POST http://localhost:8082/api/clientes
Content-Type: application/json

{
  "nombre": "Sofía",
  "apellido": "García",
  "correo": "sofia@email.com",
  "telefono": "555-1234"
}
```

Respuesta (201 Created):
```json
{
  "id": "64a1b2c3d4e5f6a7b8c9d0e1",
  "nombre": "Sofía",
  "apellido": "García",
  "correo": "sofia@email.com",
  "puntosFidelidad": 0,
  "nivel": "bronce"
}
```

### Agregar puntos

```
PATCH http://localhost:8082/api/clientes/64a1b2c3.../puntos?cantidad=250
```

Respuesta:
```json
{
  "id": "64a1b2c3...",
  "nombre": "Sofía",
  "apellido": "García",
  "correo": "sofia@email.com",
  "puntosFidelidad": 250,
  "nivel": "plata"
}
```

---

## 🧪 Ejercicios del Módulo 9

### Ejercicio 1: Niveles de fidelidad

Agrega un endpoint `GET /api/clientes/nivel/{nivel}` que devuelva todos los clientes de un nivel específico (bronce, plata, oro, platino).

### Ejercicio 2: Actualizar datos

Agrega un endpoint `PUT /api/clientes/{id}` que permita actualizar el nombre, apellido y teléfono de un cliente (no el correo, que es único e inmutable).

### Ejercicio 3: Estadísticas

Agrega un endpoint `GET /api/clientes/estadisticas` que devuelva:
```json
{
  "totalClientes": 150,
  "clientesBronce": 80,
  "clientesPlata": 40,
  "clientesOro": 25,
  "clientesPlatino": 5
}
```

---

## ✅ Resumen del Módulo 9

| Concepto | ¿Qué es? | En el servicio de clientes |
|----------|----------|---------------------------|
| Microservicio independiente | Proyecto Spring Boot propio | Solo gestiona clientes |
| Base de datos propia | `michicafe-clientes` | Solo datos de clientes |
| DTO | Objeto para transferir datos | Formulario de registro vs ficha completa |
| Lógica de negocio | Reglas del dominio | Calcular nivel según puntos |
| Baja lógica | Desactivar sin borrar | El cliente "duerme" en el archivero |

---

## ➡️ Siguiente Módulo

En el **Módulo 10** construiremos el **Servicio de Pedidos**: el corazón del Michi Café. Gestionará todo el ciclo de vida de un pedido, desde que el cliente lo hace hasta que lo recibe.

> 🐾 "Un cliente bien registrado es un cliente que siempre vuelve al Michi Café."
