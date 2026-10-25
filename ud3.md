---
title: UD3. Desarrollo de interfaces naturales
description: "<strong>Módulo:</strong> Desarrollo de Interfaces <br> <strong>Profesor:</strong> Matías Montávez Sánchez"
---

[⌂ Volver al inicio](index.md)

# Índice de contenidos

- [1. Introducción a las interfaces naturales de usuario](#1-introducción-a-las-interfaces-naturales-de-usuario)
- [2. Tecnologías para desarrollar interfaces naturales](#2-tecnologías-para-desarrollar-interfaces-naturales)
- [3. Reconocimiento de voz con Web Speech API](#3-reconocimiento-de-voz-con-web-speech-api)
- [4. Detección de manos y rostro](#4-detección-de-manos-y-rostro)
- [5. Detección de pose corporal y control por movimiento](#5-detección-de-pose-corporal-y-control-por-movimiento)
- [6. Realidad aumentada en el navegador (WebAR)](#6-realidad-aumentada-en-el-navegador-webar)
- [7. Caso práctico integrador: kiosco multimodal e interactivo NUI](#7-caso-práctico-integrador-kiosco-multimodal-e-interactivo-nui)
- [8. Criterios y rúbrica de evaluación](#8-criterios-y-rúbrica-de-evaluación)

### 1. Introducción a las interfaces naturales de usuario
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

En el siguiente ejemplo, el usuario activa el reconocimiento con un botón. La aplicación muestra el texto detectado y cambia el color de fondo cuando reconoce los comandos «rojo», «verde», «azul» o «restablecer». Observa cómo se configuran las opciones, cómo se responde a los eventos y cómo se relaciona la transcripción con una acción de la interfaz.

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

  <script>
    // Comprobación de compatibilidad con prefijos de navegador
    const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;

    if (!SpeechRecognition) {
      alert("Tu navegador no soporta la Web Speech API. Prueba con Google Chrome u Edge.");
    } else {
      const recognition = new SpeechRecognition();
      recognition.lang = 'es-ES';
      recognition.continuous = true;
      recognition.interimResults = false;

      const btnStart = document.getElementById('btnStart');
      const statusText = document.getElementById('statusText');
      const commandText = document.getElementById('commandText');

      let isListening = false;

      btnStart.addEventListener('click', () => {
        if (!isListening) {
          recognition.start();
        } else {
          recognition.stop();
        }
      });

      recognition.onstart = () => {
        isListening = true;
        statusText.textContent = "Estado: Escuchando...";
        btnStart.textContent = "Detener Micrófono";
      };

      recognition.onend = () => {
        isListening = false;
        statusText.textContent = "Estado: Inactivo";
        btnStart.textContent = "Activar Micrófono";
      };

      recognition.onresult = (event) => {
        const lastIndex = event.results.length - 1;
        const transcript = event.results[lastIndex][0].transcript.trim().toLowerCase();
        commandText.textContent = transcript;

        // Lógica de procesamiento de comandos simples
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

      recognition.onerror = (event) => {
        console.error("Error en el reconocimiento de voz:", event.error);
        statusText.textContent = "Error: " + event.error;
      };
    }
  </script>
</body>
</html>
```

#### 3.6. Ejemplo: síntesis de voz para leer un texto

Este ejemplo utiliza `SpeechSynthesis` para que el navegador lea en voz alta el texto escrito por el usuario. Se puede seleccionar el idioma, iniciar la lectura y detenerla. La voz concreta depende de las voces que tenga disponibles el dispositivo.

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

  <script>
    const texto = document.getElementById("texto");
    const idioma = document.getElementById("idioma");
    const estado = document.getElementById("estado");
    const hablar = document.getElementById("hablar");
    const detener = document.getElementById("detener");

    if (!("speechSynthesis" in window) || !("SpeechSynthesisUtterance" in window)) {
      estado.textContent = "Este navegador no admite la síntesis de voz.";
      hablar.disabled = true;
      detener.disabled = true;
    } else {
      hablar.addEventListener("click", () => {
        const mensaje = texto.value.trim();

        if (!mensaje) {
          estado.textContent = "Escribe un texto antes de iniciar la lectura.";
          return;
        }

        // Cancela una lectura anterior para evitar que se acumulen mensajes.
        window.speechSynthesis.cancel();

        const utterance = new SpeechSynthesisUtterance(mensaje);
        utterance.lang = idioma.value;
        utterance.onstart = () => {
          estado.textContent = "Leyendo el texto...";
        };
        utterance.onend = () => {
          estado.textContent = "Lectura finalizada.";
        };
        utterance.onerror = (event) => {
          estado.textContent = "No se pudo completar la lectura: " + event.error;
        };

        window.speechSynthesis.speak(utterance);
      });

      detener.addEventListener("click", () => {
        window.speechSynthesis.cancel();
        estado.textContent = "Lectura detenida.";
      });
    }
  </script>
</body>
</html>
```

Prueba a cambiar el texto y el idioma, inicia la lectura y deténla antes de que termine. Si no se escucha una voz, revisa el volumen del dispositivo y las voces disponibles en la configuración del sistema o del navegador.

#### 3.7. Casos de uso y ejercicio: asistente de voz para una página web

El reconocimiento y la síntesis de voz pueden utilizarse, por ejemplo, para:

- **Accesibilidad y uso manos libres:** navegar por un sitio o accionar controles cuando no se puede utilizar el teclado o el ratón.
- **Búsqueda y dictado:** introducir una consulta en un buscador o dictar texto en un formulario, siempre permitiendo revisar y corregir la transcripción.
- **Asistentes y experiencias educativas:** formular comandos, recibir confirmaciones habladas o practicar pronunciación e idiomas.
- **Interfaces en contextos compartidos:** consultar información en un kiosco o dispositivo público; en estos casos hay que considerar ruido, privacidad y disponibilidad de una alternativa táctil.

**Ejercicio: asistente de voz para una página web.** Amplía el ejemplo para convertirlo en un pequeño asistente de voz:

1. Añade los comandos «más grande» y «más pequeño» para cambiar el tamaño del texto principal.
2. Añade los comandos «modo oscuro» y «modo claro» para cambiar el aspecto de la página.
3. Muestra en pantalla el último comando reconocido y un estado comprensible para el usuario: inactivo, escuchando o error.
4. Añade una respuesta escrita cuando se reconozca un comando. Como ampliación, utiliza `SpeechSynthesis` para leerla. Si se reconoce una frase que no corresponde a ningún comando, muestra un mensaje sin cambiar la página.
5. Comprueba qué ocurre si el usuario deniega el permiso del micrófono, no se detecta habla o el navegador no admite el reconocimiento. La página debe informar de la situación y mantener los controles alternativos.

**Entrega:** el código actualizado y una breve explicación de los comandos implementados y de las pruebas realizadas. Prueba cada comando al menos una vez y verifica también el caso de una frase desconocida.

### 4. Detección de manos y rostro
Utilizaremos **MediaPipe Tasks Vision** mediante CDNs en JavaScript para detectar los puntos de referencia de la mano (21 landmarks) e interactuar con botones dinámicos en pantalla.

#### 4.1. Detección de manos (Hand Landmarker)
Cada mano devuelve 21 coordenadas tri-dimensionales $(x, y, z)$. El punto `0` es la muñeca, el `4` es la punta del pulgar y el `8` la punta del índice.

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Interacción con Manos - MediaPipe</title>
  <style>
    body { margin: 0; overflow: hidden; background: #1a1a1a; color: white; font-family: sans-serif; }
    #container { position: relative; width: 640px; height: 480px; margin: 20px auto; }
    video, canvas { position: absolute; top: 0; left: 0; width: 100%; height: 100%; transform: scaleX(-1); }
    .interactive-btn {
      position: absolute;
      top: 50px;
      left: 220px;
      width: 200px;
      height: 60px;
      background: rgba(255, 0, 100, 0.8);
      border: 3px solid white;
      border-radius: 8px;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 1.2rem;
      font-weight: bold;
      z-index: 10;
      pointer-events: none;
    }
    .active { background: rgba(0, 255, 100, 0.9) !important; }
  </style>
  <script type="module">
    import { HandLandmarker, FilesetResolver } from "https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@0.10.0";

    const video = document.getElementById("webcam");
    const canvas = document.getElementById("output_canvas");
    const ctx = canvas.getContext("2d");
    const virtualBtn = document.getElementById("virtualBtn");

    let handLandmarker;
    let lastVideoTime = -1;

    async function initializeHandLandmarker() {
      const vision = await FilesetResolver.forVisionTasks(
        "https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@0.10.0/wasm"
      );
      handLandmarker = await HandLandmarker.createFromOptions(vision, {
        baseOptions: {
          modelAssetPath: "https://storage.googleapis.com/mediapipe-models/hand_landmarker/hand_landmarker/float16/1/hand_landmarker.task",
          delegate: "GPU"
        },
        runningMode: "VIDEO",
        numHands: 1
      });
      startWebcam();
    }

    function startWebcam() {
      navigator.mediaDevices.getUserMedia({ video: { width: 640, height: 480 } }).then((stream) => {
        video.srcObject = stream;
        video.addEventListener("loadeddata", predictWebcam);
      });
    }

    async function predictWebcam() {
      canvas.width = video.videoWidth;
      canvas.height = video.videoHeight;

      if (lastVideoTime !== video.currentTime && handLandmarker) {
        lastVideoTime = video.currentTime;
        const results = handLandmarker.detectForVideo(video, performance.now());

        ctx.clearRect(0, 0, canvas.width, canvas.height);

        if (results.landmarks && results.landmarks.length > 0) {
          const landmarks = results.landmarks[0];
          // Punta del índice (Landmark 8)
          const indexTip = landmarks[8];

          // Invertir X para coincidir con la vista espejada
          const cursorX = (1 - indexTip.x) * canvas.width;
          const cursorY = indexTip.y * canvas.height;

          // Dibujar puntero virtual
          ctx.beginPath();
          ctx.arc(cursorX, cursorY, 15, 0, 2 * Math.PI);
          ctx.fillStyle = "yellow";
          ctx.fill();

          // Comprobar colisión con el botón
          const btnRect = virtualBtn.getBoundingClientRect();
          const containerRect = document.getElementById("container").getBoundingClientRect();

          const normalizedX = cursorX + containerRect.left;
          const normalizedY = cursorY + containerRect.top;

          if (
            normalizedX >= btnRect.left &&
            normalizedX <= btnRect.right &&
            normalizedY >= btnRect.top &&
            normalizedY <= btnRect.bottom
          ) {
            virtualBtn.classList.add("active");
            virtualBtn.textContent = "¡Tocado!";
          } else {
            virtualBtn.classList.remove("active");
            virtualBtn.textContent = "Tócame con el índice";
          }
        }
      }
      requestAnimationFrame(predictWebcam);
    }

    initializeHandLandmarker();
  </script>
</head>
<body>
  <div id="container">
    <video id="webcam" autoplay playsinline></video>
    <canvas id="output_canvas"></canvas>
    <div id="virtualBtn" class="interactive-btn">Tócame con el índice</div>
  </div>
</body>
</html>
```

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