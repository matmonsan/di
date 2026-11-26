---
title: Simulacro de examen UD4. Programación de componentes en React
description: "<strong>Módulo:</strong> Desarrollo de Interfaces <br> <strong>Profesor:</strong> Matías Montávez Sánchez"
---
{% raw %}

[⌂ Volver al inicio](index.md)

## Instrucciones

- Duración aproximada: 1 hora y 45 minutos.
- Crea un proyecto de Vite llamado `examen-ud4` y entrégalo sin la carpeta `node_modules`.

---

## Ejercicio. El tiempo en tu ciudad

Crea con componentes de React un panel del tiempo que se vea como en la imagen:

![Panel del tiempo: cabecera verde oliva con el nombre de la ciudad y un botón Actualizar; bloque Ahora con la temperatura grande y sus datos; previsión de 7 días en fila, con los días de lluvia en verde claro y el sábado marcado como máxima de la semana; media de las máximas; lista de dos avisos con un borde de color; y pie de página](./assets/img/examen-ud4.png)

Copia estos datos en `src/data/tiempo.js`:

```js
export const actual = { temperatura: 18, sensacion: 16, estado: "Parcialmente nublado", humedad: 55, viento: 12 };

export const prevision = [
  { dia: "Lunes", max: 21, min: 9, lluvia: 10 },
  { dia: "Martes", max: 19, min: 10, lluvia: 60 },
  { dia: "Miércoles", max: 16, min: 8, lluvia: 80 },
  { dia: "Jueves", max: 18, min: 7, lluvia: 30 },
  { dia: "Viernes", max: 23, min: 11, lluvia: 0 },
  { dia: "Sábado", max: 25, min: 12, lluvia: 0 },
  { dia: "Domingo", max: 22, min: 11, lluvia: 20 },
];

export const avisos = [
  { id: 1, nivel: "amarillo", texto: "Lluvias de hasta 15 mm el miércoles" },
  { id: 2, nivel: "naranja", texto: "Rachas de viento de hasta 70 km/h el martes" },
];
```

### Requisitos

1. Crea los componentes necesarios, cada uno en su archivo dentro de `src/components/` y con su propio `.module.css`. La cabecera recibe el nombre de la ciudad por props.
2. Define en `index.css` variables para los colores y el espaciado (fondo, texto, color principal, borde y espacio) y úsalas en los módulos.
3. En la cabecera, el título y el botón están en los extremos (`display: flex` con `justify-content: space-between`). Al pulsar «Actualizar» se muestra un `alert` con el texto «Datos actualizados».
4. El bloque «Ahora» muestra la temperatura en grande a la izquierda y el resto de datos en una lista a la derecha.
5. La previsión muestra los 7 días en una fila, todos del mismo ancho (`display: flex` y `flex: 1` en cada día). Cada día muestra:
   - Las tres primeras letras de su nombre («Lun», «Mar»...), obtenidas del dato con `slice`.
   - La máxima, la mínima en gris y la probabilidad de lluvia.
   - Fondo verde claro si la probabilidad de lluvia es del 50 % o más, con una clase del módulo.
   - El texto «Máx. semana» solo en el día con la máxima más alta. Ese día se calcula a partir de los datos.
6. Al pulsar un día se muestra un `alert` con su nombre completo y sus datos: «Martes: máxima 19 °C, mínima 10 °C, 60 % de lluvia».
7. Debajo de los días se muestra la media de las máximas con un decimal.
8. El título de los avisos muestra cuántos hay: «Avisos (2)». Cada aviso tiene un borde izquierdo de color según su nivel, con una clase del módulo para cada nivel. Si no hay avisos, se muestra «No hay avisos».

Se valorará que la página se vea como en la imagen, la división en componentes, el uso de props, listas y condicionales, y que los estilos estén en módulos.

{% endraw %}
