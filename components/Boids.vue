<template>
  <div>
    <canvas ref="boidsCanvas"></canvas>
    <div class="controls">
      <div>
        <label :for="velocityInputRef">Global Velocity: {{ globalVelocity }}</label>
      </div>
      <input
        type="range"
        min="0.1"
        max="20"
        :id="velocityInputRef"
        v-model="globalVelocity"
        @input="updateVelocity"
      />
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
      globalVelocity: 5.0,
      velocityInputRef: 'velocityInput',
    };
  },
  mounted() {
    // Initialize canvas and context
    this.canvas = this.$refs.boidsCanvas;
    this.ctx = this.canvas.getContext('2d');

    // Set canvas size to match the document's client width
    this.canvas.width = document.documentElement.clientWidth;
    this.canvas.height = document.documentElement.clientHeight;

    // Create a group of boids
    for (let i = 0; i < 50; i++) {
      const x = Math.random() * this.canvas.width;
      const y = Math.random() * this.canvas.height;
      const dx = Math.random() * 2 - 1;
      const dy = Math.random() * 2 - 1;
      const c = `#${Math.floor(Math.random() * 16777215).toString(16)}`;
      this.boids.push({ x, y, dx, dy, c });
    }

    // Start the animation loop
    this.animate();

    // Listen for window resize events and update canvas size accordingly
    window.addEventListener('resize', this.handleResize);
  },
  methods: {
    handleResize() {
      // Update canvas size to match the document's client size
      this.canvas.width = document.documentElement.clientWidth;
      this.canvas.height = document.documentElement.clientHeight;
    },
    updateBoid(boid) {
      // Update position based on velocity
      boid.x += boid.dx * this.globalVelocity;
      boid.y += boid.dy * this.globalVelocity;

      // Bounce off walls
      if (boid.x < 0 || boid.x > this.canvas.width) {
        boid.dx = -boid.dx;
      }
      if (boid.y < 0 || boid.y > this.canvas.height) {
        boid.dy = -boid.dy;
      }
    },
    drawBoid(boid) {
      // Draw boid as a simple circle
      this.ctx.beginPath();
      this.ctx.arc(boid.x, boid.y, 5, 0, Math.PI * 2);
      this.ctx.fillStyle = boid.c;
      this.ctx.fill();
      this.ctx.closePath();
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
    updateVelocity() {
      // Ensure that the input is a valid number
      const parsedValue = parseFloat(this.globalVelocity);
      if (!isNaN(parsedValue)) {
        this.globalVelocity = parsedValue;
      }
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
  position: absolute;
  top: 10px;
  left: 10px;
  z-index: 1;
  background-color: rgba(255, 255, 255, 0.555);
  padding: 10px;
}

</style>