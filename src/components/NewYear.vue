<template>
  <div class="container" @mousemove="handleMouseMove">
    <canvas ref="canvasRef" class="fireworks-canvas"></canvas>

    <div class="ribbon-wrapper left-ribbon">
      <svg viewBox="0 0 200 200" xmlns="http://www.w3.org/2000/svg">
        <path fill="#2e7d32" d="M0,200 C50,150 100,180 150,120 L0,120 Z" />
        <path fill="#1b5e20" d="M0,200 L50,200 C30,180 10,190 0,160 Z" opacity="0.5"/>
      </svg>
    </div>
    <div class="ribbon-wrapper right-ribbon">
      <svg viewBox="0 0 200 200" xmlns="http://www.w3.org/2000/svg">
        <path fill="#2e7d32" d="M200,200 C150,150 100,180 50,120 L200,120 Z" />
        <path fill="#1b5e20" d="M200,200 L150,200 C170,180 190,190 200,160 Z" opacity="0.5"/>
      </svg>
    </div>

    <div class="decorations">
      <span class="star s1">✨</span>
      <span class="star s2">✦</span>
      <span class="star s3">✷</span>
      <span class="star s4">✨</span>
    </div>

    <div class="content-overlay">
      <div class="card-content" v-if="showText">
        
        <transition-group name="staggered-fade" tag="div" appear>
          <h2 key="1" class="greeting-text" style="--i: 1">Selamat<br>Tahun Baru</h2>
          
          <div key="2" class="year-wrapper" style="--i: 2">
            <h1 class="main-year">2026</h1>
          </div>

          <p key="3" class="message" style="--i: 3">
            Semoga tahun ini membawa kesuksesan, <br>
            kebahagiaan, dan pencapaian baru yang luar biasa.
          </p>
          
          <!-- <div key="4" class="btn-wrapper" style="--i: 4">
            <button @click="launchConfetti" class="celebrate-btn">
              <span>Rayakan!</span> 🎉
            </button>
          </div> -->
        </transition-group>

      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';

const canvasRef = ref(null);
const showText = ref(false);
let ctx = null;
let animationFrameId = null;
let fireworks = [];
let particles = [];

const colors = ['#FFD93D', '#4D96FF', '#FF6B6B', '#6BCB77', '#FF9F43'];

const canvasWidth = ref(window.innerWidth);
const canvasHeight = ref(window.innerHeight);

const random = (min, max) => Math.random() * (max - min) + min;
const randomColor = () => colors[Math.floor(Math.random() * colors.length)];

const mouseX = ref(0);
const mouseY = ref(0);
const handleMouseMove = (e) => {
  mouseX.value = (e.clientX - window.innerWidth / 2) * 0.02;
  mouseY.value = (e.clientY - window.innerHeight / 2) * 0.02;
};

class Particle {
  constructor(x, y, color) {
    this.x = x;
    this.y = y;
    this.color = color;
    this.velocity = {
      x: random(-5, 5),
      y: random(-5, 5)
    };
    this.alpha = 1;
    this.friction = 0.95;
    this.gravity = 0.06;
  }

  draw() {
    ctx.save();
    ctx.globalAlpha = this.alpha;
    ctx.beginPath();
    ctx.moveTo(this.x, this.y);
    ctx.arc(this.x, this.y, random(2, 5), 0, Math.PI * 2);
    ctx.fillStyle = this.color;
    ctx.fill();
    ctx.restore();
  }

  update() {
    this.velocity.x *= this.friction;
    this.velocity.y *= this.friction;
    this.velocity.y += this.gravity;
    this.x += this.velocity.x;
    this.y += this.velocity.y;
    this.alpha -= 0.012;
  }
}

class Firework {
  constructor(x, y, targetY, color) {
    this.x = x;
    this.y = y;
    this.targetY = targetY;
    this.color = color;
    this.velocity = { x: random(-2, 2), y: -12 - random(0, 4) }; 
    this.exploded = false;
  }

  draw() {
    ctx.beginPath();
    ctx.arc(this.x, this.y, 3, 0, Math.PI * 2);
    ctx.fillStyle = this.color;
    ctx.fill();
  }

  update() {
    this.y += this.velocity.y;
    this.velocity.y += 0.05; 
    if (this.velocity.y >= 0 || this.y <= this.targetY) {
      this.exploded = true;
      createParticles(this.x, this.y, this.color);
    }
  }
}

const createParticles = (x, y, color) => {
  const particleCount = 50;
  for (let i = 0; i < particleCount; i++) {
    particles.push(new Particle(x, y, color || randomColor()));
  }
};

const animate = () => {
  ctx.fillStyle = 'rgba(251, 251, 247, 0.25)'; 
  ctx.fillRect(0, 0, canvasWidth.value, canvasHeight.value);

  fireworks.forEach((fw, index) => {
    fw.update();
    fw.draw();
    if (fw.exploded) fireworks.splice(index, 1);
  });

  particles.forEach((p, index) => {
    p.update();
    p.draw();
    if (p.alpha <= 0) particles.splice(index, 1);
  });

  if (Math.random() < 0.03) { 
    const isLeft = Math.random() > 0.5;
    const x = isLeft ? random(0, canvasWidth.value * 0.2) : random(canvasWidth.value * 0.8, canvasWidth.value);
    const y = canvasHeight.value;
    const targetY = random(50, canvasHeight.value / 2.5);
    fireworks.push(new Firework(x, y, targetY, randomColor()));
  }

  animationFrameId = requestAnimationFrame(animate);
};

const launchConfetti = () => {
  const bursts = [0, 200, 400, 600];
  bursts.forEach(delay => {
    setTimeout(() => {
        const x = random(canvasWidth.value * 0.2, canvasWidth.value * 0.8);
        const y = canvasHeight.value;
        const targetY = random(100, canvasHeight.value / 3);
        fireworks.push(new Firework(x, y, targetY, randomColor()));
        fireworks.push(new Firework(x, y, targetY, randomColor())); 
    }, delay);
  });
};

const handleResize = () => {
  canvasWidth.value = window.innerWidth;
  canvasHeight.value = window.innerHeight;
  if(canvasRef.value) {
      canvasRef.value.width = canvasWidth.value;
      canvasRef.value.height = canvasHeight.value;
  }
};

onMounted(() => {
  if (canvasRef.value) {
    ctx = canvasRef.value.getContext('2d');
    canvasRef.value.width = canvasWidth.value;
    canvasRef.value.height = canvasHeight.value;
    animate();
    setTimeout(() => { showText.value = true; }, 300);
  }
  window.addEventListener('resize', handleResize);
});

onUnmounted(() => {
  window.removeEventListener('resize', handleResize);
  cancelAnimationFrame(animationFrameId);
});
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Pacifico&family=Quicksand:wght@500;700&display=swap');

.container {
  position: relative;
  width: 100%;
  height: 100vh;
  overflow: hidden;
  background-color: #fbfbf7;
  background-image: radial-gradient(#d7d7d7 1px, transparent 1px);
  background-size: 25px 25px;
}

.fireworks-canvas {
  position: absolute;
  top: 0;
  left: 0;
  z-index: 1;
}

/* --- ANIMASI PITA BERGOYANG --- */
.ribbon-wrapper {
  position: absolute;
  bottom: -30px;
  width: 280px;
  height: 280px;
  z-index: 2;
  pointer-events: none;
  filter: drop-shadow(2px 4px 6px rgba(0,0,0,0.1));
}

.left-ribbon {
  left: -30px;
  transform-origin: bottom left;
  animation: swayLeft 6s ease-in-out infinite alternate;
}

.right-ribbon {
  right: -30px;
  transform: scaleX(-1);
  transform-origin: bottom right;
  animation: swayRight 7s ease-in-out infinite alternate;
}

@keyframes swayLeft {
  0% { transform: rotate(-5deg); }
  100% { transform: rotate(5deg); }
}

@keyframes swayRight {
  0% { transform: scaleX(-1) rotate(-5deg); }
  100% { transform: scaleX(-1) rotate(5deg); }
}

/* --- KONTEN UTAMA --- */
.content-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 3;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.card-content {
  position: relative;
  z-index: 10;
}

.greeting-text {
  font-family: 'Quicksand', sans-serif;
  font-weight: 700;
  color: #388e3c;
  font-size: 2.8rem;
  line-height: 1.1;
  margin-bottom: -15px;
  letter-spacing: 1px;
}

/* --- ANIMASI TEKS MENGAMBANG (FLOAT) --- */
.year-wrapper {
  position: relative;
  display: inline-block;
}

.main-year {
  font-family: 'Pacifico', cursive;
  font-size: 10rem;
  margin: 0;
  color: #2e7d32;
  line-height: 1.2;
  text-shadow: 5px 5px 0px rgba(165, 214, 167, 0.4);
  transform: rotate(-3deg);
  animation: floatYear 4s ease-in-out infinite;
}

@keyframes floatYear {
  0%, 100% { transform: rotate(-3deg) translateY(0); }
  50% { transform: rotate(-3deg) translateY(-15px); }
}

.message {
  font-family: 'Quicksand', sans-serif;
  color: #546e7a;
  font-size: 1.2rem;
  font-weight: 500;
  margin-top: 0.5rem;
  margin-bottom: 2.5rem;
}

/* --- TOMBOL DENGAN HOVER GELOMBANG --- */
.celebrate-btn {
  font-family: 'Quicksand', sans-serif;
  font-size: 1.2rem;
  font-weight: 700;
  color: #2e7d32;
  background: white;
  border: 3px solid #2e7d32;
  padding: 14px 45px;
  border-radius: 50px;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275); /* Bouncy transition */
  box-shadow: 0 6px 0 #1b5e20; /* Efek tombol tebal 3D */
  position: relative;
  top: 0;
}

.celebrate-btn:hover {
  transform: translateY(-4px);
  box-shadow: 0 10px 0 #1b5e20;
  background: #f1f8e9;
}

.celebrate-btn:active {
  transform: translateY(4px); /* Efek tombol ditekan */
  box-shadow: 0 2px 0 #1b5e20;
}

/* --- DEKORASI BINTANG (SPARKLES) --- */
.decorations .star {
  position: absolute;
  color: #fdd835;
  font-size: 2rem;
  opacity: 0;
  animation: twinkle 3s infinite ease-in-out;
}
/* Posisi statis relatif ke layar untuk contoh ini, bisa diatur ulang */
.decorations .s1 { top: 20%; left: 20%; animation-delay: 0s; }
.decorations .s2 { top: 30%; right: 25%; animation-delay: 1s; font-size: 1.5rem; color: #4D96FF; }
.decorations .s3 { bottom: 30%; left: 25%; animation-delay: 2s; font-size: 2.5rem; color: #FF6B6B; }
.decorations .s4 { top: 15%; right: 15%; animation-delay: 0.5s; font-size: 1.2rem; }

@keyframes twinkle {
  0%, 100% { opacity: 0; transform: scale(0.5) rotate(0deg); }
  50% { opacity: 1; transform: scale(1.2) rotate(45deg); }
}

/* --- ENTRANCE ANIMATION (STAGGERED) --- */
/* Animasi masuk bertahap untuk teks */
.staggered-fade-enter-active {
  transition: all 0.8s ease-out;
  transition-delay: calc(0.1s * var(--i)); /* Delay berdasarkan index custom style */
}

.staggered-fade-enter-from {
  opacity: 0;
  transform: translateY(30px);
}
.staggered-fade-enter-to {
  opacity: 1;
  transform: translateY(0);
}

/* Responsive */
@media (max-width: 768px) {
  .greeting-text { font-size: 2rem; }
  .main-year { font-size: 6.5rem; }
  .ribbon-wrapper { width: 180px; height: 180px; }
  .message { font-size: 1rem; padding: 0 20px; }
}
</style>