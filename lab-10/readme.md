
# Architecture and Optimization
## Code Optimization Practice

### Objective:
Enhance the performance of provided code snippets by applying code-level optimization techniques manually. This lab focuses on deepening your understanding and application of optimization principles in coding.

### Introduction:
At the beginning of the lab, you will receive a set of code snippets that contain common performance issues. These snippets will come from different programming languages, including JavaScript, Java, and C#, to cover a variety of scenarios and common inefficiencies found in real-world programming.

### JavaScript Snippet:

```javascript
// Inefficient loop handling and excessive DOM manipulation
function updateList(items) {
  let list = document.getElementById("itemList");
  list.innerHTML = "";
  for (let i = 0; i < items.length; i++) {
    let listItem = document.createElement("li");
    listItem.innerHTML = items[i];
    list.appendChild(listItem);
  }
}
```
---
### Copdigo optimizado:

```javascript
function updateList(items) {
  const list = document.getElementById("itemList");

  // Crear un fragmento para evitar múltiples manipulaciones del DOM
  const fragment = document.createDocumentFragment();

  // Generar los nuevos elementos de la lista
  items.forEach(item => {
    const listItem = document.createElement("li");
    listItem.textContent = item; // Más seguro que innerHTML
    fragment.appendChild(listItem);
  });

  // Limpiar la lista y añadir los nuevos elementos de una sola vez
  list.innerHTML = ""; // Limpia eficientemente el contenido existente
  list.appendChild(fragment); // Actualiza el DOM en una única operación
}
```
- Se utilza createDocumentFragment para evitar manipulaciones excesivas del DOM, lo que mejora el rendimiento.
- En lugar de hacer uso de un bucle `for`, se utiliza el método `forEach` para iterar sobre los elementos del array `items`. Esto es más legible y menos propenso a errores.
- Se utiliza `textContent` en lugar de `innerHTML` para evitar la interpretación de HTML y mejorar la seguridad ya que evita la inyección de código malicioso.

### Java Snippet:

```java 
// Redundant database queries
public class ProductLoader {
    public List<Product> loadProducts() {
        List<Product> products = new ArrayList<>();
        for (int id = 1; id <= 100; id++) {
            products.add(database.getProductById(id));
        }
        return products;
    }
}
```

- Un serio problema con el codigo original es que se realizan 100 consultas a la base de datos, una por cada producto. Esto puede ser muy ineficiente y lento, especialmente si la base de datos está en una red remota.
- Esta totalmente acoplado a la implementación de la base de datos, lo que dificulta la prueba y la evolución del código.
- No maneja adecuadamente los errores que puedan ocurrir durante la carga de los productos.
- No se valida si la lista de IDs está vacía o nula.
- No se utiliza la inyección de dependencias para proporcionar flexibilidad y facilitar las pruebas.
- No se utiliza un enfoque de programación funcional para simplificar el código y hacerlo más legible.
---
### Código optimizado:

```java
public class ProductLoader {

    private final ProductRepository productRepository;

    // Inyección de dependencia para mayor flexibilidad y testabilidad
    public ProductLoader(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }

    public List<Product> loadProducts(List<Integer> ids) {
        if (ids == null || ids.isEmpty()) { // Validación de argumentos
            throw new IllegalArgumentException("La lista de IDs no puede estar vacía"); // Lanzar excepción
        }

        try {
            // Obtener productos en una sola consulta optimizada utilizando jpa
            return productRepository.findProductsByIds(ids);
        } catch (Exception e) {
            // Manejar errores adecuadamente
            throw new RuntimeException("Error al cargar productos: " + e.getMessage(), e); // Lanzar excepción personalizada
        }
    }

    public interface ProductRepository extends JpaRepository<Product, Integer> { // Interfaz para abstraer la base de datos

        List<Product> findProductsByIds(List<Integer> ids);// Método para buscar productos por IDs
    }
}
```
- Se utiliza la inyección de dependencias para desacoplar la clase `ProductLoader` de la implementación concreta de la base de datos. Esto facilita las pruebas unitarias y permite cambiar la implementación de la base de datos sin modificar el código de `ProductLoader`.
- Se cambia el método `loadProducts` para aceptar una lista de IDs como argumento. Esto permite cargar productos específicos en lugar de todos los productos en un rango fijo.
- Se valida si la lista de IDs es nula o está vacía y se lanza una excepción si es así. Esto evita errores inesperados y mejora la robustez del código.
- Se manejan los errores adecuadamente utilizando un bloque `try-catch` para capturar excepciones y lanzar una excepción personalizada en caso de error.
- Se introduce una interfaz `ProductRepository` para abstraer la base de datos y proporcionar un punto de extensión para diferentes implementaciones de la base de datos.
- Se añade un método `findProductsByIds` en la interfaz `ProductRepository` para buscar productos por una lista de IDs. Esto permite realizar una consulta optimizada para cargar los productos en una sola operación.

Alternativa, si necesito un rango fijo de ids:

```java
import java.util.List;

public List<Product> loadProducts(int startId, int endId) {
    if (startId > endId) {
        throw new IllegalArgumentException("El ID de inicio no puede ser mayor que el ID final");
    }

    List<Integer> rangeIds = IntStream.rangeClosed(startId, endId)
            .boxed() // Convertir a Stream<Integer>
            .collect(Collectors.toList()); // Generar la lista de IDs

    try {
        // Obtener productos en una sola consulta optimizada utilizando jpa
        return productRepository.findProductsByIds(rangeIds);
    } catch (Exception e) {
        // Manejar errores adecuadamente
        throw new RuntimeException("Error al cargar productos: " + e.getMessage(), e); // Lanzar excepción personalizada
    }
}
```
- Se añade un nuevo método `loadProducts` que acepta un rango de IDs (inicio y fin) como argumentos. Esto permite cargar productos en un rango específico de IDs en lugar de todos los productos en un rango fijo.
- Se valida si el ID de inicio es mayor que el ID final y se lanza una excepción si es así. Esto evita errores lógicos y mejora la robustez del código.
- Se genera una lista de IDs en el rango especificado utilizando `IntStream.rangeClosed`, `boxed` y `collect(Collectors.toList)`. Esto simplifica la generación de la lista de IDs y mejora la legibilidad del código.
- Se utiliza el método `findProductsByIds` de la interfaz `ProductRepository` para buscar productos por una lista de IDs en un rango específico. Esto permite realizar una consulta optimizada para cargar los productos en una sola operación.

### C# Snippet:

```csharp
// Unnecessary computations in data processing
public List<int> ProcessData(List<int> data) {
    List<int> result = new List<int>();
    foreach (var d in data) {
        if (d % 2 == 0) {
            result.Add(d * 2);
        } else {
            result.Add(d * 3);
        }
    }
    return result;
}
```
- El código original realiza cálculos innecesarios al procesar los datos. Multiplica los números pares por 2 y los números impares por 3, lo que puede ser ineficiente y redundante.
- No maneja adecuadamente los errores que puedan ocurrir durante el procesamiento de los datos.

---
### Código optimizado:

```csharp
public List<int> ProcessData(List<int> data)
{
    return data.Select(d => d * (d % 2 == 0 ? 2 : 3)).ToList();
}
```
- Se utiliza LINQ (Language Integrated Query) para simplificar el procesamiento de los datos. La expresión lambda `d => d * (d % 2 == 0 ? 2 : 3)` realiza el cálculo necesario en una sola línea de código.
- Se evita el uso de un bucle `foreach` y se utiliza el método `Select` para aplicar la transformación a cada elemento de la lista `data`.
- Se utiliza el método `ToList` para convertir el resultado en una lista de enteros y devolverla como resultado de la función.
- Se elimina la necesidad de un bloque `if-else` y se realiza el cálculo de forma más concisa y legible utilizando el operador condicional ternario `? :`.
