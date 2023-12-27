<template>
  <div>
    <canvas ref="boidsCanvas" width="800" height="600"></canvas>
  </div>
</template>

<script>
export default {
  data() {
    return {
      canvas: null,
      ctx: null,
      boids: [],
    };
  },
  mounted() {
    // Initialize canvas and context
    this.canvas = this.$refs.boidsCanvas;
    this.ctx = this.canvas.getContext('2d');

    // Create a group of boids
    for (let i = 0; i < 50; i++) {
      const x = Math.random() * this.canvas.width;
      const y = Math.random() * this.canvas.height;
      const dx = Math.random() * 2 - 1;
      const dy = Math.random() * 2 - 1;
      this.boids.push({ x, y, dx, dy });
    }

    // Start the animation loop
    this.animate();
  },
  methods: {
    updateBoid(boid) {
      // Update position based on velocity
      boid.x += boid.dx;
      boid.y += boid.dy;

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
      this.ctx.fillStyle = '#000';
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
  },
};
</script>

<style>
/* Add your component styles here */
</style>
