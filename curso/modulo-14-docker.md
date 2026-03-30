# Módulo 14 — Docker: Empaquetando el Michi Café
## 🐱 Analogía: Cajas de transporte para cada gatito

---

## 🏠 Introducción

Tenemos 4 servicios funcionando en nuestra computadora:
- `api-gateway` en el puerto 8080
- `servicio-bebidas` en el puerto 8081
- `servicio-clientes` en el puerto 8082
- `servicio-pedidos` en el puerto 8083

El problema es que todo esto solo funciona en **tu computadora**. Si quieres que otra persona lo use, o si quieres subirlo a un servidor en internet, tendrías que:
1. Instalar Java en ese servidor
2. Instalar MongoDB
3. Configurar los puertos
4. Asegurarte de que las versiones sean compatibles

Esto es un dolor de cabeza. **Docker** resuelve exactamente ese problema.

---

## 💡 ¿Qué es Docker?

Docker es una herramienta que permite empaquetar una aplicación junto con **todo lo que necesita para funcionar** (Java, configuración, dependencias) en una unidad llamada **contenedor**.

Un contenedor puede correr en cualquier computadora que tenga Docker instalado, sin importar el sistema operativo o la configuración.

### Analogía 🐾

Imagina que el Michi Café quiere abrir sucursales en otras ciudades.

Sin Docker, tendrías que:
- Construir el local desde cero
- Comprar todos los equipos
- Capacitar a los empleados
- Configurar todo igual que la sucursal original

Con Docker, es como si pudieras meter **toda la sucursal** en una caja de transporte:
- El local ya armado
- Los equipos instalados
- Los empleados capacitados
- La configuración lista

Llegas a la nueva ciudad, abres la caja, y el café está listo para operar.

---

## 💡 Conceptos clave de Docker

| Concepto | ¿Qué es? | Analogía |
|----------|---------|----------|
| **Imagen** | Plantilla para crear contenedores | El plano de la sucursal |
| **Contenedor** | Instancia en ejecución de una imagen | La sucursal ya construida y abierta |
| **Dockerfile** | Instrucciones para construir una imagen | El manual de construcción de la sucursal |
| **Docker Hub** | Repositorio de imágenes públicas | La tienda de planos de sucursales |
| **Docker Compose** | Herramienta para manejar múltiples contenedores | El coordinador de todas las sucursales |
| **Volumen** | Almacenamiento persistente fuera del contenedor | El archivero que sobrevive si la sucursal cierra |
| **Red** | Comunicación entre contenedores | El sistema de teléfonos internos entre sucursales |

---

## 🛠️ Instalación de Docker

1. Ve a: **https://www.docker.com/products/docker-desktop**
2. Descarga **Docker Desktop** para tu sistema operativo
3. Instala y reinicia tu computadora
4. Verifica la instalación:

```bash
docker --version
# Docker version 24.x.x

docker compose version
# Docker Compose version 2.x.x
```

---

## 💡 El Dockerfile: el manual de construcción

Un `Dockerfile` es un archivo de texto con instrucciones para construir una imagen Docker.

### Dockerfile para un microservicio Spring Boot

```dockerfile
# Dockerfile para servicio-bebidas
# (el mismo patrón aplica para todos los microservicios)

# ── Etapa 1: Compilar el proyecto ────────────────────────────────────────────
FROM maven:3.9-eclipse-temurin-17 AS build
# Usamos una imagen que ya tiene Maven y Java 17

WORKDIR /app
# Nos movemos a la carpeta /app dentro del contenedor

COPY pom.xml .
# Copiamos el pom.xml primero (para aprovechar el caché de Docker)

RUN mvn dependency:go-offline
# Descargamos las dependencias (esto se cachea si pom.xml no cambia)

COPY src ./src
# Copiamos el código fuente

RUN mvn clean package -DskipTests
# Compilamos y empaquetamos (generamos el .jar)

# ── Etapa 2: Crear la imagen final (más pequeña) ─────────────────────────────
FROM eclipse-temurin:17-jre-alpine
# Solo necesitamos el JRE (Java Runtime), no el JDK completo
# alpine = versión mínima de Linux (muy pequeña)

WORKDIR /app

COPY --from=build /app/target/*.jar app.jar
# Copiamos solo el .jar de la etapa de compilación

EXPOSE 8081
# Indicamos que el contenedor usa el puerto 8081

ENTRYPOINT ["java", "-jar", "app.jar"]
# Comando que se ejecuta al iniciar el contenedor
```

### ¿Por qué dos etapas?

La construcción en dos etapas (multi-stage build) es una buena práctica:
- La primera etapa tiene Maven + JDK (imagen grande ~500MB)
- La segunda etapa solo tiene el JRE + el .jar (imagen pequeña ~150MB)

La imagen final es mucho más pequeña y segura porque no incluye las herramientas de compilación.

---

## 💡 El `.dockerignore`: lo que no debe entrar en la caja

```
# .dockerignore (en la raíz de cada microservicio)
target/
*.log
.git/
.idea/
*.iml
```

Es como decirle al empacador: "No metas los borradores ni los archivos temporales en la caja".

---

## 💡 Docker Compose: el coordinador de todo

**Docker Compose** permite definir y levantar todos los contenedores del Michi Café con un solo comando.

### El archivo `docker-compose.yml`

```yaml
# docker-compose.yml (en la raíz del proyecto michi-cafe/)
version: '3.8'

services:

  # ── Base de datos MongoDB ──────────────────────────────────────────────────
  mongodb:
    image: mongo:7.0                    # Imagen oficial de MongoDB
    container_name: michi-mongodb
    ports:
      - "27017:27017"                   # puerto_host:puerto_contenedor
    volumes:
      - mongodb_data:/data/db           # Persistencia de datos
    environment:
      MONGO_INITDB_DATABASE: michicafe
    networks:
      - michi-network

  # ── Servicio de Bebidas ────────────────────────────────────────────────────
  servicio-bebidas:
    build: ./servicio-bebidas           # Construye desde el Dockerfile local
    container_name: michi-bebidas
    ports:
      - "8081:8081"
    environment:
      SPRING_DATA_MONGODB_HOST: mongodb # Nombre del contenedor de MongoDB
      SPRING_DATA_MONGODB_PORT: 27017
      SPRING_DATA_MONGODB_DATABASE: michicafe_bebidas
    depends_on:
      - mongodb                         # Espera a que MongoDB esté listo
    networks:
      - michi-network

  # ── Servicio de Clientes ───────────────────────────────────────────────────
  servicio-clientes:
    build: ./servicio-clientes
    container_name: michi-clientes
    ports:
      - "8082:8082"
    environment:
      SPRING_DATA_MONGODB_HOST: mongodb
      SPRING_DATA_MONGODB_PORT: 27017
      SPRING_DATA_MONGODB_DATABASE: michicafe_clientes
    depends_on:
      - mongodb
    networks:
      - michi-network

  # ── Servicio de Pedidos ────────────────────────────────────────────────────
  servicio-pedidos:
    build: ./servicio-pedidos
    container_name: michi-pedidos
    ports:
      - "8083:8083"
    environment:
      SPRING_DATA_MONGODB_HOST: mongodb
      SPRING_DATA_MONGODB_PORT: 27017
      SPRING_DATA_MONGODB_DATABASE: michicafe_pedidos
      SERVICIO_BEBIDAS_URL: http://servicio-bebidas:8081
      SERVICIO_CLIENTES_URL: http://servicio-clientes:8082
    depends_on:
      - mongodb
      - servicio-bebidas
      - servicio-clientes
    networks:
      - michi-network

  # ── API Gateway ────────────────────────────────────────────────────────────
  api-gateway:
    build: ./api-gateway
    container_name: michi-gateway
    ports:
      - "8080:8080"                     # Solo este puerto es accesible desde fuera
    environment:
      SERVICIO_BEBIDAS_URL: http://servicio-bebidas:8081
      SERVICIO_CLIENTES_URL: http://servicio-clientes:8082
      SERVICIO_PEDIDOS_URL: http://servicio-pedidos:8083
    depends_on:
      - servicio-bebidas
      - servicio-clientes
      - servicio-pedidos
    networks:
      - michi-network

# ── Volúmenes: almacenamiento persistente ─────────────────────────────────────
volumes:
  mongodb_data:
    driver: local

# ── Redes: comunicación interna entre contenedores ────────────────────────────
networks:
  michi-network:
    driver: bridge
```

### Analogía del docker-compose.yml 🐾

Es como el plano maestro del Michi Café completo:
- Define cada área (servicio)
- Define cómo se conectan entre sí (networks)
- Define dónde guardan sus archivos (volumes)
- Define en qué orden abrir cada área (depends_on)

---

## 💡 Configuración dinámica con variables de entorno

En el `docker-compose.yml` usamos variables de entorno para configurar los servicios. Necesitamos que Spring Boot las lea.

Actualiza el `application.properties` de cada servicio para usar variables de entorno:

```properties
# servicio-bebidas/src/main/resources/application.properties

server.port=8081
spring.application.name=servicio-bebidas

# Lee la variable de entorno SPRING_DATA_MONGODB_HOST
# Si no existe, usa "localhost" como valor por defecto
spring.data.mongodb.host=${SPRING_DATA_MONGODB_HOST:localhost}
spring.data.mongodb.port=${SPRING_DATA_MONGODB_PORT:27017}
spring.data.mongodb.database=${SPRING_DATA_MONGODB_DATABASE:michicafe_bebidas}
```

```properties
# servicio-pedidos/src/main/resources/application.properties

server.port=8083
spring.application.name=servicio-pedidos

spring.data.mongodb.host=${SPRING_DATA_MONGODB_HOST:localhost}
spring.data.mongodb.port=${SPRING_DATA_MONGODB_PORT:27017}
spring.data.mongodb.database=${SPRING_DATA_MONGODB_DATABASE:michicafe_pedidos}

# URLs de otros servicios
servicio.bebidas.url=${SERVICIO_BEBIDAS_URL:http://localhost:8081}
servicio.clientes.url=${SERVICIO_CLIENTES_URL:http://localhost:8082}
```

---

## 💡 Comandos esenciales de Docker

### Construir y levantar todo

```bash
# Desde la carpeta raíz michi-cafe/

# Construir las imágenes y levantar todos los contenedores
docker compose up --build

# Levantar en segundo plano (sin bloquear la terminal)
docker compose up --build -d
```

### Ver el estado de los contenedores

```bash
# Ver contenedores corriendo
docker compose ps

# Salida esperada:
# NAME               STATUS    PORTS
# michi-gateway      Up        0.0.0.0:8080->8080/tcp
# michi-bebidas      Up        0.0.0.0:8081->8081/tcp
# michi-clientes     Up        0.0.0.0:8082->8082/tcp
# michi-pedidos      Up        0.0.0.0:8083->8083/tcp
# michi-mongodb      Up        0.0.0.0:27017->27017/tcp
```

### Ver los logs

```bash
# Logs de todos los servicios
docker compose logs

# Logs de un servicio específico
docker compose logs servicio-bebidas

# Logs en tiempo real
docker compose logs -f servicio-pedidos
```

### Detener y limpiar

```bash
# Detener todos los contenedores
docker compose stop

# Detener y eliminar contenedores (los datos en volúmenes se conservan)
docker compose down

# Detener, eliminar contenedores Y volúmenes (borra todos los datos)
docker compose down -v
```

### Comandos útiles para depurar

```bash
# Entrar a la terminal de un contenedor
docker exec -it michi-bebidas bash

# Ver los logs de un contenedor específico
docker logs michi-bebidas

# Ver el uso de recursos
docker stats
```

---

## 💡 Estructura final del proyecto

```
michi-cafe/                          ← Carpeta raíz del proyecto
├── docker-compose.yml               ← Orquestador de todos los servicios
├── api-gateway/
│   ├── Dockerfile
│   └── src/...
├── servicio-bebidas/
│   ├── Dockerfile
│   └── src/...
├── servicio-clientes/
│   ├── Dockerfile
│   └── src/...
└── servicio-pedidos/
    ├── Dockerfile
    └── src/...
```

---

## 💡 Redes en Docker: cómo se comunican los contenedores

Cuando los contenedores están en la misma red (`michi-network`), pueden comunicarse usando el **nombre del contenedor** como hostname.

```
# Dentro del contenedor servicio-pedidos:
# Para llamar al servicio-bebidas NO usas localhost:8081
# Usas el nombre del contenedor: servicio-bebidas:8081

http://servicio-bebidas:8081/api/bebidas   ✅
http://localhost:8081/api/bebidas          ❌ (no funciona entre contenedores)
```

### Analogía 🐾

Es como el sistema de teléfonos internos del Michi Café:
- Desde la cocina no llamas al número externo de la caja
- Llamas al **interno** de la caja: "extensión caja"
- Docker hace lo mismo: cada contenedor tiene su propio "nombre interno"

---

## 🧪 Ejercicios del Módulo 14

### Ejercicio 1: Tu primer contenedor

1. Crea el `Dockerfile` para el `servicio-bebidas`
2. Construye la imagen: `docker build -t michi-bebidas .`
3. Corre el contenedor: `docker run -p 8081:8081 michi-bebidas`
4. Verifica que responde en `http://localhost:8081/api/bebidas`

### Ejercicio 2: Levantar todo con Compose

1. Crea el `docker-compose.yml` en la raíz del proyecto
2. Ejecuta `docker compose up --build`
3. Verifica que todos los servicios están corriendo con `docker compose ps`
4. Prueba todas las rutas a través del Gateway en el puerto 8080

### Ejercicio 3: Persistencia de datos

1. Levanta todo con `docker compose up -d`
2. Crea algunas bebidas y clientes via Postman
3. Ejecuta `docker compose stop` (detiene pero no borra)
4. Ejecuta `docker compose start` (vuelve a levantar)
5. Verifica que los datos siguen ahí

---

## ✅ Resumen del Módulo 14

| Concepto | ¿Qué es? | Analogía del Michi Café |
|----------|----------|------------------------|
| Docker | Herramienta de contenedores | El sistema de cajas de transporte |
| Imagen | Plantilla para crear contenedores | El plano de la sucursal |
| Contenedor | Aplicación en ejecución aislada | La sucursal ya construida |
| Dockerfile | Instrucciones para construir la imagen | El manual de construcción |
| Docker Compose | Orquestador de múltiples contenedores | El coordinador de todas las sucursales |
| Volumen | Almacenamiento persistente | El archivero que sobrevive al cierre |
| Red | Comunicación entre contenedores | El sistema de teléfonos internos |
| Variable de entorno | Configuración externa al código | Las instrucciones que llegan con la caja |

---

## ➡️ Siguiente Módulo

En el **Módulo 15** construiremos el **Proyecto Final**: el Michi Café completo en producción. Integraremos todo lo aprendido, haremos pruebas end-to-end y veremos cómo desplegar el sistema completo.

> 🐾 "Con Docker, el Michi Café puede abrir sucursales en cualquier servidor del mundo en cuestión de segundos."
