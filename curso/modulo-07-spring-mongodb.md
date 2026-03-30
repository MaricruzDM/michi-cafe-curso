# Módulo 07 — Conectando Spring Boot con MongoDB
## 🐱 Analogía: El mesero anota en el libro de registros

---

## 🏠 Introducción

En el módulo anterior aprendimos cómo funciona MongoDB por dentro.
En el módulo 05 teníamos una API REST que guardaba datos en memoria (un `HashMap`).

Ahora vamos a unir ambos mundos: **Spring Boot hablará con MongoDB** para guardar y consultar datos de forma permanente.

### Analogía 🐾

Antes, el gatito mesero anotaba los pedidos en un papel que tiraba al final del día.
Ahora va a anotarlos en el **libro de registros oficial** del Michi Café, que se guarda en la caja fuerte y nunca se pierde.

---

## 💡 Spring Data MongoDB

**Spring Data MongoDB** es una librería que hace muy fácil conectar Spring Boot con MongoDB.

En lugar de escribir consultas complejas, Spring Data nos permite:
- Definir una interfaz con métodos con nombres descriptivos
- Spring genera el código de la consulta automáticamente

### Analogía 🐾

Es como tener un asistente muy inteligente en el archivero. En lugar de decirle:
> "Abre la carpeta de bebidas, busca todas las fichas donde el campo categoría diga caliente, y tráemelas ordenadas por precio"

Solo le dices:
> `findByCategoriaOrderByPrecioAsc("caliente")`

Y él sabe exactamente qué hacer.

---

## 🛠️ Paso 1: Agregar la dependencia

Abre el archivo `pom.xml` y agrega la dependencia de Spring Data MongoDB:

```xml
<!-- pom.xml -->
<dependencies>

    <!-- Spring Web (ya lo teníamos) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Spring Data MongoDB (nuevo) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-mongodb</artifactId>
    </dependency>

</dependencies>
```

Después de agregar la dependencia, Maven descargará automáticamente las librerías necesarias.

---

## 🛠️ Paso 2: Configurar la conexión

Abre el archivo `application.properties` y agrega la configuración de MongoDB:

```properties
# src/main/resources/application.properties

spring.application.name=michi-cafe
server.port=8080

# ── Conexión a MongoDB ──────────────────────────────────────────────────────

# Opción A: MongoDB local
spring.data.mongodb.host=localhost
spring.data.mongodb.port=27017
spring.data.mongodb.database=michicafe

# Opción B: MongoDB Atlas (nube) — comenta la opción A y descomenta esta
# spring.data.mongodb.uri=mongodb+srv://usuario:contraseña@cluster0.xxxxx.mongodb.net/michicafe
```

> 💡 Reemplaza `usuario`, `contraseña` y `cluster0.xxxxx` con los datos de tu cuenta de Atlas si usas la nube.

---

## 🛠️ Paso 3: Anotar el modelo con `@Document`

Necesitamos decirle a Spring Data que nuestra clase `Bebida` corresponde a una colección en MongoDB.

```java
// src/main/java/com/michicafe/model/Bebida.java
package com.michicafe.model;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document(collection = "bebidas")  // Esta clase se guarda en la colección "bebidas"
public class Bebida {

    @Id  // Este campo es el _id de MongoDB
    private String id;

    private String nombre;
    private double precio;
    private String categoria;
    private String descripcion;
    private boolean disponible;

    // Constructor vacío (requerido por Spring Data)
    public Bebida() {}

    public Bebida(String nombre, double precio,
                  String categoria, String descripcion, boolean disponible) {
        this.nombre = nombre;
        this.precio = precio;
        this.categoria = categoria;
        this.descripcion = descripcion;
        this.disponible = disponible;
    }

    // Getters
    public String getId()          { return id; }
    public String getNombre()      { return nombre; }
    public double getPrecio()      { return precio; }
    public String getCategoria()   { return categoria; }
    public String getDescripcion() { return descripcion; }
    public boolean isDisponible()  { return disponible; }

    // Setters
    public void setId(String id)               { this.id = id; }
    public void setNombre(String nombre)       { this.nombre = nombre; }
    public void setPrecio(double precio)       { this.precio = precio; }
    public void setCategoria(String categoria) { this.categoria = categoria; }
    public void setDescripcion(String desc)    { this.descripcion = desc; }
    public void setDisponible(boolean disp)    { this.disponible = disp; }
}
```

### ¿Qué hacen las nuevas anotaciones?

| Anotación | ¿Qué hace? | Analogía |
|-----------|-----------|----------|
| `@Document(collection = "bebidas")` | Indica en qué colección de MongoDB se guarda | "Esta ficha va en la carpeta de bebidas" |
| `@Id` | Marca el campo que será el `_id` de MongoDB | El número de folio de la ficha |

---

## 🛠️ Paso 4: Crear el Repository

El **Repository** es la interfaz que habla directamente con MongoDB. Es la capa más cercana a la base de datos.

```java
// src/main/java/com/michicafe/repository/BebidaRepository.java
package com.michicafe.repository;

import com.michicafe.model.Bebida;
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public interface BebidaRepository extends MongoRepository<Bebida, String> {
    //                                                      ↑       ↑
    //                                               tipo de    tipo del
    //                                               documento   _id

    // Spring Data genera el código de estas consultas automáticamente
    // solo con leer el nombre del método:

    // Buscar por categoría
    List<Bebida> findByCategoria(String categoria);

    // Buscar solo las disponibles
    List<Bebida> findByDisponibleTrue();

    // Buscar por categoría y disponibilidad
    List<Bebida> findByCategoriaAndDisponibleTrue(String categoria);

    // Buscar por precio menor o igual a un valor
    List<Bebida> findByPrecioLessThanEqual(double precioMax);

    // Buscar por precio entre dos valores
    List<Bebida> findByPrecioBetween(double min, double max);

    // Buscar por nombre que contenga un texto (sin importar mayúsculas)
    List<Bebida> findByNombreContainingIgnoreCase(String texto);
}
```

### La magia de los nombres de métodos

Spring Data lee el nombre del método y genera la consulta MongoDB automáticamente:

| Nombre del método | Consulta que genera |
|-------------------|---------------------|
| `findByCategoria(cat)` | `{ categoria: cat }` |
| `findByDisponibleTrue()` | `{ disponible: true }` |
| `findByPrecioLessThanEqual(max)` | `{ precio: { $lte: max } }` |
| `findByPrecioBetween(min, max)` | `{ precio: { $gte: min, $lte: max } }` |
| `findByNombreContainingIgnoreCase(txt)` | `{ nombre: /txt/i }` |

### Métodos que ya vienen gratis con `MongoRepository`

Al extender `MongoRepository`, ya tienes estos métodos sin escribir nada:

| Método | ¿Qué hace? |
|--------|-----------|
| `save(documento)` | Guarda o actualiza un documento |
| `findById(id)` | Busca por `_id`, devuelve `Optional` |
| `findAll()` | Devuelve todos los documentos |
| `deleteById(id)` | Elimina por `_id` |
| `existsById(id)` | Verifica si existe |
| `count()` | Cuenta todos los documentos |

---

## 🛠️ Paso 5: Actualizar el Service

Ahora el `BebidaService` usará el `BebidaRepository` en lugar del `HashMap`:

```java
// src/main/java/com/michicafe/service/BebidaService.java
package com.michicafe.service;

import com.michicafe.model.Bebida;
import com.michicafe.repository.BebidaRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

@Service
public class BebidaService {

    @Autowired
    private BebidaRepository bebidaRepository;
    // Spring inyecta automáticamente el repository
    // Ya no necesitamos el HashMap ni el contadorId

    // CREATE / UPDATE
    public Bebida guardar(Bebida bebida) {
        return bebidaRepository.save(bebida);
        // Si tiene _id → actualiza. Si no tiene _id → crea nuevo.
    }

    // READ: todas
    public List<Bebida> obtenerTodas() {
        return bebidaRepository.findAll();
    }

    // READ: por id
    public Optional<Bebida> obtenerPorId(String id) {
        return bebidaRepository.findById(id);
    }

    // READ: por categoría
    public List<Bebida> obtenerPorCategoria(String categoria) {
        return bebidaRepository.findByCategoria(categoria);
    }

    // READ: solo disponibles
    public List<Bebida> obtenerDisponibles() {
        return bebidaRepository.findByDisponibleTrue();
    }

    // READ: por rango de precio
    public List<Bebida> obtenerPorRangoPrecio(double min, double max) {
        return bebidaRepository.findByPrecioBetween(min, max);
    }

    // READ: buscar por nombre
    public List<Bebida> buscarPorNombre(String texto) {
        return bebidaRepository.findByNombreContainingIgnoreCase(texto);
    }

    // UPDATE: reemplazar completo
    public Optional<Bebida> actualizar(String id, Bebida bebidaActualizada) {
        if (!bebidaRepository.existsById(id)) {
            return Optional.empty();
        }
        bebidaActualizada.setId(id); // Mantenemos el mismo id
        return Optional.of(bebidaRepository.save(bebidaActualizada));
    }

    // PATCH: solo disponibilidad
    public Optional<Bebida> cambiarDisponibilidad(String id, boolean disponible) {
        return bebidaRepository.findById(id).map(bebida -> {
            bebida.setDisponible(disponible);
            return bebidaRepository.save(bebida);
        });
    }

    // DELETE
    public boolean eliminar(String id) {
        if (!bebidaRepository.existsById(id)) return false;
        bebidaRepository.deleteById(id);
        return true;
    }
}
```

> 💡 El Controller del módulo 05 **no necesita cambios**. La magia de las capas es que el Controller no sabe si los datos vienen de un HashMap o de MongoDB. Solo habla con el Service.

---

## 💡 Consultas personalizadas con `@Query`

A veces los nombres de métodos no son suficientes para consultas complejas. En ese caso usamos `@Query` con sintaxis de MongoDB:

```java
// En BebidaRepository.java
import org.springframework.data.mongodb.repository.Query;

// Buscar bebidas disponibles con precio menor a un valor
@Query("{ 'disponible': true, 'precio': { $lte: ?0 } }")
List<Bebida> findDisponiblesConPrecioMaximo(double precioMax);

// Buscar por categoría y ordenar por precio ascendente
@Query(value = "{ 'categoria': ?0 }", sort = "{ 'precio': 1 }")
List<Bebida> findByCategoriaOrdenadas(String categoria);

// Contar bebidas por categoría
@Query(value = "{ 'categoria': ?0 }", count = true)
long contarPorCategoria(String categoria);
```

> 💡 El `?0` representa el primer parámetro del método, `?1` el segundo, y así sucesivamente.

---

## 💡 Cargando datos iniciales

Es útil tener datos de prueba que se carguen automáticamente al iniciar la aplicación.

```java
// src/main/java/com/michicafe/config/DataLoader.java
package com.michicafe.config;

import com.michicafe.model.Bebida;
import com.michicafe.repository.BebidaRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component  // Spring ejecuta esto al arrancar la aplicación
public class DataLoader implements CommandLineRunner {

    @Autowired
    private BebidaRepository bebidaRepository;

    @Override
    public void run(String... args) {
        // Solo cargamos datos si la colección está vacía
        if (bebidaRepository.count() == 0) {
            System.out.println("🐱 Cargando menú inicial del Michi Café...");

            bebidaRepository.save(new Bebida(
                "Espresso", 35.00, "caliente",
                "Café concentrado y puro", true));
            bebidaRepository.save(new Bebida(
                "Latte de vainilla", 45.50, "caliente",
                "Espresso con leche vaporizada y vainilla", true));
            bebidaRepository.save(new Bebida(
                "Cappuccino", 48.00, "caliente",
                "Espresso con leche y espuma cremosa", true));
            bebidaRepository.save(new Bebida(
                "Matcha Latte", 55.00, "caliente",
                "Té matcha japonés con leche de avena", true));
            bebidaRepository.save(new Bebida(
                "Cold Brew", 50.00, "frio",
                "Café infusionado en frío por 12 horas", true));
            bebidaRepository.save(new Bebida(
                "Chai Latte", 52.00, "caliente",
                "Té chai especiado con leche", false));

            System.out.println("✅ Menú cargado: " + bebidaRepository.count() + " bebidas");
        } else {
            System.out.println("📋 El menú ya tiene " + bebidaRepository.count() + " bebidas");
        }
    }
}
```

---

## 💡 El modelo Cliente con MongoDB

Ahora hacemos lo mismo para los clientes:

```java
// src/main/java/com/michicafe/model/Cliente.java
package com.michicafe.model;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;
import org.springframework.data.mongodb.core.index.Indexed;
import java.time.LocalDateTime;

@Document(collection = "clientes")
public class Cliente {

    @Id
    private String id;

    private String nombre;

    @Indexed(unique = true)  // El correo debe ser único en la colección
    private String correo;

    private int puntosFidelidad;
    private LocalDateTime fechaRegistro;

    public Cliente() {}

    public Cliente(String nombre, String correo) {
        this.nombre = nombre;
        this.correo = correo;
        this.puntosFidelidad = 0;
        this.fechaRegistro = LocalDateTime.now();
    }

    // Getters
    public String getId()                  { return id; }
    public String getNombre()              { return nombre; }
    public String getCorreo()              { return correo; }
    public int getPuntosFidelidad()        { return puntosFidelidad; }
    public LocalDateTime getFechaRegistro(){ return fechaRegistro; }

    // Setters
    public void setId(String id)                       { this.id = id; }
    public void setNombre(String nombre)               { this.nombre = nombre; }
    public void setCorreo(String correo)               { this.correo = correo; }
    public void setPuntosFidelidad(int puntos)         { this.puntosFidelidad = puntos; }
    public void setFechaRegistro(LocalDateTime fecha)  { this.fechaRegistro = fecha; }
}
```

```java
// src/main/java/com/michicafe/repository/ClienteRepository.java
package com.michicafe.repository;

import com.michicafe.model.Cliente;
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Optional;

@Repository
public interface ClienteRepository extends MongoRepository<Cliente, String> {

    Optional<Cliente> findByCorreo(String correo);

    List<Cliente> findByNombreContainingIgnoreCase(String nombre);

    List<Cliente> findByPuntosFidelidadGreaterThanEqual(int puntos);

    boolean existsByCorreo(String correo);
}
```

```java
// src/main/java/com/michicafe/service/ClienteService.java
package com.michicafe.service;

import com.michicafe.model.Cliente;
import com.michicafe.repository.ClienteRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

@Service
public class ClienteService {

    @Autowired
    private ClienteRepository clienteRepository;

    public Cliente registrar(Cliente cliente) {
        if (clienteRepository.existsByCorreo(cliente.getCorreo())) {
            throw new RuntimeException(
                "Ya existe un cliente con el correo: " + cliente.getCorreo());
        }
        return clienteRepository.save(cliente);
    }

    public List<Cliente> obtenerTodos() {
        return clienteRepository.findAll();
    }

    public Optional<Cliente> obtenerPorId(String id) {
        return clienteRepository.findById(id);
    }

    public Optional<Cliente> obtenerPorCorreo(String correo) {
        return clienteRepository.findByCorreo(correo);
    }

    public Optional<Cliente> agregarPuntos(String id, int puntos) {
        return clienteRepository.findById(id).map(cliente -> {
            cliente.setPuntosFidelidad(cliente.getPuntosFidelidad() + puntos);
            return clienteRepository.save(cliente);
        });
    }

    public boolean eliminar(String id) {
        if (!clienteRepository.existsById(id)) return false;
        clienteRepository.deleteById(id);
        return true;
    }
}
```

---

## 💡 Estructura del proyecto actualizada

```
michicafe/
└── src/main/java/com/michicafe/
    ├── MichicafeApplication.java
    ├── config/
    │   └── DataLoader.java              ← Datos iniciales
    ├── controller/
    │   ├── MichiController.java
    │   ├── BebidaController.java
    │   └── ClienteController.java
    ├── service/
    │   ├── BebidaService.java           ← Ahora usa Repository
    │   └── ClienteService.java
    ├── repository/
    │   ├── BebidaRepository.java        ← Habla con MongoDB
    │   └── ClienteRepository.java
    ├── model/
    │   ├── Bebida.java                  ← Anotada con @Document
    │   ├── Cliente.java
    │   └── ErrorResponse.java
    └── exception/
        └── GlobalExceptionHandler.java
```

### El flujo completo de una petición

```
Cliente HTTP (Postman / App)
        ↓  petición HTTP
  BebidaController          ← recibe y responde
        ↓  llama al service
  BebidaService             ← lógica de negocio
        ↓  llama al repository
  BebidaRepository          ← habla con MongoDB
        ↓  consulta/guarda
     MongoDB                ← base de datos real
```

---

## 💡 Probando con Postman

Con MongoDB conectado, ahora los datos persisten entre reinicios.

### Crear una bebida nueva

```
POST http://localhost:8080/api/bebidas
Content-Type: application/json

{
  "nombre": "Frappuccino de Caramelo",
  "precio": 65.00,
  "categoria": "frio",
  "descripcion": "Café frío con caramelo y crema batida",
  "disponible": true
}
```

Respuesta (201 Created):
```json
{
  "id": "64a1b2c3d4e5f6a7b8c9d0e7",
  "nombre": "Frappuccino de Caramelo",
  "precio": 65.0,
  "categoria": "frio",
  "descripcion": "Café frío con caramelo y crema batida",
  "disponible": true
}
```

Ahora si reinicias el servidor y haces `GET /api/bebidas`, el Frappuccino sigue ahí. ¡Los datos son permanentes!

### Registrar un cliente

```
POST http://localhost:8080/api/clientes
Content-Type: application/json

{
  "nombre": "Sofía",
  "correo": "sofia@email.com"
}
```

---

## 🧪 Ejercicios del Módulo 7

### Ejercicio 1: Verificar la persistencia

1. Arranca la aplicación y crea 3 bebidas nuevas con POST
2. Detén la aplicación (Ctrl+C)
3. Vuelve a arrancarla
4. Haz GET /api/bebidas y verifica que las 3 bebidas siguen ahí

### Ejercicio 2: ClienteController completo

Crea el `ClienteController` con estos endpoints:
- `GET /api/clientes` → todos los clientes
- `GET /api/clientes/{id}` → cliente por id
- `GET /api/clientes/correo/{correo}` → cliente por correo
- `POST /api/clientes` → registrar cliente nuevo
- `PATCH /api/clientes/{id}/puntos?cantidad=50` → agregar puntos
- `DELETE /api/clientes/{id}` → eliminar cliente

### Ejercicio 3: Consultas personalizadas

Agrega en `BebidaRepository` una consulta con `@Query` que:
- Busque bebidas disponibles cuyo precio sea menor al indicado
- Ordene los resultados por precio de menor a mayor

Expónla en el controller como: `GET /api/bebidas/ofertas?precioMax=50`

---

## ✅ Resumen del Módulo 7

| Concepto | ¿Qué es? | Analogía del Michi Café |
|----------|----------|------------------------|
| Spring Data MongoDB | Librería que conecta Spring con MongoDB | El asistente del archivero |
| `@Document` | Marca la clase como documento MongoDB | "Esta ficha va en esta carpeta" |
| `@Id` | Campo que es el `_id` de MongoDB | El número de folio |
| `MongoRepository` | Interfaz con operaciones CRUD listas | El archivero con métodos predefinidos |
| Métodos por nombre | Spring genera consultas según el nombre | El asistente que entiende instrucciones en español |
| `@Query` | Consulta personalizada en sintaxis MongoDB | Instrucción específica para el archivero |
| `DataLoader` | Carga datos iniciales al arrancar | Preparar el café antes de abrir |
| `@Indexed(unique=true)` | Campo único en la colección | "No puede haber dos fichas con el mismo correo" |

---

## ➡️ Siguiente Módulo

En el **Módulo 08** aprenderemos qué son los **microservicios** en profundidad: cómo dividir nuestra aplicación en servicios independientes, por qué es mejor que tener todo junto, y cómo planificar la arquitectura del Michi Café completo.

> 🐾 "Cuando Spring Boot y MongoDB trabajan juntos, el Michi Café nunca olvida a sus clientes ni sus pedidos."
