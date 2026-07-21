# Proyecto P01 — Instalación del Entorno
## Preparando la cocina antes de abrir el café

> 📚 Referencia teórica: [Módulo 01 — Introducción](modulo-01-introduccion.md)

---

## 🏠 ¿Qué vamos a hacer en este módulo?

Vamos a instalar en tu computadora todas las herramientas que necesitas para
desarrollar el Michi Café. Al final de este módulo tendrás todo listo para
empezar a escribir código.

**Tiempo estimado:** 30-45 minutos

---

## ✅ Lista de lo que instalaremos

- [ ] Java 17 (JDK) — el lenguaje de programación
- [ ] IntelliJ IDEA Community — el editor de código
- [ ] MongoDB Community — la base de datos
- [ ] MongoDB Compass — la interfaz visual de MongoDB
- [ ] Postman — para probar las APIs
- [ ] Docker Desktop — para empaquetar todo al final

---

## 🛠️ Paso 1: Instalar Java 17

Java es el lenguaje con el que vamos a programar. Necesitamos instalar el
**JDK** (Java Development Kit), que incluye todo lo necesario para desarrollar.

### 1.1 — Descargar Java 17

1. Abre tu navegador y ve a: **https://adoptium.net/temurin/releases/**
2. Verás una página con opciones de descarga. Configura los filtros así:
   - **Version:** `17 - LTS`
   - **Operating System:** el de tu computadora (Windows, macOS, Linux)
   - **Architecture:** `x64` (si tu computadora es de 64 bits, que es lo más común)
   - **Package Type:** `JDK`
   - **Vendor:** `Eclipse Temurin`
3. Haz clic en el botón azul que dice **`.msi`** (en Windows) o **`.pkg`** (en Mac)
4. Se descargará un archivo de instalación

> 💡 ¿No sabes si tu Windows es de 32 o 64 bits? Haz clic derecho en
> "Este equipo" → "Propiedades" → busca "Tipo de sistema". La gran mayoría
> de computadoras modernas son de 64 bits.

### 1.2 — Instalar Java 17 en Windows

1. Busca el archivo descargado (normalmente en tu carpeta **Descargas**)
   - Se llama algo como: `OpenJDK17U-jdk_x64_windows_hotspot_17.x.x_x.msi`
2. Haz **doble clic** sobre el archivo para iniciar la instalación
3. Aparecerá una ventana del instalador. Haz clic en **Next**
4. En la pantalla de opciones, asegúrate de que esté marcado:
   - ✅ `Add to PATH` (muy importante, esto permite usar Java desde cualquier lugar)
   - ✅ `Set JAVA_HOME variable`
5. Haz clic en **Next** → **Install**
6. Cuando termine, haz clic en **Finish**

### 1.3 — Verificar que Java se instaló correctamente

1. Presiona las teclas **Windows + R** al mismo tiempo
2. Escribe `cmd` y presiona **Enter**
3. Se abrirá una ventana negra (la terminal). Escribe exactamente esto y presiona Enter:

```
java -version
```

4. Deberías ver algo como:

```
openjdk version "17.0.x" 2024-xx-xx LTS
OpenJDK Runtime Environment temurin-17.x.x (build 17.0.x+xx-LTS)
OpenJDK 64-Bit Server VM temurin-17.x.x
```

✅ Si ves algo parecido, ¡Java está instalado correctamente!

❌ Si ves `'java' is not recognized as an internal or external command`, cierra
la terminal, vuelve a abrirla e intenta de nuevo. Si sigue fallando, reinicia
tu computadora e intenta otra vez.

---

## 🛠️ Paso 2: Instalar IntelliJ IDEA Community

IntelliJ IDEA es donde vamos a escribir todo nuestro código. Usamos la versión
**Community** que es completamente gratuita.

### 2.1 — Descargar IntelliJ IDEA

1. Ve a: **https://www.jetbrains.com/idea/download/**
2. La página detecta automáticamente tu sistema operativo
3. Baja hasta encontrar la sección **IntelliJ IDEA Community Edition**
4. Haz clic en el botón negro que dice **Download**
   - ⚠️ Asegúrate de descargar la versión **Community**, NO la Ultimate (que es de pago)

### 2.2 — Instalar IntelliJ IDEA en Windows

1. Busca el archivo descargado (algo como `ideaIC-2024.x.x.exe`)
2. Haz **doble clic** para iniciar la instalación
3. Haz clic en **Next**
4. En la pantalla "Installation Options", marca estas opciones:
   - ✅ `Create Desktop Shortcut` (para abrirlo fácilmente)
   - ✅ `Add "Open Folder as Project"` (muy útil)
   - ✅ `.java` (asociar archivos Java con IntelliJ)
5. Haz clic en **Next** → **Install**
6. Cuando termine, marca `Run IntelliJ IDEA` y haz clic en **Finish**

### 2.3 — Configuración inicial de IntelliJ IDEA

La primera vez que abres IntelliJ te hace algunas preguntas:

1. **"Do you want to import IntelliJ IDEA settings?"**
   → Selecciona **"Do not import settings"** y haz clic en **OK**

2. Aparecerá la pantalla de bienvenida con el logo de IntelliJ.
   Por ahora ciérrala — la usaremos en el siguiente módulo del proyecto.

---

## 🛠️ Paso 3: Instalar MongoDB Community

MongoDB es la base de datos donde guardaremos toda la información del Michi Café.

### 3.1 — Descargar MongoDB

1. Ve a: **https://www.mongodb.com/try/download/community**
2. Configura las opciones:
   - **Version:** la más reciente (7.x.x o superior)
   - **Platform:** Windows
   - **Package:** `msi`
3. Haz clic en **Download**

### 3.2 — Instalar MongoDB en Windows

1. Busca el archivo descargado (algo como `mongodb-windows-x86_64-7.x.x.msi`)
2. Haz **doble clic** para iniciar la instalación
3. Haz clic en **Next** → acepta los términos → **Next**
4. En "Choose Setup Type", selecciona **Complete**
5. En "Service Configuration":
   - Deja todo como está
   - ✅ Asegúrate de que `Install MongoDB as a Service` esté marcado
   - Esto hace que MongoDB se inicie automáticamente cuando enciendes la computadora
6. En la siguiente pantalla:
   - ✅ `Install MongoDB Compass` debe estar marcado (lo necesitamos)
7. Haz clic en **Next** → **Install**
8. Cuando termine, haz clic en **Finish**

> 💡 MongoDB Compass se instala junto con MongoDB. Es la interfaz visual
> que nos permite ver y editar los datos sin escribir comandos.

### 3.3 — Verificar que MongoDB está corriendo

1. Presiona **Windows + R**, escribe `services.msc` y presiona **Enter**
2. Se abre la ventana de Servicios de Windows
3. Busca en la lista `MongoDB Server (MongoDB)`
4. En la columna "Status" debe decir **Running**

✅ Si dice Running, MongoDB está activo y listo.

❌ Si no está corriendo, haz clic derecho sobre él → **Start**

---

## 🛠️ Paso 4: Instalar Postman

Postman nos permite probar nuestra API sin necesidad de construir una interfaz visual.

### 4.1 — Descargar e instalar Postman

1. Ve a: **https://www.postman.com/downloads/**
2. Haz clic en el botón **Download the App** (detecta tu OS automáticamente)
3. Ejecuta el instalador descargado
4. La instalación es automática, no necesita configuración

### 4.2 — Crear cuenta en Postman (opcional pero recomendado)

1. Abre Postman
2. Puedes crear una cuenta gratuita con tu correo
3. O hacer clic en **"Skip signing in and take me straight to the app"**

---

## 🛠️ Paso 5: Instalar Docker Desktop

Docker lo necesitamos al final del curso para empaquetar todo.
Lo instalamos ahora para no interrumpir el flujo más adelante.

### 5.1 — Descargar Docker Desktop

1. Ve a: **https://www.docker.com/products/docker-desktop/**
2. Haz clic en **"Download Docker Desktop"** (detecta Windows automáticamente)

### 5.2 — Instalar Docker Desktop en Windows

> ⚠️ Docker Desktop requiere que **WSL 2** (Windows Subsystem for Linux 2) esté habilitado.
> El instalador lo hace automáticamente si no lo tienes.

1. Ejecuta el instalador descargado (`Docker Desktop Installer.exe`)
2. En las opciones, deja todo marcado por defecto y haz clic en **Ok**
3. La instalación tardará varios minutos
4. Cuando termine, haz clic en **Close and restart** para reiniciar tu computadora

### 5.3 — Verificar Docker

Después de reiniciar:
1. Docker Desktop se inicia automáticamente (verás el ícono de Docker en la barra de tareas)
2. Espera a que la ballena deje de moverse (eso indica que está listo)
3. Abre la terminal (Windows + R → cmd → Enter) y escribe:

```
docker --version
```

Deberías ver algo como: `Docker version 25.x.x`

---

## ✅ Verificación final: todo listo

Antes de continuar al siguiente módulo, verifica que todo funciona ejecutando
estos comandos en la terminal (Windows + R → cmd → Enter):

```
java -version
```
Debe mostrar: `openjdk version "17.x.x"`

```
docker --version
```
Debe mostrar: `Docker version 25.x.x`

---

## 🗺️ Resumen de lo que instalaste

| Herramienta | Versión | ¿Para qué? |
|-------------|---------|-----------|
| Java 17 JDK | 17.x.x | Lenguaje de programación |
| IntelliJ IDEA Community | 2024.x | Editor de código |
| MongoDB Community | 7.x | Base de datos |
| MongoDB Compass | Incluido | Ver datos visualmente |
| Postman | Última | Probar las APIs |
| Docker Desktop | 25.x | Empaquetar todo al final |

---

## ➡️ Siguiente paso

En el **[Proyecto P02 — Primer proyecto Spring Boot](proyecto-02-primer-proyecto.md)**
vamos a crear el primer proyecto del Michi Café en Spring Initializr, abrirlo
en IntelliJ y hacer que diga "¡Hola Michi Café!" por primera vez.

> 🐾 "La cocina bien equipada es la mitad del trabajo."
