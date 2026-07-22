# Proyecto P05 — Servicio de Pedidos
## El corazón del Michi Café

> ?? Referencias teóricas:
> - [Módulo 05 — APIs REST](modulo-05-apis-rest.md)
> - [Módulo 07 — Spring + MongoDB](modulo-07-spring-mongodb.md)
> - [Módulo 12 — Comunicación entre servicios](modulo-12-comunicacion.md)

---

## ?? ¿Qué vamos a construir?

El **Servicio de Pedidos** conecta clientes con bebidas. Cuando un cliente hace un pedido:
1. Verifica que el cliente exista (llama al Servicio de Clientes en 8082)
2. Verifica que cada bebida exista y esté disponible (llama al Servicio de Bebidas en 8081)
3. Calcula el total
4. Guarda el pedido en su propia base de datos MongoDB
5. Al marcar como entregado, agrega puntos al cliente automáticamente

**?? Importante:** Los servicios de Bebidas (8081) y Clientes (8082) deben estar corriendo.

**Tiempo estimado:** 90-120 minutos

---

## ??? Paso 1: Crear el proyecto en Spring Initializr

1. Ve a **https://start.spring.io**
2. Configura:

| Campo | Valor |
|-------|-------|
| Project | Maven |
| Language | Java |
| Spring Boot | 4.1.0 |
| Group | com.michicafe |
| Artifact | servicio-pedidos |
| Package name | com.michicafe.pedidos |
| Packaging | Jar |
| Java | 17 |

3. Dependencias: **Spring Web**, **Spring Data MongoDB**, **Spring Boot Actuator**
4. Clic en **GENERATE** ? extrae en `C:\Proyectos\servicio-pedidos`

---

## ??? Paso 2: Abrir en IntelliJ y configurar

1. IntelliJ ? **Open** ? selecciona `C:\Proyectos\servicio-pedidos` ? **OK**
2. Espera que carguen las dependencias

### Configurar application.properties

Expande `src ? main ? resources` ? doble clic en `application.properties`:

```properties
server.port=8083
spring.application.name=servicio-pedidos

# Con ${VARIABLE:valor_por_defecto}: usa la variable de entorno si existe (Docker),
# si no, usa el valor por defecto (IntelliJ local)
spring.data.mongodb.host=${SPRING_DATA_MONGODB_HOST:localhost}
spring.data.mongodb.port=${SPRING_DATA_MONGODB_PORT:27017}
spring.data.mongodb.database=${SPRING_DATA_MONGODB_DATABASE:michicafe_pedidos}

servicio.bebidas.url=${SERVICIO_BEBIDAS_URL:http://localhost:8081}
servicio.clientes.url=${SERVICIO_CLIENTES_URL:http://localhost:8082}
logging.level.com.michicafe=INFO
```

Guarda con **Ctrl + S**.

---

## ??? Paso 3: Crear RestTemplateConfig

`RestTemplate` permite hacer llamadas HTTP a otros servicios.

1. Clic derecho sobre `com.michicafe.pedidos` ? **New** ? **Package** ? escribe `config` ? Enter
2. Clic derecho sobre `config` ? **New** ? **Java Class** ? escribe `RestTemplateConfig` ? **Class** ? Enter

```java
package com.michicafe.pedidos.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class RestTemplateConfig {

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

**Alt + Enter** en los subrayados rojos para importar. **Ctrl + S** para guardar.

---

## ??? Paso 4: Crear el modelo ItemPedido

1. Clic derecho sobre `com.michicafe.pedidos` ? **New** ? **Package** ? `model` ? Enter
2. Clic derecho sobre `model` ? **New** ? **Java Class** ? `ItemPedido` ? **Class** ? Enter

```java
package com.michicafe.pedidos.model;

public class ItemPedido {

    private String bebidaId;
    private String nombreBebida;
    private double precio;
    private int cantidad;

    public ItemPedido() {}

    public ItemPedido(String bebidaId, String nombreBebida,
                      double precio, int cantidad) {
        this.bebidaId = bebidaId;
        this.nombreBebida = nombreBebida;
        this.precio = precio;
        this.cantidad = cantidad;
    }

    public double getSubtotal() { return precio * cantidad; }

    public String getBebidaId()     { return bebidaId; }
    public String getNombreBebida() { return nombreBebida; }
    public double getPrecio()       { return precio; }
    public int getCantidad()        { return cantidad; }

    public void setBebidaId(String bebidaId)       { this.bebidaId = bebidaId; }
    public void setNombreBebida(String nombre)     { this.nombreBebida = nombre; }
    public void setPrecio(double precio)           { this.precio = precio; }
    public void setCantidad(int cantidad)          { this.cantidad = cantidad; }
}
```

**Ctrl + S**.

---

## ??? Paso 5: Crear el modelo Pedido

1. Clic derecho sobre `model` ? **New** ? **Java Class** ? `Pedido` ? **Class** ? Enter

```java
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
    private String clienteNombre;
    private List<ItemPedido> items;
    private double total;
    private String estado;   // pendiente ? preparando ? listo ? entregado / cancelado
    private boolean paraLlevar;
    private String notas;
    private LocalDateTime fecha;

    public Pedido() {}

    public String getId()              { return id; }
    public String getClienteId()       { return clienteId; }
    public String getClienteNombre()   { return clienteNombre; }
    public List<ItemPedido> getItems() { return items; }
    public double getTotal()           { return total; }
    public String getEstado()          { return estado; }
    public boolean isParaLlevar()      { return paraLlevar; }
    public String getNotas()           { return notas; }
    public LocalDateTime getFecha()    { return fecha; }

    public void setId(String id)                  { this.id = id; }
    public void setClienteId(String clienteId)    { this.clienteId = clienteId; }
    public void setClienteNombre(String nombre)   { this.clienteNombre = nombre; }
    public void setItems(List<ItemPedido> items)  { this.items = items; }
    public void setTotal(double total)            { this.total = total; }
    public void setEstado(String estado)          { this.estado = estado; }
    public void setParaLlevar(boolean v)          { this.paraLlevar = v; }
    public void setNotas(String notas)            { this.notas = notas; }
    public void setFecha(LocalDateTime fecha)     { this.fecha = fecha; }
}
```

**Alt + Enter** para importar. **Ctrl + S**.

---

## ??? Paso 6: Crear el DTO SolicitudPedido

Un DTO es una clase que solo sirve para recibir datos del cliente.

1. Clic derecho sobre `model` ? **New** ? **Java Class** ? `SolicitudPedido` ? **Class** ? Enter

```java
package com.michicafe.pedidos.model;

import java.util.List;

public class SolicitudPedido {

    private String clienteId;
    private List<ItemSolicitud> items;
    private boolean paraLlevar;
    private String notas;

    public static class ItemSolicitud {
        private String bebidaId;
        private int cantidad;

        public String getBebidaId() { return bebidaId; }
        public int getCantidad()    { return cantidad; }
        public void setBebidaId(String v) { this.bebidaId = v; }
        public void setCantidad(int v)    { this.cantidad = v; }
    }

    public String getClienteId()          { return clienteId; }
    public List<ItemSolicitud> getItems() { return items; }
    public boolean isParaLlevar()         { return paraLlevar; }
    public String getNotas()              { return notas; }

    public void setClienteId(String v)          { this.clienteId = v; }
    public void setItems(List<ItemSolicitud> v) { this.items = v; }
    public void setParaLlevar(boolean v)        { this.paraLlevar = v; }
    public void setNotas(String v)              { this.notas = v; }
}
```

**Ctrl + S**.

---

## ??? Paso 7: Crear el Repository

1. Clic derecho sobre `com.michicafe.pedidos` ? **New** ? **Package** ? `repository` ? Enter
2. Clic derecho sobre `repository` ? **New** ? **Java Class** ? `PedidoRepository`
3. ?? Selecciona **Interface** (no Class) ? Enter

```java
package com.michicafe.pedidos.repository;

import com.michicafe.pedidos.model.Pedido;
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.stereotype.Repository;
import java.util.List;

@Repository
public interface PedidoRepository extends MongoRepository<Pedido, String> {

    List<Pedido> findByClienteId(String clienteId);

    List<Pedido> findByEstado(String estado);

    List<Pedido> findByClienteIdAndEstado(String clienteId, String estado);
}
```

**Alt + Enter** para importar. **Ctrl + S**.

---

## ??? Paso 8: Crear el Service

1. Clic derecho sobre `com.michicafe.pedidos` ? **New** ? **Package** ? `service` ? Enter
2. Clic derecho sobre `service` ? **New** ? **Java Class** ? `PedidoService` ? **Class** ? Enter

```java
package com.michicafe.pedidos.service;

import com.michicafe.pedidos.model.*;
import com.michicafe.pedidos.repository.PedidoRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

import java.time.LocalDateTime;
import java.util.*;

@Service
public class PedidoService {

    @Autowired
    private PedidoRepository pedidoRepository;

    @Autowired
    private RestTemplate restTemplate;

    @Value("${servicio.bebidas.url}")
    private String bebidaUrl;

    @Value("${servicio.clientes.url}")
    private String clienteUrl;

    public Pedido crearPedido(SolicitudPedido solicitud) {

        // 1. Verificar cliente
        Map<String, Object> clienteData;
        try {
            clienteData = restTemplate.getForObject(
                clienteUrl + "/api/clientes/" + solicitud.getClienteId(), Map.class);
        } catch (Exception e) {
            throw new RuntimeException("Cliente no encontrado: " + solicitud.getClienteId());
        }
        if (clienteData == null) {
            throw new RuntimeException("Cliente no encontrado: " + solicitud.getClienteId());
        }

        // 2. Verificar bebidas y construir ítems
        List<ItemPedido> items = new ArrayList<>();
        for (SolicitudPedido.ItemSolicitud itemSol : solicitud.getItems()) {
            Map<String, Object> bebidaData;
            try {
                bebidaData = restTemplate.getForObject(
                    bebidaUrl + "/api/bebidas/" + itemSol.getBebidaId(), Map.class);
            } catch (Exception e) {
                throw new RuntimeException("Bebida no encontrada: " + itemSol.getBebidaId());
            }
            if (bebidaData == null) {
                throw new RuntimeException("Bebida no encontrada: " + itemSol.getBebidaId());
            }
            Boolean disponible = (Boolean) bebidaData.get("disponible");
            if (disponible == null || !disponible) {
                throw new RuntimeException(
                    "La bebida '" + bebidaData.get("nombre") + "' no está disponible");
            }
            double precio = ((Number) bebidaData.get("precio")).doubleValue();
            items.add(new ItemPedido(
                itemSol.getBebidaId(),
                (String) bebidaData.get("nombre"),
                precio, itemSol.getCantidad()));
        }

        // 3. Calcular total y guardar
        double total = items.stream().mapToDouble(ItemPedido::getSubtotal).sum();

        Pedido pedido = new Pedido();
        pedido.setClienteId(solicitud.getClienteId());
        pedido.setClienteNombre((String) clienteData.get("nombre"));
        pedido.setItems(items);
        pedido.setTotal(total);
        pedido.setEstado("pendiente");
        pedido.setParaLlevar(solicitud.isParaLlevar());
        pedido.setNotas(solicitud.getNotas());
        pedido.setFecha(LocalDateTime.now());

        return pedidoRepository.save(pedido);
    }

    public List<Pedido> obtenerTodos()                        { return pedidoRepository.findAll(); }
    public Optional<Pedido> obtenerPorId(String id)          { return pedidoRepository.findById(id); }
    public List<Pedido> obtenerPorCliente(String clienteId)  { return pedidoRepository.findByClienteId(clienteId); }
    public List<Pedido> obtenerPorEstado(String estado)      { return pedidoRepository.findByEstado(estado); }

    public Optional<Pedido> cambiarEstado(String id, String nuevoEstado) {
        return pedidoRepository.findById(id).map(pedido -> {
            pedido.setEstado(nuevoEstado);
            if ("entregado".equalsIgnoreCase(nuevoEstado)) {
                int puntos = (int) pedido.getTotal() / 10;
                try {
                    restTemplate.patchForObject(
                        clienteUrl + "/api/clientes/" + pedido.getClienteId()
                        + "/puntos?cantidad=" + puntos, null, Map.class);
                    System.out.println("? " + puntos + " puntos ? " + pedido.getClienteNombre());
                } catch (Exception e) {
                    System.out.println("?? No se agregaron puntos: " + e.getMessage());
                }
            }
            return pedidoRepository.save(pedido);
        });
    }

    public Optional<Pedido> cancelarPedido(String id) {
        return pedidoRepository.findById(id).map(pedido -> {
            if (!"pendiente".equalsIgnoreCase(pedido.getEstado())) {
                throw new RuntimeException("Solo se cancelan pedidos en estado 'pendiente'");
            }
            pedido.setEstado("cancelado");
            return pedidoRepository.save(pedido);
        });
    }
}
```

**Alt + Enter** para importar. **Ctrl + S**.

---

## ??? Paso 9: Crear el Controller

1. Clic derecho sobre `com.michicafe.pedidos` ? **New** ? **Package** ? `controller` ? Enter
2. Clic derecho sobre `controller` ? **New** ? **Java Class** ? `PedidoController` ? **Class** ? Enter

```java
package com.michicafe.pedidos.controller;

import com.michicafe.pedidos.model.Pedido;
import com.michicafe.pedidos.model.SolicitudPedido;
import com.michicafe.pedidos.service.PedidoService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;
import java.util.Map;

@RestController
@RequestMapping("/api/pedidos")
public class PedidoController {

    @Autowired
    private PedidoService pedidoService;

    @GetMapping
    public ResponseEntity<List<Pedido>> obtenerTodos() {
        return ResponseEntity.ok(pedidoService.obtenerTodos());
    }

    @GetMapping("/{id}")
    public ResponseEntity<Pedido> obtenerPorId(@PathVariable String id) {
        return pedidoService.obtenerPorId(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    @GetMapping("/cliente/{clienteId}")
    public ResponseEntity<List<Pedido>> obtenerPorCliente(@PathVariable String clienteId) {
        return ResponseEntity.ok(pedidoService.obtenerPorCliente(clienteId));
    }

    @GetMapping("/estado/{estado}")
    public ResponseEntity<List<Pedido>> obtenerPorEstado(@PathVariable String estado) {
        return ResponseEntity.ok(pedidoService.obtenerPorEstado(estado));
    }

    @PostMapping
    public ResponseEntity<?> crear(@RequestBody SolicitudPedido solicitud) {
        try {
            return ResponseEntity.status(HttpStatus.CREATED)
                    .body(pedidoService.crearPedido(solicitud));
        } catch (RuntimeException e) {
            return ResponseEntity.badRequest().body(Map.of("error", e.getMessage()));
        }
    }

    @PatchMapping("/{id}/estado")
    public ResponseEntity<?> cambiarEstado(@PathVariable String id,
                                           @RequestParam String valor) {
        try {
            return pedidoService.cambiarEstado(id, valor)
                    .map(ResponseEntity::ok)
                    .orElse(ResponseEntity.notFound().build());
        } catch (RuntimeException e) {
            return ResponseEntity.badRequest().body(Map.of("error", e.getMessage()));
        }
    }

    @PatchMapping("/{id}/cancelar")
    public ResponseEntity<?> cancelar(@PathVariable String id) {
        try {
            return pedidoService.cancelarPedido(id)
                    .map(ResponseEntity::ok)
                    .orElse(ResponseEntity.notFound().build());
        } catch (RuntimeException e) {
            return ResponseEntity.badRequest().body(Map.of("error", e.getMessage()));
        }
    }
}
```

**Alt + Enter** para importar. **Ctrl + S**.

---

## ??? Paso 10: Ejecutar y probar

### 10.1 — Arrancar los 3 servicios

Debes tener los 3 servicios corriendo al mismo tiempo. En IntelliJ:

1. Abre el proyecto `servicio-bebidas` ? ejecuta `BebidasApplication` (puerto 8081)
2. Abre el proyecto `servicio-clientes` ? ejecuta `ServicioClientesApplication` (puerto 8082)
3. Abre el proyecto `servicio-pedidos` ? ejecuta `ServicioPedidosApplication` (puerto 8083)

> ?? El nombre exacto de la clase principal lo puedes ver en
> `src ? main ? java ? com.michicafe.pedidos` — busca el archivo que termina en `Application.java`.
> IntelliJ también muestra un triángulo verde ? junto al método `main` para ejecutarlo.

> ?? IntelliJ permite tener múltiples proyectos abiertos en ventanas separadas.
> Usa **File ? Open** y elige "Open in new window" para abrir cada uno.

Verifica que los 3 están corriendo:
- `http://localhost:8081/api/bebidas` ? responde con el menú
- `http://localhost:8082/api/clientes` ? responde con los clientes
- `http://localhost:8083/api/pedidos` ? responde con lista vacía `[]`

### 10.2 — Obtener IDs necesarios

Antes de crear un pedido necesitas los IDs reales de tu base de datos.

**Obtener ID de un cliente:**
```
GET http://localhost:8082/api/clientes
```
Copia el `id` de Sofía.

**Obtener IDs de bebidas:**
```
GET http://localhost:8081/api/bebidas
```
Copia el `id` del Latte de vainilla y del Matcha Latte.

### 10.3 — Crear el primer pedido

```
Método:  POST
URL:     http://localhost:8083/api/pedidos
Body ? raw ? JSON:
```

```json
{
  "clienteId": "PEGA_AQUI_EL_ID_DE_SOFIA",
  "items": [
    {
      "bebidaId": "PEGA_AQUI_ID_LATTE_VAINILLA",
      "cantidad": 1
    },
    {
      "bebidaId": "PEGA_AQUI_ID_MATCHA_LATTE",
      "cantidad": 2
    }
  ],
  "paraLlevar": false,
  "notas": "Sin azúcar en el matcha por favor"
}
```

Respuesta esperada (201 Created):
```json
{
  "id": "64a1b2c3...",
  "clienteId": "...",
  "clienteNombre": "Sofía Martínez",
  "items": [
    { "bebidaId": "...", "nombreBebida": "Latte de vainilla", "precio": 45.5, "cantidad": 1, "subtotal": 45.5 },
    { "bebidaId": "...", "nombreBebida": "Matcha Latte", "precio": 55.0, "cantidad": 2, "subtotal": 110.0 }
  ],
  "total": 155.5,
  "estado": "pendiente",
  "paraLlevar": false,
  "notas": "Sin azúcar en el matcha por favor",
  "fecha": "2024-01-15T11:30:00"
}
```

### 10.4 — Procesar el pedido

Copia el `id` del pedido que acabas de crear.

**Cambiar a "preparando":**
```
PATCH http://localhost:8083/api/pedidos/PEGA_ID_PEDIDO/estado?valor=preparando
```

**Cambiar a "listo":**
```
PATCH http://localhost:8083/api/pedidos/PEGA_ID_PEDIDO/estado?valor=listo
```

**Marcar como "entregado"** (esto agrega puntos a Sofía automáticamente):
```
PATCH http://localhost:8083/api/pedidos/PEGA_ID_PEDIDO/estado?valor=entregado
```

En la consola del servicio-pedidos verás:
```
? 15 puntos ? Sofía Martínez
```

### 10.5 — Verificar los puntos de Sofía

```
GET http://localhost:8082/api/clientes/PEGA_ID_SOFIA
```

Sofía ahora tiene 165 puntos (150 iniciales + 15 del pedido).

### 10.6 — Probar validaciones de error

**Pedido con bebida no disponible** (el Chai Latte tiene `disponible: false`):
```json
{
  "clienteId": "PEGA_ID_SOFIA",
  "items": [{ "bebidaId": "PEGA_ID_CHAI_LATTE", "cantidad": 1 }],
  "paraLlevar": false
}
```
Respuesta esperada (400 Bad Request):
```json
{ "error": "La bebida 'Chai Latte' no está disponible" }
```

**Pedido con cliente inexistente:**
```json
{
  "clienteId": "id-que-no-existe",
  "items": [{ "bebidaId": "...", "cantidad": 1 }],
  "paraLlevar": false
}
```
Respuesta esperada (400 Bad Request):
```json
{ "error": "Cliente no encontrado: id-que-no-existe" }
```

---

## ?? ¡Felicidades!

Construiste el Servicio de Pedidos: el microservicio más complejo del Michi Café.

### ¿Qué construiste?
- ? Proyecto Spring Boot 4.1.0 en puerto 8083
- ? Modelos `Pedido`, `ItemPedido` y `SolicitudPedido`
- ? Repository con búsquedas por cliente y estado
- ? Service que llama a los otros 2 microservicios con RestTemplate
- ? Controller con endpoints GET, POST y PATCH
- ? Validación de cliente y bebidas antes de crear el pedido
- ? Asignación automática de puntos al entregar

---

## ➡️ Siguiente paso

En el **[Proyecto P06 — API Gateway](proyecto-06-api-gateway.md)** crearás la puerta de entrada única del Michi Café: un solo puerto (8080) que enruta todas las peticiones al servicio correcto.

> 🐾 "El pedido llegó, se preparó y se entregó. El Michi Café ya funciona de verdad."
