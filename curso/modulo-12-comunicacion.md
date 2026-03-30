# Módulo 12 — Comunicación entre Microservicios
## 🐱 Analogía: Los gatitos se pasan notitas

---

## 🏠 Introducción

Tenemos tres microservicios corriendo de forma independiente. Pero en la vida real, necesitan colaborar:

- Cuando se crea un pedido, el servicio de pedidos necesita **consultar el precio real** de cada bebida al servicio de bebidas
- También necesita **verificar que el cliente existe** en el servicio de clientes
- Cuando se entrega un pedido, debería **agregar puntos** al cliente

En este módulo aprenderemos cómo los microservicios se comunican entre sí.

### Analogía 🐾

En el Michi Café, cuando la cocina recibe un pedido:
1. Le pregunta al área de bebidas: "¿Cuánto cuesta el Latte de vainilla?"
2. Le pregunta al área de clientes: "¿Existe este cliente?"
3. Cuando entrega el pedido, le avisa al área de clientes: "Agrega 10 puntos a este cliente"

Esas preguntas y avisos son la **comunicación entre microservicios**.

---

## 💡 Tipos de comunicación

Hay dos formas principales de comunicación entre microservicios:

### 1. Comunicación síncrona (HTTP REST)

Un servicio llama a otro y **espera la respuesta** antes de continuar.

```
Servicio Pedidos → llama → Servicio Bebidas → responde → Servicio Pedidos continúa
```

### Analogía 🐾

La cocina llama por teléfono al área de bebidas y espera que contesten para saber el precio. No puede continuar hasta tener la respuesta.

**Ventaja**: Simple de entender e implementar.
**Desventaja**: Si el servicio de bebidas está caído, el servicio de pedidos también falla.

### 2. Comunicación asíncrona (Mensajes)

Un servicio envía un mensaje a una cola y **no espera respuesta**. El otro servicio procesa el mensaje cuando puede.

```
Servicio Pedidos → envía mensaje → Cola → Servicio Clientes procesa cuando puede
```

### Analogía 🐾

La cocina deja una nota en el buzón del área de clientes: "Agrega 10 puntos al cliente X". El área de clientes la leerá cuando pueda, sin bloquear a la cocina.

**Ventaja**: Los servicios son más independientes, si uno falla el mensaje queda en la cola.
**Desventaja**: Más complejo de implementar.

En este módulo usaremos **comunicación síncrona con HTTP REST**, que es la más común para empezar.

---

## 💡 RestTemplate vs WebClient

Spring ofrece dos formas de hacer llamadas HTTP a otros servicios:

| | RestTemplate | WebClient |
|--|-------------|-----------|
| Tipo | Síncrono (bloquea el hilo) | Reactivo (no bloquea) |
| Facilidad | Más simple | Más complejo |
| Recomendado para | Aprender, proyectos simples | Producción, alto rendimiento |

Usaremos **RestTemplate** por ser más fácil de entender.

---

## 💡 Configurando RestTemplate

```java
// src/main/java/com/michicafe/pedidos/config/AppConfig.java
package com.michicafe.pedidos.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class AppConfig {

    @Bean  // Le dice a Spring que cree y gestione este objeto
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

---

## 💡 DTOs para comunicación entre servicios

Cuando el servicio de pedidos llama al servicio de bebidas, necesita un DTO para recibir la respuesta:

```java
// src/main/java/com/michicafe/pedidos/dto/BebidaExternaDTO.java
package com.michicafe.pedidos.dto;

// Representa la respuesta que recibimos del servicio de bebidas
public class BebidaExternaDTO {
    private String id;
    private String nombre;
    private double precio;
    private boolean disponible;

    public BebidaExternaDTO() {}

    public String getId()          { return id; }
    public String getNombre()      { return nombre; }
    public double getPrecio()      { return precio; }
    public boolean isDisponible()  { return disponible; }

    public void setId(String id)               { this.id = id; }
    public void setNombre(String nombre)       { this.nombre = nombre; }
    public void setPrecio(double precio)       { this.precio = precio; }
    public void setDisponible(boolean disp)    { this.disponible = disp; }
}
```

```java
// src/main/java/com/michicafe/pedidos/dto/ClienteExternoDTO.java
package com.michicafe.pedidos.dto;

// Representa la respuesta que recibimos del servicio de clientes
public class ClienteExternoDTO {
    private String id;
    private String nombre;
    private String apellido;
    private int puntosFidelidad;

    public ClienteExternoDTO() {}

    public String getId()           { return id; }
    public String getNombre()       { return nombre; }
    public String getApellido()     { return apellido; }
    public int getPuntosFidelidad() { return puntosFidelidad; }

    public void setId(String id)                  { this.id = id; }
    public void setNombre(String nombre)          { this.nombre = nombre; }
    public void setApellido(String apellido)      { this.apellido = apellido; }
    public void setPuntosFidelidad(int puntos)    { this.puntosFidelidad = puntos; }
}
```

---

## 💡 Clientes HTTP: los mensajeros del servicio

Creamos clases dedicadas para comunicarse con cada servicio externo:

```java
// src/main/java/com/michicafe/pedidos/client/BebidaClient.java
package com.michicafe.pedidos.client;

import com.michicafe.pedidos.dto.BebidaExternaDTO;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import org.springframework.web.client.HttpClientErrorException;
import org.springframework.web.client.RestTemplate;

import java.util.Optional;

@Component
public class BebidaClient {

    @Autowired
    private RestTemplate restTemplate;

    // Leemos la URL del servicio de bebidas desde application.properties
    @Value("${servicios.bebidas.url}")
    private String bebidaServiceUrl;

    public Optional<BebidaExternaDTO> obtenerBebida(String bebidaId) {
        try {
            String url = bebidaServiceUrl + "/api/bebidas/" + bebidaId;
            BebidaExternaDTO bebida = restTemplate.getForObject(url, BebidaExternaDTO.class);
            return Optional.ofNullable(bebida);
        } catch (HttpClientErrorException.NotFound e) {
            // El servicio de bebidas respondió 404
            return Optional.empty();
        } catch (Exception e) {
            // El servicio de bebidas no está disponible
            throw new RuntimeException(
                "No se pudo conectar con el servicio de bebidas: " + e.getMessage());
        }
    }
}
```

```java
// src/main/java/com/michicafe/pedidos/client/ClienteClient.java
package com.michicafe.pedidos.client;

import com.michicafe.pedidos.dto.ClienteExternoDTO;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import org.springframework.web.client.HttpClientErrorException;
import org.springframework.web.client.RestTemplate;

import java.util.Optional;

@Component
public class ClienteClient {

    @Autowired
    private RestTemplate restTemplate;

    @Value("${servicios.clientes.url}")
    private String clienteServiceUrl;

    public Optional<ClienteExternoDTO> obtenerCliente(String clienteId) {
        try {
            String url = clienteServiceUrl + "/api/clientes/" + clienteId;
            ClienteExternoDTO cliente = restTemplate.getForObject(url, ClienteExternoDTO.class);
            return Optional.ofNullable(cliente);
        } catch (HttpClientErrorException.NotFound e) {
            return Optional.empty();
        } catch (Exception e) {
            throw new RuntimeException(
                "No se pudo conectar con el servicio de clientes: " + e.getMessage());
        }
    }

    public void agregarPuntos(String clienteId, int puntos) {
        try {
            String url = clienteServiceUrl + "/api/clientes/" + clienteId
                       + "/puntos?cantidad=" + puntos;
            restTemplate.patchForObject(url, null, ClienteExternoDTO.class);
        } catch (Exception e) {
            // Si falla agregar puntos, no bloqueamos el flujo del pedido
            // Solo lo registramos en el log
            System.err.println("⚠️ No se pudieron agregar puntos al cliente: " + e.getMessage());
        }
    }
}
```

### Configurar las URLs en application.properties

```properties
# application.properties del servicio-pedidos

spring.application.name=servicio-pedidos
server.port=8083
spring.data.mongodb.host=localhost
spring.data.mongodb.port=27017
spring.data.mongodb.database=michicafe-pedidos

# URLs de los otros servicios
servicios.bebidas.url=http://localhost:8081
servicios.clientes.url=http://localhost:8082
```

---

## 💡 Actualizando el PedidoService con comunicación real

```java
// PedidoService.java actualizado
package com.michicafe.pedidos.service;

import com.michicafe.pedidos.client.BebidaClient;
import com.michicafe.pedidos.client.ClienteClient;
import com.michicafe.pedidos.dto.*;
import com.michicafe.pedidos.model.*;
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

    @Autowired
    private BebidaClient bebidaClient;      // Mensajero al servicio de bebidas

    @Autowired
    private ClienteClient clienteClient;    // Mensajero al servicio de clientes

    public Pedido crear(CrearPedidoDTO dto) {

        // 1. Verificar que el cliente existe
        ClienteExternoDTO cliente = clienteClient.obtenerCliente(dto.getClienteId())
                .orElseThrow(() -> new RuntimeException(
                    "Cliente no encontrado: " + dto.getClienteId()));

        // 2. Construir los ítems consultando precios reales al servicio de bebidas
        List<ItemPedido> items = new ArrayList<>();
        for (ItemPedidoDTO itemDTO : dto.getItems()) {

            BebidaExternaDTO bebida = bebidaClient.obtenerBebida(itemDTO.getBebidaId())
                    .orElseThrow(() -> new RuntimeException(
                        "Bebida no encontrada: " + itemDTO.getBebidaId()));

            if (!bebida.isDisponible()) {
                throw new RuntimeException(
                    "La bebida '" + bebida.getNombre() + "' no está disponible");
            }

            items.add(new ItemPedido(
                bebida.getId(),
                bebida.getNombre(),
                bebida.getPrecio(),
                itemDTO.getCantidad()
            ));
        }

        // 3. Crear el pedido con datos reales
        String nombreCompleto = cliente.getNombre() + " " + cliente.getApellido();
        Pedido pedido = new Pedido(dto.getClienteId(), nombreCompleto, items, dto.getNotas());

        return pedidoRepository.save(pedido);
    }

    public Optional<Pedido> avanzarEstado(String id) {
        return pedidoRepository.findById(id).map(pedido -> {
            EstadoPedido nuevoEstado = siguienteEstado(pedido.getEstado());
            pedido.setEstado(nuevoEstado);
            pedido.setFechaActualizacion(LocalDateTime.now());

            Pedido guardado = pedidoRepository.save(pedido);

            // Si el pedido fue entregado, agregar puntos al cliente
            if (nuevoEstado == EstadoPedido.ENTREGADO) {
                int puntos = (int) (pedido.getTotal() / 10); // 1 punto por cada $10
                clienteClient.agregarPuntos(pedido.getClienteId(), puntos);
                System.out.println("🐾 +" + puntos + " puntos para " + pedido.getNombreCliente());
            }

            return guardado;
        });
    }

    // Los demás métodos permanecen igual que en el módulo 10
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
            throw new RuntimeException("Estado inválido: " + estado);
        }
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

    private EstadoPedido siguienteEstado(EstadoPedido estadoActual) {
        switch (estadoActual) {
            case PENDIENTE:      return EstadoPedido.CONFIRMADO;
            case CONFIRMADO:     return EstadoPedido.EN_PREPARACION;
            case EN_PREPARACION: return EstadoPedido.LISTO;
            case LISTO:          return EstadoPedido.ENTREGADO;
            default:
                throw new RuntimeException("El pedido ya está en estado final: " + estadoActual);
        }
    }
}
```

---

## 💡 Patrón Circuit Breaker: cuando un servicio falla

En producción, los servicios pueden fallar. El **Circuit Breaker** (interruptor de circuito) es un patrón que evita que un fallo en cascada derribe todo el sistema.

### Analogía 🐾

Si el área de bebidas del Michi Café está cerrada temporalmente, la cocina no se queda esperando indefinidamente. Después de 3 intentos fallidos, la cocina decide usar el precio del último menú conocido y sigue trabajando.

```java
// Versión simplificada con manejo de fallback
public Optional<BebidaExternaDTO> obtenerBebidaConFallback(String bebidaId) {
    try {
        return obtenerBebida(bebidaId);
    } catch (RuntimeException e) {
        System.err.println("⚠️ Servicio de bebidas no disponible. Usando precio por defecto.");
        // Fallback: devolvemos una bebida con precio por defecto
        BebidaExternaDTO fallback = new BebidaExternaDTO();
        fallback.setId(bebidaId);
        fallback.setNombre("Bebida (precio no disponible)");
        fallback.setPrecio(0.0);
        fallback.setDisponible(true);
        return Optional.of(fallback);
    }
}
```

---

## 💡 Probando la comunicación

Con los tres servicios corriendo, prueba el flujo completo:

### 1. Crear un cliente (servicio-clientes: 8082)

```
POST http://localhost:8082/api/clientes
{
  "nombre": "Sofía",
  "apellido": "García",
  "correo": "sofia@email.com",
  "telefono": "555-1234"
}
```
Guarda el `id` del cliente: `"64a1b2c3..."`

### 2. Verificar bebidas disponibles (servicio-bebidas: 8081)

```
GET http://localhost:8081/api/bebidas
```
Guarda el `id` de una bebida: `"64a1b2c4..."`

### 3. Crear un pedido (servicio-pedidos: 8083)

```
POST http://localhost:8083/api/pedidos
{
  "clienteId": "64a1b2c3...",
  "items": [
    { "bebidaId": "64a1b2c4...", "cantidad": 2 }
  ],
  "notas": "Para llevar"
}
```

Ahora el servicio de pedidos:
- Consulta al servicio de clientes para verificar que Sofía existe
- Consulta al servicio de bebidas para obtener el precio real
- Crea el pedido con datos reales

### 4. Avanzar hasta entregado y ver los puntos

```
PATCH http://localhost:8083/api/pedidos/{id}/avanzar  (x4 veces)
```

Luego verifica los puntos de Sofía:
```
GET http://localhost:8082/api/clientes/64a1b2c3...
```

---

## 🧪 Ejercicios del Módulo 12

### Ejercicio 1: Verificar disponibilidad antes de crear

Modifica `PedidoService.crear()` para que antes de aceptar el pedido, verifique que todas las bebidas solicitadas están disponibles. Si alguna no lo está, rechaza el pedido completo con un mensaje claro.

### Ejercicio 2: Historial enriquecido

Crea un endpoint `GET /api/pedidos/{id}/detalle` que devuelva el pedido con información adicional del cliente (nombre completo, nivel de fidelidad) obtenida del servicio de clientes en tiempo real.

### Ejercicio 3: Manejo de fallos

Modifica el `BebidaClient` para que si el servicio de bebidas no responde en 3 segundos, use un precio por defecto de $0 y marque el ítem como "precio pendiente de confirmar".

---

## ✅ Resumen del Módulo 12

| Concepto | ¿Qué es? | Analogía del Michi Café |
|----------|----------|------------------------|
| Comunicación síncrona | Un servicio llama y espera respuesta | La cocina llama por teléfono y espera |
| Comunicación asíncrona | Un servicio envía mensaje sin esperar | La cocina deja una nota en el buzón |
| `RestTemplate` | Cliente HTTP de Spring | El mensajero entre áreas |
| DTO externo | Objeto para recibir datos de otro servicio | El formulario que llena el mensajero |
| Circuit Breaker | Manejo de fallos en cascada | Si el área falla, usar plan B |
| Fallback | Respuesta alternativa cuando falla un servicio | Usar el último precio conocido |

---

## ➡️ Siguiente Módulo

En el **Módulo 13** crearemos el **API Gateway**: la puerta principal del Michi Café. En lugar de que los clientes tengan que saber en qué puerto está cada servicio, el Gateway los dirige automáticamente.

> 🐾 "Los gatitos del Michi Café trabajan en equipo. Cada uno hace su parte y se comunican con respeto."
