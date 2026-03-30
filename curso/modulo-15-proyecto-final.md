# Módulo 15 — Proyecto Final: El Michi Café Completo
## 🐱 El Michi Café abre sus puertas al mundo

---

## 🏠 Introducción

¡Llegaste al módulo final! A lo largo de este curso aprendiste:

- ✅ Java desde cero: variables, condiciones, ciclos, métodos
- ✅ Programación Orientada a Objetos: clases, herencia, polimorfismo
- ✅ Spring Boot: servidores, controllers, services
- ✅ APIs REST: GET, POST, PUT, PATCH, DELETE
- ✅ MongoDB: documentos, colecciones, consultas
- ✅ Spring Data MongoDB: repositories, consultas automáticas
- ✅ Microservicios: arquitectura, independencia, responsabilidad única
- ✅ Comunicación entre servicios: RestTemplate, WebClient
- ✅ API Gateway: enrutamiento, filtros, CORS
- ✅ Docker: contenedores, imágenes, Docker Compose

En este módulo final vamos a **integrar todo** y construir el Michi Café completo, funcional y listo para producción.

---

## 💡 Arquitectura final del sistema

```
                         Internet
                            │
                            ▼
              ┌─────────────────────────────┐
              │      API Gateway  :8080      │
              │  - Enrutamiento              │
              │  - Logging                   │
              │  - CORS                      │
              └──────────────┬──────────────┘
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
         ▼                   ▼                   ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│ servicio-bebidas│ │servicio-clientes│ │ servicio-pedidos│
│     :8081       │ │     :8082       │ │     :8083       │
│                 │ │                 │ │  (llama a       │
│  - CRUD bebidas │ │  - CRUD clientes│ │   bebidas y     │
│  - Filtros      │ │  - Puntos       │ │   clientes)     │
└────────┬────────┘ └────────┬────────┘ └────────┬────────┘
         │                   │                   │
         ▼                   ▼                   ▼
    [MongoDB]           [MongoDB]           [MongoDB]
  michicafe_           michicafe_          michicafe_
   bebidas              clientes            pedidos
```

---

## 💡 Flujo completo de un pedido

Vamos a trazar el camino completo de un pedido en el Michi Café:

```
1. El cliente abre la app y ve el menú
   GET /api/bebidas → Gateway → servicio-bebidas → MongoDB

2. El cliente se registra (o ya tiene cuenta)
   POST /api/clientes → Gateway → servicio-clientes → MongoDB

3. El cliente hace un pedido
   POST /api/pedidos → Gateway → servicio-pedidos
     ├── Verifica que el cliente existe → llama a servicio-clientes
     ├── Verifica que las bebidas existen → llama a servicio-bebidas
     ├── Calcula el total
     └── Guarda el pedido → MongoDB

4. El pedido se procesa
   PATCH /api/pedidos/{id}/estado?valor=preparando → Gateway → servicio-pedidos

5. El pedido se entrega
   PATCH /api/pedidos/{id}/estado?valor=entregado → Gateway → servicio-pedidos
     └── Agrega puntos al cliente → llama a servicio-clientes
```

---

## 💡 Checklist de archivos del proyecto completo

Antes de integrar todo, verifica que tienes estos archivos en cada servicio:

### servicio-bebidas/
```
servicio-bebidas/
├── Dockerfile
├── pom.xml
└── src/main/
    ├── java/com/michicafe/bebidas/
    │   ├── BebidasApplication.java
    │   ├── config/DataLoader.java
    │   ├── controller/BebidaController.java
    │   ├── service/BebidaService.java
    │   ├── repository/BebidaRepository.java
    │   ├── model/Bebida.java
    │   └── exception/GlobalExceptionHandler.java
    └── resources/application.properties
```

### servicio-clientes/
```
servicio-clientes/
├── Dockerfile
├── pom.xml
└── src/main/
    ├── java/com/michicafe/clientes/
    │   ├── ClientesApplication.java
    │   ├── controller/ClienteController.java
    │   ├── service/ClienteService.java
    │   ├── repository/ClienteRepository.java
    │   ├── model/Cliente.java
    │   └── exception/GlobalExceptionHandler.java
    └── resources/application.properties
```

### servicio-pedidos/
```
servicio-pedidos/
├── Dockerfile
├── pom.xml
└── src/main/
    ├── java/com/michicafe/pedidos/
    │   ├── PedidosApplication.java
    │   ├── config/WebClientConfig.java
    │   ├── controller/PedidoController.java
    │   ├── service/PedidoService.java
    │   ├── repository/PedidoRepository.java
    │   ├── model/
    │   │   ├── Pedido.java
    │   │   └── ItemPedido.java
    │   └── exception/GlobalExceptionHandler.java
    └── resources/application.properties
```

### api-gateway/
```
api-gateway/
├── Dockerfile
├── pom.xml
└── src/main/
    ├── java/com/michicafe/gateway/
    │   ├── ApiGatewayApplication.java
    │   ├── filter/LoggingFilter.java
    │   └── handler/GatewayErrorHandler.java
    └── resources/application.yml
```

---

## 💡 Pruebas end-to-end: el día de apertura del Michi Café

Vamos a simular el primer día de operación del Michi Café completo.

### Paso 1: Levantar todo el sistema

```bash
cd michi-cafe
docker compose up --build -d

# Esperar ~30 segundos a que todo inicie
docker compose ps
```

Todos los servicios deben mostrar `Up`.

### Paso 2: Verificar el menú inicial

```
GET http://localhost:8080/api/bebidas
```

Respuesta esperada: lista de 6 bebidas cargadas por el DataLoader.

### Paso 3: Registrar los primeros clientes

```
POST http://localhost:8080/api/clientes
Content-Type: application/json

{ "nombre": "Sofía", "correo": "sofia@email.com" }
```

```
POST http://localhost:8080/api/clientes
Content-Type: application/json

{ "nombre": "Carlos", "correo": "carlos@email.com" }
```

Guarda los IDs que devuelve MongoDB (los necesitarás para los pedidos).

### Paso 4: Hacer el primer pedido

```
POST http://localhost:8080/api/pedidos
Content-Type: application/json

{
  "clienteId": "[id de Sofía]",
  "items": [
    { "bebidaId": "[id del Latte de vainilla]", "cantidad": 1 },
    { "bebidaId": "[id del Matcha Latte]",      "cantidad": 2 }
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
  "clienteNombre": "Sofía",
  "items": [
    { "bebidaId": "...", "nombre": "Latte de vainilla", "precio": 45.50, "cantidad": 1 },
    { "bebidaId": "...", "nombre": "Matcha Latte",      "precio": 55.00, "cantidad": 2 }
  ],
  "total": 155.50,
  "estado": "pendiente",
  "paraLlevar": false,
  "notas": "Sin azúcar en el matcha por favor",
  "fecha": "2024-01-15T11:30:00"
}
```

### Paso 5: Procesar el pedido

```
PATCH http://localhost:8080/api/pedidos/[id]/estado?valor=preparando
```

```
PATCH http://localhost:8080/api/pedidos/[id]/estado?valor=listo
```

```
PATCH http://localhost:8080/api/pedidos/[id]/estado?valor=entregado
```

Al marcar como entregado, el sistema automáticamente agrega puntos a Sofía.

### Paso 6: Verificar los puntos de Sofía

```
GET http://localhost:8080/api/clientes/[id de Sofía]
```

Sofía debe tener puntos acumulados (1 punto por cada peso gastado, redondeado).

### Paso 7: Ver todos los pedidos del día

```
GET http://localhost:8080/api/pedidos
```

---

## 💡 Colección de Postman para el proyecto final

Crea una colección en Postman con todas estas peticiones organizadas por carpetas:

```
📁 Michi Café - Proyecto Final
  📁 Bebidas
    GET  Obtener menú completo
    GET  Bebidas disponibles
    GET  Bebidas por categoría (caliente)
    GET  Bebidas por precio (40-55)
    POST Agregar bebida nueva
    PUT  Actualizar bebida
    PATCH Cambiar disponibilidad
    DELETE Eliminar bebida

  📁 Clientes
    GET  Obtener todos los clientes
    GET  Buscar cliente por correo
    POST Registrar cliente nuevo
    PATCH Agregar puntos
    DELETE Eliminar cliente

  📁 Pedidos
    GET  Obtener todos los pedidos
    GET  Pedidos por cliente
    GET  Pedidos por estado
    POST Crear pedido nuevo
    PATCH Cambiar estado del pedido
    DELETE Cancelar pedido
```

---

## 💡 Monitoreo: saber que todo está bien

### Health checks de todos los servicios

```bash
# Gateway
curl http://localhost:8080/actuator/health

# Bebidas
curl http://localhost:8081/actuator/health

# Clientes
curl http://localhost:8082/actuator/health

# Pedidos
curl http://localhost:8083/actuator/health
```

Todos deben responder: `{ "status": "UP" }`

### Ver logs en tiempo real

```bash
# Ver todos los logs
docker compose logs -f

# Ver solo los logs del gateway
docker compose logs -f api-gateway

# Ver solo errores
docker compose logs | grep ERROR
```

---

## 💡 Manejo de errores: casos de prueba importantes

Prueba estos escenarios para verificar que el sistema maneja los errores correctamente:

### Pedido con cliente inexistente

```
POST http://localhost:8080/api/pedidos
{
  "clienteId": "id-que-no-existe",
  "items": [...]
}
```

Respuesta esperada: `404 Not Found` con mensaje claro.

### Pedido con bebida no disponible

```
POST http://localhost:8080/api/pedidos
{
  "clienteId": "[id válido]",
  "items": [{ "bebidaId": "[id de Chai Latte - no disponible]", "cantidad": 1 }]
}
```

Respuesta esperada: `400 Bad Request` con mensaje "La bebida Chai Latte no está disponible".

### Registrar cliente con correo duplicado

```
POST http://localhost:8080/api/clientes
{ "nombre": "Sofía 2", "correo": "sofia@email.com" }
```

Respuesta esperada: `409 Conflict` con mensaje "Ya existe un cliente con ese correo".

---

## 💡 Mejoras opcionales para seguir aprendiendo

Una vez que el Michi Café básico funciona, aquí hay ideas para seguir mejorando:

### Seguridad con JWT

Agregar autenticación con tokens JWT para que solo usuarios registrados puedan hacer pedidos.

```
POST /api/auth/login → devuelve un token JWT
GET  /api/pedidos    → requiere el token en el header Authorization
```

### Notificaciones con eventos

Cuando un pedido cambia de estado, enviar una notificación al cliente usando un sistema de mensajería como **Apache Kafka** o **RabbitMQ**.

### Caché con Redis

Guardar el menú en caché para que no se consulte MongoDB en cada petición:

```java
@Cacheable("bebidas")
public List<Bebida> obtenerTodas() {
    return bebidaRepository.findAll();
}
```

### Documentación automática con Swagger

Agregar la dependencia `springdoc-openapi` para generar documentación interactiva de la API automáticamente:

```
GET http://localhost:8081/swagger-ui.html
```

### Despliegue en la nube

Subir las imágenes Docker a **Docker Hub** y desplegar en:
- **AWS ECS** (Elastic Container Service)
- **Google Cloud Run**
- **Railway** (más sencillo para empezar)

---

## 🧪 Proyecto Final: Reto completo

### El reto

Construye el Michi Café completo desde cero siguiendo todos los módulos del curso. Al terminar, tu sistema debe:

1. ✅ Tener los 3 microservicios corriendo independientemente
2. ✅ Cada servicio con su propia base de datos MongoDB
3. ✅ El API Gateway enrutando todas las peticiones
4. ✅ Todo levantado con `docker compose up`
5. ✅ Flujo completo: registrar cliente → ver menú → hacer pedido → procesar → entregar → acumular puntos

### Criterios de éxito

| Criterio | Verificación |
|----------|-------------|
| Servicios independientes | Cada uno corre en su propio puerto |
| Persistencia | Los datos sobreviven al reinicio |
| Comunicación | servicio-pedidos llama a los otros dos |
| Gateway | Todo accesible desde el puerto 8080 |
| Errores manejados | Respuestas claras cuando algo falla |
| Docker | `docker compose up` levanta todo |

---

## ✅ Resumen del Curso Completo

### Lo que construiste

Un sistema de microservicios completo para el **Michi Café** con:

| Componente | Tecnología | Puerto |
|------------|-----------|--------|
| API Gateway | Spring Cloud Gateway | 8080 |
| Servicio de Bebidas | Spring Boot + MongoDB | 8081 |
| Servicio de Clientes | Spring Boot + MongoDB | 8082 |
| Servicio de Pedidos | Spring Boot + MongoDB | 8083 |
| Base de datos | MongoDB | 27017 |
| Contenedores | Docker + Docker Compose | - |

### Las habilidades que adquiriste

| Habilidad | Módulos |
|-----------|---------|
| Programación en Java | 02, 03 |
| APIs REST con Spring Boot | 04, 05 |
| Bases de datos con MongoDB | 06, 07 |
| Arquitectura de microservicios | 08 - 13 |
| Contenedores con Docker | 14 |

### El camino que recorriste

```
No programador
      ↓
Variables y condiciones (Módulo 02)
      ↓
Clases y objetos (Módulo 03)
      ↓
Primer servidor web (Módulo 04)
      ↓
API REST completa (Módulo 05)
      ↓
Base de datos real (Módulos 06-07)
      ↓
Arquitectura de microservicios (Módulos 08-13)
      ↓
Sistema en producción con Docker (Módulo 14)
      ↓
🎉 Desarrollador de microservicios con Java
```

---

## 🎓 ¿Qué sigue después de este curso?

### Próximos pasos recomendados

1. **Spring Security** — Agregar autenticación y autorización con JWT
2. **Apache Kafka** — Comunicación asíncrona entre microservicios
3. **Kubernetes** — Orquestación de contenedores a escala
4. **CI/CD con GitHub Actions** — Automatizar pruebas y despliegues
5. **Testing** — JUnit, Mockito, pruebas de integración

### Recursos para seguir aprendiendo

- Documentación oficial de Spring: **https://spring.io/docs**
- MongoDB University (cursos gratuitos): **https://university.mongodb.com**
- Docker docs: **https://docs.docker.com**
- Baeldung (tutoriales de Spring): **https://www.baeldung.com**

---

> 🐾 "Empezaste sin saber qué era una variable y terminaste construyendo un sistema de microservicios completo con base de datos, API Gateway y Docker. Eso no es poco. Eso es mucho."
>
> **El Michi Café está abierto. ¡Bienvenido al mundo del desarrollo de software!** ☕🐱
