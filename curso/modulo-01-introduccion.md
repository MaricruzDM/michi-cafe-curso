# Módulo 01 — Introducción: ¿Qué es programar?
## 🐱 Conociendo el Michi Café

---

## 🏠 Bienvenida

Imagina que vas a abrir una cafetería temática de gatitos llamada **Michi Café**.

En este café:
- Los clientes llegan, ven el menú y piden su bebida favorita
- Los meseros toman los pedidos y los llevan a la cocina
- La cocina prepara cada orden
- La caja registra los pagos y acumula puntos de fidelidad
- El dueño puede ver reportes de cuánto se vendió

Ahora imagina que quieres que **todo eso funcione de forma automática**, sin que tengas que estar presente todo el tiempo. Quieres que una computadora lo haga por ti.

**Eso es exactamente lo que hace un programa de computadora.**

---

## 💡 ¿Qué es programar?

Programar es darle instrucciones a una computadora para que haga algo.

Pero las computadoras no entienden español ni inglés como nosotros.
Necesitan instrucciones muy precisas, escritas en un **lenguaje de programación**.

### Analogía 🐾

Imagina que contratas a un **gatito robot** para ser mesero.
Este gatito es muy obediente pero necesita instrucciones **exactas**. No puede adivinar nada.

Si le dices:
> "Atiende al cliente"

El gatito robot no sabe qué hacer. ¿Saludarlo? ¿Traerle agua? ¿Preguntarle qué quiere?

Pero si le dices:
> 1. Camina hacia la mesa número 3
> 2. Di "Bienvenido al Michi Café, ¿qué desea ordenar?"
> 3. Escucha la respuesta
> 4. Escribe el pedido en tu libreta
> 5. Lleva la libreta a la cocina

¡Ahora sí puede hacerlo! Eso es un **programa**: una lista de instrucciones precisas y ordenadas.

---

## 💡 ¿Qué es Java?

Java es el lenguaje de programación que usaremos en este curso.

Es como el idioma en el que le daremos instrucciones a nuestra computadora.

### ¿Por qué Java?

Java es como el **idioma oficial de las grandes empresas**:

- Lo usan bancos, tiendas en línea, aplicaciones empresariales, Netflix, Uber, Amazon
- Lleva más de 25 años en uso y sigue siendo uno de los más populares del mundo
- Es muy seguro y estable: difícil de romper accidentalmente
- Funciona en Windows, Mac y Linux sin cambios

En el Michi Café, Java sería el **manual de operaciones oficial** que todos los
empleados siguen. Está bien escrito, es claro y funciona en cualquier sucursal.

---

## 💡 ¿Qué es Spring Boot?

Spring Boot es una herramienta que hace más fácil trabajar con Java para crear
aplicaciones que respondan a peticiones de internet (servidores web).

### Analogía 🐾

- **Java solo** es como construir el café desde los cimientos: poner ladrillos,
  instalar tuberías, cablear electricidad. Puedes hacerlo, pero tarda mucho.

- **Spring Boot** es como comprar una **franquicia del Michi Café**: ya viene con
  el diseño, los equipos, el manual de operaciones y los uniformes. Tú solo pones
  tu menú y abres las puertas.

Spring Boot se encarga de todas las partes complicadas y repetitivas para que tú
te concentres en la lógica de tu negocio.

### Versión que usamos

En este curso usamos **Spring Boot 4.1.0** con **Java 17**.

> 💡 Spring Boot 4.1.0 requiere Java 17 como mínimo. Nosotros usamos Java 17
> porque es la versión LTS (Long Term Support) más reciente, la que más empresas
> están adoptando actualmente.

---

## 💡 ¿Qué es MongoDB?

MongoDB es una **base de datos**. Una base de datos es donde guardamos información
de forma organizada y permanente para poder consultarla después.

### Analogía 🐾

En el Michi Café, la base de datos es el **libro de registros** donde se anota:
- Qué clientes han venido y cuántos puntos tienen
- Qué pedidos se han hecho y cuál es su estado
- Qué bebidas hay en el menú y cuáles están disponibles

Sin ese libro, cada vez que un cliente regresa tendrías que preguntarle todo de nuevo.
Con el libro puedes decirle:
> "¡Hola Sofía! ¿Quieres tu Latte de vainilla de siempre?"

MongoDB guarda la información en un formato llamado **documentos** que se parecen
mucho a las fichas de un archivero. Cada ficha tiene campos como nombre, pedido, fecha, etc.

---

## 💡 ¿Qué son los Microservicios?

Esta es una de las partes más importantes del curso.

### Primero: ¿qué es una aplicación monolítica?

Imagina que el Michi Café tiene **un solo empleado** que hace absolutamente todo:
- Toma el pedido
- Prepara el café
- Cobra en caja
- Hace el inventario
- Atiende el teléfono
- Limpia las mesas

¿Qué pasa si ese empleado se enferma? **Todo el café se detiene.**

Eso es una aplicación **monolítica**: todo el código en un solo programa.
Si una parte falla, todo falla.

### Ahora: ¿qué son los microservicios?

La solución es **dividir el trabajo en áreas independientes**:
- El área de pedidos solo toma y gestiona pedidos
- El área de cocina solo prepara bebidas
- El área de caja solo cobra y gestiona puntos
- El área de inventario solo controla el stock

Si la caja tiene un problema, la cocina sigue trabajando.
Si la cocina está saturada, puedes agregar más cocineros sin tocar la caja.

**Eso son los microservicios**: dividir una aplicación grande en partes pequeñas
e independientes, donde cada parte hace una sola cosa y la hace muy bien.

### ¿Cómo se conecta todo?

```
[Aplicación del cliente / Navegador / App móvil]
                    |
                    ▼
            [API Gateway]         ← La puerta principal del Michi Café
                    |
        ____________|____________
       |             |           |
       ▼             ▼           ▼
  [Bebidas]     [Clientes]   [Pedidos]   ← Microservicios (áreas del café)
       |             |           |
       ▼             ▼           ▼
  [MongoDB]     [MongoDB]   [MongoDB]    ← Cada área tiene su propio libro
```

Cada microservicio:
- Tiene su propia base de datos independiente
- Funciona aunque los otros fallen
- Puede actualizarse sin afectar a los demás
- Puede escalar de forma independiente (si hay muchos pedidos, solo agrandamos ese servicio)

---

## 💡 ¿Qué es IntelliJ IDEA?

IntelliJ IDEA es el programa donde vamos a escribir nuestro código.
Se llama **IDE** (Integrated Development Environment — Entorno de Desarrollo Integrado).

Piénsalo como el **cuaderno de recetas** del Michi Café, pero inteligente:
- Te avisa cuando cometes un error antes de que lo ejecutes
- Te sugiere cómo completar lo que estás escribiendo
- Te ayuda a organizar todos los archivos del proyecto
- Tiene colores para que el código sea más fácil de leer

Usamos la versión **Community**, que es completamente **gratuita**.

---

## 💡 ¿Qué es Postman?

Postman es una herramienta que nos permite **probar nuestras APIs** sin necesidad
de construir una interfaz visual (pantalla de usuario).

### Analogía 🐾

Es como el **inspector de calidad** del Michi Café. Antes de abrir al público,
el inspector llega, hace pedidos de prueba y verifica que todo funciona correctamente.

Con Postman puedes enviar peticiones a tu servidor y ver exactamente qué responde,
sin necesidad de tener una aplicación web o móvil construida.

---

## 💡 ¿Qué es Docker?

Docker es una herramienta que empaqueta tu aplicación junto con todo lo que necesita
para funcionar, en una unidad llamada **contenedor**.

### Analogía 🐾

Es como meter **toda la sucursal del Michi Café en una caja de transporte**:
el local armado, los equipos instalados, la configuración lista.
Llegas a cualquier servidor, abres la caja, y el café está listo para operar.

---

## 💡 El panorama completo: ¿qué vamos a construir?

Al final del curso tendrás corriendo este sistema:

| Componente | Tecnología | Puerto | Función |
|------------|-----------|--------|---------|
| API Gateway | Spring Cloud Gateway | 8080 | Puerta de entrada única |
| Servicio Bebidas | Spring Boot 4.1.0 + MongoDB | 8081 | Gestiona el menú |
| Servicio Clientes | Spring Boot 4.1.0 + MongoDB | 8082 | Gestiona clientes y puntos |
| Servicio Pedidos | Spring Boot 4.1.0 + MongoDB | 8083 | Gestiona pedidos |
| Base de datos | MongoDB 7 | 27017 | Almacena toda la información |
| Contenedores | Docker + Docker Compose | — | Empaqueta todo |

---

## 🧪 Ejercicio de comprensión — Módulo 01

Antes de escribir una sola línea de código, hagamos un ejercicio mental.
**Piensa en el Michi Café y responde:**

1. ¿Qué información necesitarías guardar sobre un cliente?
   - Ejemplo: nombre, correo, bebida favorita, puntos acumulados...

2. ¿Qué pasos sigue un pedido desde que el cliente lo hace hasta que lo recibe?
   - Ejemplo: cliente pide → mesero anota → cocina prepara → mesero entrega → caja cobra

3. ¿Qué áreas del café podrían funcionar de forma independiente si otra falla?

Estas respuestas te ayudarán a entender mejor la arquitectura que vamos a construir.

---

## ✅ Resumen del Módulo 01

| Concepto | ¿Qué es? | En el Michi Café |
|----------|----------|-----------------|
| Programar | Dar instrucciones precisas a una computadora | El manual del gatito robot |
| Java | Lenguaje de programación empresarial | El idioma oficial del café |
| Spring Boot 4.1.0 | Framework que facilita crear servidores con Java | La franquicia lista para abrir |
| MongoDB | Base de datos orientada a documentos | El libro de registros del café |
| Microservicios | App dividida en partes independientes | Las áreas especializadas del café |
| IntelliJ IDEA | Editor de código inteligente | El cuaderno de recetas inteligente |
| Postman | Herramienta para probar APIs | El inspector de calidad |
| Docker | Empaqueta la app con todo lo que necesita | La caja de transporte de la sucursal |

---

## ➡️ Siguiente módulo

En el **Módulo 02** aprenderemos Java desde cero: variables, tipos de datos,
condiciones y ciclos. Todo con ejemplos del Michi Café.

> 🐾 "El viaje de mil líneas de código comienza con entender para qué sirve cada una."
