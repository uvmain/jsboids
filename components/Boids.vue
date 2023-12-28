<script setup lang="ts">

import pallete from '../pallet'

const numberOfBoids = ref(2000);
const globalGridPartitions = ref(10);
const globalVelocity = ref(5.0);
const globalSeparation = ref(100.0);
const globalCohesionDistance = ref(30.0);
const globalCohesionFactor = ref(0.7);
const globalAlignmentDistance = ref(40.0);
const globalAlignmentFactor = ref(0.7);
const trails = ref(true);
const boidSize = ref(30);
const jitterAmount = ref(20);
const mouseAttractionFactor = ref(0.1);

type Boid = { x: number, y: number, dx: number, dy: number, c: string };

let canvas: HTMLCanvasElement;
let ctx: CanvasRenderingContext2D;
let boids: Boid[] = [];
let grid: any[] = [];

onMounted(() => {
  // Initialize canvas and context
  canvas = document.getElementById('boidsCanvas') as HTMLCanvasElement;
  canvas.width = document.documentElement.clientWidth - 20;
  canvas.height = document.documentElement.clientHeight - 20;
  ctx = canvas.getContext('2d') as CanvasRenderingContext2D;
  
  // Create a group of boids
  for (let i = 0; i < numberOfBoids.value; i++) {
    addBoid()
  }

  // Start the animation loop
  animate();

  window.addEventListener('resize', handleResize);
  canvas.addEventListener('mousemove', handleMouseMove);
})

onUnmounted(() => {
  canvas.removeEventListener('mousemove', handleMouseMove);
  window.removeEventListener('resize', handleResize);
})

function handleMouseMove(event: MouseEvent) {
  const mouseX = event.clientX;
  const mouseY = event.clientY;
  for (const boid of boids) {
    const angleToMouse = Math.atan2(mouseY - boid.y, mouseX - boid.x);
    boid.dx += Math.cos(angleToMouse) * mouseAttractionFactor.value;
    boid.dy += Math.sin(angleToMouse) * mouseAttractionFactor.value;
    // Normalize the velocity
    normaliseVelocity(boid);
  }
}

function getRandomColor() {
  let randomIndex = 0
  while (randomIndex === 0) {
    randomIndex = Math.floor(Math.random() * pallete.length);
  }
  return `#${pallete[randomIndex]}`;
}

function addBoid() {
  const x = Math.random() * canvas.width;
  const y = Math.random() * canvas.height;
  const dx = Math.random() * 2 - 1;
  const dy = Math.random() * 2 - 1;
  const c = getRandomColor();
  boids.push({ x, y, dx, dy, c });
}

function clearGrid() {
  grid = [];
  const cols = Math.ceil(canvas.width / globalGridPartitions.value);
  const rows = Math.ceil(canvas.height / globalGridPartitions.value);
  for (let x = 0; x < cols; x++) {
    grid[x] = [];
    for (let y = 0; y < rows; y++) {
      grid[x][y] = [];
    }
  }
}

function addToGrid(boid: Boid) {
  const gridX = Math.max(0, Math.floor(Math.min(boid.x, canvas.width - 1) / globalGridPartitions.value));
  const gridY = Math.max(0, Math.floor(Math.min(boid.y, canvas.height - 1) / globalGridPartitions.value));

  if (gridX >= 0 && gridX < grid.length && gridY >= 0 && gridY < grid[0].length) {
    try {
      grid[gridX][gridY].push(boid);
    } catch (err) {
      console.log(`Error adding to grid: ${gridX} ${gridY}`);
      console.error(err);
    }
  } else {
    console.log(`Invalid indices: ${gridX}, ${gridY}`);
  }
}

function getNeighbors(boid: Boid): Boid[] {
  const gridX = Math.floor(Math.min(boid.x, canvas.width - 1) / globalGridPartitions.value);
  const gridY = Math.floor(Math.min(boid.y, canvas.height - 1) / globalGridPartitions.value);
  const neighbors = [];

  for (let i = -1; i <= 1; i++) {
    for (let j = -1; j <= 1; j++) {
      const neighborX = gridX + i;
      const neighborY = gridY + j;
      if (
        neighborX >= 0 &&
        neighborX < grid.length &&
        neighborY >= 0 &&
        neighborY < grid[0].length
      ) {
        neighbors.push(...grid[neighborX][neighborY]);
      }
    }
  }
  return neighbors;
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
  updateGrid()
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
  applyJitter(boid);
}

function applyJitter(boid: Boid) {
  if (Math.random() > 0.5)
    return
  let angleChange = (jitterAmount.value / 100) * (Math.random() * 2 - 1);
  const newAngle = Math.atan2(boid.dy, boid.dx) + angleChange;
  const speed = Math.sqrt(boid.dx ** 2 + boid.dy ** 2);
  boid.dx = speed * Math.cos(newAngle);
  boid.dy = speed * Math.sin(newAngle);
}

function applySeparation(boid: Boid) {
  const neighbors = getNeighbors(boid);
  for (const otherBoid of neighbors) {
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

  const neighbors = getNeighbors(boid);
  for (const otherBoid of neighbors) {
    const distance = Math.hypot(boid.x - otherBoid.x, boid.y - otherBoid.y);
    if (otherBoid !== boid && distance < globalAlignmentDistance.value) {
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
    boid.dx += (avgDx - boid.dx) * globalAlignmentFactor.value;
    boid.dy += (avgDy - boid.dy) * globalAlignmentFactor.value;
  }
}

function applyCohesion(boid: Boid) {
  let avgX = 0;
  let avgY = 0;
  let count = 0;
  const neighbors = getNeighbors(boid);
  for (const otherBoid of neighbors) {  
    const distance = Math.hypot(boid.x - otherBoid.x, boid.y - otherBoid.y);
    if (otherBoid !== boid && distance < globalCohesionDistance.value) {
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
    boid.dx += (avgX - boid.x) * (0.01 * globalCohesionFactor.value);
    boid.dy += (avgY - boid.y) * (0.01 * globalCohesionFactor.value);
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
  const size = boidSize.value;

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

function updateGrid() {
  clearGrid()
  for (const boid of boids) {
    addToGrid(boid);
  }
}

function animate() {
  if (!trails.value)
    ctx.clearRect(0, 0, canvas.width, canvas.height);
  // Update and draw each boid
  updateGrid()
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
          max="5000"
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
          Cohesion Distance: {{ globalCohesionDistance }}
        </div>
        <input
          type="range"
          min="0.1"
          max="100"
          v-model="globalCohesionDistance"
        />
      </div>
      <div>
        <div>
          Cohesion Factor: {{ globalCohesionFactor }}
        </div>
        <input
          type="range"
          min="0.0"
          max="1.0"
          step="0.1"
          v-model="globalCohesionFactor"
        />
      </div>
      <div>
        <div>
          Alignment Distance: {{ globalAlignmentDistance }}
        </div>
        <input
          type="range"
          min="0.1"
          max="100"
          v-model="globalAlignmentDistance"
        />
      </div>
      <div>
        <div>
          Alignment Factor: {{ globalAlignmentFactor }}
        </div>
        <input
          type="range"
          min="0.0"
          max="1.0"
          step="0.1"
          v-model="globalAlignmentFactor"
        />
      </div>
      <div>
        <div>
          Grid Partition Count: {{ globalGridPartitions }}
        </div>
        <input
          type="range"
          min="10"
          step="10"
          max="50"
          v-model="globalGridPartitions"
          @input="updateGrid"
        />
      </div>
      <div>
        <div>
          Jitter Amount: {{ jitterAmount }}
        </div>
        <input
          type="range"
          min="0"
          step="10"
          max="100"
          v-model="jitterAmount"
        />
      </div>
      <div>
        <div>
          Trails Enabled
        </div>
        <input
          type="checkbox" v-model="trails"
        />
      </div>
      <div>
        <div>
          Boid Size: {{ boidSize }}
        </div>
        <input
          type="range"
          min="1.0"
          max="100"
          step="1"
          v-model="boidSize"
        />
      </div>
      <div>
        <div>
          Mouse Attraction Factor: {{ mouseAttractionFactor }}
        </div>
        <input
          type="range"
          min="0.0"
          max="1.0"
          step="0.1"
          v-model="mouseAttractionFactor"
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