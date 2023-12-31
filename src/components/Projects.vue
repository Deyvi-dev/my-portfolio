<template>
  <div>
    <div class="container">
      <canvas ref="canvas"></canvas>
      <div class="content">
        <h1 class="title">Projetos</h1>
        <p class="construction">Página em Construção</p>
        <router-link to="/">Home</router-link>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from "vue"
import * as THREE from "three"

let renderer: THREE.WebGLRenderer,
  scene: THREE.Scene,
  camera: THREE.PerspectiveCamera,
  particles: THREE.Points,
  mouseX = 0,
  mouseY = 0

const canvas = ref<HTMLCanvasElement | null>(null)
const targetCameraPosition = { x: 0, y: 0, z: 0 }
let animationInProgress = false

onMounted(() => {
  // Configuração do Three.js
  renderer = new THREE.WebGLRenderer({ canvas: canvas.value! })
  renderer.setSize(window.innerWidth, window.innerHeight)

  scene = new THREE.Scene()
  camera = new THREE.PerspectiveCamera(
    100,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  )
  camera.position.z = 3

  const particleCount = 2500
  const particlesGeometry = new THREE.BufferGeometry()
  const particlesPositions = new Float32Array(particleCount * 3)

  for (let i = 0; i < particleCount; i++) {
    particlesPositions[i * 3] = (Math.random() - 0.5) * 10
    particlesPositions[i * 3 + 1] = (Math.random() - 0.5) * 10
    particlesPositions[i * 3 + 2] = (Math.random() - 0.5) * 10
  }

  particlesGeometry.setAttribute(
    "position",
    new THREE.BufferAttribute(particlesPositions, 3)
  )

  const particlesMaterial = new THREE.PointsMaterial({
    size: 0.01,
    color: 0xeeeeee,
  })
  particles = new THREE.Points(particlesGeometry, particlesMaterial)
  scene.add(particles)

  window.addEventListener("resize", handleResize)
  window.addEventListener("mousemove", handleMouseMove)
  window.addEventListener("deviceorientation", handleDeviceOrientation)
  animate()
})

onUnmounted(() => {
  // Limpar recursos ao desmontar o componente
  renderer.dispose()
  window.removeEventListener("resize", handleResize)
  window.removeEventListener("mousemove", handleMouseMove)
  window.removeEventListener("deviceorientation", handleDeviceOrientation)
})

function handleResize() {
  const container = canvas.value?.parentElement
  if (container) {
    const { width, height } = container.getBoundingClientRect()
    renderer.setSize(width, height)
    camera.aspect = width / height
    camera.updateProjectionMatrix()
  }
}

function handleMouseMove(event: MouseEvent) {
  const windowHalfX = window.innerWidth / 2
  const windowHalfY = window.innerHeight / 2

  mouseX = (event.clientX - windowHalfX) * 0.001
  mouseY = (event.clientY - windowHalfY) * 0.001
}

function handleDeviceOrientation(event: DeviceOrientationEvent) {
  if (event.gamma !== null && event.beta !== null) {
    mouseX = event.gamma * 0.05
    mouseY = event.beta * 0.05
  }
}

function animate() {
  requestAnimationFrame(animate)

  particles.rotation.x += 0.05 * (mouseY - particles.rotation.x)
  particles.rotation.y += 0.05 * (mouseX - particles.rotation.y)

  updateCameraPosition()

  renderer.render(scene, camera)
}

function updateCameraPosition() {
  if (animationInProgress) {
    const speed = 0.01
    camera.position.x += (targetCameraPosition.x - camera.position.x) * speed
    camera.position.y += (targetCameraPosition.y - camera.position.y) * speed
    camera.position.z += (targetCameraPosition.z - camera.position.z) * speed
  }
}
</script>

<style scoped>
.container {
  position: relative;
  width: 100%;
  height: 100vh;
  margin: 0;
  padding: 0;
}

canvas {
  display: block;
  width: 100%;
  height: 100%;
  margin: 0;
  padding: 0;
}

.content {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
}

.title {
  color: #ffffff;
  font-size: 3rem;
  margin-bottom: 0.5rem;
}

.construction {
  color: #ffffff;
  font-size: 1.5rem;
  margin-top: 2rem;
}
</style>
