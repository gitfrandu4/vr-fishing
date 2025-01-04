# VR Fishing 🎣

**URL del Proyecto**: [https://gitfrandu4.github.io/vr-fishing/](https://gitfrandu4.github.io/vr-fishing/)

---

## Índice

- [VR Fishing 🎣](#vr-fishing-)
  - [Índice](#índice)
  - [Descripción General](#descripción-general)
  - [Estructura de Archivos y Directorios](#estructura-de-archivos-y-directorios)
  - [Tecnologías Utilizadas](#tecnologías-utilizadas)
  - [Principales Módulos](#principales-módulos)
    - [SceneManager](#scenemanager)
    - [Environment](#environment)
    - [FishingRod](#fishingrod)
    - [FishManager](#fishmanager)
  - [Funcionamiento General](#funcionamiento-general)
  - [Controles y Uso](#controles-y-uso)
  - [Uso de Librerías Externas](#uso-de-librerías-externas)
    - [Ammo.js](#ammojs)
    - [three.js](#threejs)
    - [FBXLoader](#fbxloader)
  - [Configuración y Formato de Código](#configuración-y-formato-de-código)
  - [Pasos para Ejecutar el Proyecto](#pasos-para-ejecutar-el-proyecto)

---

## Descripción General

VR Fishing es un simulador de pesca en Realidad Virtual (VR) desarrollado con **three.js** y **Ammo.js**. El proyecto recrea un lago virtual con peces interactivos, donde el usuario puede lanzar una caña de pescar, capturar peces y acumular puntuación. Está diseñado para funcionar tanto en modo VR como en modo escritorio.

🔗 **Accede al proyecto aquí**: [https://gitfrandu4.github.io/vr-fishing/](https://gitfrandu4.github.io/vr-fishing/)

---

## Estructura de Archivos y Directorios

```bash
.
├── index.html
├── script.js
├── game.js
├── style.css
├── lib/
│   └── ammo.js
├── modules/
│   ├── SceneManager.js
│   ├── Environment.js
│   ├── FishingRod.js
│   ├── FishManager.js
│   ├── celestials/
│   │   ├── CelestialManager.js
│   │   ├── Sky.js
│   │   ├── Sun.js
│   │   ├── Moon.js
│   │   └── Stars.js
│   └── shaders/
│       ├── skyShaders.js
│       ├── fishShaders.js
│       └── sunShaders.js
├── models/
│   ├── fish.fbx
│   └── fishred.fbx
├── textures/
│   ├── wood/
│   ├── metal_mesh/
│   ├── grass/
│   ├── rock/
│   ├── water/
│   └── solarsystem/
├── .prettierrc.json
└── package.json
```

---

## Tecnologías Utilizadas

- **[three.js](https://threejs.org/)** - Renderizado 3D en WebGL.
- **[Ammo.js](https://github.com/kripken/ammo.js/)** - Simulación física y colisiones.
- **WebXR** - APIs para Realidad Virtual y Realidad Aumentada en navegadores.
- **HTML5/CSS3** - Estructura y estilo del juego.
- **JavaScript (ES6+)** - Lógica del juego y controladores interactivos.

---

## Principales Módulos

### SceneManager

Gestiona la escena 3D principal, incluyendo la configuración de la cámara, el renderizador y el bucle de animación. Implementa el patrón Singleton para mantener una única instancia de la escena.

```javascript
export class SceneManager {
  constructor() {
    // Configuración básica de Three.js
    this.scene = new THREE.Scene();
    this.camera = new THREE.PerspectiveCamera(
      70, // FOV
      window.innerWidth / window.innerHeight, // Aspect Ratio
      0.1, // Near plane
      1000000, // Far plane
    );

    // Configuración del renderizador con soporte VR
    this.renderer = new THREE.WebGLRenderer({ antialias: true });
    this.renderer.xr.enabled = true;
    this.renderer.toneMapping = THREE.ACESFilmicToneMapping;
    this.renderer.shadowMap.enabled = true;
  }

  startAnimation(renderCallback) {
    // Bucle de renderizado optimizado para VR
    this.renderer.setAnimationLoop(() => {
      if (renderCallback) renderCallback();
      this.renderer.render(this.scene, this.camera);
    });
  }
}
```

### Environment

Crea y gestiona el entorno virtual del lago, implementando técnicas avanzadas de shaders para el agua, terreno y efectos atmosféricos. Utiliza mapas de normales y técnicas de iluminación PBR (Physically Based Rendering).

```javascript
export class Environment {
  constructor(scene) {
    this.scene = scene;
    this.water = null;
    this.celestials = new CelestialManager(scene);
  }

  createWater() {
    const waterGeometry = new THREE.CircleGeometry(5, 64);
    this.water = new Water(waterGeometry, {
      textureWidth: 512,
      textureHeight: 512,
      flowDirection: new THREE.Vector2(1, 1),
      scale: 7,
      flowSpeed: 0.25,
      reflectivity: 0.35,
      opacity: 0.65,
    });
  }

  update(time) {
    // Actualización de shaders y efectos dinámicos
    if (this.water?.material?.uniforms) {
      this.water.material.uniforms.config.value.x = time * 0.5;
      this.water.material.uniforms.flowDirection.value.set(
        Math.sin(time * 0.1),
        Math.cos(time * 0.1),
      );
    }
  }
}
```

### FishingRod

Implementa la física e interacción de la caña de pescar utilizando técnicas de cinemática y simulación de cuerdas. Integra controles tanto para VR como para teclado/ratón.

```javascript
export class FishingRod {
  constructor(scene) {
    this.scene = scene;
    this.isGrabbed = false;
    this.isCasting = false;
    this.castPower = 0;
  }

  createLine() {
    // Sistema de física para la línea de pesca
    const lineGeometry = new THREE.BufferGeometry();
    const lineMaterial = new THREE.LineBasicMaterial({
      color: 0xffffff,
      transparent: true,
      opacity: 0.6,
    });

    // Simulación de física de cuerda
    this.updateLinePhysics = (time) => {
      const positions = this.line.geometry.attributes.position.array;
      const tension = this.calculateLineTension();
      const windEffect = Math.sin(time * 2) * 0.1;

      // Aplicar física a cada segmento de la línea
      for (let i = 0; i < positions.length; i += 3) {
        positions[i + 1] += windEffect * (1 - tension);
      }
      this.line.geometry.attributes.position.needsUpdate = true;
    };
  }
}
```

### FishManager

Gestiona el movimiento y comportamiento de los peces. Utiliza shaders personalizados para efectos submarinos.

```javascript
import { fishShaders } from './shaders/fishShaders.js';

export class FishManager {
  constructor(scene) {
    this.scene = scene;
    this.fishes = [];
  }

  createFishMaterial(color) {
    // Shader personalizado para efectos submarinos
    return new THREE.ShaderMaterial({
      uniforms: {
        time: { value: 0 },
        waterLevel: { value: -0.3 },
        color: { value: new THREE.Color(color) },
      },
      vertexShader: fishShaders.vertexShader,
      fragmentShader: fishShaders.fragmentShader,
    });
  }

  update(time) {
    this.fishes.forEach((fish) => {
      if (!fish.userData.isCaught) {
        // Movimiento procedural de los peces
        fish.userData.angle += fish.userData.speed;
        const newX =
          fish.userData.centerX +
          Math.cos(fish.userData.angle) * fish.userData.radius;
        const newZ =
          fish.userData.centerZ +
          Math.sin(fish.userData.angle) * fish.userData.radius;

        // Actualizar posición y rotación
        fish.position.set(newX, fish.userData.baseY, newZ);
        fish.rotation.y =
          Math.atan2(newX - fish.position.x, newZ - fish.position.z) +
          Math.PI / 2;
      }
    });
  }
}

export const fishShaders = {
  vertexShader: `
    varying vec3 vPosition;
    varying vec3 vNormal;
    void main() {
      vPosition = position;
      vNormal = normal;
      gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
    }
  `,
  fragmentShader: `
    uniform float time;
    uniform float waterLevel;
    uniform vec3 color;
    
    varying vec3 vPosition;
    varying vec3 vNormal;
    
    void main() {
      // Efectos submarinos caustics
      float caustics = sin(vPosition.x * 3.0 + time) * 
                      sin(vPosition.z * 3.0 + time) * 0.8 + 0.2;
      
      // Rim lighting submarino
      vec3 viewDirection = normalize(cameraPosition - vPosition);
      float rimLight = pow(1.0 - max(0.0, dot(viewDirection, vNormal)), 2.0);
      
      vec3 finalColor = color * (1.0 + caustics + rimLight);
      gl_FragColor = vec4(finalColor, 0.95);
    }
  `,
};
```

Los shaders han sido modularizados en archivos separados dentro de la carpeta `shaders/` para mejorar la mantenibilidad y reutilización del código. El shader de los peces implementa:

1. **Vertex Shader**: Prepara las variables necesarias para los cálculos de iluminación y efectos.

   - Pasa la posición y normal del vértice al fragment shader
   - Calcula la posición final del vértice en el espacio de la pantalla

2. **Fragment Shader**: Implementa efectos visuales submarinos:
   - Caustics: Simula el efecto de la luz atravesando el agua
   - Rim lighting: Añade un efecto de borde iluminado para mejor visibilidad
   - Color dinámico: Modula el color base con los efectos para dar realismo

Cada módulo está diseñado siguiendo principios de programación orientada a objetos y patrones de diseño comunes en el desarrollo de aplicaciones 3D. La arquitectura modular permite una fácil extensibilidad y mantenimiento del código, mientras que el uso de shaders personalizados y técnicas avanzadas de renderizado asegura un rendimiento óptimo y efectos visuales de alta calidad.

---

## Funcionamiento General

1. **Inicialización** – Se carga el entorno con `SceneManager` y `Environment`.
2. **Creación de la caña** – `FishingRod` crea la caña de pescar interactiva.
3. **Simulación de peces** – `FishManager` posiciona y anima a los peces.
4. **Interacción del usuario** – Agarrar caña (`E`), lanzar (`ESPACIO`), atrapar (`F`).

---

## Controles y Uso

- **Modo Escritorio**

  - `WASD` / Flechas - Mover cámara.
  - `E` - Agarrar/Soltar caña.
  - `ESPACIO` - Lanzar línea.
  - `F` - Atrapar pez.
  - `R` - Reiniciar caña.
  - `Q` - Activar/Desactivar depuración.

- **Modo VR**
  - **Controlador derecho**:
    - `Trigger` – Agarrar y lanzar línea.
    - `Grip` – Recoger línea.

---

## Uso de Librerías Externas

### Ammo.js

Integrado para física y simulación de colisiones.

### three.js

Librería base para renderizado 3D.

### FBXLoader

Carga modelos `.fbx` de peces animados.

---

## Configuración y Formato de Código

**Configuración de Prettier**:

```json
{
  "singleQuote": true,
  "trailingComma": "all"
}
```

---

## Pasos para Ejecutar el Proyecto

1. **Clonar el repositorio**:

```bash
git clone https://github.com/gitfrandu4/vr-fishing.git
cd vr-fishing
```

2. **Ejecutar un servidor local** (Python o Live Server de VSCode):

```bash
python -m http.server 8080
```

3. **Abrir en el navegador**:

- Abre el navegador y navega a `http://localhost:8080`.

1. **Entrar en modo VR** (si es compatible):

- Usa el botón "Enter VR" que aparece en la esquina inferior derecha.
- Coloca el visor VR y disfruta de la experiencia.

---

🎯 **¡Buena pesca!**
