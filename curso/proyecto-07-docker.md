# Proyecto P07 — Docker: Todo el Michi Cafe en un solo comando
## Empaquetando y levantando todo con Docker Compose

> 📚 Referencia teórica: [Módulo 14 — Docker](modulo-14-docker.md)

---

## Que vamos a construir

Hasta ahora tienes 4 proyectos que debes abrir y arrancar manualmente uno por uno.
Con Docker Compose vas a poder levantar TODO el Michi Cafe con un solo comando:

```
docker compose up --build
```

Y detenerlo todo con:

```
docker compose down
```

**Tiempo estimado:** 60-90 minutos

---

## Estructura de carpetas

Todos los proyectos deben estar en una carpeta raiz llamada `michi-cafe`:

```
C:\Proyectos\michi-cafe\
├── docker-compose.yml       ← el archivo que levanta todo
├── api-gateway\
├── servicio-bebidas\
├── servicio-clientes\
└── servicio-pedidos\
```

### Mover los proyectos a la carpeta raiz

Si tus proyectos estan en `C:\Proyectos\` separados, crealos dentro de una carpeta comun:

1. Abre el explorador de archivos
2. Ve a `C:\Proyectos\`
3. Crea una carpeta nueva llamada `michi-cafe`
4. Mueve dentro: `api-gateway`, `servicio-bebidas`, `servicio-clientes`, `servicio-pedidos`

---

## Paso 1: Crear el Dockerfile para servicio-bebidas

Un Dockerfile son las instrucciones para empaquetar un servicio en una imagen Docker.

### 1.1 — Abrir la carpeta en el explorador

1. Ve a `C:\Proyectos\michi-cafe\servicio-bebidas\`
2. Verifica que no hay un archivo llamado `Dockerfile` (sin extension)

### 1.2 — Crear el Dockerfile en IntelliJ

1. Abre el proyecto `servicio-bebidas` en IntelliJ
2. Haz clic derecho en la raiz del proyecto (el nombre del proyecto arriba en el panel)
3. **New** → **File**
4. Escribe exactamente: `Dockerfile` (sin extension, con D mayuscula)
5. Presiona Enter

### 1.3 — Escribir el Dockerfile

```dockerfile
# Etapa 1: compilar el proyecto con Maven
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline -q
COPY src ./src
RUN mvn clean package -DskipTests -q

# Etapa 2: imagen final solo con el JAR
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8081
ENTRYPOINT ["java", "-jar", "app.jar"]
```

Guarda con **Ctrl + S**.

---

## Paso 2: Dockerfile para servicio-clientes

1. Abre el proyecto `servicio-clientes` en IntelliJ
2. Clic derecho en la raiz → **New** → **File** → `Dockerfile` → Enter
3. Escribe:

```dockerfile
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline -q
COPY src ./src
RUN mvn clean package -DskipTests -q

FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8082
ENTRYPOINT ["java", "-jar", "app.jar"]
```

Guarda con **Ctrl + S**.

---

## Paso 3: Dockerfile para servicio-pedidos

1. Abre el proyecto `servicio-pedidos` en IntelliJ
2. Clic derecho en la raiz → **New** → **File** → `Dockerfile` → Enter
3. Escribe:

```dockerfile
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline -q
COPY src ./src
RUN mvn clean package -DskipTests -q

FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8083
ENTRYPOINT ["java", "-jar", "app.jar"]
```

Guarda con **Ctrl + S**.

---

## Paso 4: Dockerfile para api-gateway

1. Abre el proyecto `api-gateway` en IntelliJ
2. Clic derecho en la raiz → **New** → **File** → `Dockerfile` → Enter
3. Escribe:

```dockerfile
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline -q
COPY src ./src
RUN mvn clean package -DskipTests -q

FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

Guarda con **Ctrl + S**.

---

## Paso 5: Crear el docker-compose.yml

Este es el archivo mas importante. Describe como levantar todos los servicios juntos.

### 5.1 — Crear el archivo

1. Abre el explorador de archivos y ve a `C:\Proyectos\michi-cafe\`
2. Haz clic derecho en un espacio vacio → **Nuevo** → **Documento de texto**
3. Nombra el archivo: `docker-compose.yml`
   > IMPORTANTE: debe quedar `docker-compose.yml` no `docker-compose.yml.txt`
   > Si Windows agrega .txt al final: Ve a Ver → marca "Extensiones de nombre de archivo" y quita el .txt

4. Abre el archivo con IntelliJ o con el Bloc de notas
5. Escribe exactamente esto:

```yaml
version: '3.8'

services:

  mongodb:
    image: mongo:7.0
    container_name: michi-mongodb
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db
    networks:
      - michi-network

  servicio-bebidas:
    build: ./servicio-bebidas
    container_name: michi-bebidas
    ports:
      - "8081:8081"
    environment:
      SPRING_DATA_MONGODB_HOST: mongodb
      SPRING_DATA_MONGODB_PORT: 27017
      SPRING_DATA_MONGODB_DATABASE: michicafe_bebidas
    depends_on:
      - mongodb
    networks:
      - michi-network

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

  api-gateway:
    build: ./api-gateway
    container_name: michi-gateway
    ports:
      - "8080:8080"
    depends_on:
      - servicio-bebidas
      - servicio-clientes
      - servicio-pedidos
    networks:
      - michi-network

volumes:
  mongodb_data:
    driver: local

networks:
  michi-network:
    driver: bridge
```

Guarda con **Ctrl + S**.

---

## Paso 6: Actualizar application.properties para Docker

Cuando los servicios corren en Docker, MongoDB ya no esta en `localhost`.
Esta en un contenedor llamado `mongodb`. Ya configuramos esto con variables de entorno,
pero debemos verificar que los `application.properties` usan las variables correctamente.

### Verificar servicio-bebidas

Abre `servicio-bebidas/src/main/resources/application.properties`.
Debe tener estas lineas exactamente asi:

```properties
server.port=8081
spring.application.name=servicio-bebidas
spring.data.mongodb.host=${SPRING_DATA_MONGODB_HOST:localhost}
spring.data.mongodb.port=${SPRING_DATA_MONGODB_PORT:27017}
spring.data.mongodb.database=${SPRING_DATA_MONGODB_DATABASE:michicafe_bebidas}
logging.level.com.michicafe=INFO
```

La sintaxis `${VARIABLE:valor_por_defecto}` significa:
- Si la variable de entorno existe (cuando corre en Docker) → usa la variable
- Si no existe (cuando corre en IntelliJ) → usa el valor por defecto

Aplica el mismo patron a `servicio-clientes` y `servicio-pedidos`.

---

## Paso 7: Levantar todo con Docker Compose

### 7.1 — Abrir la terminal en la carpeta correcta

1. Presiona **Windows + R** → escribe `cmd` → Enter
2. Navega a la carpeta del proyecto:
   ```
   cd C:\Proyectos\michi-cafe
   ```
3. Verifica que estas en la carpeta correcta:
   ```
   dir
   ```
   Debes ver: `docker-compose.yml`, `api-gateway`, `servicio-bebidas`, etc.

### 7.2 — Construir y levantar todo

Escribe este comando y presiona Enter:

```
docker compose up --build
```

La primera vez tarda varios minutos porque:
- Descarga las imagenes base de Java y MongoDB
- Compila cada proyecto con Maven
- Construye las imagenes Docker

Veras mucho texto en la terminal. Es normal. Espera hasta ver algo como:

```
michi-bebidas   | Started BebidasApplication in X seconds
michi-clientes  | Started ServicioClientesApplication in X seconds
michi-pedidos   | Started PedidosApplication in X seconds
michi-gateway   | Started ApiGatewayApplication in X seconds
```

### 7.3 — Verificar que todo esta corriendo

Abre una nueva terminal y escribe:

```
docker compose ps
```

Debes ver todos los contenedores con estado `Up`:

```
NAME              STATUS    PORTS
michi-gateway     Up        0.0.0.0:8080->8080/tcp
michi-bebidas     Up        0.0.0.0:8081->8081/tcp
michi-clientes    Up        0.0.0.0:8082->8082/tcp
michi-pedidos     Up        0.0.0.0:8083->8083/tcp
michi-mongodb     Up        0.0.0.0:27017->27017/tcp
```

---

## Paso 8: Prueba final completa

Con todo corriendo en Docker, haz estas pruebas en Postman.
Todo debe funcionar igual que antes pero ahora corre en contenedores.

### Prueba 1 — Ver el menu

```
GET http://localhost:8080/api/bebidas
```

### Prueba 2 — Registrar un cliente nuevo

```
POST http://localhost:8080/api/clientes
Body:
```
```json
{
  "nombre": "Ana Garcia",
  "correo": "ana@email.com"
}
```

### Prueba 3 — Hacer un pedido completo

```
POST http://localhost:8080/api/pedidos
Body:
```
```json
{
  "clienteId": "PEGA_ID_DE_ANA",
  "items": [
    { "bebidaId": "PEGA_ID_CAPPUCCINO", "cantidad": 2 }
  ],
  "paraLlevar": true,
  "notas": "Extra caliente por favor"
}
```

### Prueba 4 — Entregar el pedido y verificar puntos

```
PATCH http://localhost:8080/api/pedidos/PEGA_ID_PEDIDO/estado?valor=entregado
```

Luego verifica los puntos de Ana:
```
GET http://localhost:8080/api/clientes/PEGA_ID_ANA
```

---

## Comandos Docker utiles

```bash
# Ver logs de un servicio especifico
docker logs michi-bebidas

# Ver logs en tiempo real
docker logs -f michi-pedidos

# Detener todo (los datos se conservan)
docker compose stop

# Volver a levantar todo sin reconstruir
docker compose start

# Detener y eliminar contenedores (datos conservados en volumen)
docker compose down

# Detener, eliminar contenedores Y borrar todos los datos
docker compose down -v
```

---

## Que construiste

- Un `Dockerfile` para cada uno de los 4 servicios
- Un `docker-compose.yml` que orquesta todo
- Variables de entorno para configuracion dinamica
- El Michi Cafe completo corriendo en contenedores Docker

### Arquitectura final

```
                    Puerto 8080
                    API Gateway
                        |
        ________________|________________
        |               |               |
   Puerto 8081     Puerto 8082     Puerto 8083
  Svc Bebidas     Svc Clientes    Svc Pedidos
        |               |               |
        └───────────────┴───────────────┘
                        |
                   Puerto 27017
                    MongoDB
                 (volumen persistente)
```

---

## Felicidades — El Michi Cafe esta completo!

Construiste desde cero un sistema de microservicios completo:

- Java 17 + Spring Boot 4.1.0
- 3 microservicios independientes con sus propias bases de datos MongoDB
- Comunicacion entre servicios con RestTemplate
- API Gateway como punto de entrada unico
- Todo empaquetado y desplegado con Docker

### Lo que aprendiste en este curso

| Tema | Donde lo viste |
|------|---------------|
| Java desde cero | Modulos 02 y 03 |
| Spring Boot y APIs REST | Modulos 04 y 05 |
| MongoDB | Modulos 06 y 07 |
| Arquitectura de microservicios | Modulos 08-12 |
| API Gateway | Modulo 13, Proyecto P06 |
| Docker | Modulo 14, Proyecto P07 |

> 🐾 "Empezaste sin saber que era una variable. Terminaste con un sistema de
> microservicios corriendo en Docker. Eso es mucho. Bienvenido al mundo
> del desarrollo de software."
