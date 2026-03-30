# Módulo 10 — Microservicio de Pedidos
## 🐱 Analogía: La cocina recibe órdenes

---

## 🏠 Introducción

El **Servicio de Pedidos** es el corazón del Michi Café. Gestiona todo el ciclo de vida de un pedido:

```
Cliente hace pedido → Pendiente → En preparación → Listo → Entregado
```

Este servicio necesita saber sobre clientes y bebidas, pero no tiene acceso directo a sus bases de datos. Aprenderemos a comunicarse con otros servicios de forma correcta.

### Analogía 🐾

La cocina del Michi Café recibe las órdenes del mesero. Para preparar un pedido, la cocina necesita saber:
- ¿Quién es el cliente? (pregunta al área de clientes)
- ¿Qué bebidas pidió? (pregunta al área de bebidas)

Pero la cocina no entra al archivero de clientes ni al menú directamente. Manda un mensaje y espera la respuesta.

---

## 🛠️ Creando el proyecto

### Spring Initializr

| Campo | Valor |
|-------|-------|
| Artifact | `servicio-pedidos` |
| Puerto | `8083` |
| Base de datos | `michicafe-pedidos` |

Dependencias:
- `Spring Web`
- `Spring Data MongoDB`

```properties
# application.properties
spring.application.name=servicio-pedidos
server.port=8083
spring.data.mongodb.host=localhost
spring.data.mongodb.port=27017
spring.data.mongodb.database=michicafe-pedidos
```

---

## 💡 El modelo Pedido

Un pedido tiene una estructura más compleja porque contiene una lista de ítems:

```java
// src/main/java/com/michicafe/pedidos/model/ItemPedido.java
package com.michicafe.pedidos.model;

// Representa una bebida dentro de un pedido
// No es un @Document porque vive DENTRO del documento Pedido
public class ItemPedido {
    private String bebidaId;
    private String nombreBebida;
    private double precio;
    private int cantidad;

    public ItemPedido() {}

    public ItemPedido(String bebidaId, String nombreBebida, double precio, int cantidad) {
        this.bebidaId = bebidaId;
        this.nombreBebida = nombreBebida;
        this.precio = precio;
        this.cantidad = cantidad;
    }

    public double getSubtotal() {
        return precio * cantidad;
    }

    // Getters
    public String getBebidaId()     { return bebidaId; }
    public String getNombreBebida() { return nombreBebida; }
    public double getPrecio()       { return precio; }
    public int getCantidad()        { return cantidad; }

    // Setters
    public void setBebidaId(String bebidaId)         { this.bebidaId = bebidaId; }
    public void setNombreBebida(String nombreBebida) { this.nombreBebida = nombreBebida; }
    public void setPrecio(double precio)             { this.precio = precio; }
    public void setCantidad(int cantidad)            { this.cantidad = cantidad; }
}
```

```java
// src/main/java/com/michicafe/pedidos/model/Pedido.java
package com.michicafe.pedidos.model;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;
import java.time.LocalDateTime;
import java.util.List;

@Document(collection = "pedidos")
public class Pedido {

    @Id
    private String id;

    private String clienteId;
    private String nombreCliente;       // Guardamos el nombre para no depender del servicio
    private List<ItemPedido> items;
    private double total;
    private EstadoPedido estado;
    private String notas;               // Instrucciones especiales ("sin azúcar", "para llevar")
    private LocalDateTime fechaCreacion;
    private LocalDateTime fechaActualizacion;

    public Pedido() {}

    public Pedido(String clienteId, String nombreCliente,
                  List<ItemPedido> items, String notas) {
        this.clienteId = clienteId;
        this.nombreCliente = nombreCliente;
        this.items = items;
        this.notas = notas;
        this.estado = EstadoPedido.PENDIENTE;
        this.fechaCreacion = LocalDateTime.now();
        this.fechaActualizacion = LocalDateTime.now();
        this.total = calcularTotal();
    }

    public double calcularTotal() {
        if (items == null) return 0;
        return items.stream()
                    .mapToDouble(ItemPedido::getSubtotal)
                    .sum();
    }

    // Getters
    public String getId()                        { return id; }
    public String getClienteId()                 { return clienteId; }
    public String getNombreCliente()             { return nombreCliente; }
    public List<ItemPedido> getItems()           { return items; }
    public double getTotal()                     { return total; }
    public EstadoPedido getEstado()              { return estado; }
    public String getNotas()                     { return notas; }
    public LocalDateTime getFechaCreacion()      { return fechaCreacion; }
    public LocalDateTime getFechaActualizacion() { return fechaActualizacion; }

    // Setters
    public void setId(String id)                               { this.id = id; }
    public void setClienteId(String clienteId)                 { this.clienteId = clienteId; }
    public void setNombreCliente(String nombreCliente)         { this.nombreCliente = nombreCliente; }
    public void setItems(List<ItemPedido> items)               { this.items = items; }
    public void setTotal(double total)                         { this.total = total; }
    public void setEstado(EstadoPedido estado)                 { this.estado = estado; }
    public void setNotas(String notas)                         { this.notas = notas; }
    public void setFechaCreacion(LocalDateTime fecha)          { this.fechaCreacion = fecha; }
    public void setFechaActualizacion(LocalDateTime fecha)     { this.fechaActualizacion = fecha; }
}
```

```java
// src/main/java/com/michicafe/pedidos/model/EstadoPedido.java
package com.michicafe.pedidos.model;

// Enum: lista de valores posibles para el estado del pedido
public enum EstadoPedido {
    PENDIENTE,      // Recién creado, esperando confirmación
    CONFIRMADO,     // Confirmado, esperando preparación
    EN_PREPARACION, // La cocina está preparando
    LISTO,          // Listo para entregar
    ENTREGADO,      // Entregado al cliente
    CANCELADO       // Cancelado
}
```

### ¿Qué es un `enum`?

Un `enum` es una lista de valores posibles para un campo. En lugar de usar un String libre (donde alguien podría escribir "entregadoo" con error), usamos un enum que solo permite valores válidos.

### Analogía 🐾

El estado de un pedido en el Michi Café solo puede ser uno de estos valores. No puede ser "más o menos listo" ni "casi entregado". El enum garantiza que solo usemos estados válidos.

---

## 💡 Los DTOs del pedido

```java
// src/main/java/com/michicafe/pedidos/dto/ItemPedidoDTO.java
package com.michicafe.pedidos.dto;

public class ItemPedidoDTO {
    private String bebidaId;
    private int cantidad;

    public ItemPedidoDTO() {}

    public String getBebidaId() { return bebidaId; }
    public int getCantidad()    { return cantidad; }
    public void setBebidaId(String bebidaId) { this.bebidaId = bebidaId; }
    public void setCantidad(int cantidad)    { this.cantidad = cantidad; }
}
```

```java
// src/main/java/com/michicafe/pedidos/dto/CrearPedidoDTO.java
package com.michicafe.pedidos.dto;

import java.util.List;

// Lo que recibimos cuando el cliente hace un pedido
public class CrearPedidoDTO {
    private String clienteId;
    private List<ItemPedidoDTO> items;
    private String notas;

    public CrearPedidoDTO() {}

    public String getClienteId()          { return clienteId; }
    public List<ItemPedidoDTO> getItems() { return items; }
    public String getNotas()              { return notas; }

    public void setClienteId(String clienteId)          { this.clienteId = clienteId; }
    public void setItems(List<ItemPedidoDTO> items)     { this.items = items; }
    public void setNotas(String notas)                  { this.notas = notas; }
}
```

---

## 💡 El Repository

```java
// src/main/java/com/michicafe/pedidos/repository/PedidoRepository.java
package com.michicafe.pedidos.repository;

import com.michicafe.pedidos.model.EstadoPedido;
import com.michicafe.pedidos.model.Pedido;
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.stereotype.Repository;
import java.util.List;

@Repository
public interface PedidoRepository extends MongoRepository<Pedido, String> {

    List<Pedido> findByClienteId(String clienteId);

    List<Pedido> findByEstado(EstadoPedido estado);

    List<Pedido> findByClienteIdOrderByFechaCreacionDesc(String clienteId);

    long countByEstado(EstadoPedido estado);
}
```

---

## 💡 El Service con lógica de negocio

```java
// src/main/java/com/michicafe/pedidos/service/PedidoService.java
package com.michicafe.pedidos.service;

import com.michicafe.pedidos.dto.CrearPedidoDTO;
import com.michicafe.pedidos.dto.ItemPedidoDTO;
import com.michicafe.pedidos.model.EstadoPedido;
import com.michicafe.pedidos.model.ItemPedido;
import com.michicafe.pedidos.model.Pedido;
import com.michicafe.pedidos.repository.PedidoRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

@Service
public class PedidoService {

    @Autowired
    private PedidoRepository pedidoRepository;

    // En el módulo 12 aprenderemos a llamar a otros servicios
    // Por ahora usamos datos simulados para los precios de bebidas

    public Pedido crear(CrearPedidoDTO dto) {
        // Construir los ítems del pedido
        // (En módulo 12 consultaremos el servicio de bebidas para obtener precios reales)
        List<ItemPedido> items = new ArrayList<>();
        for (ItemPedidoDTO itemDTO : dto.getItems()) {
            // Simulamos el precio hasta conectar con servicio-bebidas
            ItemPedido item = new ItemPedido(
                itemDTO.getBebidaId(),
                "Bebida " + itemDTO.getBebidaId(), // nombre simulado
                45.00,                              // precio simulado
                itemDTO.getCantidad()
            );
            items.add(item);
        }

        // Simulamos el nombre del cliente hasta conectar con servicio-clientes
        Pedido pedido = new Pedido(
            dto.getClienteId(),
            "Cliente " + dto.getClienteId(), // nombre simulado
            items,
            dto.getNotas()
        );

        return pedidoRepository.save(pedido);
    }

    public List<Pedido> obtenerTodos() {
        return pedidoRepository.findAll();
    }

    public Optional<Pedido> obtenerPorId(String id) {
        return pedidoRepository.findById(id);
    }

    public List<Pedido> obtenerPorCliente(String clienteId) {
        return pedidoRepository.findByClienteIdOrderByFechaCreacionDesc(clienteId);
    }

    public List<Pedido> obtenerPorEstado(String estado) {
        try {
            EstadoPedido estadoEnum = EstadoPedido.valueOf(estado.toUpperCase());
            return pedidoRepository.findByEstado(estadoEnum);
        } catch (IllegalArgumentException e) {
            throw new RuntimeException("Estado inválido: " + estado +
                ". Valores válidos: PENDIENTE, CONFIRMADO, EN_PREPARACION, LISTO, ENTREGADO, CANCELADO");
        }
    }

    public Optional<Pedido> avanzarEstado(String id) {
        return pedidoRepository.findById(id).map(pedido -> {
            EstadoPedido nuevoEstado = siguienteEstado(pedido.getEstado());
            pedido.setEstado(nuevoEstado);
            pedido.setFechaActualizacion(LocalDateTime.now());
            return pedidoRepository.save(pedido);
        });
    }

    public Optional<Pedido> cancelar(String id) {
        return pedidoRepository.findById(id).map(pedido -> {
            if (pedido.getEstado() == EstadoPedido.ENTREGADO) {
                throw new RuntimeException("No se puede cancelar un pedido ya entregado");
            }
            pedido.setEstado(EstadoPedido.CANCELADO);
            pedido.setFechaActualizacion(LocalDateTime.now());
            return pedidoRepository.save(pedido);
        });
    }

    // Determina cuál es el siguiente estado en el flujo
    private EstadoPedido siguienteEstado(EstadoPedido estadoActual) {
        switch (estadoActual) {
            case PENDIENTE:      return EstadoPedido.CONFIRMADO;
            case CONFIRMADO:     return EstadoPedido.EN_PREPARACION;
            case EN_PREPARACION: return EstadoPedido.LISTO;
            case LISTO:          return EstadoPedido.ENTREGADO;
            default:
                throw new RuntimeException(
                    "El pedido ya está en estado final: " + estadoActual);
        }
    }
}
```

---

## 💡 El Controller

```java
// src/main/java/com/michicafe/pedidos/controller/PedidoController.java
package com.michicafe.pedidos.controller;

import com.michicafe.pedidos.dto.CrearPedidoDTO;
import com.michicafe.pedidos.model.Pedido;
import com.michicafe.pedidos.service.PedidoService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/pedidos")
public class PedidoController {

    @Autowired
    private PedidoService pedidoService;

    // GET /api/pedidos
    @GetMapping
    public ResponseEntity<List<Pedido>> obtenerTodos() {
        return ResponseEntity.ok(pedidoService.obtenerTodos());
    }

    // GET /api/pedidos/{id}
    @GetMapping("/{id}")
    public ResponseEntity<Pedido> obtenerPorId(@PathVariable String id) {
        return pedidoService.obtenerPorId(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // GET /api/pedidos/cliente/{clienteId}
    @GetMapping("/cliente/{clienteId}")
    public ResponseEntity<List<Pedido>> obtenerPorCliente(@PathVariable String clienteId) {
        return ResponseEntity.ok(pedidoService.obtenerPorCliente(clienteId));
    }

    // GET /api/pedidos/estado/PENDIENTE
    @GetMapping("/estado/{estado}")
    public ResponseEntity<List<Pedido>> obtenerPorEstado(@PathVariable String estado) {
        return ResponseEntity.ok(pedidoService.obtenerPorEstado(estado));
    }

    // POST /api/pedidos
    @PostMapping
    public ResponseEntity<Pedido> crear(@RequestBody CrearPedidoDTO dto) {
        Pedido nuevo = pedidoService.crear(dto);
        return ResponseEntity.status(HttpStatus.CREATED).body(nuevo);
    }

    // PATCH /api/pedidos/{id}/avanzar → avanza al siguiente estado
    @PatchMapping("/{id}/avanzar")
    public ResponseEntity<Pedido> avanzarEstado(@PathVariable String id) {
        return pedidoService.avanzarEstado(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // PATCH /api/pedidos/{id}/cancelar
    @PatchMapping("/{id}/cancelar")
    public ResponseEntity<Pedido> cancelar(@PathVariable String id) {
        return pedidoService.cancelar(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }
}
```

---

## 💡 Probando el flujo completo de un pedido

### 1. Crear un pedido

```
POST http://localhost:8083/api/pedidos
Content-Type: application/json

{
  "clienteId": "64a1b2c3d4e5f6a7b8c9d0e1",
  "items": [
    { "bebidaId": "BEB-1", "cantidad": 1 },
    { "bebidaId": "BEB-3", "cantidad": 2 }
  ],
  "notas": "Sin azúcar en el cappuccino"
}
```

Respuesta (201 Created):
```json
{
  "id": "64a1b2c3d4e5f6a7b8c9d0f1",
  "clienteId": "64a1b2c3...",
  "nombreCliente": "Cliente 64a1b2c3...",
  "items": [
    { "bebidaId": "BEB-1", "nombreBebida": "Bebida BEB-1", "precio": 45.0, "cantidad": 1, "subtotal": 45.0 },
    { "bebidaId": "BEB-3", "nombreBebida": "Bebida BEB-3", "precio": 45.0, "cantidad": 2, "subtotal": 90.0 }
  ],
  "total": 135.0,
  "estado": "PENDIENTE",
  "notas": "Sin azúcar en el cappuccino",
  "fechaCreacion": "2024-01-15T11:30:00"
}
```

### 2. Avanzar el estado del pedido

```
PATCH http://localhost:8083/api/pedidos/64a1b2c3.../avanzar
```
Estado: `PENDIENTE → CONFIRMADO`

```
PATCH http://localhost:8083/api/pedidos/64a1b2c3.../avanzar
```
Estado: `CONFIRMADO → EN_PREPARACION`

```
PATCH http://localhost:8083/api/pedidos/64a1b2c3.../avanzar
```
Estado: `EN_PREPARACION → LISTO`

```
PATCH http://localhost:8083/api/pedidos/64a1b2c3.../avanzar
```
Estado: `LISTO → ENTREGADO`

---

## 🧪 Ejercicios del Módulo 10

### Ejercicio 1: Dashboard de la cocina

Agrega un endpoint `GET /api/pedidos/cocina` que devuelva todos los pedidos en estado `CONFIRMADO` o `EN_PREPARACION`, ordenados por fecha de creación (los más antiguos primero).

### Ejercicio 2: Estadísticas del día

Agrega `GET /api/pedidos/estadisticas` que devuelva:
```json
{
  "totalPedidos": 45,
  "pendientes": 3,
  "enPreparacion": 5,
  "entregados": 35,
  "cancelados": 2,
  "ingresoTotal": 4500.50
}
```

### Ejercicio 3: Validaciones

Agrega validaciones en `PedidoService.crear()`:
- La lista de ítems no puede estar vacía
- La cantidad de cada ítem debe ser mayor a 0
- El `clienteId` no puede estar vacío

---

## ✅ Resumen del Módulo 10

| Concepto | ¿Qué es? | En el servicio de pedidos |
|----------|----------|--------------------------|
| `enum` | Lista de valores posibles | Los estados del pedido |
| Documentos embebidos | Objetos dentro de un documento | Los ítems dentro del pedido |
| Ciclo de vida | Flujo de estados de un objeto | PENDIENTE → ENTREGADO |
| Datos desnormalizados | Guardar datos de otros servicios | Guardar el nombre del cliente en el pedido |

---

## ➡️ Siguiente Módulo

En el **Módulo 11** construiremos el **Servicio de Bebidas** como microservicio independiente. Luego en el módulo 12 aprenderemos a conectar los tres servicios entre sí.

> 🐾 "La cocina del Michi Café nunca pierde un pedido. Cada orden tiene su estado y su historia."
