---
title: UD3. Desarrollo de interfaces naturales
description: "<strong>Módulo:</strong> Desarrollo de Interfaces <br> <strong>Profesor:</strong> Matías Montávez Sánchez"
---

[⌂ Volver al inicio](index.md)

# Índice

- [1. Introducción a las interfaces naturales de usuario](#1-introducción-a-las-interfaces-naturales-de-usuario)
- [2. Tecnologías para desarrollar interfaces naturales](#2-tecnologías-para-desarrollar-interfaces-naturales)
- [3. Reconocimiento de voz con Web Speech API](#3-reconocimiento-de-voz-con-web-speech-api)
- [4. Detección de manos y rostro](#4-detección-de-manos-y-rostro)
- [5. Detección de pose corporal y control por movimiento](#5-detección-de-pose-corporal-y-control-por-movimiento)
- [6. Realidad aumentada en el navegador (WebAR)](#6-realidad-aumentada-en-el-navegador-webar)
- [7. Caso práctico integrador: kiosco multimodal e interactivo NUI](#7-caso-práctico-integrador-kiosco-multimodal-e-interactivo-nui)
- [8. Criterios y rúbrica de evaluación](#8-criterios-y-rúbrica-de-evaluación)

### 1. Introducción a las interfaces naturales de usuario

![Interfaces de usuario naturales](./assets/img/naturales.png)

Las **Interfaces Naturales de Usuario (NUI - Natural User Interfaces)** representan un paradigma de interacción persona-ordenador donde los medios tradicionales (teclado y ratón) son sustituidos o complementados por habilidades humanas innatas: la voz, la gestualidad corporal, la expresión facial y la percepción espacial mediante Realidad Aumentada (RA).

En esta unidad se abordará el desarrollo de INUs web utilizando estándares modernos de JavaScript (Web APIs nativas como Web Speech API y WebRTC) junto con modelos de Machine Learning preentrenados en el cliente (MediaPipe / TensorFlow.js) y frameworks de Realidad Aumentada Web (MindAR / A-Frame).

A diferencia de las **GUI** (Interfaces Gráficas basadas en ventanas e íconos) o las **CUI** (Interfaces de Línea de Comandos), las **NUI** buscan reducir la brecha cognitiva de aprendizaje. El usuario interactúa a través de conductas cotidianas:
- **Voz:** Comandos orales, dictado, análisis semántico.
- **Gesto y Postura:** Control gestual con las manos, inclinación de cabeza, postura corporal completa.
- **Rostro:** Reconocimiento de expresiones faciales o dirección de la mirada (*eye tracking*).
- **Entorno Aumentado (RA):** Inserción de capas de información y objetos 3D sobre la emisión de video en tiempo real.

### 2. Tecnologías para desarrollar interfaces naturales
Para implementar NUI en el navegador moderno sin necesidad de plugins externos, empleamos:

1. **APIs Nativas del Navegador:**
   - `navigator.mediaDevices.getUserMedia()`: Acceso a cámaras y micrófonos.
   - `Web Speech API`: Reconocimiento de voz (`SpeechRecognition`) y síntesis de voz (`SpeechSynthesis`).
   - `WebXR Device API`: Interfaces de Realidad Virtual y Aumentada.
2. **Bibliotecas de Machine Learning Ejecutadas en Cliente (WebAssembly / WebGL):**
   - **MediaPipe (Google):** Detección ultra-rápida de landmarks corporales (manos, rostro, pose completa).
   - **TensorFlow.js:** Inferencia de modelos de visión por computador y clasificación personalizada.
3. **Frameworks de Realidad Aumentada y 3D:**
   - **A-Frame:** Framework declarativo basado en Custom Elements de HTML para WebVR/WebAR.
   - **MindAR.js:** Librería ligera de visión por ordenador para seguimiento de imágenes (*image tracking*) y rostros (*face tracking*).

### 3. Reconocimiento de voz con Web Speech API

![Web Speech API](./assets/img/audio.png){: width="300"}

La **Web Speech API** es una interfaz del navegador para incorporar voz a una aplicación web. Se compone de dos partes:

- **SpeechRecognition:** convierte el habla en texto. La aplicación puede analizar ese texto y utilizarlo para ejecutar acciones, como cambiar el contenido o el aspecto de la página.
- **SpeechSynthesis:** convierte texto en voz y permite que la aplicación responda oralmente al usuario.

#### 3.1. Componentes y flujo del reconocimiento

El reconocimiento de voz sigue un flujo parecido al de otros eventos del navegador:

1. Se crea un objeto `SpeechRecognition`.
2. Se configura el idioma y el tipo de resultados que se quieren recibir.
3. Tras una acción del usuario (por ejemplo, pulsar un botón), se llama a `start()` y el navegador comienza a escuchar.
4. El servicio de reconocimiento procesa el audio y emite eventos. Si reconoce palabras, las comunica mediante `result`; si ocurre un problema, lo hace mediante `error`.
5. La aplicación interpreta el texto y actualiza la interfaz. Al detenerse el reconocimiento, se produce `end`.

**Ejemplo.** Imaginemos una página con un botón «Hablar» y un párrafo donde se muestra lo que el usuario dice:

```js
const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;

// 1. Se crea el objeto
const reconocimiento = new SpeechRecognition();

// 2. Se configura el idioma y el tipo de resultados
reconocimiento.lang = "es-ES";
reconocimiento.interimResults = false; // solo resultados finales
reconocimiento.maxAlternatives = 1;

// 3. Una acción del usuario inicia la escucha
document.querySelector("#hablar").addEventListener("click", () => {
  reconocimiento.start();
});

// 4. El servicio emite eventos
reconocimiento.addEventListener("result", (e) => {
  // 5. La aplicación interpreta el texto y actualiza la interfaz
  const texto = e.results[0][0].transcript.toLowerCase();
  document.querySelector("#salida").textContent = `Has dicho: ${texto}`;
  if (texto.includes("azul")) document.body.style.background = "blue";
});

reconocimiento.addEventListener("error", (e) => {
  document.querySelector("#salida").textContent = `Error: ${e.error}`;
});

reconocimiento.addEventListener("end", () => {
  console.log("El reconocimiento ha terminado");
});
```

Secuencia: el usuario pulsa «Hablar» (se ejecuta `start()` y el navegador pide permiso de micrófono) y dice «pon la página en azul». El servicio procesa el audio y lanza `result` con la transcripción «pon la página en azul». La aplicación busca la palabra «azul» y cambia el fondo. Como `continuous` es `false` por defecto, el reconocimiento se detiene solo tras la frase y se dispara `end`. Si el usuario deniega el micrófono o no se oye nada, se lanza `error` (por ejemplo, `not-allowed` o `no-speech`) en lugar de `result`.

**¿Qué es `maxAlternatives`?** Indica cuántas transcripciones posibles devuelve el reconocedor para cada frase. El audio puede ser ambiguo y el motor puede proponer varias interpretaciones, ordenadas de más a menos probable (por ejemplo, «bola» y «bota»). Con `1` solo se recibe la mejor, que es el valor por defecto, por lo que esa línea es opcional.

**¿Por qué `results[0][0]` tiene dos índices?** Porque `e.results` es una lista de resultados y cada resultado es, a su vez, una lista de alternativas:

- Primer índice (`results[i]`): cada fragmento de habla reconocido. Con `continuous = false` suele haber uno solo; con `continuous = true`, uno por frase o pausa.
- Segundo índice (`results[i][j]`): cada alternativa de ese fragmento. La posición `0` es la más probable y llegan tantas como indique `maxAlternatives`.

Cada alternativa es un objeto con `transcript` (el texto) y `confidence` (valor entre 0 y 1). Con `maxAlternatives = 3`:

```js
e.results[0][0].transcript  // "pon la página en azul"  (confidence 0.92)
e.results[0][1].transcript  // "pon la pagina en azul"  (confidence 0.05)
e.results[0][2].transcript  // "pon la página en as ul" (confidence 0.01)
```

Así, `results[0][0]` significa «la mejor alternativa del primer resultado».

El objeto de reconocimiento no es el micrófono ni un traductor de intenciones: entrega una o más transcripciones posibles del audio. Es la aplicación la que debe decidir qué significa una frase y qué acciones permite ejecutar. Por ejemplo, «pon la página en azul» podría analizarse para identificar la intención «cambiar color» y el valor «azul». El ejemplo de esta unidad utiliza una lógica más sencilla: busca palabras concretas dentro de la transcripción.

El constructor y algunas propiedades habituales son:

| Elemento | Función |
| :--- | :--- |
| `new SpeechRecognition()` | Crea el controlador del reconocimiento. En algunos navegadores se encuentra como `webkitSpeechRecognition`. |
| `lang` | Establece el idioma que se espera reconocer, por ejemplo, `es-ES`. |
| `continuous` | Solicita resultados continuos en lugar de finalizar tras un único resultado. No garantiza que la escucha permanezca activa indefinidamente: el servicio puede finalizar y emitir `end`. |
| `interimResults` | Si vale `true`, permite recibir hipótesis provisionales mientras se habla. Si vale `false`, se reciben solo resultados finales. |
| `maxAlternatives` | Limita el número de transcripciones alternativas devueltas por cada resultado. Por defecto se devuelve una. |
| `start()` | Inicia la escucha. Normalmente debe llamarse como respuesta a una acción explícita del usuario. |
| `stop()` | Finaliza la captura e intenta devolver el resultado de lo que se haya reconocido hasta ese momento. |
| `abort()` | Detiene el reconocimiento sin intentar devolver un resultado. |

#### 3.2. Eventos y lectura de resultados

Los eventos permiten actualizar la interfaz según el estado del servicio:

- `start` indica que el servicio ha comenzado a escuchar; `end`, que ha dejado de hacerlo.
- `audiostart` y `audioend` señalan el inicio y el final de la captura de audio.
- `speechstart` y `speechend` indican cuándo se detecta habla, que no necesariamente coincide con toda la captura de audio.
- `result` se dispara cuando hay uno o más resultados de reconocimiento.
- `error` informa de errores como `not-allowed` (permiso denegado), `no-speech` (no se detectó habla), `audio-capture` (no se pudo capturar audio) o `network` (problema de red).
- `nomatch` puede indicar que el servicio no encontró una transcripción suficientemente reconocible.

El evento `result` contiene una lista `results`. Cada posición representa un resultado y puede contener varias alternativas. La alternativa más probable suele estar en la posición `0`; sus propiedades `transcript` y `confidence` permiten leer el texto reconocido y la confianza estimada por el servicio. La confianza es una estimación del reconocedor, no una garantía de que la frase sea correcta.

Si `interimResults` está activado, un resultado puede ser provisional (`isFinal === false`) y cambiar mientras la persona continúa hablando. Conviene mostrarlo como texto provisional, pero esperar a que sea final antes de ejecutar acciones con consecuencias. `resultIndex` indica el primer resultado de la lista que ha cambiado en el evento actual. Un patrón básico para recorrer los resultados es:

```javascript
recognition.addEventListener("result", (event) => {
  for (let i = event.resultIndex; i < event.results.length; i++) {
    const result = event.results[i];
    const transcript = result[0].transcript.trim();

    if (result.isFinal) {
      console.log("Resultado final:", transcript);
    } else {
      console.log("Resultado provisional:", transcript);
    }
  }
});
```

Para comandos de una interfaz, suele ser más seguro normalizar el texto (por ejemplo, pasar a minúsculas y quitar espacios sobrantes), compararlo con una lista permitida y ofrecer una alternativa visual o escrita. La frase reconocida puede contener errores, ruido o palabras adicionales; por eso, una acción importante no debería ejecutarse basándose únicamente en una transcripción ambigua.

#### 3.3. Síntesis de voz: respuestas habladas

La otra parte de la API, `SpeechSynthesis`, permite que la página lea texto en voz alta. Se crea un mensaje con `SpeechSynthesisUtterance` y se envía al sintetizador del dispositivo:

```javascript
const respuesta = new SpeechSynthesisUtterance("El color de la página es azul.");
respuesta.lang = "es-ES";
window.speechSynthesis.speak(respuesta);
```

La voz y el resultado pueden variar según el sistema y los idiomas disponibles. La síntesis es útil para confirmar una acción, leer instrucciones o facilitar el acceso a contenido; se puede combinar con el reconocimiento para crear un diálogo por voz. Si se utiliza en una aplicación real, se debe permitir detener o silenciar la lectura y evitar que la respuesta hablada interfiera con la captura del micrófono.

#### 3.4. Compatibilidad, permisos y privacidad

La compatibilidad de `SpeechRecognition` no es uniforme entre navegadores. Algunas implementaciones utilizan un servicio de reconocimiento remoto y otras pueden ofrecer reconocimiento en el propio dispositivo. Por tanto, no se debe asumir que el audio se procesa localmente ni que la API estará disponible sin conexión: hay que comprobar las capacidades del navegador y explicar al usuario cuándo se activa el micrófono.

Al iniciar el reconocimiento, el navegador puede solicitar permiso para usar el micrófono. La interfaz debe comunicar claramente si el acceso se ha denegado o no hay entrada de audio. Para una experiencia accesible y robusta, ofrece siempre un control alternativo que no dependa de la voz, como botones o un campo de texto. No uses reconocimiento de voz como único método de autenticación ni para confirmar operaciones sensibles.

Para probar el ejemplo, abre la página desde `localhost` o una conexión segura (`HTTPS`), concede acceso al micrófono y habla en el idioma configurado. Los resultados pueden variar por el ruido, el acento, el micrófono y el servicio disponible.

**Documentación oficial de referencia:**

- [Especificación de Web Speech API (WICG)](https://wicg.github.io/speech-api/).
- [Guía de Web Speech API en MDN](https://developer.mozilla.org/es/docs/Web/API/Web_Speech_API).
- Referencias de MDN para [`SpeechRecognition`](https://developer.mozilla.org/en-US/docs/Web/API/SpeechRecognition), [`SpeechRecognitionEvent`](https://developer.mozilla.org/en-US/docs/Web/API/SpeechRecognitionEvent), [`SpeechSynthesis`](https://developer.mozilla.org/en-US/docs/Web/API/SpeechSynthesis) y [`SpeechSynthesisUtterance`](https://developer.mozilla.org/en-US/docs/Web/API/SpeechSynthesisUtterance).

#### 3.5. Ejemplo: control de interfaz mediante voz

En el siguiente ejemplo, el usuario activa el reconocimiento con un botón. La aplicación muestra el texto detectado y cambia el color de fondo cuando reconoce los comandos «rojo», «verde», «azul» o «restablecer». Observa cómo se configuran las opciones, cómo se responde a los eventos y cómo se relaciona la transcripción con una acción de la interfaz. El código se separa en dos archivos: `index.html` (estructura y estilos) y `app.js` (lógica).

Archivo `index.html`:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Control por Voz - NUI</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      transition: background-color 0.5s ease;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      height: 100vh;
      margin: 0;
    }
    .card {
      background: white;
      padding: 2rem;
      border-radius: 12px;
      box-shadow: 0 4px 15px rgba(0,0,0,0.1);
      text-align: center;
    }
    .status {
      font-weight: bold;
      color: #007bff;
      margin-top: 1rem;
    }
  </style>
</head>
<body>

  <div class="card">
    <h1>Panel de Control por Voz</h1>
    <p>Di ordenes como: <strong>"Rojo"</strong>, <strong>"Verde"</strong>, <strong>"Azul"</strong> o <strong>"Restablecer"</strong>.</p>
    <button id="btnStart">Activar Micrófono</button>
    <p class="status" id="statusText">Estado: Inactivo</p>
    <p>Último comando detectado: <span id="commandText">-</span></p>
  </div>

  <script src="app.js"></script>
</body>
</html>
```

Archivo `app.js` (debe estar en la misma carpeta que `index.html`):

```js
// Chrome y Edge exponen la API con el prefijo "webkit"; otros navegadores,
// sin prefijo. Se usa la que exista. Si ninguna existe, el valor es undefined.
const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;

// Si el navegador no soporta la API, se avisa y no se ejecuta el resto.
if (!SpeechRecognition) {
  alert("Tu navegador no soporta la Web Speech API. Prueba con Google Chrome u Edge.");
} else {
  // Se crea el objeto que se encargará de escuchar y transcribir.
  const recognition = new SpeechRecognition();

  // Idioma esperado del habla: español de España.
  recognition.lang = 'es-ES';
  // true: sigue escuchando tras cada frase, hasta que se llame a stop().
  // false (por defecto) pararía solo después de la primera frase.
  recognition.continuous = true;
  // false: solo se reciben resultados finales, no las transcripciones parciales.
  recognition.interimResults = false;

  // Se guardan referencias a los elementos del HTML con los que se va a trabajar.
  const btnStart = document.getElementById('btnStart');
  const statusText = document.getElementById('statusText');
  const commandText = document.getElementById('commandText');

  // Variable de control para saber si el micrófono está activo ahora mismo.
  let isListening = false;

  // El botón alterna entre iniciar y detener la escucha.
  // start() debe llamarse tras una acción del usuario: así el navegador
  // puede pedir el permiso del micrófono.
  btnStart.addEventListener('click', () => {
    if (!isListening) {
      recognition.start();
    } else {
      recognition.stop();
    }
  });

  // Se dispara cuando el navegador empieza a escuchar de verdad.
  // Aquí se actualiza la interfaz para reflejar el nuevo estado.
  recognition.onstart = () => {
    isListening = true;
    statusText.textContent = "Estado: Escuchando...";
    btnStart.textContent = "Detener Micrófono";
  };

  // Se dispara siempre que la escucha termina: por stop(), por un error
  // o porque el navegador la cierra. Se restaura la interfaz al estado inicial.
  recognition.onend = () => {
    isListening = false;
    statusText.textContent = "Estado: Inactivo";
    btnStart.textContent = "Activar Micrófono";
  };

  // Se dispara cada vez que el servicio reconoce una frase.
  recognition.onresult = (event) => {
    // Con continuous = true, event.results acumula todas las frases de la sesión.
    // La que acaba de llegar es la última de la lista.
    const lastIndex = event.results.length - 1;

    // [lastIndex] es el resultado y [0] su mejor alternativa (la más probable).
    // trim() quita espacios sobrantes y toLowerCase() pasa a minúsculas
    // para que "Rojo" y "rojo" se traten igual.
    const transcript = event.results[lastIndex][0].transcript.trim().toLowerCase();

    // Se muestra en pantalla lo que se ha entendido.
    commandText.textContent = transcript;

    // Se busca una palabra clave dentro de la frase y se aplica la acción.
    // includes() permite que funcione con frases como "pon el fondo en rojo".
    // Si no coincide ningún comando, no se cambia nada.
    if (transcript.includes('rojo')) {
      document.body.style.backgroundColor = '#f8d7da';
    } else if (transcript.includes('verde')) {
      document.body.style.backgroundColor = '#d4edda';
    } else if (transcript.includes('azul')) {
      document.body.style.backgroundColor = '#cce5ff';
    } else if (transcript.includes('restablecer') || transcript.includes('blanco')) {
      document.body.style.backgroundColor = '#ffffff';
    }
  };

  // Se dispara si algo falla (permiso denegado, sin micrófono, sin red...).
  // event.error contiene el código, por ejemplo "not-allowed" o "no-speech".
  // Después del error, el navegador lanza también "end".
  recognition.onerror = (event) => {
    console.error("Error en el reconocimiento de voz:", event.error);
    statusText.textContent = "Error: " + event.error;
  };
}
```

#### 3.6. Ejemplo: síntesis de voz para leer un texto

Este ejemplo utiliza `SpeechSynthesis` para que el navegador lea en voz alta el texto escrito por el usuario. Se puede seleccionar el idioma, iniciar la lectura y detenerla. La voz concreta depende de las voces que tenga disponibles el dispositivo.

Archivo `index.html`:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Lectura de texto con voz</title>
</head>
<body>
  <h1>Lectura de texto con voz</h1>
  <label for="texto">Texto que se leerá:</label>
  <textarea id="texto" rows="4" cols="50">Hola. Esta página puede leer texto en voz alta.</textarea>

  <p>
    <label for="idioma">Idioma:</label>
    <select id="idioma">
      <option value="es-ES">Español</option>
      <option value="en-US">Inglés</option>
    </select>
  </p>

  <button id="hablar" type="button">Leer texto</button>
  <button id="detener" type="button">Detener</button>
  <p id="estado" role="status" aria-live="polite">Preparado.</p>

  <script src="lectura.js"></script>
</body>
</html>
```

Archivo `lectura.js` (en la misma carpeta que `index.html`):

```js
// Referencias a los elementos del HTML que se van a leer o modificar.
const texto = document.getElementById("texto");
const idioma = document.getElementById("idioma");
const estado = document.getElementById("estado");
const hablar = document.getElementById("hablar");
const detener = document.getElementById("detener");

// Se comprueba que el navegador dispone de las dos piezas necesarias:
// speechSynthesis (el sintetizador que reproduce) y
// SpeechSynthesisUtterance (el "mensaje" que se le entrega para leer).
if (!("speechSynthesis" in window) || !("SpeechSynthesisUtterance" in window)) {
  // Si falta alguna, se informa y se desactivan los botones para que
  // el usuario no pulse algo que no va a funcionar.
  estado.textContent = "Este navegador no admite la síntesis de voz.";
  hablar.disabled = true;
  detener.disabled = true;
} else {
  // Botón "Leer texto".
  hablar.addEventListener("click", () => {
    // Se toma el contenido del textarea sin espacios al principio ni al final.
    const mensaje = texto.value.trim();

    // Si está vacío no hay nada que leer: se avisa y se sale de la función.
    if (!mensaje) {
      estado.textContent = "Escribe un texto antes de iniciar la lectura.";
      return;
    }

    // Cancela una lectura anterior para evitar que se acumulen mensajes.
    // Sin esto, pulsar varias veces el botón encolaría las lecturas una tras otra.
    window.speechSynthesis.cancel();

    // Se crea el mensaje que se va a pronunciar, con el texto escrito.
    const utterance = new SpeechSynthesisUtterance(mensaje);

    // Se indica el idioma elegido en el desplegable; el navegador usará
    // una voz instalada que coincida con ese idioma.
    utterance.lang = idioma.value;

    // Eventos del mensaje: permiten reflejar en la interfaz cada fase de la lectura.
    // Cuando empieza a sonar la voz:
    utterance.onstart = () => {
      estado.textContent = "Leyendo el texto...";
    };
    // Cuando termina de leer todo el texto:
    utterance.onend = () => {
      estado.textContent = "Lectura finalizada.";
    };
    // Si algo falla durante la lectura (por ejemplo, no hay voz disponible):
    utterance.onerror = (event) => {
      estado.textContent = "No se pudo completar la lectura: " + event.error;
    };

    // Se entrega el mensaje al sintetizador, que empieza a leerlo.
    window.speechSynthesis.speak(utterance);
  });

  // Botón "Detener": cancela la lectura en curso y vacía la cola de mensajes.
  // Se actualiza el estado a mano porque, al cancelar, onend puede no
  // dispararse de forma fiable en todos los navegadores.
  detener.addEventListener("click", () => {
    window.speechSynthesis.cancel();
    estado.textContent = "Lectura detenida.";
  });
}
```

Prueba a cambiar el texto y el idioma, inicia la lectura y deténla antes de que termine. Si no se escucha una voz, revisa el volumen del dispositivo y las voces disponibles en la configuración del sistema o del navegador.

#### 3.7. Casos de uso de la voz en interfaces

El reconocimiento y la síntesis de voz pueden utilizarse, por ejemplo, para:

- **Accesibilidad y uso manos libres:** navegar por un sitio o accionar controles cuando no se puede utilizar el teclado o el ratón.
- **Búsqueda y dictado:** introducir una consulta en un buscador o dictar texto en un formulario, siempre permitiendo revisar y corregir la transcripción.
- **Asistentes y experiencias educativas:** formular comandos, recibir confirmaciones habladas o practicar pronunciación e idiomas.
- **Interfaces en contextos compartidos:** consultar información en un kiosco o dispositivo público; en estos casos hay que considerar ruido, privacidad y disponibilidad de una alternativa táctil.

#### 3.8. Actividades

**Actividad 1: asistente de voz para una página web.** Amplía el ejemplo para convertirlo en un pequeño asistente de voz:

1. Añade los comandos «más grande» y «más pequeño» para cambiar el tamaño del texto principal.
2. Añade los comandos «modo oscuro» y «modo claro» para cambiar el aspecto de la página.
3. Muestra en pantalla el último comando reconocido y un estado comprensible para el usuario: inactivo, escuchando o error.
4. Añade una respuesta escrita cuando se reconozca un comando. Como ampliación, utiliza `SpeechSynthesis` para leerla. Si se reconoce una frase que no corresponde a ningún comando, muestra un mensaje sin cambiar la página.
5. Comprueba qué ocurre si el usuario deniega el permiso del micrófono, no se detecta habla o el navegador no admite el reconocimiento. La página debe informar de la situación y mantener los controles alternativos.

**Entrega:** el código actualizado y una breve explicación de los comandos implementados (mediante comentarios en el código) y de las pruebas realizadas. Prueba cada comando al menos una vez y verifica también el caso de una frase desconocida.

**Actividad 2: lista de la compra por voz.** Crea una aplicación web que permita gestionar una lista de la compra usando la voz. Tú decides la estructura, los archivos, el diseño y cómo interpretar las frases.

Requisitos mínimos:

- El usuario puede añadir un producto diciendo una frase como «añade leche», eliminar uno («quita leche») y vaciar la lista completa.
- La lista se muestra en pantalla y se conserva al recargar la página.
- La aplicación responde hablando para confirmar cada acción (por ejemplo, «He añadido leche»).
- Debe gestionar los casos límite: navegador no compatible, permiso de micrófono denegado, silencio, frase no entendida y producto que no está en la lista.
- Debe ofrecer una alternativa sin voz (por ejemplo, un campo de texto y botones) para poder usarse sin micrófono.

**Entrega:** los archivos del proyecto y una breve explicación (comentada en el código) de cómo interpretas las frases, qué decisiones de diseño has tomado y qué pruebas has realizado (incluidos los casos límite).

### 4. Detección de manos y rostro
La cámara capta imágenes y **MediaPipe Tasks Vision** las analiza en el navegador con un modelo ya entrenado. La biblioteca devuelve puntos de referencia (*landmarks*) que podemos usar para crear interacciones.

#### 4.1. Detección de manos (Hand Landmarker)
El modelo localiza hasta 21 puntos por mano. Cada punto tiene coordenadas `x`, `y` y `z`: `x` e `y` indican su posición en la imagen (entre `0` y `1`), y `z` representa la profundidad aproximada. Por ejemplo, el punto `0` es la muñeca, el `4` es la punta del pulgar y el `8` la punta del índice.

En este ejemplo, la página muestra la posición de la punta del índice en un campo, dibuja un botón en la posición que indiquemos y reacciona cuando el índice entra en él: el botón cambia de color y aparece el mensaje «Botón pulsado». Las coordenadas X e Y se expresan como porcentajes de la imagen: (0 %, 0 %) es la esquina superior izquierda y (100 %, 100 %) la esquina inferior derecha. Crea tres archivos en la misma carpeta, `index.html`, `styles.css` y `app.js`, y copia cada bloque en el archivo correspondiente. Abre `index.html` desde un servidor local (por ejemplo, Live Server en VS Code) o una página HTTPS. La cámara no suele estar disponible al abrir el archivo directamente con `file://`. Acepta el permiso de cámara y comprueba que el navegador tenga acceso a ella.

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Detección de manos - MediaPipe</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <h1>Detección de manos</h1>
  <button id="botonInicio">Iniciar cámara</button>
  <p id="estado" role="status">Pulsa el botón y permite el acceso a la cámara.</p>
  <p>Posición del índice: <output id="posicionIndice">Sin detectar</output></p>
  <p id="mensaje" role="status"></p>
  <div id="contenedorVideo">
    <video id="videoCamara" autoplay muted playsinline></video>
  </div>
  <script type="module" src="app.js"></script>
</body>
</html>
```

El archivo `styles.css` contiene los estilos y la posición de los elementos:

```css
body {
  margin: 0;
  padding: 1rem;
  background: #1a1a1a;
  color: white;
  font-family: sans-serif;
  text-align: center;
}

button {
  padding: 0.8rem 1rem;
  font-size: 1rem;
  cursor: pointer;
}

#estado { min-height: 1.5em; }

#mensaje { min-height: 1.5em; font-weight: bold; }

video {
  width: 100%;
  height: 100%;
  background: #333;
}

#contenedorVideo {
  position: relative;
  width: min(640px, 100%);
  aspect-ratio: 4 / 3;
  margin: 1rem auto;
  background: #333;
}

#botonDibujado {
  position: absolute;
  transform: translate(-50%, -50%);
  padding: 0.6rem 1rem;
  border: 2px solid white;
  border-radius: 6px;
  background: #087f5b;
  color: white;
}

#botonDibujado.pulsado { background: #c92a2a; }
```

El archivo `app.js` contiene la lógica de la detección:

```javascript
import { FilesetResolver, HandLandmarker } from "https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@0.10.14/vision_bundle.mjs";

const videoCamara = document.getElementById("videoCamara");
const campoPosicion = document.getElementById("posicionIndice");
const botonInicio = document.getElementById("botonInicio");
const estado = document.getElementById("estado");
const mensaje = document.getElementById("mensaje");
const contenedorVideo = document.getElementById("contenedorVideo");
let detectorManos;
let ultimoTiempoVideo = -1;
let flujoCamara;

// Solicita la cámara, carga el modelo de manos e inicia la detección.
async function iniciarDeteccion() {
  botonInicio.disabled = true;
  estado.textContent = "Solicitando acceso a la cámara...";

  try {
    flujoCamara = await navigator.mediaDevices.getUserMedia({
      video: { width: 640, height: 480 }
    });
    videoCamara.srcObject = flujoCamara;
    await videoCamara.play();

    estado.textContent = "Cargando el detector...";
    const archivosVision = await FilesetResolver.forVisionTasks(
      "https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@0.10.14/wasm"
    );
    detectorManos = await HandLandmarker.createFromOptions(archivosVision, {
      baseOptions: {
        modelAssetPath: "https://storage.googleapis.com/mediapipe-models/hand_landmarker/hand_landmarker/float16/1/hand_landmarker.task"
      },
      runningMode: "VIDEO",
      numHands: 1
    });

    estado.textContent = "Cámara activa: muestra la punta del índice.";
    dibujarBoton(50, 50);
    requestAnimationFrame(detectarManoEnVideo);
  } catch (errorInicio) {
    console.error(errorInicio);
    flujoCamara?.getTracks().forEach((pista) => pista.stop());
    flujoCamara = undefined;
    videoCamara.srcObject = null;
    detectorManos?.close();
    detectorManos = undefined;
    botonInicio.disabled = false;
    estado.textContent = "No se pudo iniciar. Comprueba el permiso de cámara, la conexión a Internet y que uses HTTPS o localhost.";
  }
}

// Crea un botón en el vídeo usando porcentajes desde su esquina superior izquierda.
function dibujarBoton(posicionX, posicionY) {
  let boton = document.getElementById("botonDibujado");

  if (!boton) {
    boton = document.createElement("button");
    boton.id = "botonDibujado";
    boton.textContent = "Botón";
    boton.type = "button";

    contenedorVideo.appendChild(boton);
  }

  boton.style.left = `${posicionX}%`;
  boton.style.top = `${posicionY}%`;
}

// Indica si un punto (en porcentajes del vídeo) está dentro del botón.
function estaDentroDelBoton(posicionX, posicionY) {
  const rectBoton = document.getElementById("botonDibujado").getBoundingClientRect();
  const rectContenedor = contenedorVideo.getBoundingClientRect();
  const izquierda = (rectBoton.left - rectContenedor.left) / rectContenedor.width * 100;
  const derecha = (rectBoton.right - rectContenedor.left) / rectContenedor.width * 100;
  const arriba = (rectBoton.top - rectContenedor.top) / rectContenedor.height * 100;
  const abajo = (rectBoton.bottom - rectContenedor.top) / rectContenedor.height * 100;

  return posicionX >= izquierda && posicionX <= derecha &&
         posicionY >= arriba && posicionY <= abajo;
}

// Cambia el aspecto del botón y el mensaje según si el índice está dentro.
function actualizarBoton(dentro) {
  const boton = document.getElementById("botonDibujado");

  if (dentro) {
    boton.classList.add("pulsado");
    mensaje.textContent = "Botón pulsado";
  } else {
    boton.classList.remove("pulsado");
    mensaje.textContent = "";
  }
}

/**
 * Analiza el fotograma actual, muestra la posición de la punta del índice
 * y cambia el botón según el índice esté dentro o fuera.
 * Se vuelve a ejecutar en el siguiente fotograma mientras la cámara está activa.
 */
function detectarManoEnVideo() {
  // Solo analiza una imagen cuando el vídeo ha avanzado y el detector está listo.
  if (videoCamara.currentTime !== ultimoTiempoVideo && detectorManos) {
    ultimoTiempoVideo = videoCamara.currentTime;
    const resultados = detectorManos.detectForVideo(videoCamara, performance.now());

    // Si hay una mano, el punto 8 corresponde a la punta del dedo índice.
    if (resultados.landmarks && resultados.landmarks.length > 0) {
      const puntaIndice = resultados.landmarks[0][8];
      // MediaPipe entrega coordenadas entre 0 y 1; las convertimos a porcentajes.
      const posicionX = puntaIndice.x * 100;
      const posicionY = puntaIndice.y * 100;
      campoPosicion.textContent = `X: ${posicionX.toFixed(1)} %, Y: ${posicionY.toFixed(1)} %`;
      actualizarBoton(estaDentroDelBoton(posicionX, posicionY));
    } else {
      campoPosicion.textContent = "No se detecta ninguna mano";
      actualizarBoton(false);
    }
  }

  // Solicita analizar el siguiente fotograma de la cámara.
  requestAnimationFrame(detectarManoEnVideo);
}

botonInicio.addEventListener("click", iniciarDeteccion);
```

**Explicación paso a paso del JavaScript:**

1. **Importar MediaPipe.** La primera línea descarga la biblioteca que contiene el detector. `FilesetResolver` prepara los archivos necesarios para ejecutarlo en el navegador y `HandLandmarker` es el detector de manos.
2. **Conectar JavaScript con el HTML.** Las llamadas a `getElementById` guardan referencias al vídeo, al campo de posición, al botón de inicio, al texto de estado y al contenedor del vídeo. `mensaje` es el texto donde aparece «Botón pulsado». `detectorManos`, `ultimoTiempoVideo` y `flujoCamara` guardan, respectivamente, el detector, el instante de la última imagen procesada y la conexión con la cámara.
3. **Esperar a que el usuario inicie.** La última línea indica que `iniciarDeteccion` se ejecuta al pulsar «Iniciar cámara». La función desactiva el botón temporalmente y actualiza el mensaje.
4. **Solicitar la cámara.** `getUserMedia` pide permiso para recibir vídeo. Cuando se concede, se conecta la cámara al elemento `<video>` y `videoCamara.play()` inicia la reproducción.
5. **Preparar el detector.** `forVisionTasks` carga los archivos de MediaPipe. Después, `createFromOptions` carga el modelo de manos, indica que se analizará vídeo (`VIDEO`) y limita la detección a una mano (`numHands: 1`).
6. **Dibujar un botón.** `dibujarBoton(50, 50)` crea un botón en el centro del contenedor del vídeo. La función recibe la posición horizontal y vertical como porcentajes; por ejemplo, `dibujarBoton(20, 30)` lo coloca a un 20 % desde la izquierda y un 30 % desde arriba. Si se vuelve a llamar, mueve el botón existente en vez de crear otro.
7. **Cambiar el botón.** `actualizarBoton` recibe `true` o `false`. Si el índice está dentro, añade la clase CSS `pulsado` (cambia el fondo) y escribe «Botón pulsado»; si está fuera, quita la clase y borra el mensaje.
8. **Comprobar si el índice está dentro.** `estaDentroDelBoton` calcula, en porcentajes del vídeo, los bordes del botón y comprueba si la posición del índice queda entre ellos.
9. **Cómo funciona `detectarManoEnVideo`.** La función se ejecuta una vez por fotograma mediante `requestAnimationFrame`. Comprueba si el vídeo ha avanzado y analiza la imagen con `detectForVideo`. Si encuentra una mano, obtiene el punto `8` (punta del índice), convierte sus coordenadas `x` e `y` a porcentajes, las escribe en el campo de posición y avisa al botón con `actualizarBoton`. Si no hay mano, muestra un mensaje y considera que el índice está fuera. Al final solicita el siguiente fotograma.
10. **Mostrar errores.** Si no se puede acceder a la cámara o cargar el modelo, `catch` escribe el error en la consola, libera los recursos y muestra un mensaje para revisar el permiso, la conexión y el servidor HTTPS o `localhost`.

#### 4.2. Detección del rostro (Face Landmarker)
El detector de rostro funciona de forma parecida: analiza la imagen de la cámara y devuelve puntos situados en zonas como los ojos, la nariz y la boca. Si se activa una opción adicional, también devuelve indicadores de expresiones faciales. Esto permite, por ejemplo, mover un elemento siguiendo la cabeza o reaccionar a una sonrisa.

Detectar puntos del rostro **no significa identificar a la persona**. Para empezar, basta con entender el mismo proceso del ejemplo: capturar una imagen, analizarla y utilizar los puntos devueltos. MediaPipe ofrece el modelo **Face Landmarker** para este tipo de detección.

### 5. Detección de pose corporal y control por movimiento
La detección de postura (Pose Detection) nos permite capturar el movimiento global del usuario (cabeza, hombros, codos, rodillas) para reaccionar ante acciones físicas, como levantar los brazos o posicionarse a los lados de la pantalla.

#### 5.1. Algoritmo de control por movimiento corporal

```javascript
// Fragmento conceptual de integración con Pose Landmarker de MediaPipe
function checkPoseAction(poseLandmarks) {
  // Puntos clave de referencia:
  // 15: Muñeca izquierda, 16: Muñeca derecha
  // 0: Nariz, 11: Hombro izquierdo, 12: Hombro derecho

  const leftWrist = poseLandmarks[15];
  const rightWrist = poseLandmarks[16];
  const nose = poseLandmarks[0];

  // Regla 1: Ambos brazos levantados por encima de la cabeza
  if (leftWrist.y < nose.y && rightWrist.y < nose.y) {
    triggerAction("BRAZOS_ARRIBA");
  } 
  // Regla 2: Inclinación lateral o desplazamiento a la derecha
  else if (nose.x < 0.3) {
    triggerAction("MOVER_IZQUIERDA"); // Espejado
  } else if (nose.x > 0.7) {
    triggerAction("MOVER_DERECHA");
  }
}

function triggerAction(actionName) {
  console.log("Acción NUI detectada:", actionName);
  // Lógica para modificar la UI según la acción
}
```

### 6. Realidad aumentada en el navegador (WebAR)
La **Realidad Aumentada (RA)** en la web permite superponer elementos digitales 3D sobre imágenes omarcadores detectados por la cámara del usuario en tiempo real sin instalar aplicaciones nativas.

#### 6.1. Stack tecnológico
- **A-Frame:** Motor 3D declarativo basado en la especificación WebGL.
- **MindAR:** Librería de visión computacional para seguimiento de marcadores o imágenes objetivo.

#### 6.2. Código WebAR con seguimiento de marcador

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Interfaces Naturales - Realidad Aumentada Web</title>
  <!-- Importación de A-Frame y MindAR -->
  <script src="https://aframe.io/releases/1.4.2/aframe.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/mind-ar@1.2.2/dist/mindar-image-aframe.prod.js"></script>
  <style>
    body { margin: 0; overflow: hidden; }
    .ar-ui {
      position: absolute;
      bottom: 20px;
      left: 50%;
      transform: translateX(-50%);
      z-index: 100;
    }
    button {
      padding: 12px 24px;
      font-size: 16px;
      background: #007bff;
      color: white;
      border: none;
      border-radius: 20px;
      cursor: pointer;
    }
  </style>
</head>
<body>

  <div class="ar-ui">
    <button id="colorBtn">Cambiar Color del Objeto RA</button>
  </div>

  <!-- Escena A-Frame configurada con MindAR para seguimiento de imagen -->
  <a-scene 
    mindar-image="imageTargetSrc: https://cdn.jsdelivr.net/gh/hiukim/mind-ar-js@1.2.2/examples/image-tracking/assets/card-example/card.mind;" 
    color-space="sRGB" 
    renderer="colorManagement: true, physicallyCorrectLights" 
    vr-mode-ui="enabled: false" 
    device-orientation-permission-ui="enabled: false">
    
    <a-camera position="0 0 0" look-controls="enabled: false"></a-camera>

    <!-- Entidad vinculada al objetivo de imagen (Target 0) -->
    <a-entity mindar-image-target="targetIndex: 0">
      <!-- Objeto 3D (Cubo) superpuesto en el plano real -->
      <a-box id="raBox" position="0 0.25 0" rotation="0 45 0" color="#4CC3D9" scale="0.5 0.5 0.5"
             animation="property: rotation; to: 0 405 0; loop: true; dur: 5000; startEvents: click">
      </a-box>
      <a-text value="Interfaz RA Interactiva" position="-0.8 0.8 0" color="#FFFFFF" width="3"></a-text>
    </a-entity>

  </a-scene>

  <script>
    const raBox = document.getElementById('raBox');
    const colorBtn = document.getElementById('colorBtn');
    const colors = ['#4CC3D9', '#FFC65D', '#7BC8A4', '#EF2D5E', '#9B59B6'];
    let colorIndex = 0;

    colorBtn.addEventListener('click', () => {
      colorIndex = (colorIndex + 1) % colors.length;
      raBox.setAttribute('material', 'color', colors[colorIndex]);
    });
  </script>
</body>
</html>
```

### 7. Caso práctico integrador: kiosco multimodal e interactivo NUI

Los alumnos desarrollarán una aplicación web unificada de tipo catálogo digital que responda a dos vías de entrada natural de forma simultánea:
1. **Comandos de voz:** "Siguiente", "Anterior", "Seleccionar".
2. **Gestos manuales:** Mover la mano a la derecha/izquierda para navegar entre productos y realizar el gesto de puño cerrado o toque con dedo índice para seleccionar.

---

### 8. Criterios y rúbrica de evaluación

#### 8.1. Criterios de evaluación (asociados al RA2)
- **CE2.a:** Se han identificado las herramientas y librerías clave para el desarrollo de interfaces naturales en entornos web.
- **CE2.b:** Se ha implementado el procesamiento de voz mediante Web Speech API para accionar elementos de interfaz.
- **CE2.c:** Se ha integrado la captura de video y tracking de partes del cuerpo (manos, postura o rostro) con frameworks de ML en JavaScript.
- **CE2.d:** Se han integrado elementos 3D / RA superpuestos en la emision de la camara mediante librerías de WebAR.

#### 8.2. Rúbrica de evaluación

| Criterio | Excelente (9-10) | Notable / Aprobado (5-8) | Insuficiente (1-4) |
| :--- | :--- | :--- | :--- |
| **Reconocimiento de Voz** | Integra comandos continuos, gestiona errores de audio y proporciona feedback visual instantáneo. | Implementa reconocimiento de voz funcional básico con errores puntuales de contexto. | No logra capturar o procesar eventos de voz desde la API nativa. |
| **Detección Gestual (ML)** | Mapea con precisión puntos de referencia (*landmarks*) y activa elementos del DOM fluidamente. | Detecta manos o postura pero la interacción con la UI presenta retrasos o imprecisiones. | No integra librerías de visión por ordenador o no procesa las coordenadas. |
| **Realidad Aumentada (WebAR)** | Despliega escenas RA estables, con seguimiento de marcadores y respuesta a eventos táctiles/UI. | Renderiza elementos 3D sobre la cámara pero la interacción es limitada o inestable. | No consigue inicializar la cámara o cargar la biblioteca de RA. |
| **Calidad de Código y Estructura** | Código modular en ES6+, asíncrono, limpio y documentado siguiendo buenas prácticas. | Código funcional en un único bloque con poca estructuración o falta de documentación. | Código con errores de sintaxis o fallos de ejecución no resueltos. |