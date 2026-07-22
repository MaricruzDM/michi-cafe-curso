# Módulo 03 — Programación Orientada a Objetos (POO)
## 🐱 Analogía: Los roles del Michi Café

---

## 🏠 Introducción

En el módulo anterior aprendimos a escribir instrucciones básicas en Java: variables, condiciones, ciclos y métodos.

Pero hay un problema: si el Michi Café tiene 50 clientes, ¿vamos a crear 50 variables separadas para cada uno?

```java
// Esto sería un caos 😱
String nombreCliente1 = "Sofía";
int edadCliente1 = 25;
String nombreCliente2 = "Carlos";
int edadCliente2 = 30;
// ... y así 48 veces más
```

La solución es la **Programación Orientada a Objetos (POO)**: una forma de organizar el código que imita cómo funciona el mundo real.

---

## 💡 ¿Qué es la POO?

La POO es una forma de programar donde organizamos el código en **objetos**.

Un objeto es una representación de algo del mundo real dentro del programa.

### Analogía 🐾

En el Michi Café tenemos:
- **Clientes** (cada uno con nombre, correo, puntos de fidelidad)
- **Bebidas** (cada una con nombre, precio, tamaño)
- **Pedidos** (cada uno con cliente, bebidas, total, estado)
- **Gatitos empleados** (cada uno con nombre, rol, turno)

Cada uno de estos es un tipo de **objeto**. Y para crear objetos, primero necesitamos un **molde**.

---

## 💡 Clases: el molde del gatito

Una **clase** es el molde o plantilla para crear objetos.

### Analogía 🐾

Imagina que tienes un molde para hacer galletas con forma de gatito. Ese molde es la **clase**. Cada galleta que haces con ese molde es un **objeto** (una instancia de la clase).

Todas las galletas tienen la misma forma (misma clase), pero pueden tener diferente decoración (diferentes valores en sus atributos).

```
Clase Cliente  →  molde
    ↓
objeto cliente1 (Sofía, 25 años, 150 puntos)
objeto cliente2 (Carlos, 30 años, 80 puntos)
objeto cliente3 (Ana, 22 años, 200 puntos)
```

### Cómo crear una clase en Java

```java
public class Cliente {
    // ATRIBUTOS: las características del cliente
    String nombre;
    String correo;
    int puntosFidelidad;
    boolean tieneTarjeta;
}
```

### Cómo crear objetos a partir de la clase

```java
public class MichiCafe {
    public static void main(String[] args) {
        // Creamos un objeto de tipo Cliente
        Cliente cliente1 = new Cliente();

        // Asignamos valores a sus atributos
        cliente1.nombre = "Sofía";
        cliente1.correo = "sofia@email.com";
        cliente1.puntosFidelidad = 150;
        cliente1.tieneTarjeta = true;

        // Creamos otro cliente
        Cliente cliente2 = new Cliente();
        cliente2.nombre = "Carlos";
        cliente2.correo = "carlos@email.com";
        cliente2.puntosFidelidad = 80;
        cliente2.tieneTarjeta = false;

        // Mostramos la información
        System.out.println("Cliente: " + cliente1.nombre);
        System.out.println("Puntos: " + cliente1.puntosFidelidad);
    }
}
```

> 💡 La palabra `new` es la que "fabrica" el objeto usando el molde (la clase).

---

## 💡 Atributos y Métodos: lo que el gatito ES y lo que HACE

Una clase tiene dos partes:

- **Atributos**: las características (lo que el objeto ES o TIENE)
- **Métodos**: las acciones (lo que el objeto PUEDE HACER)

### Analogía 🐾

Un gatito barista:
- **Atributos**: nombre, edad, turno, especialidad
- **Métodos**: prepararCafe(), tomarPedido(), cobrar()

```java
public class GatitoBarista {

    // ATRIBUTOS (lo que el gatito ES/TIENE)
    String nombre;
    String especialidad;
    String turno;
    int cafesPreparados;

    // MÉTODOS (lo que el gatito HACE)
    public void prepararCafe(String tipoCafe) {
        cafesPreparados++;
        System.out.println(nombre + " está preparando un " + tipoCafe + " ☕");
        System.out.println("Cafés preparados hoy: " + cafesPreparados);
    }

    public void saludar() {
        System.out.println("¡Miau! Soy " + nombre + ", tu barista del turno " + turno);
    }

    public void mostrarEspecialidad() {
        System.out.println(nombre + " se especializa en: " + especialidad);
    }
}
```

Usando la clase:

```java
public class MichiCafe {
    public static void main(String[] args) {
        GatitoBarista gatito1 = new GatitoBarista();
        gatito1.nombre = "Mochi";
        gatito1.especialidad = "Latte Art";
        gatito1.turno = "mañana";
        gatito1.cafesPreparados = 0;

        gatito1.saludar();
        gatito1.prepararCafe("Cappuccino");
        gatito1.prepararCafe("Latte de vainilla");
        gatito1.mostrarEspecialidad();
    }
}
```

Salida:
```
¡Miau! Soy Mochi, tu barista del turno mañana
Mochi está preparando un Cappuccino ☕
Cafés preparados hoy: 1
Mochi está preparando un Latte de vainilla ☕
Cafés preparados hoy: 2
Mochi se especializa en: Latte Art
```

---

## 💡 Constructores: el formulario de contratación

Un **constructor** es un método especial que se ejecuta automáticamente cuando creamos un objeto. Sirve para darle valores iniciales.

### Analogía 🐾

Cuando contratas a un nuevo gatito para el café, llenas un formulario con su nombre, especialidad y turno. Ese formulario es el **constructor**: te obliga a dar la información necesaria desde el principio.

Sin constructor (incómodo):
```java
GatitoBarista gatito = new GatitoBarista();
gatito.nombre = "Mochi";         // tienes que asignar
gatito.especialidad = "Latte Art"; // cada cosa
gatito.turno = "mañana";         // por separado
```

Con constructor (mucho mejor):
```java
GatitoBarista gatito = new GatitoBarista("Mochi", "Latte Art", "mañana");
// Todo en una sola línea 🎉
```

### Cómo crear un constructor

```java
public class GatitoBarista {

    String nombre;
    String especialidad;
    String turno;
    int cafesPreparados;

    // CONSTRUCTOR: mismo nombre que la clase, sin tipo de retorno
    public GatitoBarista(String nombre, String especialidad, String turno) {
        this.nombre = nombre;             // "this" se refiere al objeto actual
        this.especialidad = especialidad;
        this.turno = turno;
        this.cafesPreparados = 0;         // siempre empieza en 0
    }

    public void prepararCafe(String tipoCafe) {
        cafesPreparados++;
        System.out.println(nombre + " prepara un " + tipoCafe + " ☕ (#" + cafesPreparados + ")");
    }

    public void saludar() {
        System.out.println("¡Miau! Soy " + nombre + ", turno " + turno);
    }
}
```

Usando el constructor:

```java
public class MichiCafe {
    public static void main(String[] args) {
        // Creamos gatitos con el constructor
        GatitoBarista mochi = new GatitoBarista("Mochi", "Latte Art", "mañana");
        GatitoBarista nube  = new GatitoBarista("Nube", "Café frío", "tarde");
        GatitoBarista luna  = new GatitoBarista("Luna", "Repostería", "noche");

        mochi.saludar();
        nube.saludar();
        luna.saludar();

        mochi.prepararCafe("Cappuccino");
        mochi.prepararCafe("Espresso");
        nube.prepararCafe("Cold Brew");
    }
}
```

Salida:
```
¡Miau! Soy Mochi, turno mañana
¡Miau! Soy Nube, turno tarde
¡Miau! Soy Luna, turno noche
Mochi prepara un Cappuccino ☕ (#1)
Mochi prepara un Espresso ☕ (#2)
Nube prepara un Cold Brew ☕ (#1)
```

> 💡 `this` es una palabra especial que significa "este objeto". Se usa cuando el nombre del parámetro es igual al nombre del atributo, para diferenciarlos.

---

## 💡 Encapsulamiento: la caja fuerte del gatito

El **encapsulamiento** es uno de los pilares de la POO. Consiste en proteger los datos de un objeto para que no puedan ser modificados directamente desde afuera.

### Analogía 🐾

La caja registradora del Michi Café tiene un candado. Solo el gatito cajero puede abrirla y modificar el dinero. Los clientes no pueden meter la mano directamente.

Si alguien quiere saber cuánto hay en la caja, tiene que preguntarle al cajero (usar un método). Si quiere agregar dinero, tiene que dárselo al cajero (usar otro método).

### Sin encapsulamiento (peligroso)

```java
Cliente cliente = new Cliente();
cliente.puntosFidelidad = -9999; // ¡Cualquiera puede poner valores inválidos!
```

### Con encapsulamiento (seguro)

Usamos `private` para proteger los atributos, y `getters/setters` para acceder a ellos de forma controlada:

```java
public class Cliente {

    // Atributos PRIVADOS (nadie puede tocarlos directamente)
    private String nombre;
    private String correo;
    private int puntosFidelidad;

    // Constructor
    public Cliente(String nombre, String correo) {
        this.nombre = nombre;
        this.correo = correo;
        this.puntosFidelidad = 0; // siempre empieza en 0
    }

    // GETTER: permite LEER el valor (el cajero te dice cuántos puntos tienes)
    public String getNombre() {
        return nombre;
    }

    public int getPuntosFidelidad() {
        return puntosFidelidad;
    }

    public String getCorreo() {
        return correo;
    }

    // SETTER: permite MODIFICAR el valor con validación
    public void setCorreo(String correo) {
        // Validamos que el correo tenga @ antes de guardarlo
        if (correo.contains("@")) {
            this.correo = correo;
        } else {
            System.out.println("Error: correo inválido");
        }
    }

    // Método para agregar puntos (con validación)
    public void agregarPuntos(int puntos) {
        if (puntos > 0) {
            this.puntosFidelidad += puntos;
            System.out.println("+" + puntos + " puntos para " + nombre +
                               ". Total: " + puntosFidelidad);
        } else {
            System.out.println("Error: los puntos deben ser positivos");
        }
    }
}
```

Usando la clase encapsulada:

```java
public class MichiCafe {
    public static void main(String[] args) {
        Cliente sofia = new Cliente("Sofía", "sofia@email.com");

        // Leemos datos con getters
        System.out.println("Cliente: " + sofia.getNombre());
        System.out.println("Puntos: " + sofia.getPuntosFidelidad());

        // Agregamos puntos de forma controlada
        sofia.agregarPuntos(50);
        sofia.agregarPuntos(30);
        sofia.agregarPuntos(-10); // Esto dará error controlado

        // Actualizamos correo con validación
        sofia.setCorreo("nuevo-correo@email.com"); // OK
        sofia.setCorreo("correo-sin-arroba");       // Error controlado
    }
}
```

Salida:
```
Cliente: Sofía
Puntos: 0
+50 puntos para Sofía. Total: 50
+30 puntos para Sofía. Total: 80
Error: los puntos deben ser positivos
Error: correo inválido
```

---

## 💡 Herencia: el gatito aprende de otro gatito

La **herencia** permite crear una clase nueva basada en una clase existente, heredando sus atributos y métodos.

### Analogía 🐾

En el Michi Café todos los empleados comparten cosas en común:
- Tienen nombre, turno y salario
- Pueden fichar entrada y salida
- Reciben su pago

Pero cada rol tiene cosas específicas:
- El **barista** prepara bebidas
- El **mesero** toma pedidos y los lleva
- El **cajero** cobra y da cambio

En lugar de repetir el código común en cada clase, creamos una clase **Empleado** con lo común, y las clases específicas **heredan** de ella.

```java
// Clase PADRE (superclase): lo que todos los empleados tienen en común
public class Empleado {

    private String nombre;
    private String turno;
    private double salario;

    public Empleado(String nombre, String turno, double salario) {
        this.nombre = nombre;
        this.turno = turno;
        this.salario = salario;
    }

    public void ficharEntrada() {
        System.out.println(nombre + " fichó entrada. Turno: " + turno + " 🕐");
    }

    public void ficharSalida() {
        System.out.println(nombre + " fichó salida. ¡Hasta mañana! 🌙");
    }

    public String getNombre() { return nombre; }
    public String getTurno()  { return turno; }
    public double getSalario(){ return salario; }
}
```

```java
// Clase HIJA: hereda de Empleado con "extends"
public class Barista extends Empleado {

    private String especialidad;
    private int cafesPreparados;

    // Constructor: llama al constructor del padre con "super"
    public Barista(String nombre, String turno, double salario, String especialidad) {
        super(nombre, turno, salario); // llama al constructor de Empleado
        this.especialidad = especialidad;
        this.cafesPreparados = 0;
    }

    // Método propio del Barista (no lo tienen otros empleados)
    public void prepararCafe(String tipoCafe) {
        cafesPreparados++;
        System.out.println(getNombre() + " prepara " + tipoCafe +
                           " (especialidad: " + especialidad + ") ☕");
    }

    public int getCafesPreparados() { return cafesPreparados; }
}
```

```java
// Otra clase hija
public class Mesero extends Empleado {

    private int pedidosAtendidos;

    public Mesero(String nombre, String turno, double salario) {
        super(nombre, turno, salario);
        this.pedidosAtendidos = 0;
    }

    public void tomarPedido(String cliente, String bebida) {
        pedidosAtendidos++;
        System.out.println(getNombre() + " toma pedido de " + cliente +
                           ": " + bebida + " 📝");
    }

    public void entregarPedido(String cliente) {
        System.out.println(getNombre() + " entrega pedido a " + cliente + " 🐾");
    }
}
```

Usando la herencia:

```java
public class MichiCafe {
    public static void main(String[] args) {
        Barista mochi  = new Barista("Mochi", "mañana", 8000.0, "Latte Art");
        Mesero  canela = new Mesero("Canela", "mañana", 7500.0);

        // Métodos heredados de Empleado
        mochi.ficharEntrada();
        canela.ficharEntrada();

        // Métodos propios de cada rol
        canela.tomarPedido("Sofía", "Latte de vainilla");
        mochi.prepararCafe("Latte de vainilla");
        canela.entregarPedido("Sofía");

        mochi.ficharSalida();
        canela.ficharSalida();

        System.out.println("Cafés preparados por Mochi: " + mochi.getCafesPreparados());
    }
}
```

Salida:
```
Mochi fichó entrada. Turno: mañana 🕐
Canela fichó entrada. Turno: mañana 🕐
Canela toma pedido de Sofía: Latte de vainilla 📝
Mochi prepara Latte de vainilla (especialidad: Latte Art) ☕
Canela entrega pedido a Sofía 🐾
Mochi fichó salida. ¡Hasta mañana! 🌙
Canela fichó salida. ¡Hasta mañana! 🌙
Cafés preparados por Mochi: 1
```

---

## 💡 Polimorfismo: el gatito que hace lo mismo pero diferente

El **polimorfismo** significa que objetos de diferentes clases pueden responder al mismo mensaje de formas distintas.

### Analogía 🐾

Si le dices a cualquier empleado del Michi Café "preséntate", cada uno lo hace a su manera:
- El barista dice su nombre y especialidad
- El mesero dice su nombre y cuántas mesas atiende
- El cajero dice su nombre y el turno de caja

Todos "se presentan", pero cada uno lo hace diferente.

```java
public class Empleado {
    protected String nombre;

    public Empleado(String nombre) {
        this.nombre = nombre;
    }

    // Método que cada hijo puede sobreescribir
    public void presentarse() {
        System.out.println("Hola, soy " + nombre + ", empleado del Michi Café.");
    }
}

public class Barista extends Empleado {
    private String especialidad;

    public Barista(String nombre, String especialidad) {
        super(nombre);
        this.especialidad = especialidad;
    }

    @Override // Esta anotación indica que estamos sobreescribiendo el método del padre
    public void presentarse() {
        System.out.println("¡Miau! Soy " + nombre +
                           ", barista especializado en " + especialidad + " ☕");
    }
}

public class Mesero extends Empleado {
    private int mesas;

    public Mesero(String nombre, int mesas) {
        super(nombre);
        this.mesas = mesas;
    }

    @Override
    public void presentarse() {
        System.out.println("¡Hola! Soy " + nombre +
                           ", mesero. Atiendo " + mesas + " mesas 🐾");
    }
}
```

Usando el polimorfismo:

```java
public class MichiCafe {
    public static void main(String[] args) {
        // Podemos guardar diferentes tipos en un array del tipo padre
        Empleado[] equipo = {
            new Barista("Mochi", "Latte Art"),
            new Mesero("Canela", 5),
            new Barista("Nube", "Café frío"),
            new Mesero("Luna", 4)
        };

        System.out.println("=== EQUIPO DEL MICHI CAFÉ ===");
        for (Empleado empleado : equipo) {
            empleado.presentarse(); // Cada uno se presenta a su manera
        }
    }
}
```

Salida:
```
=== EQUIPO DEL MICHI CAFÉ ===
¡Miau! Soy Mochi, barista especializado en Latte Art ☕
¡Hola! Soy Canela, mesero. Atiendo 5 mesas 🐾
¡Miau! Soy Nube, barista especializado en Café frío ☕
¡Hola! Soy Luna, mesero. Atiendo 4 mesas 🐾
```

---

## 💡 Interfaces: el contrato del gatito

Una **interfaz** es un contrato que dice "cualquier clase que me implemente DEBE tener estos métodos".

### Analogía 🐾

El Michi Café tiene una regla: cualquier empleado que trabaje en el área de atención al cliente DEBE saber:
- Saludar al cliente
- Tomar un pedido
- Despedirse

No importa si es un gatito barista, mesero o cajero. Si trabaja en atención al cliente, debe cumplir ese contrato.

```java
// La interfaz define el CONTRATO
public interface AtencionAlCliente {
    void saludarCliente(String nombreCliente);
    void tomarPedido(String pedido);
    void despedirCliente(String nombreCliente);
}
```

```java
// El Barista implementa el contrato
public class Barista extends Empleado implements AtencionAlCliente {

    public Barista(String nombre) {
        super(nombre);
    }

    @Override
    public void saludarCliente(String nombreCliente) {
        System.out.println("¡Bienvenido " + nombreCliente +
                           " al Michi Café! Soy " + nombre + " ☕");
    }

    @Override
    public void tomarPedido(String pedido) {
        System.out.println(nombre + " anota: " + pedido);
    }

    @Override
    public void despedirCliente(String nombreCliente) {
        System.out.println("¡Hasta pronto " + nombreCliente + "! 🐾");
    }
}
```

---

## 💡 Juntando todo: el modelo completo del Michi Café

Ahora vamos a crear el modelo de datos que usaremos en los próximos módulos:

```java
// Bebida.java
public class Bebida {
    private String nombre;
    private double precio;
    private String categoria; // "caliente", "frio", "especial"

    public Bebida(String nombre, double precio, String categoria) {
        this.nombre = nombre;
        this.precio = precio;
        this.categoria = categoria;
    }

    public String getNombre()    { return nombre; }
    public double getPrecio()    { return precio; }
    public String getCategoria() { return categoria; }

    @Override
    public String toString() {
        return nombre + " ($" + precio + ") [" + categoria + "]";
    }
}
```

```java
// Cliente.java
public class Cliente {
    private String nombre;
    private String correo;
    private int puntosFidelidad;

    public Cliente(String nombre, String correo) {
        this.nombre = nombre;
        this.correo = correo;
        this.puntosFidelidad = 0;
    }

    public void agregarPuntos(int puntos) {
        if (puntos > 0) this.puntosFidelidad += puntos;
    }

    public String getNombre()       { return nombre; }
    public String getCorreo()       { return correo; }
    public int getPuntosFidelidad() { return puntosFidelidad; }

    @Override
    public String toString() {
        return nombre + " (" + correo + ") - Puntos: " + puntosFidelidad;
    }
}
```

```java
// Pedido.java
import java.util.ArrayList;
import java.util.List;

public class Pedido {
    private Cliente cliente;
    private List<Bebida> bebidas;
    private String estado; // "pendiente", "preparando", "listo", "entregado"

    public Pedido(Cliente cliente) {
        this.cliente = cliente;
        this.bebidas = new ArrayList<>();
        this.estado = "pendiente";
    }

    public void agregarBebida(Bebida bebida) {
        bebidas.add(bebida);
        System.out.println("Agregado al pedido: " + bebida.getNombre());
    }

    public double calcularTotal() {
        double total = 0;
        for (Bebida b : bebidas) {
            total += b.getPrecio();
        }
        return total;
    }

    public void cambiarEstado(String nuevoEstado) {
        this.estado = nuevoEstado;
        System.out.println("Pedido de " + cliente.getNombre() +
                           " → Estado: " + nuevoEstado);
    }

    public void imprimirResumen() {
        System.out.println("=============================");
        System.out.println("PEDIDO - MICHI CAFÉ 🐱☕");
        System.out.println("=============================");
        System.out.println("Cliente: " + cliente.getNombre());
        System.out.println("Bebidas:");
        for (Bebida b : bebidas) {
            System.out.println("  - " + b);
        }
        System.out.println("Total: $" + calcularTotal());
        System.out.println("Estado: " + estado);
        System.out.println("=============================");
    }
}
```

Usando el modelo completo:

```java
public class MichiCafe {
    public static void main(String[] args) {
        // Creamos bebidas del menú
        Bebida latte    = new Bebida("Latte de vainilla", 45.50, "caliente");
        Bebida matcha   = new Bebida("Matcha Latte", 55.00, "caliente");
        Bebida coldBrew = new Bebida("Cold Brew", 50.00, "frio");

        // Creamos un cliente
        Cliente sofia = new Cliente("Sofía", "sofia@email.com");

        // Creamos un pedido
        Pedido pedido = new Pedido(sofia);
        pedido.agregarBebida(latte);
        pedido.agregarBebida(matcha);

        // Procesamos el pedido
        pedido.cambiarEstado("preparando");
        pedido.cambiarEstado("listo");
        pedido.cambiarEstado("entregado");

        // Imprimimos el resumen
        pedido.imprimirResumen();

        // Agregamos puntos al cliente
        sofia.agregarPuntos((int) pedido.calcularTotal() / 10);
        System.out.println("Puntos acumulados: " + sofia.getPuntosFidelidad());
    }
}
```

Salida:
```
Agregado al pedido: Latte de vainilla
Agregado al pedido: Matcha Latte
Pedido de Sofía → Estado: preparando
Pedido de Sofía → Estado: listo
Pedido de Sofía → Estado: entregado
=============================
PEDIDO - MICHI CAFÉ 🐱☕
=============================
Cliente: Sofía
Bebidas:
  - Latte de vainilla ($45.5) [caliente]
  - Matcha Latte ($55.0) [caliente]
Total: $100.5
Estado: entregado
=============================
Puntos acumulados: 10
```

---

## 🧪 Ejercicios del Módulo 3

### Ejercicio 1: La clase Producto

Crea una clase `Producto` para el menú del Michi Café con:
- Atributos privados: nombre, precio, categoria, disponible
- Constructor con todos los atributos
- Getters y setters con validación (precio no puede ser negativo)
- Método `mostrarInfo()` que imprima toda la información

### Ejercicio 2: Herencia de empleados

Crea la clase `Cajero` que herede de `Empleado` con:
- Atributo extra: `dineroEnCaja`
- Método `cobrar(double monto)` que sume al dinero en caja
- Método `darCambio(double pagado, double total)` que calcule el cambio
- Sobreescribe `presentarse()` con un mensaje propio

### Ejercicio 3: El turno completo

Simula un turno del Michi Café:
1. Crea 2 clientes
2. Crea un barista y un mesero
3. El mesero toma pedidos de ambos clientes
4. El barista prepara las bebidas
5. El mesero entrega los pedidos
6. Agrega puntos a los clientes según su total

---

## ✅ Resumen del Módulo 3

| Concepto | ¿Qué es? | Analogía del Michi Café |
|----------|----------|------------------------|
| Clase | Molde para crear objetos | El formulario de contratación |
| Objeto | Instancia de una clase | Un gatito específico contratado |
| Atributos | Características del objeto | Nombre, turno, especialidad |
| Métodos | Acciones del objeto | Preparar café, tomar pedido |
| Constructor | Inicializa el objeto | Llenar el formulario al contratar |
| Encapsulamiento | Proteger los datos | La caja registradora con candado |
| Herencia | Una clase basada en otra | Barista y Mesero son Empleados |
| Polimorfismo | Mismo método, diferente comportamiento | Cada gatito se presenta diferente |
| Interfaz | Contrato de métodos obligatorios | Reglas de atención al cliente |

---

## ➡️ Siguiente Módulo

En el **[Módulo 04 — Spring Boot](modulo-04-spring-boot.md)** vamos a levantar nuestro primer servidor. Cualquier persona podrá hacer una petición y recibir una respuesta.

> 🐾 "Un objeto bien diseñado es como un buen gatito: sabe lo que tiene que hacer y lo hace bien."
