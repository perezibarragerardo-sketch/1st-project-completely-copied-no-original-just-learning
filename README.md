<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mathematical 3D Heart with Trails</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            background: #030712;
            font-family: sans-serif;
        }
        #info {
            position: absolute;
            bottom: 20px;
            width: 100%;
            text-align: center;
            color: #60defa;
            font-size: 14px;
            pointer-events: none;
            letter-spacing: 1px;
        }
    </style>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
</head>
<body>

<div id="info">Click Everywhere</div>

<script>
// ==========================================
// 1. ESCENA Y CÁMARA
// ==========================================
const scene = new THREE.Scene();

const camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000);
camera.position.set(0, 0, 34);

// Para crear la estela, activamos preserveDrawingBuffer: true y desactivamos autoClear
const renderer = new THREE.WebGLRenderer({ antialias: true, preserveDrawingBuffer: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
renderer.autoClearColor = false; // Permite acumular fotogramas para generar la estela
document.body.appendChild(renderer.domElement);

// Plano transparente superpuesto para desvanecer progresivamente los fotogramas anteriores (efecto de estela)
const fadeScene = new THREE.Scene();
const fadeCamera = new THREE.OrthographicCamera(-1, 1, 1, -1, 0, 1);
const fadeMaterial = new THREE.MeshBasicMaterial({
    color: 0x030712,
    transparent: true,
    opacity: 0.18 // Ajusta este valor: menor = estela más larga, mayor = estela más corta
});
const fadePlane = new THREE.Mesh(new THREE.PlaneGeometry(2, 2), fadeMaterial);
fadeScene.add(fadePlane);

// ==========================================
// 2. CONTROLES
// ==========================================
const controls = new THREE.OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;
controls.dampingFactor = 0.05;

// ==========================================
// 3. PALETA DE AZULES TURQUESA BRILLANTES
// ==========================================
const BLUE_SHADES = [
    0x00f5ff, // Turquesa neón brillante
    0x22d3ee, // Cian / Turquesa vibrante
    0x06b6d4, // Turquesa eléctrico
    0x38bdf8, // Azul cielo brillante
    0x00ced1, // Turquesa clásico intenso
    0x7dd3fc  // Celeste luminoso
];
// ==========================================
// 4. ECUACIÓN MATEMÁTICA DEL CORAZÓN 3D
// ==========================================
function getHeartPoint(t, scale = 0.72) {
    const x = 16 * Math.pow(Math.sin(t), 3);
    const y = 13 * Math.cos(t) - 5 * Math.cos(2 * t) - 2 * Math.cos(3 * t) - Math.cos(4 * t);
    return {
        x: x * scale,
        y: y * scale
    };
}

// ==========================================
// 5. INICIALIZAR PARTÍCULAS
// ==========================================
const PARTICLE_COUNT = 8500;
const heartGeometry = new THREE.BufferGeometry();

const currentPositions = new Float32Array(PARTICLE_COUNT * 3);
const scatterVelocities = new Float32Array(PARTICLE_COUNT * 3);
const targetPositions  = new Float32Array(PARTICLE_COUNT * 3);
const colors           = new Float32Array(PARTICLE_COUNT * 3);
const speeds           = new Float32Array(PARTICLE_COUNT);
const delays           = new Float32Array(PARTICLE_COUNT);

const heartCenterY = -2.3;

for (let i = 0; i < PARTICLE_COUNT; i++) {
    const i3 = i * 3;

    const t = Math.random() * Math.PI * 2;
    const r = Math.sqrt(Math.random());
    const u = (Math.random() - 0.5) * 2;
    const p = getHeartPoint(t);

    targetPositions[i3]     = p.x * r;
    targetPositions[i3 + 1] = heartCenterY + (p.y - heartCenterY) * r;
    targetPositions[i3 + 2] = u * 6 * Math.sqrt(Math.max(0, 1 - r * r));

    currentPositions[i3]     = (Math.random() - 0.5) * 35;
    currentPositions[i3 + 1] = -18 - Math.random() * 8;
    currentPositions[i3 + 2] = (Math.random() - 0.5) * 35;

    speeds[i] = 0.02 + Math.random() * 0.025;
    delays[i] = Math.random() * 140;

    const colHex = BLUE_SHADES[Math.floor(Math.random() * BLUE_SHADES.length)];
    const color = new THREE.Color(colHex);
    colors[i3]     = color.r;
    colors[i3 + 1] = color.g;
    colors[i3 + 2] = color.b;
}

heartGeometry.setAttribute("position", new THREE.BufferAttribute(currentPositions, 3));
heartGeometry.setAttribute("color", new THREE.BufferAttribute(colors, 3));

// ==========================================
// 6. MATERIAL Y SISTEMA DE PUNTOS
// ==========================================
const heartMaterial = new THREE.PointsMaterial({
    size: 0.18,
    vertexColors: true,
    transparent: true,
    opacity: 0.95,
    blending: THREE.AdditiveBlending,
    depthWrite: false
});

const particleSystem = new THREE.Points(heartGeometry, heartMaterial);
scene.add(particleSystem);

// ==========================================
// 7. EXPLOSIONES DE CLIC (BURSTS)
// ==========================================
const burstParticles = [];

function spawnHeartBurst(x, y) {
    const count = 140;
    const burstGeometry = new THREE.BufferGeometry();
    const burstPositions = new Float32Array(count * 3);
    const velocities = [];
    const burstColors = new Float32Array(count * 3);

    for (let i = 0; i < count; i++) {
        const idx = i * 3;
        burstPositions[idx]     = x;
        burstPositions[idx + 1] = y;
        burstPositions[idx + 2] = 0;

        const angle = Math.random() * Math.PI * 2;
        const speed = Math.random() * 0.18 + 0.04;
        velocities.push({
            x: Math.cos(angle) * speed,
            y: Math.sin(angle) * speed,
            z: (Math.random() - 0.5) * 0.12
        });

        const color = new THREE.Color(BLUE_SHADES[Math.floor(Math.random() * BLUE_SHADES.length)]);
        burstColors[idx]     = color.r;
        burstColors[idx + 1] = color.g;
        burstColors[idx + 2] = color.b;
    }

    burstGeometry.setAttribute("position", new THREE.BufferAttribute(burstPositions, 3));
    burstGeometry.setAttribute("color", new THREE.BufferAttribute(burstColors, 3));

    const burstMat = new THREE.PointsMaterial({
        size: 0.17,
        vertexColors: true,
        transparent: true,
        opacity: 1,
        blending: THREE.AdditiveBlending,
        depthWrite: false
    });

    const burst = new THREE.Points(burstGeometry, burstMat);
    scene.add(burst);
    burstParticles.push({ object: burst, velocities, life: 1 });
}

const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();
const clickPlane = new THREE.Plane(new THREE.Vector3(0, 0, 1), 0);

window.addEventListener("click", (event) => {
    mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
    mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
    raycaster.setFromCamera(mouse, camera);

    const point = new THREE.Vector3();
    if (raycaster.ray.intersectPlane(clickPlane, point)) {
        spawnHeartBurst(point.x, point.y);
    }
});

// ==========================================
// 8. BUCLE DE ANIMACIÓN
// ==========================================
let frameCount = 0;
let cycleFrame = 0;
let state = "ASSEMBLING";

const ASSEMBLE_DURATION = 260;
const HOLD_DURATION     = 180;
const DESTROY_DURATION  = 220;

const affected = new Uint8Array(PARTICLE_COUNT);

function pickHalf() {
    for (let i = 0; i < PARTICLE_COUNT; i++) {
        const y = targetPositions[i * 3 + 1] - heartCenterY;
        affected[i] = (y < 0) ? 1 : 0;
    }
}
pickHalf();

function setScatterVelocities() {
    for (let i = 0; i < PARTICLE_COUNT; i++) {
        const i3 = i * 3;
        scatterVelocities[i3]     = (Math.random() - 0.5) * 0.45;
        scatterVelocities[i3 + 1] = (Math.random() - 0.3) * 0.45;
        scatterVelocities[i3 + 2] = (Math.random() - 0.5) * 0.45;
    }
}
setScatterVelocities();

const vortexFall = new Float32Array(PARTICLE_COUNT);
for (let i = 0; i < PARTICLE_COUNT; i++) {
    vortexFall[i] = 0.03 + Math.random() * 0.10;
}

const puddleRadius = new Float32Array(PARTICLE_COUNT);
const puddleLift   = new Float32Array(PARTICLE_COUNT);
for (let i = 0; i < PARTICLE_COUNT; i++) {
    puddleRadius[i] = Math.sqrt(Math.random()) * 13;
    puddleLift[i]   = Math.random() * 0.5;
}

const VORTEX_CENTER_X = 0;
const VORTEX_CENTER_Z = 0;
const FLOOR_Y         = -18;
const SWIRL_BASE      = 0.01;
const SWIRL_POWER     = 0.35;
const WAVE_HEIGHT     = 0.25;

function animate() {
    requestAnimationFrame(animate);
    frameCount++;
    cycleFrame++;

    const pos = heartGeometry.attributes.position.array;

    // --- FASE 1: FORMACIÓN ---
    if (state === "ASSEMBLING") {
        for (let i = 0; i < PARTICLE_COUNT; i++) {
            const i3 = i * 3;
            if (cycleFrame > delays[i]) {
                pos[i3]     += (targetPositions[i3]     - pos[i3])     * speeds[i];
                pos[i3 + 1] += (targetPositions[i3 + 1] - pos[i3 + 1]) * speeds[i];
                pos[i3 + 2] += (targetPositions[i3 + 2] - pos[i3 + 2]) * speeds[i];
            }
        }
        if (cycleFrame >= ASSEMBLE_DURATION) {
            state = "HOLDING";
            cycleFrame = 0;
        }
    }
    // --- FASE 2: REPOSO / LATIDO ---
    else if (state === "HOLDING") {
        for (let i = 0; i < PARTICLE_COUNT; i++) {
            const i3 = i * 3;
            pos[i3]     += (targetPositions[i3]     - pos[i3])     * 0.1;
            pos[i3 + 1] += (targetPositions[i3 + 1] - pos[i3 + 1]) * 0.1;
            pos[i3 + 2] += (targetPositions[i3 + 2] - pos[i3 + 2]) * 0.1;
        }
        if (cycleFrame >= HOLD_DURATION) {
            state = "DESTROYING";
            cycleFrame = 0;
            setScatterVelocities();
            pickHalf();
        }
    }
    // --- FASE 3: DESTRUCCIÓN EN REMOLINO ---
    else if (state === "DESTROYING") {
        for (let i = 0; i < PARTICLE_COUNT; i++) {
            const i3 = i * 3;

            if (affected[i]) {
                pos[i3]     += scatterVelocities[i3];
                pos[i3 + 1] += scatterVelocities[i3 + 1];
                pos[i3 + 2] += scatterVelocities[i3 + 2];
                scatterVelocities[i3]     *= 0.90;
                scatterVelocities[i3 + 1] *= 0.90;
                scatterVelocities[i3 + 2] *= 0.90;

                const dx = pos[i3]     - VORTEX_CENTER_X;
                const dz = pos[i3 + 2] - VORTEX_CENTER_Z;
                let r     = Math.sqrt(dx * dx + dz * dz);
                let angle = Math.atan2(dz, dx);

                const ramp = Math.min(cycleFrame / 40, 1);
                angle += (SWIRL_BASE + SWIRL_POWER / (r + 2)) * ramp;

                r += (puddleRadius[i] - r) * 0.04;

                pos[i3]     = VORTEX_CENTER_X + Math.cos(angle) * r;
                pos[i3 + 2] = VORTEX_CENTER_Z + Math.sin(angle) * r;

                const waterY = FLOOR_Y + puddleLift[i] + Math.sin(frameCount * 0.08 + r * 0.6) * WAVE_HEIGHT;
                pos[i3 + 1] -= vortexFall[i] * ramp * 3;
                if (pos[i3 + 1] < waterY) {
                    pos[i3 + 1] = waterY;
                }
            } else {
                pos[i3]     += (targetPositions[i3]     - pos[i3])     * 0.1;
                pos[i3 + 1] += (targetPositions[i3 + 1] - pos[i3 + 1]) * 0.1;
                pos[i3 + 2] += (targetPositions[i3 + 2] - pos[i3 + 2]) * 0.1;
            }
        }

        if (cycleFrame >= DESTROY_DURATION) {
            state = "ASSEMBLING";
            cycleFrame = 0;
            for (let i = 0; i < PARTICLE_COUNT; i++) {
                delays[i] = Math.random() * 140;
            }
        }
    }

    heartGeometry.attributes.position.needsUpdate = true;

    const heartBeat = state === "DESTROYING" ? 1 : 1 + Math.sin(frameCount * 0.05) * 0.03;
    particleSystem.scale.set(heartBeat, heartBeat, heartBeat);

    particleSystem.rotation.y += 0.0015;
    particleSystem.rotation.x = Math.sin(frameCount * 0.002) * 0.03;

    // Explosiones de clic
    for (let b = burstParticles.length - 1; b >= 0; b--) {
        const bp = burstParticles[b];
        const arr = bp.object.geometry.attributes.position.array;
        for (let i = 0; i < bp.velocities.length; i++) {
            arr[i * 3]     += bp.velocities[i].x;
            arr[i * 3 + 1] += bp.velocities[i].y;
            arr[i * 3 + 2] += bp.velocities[i].z;
        }
        bp.object.geometry.attributes.position.needsUpdate = true;
        bp.life -= 0.02;
        bp.object.material.opacity = Math.max(bp.life, 0);
        if (bp.life <= 0) {
            scene.remove(bp.object);
            bp.object.geometry.dispose();
            bp.object.material.dispose();
            burstParticles.splice(b, 1);
        }
    }

    controls.update();

    // RENDERIZADO CON EFECTO DE ESTELA (MOTION BLUR TRAIL)
    renderer.render(fadeScene, fadeCamera); // Aplica una capa de sombra sobre la imagen previa
    renderer.render(scene, camera);         // Dibuja el estado actual encima
}

// ==========================================
// 9. AJUSTAR VENTANA
// ==========================================
window.addEventListener("resize", () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
});

animate();
</script>
</body>
</html># 1st-project-completely-copied-no-original-just-learning
Lighting Heart 
