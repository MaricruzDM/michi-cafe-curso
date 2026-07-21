# Proyecto P02 — Primer Proyecto Spring Boot
## Creando la base del Michi Café

> 📚 Referencias teóricas:
> - [Módulo 04 — Spring Boot](modulo-04-spring-boot.md) → Estructura del proyecto, anotaciones, capas
> - [Módulo 02 — Java desde Cero](modulo-02-java-desde-cero.md) → Variables y tipos de datos

---

## 🏠 ¿Qué vamos a construir?

En este módulo vamos a:
1. Crear el primer proyecto Spring Boot desde cero usando Spring Initializr
2. Abrirlo en IntelliJ IDEA
3. Entender cada archivo que se genera
4. Crear nuestro primer endpoint que responde "¡Bienvenido al Michi Café!"
5. Probarlo en el navegador y en Postman

**Tiempo estimado:** 45-60 minutos

---

## 🛠️ Paso 1: Crear el proyecto en Spring Initializr

Spring Initializr genera el esqueleto del proyecto automáticamente.

### 1.1 — Abrir Spring Initializr

1. Abre tu navegador (Chrome, Firefox, Edge)
2. Ve a la dirección: **https://start.spring.io**
3. Verás una página con un formulario de configuración

### 1.2 — Configurar el proyecto

Configura cada campo **exactamente** como se muestra:

**Sección izquierda:**

| Campo | Valor a seleccionar |
|-------|-------------------|
| Project | `Maven` (haz clic en el botón Maven) |
| Language | `Java` (ya debe estar seleccionado) |
| Spring Boot | `4.1.0` (o la más reciente disponible) |

> ⚠️ Asegúrate de NO seleccionar versiones marcadas con `(SNAPSHOT)` o `(M1)`,
> esas son versiones de prueba inestables. Elige solo la versión estable.

**Sección "Project Metadata":**

| Campo | Valor a escribir |
|-------|-----------------|
| Group | `com.michicafe` |
| Artifact | `michicafe` |
| Name | `michicafe` |
| Description | `Michi Café - Sistema de microservicios` |
| Package name | Se llena solo: `com.michicafe.michicafe` |
| Packaging | `Jar` |
| Java | `17` |

### 1.3 — Agregar dependencias

Las dependencias son las librerías que necesita nuestro proyecto.

1. Haz clic en el botón **"ADD DEPENDENCIES"** (lado derecho de la página, en azul)
2. Se abre un buscador. Escribe: `web`
3. Aparecerá `Spring Web` — haz clic en él para seleccionarlo
4. Verás que se agrega en el lado derecho bajo "Dependencies"

> 💡 Por ahora solo agregamos `Spring Web`. En módulos posteriores agregaremos MongoDB y más.

### 1.4 — Generar y descargar el proyecto

1. Haz clic en el botón verde **"GENERATE"** (abajo al centro)
2. Se descargará automáticamente un archivo ZIP llamado `michicafe.zip`
3. Busca ese archivo en tu carpeta de **Descargas**

### 1.5 — Descomprimir el proyecto

1. Haz clic derecho sobre `michicafe.zip`
2. Selecciona **"Extraer todo..."**
3. En la ventana que aparece, haz clic en **Examinar** y elige una ubicación fácil de encontrar,
   por ejemplo: `C:\Proyectos\michicafe`
4. Haz clic en **Extraer**

---

## 🛠️ Paso 2: Abrir el proyecto en IntelliJ IDEA

### 2.1 — Abrir IntelliJ IDEA

1. Haz doble clic en el ícono de IntelliJ IDEA en tu escritorio
2. Aparecerá la pantalla de bienvenida con el título **"Welcome to IntelliJ IDEA"**

### 2.2 — Abrir el proyecto

1. Haz clic en el botón **"Open"** (en la pantalla de bienvenida)
2. Se abre el explorador de archivos
3. Navega hasta donde descomprimiste el proyecto (ej: `C:\Proyectos\michicafe`)
4. Haz clic **una vez** sobre la carpeta `michicafe` para seleccionarla
5. Haz clic en **"OK"**

### 2.3 — Esperar a que IntelliJ cargue el proyecto

La primera vez que abres un proyecto Spring Boot, IntelliJ descarga todas las
dependencias. Esto puede tardar **2-5 minutos** dependiendo de tu conexión.

Sabrás que terminó cuando:
- La barra de progreso en la parte inferior de la pantalla desaparece
- Ya no hay mensajes de "Downloading..." en la esquina inferior

> ⚠️ No cierres IntelliJ mientras descarga. Espera pacientemente.

### 2.4 — Explorar la estructura del proyecto

En el lado izquierdo de IntelliJ verás el panel **"Project"** con la estructura de archivos.
Si no lo ves, haz clic en el ícono de carpeta en la barra lateral izquierda.

Deberías ver esto:

```
michicafe
├── src
│   ├── main
│   │   ├── java
│   │   │   └── com.michicafe.michicafe
│   │   │       └── MichicafeApplication.java   ← Punto de entrada
│   │   └── resources
│   │       └── application.properties           ← Configuración
│   └── test
│       └── java
│           └── com.michicafe.michicafe
│               └── MichicafeApplicationTests.java
└── pom.xml                                       ← Lista de dependencias
```

---

## 🛠️ Paso 3: Explorar los archivos generados

### 3.1 — Ver el archivo principal

1. En el panel izquierdo, haz clic en la flecha junto a `src` para expandirlo
2. Sigue expandiendo: `main` → `java` → `com.michicafe.michicafe`
3. Haz **doble clic** en `MichicafeApplication` para abrirlo

Verás este código:

```java
package com.michicafe.michicafe;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MichicafeApplication {

    public static void main(String[] args) {
        SpringApplication.run(MichicafeApplication.class, args);
    }
}
```

> 📚 ¿No entiendes qué hace `@SpringBootApplication`?
> Consulta [Módulo 04 — La clase principal](modulo-04-spring-boot.md#la-clase-principal-el-interruptor-que-enciende-todo)

### 3.2 — Ver el `pom.xml`

1. En el panel izquierdo, haz doble clic en `pom.xml` (está en la raíz del proyecto)
2. Verás el archivo de dependencias. Busca la sección `<dependencies>`:

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

> 📚 ¿No sabes qué es el `pom.xml`?
> Consulta [Módulo 04 — Maven y el pom.xml](modulo-04-spring-boot.md#qué-es-maven-y-el-archivo-pomxml)

---

## 🛠️ Paso 4: Crear el primer Controller

Vamos a crear la clase que recibirá las peticiones HTTP.

### 4.1 — Crear el paquete `controller`

En Java, los paquetes son carpetas que organizan las clases.

1. En el panel izquierdo, busca la carpeta `com.michicafe.michicafe`
   (está dentro de `src` → `main` → `java`)
2. Haz **clic derecho** sobre `com.michicafe.michicafe`
3. En el menú que aparece, pasa el mouse sobre **"New"**
4. En el submenú, haz clic en **"Package"**
5. En el campo de texto que aparece, escribe exactamente: `controller`
6. Presiona **Enter**

Verás que se creó una nueva carpeta llamada `controller` dentro de tu paquete.

> 💡 El paquete completo queda: `com.michicafe.michicafe.controller`

### 4.2 — Crear la clase `MichiController`

1. Haz **clic derecho** sobre el paquete `controller` que acabas de crear
2. Pasa el mouse sobre **"New"**
3. Haz clic en **"Java Class"**
4. En el campo de texto, escribe exactamente: `MichiController`
5. Asegúrate de que esté seleccionado **"Class"** (no Interface, Enum, etc.)
6. Presiona **Enter**

Se abrirá un archivo nuevo con este contenido inicial:

```java
package com.michicafe.michicafe.controller;

public class MichiController {
}
```

### 4.3 — Escribir el código del Controller

Vamos a reemplazar el contenido del archivo con el siguiente código.
**Escríbelo tú mismo**, no lo copies y pegues — escribirlo te ayuda a aprenderlo:

```java
package com.michicafe.michicafe.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class MichiController {

    @GetMapping("/")
    public String bienvenida() {
        return "¡Bienvenido al Michi Café! 🐱☕";
    }

    @GetMapping("/estado")
    public String estado() {
        return "El Michi Café está ABIERTO y listo para servirte 🐾";
    }
}
```

### 4.4 — Entender los imports (las importaciones)

Cuando escribiste `@RestController` y `@GetMapping`, IntelliJ los subrayó en rojo
porque aún no sabe de dónde vienen esas clases. Necesitas **importarlas**.

**Forma automática (recomendada):**
1. Haz clic sobre `@RestController` (donde está el subrayado rojo)
2. Presiona las teclas **Alt + Enter** al mismo tiempo
3. Aparecerá un menú con la sugerencia `Import class`
4. Haz clic en `Import class`
5. Repite para `@GetMapping`

Verás que IntelliJ agrega automáticamente las líneas de import arriba:
```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
```

> 💡 Los imports le dicen a Java exactamente en qué librería vive cada clase
> que estamos usando. Sin el import, Java no sabe qué es `@RestController`.

---

## 🛠️ Paso 5: Ejecutar el servidor por primera vez

### 5.1 — Iniciar la aplicación

1. En el panel izquierdo, haz doble clic en `MichicafeApplication` para abrirlo
2. Busca el método `main` (la línea que dice `public static void main`)
3. Verás un triángulo verde ▶ a la izquierda de esa línea
4. Haz clic en ese triángulo verde ▶
5. Selecciona **"Run 'MichicafeApplication'"**

### 5.2 — Observar la consola

En la parte inferior de IntelliJ se abrirá la consola. Verás muchos mensajes.
Espera hasta ver algo como:

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/

 :: Spring Boot ::               (v4.1.0)

...
Started MichicafeApplication in 2.345 seconds (process running for 2.8)
```

La línea más importante es: `Started MichicafeApplication in X seconds`

✅ ¡El servidor está corriendo!

### 5.3 — Ver el resultado en el navegador

1. Abre tu navegador
2. En la barra de direcciones escribe: `http://localhost:8080/`
3. Presiona **Enter**

Deberías ver:
```
¡Bienvenido al Michi Café! 🐱☕
```

Ahora prueba también: `http://localhost:8080/estado`

---

## 🛠️ Paso 6: Probar con Postman

### 6.1 — Abrir Postman y crear una petición

1. Abre **Postman**
2. Haz clic en el botón **"+"** (nueva pestaña) o en **"New"** → **"HTTP"**
3. Verás una barra donde dice "Enter URL or paste text"

### 6.2 — Hacer tu primera petición GET

1. Asegúrate de que en el desplegable de la izquierda diga **"GET"**
2. En el campo de URL escribe: `http://localhost:8080/`
3. Haz clic en el botón azul **"Send"**
4. En la parte inferior verás la **respuesta**:
   - **Status:** `200 OK` (en verde)
   - **Body:** `¡Bienvenido al Michi Café! 🐱☕`

### 6.3 — Probar el segundo endpoint

1. Cambia la URL a: `http://localhost:8080/estado`
2. Haz clic en **"Send"**
3. Deberías ver: `El Michi Café está ABIERTO y listo para servirte 🐾`

---

## 🛠️ Paso 7: Detener el servidor

Cuando termines de trabajar, debes detener el servidor.

1. En la consola de IntelliJ (parte inferior), busca el botón cuadrado rojo ⏹
2. Haz clic en él
3. El servidor se detendrá y verás: `Process finished with exit code 130`

---

## 🎉 ¡Felicidades!

Acabas de crear y ejecutar tu primer servidor Spring Boot.
Este es el fundamento de todo lo que vamos a construir.

### ¿Qué hiciste en este módulo?

- ✅ Creaste un proyecto Spring Boot 4.1.0 con Java 17
- ✅ Lo abriste en IntelliJ IDEA
- ✅ Entendiste la estructura de archivos
- ✅ Creaste un paquete `controller`
- ✅ Creaste una clase `MichiController` con dos endpoints
- ✅ Aprendiste a importar clases en IntelliJ (Alt + Enter)
- ✅ Ejecutaste el servidor
- ✅ Probaste los endpoints en el navegador y en Postman

---

## ➡️ Siguiente paso

En el **[Proyecto P03 — Servicio de Bebidas](proyecto-03-servicio-bebidas.md)**
vamos a construir el primer microservicio real: una API REST completa para
gestionar el menú del Michi Café, conectada a MongoDB.

> 🐾 "El primer `Hello World` es el comienzo de todo."
