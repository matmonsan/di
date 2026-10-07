# Unidad Didáctica 3: Desarrollo de Interfaces Naturales de Usuario (INU/NUI) con JavaScript

**Módulo Profesional:** Desarrollo de Interfaces  
**Duración:** 6 Horas lectivas  
**Resultado de Aprendizaje:**  
- **RA2.** Genera interfaces naturales de usuario utilizando herramientas visuales y librerías JavaScript.

---

## 1. Ficha Técnica y Objetivos Didácticos

### Contextualización
Las **Interfaces Naturales de Usuario (NUI - Natural User Interfaces)** representan un paradigma de interacción persona-ordenador donde los medios tradicionales (teclado y ratón) son sustituidos o complementados por habilidades humanas innatas: la voz, la gestualidad corporal, la expresión facial y la percepción espacial mediante Realidad Aumentada (RA).

En esta unidad se abordará el desarrollo de INUs web utilizando estándares modernos de JavaScript (Web APIs nativas como Web Speech API y WebRTC) junto con modelos de Machine Learning preentrenados en el cliente (MediaPipe / TensorFlow.js) y frameworks de Realidad Aumentada Web (MindAR / A-Frame).

### Objetivos Específicos
1. Identificar y seleccionar herramientas visuales y de Machine Learning aplicables a interfaces naturales en el entorno web (voz, gesto, postura, RA).
2. Desarrollar prototipos funcionales que respondan a comandos de voz, gestos y movimiento corporal directamente en el navegador.
3. Integrar algoritmos de detección de partes del cuerpo (manos, rostro, pose) para accionar la interfaz sin contacto físico.
4. Incorporar Realidad Aumentada como capa de interacción sobre el mundo real dentro del ecosistema web.

---

## 2. Temporalización y Estructura del Curso (6 Horas)

| Sesión | Contenido Principal | Modalidad | Duración |
| :--- | :--- | :--- | :--- |
| **Sesión 1** | Conceptos de INU/NUI y ecosistema de herramientas en JavaScript | Teórica - Práctica | 1 h |
| **Sesión 2** | Reconocimiento de voz interactivo con Web Speech API | Práctica guiada | 1 h |
| **Sesión 3** | Detección de manos y rostro con MediaPipe Tasks Vision | Práctica guiada | 1.5 h |
| **Sesión 4** | Detección de pose corporal y control gestual de acciones | Práctica guiada | 1 h |
| **Sesión 5** | Realidad Aumentada Web (WebAR) con MindAR y A-Frame | Práctica guiada / Proyecto | 1.5 h |

---

## 3. Desarrollo Contenido Teórico y Práctico

---

### SESIÓN 1: Conceptos y Herramientas para INU en la Web (1 Hora)

#### 1.1 Introducción a las Interfaces Naturales de Usuario (NUI)
A diferencia de las **GUI** (Interfaces Gráficas basadas en ventanas e íconos) o las **CUI** (Interfaces de Línea de Comandos), las **NUI** buscan reducir la brecha cognitiva de aprendizaje. El usuario interactúa a través de conductas cotidianas:
- **Voz:** Comandos orales, dictado, análisis semántico.
- **Gesto y Postura:** Control gestual con las manos, inclinación de cabeza, postura corporal completa.
- **Rostro:** Reconocimiento de expresiones faciales o dirección de la mirada (*eye tracking*).
- **Entorno Aumentado (RA):** Inserción de capas de información y objetos 3D sobre la emisión de video en tiempo real.

#### 1.2 Ecosistema Tecnológico en JavaScript
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

---

### SESIÓN 2: Reconocimiento de Voz con Web Speech API (1 Hora)

La `Web Speech API` permite procesar eventos de audio e interpretar lo que el usuario pronuncia, transformándolo en texto utilizable para ejecutar eventos del DOM.

#### Código Fuente Completo: Control de Interfaz mediante Voz
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

---

### SESIÓN 3: Detección de Partes del Cuerpo (Manos y Rostro) (1.5 Horas)

Utilizaremos **MediaPipe Tasks Vision** mediante CDNs en JavaScript para detectar los puntos de referencia de la mano (21 landmarks) e interactuar con botones dinámicos en pantalla.

#### 3.1 Detección de Manos (Hand Landmarker)
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

---

### SESIÓN 4: Detección de Pose Corporal y Control por Movimiento (1 Hora)

La detección de postura (Pose Detection) nos permite capturar el movimiento global del usuario (cabeza, hombros, codos, rodillas) para reaccionar ante acciones físicas, como levantar los brazos o posicionarse a los lados de la pantalla.

#### Algoritmo de Control por Movimiento Corporal

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

---

### SESIÓN 5: Realidad Aumentada en el Navegador (WebAR) (1.5 Horas)

La **Realidad Aumentada (RA)** en la web permite superponer elementos digitales 3D sobre imágenes omarcadores detectados por la cámara del usuario en tiempo real sin instalar aplicaciones nativas.

#### Stack Tecnológico:
- **A-Frame:** Motor 3D declarativo basado en la especificación WebGL.
- **MindAR:** Librería de visión computacional para seguimiento de marcadores o imágenes objetivo.

#### Código Fuente Completo: WebAR con Seguimiento de Marcador

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

---

## 4. Prácticas Guiadas y Caso Práctico Integrador

### Caso Práctico Integrador: "Kiosco Multimodal e Interactivo NUI"
Los alumnos desarrollarán una aplicación web unificada de tipo catálogo digital que responda a dos vías de entrada natural de forma simultánea:
1. **Comandos de voz:** "Siguiente", "Anterior", "Seleccionar".
2. **Gestos manuales:** Mover la mano a la derecha/izquierda para navegar entre productos y realizar el gesto de puño cerrado o toque con dedo índice para seleccionar.

---

## 5. Criterios y Rúbrica de Evaluación

### Criterios de Evaluación (Asociados al RA2)
- **CE2.a:** Se han identificado las herramientas y librerías clave para el desarrollo de interfaces naturales en entornos web.
- **CE2.b:** Se ha implementado el procesamiento de voz mediante Web Speech API para accionar elementos de interfaz.
- **CE2.c:** Se ha integrado la captura de video y tracking de partes del cuerpo (manos, postura o rostro) con frameworks de ML en JavaScript.
- **CE2.d:** Se han integrado elementos 3D / RA superpuestos en la emision de la camara mediante librerías de WebAR.

### Rúbrica de Evaluación

| Criterio | Excelente (9-10) | Notable / Aprobado (5-8) | Insuficiente (1-4) |
| :--- | :--- | :--- | :--- |
| **Reconocimiento de Voz** | Integra comandos continuos, gestiona errores de audio y proporciona feedback visual instantáneo. | Implementa reconocimiento de voz funcional básico con errores puntuales de contexto. | No logra capturar o procesar eventos de voz desde la API nativa. |
| **Detección Gestual (ML)** | Mapea con precisión puntos de referencia (*landmarks*) y activa elementos del DOM fluidamente. | Detecta manos o postura pero la interacción con la UI presenta retrasos o imprecisiones. | No integra librerías de visión por ordenador o no procesa las coordenadas. |
| **Realidad Aumentada (WebAR)** | Despliega escenas RA estables, con seguimiento de marcadores y respuesta a eventos táctiles/UI. | Renderiza elementos 3D sobre la cámara pero la interacción es limitada o inestable. | No consigue inicializar la cámara o cargar la biblioteca de RA. |
| **Calidad de Código y Estructura** | Código modular en ES6+, asíncrono, limpio y documentado siguiendo buenas prácticas. | Código funcional en un único bloque con poca estructuración o falta de documentación. | Código con errores de sintaxis o fallos de ejecución no resueltos. |