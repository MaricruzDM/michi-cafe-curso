# Módulo 02 — Java desde Cero
## 🐱 Analogía: Los gatitos aprenden a hablar

---

## 🏠 Introducción

En el módulo anterior conocimos el Michi Café y entendimos para qué sirve cada herramienta.
Ahora es momento de aprender el idioma: **Java**.

Imagina que contrataste a varios gatitos para trabajar en el café, pero ninguno sabe hablar todavía.
Este módulo es la **escuela de gatitos**: aquí aprenderán a comunicarse, recordar cosas, tomar decisiones y repetir tareas.

Al final de este módulo podrás escribir tus primeros programas reales en Java.

---

## 💡 ¿Cómo funciona un programa en Java?

Antes de escribir código, entendamos el flujo básico:

1. Tú escribes instrucciones en un archivo `.java`
2. Java las **compila** (las traduce a un idioma que la computadora entiende)
3. La computadora **ejecuta** esas instrucciones

### Analogía 🐾

Es como escribir una receta:
- Tú escribes la receta en papel (archivo `.java`)
- Un chef la lee y la interpreta (compilación)
- La cocina prepara el platillo (ejecución)

---

## 💻 Tu primer programa: Hola Michi Café

Todo programa en Java empieza con una estructura básica.

```java
public class MichiCafe {
    public static void main(String[] args) {
        System.out.println("¡Bienvenido al Michi Café! 🐱☕");
    }
}
```

### ¿Qué significa cada parte?

| Parte | Significado | Analogía del café |
|-------|-------------|-------------------|
| `public class MichiCafe` | Define el contenedor del programa | El nombre del café en el letrero |
| `public static void main` | El punto de inicio del programa | La puerta principal del café |
| `System.out.println(...)` | Muestra un mensaje en pantalla | El gatito mesero habla en voz alta |

> 💡 En Java, cada instrucción termina con `;`. Es como el punto al final de una oración.

---

## 💡 Variables: la libretita del gatito

Una **variable** es un espacio en memoria donde guardamos información.

### Analogía 🐾

El gatito mesero tiene una libretita. En ella anota:
- El nombre del cliente
- La cantidad de cafés pedidos
- Si el cliente quiere leche o no

Cada anotación es una **variable**.

### Cómo crear variables en Java

```java
// Sintaxis: tipo nombreVariable = valor;

String nombreCliente = "Sofía";
int cantidadCafes = 2;
double precio = 45.50;
boolean quiereLeche = true;
```

### Los tipos de datos básicos

| Tipo | ¿Para qué sirve? | Ejemplo | Analogía del café |
|------|-----------------|---------|-------------------|
| `String` | Texto | `"Latte de vainilla"` | Nombre de un producto |
| `int` | Números enteros | `3` | Cantidad de cafés |
| `double` | Números con decimales | `45.50` | Precio de un café |
| `boolean` | Verdadero o falso | `true` / `false` | ¿Tiene azúcar? |
| `char` | Un solo carácter | `'M'` | Inicial del nombre |

### Ejemplo completo

```java
public class PedidoCafe {
    public static void main(String[] args) {
        String nombreCliente = "Sofía";
        String bebida = "Latte de vainilla";
        int cantidad = 2;
        double precioPorUnidad = 45.50;
        boolean paraLlevar = true;

        double total = cantidad * precioPorUnidad;

        System.out.println("=== PEDIDO MICHI CAFÉ ===");
        System.out.println("Cliente: " + nombreCliente);
        System.out.println("Bebida: " + bebida);
        System.out.println("Cantidad: " + cantidad);
        System.out.println("Para llevar: " + paraLlevar);
        System.out.println("Total: $" + total);
    }
}
```

Salida:
```
=== PEDIDO MICHI CAFÉ ===
Cliente: Sofía
Bebida: Latte de vainilla
Cantidad: 2
Para llevar: true
Total: $91.0
```

> 💡 El `+` entre texto y variables une el texto con el valor de la variable.

---

## 💡 Operadores: el gatito cajero hace cuentas

```java
int cafes = 5;
int pastelitos = 3;

int total       = cafes + pastelitos;  // Suma:           8
int diferencia  = cafes - pastelitos;  // Resta:          2
int doble       = cafes * 2;           // Multiplicación: 10
double mitad    = cafes / 2.0;         // División:       2.5
int sobrante    = cafes % 2;           // Módulo:         1
```

### Operadores de comparación

Comparan dos valores y devuelven `true` o `false`:

```java
int edad = 20;

System.out.println(edad > 18);   // true
System.out.println(edad < 18);   // false
System.out.println(edad == 20);  // true  (doble igual para comparar)
System.out.println(edad != 20);  // false
System.out.println(edad >= 20);  // true
```

> ⚠️ Para comparar usamos `==`. El `=` solo asigna valores.

---

## 💡 Condicionales: el gatito toma decisiones

### Analogía 🐾

El gatito mesero tiene una regla:
- Si el cliente tiene tarjeta de fidelidad → 10% de descuento
- Si no → precio normal

Eso es un `if` en Java.

```java
if (condición) {
    // Se ejecuta si la condición es VERDADERA
} else {
    // Se ejecuta si la condición es FALSA
}
```

### Ejemplo en el Michi Café

```java
public class DescuentoCliente {
    public static void main(String[] args) {
        String nombreCliente = "Sofía";
        boolean tieneTarjeta = true;
        double total = 91.0;

        if (tieneTarjeta) {
            double descuento = total * 0.10;
            total = total - descuento;
            System.out.println("¡Hola " + nombreCliente + "! Tienes 10% de descuento 🐾");
        } else {
            System.out.println("Hola " + nombreCliente + ", ¿quieres una tarjeta de fidelidad?");
        }

        System.out.println("Total a pagar: $" + total);
    }
}
```

### `else if`: múltiples condiciones

```java
int puntos = 150;
String recompensa;

if (puntos >= 200) {
    recompensa = "Café gratis";
} else if (puntos >= 100) {
    recompensa = "Pastelito gratis";
} else if (puntos >= 50) {
    recompensa = "10% de descuento";
} else {
    recompensa = "¡Sigue acumulando puntos!";
}

System.out.println("Tu recompensa: " + recompensa);
// Resultado: Tu recompensa: Pastelito gratis
```

### `switch`: el menú de opciones

Cuando hay muchas opciones posibles, `switch` es más limpio:

```java
String bebida = "Cappuccino";
double precio;

switch (bebida) {
    case "Espresso":
        precio = 35.0;
        break;
    case "Latte":
        precio = 45.0;
        break;
    case "Cappuccino":
        precio = 48.0;
        break;
    case "Matcha Latte":
        precio = 55.0;
        break;
    default:
        precio = 0.0;
        System.out.println("Bebida no encontrada en el menú");
        break;
}

System.out.println("Precio: $" + precio);
// Resultado: Precio: $48.0
```

> 💡 El `break` le dice al switch "ya encontré lo que buscaba, para aquí".


---

## 💡 Ciclos: el gatito repite tareas

Un ciclo (o bucle) permite repetir instrucciones varias veces sin tener que escribirlas una y una.

### Analogía 🐾

El gatito de la caja tiene que imprimir el ticket de cada cliente. Si hay 100 clientes, no va a escribir 100 veces "imprimir ticket". Le dice a la computadora:
> "Repite esto 100 veces"

### El ciclo `for`: cuando sabes cuántas veces repetir

```java
// Sintaxis:
// for (inicio; condición; incremento) { ... }

for (int i = 1; i <= 5; i++) {
    System.out.println("Preparando café número " + i + " ☕");
}
```

Salida:
```
Preparando café número 1 ☕
Preparando café número 2 ☕
Preparando café número 3 ☕
Preparando café número 4 ☕
Preparando café número 5 ☕
```

### ¿Qué significa `int i = 1; i <= 5; i++`?

| Parte | Significado |
|-------|-------------|
| `int i = 1` | Empieza contando desde 1 |
| `i <= 5` | Sigue mientras i sea menor o igual a 5 |
| `i++` | Después de cada vuelta, suma 1 a i |

### El ciclo `while`: cuando no sabes cuántas veces repetir

```java
// El gatito sigue tomando pedidos mientras el café esté abierto

boolean cafeAbierto = true;
int pedidosAtendidos = 0;

while (cafeAbierto) {
    pedidosAtendidos++;
    System.out.println("Pedido #" + pedidosAtendidos + " atendido");

    if (pedidosAtendidos == 3) {
        cafeAbierto = false; // Cerramos después de 3 pedidos (para este ejemplo)
        System.out.println("El Michi Café cerró por hoy 🌙");
    }
}
```

Salida:
```
Pedido #1 atendido
Pedido #2 atendido
Pedido #3 atendido
El Michi Café cerró por hoy 🌙
```

> ⚠️ Cuidado con los ciclos infinitos: si la condición del `while` nunca se vuelve `false`, el programa nunca termina. Es como decirle al gatito "sigue trabajando" sin decirle cuándo parar.

### El ciclo `do-while`: ejecuta al menos una vez

```java
// El gatito pregunta al menos una vez si el cliente quiere algo más

int extras = 0;

do {
    System.out.println("¿Desea algo más? (simulando respuesta automática)");
    extras++;
} while (extras < 1); // Solo pregunta una vez en este ejemplo

System.out.println("Gracias por su visita al Michi Café 🐾");
```

---

## 💡 Arrays: la lista de productos

Un **array** es una lista de valores del mismo tipo guardados juntos.

### Analogía 🐾

El menú del Michi Café es una lista de bebidas. En lugar de crear una variable para cada bebida, creamos un array (una lista).

```java
// Crear un array de Strings (textos)
String[] menu = {"Espresso", "Latte", "Cappuccino", "Matcha Latte", "Chai"};

// Acceder a un elemento (los índices empiezan en 0)
System.out.println(menu[0]); // Espresso
System.out.println(menu[1]); // Latte
System.out.println(menu[4]); // Chai

// Saber cuántos elementos tiene
System.out.println("El menú tiene " + menu.length + " bebidas");
```

> 💡 Los índices en Java empiezan en **0**, no en 1. El primer elemento es `[0]`, el segundo es `[1]`, etc.

### Recorrer un array con `for`

```java
String[] menu = {"Espresso", "Latte", "Cappuccino", "Matcha Latte", "Chai"};

System.out.println("=== MENÚ DEL MICHI CAFÉ ===");
for (int i = 0; i < menu.length; i++) {
    System.out.println((i + 1) + ". " + menu[i]);
}
```

Salida:
```
=== MENÚ DEL MICHI CAFÉ ===
1. Espresso
2. Latte
3. Cappuccino
4. Matcha Latte
5. Chai
```

### El `for-each`: forma más elegante de recorrer listas

```java
String[] menu = {"Espresso", "Latte", "Cappuccino", "Matcha Latte", "Chai"};

for (String bebida : menu) {
    System.out.println("☕ " + bebida);
}
```

Esto se lee como: "Para cada `bebida` en `menu`, haz esto..."

---

## 💡 Métodos: las habilidades del gatito

Un **método** es un bloque de código con nombre que hace una tarea específica. Puedes llamarlo (usarlo) cuando lo necesites.

### Analogía 🐾

El gatito barista sabe preparar café. Esa habilidad se llama `prepararCafe()`. Cada vez que alguien pide un café, el gatito usa esa habilidad. No tiene que aprender a hacerlo de nuevo cada vez.

### Crear y usar un método

```java
public class MichiBarista {

    // Este método calcula el total de un pedido
    public static double calcularTotal(int cantidad, double precioPorUnidad) {
        double total = cantidad * precioPorUnidad;
        return total;
    }

    // Este método imprime un recibo
    public static void imprimirRecibo(String cliente, String bebida, double total) {
        System.out.println("=============================");
        System.out.println("  RECIBO - MICHI CAFÉ 🐱☕  ");
        System.out.println("=============================");
        System.out.println("Cliente: " + cliente);
        System.out.println("Bebida:  " + bebida);
        System.out.println("Total:   $" + total);
        System.out.println("=============================");
        System.out.println("¡Gracias! Vuelve pronto 🐾");
    }

    public static void main(String[] args) {
        // Usamos los métodos que creamos
        double total = calcularTotal(2, 45.50);
        imprimirRecibo("Sofía", "Latte de vainilla", total);
    }
}
```

Salida:
```
=============================
  RECIBO - MICHI CAFÉ 🐱☕  
=============================
Cliente: Sofía
Bebida:  Latte de vainilla
Total:   $91.0
=============================
¡Gracias! Vuelve pronto 🐾
```

### Anatomía de un método

```java
public static double calcularTotal(int cantidad, double precio) {
//  ↑       ↑      ↑               ↑                ↑
// acceso  tipo  nombre         parámetro 1      parámetro 2
// (quién  de    del            (lo que recibe)
//  puede  retorno método
//  usarlo)
    return cantidad * precio;
//  ↑
//  devuelve el resultado
}
```

| Parte | Significado | Analogía |
|-------|-------------|----------|
| `public` | Cualquiera puede usar este método | La habilidad es pública, todos la conocen |
| `static` | No necesita crear un objeto para usarlo | El gatito puede hacerlo solo |
| `double` | El tipo de dato que devuelve | El resultado es un número con decimales |
| `calcularTotal` | El nombre del método | El nombre de la habilidad |
| `(int cantidad, double precio)` | Los datos que necesita para trabajar | Los ingredientes que necesita |
| `return` | Lo que devuelve al terminar | El resultado que entrega |

---

## 💡 Comentarios: las notas del gatito

Los comentarios son texto en el código que Java ignora. Sirven para explicar qué hace el código.

```java
// Esto es un comentario de una línea

/*
   Esto es un comentario
   de varias líneas
*/

/**
 * Este es un comentario de documentación (Javadoc)
 * Se usa para explicar métodos y clases
 * @param cantidad el número de bebidas
 * @return el precio total
 */
public static double calcularTotal(int cantidad, double precio) {
    return cantidad * precio;
}
```

> 💡 Comentar tu código es como dejar notas en la receta del café. Cuando alguien más (o tú mismo en el futuro) lea el código, entenderá qué hace cada parte.

---

## 🧪 Ejercicios del Módulo 2

### Ejercicio 1: El pedido completo

Crea un programa que:
1. Guarde el nombre del cliente, la bebida pedida, la cantidad y el precio unitario
2. Calcule el total
3. Si el total es mayor a $100, aplique un 5% de descuento
4. Imprima el recibo completo

### Ejercicio 2: El menú del día

Crea un array con 5 bebidas del Michi Café y sus precios.
Recorre el array e imprime el menú numerado.

### Ejercicio 3: El contador de pedidos

Usa un ciclo `while` que simule la atención de pedidos.
El café atiende pedidos hasta que se acaben los ingredientes (simula con un contador que llegue a 10).

### Solución del Ejercicio 1

```java
public class EjercicioUno {
    public static void main(String[] args) {
        // Datos del pedido
        String cliente = "Carlos";
        String bebida = "Matcha Latte";
        int cantidad = 3;
        double precioUnitario = 55.0;

        // Calcular total
        double total = cantidad * precioUnitario;

        // Aplicar descuento si aplica
        if (total > 100) {
            double descuento = total * 0.05;
            total = total - descuento;
            System.out.println("Descuento del 5% aplicado 🎉");
        }

        // Imprimir recibo
        System.out.println("=== RECIBO MICHI CAFÉ ===");
        System.out.println("Cliente:  " + cliente);
        System.out.println("Bebida:   " + bebida);
        System.out.println("Cantidad: " + cantidad);
        System.out.println("Total:    $" + total);
    }
}
```

---

## ✅ Resumen del Módulo 2

| Concepto | ¿Qué es? | Analogía del Michi Café |
|----------|----------|------------------------|
| Variable | Espacio para guardar un dato | Anotación en la libretita del mesero |
| Tipos de datos | El tipo de información que guardamos | Texto, número, precio, sí/no |
| Operadores | Operaciones matemáticas y comparaciones | Las cuentas del cajero |
| `if / else` | Tomar decisiones según una condición | Aplicar o no el descuento |
| `switch` | Elegir entre muchas opciones | Buscar el precio en el menú |
| `for` | Repetir un número conocido de veces | Preparar N cafés |
| `while` | Repetir mientras algo sea verdad | Atender mientras el café esté abierto |
| Array | Lista de valores del mismo tipo | El menú de bebidas |
| Método | Bloque de código reutilizable | Una habilidad del gatito |

---

## ➡️ Siguiente Módulo

En el **Módulo 03** aprenderemos **Programación Orientada a Objetos (POO)**: cómo crear "moldes" para nuestros gatitos y objetos del café. Aprenderemos clases, objetos, atributos y métodos de una forma que tiene mucho sentido con la analogía del Michi Café.

> 🐾 "Un gatito que sabe Java es un gatito imparable."
