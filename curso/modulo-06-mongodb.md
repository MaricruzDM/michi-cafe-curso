# Módulo 06 — MongoDB: La Base de Datos del Michi Café
## 🐱 Analogía: El libro de registros del café

---

## 🏠 Introducción

Hasta ahora nuestros datos viven en la memoria del servidor. Cada vez que reiniciamos la aplicación, todos los datos desaparecen. Es como si el Michi Café borrara su lista de clientes y pedidos cada vez que cierra por la noche.

Necesitamos un lugar donde guardar la información de forma **permanente**. Ese lugar es la **base de datos**.

En este módulo aprenderemos **MongoDB**, la base de datos que usaremos en todo el curso.

---

## 💡 ¿Qué es una base de datos?

Una base de datos es un sistema que guarda información de forma organizada y persistente, permitiendo consultarla, modificarla y eliminarla cuando sea necesario.

### Analogía 🐾

El Michi Café tiene un archivero con carpetas:
- Una carpeta para **clientes** (fichas de cada cliente)
- Una carpeta para **bebidas** (fichas del menú)
- Una carpeta para **pedidos** (registros de cada orden)

Ese archivero existe aunque el café esté cerrado. La información no desaparece. Eso es una base de datos.

---

## 💡 ¿Qué es MongoDB?

MongoDB es una base de datos **NoSQL** orientada a **documentos**.

Esto significa que en lugar de guardar datos en tablas con filas y columnas (como Excel), MongoDB guarda datos en **documentos** con formato JSON.

### Bases de datos relacionales vs MongoDB

| Concepto SQL (tradicional) | Concepto MongoDB | Analogía del café |
|---------------------------|-----------------|-------------------|
| Base de datos | Base de datos | El archivero completo |
| Tabla | Colección | Una carpeta del archivero |
| Fila / Registro | Documento | Una ficha dentro de la carpeta |
| Columna | Campo | Un dato en la ficha (nombre, precio...) |
| Primary Key | `_id` | El número único de cada ficha |

### Analogía 🐾

En una base de datos tradicional (SQL), los clientes se guardan así:

```
TABLA CLIENTES
| id | nombre | correo           | puntos |
|----|--------|------------------|--------|
| 1  | Sofía  | sofia@email.com  | 150    |
| 2  | Carlos | carlos@email.com | 80     |
```

En MongoDB, los clientes se guardan así (como documentos JSON):

```json
// Documento 1
{
  "_id": "64a1b2c3d4e5f6a7b8c9d0e1",
  "nombre": "Sofía",
  "correo": "sofia@email.com",
  "puntos": 150,
  "bebidaFavorita": "Latte de vainilla",
  "direcciones": ["Calle Gatitos 123", "Av. Michi 456"]
}

// Documento 2
{
  "_id": "64a1b2c3d4e5f6a7b8c9d0e2",
  "nombre": "Carlos",
  "correo": "carlos@email.com",
  "puntos": 80
  // Carlos no tiene bebida favorita ni direcciones guardadas — ¡y está bien!
}
```

> 💡 En MongoDB cada documento puede tener campos diferentes. No todos los documentos necesitan tener exactamente los mismos campos. Esto se llama **esquema flexible**.

---

## 💡 ¿Por qué MongoDB para microservicios?

MongoDB es muy popular en arquitecturas de microservicios por varias razones:

1. **Esquema flexible**: cada microservicio puede guardar exactamente lo que necesita
2. **Escala fácilmente**: puede manejar millones de documentos
3. **JSON nativo**: los datos ya están en el formato que usan las APIs REST
4. **Rápido para leer**: optimizado para consultas frecuentes

### Analogía 🐾

En el Michi Café con múltiples sucursales:
- La sucursal de pedidos guarda sus fichas de una forma
- La sucursal de clientes guarda sus fichas de otra forma
- La sucursal de inventario guarda sus fichas de otra forma

MongoDB permite que cada área tenga su propio archivero con su propio formato, sin que una afecte a la otra.

---

## 🛠️ Instalación de MongoDB

### Opción 1: MongoDB local (en tu computadora)

1. Ve a: **https://www.mongodb.com/try/download/community**
2. Selecciona tu sistema operativo
3. Descarga e instala
4. MongoDB corre en el puerto **27017** por defecto

Para verificar que está corriendo, abre una terminal y escribe:
```bash
mongosh
```

Si ves el prompt `test>`, MongoDB está funcionando.

### Opción 2: MongoDB Atlas (en la nube — recomendado para aprender)

MongoDB Atlas es la versión en la nube de MongoDB. Tiene un plan gratuito perfecto para aprender.

1. Ve a: **https://www.mongodb.com/atlas**
2. Crea una cuenta gratuita
3. Crea un **Cluster** gratuito (M0 Free Tier)
4. Crea un usuario de base de datos
5. Obtén tu **connection string** (cadena de conexión)

La cadena de conexión se ve así:
```
mongodb+srv://usuario:contraseña@cluster0.xxxxx.mongodb.net/michicafe
```

---

## 💡 MongoDB Compass: la interfaz visual

**MongoDB Compass** es una aplicación de escritorio que te permite ver y manipular tus datos de forma visual, sin escribir comandos.

- Descarga: **https://www.mongodb.com/products/compass**

### Analogía 🐾

Si MongoDB es el archivero, Compass es la persona que te ayuda a abrir las carpetas, buscar fichas y organizarlas visualmente. Muy útil para aprender y depurar.

---

## 💡 Conceptos clave de MongoDB

### Documentos

Un documento es la unidad básica de datos en MongoDB. Es un objeto JSON con pares clave-valor.

```json
{
  "_id": "64a1b2c3d4e5f6a7b8c9d0e1",
  "nombre": "Latte de vainilla",
  "precio": 45.50,
  "categoria": "caliente",
  "disponible": true,
  "ingredientes": ["espresso", "leche", "vainilla"],
  "informacionNutricional": {
    "calorias": 250,
    "proteinas": 8
  }
}
```

> 💡 El campo `_id` es generado automáticamente por MongoDB. Es el identificador único de cada documento, como el número de folio de una ficha.

### Colecciones

Una colección es un grupo de documentos relacionados. Es el equivalente a una tabla en SQL.

```
Base de datos: michicafe
├── Colección: bebidas      (todos los documentos de bebidas)
├── Colección: clientes     (todos los documentos de clientes)
└── Colección: pedidos      (todos los documentos de pedidos)
```

### El `_id` de MongoDB

MongoDB genera automáticamente un `_id` de tipo `ObjectId` para cada documento. Se ve así:

```
64a1b2c3d4e5f6a7b8c9d0e1
```

Es una cadena de 24 caracteres hexadecimales que garantiza ser única en todo el mundo.

---

## 💡 Operaciones básicas en MongoDB

Antes de conectarlo con Spring Boot, veamos cómo funciona MongoDB directamente usando `mongosh` (la terminal de MongoDB).

### Seleccionar/crear una base de datos

```javascript
use michicafe
// Si no existe, MongoDB la crea automáticamente al insertar el primer documento
```

### Insertar documentos

```javascript
// Insertar una bebida
db.bebidas.insertOne({
  nombre: "Espresso",
  precio: 35.00,
  categoria: "caliente",
  disponible: true
})

// Insertar varios documentos a la vez
db.bebidas.insertMany([
  { nombre: "Latte de vainilla", precio: 45.50, categoria: "caliente", disponible: true },
  { nombre: "Cold Brew",         precio: 50.00, categoria: "frio",     disponible: true },
  { nombre: "Matcha Latte",      precio: 55.00, categoria: "caliente", disponible: true }
])
```

### Consultar documentos

```javascript
// Obtener todos los documentos de la colección
db.bebidas.find()

// Obtener con formato legible
db.bebidas.find().pretty()

// Filtrar: solo bebidas calientes
db.bebidas.find({ categoria: "caliente" })

// Filtrar: bebidas con precio menor a 50
db.bebidas.find({ precio: { $lt: 50 } })

// Filtrar: bebidas disponibles y calientes
db.bebidas.find({ disponible: true, categoria: "caliente" })

// Obtener solo un documento
db.bebidas.findOne({ nombre: "Espresso" })
```

### Operadores de consulta

| Operador | Significado | Ejemplo |
|----------|-------------|---------|
| `$lt` | Menor que | `{ precio: { $lt: 50 } }` |
| `$lte` | Menor o igual | `{ precio: { $lte: 50 } }` |
| `$gt` | Mayor que | `{ precio: { $gt: 40 } }` |
| `$gte` | Mayor o igual | `{ precio: { $gte: 40 } }` |
| `$ne` | Diferente de | `{ categoria: { $ne: "frio" } }` |
| `$in` | Está en la lista | `{ categoria: { $in: ["caliente", "frio"] } }` |

### Actualizar documentos

```javascript
// Actualizar un campo de un documento
db.bebidas.updateOne(
  { nombre: "Espresso" },          // filtro: ¿cuál actualizar?
  { $set: { precio: 38.00 } }      // cambio: qué actualizar
)

// Actualizar varios documentos
db.bebidas.updateMany(
  { categoria: "caliente" },
  { $set: { disponible: true } }
)
```

### Eliminar documentos

```javascript
// Eliminar un documento
db.bebidas.deleteOne({ nombre: "Chai Latte" })

// Eliminar varios documentos
db.bebidas.deleteMany({ disponible: false })

// Eliminar todos los documentos de una colección
db.bebidas.deleteMany({})
```

---

## 💡 Agregaciones: consultas avanzadas

Las agregaciones permiten hacer operaciones más complejas, como calcular totales, agrupar datos o hacer estadísticas.

### Analogía 🐾

El dueño del Michi Café quiere saber:
- ¿Cuántas bebidas hay por categoría?
- ¿Cuál es el precio promedio del menú?
- ¿Cuánto se vendió en total hoy?

Para eso usamos agregaciones.

```javascript
// ¿Cuántas bebidas hay por categoría?
db.bebidas.aggregate([
  { $group: {
      _id: "$categoria",
      total: { $sum: 1 },
      precioPromedio: { $avg: "$precio" }
  }}
])

// Resultado:
// { _id: "caliente", total: 4, precioPromedio: 46.375 }
// { _id: "frio",     total: 1, precioPromedio: 50.0   }
```

---

## 💡 Índices: el índice del archivero

Un **índice** en MongoDB es como el índice de un libro: permite encontrar documentos rápidamente sin tener que revisar todos uno por uno.

### Analogía 🐾

Si el archivero tiene 10,000 fichas de clientes y quieres encontrar a "Sofía", sin índice tendrías que revisar las 10,000 fichas. Con un índice por nombre, vas directo a la "S" y encuentras a Sofía en segundos.

```javascript
// Crear un índice en el campo "nombre"
db.clientes.createIndex({ nombre: 1 })  // 1 = ascendente, -1 = descendente

// Crear un índice único (no permite duplicados)
db.clientes.createIndex({ correo: 1 }, { unique: true })

// Ver los índices de una colección
db.clientes.getIndexes()
```

---

## 💡 Estructura de datos del Michi Café en MongoDB

Así se verán nuestros datos cuando los guardemos en MongoDB:

### Colección `bebidas`

```json
{
  "_id": "64a1b2c3d4e5f6a7b8c9d0e1",
  "nombre": "Latte de vainilla",
  "precio": 45.50,
  "categoria": "caliente",
  "descripcion": "Espresso con leche vaporizada y vainilla",
  "disponible": true
}
```

### Colección `clientes`

```json
{
  "_id": "64a1b2c3d4e5f6a7b8c9d0e2",
  "nombre": "Sofía",
  "correo": "sofia@email.com",
  "puntosFidelidad": 150,
  "fechaRegistro": "2024-01-15T10:00:00Z"
}
```

### Colección `pedidos`

```json
{
  "_id": "64a1b2c3d4e5f6a7b8c9d0e3",
  "clienteId": "64a1b2c3d4e5f6a7b8c9d0e2",
  "bebidas": [
    { "bebidaId": "64a1b2c3d4e5f6a7b8c9d0e1", "nombre": "Latte de vainilla", "precio": 45.50 },
    { "bebidaId": "64a1b2c3d4e5f6a7b8c9d0e4", "nombre": "Matcha Latte",      "precio": 55.00 }
  ],
  "total": 100.50,
  "estado": "entregado",
  "fecha": "2024-01-15T11:30:00Z"
}
```

---

## 🧪 Ejercicios del Módulo 6

### Ejercicio 1: Explorar con mongosh

Abre `mongosh` y realiza las siguientes operaciones:
1. Crea la base de datos `michicafe`
2. Inserta 5 bebidas en la colección `bebidas`
3. Consulta todas las bebidas disponibles
4. Consulta las bebidas con precio entre $40 y $55
5. Actualiza el precio del Espresso a $38
6. Elimina las bebidas no disponibles

### Ejercicio 2: Explorar con Compass

Instala MongoDB Compass y:
1. Conéctate a tu base de datos local
2. Visualiza la colección `bebidas`
3. Usa el filtro visual para buscar bebidas calientes
4. Edita un documento directamente desde la interfaz

### Ejercicio 3: Diseño de documentos

Diseña (en papel o en un archivo) cómo guardarías estos datos en MongoDB:
- Un pedido que tiene múltiples bebidas
- Un cliente con múltiples direcciones de entrega
- Una bebida con sus ingredientes y valores nutricionales

Piensa: ¿qué campos necesita cada documento? ¿Qué tipo de dato tiene cada campo?

---

## ✅ Resumen del Módulo 6

| Concepto | ¿Qué es? | Analogía del Michi Café |
|----------|----------|------------------------|
| Base de datos | Sistema de almacenamiento persistente | El archivero del café |
| MongoDB | Base de datos NoSQL orientada a documentos | Archivero con fichas flexibles |
| Documento | Unidad básica de datos (JSON) | Una ficha del archivero |
| Colección | Grupo de documentos relacionados | Una carpeta del archivero |
| `_id` | Identificador único del documento | El número de folio de la ficha |
| `find()` | Consultar documentos | Buscar fichas en el archivero |
| `insertOne()` | Insertar un documento | Agregar una ficha nueva |
| `updateOne()` | Actualizar un documento | Corregir datos en una ficha |
| `deleteOne()` | Eliminar un documento | Sacar una ficha del archivero |
| Índice | Estructura para búsquedas rápidas | El índice alfabético del archivero |

---

## ➡️ Siguiente Módulo

En el **Módulo 07** conectaremos Spring Boot con MongoDB usando **Spring Data MongoDB**. Vamos a reemplazar nuestro `HashMap` de prueba por una base de datos real. El Michi Café tendrá memoria permanente.

> 🐾 "Una base de datos bien diseñada es como un archivero bien organizado: encontrar cualquier cosa es cuestión de segundos."
