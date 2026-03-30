# Módulo 13 — API Gateway: La Puerta Principal del Michi Café
## 🐱 Analogía: La recepción que dirige a todos los visitantes

---

## 🏠 Introducción

Tenemos tres microservicios corriendo en puertos diferentes:
- `servicio-bebidas` en el puerto **8081**
- `servicio-clientes` en el puerto **8082**
- `servicio-pedidos` en el puerto **8083**

Esto crea un problema: el cliente (app móvil, navegador, etc.) tendría que saber la dirección exacta de cada servicio. Si un servicio cambia de puerto o de servidor, habría que actualizar todos los clientes.

La solución es el **API Gateway**: un único punto de entrada que recibe todas las peticiones y las redirige al servicio correcto.

### Analogía 🐾

Imagina que el Michi Café tiene tres áreas en pisos diferentes:
- Piso 1: Bebidas
- Piso 2: Clientes
- Piso 3: Pedidos

Sin recepción, cada visitante tendría que saber a qué piso ir. Con recepción, todos entran por la misma puerta y la recepcionista los dirige:
- "¿Vienes por el menú? → Piso 1"
- "¿Vienes a registrarte? → Piso 2"
- "¿Vienes a hacer un pedido? → Piso 3"

El **API Gateway** es esa recepcionista.

---

## 💡 ¿Qué hace un API Gateway?

Además de redirigir peticiones, el API Gateway puede:

| Función | ¿Qué hace? | Analogía |
|---------|-----------|----------|
| **Enrutamiento** | Redirige cada petición al servicio correcto | La recepcionista que indica el piso |
| **Autenticación** | Verifica que el usuario tenga permiso | El guardia de seguridad en la entrada |
| **Rate limiting** | Limita cuántas peticiones puede hacer un cliente | "Solo 10 pedidos por minuto por persona" |
| **Load balancing** | Distribuye la carga entre instancias | Repartir clientes entre varios cajeros |
| **Logging** | Registra todas las peticiones | El libro de visitas de la recepción |
| **CORS** | Permite peticiones desde otros dominios | "Clientes de otras ciudades también son bienvenidos" |

---

## 🛠️ Creando el API Gateway

### Paso 1: Crear el proyecto

En Spring Initializr (**https://start.spring.io**):

| Campo | Valor |
|-------|-------|
| Artifact | `api-gateway` |
| Group | `com.michicafe` |
| Dependencies | `Gateway`, `Actuator` |

> ⚠️ El API Gateway usa **Spring WebFlux** (reactivo) en lugar de Spring MVC. No agregues `Spring Web` porque son incompatibles con Gateway.

### Paso 2: El `pom.xml`

```xml
<dependencies>
    <!-- Spring Cloud Gateway -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>

    <!-- Actuator: endpoints de salud y métricas -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>

<!-- Spring Cloud BOM: gestiona versiones compatibles -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>2023.0.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### Paso 3: Configuración de rutas

La configuración del Gateway se hace en `application.yml` (usamos YAML en lugar de `.properties` porque es más legible para configuraciones complejas):

```yaml
# src/main/resources/application.yml

server:
  port: 8080

spring:
  application:
    name: api-gateway

  cloud:
    gateway:
      routes:

        # ── Servicio de Bebidas ──────────────────────────────────────────────
        - id: servicio-bebidas
          uri: http://localhost:8081          # ¿A dónde redirigir?
          predicates:
            - Path=/api/bebidas/**            # ¿Qué rutas captura?
          filters:
            - StripPrefix=0                   # No quitar prefijo de la ruta

        # ── Servicio de Clientes ─────────────────────────────────────────────
        - id: servicio-clientes
          uri: http://localhost:8082
          predicates:
            - Path=/api/clientes/**

        # ── Servicio de Pedidos ──────────────────────────────────────────────
        - id: servicio-pedidos
          uri: http://localhost:8083
          predicates:
            - Path=/api/pedidos/**

# Actuator: exponer endpoints de salud
management:
  endpoints:
    web:
      exposure:
        include: health, info, gateway
  endpoint:
    gateway:
      enabled: true
```

### Paso 4: La clase principal

```java
// src/main/java/com/michicafe/gateway/ApiGatewayApplication.java
package com.michicafe.gateway;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ApiGatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(ApiGatewayApplication.class, args);
        System.out.println("🐱 API Gateway del Michi Café corriendo en puerto 8080");
    }
}
```

---

## 💡 Cómo funciona el enrutamiento

Con el Gateway corriendo, todas las peticiones van al puerto **8080** y él las redirige:

```
Cliente                    API Gateway (8080)         Microservicio
  │                              │                         │
  │  GET /api/bebidas            │                         │
  │─────────────────────────────>│                         │
  │                              │  GET /api/bebidas        │
  │                              │────────────────────────>│ (8081)
  │                              │                         │
  │                              │  200 OK + [bebidas]     │
  │                              │<────────────────────────│
  │  200 OK + [bebidas]          │                         │
  │<─────────────────────────────│                         │
```

### Tabla de enrutamiento

| Petición al Gateway (8080) | Se redirige a |
|---------------------------|---------------|
| `GET /api/bebidas` | `http://localhost:8081/api/bebidas` |
| `POST /api/clientes` | `http://localhost:8082/api/clientes` |
| `GET /api/pedidos/P-001` | `http://localhost:8083/api/pedidos/P-001` |

---

## 💡 Filtros: procesamiento antes y después

Los filtros permiten ejecutar lógica antes de enviar la petición al servicio o después de recibir la respuesta.

### Filtros globales (aplican a todas las rutas)

```java
// src/main/java/com/michicafe/gateway/filter/LoggingFilter.java
package com.michicafe.gateway.filter;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

@Component
public class LoggingFilter implements GlobalFilter, Ordered {

    private static final Logger log = LoggerFactory.getLogger(LoggingFilter.class);

    @Override
    public Mono<Void> filter(ServerWebExchange exchange,
                             org.springframework.cloud.gateway.filter.GatewayFilterChain chain) {
        String path   = exchange.getRequest().getPath().toString();
        String method = exchange.getRequest().getMethod().toString();

        log.info("🐾 Michi Gateway → {} {}", method, path);

        return chain.filter(exchange).then(Mono.fromRunnable(() -> {
            int statusCode = exchange.getResponse().getStatusCode().value();
            log.info("✅ Respuesta → {} {} → {}", method, path, statusCode);
        }));
    }

    @Override
    public int getOrder() {
        return -1; // Prioridad alta: se ejecuta primero
    }
}
```

Con este filtro, cada petición que pase por el Gateway se registra en los logs:
```
🐾 Michi Gateway → GET /api/bebidas
✅ Respuesta → GET /api/bebidas → 200
```

### Filtros por ruta (en application.yml)

```yaml
routes:
  - id: servicio-bebidas
    uri: http://localhost:8081
    predicates:
      - Path=/api/bebidas/**
    filters:
      # Agrega un header a todas las peticiones que van a este servicio
      - AddRequestHeader=X-Gateway-Source, michi-gateway
      # Agrega un header a todas las respuestas
      - AddResponseHeader=X-Powered-By, MichiCafe-Gateway
```

---

## 💡 CORS: permitir peticiones desde el navegador

Cuando una app web (React, Angular, etc.) hace peticiones al Gateway, el navegador bloquea las peticiones si no están configuradas las políticas CORS.

```yaml
# En application.yml
spring:
  cloud:
    gateway:
      globalcors:
        cors-configurations:
          '[/**]':
            allowedOrigins:
              - "http://localhost:3000"   # App React en desarrollo
              - "https://michicafe.com"   # App en producción
            allowedMethods:
              - GET
              - POST
              - PUT
              - PATCH
              - DELETE
            allowedHeaders:
              - "*"
```

### Analogía 🐾

CORS es como la política del Michi Café sobre quién puede hacer pedidos:
- "Solo aceptamos pedidos de clientes registrados en nuestra app (localhost:3000)"
- "También aceptamos pedidos desde nuestra web oficial (michicafe.com)"
- "No aceptamos pedidos de fuentes desconocidas"

---

## 💡 Health checks: ¿están vivos los servicios?

El Actuator expone endpoints para verificar el estado del Gateway y los servicios:

```
GET http://localhost:8080/actuator/health
```

Respuesta:
```json
{
  "status": "UP",
  "components": {
    "gateway": {
      "status": "UP"
    }
  }
}
```

---

## 💡 Manejo de errores en el Gateway

Cuando un microservicio no está disponible, el Gateway debe responder con un error claro:

```java
// src/main/java/com/michicafe/gateway/handler/GatewayErrorHandler.java
package com.michicafe.gateway.handler;

import org.springframework.boot.web.reactive.error.ErrorWebExceptionHandler;
import org.springframework.core.annotation.Order;
import org.springframework.core.io.buffer.DataBuffer;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

@Component
@Order(-2)
public class GatewayErrorHandler implements ErrorWebExceptionHandler {

    @Override
    public Mono<Void> handle(ServerWebExchange exchange, Throwable ex) {
        exchange.getResponse().setStatusCode(HttpStatus.SERVICE_UNAVAILABLE);
        exchange.getResponse().getHeaders().setContentType(MediaType.APPLICATION_JSON);

        String body = """
            {
              "codigo": 503,
              "mensaje": "El servicio del Michi Café no está disponible en este momento 🐱",
              "detalle": "%s"
            }
            """.formatted(ex.getMessage());

        DataBuffer buffer = exchange.getResponse()
                .bufferFactory()
                .wrap(body.getBytes());

        return exchange.getResponse().writeWith(Mono.just(buffer));
    }
}
```

---

## 💡 Estructura del proyecto api-gateway

```
api-gateway/
└── src/main/java/com/michicafe/gateway/
    ├── ApiGatewayApplication.java
    ├── filter/
    │   └── LoggingFilter.java
    └── handler/
        └── GatewayErrorHandler.java
└── src/main/resources/
    └── application.yml
```

---

## 💡 Arquitectura completa hasta ahora

```
                    ┌─────────────────────────────┐
                    │   API Gateway  :8080         │
                    │   (puerta principal)         │
                    └──────────────┬──────────────┘
                                   │
              ┌────────────────────┼────────────────────┐
              │                    │                    │
              ▼                    ▼                    ▼
  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
  │ servicio-bebidas│  │servicio-clientes│  │ servicio-pedidos│
  │     :8081       │  │     :8082       │  │     :8083       │
  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘
           │                    │                    │
           ▼                    ▼                    ▼
       [MongoDB]            [MongoDB]            [MongoDB]
      michicafe_           michicafe_           michicafe_
       bebidas              clientes              pedidos
```

---

## 🧪 Ejercicios del Módulo 13

### Ejercicio 1: Verificar el enrutamiento

1. Arranca los 3 microservicios y el Gateway
2. Haz todas las peticiones **solo al puerto 8080** y verifica que llegan al servicio correcto:
   - `GET http://localhost:8080/api/bebidas`
   - `GET http://localhost:8080/api/clientes`
   - `GET http://localhost:8080/api/pedidos`

### Ejercicio 2: Observar los logs

Con el `LoggingFilter` activo, haz varias peticiones y observa los logs del Gateway. Verifica que registra método, ruta y código de respuesta.

### Ejercicio 3: Simular un servicio caído

1. Detén el `servicio-bebidas`
2. Haz `GET http://localhost:8080/api/bebidas`
3. Verifica que el Gateway responde con el error personalizado (503) en lugar de un error genérico

---

## ✅ Resumen del Módulo 13

| Concepto | ¿Qué es? | Analogía del Michi Café |
|----------|----------|------------------------|
| API Gateway | Punto de entrada único para todos los servicios | La recepción del café |
| Enrutamiento | Redirigir peticiones al servicio correcto | La recepcionista que indica el piso |
| Filtro global | Lógica que aplica a todas las peticiones | El guardia que revisa a todos |
| Filtro por ruta | Lógica específica para un servicio | Reglas especiales para el área VIP |
| CORS | Política de quién puede hacer peticiones | Lista de clientes autorizados |
| Health check | Verificar que los servicios están vivos | El supervisor que revisa que todo funcione |

---

## ➡️ Siguiente Módulo

En el **Módulo 14** aprenderemos **Docker**: cómo empaquetar cada microservicio en un contenedor para que pueda correr en cualquier computadora o servidor sin problemas de configuración.

> 🐾 "Un buen Gateway es como una buena recepcionista: sabe exactamente a dónde mandar a cada quien, sin importar cuántos lleguen al mismo tiempo."
