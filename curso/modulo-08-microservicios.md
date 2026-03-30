# Módulo 08 — Microservicios: ¿Qué son?
## 🐱 Analogía: Cada área del Michi Café es independiente

---

## 🏠 Introducción

Hasta ahora hemos construido una sola aplicación que hace todo: gestiona bebidas, clientes y pedidos. Esto se llama **arquitectura monolítica**.

En este módulo aprenderemos por qué las empresas grandes dividen sus aplicaciones en **microservicios** y cómo vamos a hacerlo en el Michi Café.

---

## 💡 Arquitectura Monolítica vs Microservicios

### El monolito: todo en uno

Una arquitectura monolítica es una sola aplicación que contiene toda la lógica del negocio.

```
┌─────────────────────────────────────┐
│         MICHI CAFÉ APP              │
│                                     │
│  ┌──────────┐  ┌──────────────┐     │
│  │ Bebidas  │  │   Clientes   │     │
│  └──────────┘  └──────────────┘     │
│  ┌──────────┐  ┌──────────────┐     │
│  │ Pedidos  │  │  Pagos       │     │
│  └──────────┘  └──────────────┘     │
│                                     │
│         Una sola base de datos      │
└─────────────────────────────────────┘
```

### Analogía 🐾

Imagina que el Michi Café tiene **un solo empleado** que hace absolutamente todo:
- Toma el pedido
- Prepara el café
- Cobra
- Limpia las mesas
- Hace el inventario
- Atiende el teléfono

¿Qué pasa si ese empleado se enferma? **Todo el café se detiene.**
¿Qué pasa si hay mucha demanda en la caja? No puedes contratar solo un cajero más, tienes que contratar otro empleado que haga todo.

Eso es el problema del monolito.

---

## 💡 Los problemas del monolito

| Problema | Descripción | Analogía |
|----------|-------------|----------|
| **Punto único de falla** | Si falla una parte, falla todo | Si el empleado se enferma, el café cierra |
| **Difícil de escalar** | Tienes que escalar todo aunque solo una parte esté saturada | Para más cajeros, contratas empleados que hacen todo |
| **Despliegues lentos** | Cambiar una cosa requiere redesplegar todo | Cambiar el precio de un café requiere cerrar el café |
| **Equipos bloqueados** | Varios equipos trabajando en el mismo código se pisan | Dos cocineros en la misma cocina pequeña |
| **Tecnología única** | Todo debe usar el mismo lenguaje y framework | Todos los empleados deben hablar el mismo idioma |

---

## 💡 La solución: Microservicios

Los microservicios dividen la aplicación en **servicios pequeños e independientes**, donde cada uno:
- Hace **una sola cosa** y la hace bien
- Tiene su **propia base de datos**
- Se **despliega de forma independiente**
- Se **comunica** con los demás a través de APIs

```
┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│  SERVICIO    │   │  SERVICIO    │   │  SERVICIO    │
│   BEBIDAS    │   │  CLIENTES    │   │   PEDIDOS    │
│              │   │              │   │              │
│  Puerto 8081 │   │  Puerto 8082 │   │  Puerto 8083 │
│              │   │              │   │              │
│  MongoDB     │   │  MongoDB     │   │  MongoDB     │
│  bebidas     │   │  clientes    │   │  pedidos     │
└──────────────┘   └──────────────┘   └──────────────┘
        ↑                  ↑                  ↑
        └──────────────────┴──────────────────┘
                           ↑
                    API Gateway :8080
                           ↑
                    Clientes / Apps
```

### Analogía 🐾

El Michi Café crece y ahora tiene **áreas especializadas**:

- **Área de Bebidas** (puerto 8081): solo gestiona el menú de bebidas
- **Área de Clientes** (puerto 8082): solo gestiona el registro de clientes
- **Área de Pedidos** (puerto 8083): solo gestiona los pedidos
- **Recepción / API Gateway** (puerto 8080): la puerta principal que dirige a cada área

Si el sistema de pedidos falla, los clientes todavía pueden ver el menú y registrarse. El café no cierra completamente.

---

## 💡 Ventajas de los microservicios

| Ventaja | Descripción | Analogía |
|---------|-------------|----------|
| **Independencia** | Si falla un servicio, los demás siguen | Si la caja falla, la cocina sigue |
| **Escalabilidad selectiva** | Escala solo lo que necesitas | Contratas más cajeros sin tocar la cocina |
| **Despliegues independientes** | Actualiza un servicio sin afectar los demás | Cambias el menú sin cerrar el café |
| **Equipos autónomos** | Cada equipo dueño de su servicio | Cada área tiene su propio jefe |
| **Tecnología flexible** | Cada servicio puede usar diferente tecnología | Cada área puede tener su propio sistema |

---

## 💡 Desventajas y retos

Los microservicios no son perfectos. También tienen retos:

| Reto | Descripción | Analogía |
|------|-------------|----------|
| **Complejidad** | Más servicios = más cosas que gestionar | Coordinar 5 áreas es más difícil que una |
| **Comunicación** | Los servicios deben hablar entre sí | Las áreas necesitan un sistema de comunicación |
| **Consistencia de datos** | Cada servicio tiene su BD, mantener consistencia es difícil | Que el inventario y los pedidos estén sincronizados |
| **Monitoreo** | Hay que vigilar muchos servicios | Necesitas cámaras en todas las áreas |

> 💡 Los microservicios son ideales para aplicaciones grandes con equipos grandes. Para proyectos pequeños, un monolito bien estructurado puede ser mejor opción.

---

## 💡 Principios de diseño de microservicios

### 1. Responsabilidad única (Single Responsibility)

Cada microservicio debe hacer **una sola cosa**.

✅ Correcto:
- Servicio de Bebidas: gestiona el catálogo de bebidas
- Servicio de Pedidos: gestiona los pedidos

❌ Incorrecto:
- Servicio de Bebidas y Pedidos: hace demasiado

### 2. Base de datos por servicio

Cada microservicio tiene **su propia base de datos**. Ningún servicio accede directamente a la base de datos de otro.

```
✅ Correcto:
Servicio Pedidos → BD Pedidos
Servicio Clientes → BD Clientes

❌ Incorrecto:
Servicio Pedidos → BD Pedidos + BD Clientes (acceso directo)
```

### 3. Comunicación a través de APIs

Los servicios se comunican entre sí usando **APIs HTTP** o **mensajes asíncronos**.

```
Servicio Pedidos necesita info del cliente:
→ Llama a la API del Servicio Clientes
→ GET http://servicio-clientes/api/clientes/{id}
```

### 4. Diseño para fallos

Cada servicio debe estar preparado para que **otros servicios fallen**.

```java
// Si el servicio de clientes no responde, el pedido igual se procesa
// con información básica del cliente
```

---

## 💡 La arquitectura del Michi Café completo

Esta es la arquitectura que vamos a construir en los próximos módulos:

```
                    ┌─────────────────┐
                    │   API GATEWAY   │
                    │   Puerto: 8080  │
                    │  (Spring Cloud  │
                    │   Gateway)      │
                    └────────┬────────┘
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
          ▼                  ▼                  ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│    SERVICIO     │ │    SERVICIO     │ │    SERVICIO     │
│    BEBIDAS      │ │    CLIENTES     │ │    PEDIDOS      │
│   Puerto: 8081  │ │   Puerto: 8082  │ │   Puerto: 8083  │
│                 │ │                 │ │                 │
│  Spring Boot    │ │  Spring Boot    │ │  Spring Boot    │
│  MongoDB        │ │  MongoDB        │ │  MongoDB        │
└─────────────────┘ └─────────────────┘ └─────────────────┘
```

### Los microservicios que construiremos

| Módulo | Servicio | Puerto | Responsabilidad |
|--------|---------|--------|-----------------|
| 09 | servicio-clientes | 8082 | Registro y gestión de clientes |
| 10 | servicio-pedidos | 8083 | Creación y seguimiento de pedidos |
| 11 | servicio-bebidas | 8081 | Catálogo de bebidas del menú |
| 12 | Comunicación | - | Cómo los servicios se hablan entre sí |
| 13 | api-gateway | 8080 | Puerta de entrada única |

---

## 💡 Estructura de carpetas del proyecto completo

Vamos a organizar el proyecto como un **monorepo**: todos los microservicios en una sola carpeta raíz.

```
michi-cafe/
├── servicio-bebidas/          ← Proyecto Spring Boot independiente
│   ├── src/
│   └── pom.xml
├── servicio-clientes/         ← Proyecto Spring Boot independiente
│   ├── src/
│   └── pom.xml
├── servicio-pedidos/          ← Proyecto Spring Boot independiente
│   ├── src/
│   └── pom.xml
├── api-gateway/               ← Proyecto Spring Boot independiente
│   ├── src/
│   └── pom.xml
└── docker-compose.yml         ← Levanta todo con un solo comando
```

Cada carpeta es un proyecto Spring Boot completamente independiente con su propio `pom.xml`.

---

## 💡 Preparando el entorno

Antes de empezar a construir los microservicios, asegúrate de tener:

1. **Java 17+** instalado
2. **Maven** instalado
3. **MongoDB** corriendo (local o Atlas)
4. **IntelliJ IDEA** (puedes abrir cada proyecto por separado)
5. **Postman** para probar

### Verificar que todo está listo

```bash
# Verificar Java
java -version
# Debe mostrar: openjdk version "17.x.x"

# Verificar Maven
mvn -version
# Debe mostrar: Apache Maven 3.x.x

# Verificar MongoDB (si es local)
mongosh --eval "db.runCommand({ connectionStatus: 1 })"
# Debe mostrar: ok: 1
```

---

## 🧪 Ejercicio del Módulo 8

### Ejercicio: Diseña tu arquitectura

Antes de escribir código, diseña en papel (o en un archivo de texto) la arquitectura del Michi Café:

1. **Lista los microservicios** que necesitarías para un café real con:
   - App móvil para clientes
   - Sistema de cocina
   - Sistema de inventario
   - Sistema de pagos
   - Notificaciones por email/SMS

2. **Para cada microservicio define**:
   - Nombre del servicio
   - Responsabilidad principal
   - Datos que gestiona
   - Con qué otros servicios necesita comunicarse

3. **Dibuja el diagrama** de cómo se conectan todos los servicios.

Este ejercicio te ayudará a pensar como un arquitecto de software.

---

## ✅ Resumen del Módulo 8

| Concepto | ¿Qué es? | Analogía del Michi Café |
|----------|----------|------------------------|
| Monolito | Una sola aplicación que hace todo | Un empleado que hace todo |
| Microservicio | Servicio pequeño con una responsabilidad | Un área especializada del café |
| API Gateway | Punto de entrada único | La recepción del café |
| BD por servicio | Cada servicio tiene su propia base de datos | Cada área tiene su propio archivero |
| Comunicación entre servicios | Los servicios se llaman entre sí por HTTP | Las áreas se mandan mensajes |
| Escalabilidad selectiva | Escalar solo lo que necesitas | Contratar más cajeros sin tocar la cocina |

---

## ➡️ Siguiente Módulo

En el **Módulo 09** construiremos el primer microservicio real: el **Servicio de Clientes**. Será un proyecto Spring Boot independiente con su propia base de datos MongoDB.

> 🐾 "Divide y vencerás. En el Michi Café, cada gatito hace lo suyo y lo hace perfecto."
