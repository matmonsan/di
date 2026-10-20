# Examen — UD2: Fundamentos de programación para frontend con JavaScript

**Duración:** 120 minutos  
**Calificación:** 10 puntos  
**Número de ejercicios:** 2

## Instrucciones

- Resuelve los dos ejercicios. Cada uno vale **5 puntos** y se recomienda dedicarle **60 minutos**.
- Escribe código legible, utiliza nombres descriptivos y separa las responsabilidades en funciones y métodos.
- Salvo que se indique lo contrario, usa JavaScript moderno (ES6 o posterior).
- Controla los datos no válidos y los errores; no dejes que un fallo interrumpa la aplicación sin informar al usuario.
- En el ejercicio 2 se necesita conexión a Internet para consultar la API indicada.

---

## Ejercicio 1. Un pedido en la cafetería (5 puntos — 60 minutos)

La cafetería quiere llevar sus pedidos con una pequeña aplicación. Para empezar, trabajarás con este catálogo:

| ID | Nombre | Precio | Disponible |
|---:|---|---:|:---:|
| 1 | Café | 1,50 € | Sí |
| 2 | Té | 1,80 € | Sí |
| 3 | Zumo | 2,50 € | No |
| 4 | Tostada | 2,20 € | Sí |

Escribe el JavaScript que permita gestionar un pedido. No hace falta construir una página web completa; céntrate en que la lógica funcione.

1. **El catálogo (1 punto).** Crea una clase `Producto` con `id`, `nombre`, `precio` y `disponible`, y guarda el catálogo en un array de productos. Evita crear productos con el nombre vacío o con un precio que no sea un número finito mayor que cero: en esos casos, lanza un error.

2. **El pedido (1,5 puntos).** Crea una clase `Pedido` que guarde sus líneas. Cada línea debe tener un producto y una cantidad. Al añadir un producto, comprueba que esté disponible y que la cantidad sea un entero positivo. Si ya estaba en el pedido, suma la cantidad; si no, crea una línea nueva.

   Añade métodos para calcular el subtotal y el total. Si el subtotal llega a 20 €, aplica un descuento del 10 % y redondea el total a dos decimales. Escribe también una función que reciba el ID y la cantidad como cadenas, busque el producto y lo añada al pedido. Si algo no es válido o el producto no existe, maneja el error con `try`/`catch` e informa de lo ocurrido.

3. **Buscar y consultar el pedido (1 punto).** Crea una función para buscar productos disponibles por nombre. La búsqueda debe funcionar aunque se escriba en mayúsculas o minúsculas. Prepara un resumen del pedido con el nombre, el precio y la cantidad de cada producto. Usa un método de arrays para sumar los importes.

4. **Guardar el resumen (1 punto).** Convierte el resumen del pedido a JSON e incluye la fecha de creación en formato ISO. Escribe otra función que lea una cadena JSON y la convierta en un valor JavaScript. Si la cadena está mal formada, captura el error y muestra un mensaje claro. No hace falta reconstruir los productos como instancias de `Producto`.

5. **Una pausa para pensar (0,5 puntos).** Responde brevemente:
   - ¿Qué diferencia práctica hay entre declarar una variable con `const` y con `let`? ¿Puede modificarse un array declarado con `const`?
   - ¿Qué significa transpilar JavaScript y en qué se diferencia de compilarlo a código máquina?

Se valorará que el pedido funcione, que las comprobaciones y los errores estén bien resueltos y que el código sea claro. También cuentan las respuestas conceptuales.

---

## Ejercicio 2. Publicaciones en una página web (5 puntos — 60 minutos)

Ahora vas a crear una página sencilla para consultar publicaciones de Internet y buscar por título. Cada publicación recibida tiene las propiedades `userId`, `id`, `title` y `body`.

La página consultará esta dirección:

```text
https://jsonplaceholder.typicode.com/posts?_limit=20
```

Usa este HTML mínimo como punto de partida. Puedes añadir lo que necesites, pero conserva los identificadores para conectarlos desde JavaScript:

```html
<!DOCTYPE html>
<html lang="es">
  <head>
    <meta charset="UTF-8">
    <title>Publicaciones</title>
    <script src="app.js" defer></script>
  </head>
  <body>
    <h1>Publicaciones</h1>
    <form id="formulario-busqueda">
      <input id="busqueda" type="search" placeholder="Buscar por título">
      <button type="submit">Buscar</button>
    </form>
    <p id="estado"></p>
    <ul id="resultados"></ul>
  </body>
</html>
```

Completa `app.js` para que la página haga lo siguiente:

1. **Traer las publicaciones (1,5 puntos).** Cuando se abra la página, pide los datos con `fetch` y `async`/`await`. Comprueba que la respuesta ha ido bien antes de leer el JSON y guarda las publicaciones en un array.

2. **Contar qué está pasando (1 punto).** Mientras llegan los datos, muestra un mensaje de carga. Si algo falla, informa del problema en la página. Cuando termine la petición, actualiza el mensaje tanto si salió bien como si hubo un error.

3. **Mostrar los resultados (1,25 puntos).** Añade cada publicación a la lista con su título y su texto. Crea los elementos desde JavaScript. Si no hay publicaciones que mostrar, indícalo en la página.

4. **Buscar por título (1 punto).** Al enviar el formulario, evita que la página se recargue y muestra solo las publicaciones cuyo título contenga el texto escrito. La búsqueda debe ignorar mayúsculas y minúsculas. Si el campo está vacío, vuelve a mostrar todas las publicaciones.

5. **Código ordenado (0,25 puntos).** Organiza la carga, la búsqueda y la actualización de la lista en funciones con nombres claros. Evita repetir el mismo código.

Se valorará que la página responda correctamente, que los errores se comuniquen y que uses bien `fetch`, las promesas, los eventos, los arrays y el DOM.
