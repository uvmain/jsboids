<template>
  <div>
    <canvas ref="boidsCanvas"></canvas>
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

<script>
export default {
  data() {
    return {
      canvas: null,
      ctx: null,
      boids: [],
      numberOfBoids: 50,
      globalVelocity: 5.0,
      globalSeparation: 20.0,
      globalCohesion: 30.0,
      globalAlignment: 50.0
    };
  },
  mounted() {
    // Initialize canvas and context
    this.canvas = this.$refs.boidsCanvas;
    this.ctx = this.canvas.getContext('2d');

    // Set canvas size to match the document's client width
    this.canvas.width = document.documentElement.clientWidth - 20;
    this.canvas.height = document.documentElement.clientHeight - 20;

    // Create a group of boids
    for (let i = 0; i < this.numberOfBoids; i++) {
      this.addBoid()
    }

    // Start the animation loop
    this.animate();

    // Listen for window resize events and update canvas size accordingly
    window.addEventListener('resize', this.handleResize);
  },
  methods: {
    addBoid() {
      const x = Math.random() * this.canvas.width;
      const y = Math.random() * this.canvas.height;
      const dx = Math.random() * 2 - 1;
      const dy = Math.random() * 2 - 1;
      const c = `#${Math.floor(Math.random() * 16777215).toString(16)}`;
      this.boids.push({ x, y, dx, dy, c });
    },
    removeBoid() {
      this.boids.pop()
    },
    updateNumberOfBoids() {
      const currentBoidsCount = this.boids.length;
      if (this.numberOfBoids > currentBoidsCount) {
        const boidsToAdd = this.numberOfBoids - currentBoidsCount;
        for (let i = 0; i < boidsToAdd; i++) {
          this.addBoid();
        }
      } else if (this.numberOfBoids < currentBoidsCount) {
        const boidsToRemove = currentBoidsCount - this.numberOfBoids;
        for (let i = 0; i < boidsToRemove; i++) {
          this.removeBoid();
        }
      }
    },
    handleResize() {
      // Update canvas size to match the document's client size
      this.canvas.width = document.documentElement.clientWidth - 20;
      this.canvas.height = document.documentElement.clientHeight - 20;

      // Ensure boids stay within the new canvas size
      for (const boid of this.boids) {
        boid.x = Math.min(boid.x, this.canvas.width);
        boid.y = Math.min(boid.y, this.canvas.height);
      }
    },
    updateBoid(boid) {
      // Update position based on velocity
      boid.x += boid.dx * this.globalVelocity;
      boid.y += boid.dy * this.globalVelocity;
      this.normaliseVelocity(boid)

      // Bounce off walls
      if (boid.x < 0 || boid.x > this.canvas.width) {
        boid.dx = -boid.dx;
      }
      if (boid.y < 0 || boid.y > this.canvas.height) {
        boid.dy = -boid.dy;
      }

      // Apply boid rules
      this.applySeparation(boid);
      this.applyAlignment(boid);
      this.applyCohesion(boid);
    },
    applySeparation(boid) {
      for (const otherBoid of this.boids) {
        const distance = Math.hypot(boid.x - otherBoid.x, boid.y - otherBoid.y);
        if (otherBoid !== boid && distance < this.globalSeparation) {
          // Move away from nearby boids
          boid.dx += (boid.x - otherBoid.x) / distance;
          boid.dy += (boid.y - otherBoid.y) / distance;
        }
      }
    },
    applyAlignment(boid) {
      let avgDx = 0;
      let avgDy = 0;
      let count = 0;

      for (const otherBoid of this.boids) {
        const distance = Math.hypot(boid.x - otherBoid.x, boid.y - otherBoid.y);

        if (otherBoid !== boid && distance < this.globalAlignment) {
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
    },
    applyCohesion(boid) {
      let avgX = 0;
      let avgY = 0;
      let count = 0;
      for (const otherBoid of this.boids) {
        const distance = Math.hypot(boid.x - otherBoid.x, boid.y - otherBoid.y);
        if (otherBoid !== boid && distance < this.globalCohesion) {
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
    },
    normaliseVelocity(boid) {
      // Calculate the magnitude of the velocity vector
      const magnitude = Math.sqrt(boid.dx ** 2 + boid.dy ** 2);
      // Normalize dx and dy
      const normalizedDx = boid.dx / magnitude;
      const normalizedDy = boid.dy / magnitude;
      // Update boid's velocity with normalized values
      boid.dx = normalizedDx;
      boid.dy = normalizedDy;
    },
    drawBoid(boid) {
      const size = 10; // Adjust the size of the triangle as needed

      // Calculate the angle of the boid's velocity
      const angle = Math.atan2(boid.dy, boid.dx);

      // Draw boid as a triangle
      this.ctx.save();
      this.ctx.translate(boid.x, boid.y);
      this.ctx.rotate(angle + Math.PI / 2); // Adjusted angle

      this.ctx.beginPath();
      this.ctx.moveTo(0, -size / 2);
      this.ctx.lineTo(size / 2, size / 2);
      this.ctx.lineTo(-size / 2, size / 2);
      this.ctx.closePath();

      this.ctx.fillStyle = boid.c;
      this.ctx.fill();
      this.ctx.restore();
    },
    animate() {
      this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
      // Update and draw each boid
      for (const boid of this.boids) {
        this.updateBoid(boid);
        this.drawBoid(boid);
      }

      requestAnimationFrame(this.animate);
    },
  },
  beforeDestroy() {
    // Remove the resize event listener when the component is destroyed
    window.removeEventListener('resize', this.handleResize);
  },
};
</script>

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