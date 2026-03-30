# Módulo 11 — Microservicio de Bebidas
## 🐱 Analogía: El menú digital del Michi Café

---

## 🏠 Introducción

El **Servicio de Bebidas** gestiona el catálogo completo del menú del Michi Café. Es el microservicio que otros servicios consultarán para obtener información de las bebidas (nombre, precio, disponibilidad).

Este módulo es más corto porque ya conocemos bien los patrones. Lo importante aquí es ver cómo queda como microservicio independiente y prepararlo para ser consultado por otros servicios.

---

## 🛠️ Creando el proyecto

### Spring Initializr

| Campo | Valor |
|-------|-------|
| Artifact | `servicio-bebidas` |
| Puerto | `8081` |
| Base de datos | `michicafe-bebidas` |

```properties
# application.properties
spring.application.name=servicio-bebidas
server.port=8081
spring.data.mongodb.host=localhost
spring.data.mongodb.port=27017
spring.data.mongodb.database=michicafe-bebidas
```

---

## 💡 El modelo Bebida

```java
// src/main/java/com/michicafe/bebidas/model/Bebida.java
package com.michicafe.bebidas.model;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;
import java.util.List;

@Document(collection = "bebidas")
public class Bebida {

    @Id
    private String id;

    private String nombre;
    private String descripcion;
    private double precio;
    private String categoria;       // "caliente", "frio", "especial"
    private String tamanio;         // "chico", "mediano", "grande"
    private List<String> ingredientes;
    private InformacionNutricional informacionNutricional;
    private boolean disponible;
    private String imagenUrl;

    public Bebida() {}

    public Bebida(String nombre, String descripcion, double precio,
                  String categoria, String tamanio, boolean disponible) {
        this.nombre = nombre;
        this.descripcion = descripcion;
        this.precio = precio;
        this.categoria = categoria;
        this.tamanio = tamanio;
        this.disponible = disponible;
    }

    // Getters
    public String getId()                                    { return id; }
    public String getNombre()                                { return nombre; }
    public String getDescripcion()                           { return descripcion; }
    public double getPrecio()                                { return precio; }
    public String getCategoria()                             { return categoria; }
    public String getTamanio()                               { return tamanio; }
    public List<String> getIngredientes()                    { return ingredientes; }
    public InformacionNutricional getInformacionNutricional(){ return informacionNutricional; }
    public boolean isDisponible()                            { return disponible; }
    public String getImagenUrl()                             { return imagenUrl; }

    // Setters
    public void setId(String id)                                              { this.id = id; }
    public void setNombre(String nombre)                                      { this.nombre = nombre; }
    public void setDescripcion(String descripcion)                            { this.descripcion = descripcion; }
    public void setPrecio(double precio)                                      { this.precio = precio; }
    public void setCategoria(String categoria)                                { this.categoria = categoria; }
    public void setTamanio(String tamanio)                                    { this.tamanio = tamanio; }
    public void setIngredientes(List<String> ingredientes)                    { this.ingredientes = ingredientes; }
    public void setInformacionNutricional(InformacionNutricional info)        { this.informacionNutricional = info; }
    public void setDisponible(boolean disponible)                             { this.disponible = disponible; }
    public void setImagenUrl(String imagenUrl)                                { this.imagenUrl = imagenUrl; }
}
```

```java
// src/main/java/com/michicafe/bebidas/model/InformacionNutricional.java
package com.michicafe.bebidas.model;

// Objeto embebido dentro de Bebida (no es un @Document)
public class InformacionNutricional {
    private int calorias;
    private double proteinas;
    private double carbohidratos;
    private double grasas;
    private boolean contieneLactosa;
    private boolean contieneCafeina;

    public InformacionNutricional() {}

    // Getters y Setters
    public int getCalorias()           { return calorias; }
    public double getProteinas()       { return proteinas; }
    public double getCarbohidratos()   { return carbohidratos; }
    public double getGrasas()          { return grasas; }
    public boolean isContieneLactosa() { return contieneLactosa; }
    public boolean isContieneCafeina() { return contieneCafeina; }

    public void setCalorias(int calorias)                 { this.calorias = calorias; }
    public void setProteinas(double proteinas)            { this.proteinas = proteinas; }
    public void setCarbohidratos(double carbohidratos)    { this.carbohidratos = carbohidratos; }
    public void setGrasas(double grasas)                  { this.grasas = grasas; }
    public void setContieneLactosa(boolean contieneLactosa){ this.contieneLactosa = contieneLactosa; }
    public void setContieneCafeina(boolean contieneCafeina){ this.contieneCafeina = contieneCafeina; }
}
```

---

## 💡 El Repository

```java
// src/main/java/com/michicafe/bebidas/repository/BebidaRepository.java
package com.michicafe.bebidas.repository;

import com.michicafe.bebidas.model.Bebida;
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.data.mongodb.repository.Query;
import org.springframework.stereotype.Repository;
import java.util.List;

@Repository
public interface BebidaRepository extends MongoRepository<Bebida, String> {

    List<Bebida> findByCategoria(String categoria);

    List<Bebida> findByDisponibleTrue();

    List<Bebida> findByCategoriaAndDisponibleTrue(String categoria);

    List<Bebida> findByNombreContainingIgnoreCase(String nombre);

    List<Bebida> findByPrecioBetween(double min, double max);

    @Query("{ 'disponible': true, 'precio': { $lte: ?0 } }")
    List<Bebida> findDisponiblesConPrecioMaximo(double precioMax);

    @Query("{ 'informacionNutricional.contieneLactosa': false, 'disponible': true }")
    List<Bebida> findSinLactosa();

    @Query("{ 'informacionNutricional.contieneCafeina': false, 'disponible': true }")
    List<Bebida> findSinCafeina();
}
```

---

## 💡 El Service

```java
// src/main/java/com/michicafe/bebidas/service/BebidaService.java
package com.michicafe.bebidas.service;

import com.michicafe.bebidas.model.Bebida;
import com.michicafe.bebidas.repository.BebidaRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

@Service
public class BebidaService {

    @Autowired
    private BebidaRepository bebidaRepository;

    public Bebida guardar(Bebida bebida) {
        validar(bebida);
        return bebidaRepository.save(bebida);
    }

    public List<Bebida> obtenerTodas() {
        return bebidaRepository.findAll();
    }

    public List<Bebida> obtenerDisponibles() {
        return bebidaRepository.findByDisponibleTrue();
    }

    public Optional<Bebida> obtenerPorId(String id) {
        return bebidaRepository.findById(id);
    }

    public List<Bebida> obtenerPorCategoria(String categoria) {
        return bebidaRepository.findByCategoriaAndDisponibleTrue(categoria);
    }

    public List<Bebida> buscarPorNombre(String nombre) {
        return bebidaRepository.findByNombreContainingIgnoreCase(nombre);
    }

    public List<Bebida> obtenerPorRangoPrecio(double min, double max) {
        return bebidaRepository.findByPrecioBetween(min, max);
    }

    public List<Bebida> obtenerSinLactosa() {
        return bebidaRepository.findSinLactosa();
    }

    public List<Bebida> obtenerSinCafeina() {
        return bebidaRepository.findSinCafeina();
    }

    public Optional<Bebida> actualizar(String id, Bebida bebidaActualizada) {
        if (!bebidaRepository.existsById(id)) return Optional.empty();
        validar(bebidaActualizada);
        bebidaActualizada.setId(id);
        return Optional.of(bebidaRepository.save(bebidaActualizada));
    }

    public Optional<Bebida> cambiarDisponibilidad(String id, boolean disponible) {
        return bebidaRepository.findById(id).map(bebida -> {
            bebida.setDisponible(disponible);
            return bebidaRepository.save(bebida);
        });
    }

    public boolean eliminar(String id) {
        if (!bebidaRepository.existsById(id)) return false;
        bebidaRepository.deleteById(id);
        return true;
    }

    private void validar(Bebida bebida) {
        if (bebida.getNombre() == null || bebida.getNombre().isBlank()) {
            throw new RuntimeException("El nombre de la bebida no puede estar vacío");
        }
        if (bebida.getPrecio() <= 0) {
            throw new RuntimeException("El precio debe ser mayor a 0");
        }
        List<String> categoriasValidas = List.of("caliente", "frio", "especial");
        if (!categoriasValidas.contains(bebida.getCategoria())) {
            throw new RuntimeException("Categoría inválida. Use: caliente, frio o especial");
        }
    }
}
```

---

## 💡 El Controller

```java
// src/main/java/com/michicafe/bebidas/controller/BebidaController.java
package com.michicafe.bebidas.controller;

import com.michicafe.bebidas.model.Bebida;
import com.michicafe.bebidas.service.BebidaService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/bebidas")
public class BebidaController {

    @Autowired
    private BebidaService bebidaService;

    @GetMapping
    public ResponseEntity<List<Bebida>> obtenerDisponibles() {
        return ResponseEntity.ok(bebidaService.obtenerDisponibles());
    }

    @GetMapping("/todas")
    public ResponseEntity<List<Bebida>> obtenerTodas() {
        return ResponseEntity.ok(bebidaService.obtenerTodas());
    }

    @GetMapping("/{id}")
    public ResponseEntity<Bebida> obtenerPorId(@PathVariable String id) {
        return bebidaService.obtenerPorId(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    @GetMapping("/categoria/{categoria}")
    public ResponseEntity<List<Bebida>> obtenerPorCategoria(@PathVariable String categoria) {
        return ResponseEntity.ok(bebidaService.obtenerPorCategoria(categoria));
    }

    @GetMapping("/buscar")
    public ResponseEntity<List<Bebida>> buscar(@RequestParam String nombre) {
        return ResponseEntity.ok(bebidaService.buscarPorNombre(nombre));
    }

    @GetMapping("/precio")
    public ResponseEntity<List<Bebida>> obtenerPorPrecio(
            @RequestParam(defaultValue = "0") double min,
            @RequestParam(defaultValue = "9999") double max) {
        return ResponseEntity.ok(bebidaService.obtenerPorRangoPrecio(min, max));
    }

    @GetMapping("/sin-lactosa")
    public ResponseEntity<List<Bebida>> obtenerSinLactosa() {
        return ResponseEntity.ok(bebidaService.obtenerSinLactosa());
    }

    @GetMapping("/sin-cafeina")
    public ResponseEntity<List<Bebida>> obtenerSinCafeina() {
        return ResponseEntity.ok(bebidaService.obtenerSinCafeina());
    }

    @PostMapping
    public ResponseEntity<Bebida> crear(@RequestBody Bebida bebida) {
        return ResponseEntity.status(HttpStatus.CREATED).body(bebidaService.guardar(bebida));
    }

    @PutMapping("/{id}")
    public ResponseEntity<Bebida> actualizar(@PathVariable String id, @RequestBody Bebida bebida) {
        return bebidaService.actualizar(id, bebida)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    @PatchMapping("/{id}/disponibilidad")
    public ResponseEntity<Bebida> cambiarDisponibilidad(
            @PathVariable String id,
            @RequestParam boolean valor) {
        return bebidaService.cambiarDisponibilidad(id, valor)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> eliminar(@PathVariable String id) {
        if (bebidaService.eliminar(id)) return ResponseEntity.noContent().build();
        return ResponseEntity.notFound().build();
    }
}
```

---

## 💡 DataLoader con menú completo

```java
// src/main/java/com/michicafe/bebidas/config/DataLoader.java
package com.michicafe.bebidas.config;

import com.michicafe.bebidas.model.Bebida;
import com.michicafe.bebidas.model.InformacionNutricional;
import com.michicafe.bebidas.repository.BebidaRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

import java.util.Arrays;

@Component
public class DataLoader implements CommandLineRunner {

    @Autowired
    private BebidaRepository bebidaRepository;

    @Override
    public void run(String... args) {
        if (bebidaRepository.count() > 0) return;

        System.out.println("🐱 Cargando menú del Michi Café...");

        // Espresso
        Bebida espresso = new Bebida("Espresso",
            "Café concentrado y puro, el clásico del Michi Café",
            35.00, "caliente", "chico", true);
        espresso.setIngredientes(Arrays.asList("café molido", "agua"));
        InformacionNutricional infoEspresso = new InformacionNutricional();
        infoEspresso.setCalorias(5);
        infoEspresso.setContieneLactosa(false);
        infoEspresso.setContieneCafeina(true);
        espresso.setInformacionNutricional(infoEspresso);
        bebidaRepository.save(espresso);

        // Latte de vainilla
        Bebida latte = new Bebida("Latte de vainilla",
            "Espresso suave con leche vaporizada y toque de vainilla",
            45.50, "caliente", "mediano", true);
        latte.setIngredientes(Arrays.asList("espresso", "leche", "jarabe de vainilla"));
        InformacionNutricional infoLatte = new InformacionNutricional();
        infoLatte.setCalorias(180);
        infoLatte.setContieneLactosa(true);
        infoLatte.setContieneCafeina(true);
        latte.setInformacionNutricional(infoLatte);
        bebidaRepository.save(latte);

        // Matcha Latte
        Bebida matcha = new Bebida("Matcha Latte",
            "Té matcha japonés premium con leche de avena",
            55.00, "caliente", "mediano", true);
        matcha.setIngredientes(Arrays.asList("matcha en polvo", "leche de avena", "miel"));
        InformacionNutricional infoMatcha = new InformacionNutricional();
        infoMatcha.setCalorias(150);
        infoMatcha.setContieneLactosa(false);
        infoMatcha.setContieneCafeina(true);
        matcha.setInformacionNutricional(infoMatcha);
        bebidaRepository.save(matcha);

        // Cold Brew
        Bebida coldBrew = new Bebida("Cold Brew",
            "Café infusionado en frío por 12 horas, suave y refrescante",
            50.00, "frio", "grande", true);
        coldBrew.setIngredientes(Arrays.asList("café molido grueso", "agua fría"));
        InformacionNutricional infoColdBrew = new InformacionNutricional();
        infoColdBrew.setCalorias(10);
        infoColdBrew.setContieneLactosa(false);
        infoColdBrew.setContieneCafeina(true);
        coldBrew.setInformacionNutricional(infoColdBrew);
        bebidaRepository.save(coldBrew);

        System.out.println("✅ Menú cargado: " + bebidaRepository.count() + " bebidas");
    }
}
```

---

## 💡 Tabla de endpoints del servicio

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/bebidas` | Bebidas disponibles |
| GET | `/api/bebidas/todas` | Todas (admin) |
| GET | `/api/bebidas/{id}` | Por ID |
| GET | `/api/bebidas/categoria/{cat}` | Por categoría |
| GET | `/api/bebidas/buscar?nombre=latte` | Buscar por nombre |
| GET | `/api/bebidas/precio?min=40&max=55` | Por rango de precio |
| GET | `/api/bebidas/sin-lactosa` | Sin lactosa |
| GET | `/api/bebidas/sin-cafeina` | Sin cafeína |
| POST | `/api/bebidas` | Crear bebida |
| PUT | `/api/bebidas/{id}` | Actualizar bebida |
| PATCH | `/api/bebidas/{id}/disponibilidad` | Cambiar disponibilidad |
| DELETE | `/api/bebidas/{id}` | Eliminar bebida |

---

## 💡 Los tres servicios corriendo juntos

En este punto tenemos tres microservicios independientes:

```
servicio-bebidas   → http://localhost:8081
servicio-clientes  → http://localhost:8082
servicio-pedidos   → http://localhost:8083
```

Cada uno con su propia base de datos en MongoDB:
```
michicafe-bebidas
michicafe-clientes
michicafe-pedidos
```

Para probarlos, abre tres terminales y ejecuta cada uno:
```bash
# Terminal 1
cd servicio-bebidas && mvn spring-boot:run

# Terminal 2
cd servicio-clientes && mvn spring-boot:run

# Terminal 3
cd servicio-pedidos && mvn spring-boot:run
```

---

## 🧪 Ejercicios del Módulo 11

### Ejercicio 1: Menú completo

Agrega al DataLoader las siguientes bebidas con su información nutricional:
- Cappuccino (caliente, $48, con lactosa, con cafeína)
- Chai Latte (caliente, $52, con lactosa, sin cafeína)
- Frappuccino de Caramelo (frío, $65, con lactosa, con cafeína)

### Ejercicio 2: Filtros combinados

Agrega el endpoint `GET /api/bebidas/filtrar` que acepte parámetros opcionales:
- `categoria` (caliente/frio/especial)
- `sinLactosa` (true/false)
- `precioMax` (número)

Y devuelva las bebidas que cumplan todos los filtros indicados.

### Ejercicio 3: Verificar disponibilidad

Agrega el endpoint `GET /api/bebidas/{id}/disponible` que devuelva simplemente:
```json
{ "bebidaId": "64a1...", "nombre": "Espresso", "disponible": true }
```

Este endpoint será muy útil cuando el servicio de pedidos necesite verificar si una bebida está disponible antes de aceptar el pedido.

---

## ✅ Resumen del Módulo 11

| Concepto | Aplicado en este módulo |
|----------|------------------------|
| Microservicio independiente | Puerto 8081, BD propia |
| Documentos embebidos | `InformacionNutricional` dentro de `Bebida` |
| Consultas con `@Query` | Filtros por lactosa y cafeína |
| Validaciones en el Service | Precio, nombre y categoría |
| DataLoader | Menú inicial con datos reales |

---

## ➡️ Siguiente Módulo

En el **Módulo 12** aprenderemos cómo los microservicios se comunican entre sí. El servicio de pedidos necesita consultar precios reales al servicio de bebidas y verificar clientes en el servicio de clientes.

> 🐾 "El menú del Michi Café siempre está actualizado, sin importar cuántas sucursales haya."
