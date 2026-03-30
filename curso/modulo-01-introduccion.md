# Módulo 01 — Introducción: ¿Qué es programar?
## 🐱 Analogía: Conociendo el Michi Café

---

## 🏠 Bienvenida al Michi Café

Imagina que vas a abrir una cafetería temática de gatitos llamada **Michi Café**.

En este café:
- Los clientes llegan, piden su café y sus pastelitos
- Los meseros toman los pedidos
- La cocina prepara todo
- La caja registra los pagos
- El dueño puede ver reportes de cuánto se vendió

Ahora imagina que quieres que **todo eso funcione de forma automática**, sin que tengas que estar presente todo el tiempo. Quieres que una computadora lo haga por ti.

**Eso es exactamente lo que hace un programa de computadora.**

---

## 💡 ¿Qué es programar?

Programar es darle instrucciones a una computadora para que haga algo.

Pero las computadoras no entienden español ni inglés como nosotros. Necesitan instrucciones muy precisas, escritas en un **lenguaje de programación**.

### Analogía del Michi Café 🐾

Imagina que contratas a un gatito robot para ser mesero. Este gatito robot es muy obediente, pero necesita instrucciones **exactas**. No puede adivinar nada.

Si le dices:
> "Atiende al cliente"

El gatito robot no sabe qué hacer. ¿Saludarlo? ¿Traerle agua? ¿Preguntarle qué quiere?

Pero si le dices:
> 1. Camina hacia la mesa
> 2. Di "Bienvenido al Michi Café, ¿qué desea ordenar?"
> 3. Escucha la respuesta
> 4. Escribe el pedido en tu libreta
> 5. Lleva el pedido a la cocina

¡Ahora sí puede hacerlo! Eso es un **programa**: una lista de instrucciones precisas y ordenadas.

---

## 💡 ¿Qué es un lenguaje de programación?

Es el idioma que usamos para escribir esas instrucciones. Hay muchos lenguajes, como:

- **Java** (el que usaremos en este curso)
- Python
- JavaScript
- C++

Cada uno tiene su sintaxis (su forma de escribirse), pero todos sirven para lo mismo: decirle a la computadora qué hacer.

### ¿Por qué Java?

Java es como el idioma oficial de las grandes empresas. Es:

- **Muy usado**: bancos, tiendas en línea, aplicaciones empresariales
- **Estable**: lleva más de 25 años en uso
- **Seguro**: tiene muchas herramientas para evitar errores
- **Multiplataforma**: funciona en Windows, Mac, Linux sin cambios

En el Michi Café, Java sería como el **manual de operaciones oficial** que todos los empleados deben seguir. Está bien escrito, es claro y funciona en cualquier sucursal.

---

## 💡 ¿Qué es Spring Boot?

Spring Boot es una herramienta que hace más fácil trabajar con Java.

Imagina que Java es el idioma, pero Spring Boot es como una **plantilla de manual ya preparada**. En lugar de escribir el manual desde cero, Spring Boot ya tiene las partes más comunes listas, y tú solo agregas lo que es específico de tu negocio.

En el Michi Café, Spring Boot sería como tener ya impreso el formato del menú, las hojas de pedidos y los uniformes. Tú solo llenas los espacios con tus productos y precios.

---

## 💡 ¿Qué es MongoDB?

MongoDB es una **base de datos**. Una base de datos es donde guardamos información de forma organizada para poder consultarla después.

### Analogía 🐾

En el Michi Café, la base de datos es el **libro de registros** donde se anota:
- Qué clientes han venido
- Qué pedidos se han hecho
- Qué productos hay en el menú
- Cuánto se ha vendido

Sin ese libro, cada vez que un cliente regresa tendrías que preguntarle todo de nuevo. Con el libro, puedes decirle:
> "¡Hola Sofía! ¿Quieres tu latte de vainilla de siempre?"

MongoDB guarda esa información en un formato llamado **documentos**, que se parecen mucho a las fichas de un archivero. Cada ficha tiene campos como nombre, pedido, fecha, etc.

---

## 💡 ¿Qué son los Microservicios?

Esta es una de las partes más importantes del curso.

Imagina que el Michi Café crece mucho. Ahora tiene:
- 5 sucursales
- Servicio a domicilio
- Tienda en línea
- App móvil

Si todo el negocio dependiera de **un solo empleado** que hace todo (toma pedidos, cocina, cobra, hace reportes), sería un caos. Si ese empleado se enferma, todo se detiene.

La solución es **dividir el trabajo en áreas independientes**:
- El área de pedidos
- El área de cocina
- El área de caja
- El área de inventario

Cada área funciona de forma independiente. Si la caja tiene un problema, la cocina sigue trabajando.

**Eso son los microservicios**: dividir una aplicación grande en partes pequeñas e independientes, donde cada parte hace una sola cosa y la hace bien.

---

## 💡 ¿Cómo se conecta todo?

Aquí está el panorama completo de lo que vamos a construir:

```
[App del cliente / Navegador]
          |
          v
    [API Gateway]          ← La puerta principal del Michi Café
          |
    ______|______
   |      |      |
   v      v      v
[Clientes] [Pedidos] [Productos]   ← Microservicios (áreas del café)
   |          |          |
   v          v          v
[MongoDB] [MongoDB] [MongoDB]      ← Bases de datos de cada área
```

Cada microservicio:
- Tiene su propia base de datos
- Funciona de forma independiente
- Se comunica con los demás cuando es necesario

---

## 🛠️ ¿Qué necesitas instalar?

Antes de empezar a programar, necesitas preparar tu computadora. Esto es como preparar la cocina antes de abrir el café.

### 1. Java Development Kit (JDK)
Es el conjunto de herramientas para programar en Java.

- Descarga: https://adoptium.net/
- Versión recomendada: **Java 17** o superior
- Elige tu sistema operativo (Windows, Mac, Linux)

### 2. IntelliJ IDEA (Editor de código)
Es el programa donde vas a escribir tu código. Es como el cuaderno donde escribes las recetas.

- Descarga: https://www.jetbrains.com/idea/download/
- Elige la versión **Community** (es gratis)

### 3. MongoDB
La base de datos donde guardaremos la información.

- Descarga: https://www.mongodb.com/try/download/community
- También puedes usar **MongoDB Atlas** (versión en la nube, gratis para aprender)

### 4. Postman
Una herramienta para probar nuestras APIs (lo explicaremos más adelante).

- Descarga: https://www.postman.com/downloads/

### 5. Maven
Una herramienta que gestiona las dependencias de Java (como un asistente que consigue los ingredientes que necesitas).

- Generalmente viene incluido con IntelliJ IDEA

---

## ✅ Verificando la instalación

Una vez instalado Java, abre una terminal (en Windows: busca "cmd" o "PowerShell") y escribe:

```bash
java -version
```

Deberías ver algo como:
```
openjdk version "17.0.x" ...
```

Si ves eso, ¡Java está listo!

---

## 🧪 Ejercicio del Módulo 1

Antes de escribir código, hagamos un ejercicio mental:

**Piensa en el Michi Café y responde:**

1. ¿Qué información necesitarías guardar sobre un cliente?
   - Ejemplo: nombre, correo, bebida favorita...

2. ¿Qué pasos sigue un pedido desde que el cliente lo hace hasta que lo recibe?
   - Ejemplo: cliente pide → mesero anota → cocina prepara → mesero entrega

3. ¿Qué áreas del café podrían funcionar de forma independiente?
   - Ejemplo: caja, cocina, inventario...

Estas respuestas te ayudarán a entender mejor la arquitectura que vamos a construir.

---

## ✅ Resumen del Módulo 1

| Concepto | ¿Qué es? | En el Michi Café |
|----------|----------|-----------------|
| Programar | Dar instrucciones precisas a una computadora | El manual del gatito robot |
| Java | Lenguaje de programación empresarial | El idioma oficial del café |
| Spring Boot | Herramienta que facilita Java | La plantilla del manual ya preparada |
| MongoDB | Base de datos | El libro de registros del café |
| Microservicios | Dividir la app en partes independientes | Las áreas del café |

---

## ➡️ Siguiente Módulo

En el **Módulo 02** aprenderemos Java desde cero: variables, tipos de datos, condiciones y ciclos. Todo con ejemplos del Michi Café.

> 🐾 "El viaje de mil líneas de código comienza con un solo `System.out.println`"
