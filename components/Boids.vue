<script setup lang="ts">
import * as PIXI from 'pixi.js';

const numberOfBoids = ref(50);
const globalScale = ref(1.0);
const globalVelocity = ref(5.0);
const globalSeparation = ref(20.0);
const globalCohesion = ref(30.0);
const globalAlignment = ref(50.0);
const boids = ref<any[]>([]);
const boidSprites = ref<PIXI.Sprite[]>([]);

function addBoid() {
  const x = Math.random() * pixiApp.renderer.width;
  const y = Math.random() * pixiApp.renderer.height;
  const dx = Math.random() * 2 - 1;
  const dy = Math.random() * 2 - 1;
  // const c = `#${Math.floor(Math.random() * 16777215).toString(16)}`;
  const tint = Math.floor(Math.random() * 0xFFFFFF);
  boids.value.push({ x, y, dx, dy, tint });

  // Create corresponding sprite and add it to boidSprites
  const boidSprite = PIXI.Sprite.from('/boid.png');
  boidSprite.x = x;
  boidSprite.y = y;
  boidSprite.tint = tint;
  boidSprite.anchor.set(0.5);
  boidSprite.scale.x = globalScale.value;
  boidSprite.scale.y = globalScale.value;
  pixiApp.stage.addChild(boidSprite);
  boidSprites.value.push(boidSprite);
};

function removeBoid() {
  boids.value.pop();
  // Remove the corresponding sprite
  const removedSprite = boidSprites.value.pop();
  pixiApp.stage.removeChild(removedSprite as any);
  removedSprite?.destroy()
};

function updateNumberOfBoids() {
  const currentBoidsCount = boids.value.length;
  if (numberOfBoids.value > currentBoidsCount) {
    const boidsToAdd = numberOfBoids.value - currentBoidsCount;
    for (let i = 0; i < boidsToAdd; i++) {
      addBoid();
    }
  } else if (numberOfBoids.value < currentBoidsCount) {
    const boidsToRemove = currentBoidsCount - numberOfBoids.value;
    for (let i = 0; i < boidsToRemove; i++) {
      removeBoid();
    }
  }
};

function updateScale() {
  for (const boid of boidSprites.value) {
    boid.scale.x = globalScale.value
    boid.scale.y = globalScale.value
  }
};

function handleResize() {
  // Ensure boids stay within the new canvas size
  for (const boid of boids.value) {
    boid.x = Math.min(boid.x, pixiApp.renderer.width);
    boid.y = Math.min(boid.y, pixiApp.renderer.height);
  }
};

function updateBoid(boid: any, index: number) {
  // Update position based on velocity
  boid.x += boid.dx * globalVelocity.value;
  boid.y += boid.dy * globalVelocity.value;
  normaliseVelocity(boid);

  // Bounce off walls
  if (boid.x < 0 || boid.x > pixiApp.renderer.width) {
    boid.dx = -boid.dx;
  }
  if (boid.y < 0 || boid.y > pixiApp.renderer.height) {
    boid.dy = -boid.dy;
  }

  // Apply boid rules
  applySeparation(boid);
  applyAlignment(boid);
  applyCohesion(boid);

  // Update the corresponding sprite position and rotation
  const sprite = boidSprites.value[index];
  sprite.x = boid.x;
  sprite.y = boid.y;
  sprite.rotation = Math.atan2(boid.dy, boid.dx) + 1.5708; // rotate 90 degress in radians

  // Update the corresponding sprite position
  boidSprites.value[index].x = boid.x;
  boidSprites.value[index].y = boid.y;
};

function applySeparation(boid: any) {
  for (const otherBoid of boids.value) {
    const distance = Math.hypot(boid.x - otherBoid.x, boid.y - otherBoid.y);
    if (otherBoid !== boid && distance < globalSeparation.value) {
      // Move away from nearby boids
      boid.dx += (boid.x - otherBoid.x) / distance;
      boid.dy += (boid.y - otherBoid.y) / distance;
    }
  }
};

function applyAlignment(boid: any) {
  let avgDx = 0;
  let avgDy = 0;
  let count = 0;

  for (const otherBoid of boids.value) {
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
};

function applyCohesion(boid: any) {
  let avgX = 0;
  let avgY = 0;
  let count = 0;
  for (const otherBoid of boids.value) {
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
};

function normaliseVelocity(boid: any) {
  // Calculate the magnitude of the velocity vector
  const magnitude = Math.sqrt(boid.dx ** 2 + boid.dy ** 2);
  // Normalize dx and dy
  const normalizedDx = boid.dx / magnitude;
  const normalizedDy = boid.dy / magnitude;
  // Update boid's velocity with normalized values
  boid.dx = normalizedDx;
  boid.dy = normalizedDy;
};

function animate() {
  // Update and draw each boid
  for (let i = 0; i < boids.value.length; i++) {
    updateBoid(boids.value[i], i);
  }

  requestAnimationFrame(() => animate());
};

// Initialize PixiJS
let pixiApp: PIXI.Application<HTMLCanvasElement>

onMounted(() => {
  // Initialize PixiJS
  pixiApp = new PIXI.Application({
    background: '#f0f0f0',
    width: window.innerWidth,
    height: window.innerHeight,
  });

  // Create a container element within the component
  const pixiContainer = document.createElement('div');
  pixiContainer.style.position = 'absolute';
  pixiContainer.style.top = '0';
  pixiContainer.style.left = '0';
  pixiContainer.style.width = '100%';
  pixiContainer.style.height = '100%';
  document.body.appendChild(pixiContainer);

  // Append the PixiJS view to the container
  pixiContainer.appendChild(pixiApp.view);

  // Create boid sprites
  for (let i = 0; i < numberOfBoids.value; i++) {
    addBoid();
  }

  // Start the animation loop
  animate();

  // Listen for window resize events and update PixiJS renderer size accordingly
  window.addEventListener('resize', handleResize);
});

onBeforeUnmount(() => {
  // Remove the resize event listener when the component is destroyed
  window.removeEventListener('resize', handleResize);
});
</script>

<template>
  <div>
    <div ref="pixiContainer"></div>
    <div class="controls">
      <div>
        <div>
          Scale: {{ globalScale }}
        </div>
        <input
          type="range"
          min="0.1"
          max="5.0"
          step="0.1"
          v-model="globalScale"
          @input="updateScale"
        />
      </div>
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

<style scoped>
.controls {
  display: flex;
  flex-direction: column;
  position: absolute;
  top: 10px;
  left: 10px;
  z-index: 1;
  background-color: #f0f0f0;
  padding: 10px;
  width: 10%;
  min-width: 80px;
}

.controls div {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
}

#pixiContainer canvas {
  display: block;
}
</style>
