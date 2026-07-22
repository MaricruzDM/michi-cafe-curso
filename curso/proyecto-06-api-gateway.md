# Proyecto P06 — API Gateway
## La puerta única de entrada al Michi Café

> ?? Referencias teóricas:
> - [Módulo 13 — API Gateway](modulo-13-api-gateway.md)
> - [Módulo 04 — Spring Boot](modulo-04-spring-boot.md)

---

## Que vamos a construir

El **API Gateway** es una pieza que recibe TODAS las peticiones en el puerto 8080
y las redirige al microservicio correcto:

- `/api/bebidas/**` ? servicio-bebidas en 8081
- `/api/clientes/**` ? servicio-clientes en 8082
- `/api/pedidos/**` ? servicio-pedidos en 8083

Ventaja: desde afuera solo existe un puerto. Los microservicios quedan ocultos.

**Tiempo estimado:** 45-60 minutos

---

## Paso 1: Crear el proyecto en Spring Initializr

1. Ve a **https://start.spring.io**
2. Configura:

| Campo | Valor |
|-------|-------|
| Project | Maven |
| Language | Java |
| Spring Boot | 4.1.0 |
| Group | com.michicafe |
| Artifact | api-gateway |
| Package name | com.michicafe.gateway |
| Packaging | Jar |
| Java | 17 |

3. Dependencias — solo estas dos:
   - **Gateway**
   - **Spring Boot Actuator**

> IMPORTANTE: NO agregues Spring Web. Gateway usa un motor distinto (WebFlux)
> que es incompatible con Spring Web. Si los agregas juntos el proyecto no arranca.

4. Clic en **GENERATE** ? extrae en `C:\Proyectos\api-gateway`

---

## Paso 2: Abrir en IntelliJ

1. IntelliJ ? **Open** ? selecciona `C:\Proyectos\api-gateway` ? **OK**
2. Espera que carguen las dependencias

---

## Paso 3: Configurar las rutas en application.yml

El Gateway se configura mejor en formato YAML que en `.properties`.
Vamos a RENOMBRAR el archivo de configuración.

### 3.1 — Renombrar application.properties a application.yml

1. En el panel izquierdo expande `src ? main ? resources`
2. Haz clic derecho sobre `application.properties`
3. Selecciona **Refactor ? Rename**
4. Borra `application.properties` y escribe: `application.yml`
5. Presiona **Enter** ? clic en **Refactor**

### 3.2 — Escribir la configuracion

Haz doble clic en `application.yml` y escribe exactamente esto:

```yaml
server:
  port: 8080

spring:
  application:
    name: api-gateway

  cloud:
    gateway:
      routes:

        - id: servicio-bebidas
          uri: http://localhost:8081
          predicates:
            - Path=/api/bebidas/**

        - id: servicio-clientes
          uri: http://localhost:8082
          predicates:
            - Path=/api/clientes/**

        - id: servicio-pedidos
          uri: http://localhost:8083
          predicates:
            - Path=/api/pedidos/**

management:
  endpoints:
    web:
      exposure:
        include: health,gateway
  endpoint:
    gateway:
      enabled: true
```

> Atencion con la indentacion (los espacios). YAML es muy sensible a los espacios.
> Usa siempre 2 espacios para indentar, nunca tabs.

Guarda con **Ctrl + S**.

---

## Paso 4: Crear el filtro de logs

Vamos a crear un filtro que registre en consola cada peticion que pase por el Gateway.

### 4.1 — Crear el paquete filter

1. Clic derecho sobre `com.michicafe.gateway` ? **New** ? **Package**
2. Escribe: `filter` ? Enter

### 4.2 — Crear la clase LoggingFilter

1. Clic derecho sobre `filter` ? **New** ? **Java Class**
2. Escribe: `LoggingFilter` ? **Class** ? Enter

```java
package com.michicafe.gateway.filter;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.core.Ordered;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

@Component
public class LoggingFilter implements GlobalFilter, Ordered {

    private static final Logger log = LoggerFactory.getLogger(LoggingFilter.class);

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String metodo = exchange.getRequest().getMethod().toString();
        String ruta   = exchange.getRequest().getPath().toString();

        log.info("?? Gateway recibio: {} {}", metodo, ruta);

        return chain.filter(exchange).then(Mono.fromRunnable(() -> {
            int codigo = exchange.getResponse().getStatusCode().value();
            log.info("? Respuesta: {} {} ? {}", metodo, ruta, codigo);
        }));
    }

    @Override
    public int getOrder() {
        return -1;
    }
}
```

Importa con **Alt + Enter**. Guarda con **Ctrl + S**.

---

## Paso 5: Ejecutar el Gateway

### 5.1 — Arrancar todos los servicios

Antes de arrancar el Gateway, asegurate de que los 3 microservicios esten corriendo:
- servicio-bebidas en puerto 8081
- servicio-clientes en puerto 8082
- servicio-pedidos en puerto 8083

### 5.2 — Arrancar el Gateway

1. Abre `ApiGatewayApplication.java`
2. Clic en el triangulo verde junto al metodo `main`
3. Selecciona **Run 'ApiGatewayApplication'**

Busca en la consola:
```
Started ApiGatewayApplication in X seconds
Netty started on port 8080
```

> El Gateway usa Netty (no Tomcat). Eso es normal con WebFlux.

---

## Paso 6: Probar el enrutamiento

Ahora TODAS las peticiones van al puerto 8080 solamente.

### Probar bebidas a traves del Gateway

```
GET http://localhost:8080/api/bebidas
```

El Gateway redirige esto a `http://localhost:8081/api/bebidas` y devuelve el resultado.

### Probar clientes a traves del Gateway

```
GET http://localhost:8080/api/clientes
```

### Probar pedidos a traves del Gateway

```
GET http://localhost:8080/api/pedidos
```

### Crear un pedido a traves del Gateway

```
POST http://localhost:8080/api/pedidos
Body ? raw ? JSON:
```
```json
{
  "clienteId": "PEGA_ID_DE_SOFIA",
  "items": [
    { "bebidaId": "PEGA_ID_BEBIDA", "cantidad": 1 }
  ],
  "paraLlevar": false
}
```

El Gateway lo redirige a `http://localhost:8083/api/pedidos` automaticamente.

### Ver los logs del Gateway

En la consola de IntelliJ del Gateway veras:
```
?? Gateway recibio: GET /api/bebidas
? Respuesta: GET /api/bebidas ? 200
?? Gateway recibio: POST /api/pedidos
? Respuesta: POST /api/pedidos ? 201
```

---

## Paso 7: Verificar el health check

```
GET http://localhost:8080/actuator/health
```

Respuesta:
```json
{ "status": "UP" }
```

---

## Que construiste

- Proyecto Spring Cloud Gateway en puerto 8080
- Reglas de enrutamiento para los 3 microservicios
- Filtro de logs que registra todas las peticiones
- Un solo punto de entrada para todo el Michi Cafe

### Estructura del proyecto

```
api-gateway/
+-- src/main/java/com/michicafe/gateway/
    +-- ApiGatewayApplication.java
    +-- filter/
        +-- LoggingFilter.java
+-- src/main/resources/
    +-- application.yml
```

---

## ➡️ Siguiente paso

En el **[Proyecto P07 — Docker Compose](proyecto-07-docker.md)** vamos a empaquetar todos los servicios en contenedores Docker y levantarlos todos con un solo comando.

> 🐾 "Una sola puerta para todo el Michi Café. Mucho más ordenado."
