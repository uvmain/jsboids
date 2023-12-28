<script setup lang="ts">

const numberOfBoids = ref(50)
const globalVelocity = ref(5.0)
const globalSeparation = ref(20.0)
const globalCohesion = ref(30.0)
const globalAlignment = ref(50.0)

let canvas: HTMLCanvasElement
let ctx: any = null
let boids: any[] = []

type Boid = { x: number, y: number, dx: number, dy: number, c: string }

onMounted(() => {
  // Initialize canvas and context
  canvas = document.getElementById('boidsCanvas') as HTMLCanvasElement;
  console.log(`canvas: ${canvas}`)
  ctx = canvas.getContext('2d');

  // Set canvas size to match the document's client width
  canvas.width = document.documentElement.clientWidth - 20;
  canvas.height = document.documentElement.clientHeight - 20;

  // Create a group of boids
  for (let i = 0; i < numberOfBoids.value; i++) {
    addBoid()
  }

  // Start the animation loop
  animate();

  // Listen for window resize events and update canvas size accordingly
  window.addEventListener('resize', handleResize);
})

onMounted(() => {
  // Remove the resize event listener when the component is destroyed
  window.removeEventListener('resize', handleResize);
})

function addBoid() {
  const x = Math.random() * canvas.width;
  const y = Math.random() * canvas.height;
  const dx = Math.random() * 2 - 1;
  const dy = Math.random() * 2 - 1;
  const c = `#${Math.floor(Math.random() * 16777215).toString(16)}`;
  boids.push({ x, y, dx, dy, c });
}

function removeBoid() {
  boids.pop()
}

function updateNumberOfBoids() {
  const currentBoidsCount = boids.length;
  if (numberOfBoids.value > currentBoidsCount) {
    const boidsToAdd = numberOfBoids.value - currentBoidsCount;
    for (let i = 0; i < boidsToAdd; i++) {
      addBoid();
    }
  }
  else if (numberOfBoids.value < currentBoidsCount) {
    const boidsToRemove = currentBoidsCount - numberOfBoids.value;
    for (let i = 0; i < boidsToRemove; i++) {
      removeBoid();
    }
  }
}

function handleResize() {
  // Update canvas size to match the document's client size
  canvas.width = document.documentElement.clientWidth - 20;
  canvas.height = document.documentElement.clientHeight - 20;

  // Ensure boids stay within the new canvas size
  for (const boid of boids) {
    boid.x = Math.min(boid.x, canvas.width);
    boid.y = Math.min(boid.y, canvas.height);
  }
}

function updateBoid(boid: Boid) {
  // Update position based on velocity
  boid.x += boid.dx * globalVelocity.value;
  boid.y += boid.dy * globalVelocity.value;
  normaliseVelocity(boid)

  // Bounce off walls
  if (boid.x < 0 || boid.x > canvas.width) {
    boid.dx = -boid.dx;
  }
  if (boid.y < 0 || boid.y > canvas.height) {
    boid.dy = -boid.dy;
  }

  // Apply boid rules
  applySeparation(boid);
  applyAlignment(boid);
  applyCohesion(boid);
}

function applySeparation(boid: Boid) {
  for (const otherBoid of boids) {
    const distance = Math.hypot(boid.x - otherBoid.x, boid.y - otherBoid.y);
    if (otherBoid !== boid && distance < globalSeparation.value) {
      // Move away from nearby boids
      boid.dx += (boid.x - otherBoid.x) / distance;
      boid.dy += (boid.y - otherBoid.y) / distance;
    }
  }
}

function applyAlignment(boid: Boid) {
  let avgDx = 0;
  let avgDy = 0;
  let count = 0;

  for (const otherBoid of boids) {
    const distance = Math.hypot(boid.x - otherBoid.x, boid.y - otherBoid.y);

    if (otherBoid !== boid && distance < globalAlignment.value) {
      // Align with nearby boids
      avgDx += otherBoid.dx;
      avgDy += otherBoid.dy;
      count++;
    }
  }

  if (count > 0) {
    avgDx /= count;
    avgDy /= count;

    // Apply alignment
    boid.dx += (avgDx - boid.dx) * 0.1;
    boid.dy += (avgDy - boid.dy) * 0.1;
  }
}

function applyCohesion(boid: Boid) {
  let avgX = 0;
  let avgY = 0;
  let count = 0;
  for (const otherBoid of boids) {
    const distance = Math.hypot(boid.x - otherBoid.x, boid.y - otherBoid.y);
    if (otherBoid !== boid && distance < globalCohesion.value) {
      // Cohere towards the center of nearby boids
      avgX += otherBoid.x;
      avgY += otherBoid.y;
      count++;
    }
  }

  if (count > 0) {
    avgX /= count;
    avgY /= count;

    // Apply cohesion
    boid.dx += (avgX - boid.x) * 0.005;
    boid.dy += (avgY - boid.y) * 0.005;
  }
}

function normaliseVelocity(boid: Boid) {
  // Calculate the magnitude of the velocity vector
  const magnitude = Math.sqrt(boid.dx ** 2 + boid.dy ** 2);
  // Normalize dx and dy
  const normalizedDx = boid.dx / magnitude;
  const normalizedDy = boid.dy / magnitude;
  // Update boid's velocity with normalized values
  boid.dx = normalizedDx;
  boid.dy = normalizedDy;
}

function drawBoid(boid: Boid) {
  const size = 10; // Adjust the size of the triangle as needed

  // Calculate the angle of the boid's velocity
  const angle = Math.atan2(boid.dy, boid.dx);

  // Draw boid as a triangle
  ctx.save();
  ctx.translate(boid.x, boid.y);
  ctx.rotate(angle + Math.PI / 2); // Adjusted angle

  ctx.beginPath();
  ctx.moveTo(0, -size / 2);
  ctx.lineTo(size / 2, size / 2);
  ctx.lineTo(-size / 2, size / 2);
  ctx.closePath();

  ctx.fillStyle = boid.c;
  ctx.fill();
  ctx.restore();
}

function animate() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  // Update and draw each boid
  for (const boid of boids) {
    updateBoid(boid);
    drawBoid(boid);
  }
  requestAnimationFrame(animate);
}
</script>

<template>
  <div>
    <canvas id="boidsCanvas"></canvas>
    <div class="controls">
      <div>
        <div>
          Boids: {{ numberOfBoids }}
        </div>
        <input
          type="range"
          min="0"
          max="100"
          v-model="numberOfBoids"
          step="1"
          @input="updateNumberOfBoids" 
        />
      </div>
      <div>
        <div>
          Velocity: {{ globalVelocity }}
        </div>
        <input
          type="range"
          min="0.1"
          max="20"
          v-model="globalVelocity"
        />
      </div>
      <div>
        <div>
          Separation: {{ globalSeparation }}
        </div>
        <input
          type="range"
          min="0.1"
          max="100"
          v-model="globalSeparation"
        />
      </div>
      <div>
        <div>
          Cohesion: {{ globalCohesion }}
        </div>
        <input
          type="range"
          min="0.1"
          max="100"
          v-model="globalCohesion"
        />
      </div>
      <div>
        <div>
          Alignment: {{ globalAlignment }}
        </div>
        <input
          type="range"
          min="0.1"
          max="100"
          v-model="globalAlignment"
        />
      </div>
    </div>
  </div>
</template>

<style>
.controls {
  display: flex;
  flex-direction: column;
  position: absolute;
  top: 10px;
  left: 10px;
  z-index: 1;
  background-color: rgba(255, 255, 255, 0.644);
  padding: 10px;
  width: 10%;
  min-width: 80px;
}

.controls div {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
}
</style>