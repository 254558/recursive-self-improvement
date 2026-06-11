<template>
  <div class="spark-animation-container">
    <canvas ref="canvasRef"></canvas>
    <div class="headline">
      <h1>When AI <br> builds itself</h1>
      <p>Our progress toward recursive self-improvement, and its implications.</p>
    </div>
  </div>
</template>
<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
const canvasRef = ref(null);
let animationInstance = null;
const sparkPath = new Path2D("M18.3658 62.2435L36.7823 51.9165L37.0858 51.012L36.7823 50.5083H35.8716L32.7853 50.3206L22.2616 50.0389L13.1546 49.6634L4.30054 49.194L2.07438 48.7246L0 45.9551L0.202378 44.5938L2.07438 43.3264L4.75589 43.5611L10.6755 43.9836L19.5801 44.5938L26.0056 44.9693L35.568 45.9551H37.0858L37.2882 45.3448L36.7823 44.9693L36.3775 44.5938L27.1693 38.3507L17.2022 31.7789L11.9909 27.9767L9.20822 26.0522L7.79157 24.2684L7.18443 20.3254L9.71416 17.5089L13.1546 17.7436L14.0147 17.9783L17.5057 20.654L24.9431 26.4277L34.6573 33.5627L36.0739 34.7362L36.6444 34.3512L36.7317 34.079L36.0739 32.9994L30.8121 23.4704L25.1961 13.7537L22.6664 9.71675L22.0086 7.32277C21.7539 6.31812 21.6039 5.48695 21.6039 4.45938L24.4878 0.516349L26.1068 0L30.0026 0.516349L31.6216 1.92457L34.0502 7.46359L37.9459 16.1476L44.0173 27.9767L45.7881 31.4973L46.7494 34.7362L47.1036 35.722H47.7107V35.1587L48.2166 28.4931L49.1274 20.3254L50.0381 9.81063L50.3416 6.85336L51.8089 3.28586L54.7434 1.36128L57.0201 2.44092L58.8921 5.11655L58.6391 6.85336L57.5261 14.0822L55.3505 25.395L53.9338 32.9994H54.7434L55.7047 32.0136L59.5498 26.944L65.9753 18.8702L68.8086 15.6782L72.1479 12.1577L74.2729 10.4678H78.3204L81.2549 14.8802L79.9395 19.4335L75.7907 24.6909L72.3503 29.1503L67.4173 35.7593L64.3563 41.0732L64.6308 41.5116L65.3682 41.4487L76.499 39.0548L82.5198 37.9751L89.7042 36.7547L92.9423 38.2568L93.2964 39.8058L92.0316 42.9509L84.3412 44.8285L75.3354 46.6592L61.9245 49.8162L61.776 49.9356L61.9513 50.1956L67.9991 50.743L70.5795 50.8839H76.9038L88.6923 51.7757L91.7786 53.7942L93.6 56.282L93.2964 58.2066L88.5405 60.6006L82.1656 59.0985L67.2402 55.531L62.1302 54.2636H61.4218V54.6861L65.6718 58.8638L73.514 65.9049L83.2787 75.0114L83.7846 77.2646L82.5198 79.0483L81.2043 78.8606L72.6032 72.3827L69.264 69.4724L61.776 63.1354H61.2701V63.7926L62.9903 66.3274L72.1479 80.081L72.6032 84.3057L71.9455 85.667L69.5676 86.5119L66.9872 86.0425L61.5736 78.4851L56.0588 70.0357L51.6065 62.4313L51.0687 62.7708L48.419 91.0652L47.2048 92.5204L44.3715 93.6L41.9935 91.8162L40.7286 88.9059L41.9935 83.1322L43.5114 75.6217L44.7256 69.6602L45.8387 62.2435L46.5185 59.7659L46.4584 59.6001L45.9153 59.6914L40.3239 67.3601L31.824 78.8606L25.0949 86.0425L23.4759 86.6997L20.6932 85.2445L20.9462 82.6628L22.5146 80.3627L31.824 68.5336L37.44 61.1639L41.0595 56.9335L41.0243 56.3216L40.8245 56.3046L16.0891 72.4297L11.6874 72.993L9.76476 71.2092L10.0177 68.2989L10.9284 67.3601L18.3658 62.2435Z");
const colors = ["#D97757", "#E08B6E", "#DD8263", "#E29478", "#D17A60", "#E6A085"];
const solidColors = ["#CE6C4C", "#C9694C", "#D17052", "#CB6B4E"];
function createRng(seed) {
  return () => {
    seed |= 0;
    let t = Math.imul((seed = seed + 0x6d2b79f5 | 0) ^ seed >>> 15, 1 | seed);
    return (((t = t + Math.imul(t ^ t >>> 7, 61 | t) ^ t) ^ t >>> 14) >>> 0) / 0x100000000;
  };
}
function createMipmaps(canvas) {
  const mipmaps = [canvas];
  let current = canvas;
  while (current.width > 38) {
    const size = Math.max(19, current.width >> 1);
    const mip = document.createElement("canvas");
    mip.width = mip.height = size;
    const ctx = mip.getContext("2d");
    ctx.imageSmoothingEnabled = true;
    ctx.imageSmoothingQuality = "high";
    ctx.drawImage(current, 0, 0, size, size);
    mipmaps.push(mip);
    current = mip;
  }
  return mipmaps;
}
function getMipmap(mipmaps, targetSize) {
  for (let i = mipmaps.length - 1; i >= 0; i--) {
    if (mipmaps[i].width >= targetSize) return mipmaps[i];
  }
  return mipmaps[0];
}
class SparkCellularAnimation {
  constructor(canvas) {
    this.canvas = canvas;
    this.ctx = canvas.getContext("2d");
    this.running = false;
    this.animationId = null;
    this.dpr = 1;
    this.width = 0;
    this.height = 0;
    this.cells = new Map();
    this.agents = [];
    this.lastTick = 0;
    this.ticks = 0;
    this.doneAt = 0;
    this.phase = 0;
    this.velocity = 0;
    this.position = 0;
    this.lastTimestamp = 0;
    this.rng = createRng(0xc1a0de);
    this.observer = null;
    this.sparkCells = this.calculateSparkCells();
    this.neighborMap = this.buildNeighborMap();
    this.totalSparkCells = this.calculateTotalSparkCells();
    this.baseTextures = this.createBaseTextures();
    this.cellImages = this.createCellImages();
    this.fractalSolidMipmaps = this.createFractalSolidMipmaps();
    this.maxScale = 0;
    this.centerX = 0;
    this.centerY = 0;
    this.resizeHandler = () => this.resize();
    window.addEventListener('resize', this.resizeHandler);
    this.resize();
    this.setupIntersectionObserver();
  }
  calculateSparkCells() {
    const tempCanvas = document.createElement("canvas");
    tempCanvas.width = tempCanvas.height = 152;
    const ctx = tempCanvas.getContext("2d", { willReadFrequently: true });
    const path = new Path2D(sparkPath);
    const matrix = new DOMMatrix().scale(1 / 94, 1 / 94);
    const transformedPath = new Path2D();
    transformedPath.addPath(path, matrix);
    ctx.save();
    ctx.scale(152, 152);
    ctx.fillStyle = "#000";
    ctx.fill(transformedPath);
    ctx.globalCompositeOperation = "destination-out";
    ctx.lineWidth = 0.025;
    ctx.lineJoin = "round";
    ctx.strokeStyle = "#000";
    ctx.stroke(transformedPath);
    ctx.restore();
    const imageData = ctx.getImageData(0, 0, 152, 152).data;
    const cells = [];
    for (let y = 0; y < 19; y++) {
      for (let x = 0; x < 19; x++) {
        let count = 0;
        for (let dy = 0; dy < 8; dy++) {
          for (let dx = 0; dx < 8; dx++) {
            if (imageData[((8 * y + dy) * 152 + (8 * x + dx)) * 4 + 3] > 8) count++;
          }
        }
        if (count >= 11.52) {
          cells.push({ idx: 19 * y + x, gx: x, gy: y, dist: Math.hypot(x - 9, y - 9) });
        }
      }
    }
    cells.sort((a, b) => a.dist - b.dist || a.idx - b.idx);
    return cells;
  }
  buildNeighborMap() {
    const cellSet = new Set(this.sparkCells.map(c => c.idx));
    const map = new Map();
    for (const {idx, gx, gy} of this.sparkCells) {
      const neighbors = [];
      for (const [dx, dy] of [[1, 0], [-1, 0], [0, 1], [0, -1]]) {
        const nx = gx + dx, ny = gy + dy;
        if (nx >= 0 && ny >= 0 && nx < 19 && ny < 19 && cellSet.has(19 * ny + nx)) {
          neighbors.push(19 * ny + nx);
        }
      }
      map.set(idx, neighbors);
    }
    return map;
  }
  calculateTotalSparkCells() {
    const visited = new Set([180]);
    const queue = [180];
    while (queue.length) {
      for (const n of (this.neighborMap.get(queue.shift()) || [])) {
        if (!visited.has(n)) { visited.add(n); queue.push(n); }
      }
    }
    return visited.size;
  }
  createBaseTextures() {
    const textures = [];
    for (let i = 0; i < 12; i++) {
      const rng = createRng(9301 * (i + 1) + 49297);
      const c = document.createElement("canvas");
      c.width = c.height = 128;
      const ctx = c.getContext("2d");
      ctx.globalAlpha = 0.88;
      ctx.fillStyle = colors[rng() * colors.length | 0];
      ctx.fillRect(0, 0, 128, 128);
      for (let j = 0; j < 5; j++) {
        const x = 128 * rng(), y = 128 * rng(), r = 128 * (0.25 + 0.35 * rng());
        const grad = ctx.createRadialGradient(x, y, 0, x, y, r);
        grad.addColorStop(0, colors[rng() * colors.length | 0]);
        grad.addColorStop(1, "rgba(0,0,0,0)");
        ctx.globalAlpha = 0.14 + 0.14 * rng();
        ctx.fillStyle = grad;
        ctx.fillRect(0, 0, 128, 128);
      }
      ctx.globalAlpha = 0.06;
      for (let j = 0; j < 220; j++) {
        ctx.fillStyle = colors[rng() * colors.length | 0];
        const size = 1 + 2 * rng();
        ctx.fillRect(128 * rng(), 128 * rng(), size, size);
      }
      ctx.globalAlpha = 0.14;
      ctx.strokeStyle = "#CF6E50";
      ctx.lineWidth = 6.4;
      ctx.shadowColor = "#CF6E50";
      ctx.shadowBlur = 12.8;
      ctx.beginPath();
      ctx.roundRect(8.96, 8.96, 110.08, 110.08, 25.6);
      ctx.stroke();
      ctx.shadowBlur = 0;
      ctx.globalAlpha = 1;
      ctx.globalCompositeOperation = "destination-in";
      ctx.fillStyle = "#000";
      const a1 = rng() * Math.PI * 2, a2 = rng() * Math.PI * 2, a3 = rng() * Math.PI * 2;
      ctx.beginPath();
      for (let j = 0; j <= 56; j++) {
        const t = j / 56 * Math.PI * 2;
        const cos = Math.cos(t), sin = Math.sin(t);
        const r = 1 / Math.pow(cos ** 4 + sin ** 4, 0.25) * 58.88 * 
                 (1 + 0.03 * Math.sin(3 * t + a1) + 0.018 * Math.sin(7 * t + a2) + 0.01 * Math.sin(11 * t + a3));
        const px = 64 + cos * r, py = 64 + sin * r;
        j === 0 ? ctx.moveTo(px, py) : ctx.lineTo(px, py);
      }
      ctx.closePath();
      ctx.fill();
      ctx.globalCompositeOperation = "source-over";
      textures.push(c);
    }
    return textures;
  }
  renderGrid(seed, size, cells, count) {
    const c = document.createElement("canvas");
    c.width = c.height = size;
    const ctx = c.getContext("2d");
    const cellSize = size / 19;
    const innerSize = 0.78 * cellSize;
    const offset = (cellSize - innerSize) / 2;
    for (let i = 0; i < count; i++) {
      const { gx, gy } = cells[i];
      const hash = (0x466f45d * gx ^ 0x127409f * gy ^ 0x4f9ffb7 * seed) >>> 0;
      ctx.save();
      ctx.translate(gx * cellSize + offset + innerSize / 2, gy * cellSize + offset + innerSize / 2);
      ctx.scale(hash >> 4 & 1 ? -1 : 1, hash >> 5 & 1 ? -1 : 1);
      ctx.drawImage(this.baseTextures[hash % 12], -innerSize / 2, -innerSize / 2, innerSize, innerSize);
      ctx.restore();
    }
    ctx.globalCompositeOperation = "destination-in";
    ctx.fillStyle = "#000";
    ctx.beginPath();
    ctx.roundRect(0, 0, size, size, 0.18 * size);
    ctx.fill();
    return c;
  }
  renderSolid(seed) {
    const rng = createRng(0x165667b1 * seed >>> 0);
    const c = document.createElement("canvas");
    c.width = c.height = 152;
    const ctx = c.getContext("2d");
    ctx.beginPath();
    ctx.roundRect(0, 0, 152, 152, 27.36);
    ctx.fillStyle = solidColors[seed % solidColors.length];
    ctx.fill();
    ctx.save();
    ctx.clip();
    for (let i = 0; i < 4; i++) {
      const x = 152 * rng(), y = 152 * rng(), r = 152 * (0.35 + 0.35 * rng());
      const grad = ctx.createRadialGradient(x, y, 0, x, y, r);
      grad.addColorStop(0, solidColors[rng() * solidColors.length | 0]);
      grad.addColorStop(1, "rgba(0,0,0,0)");
      ctx.globalAlpha = 0.22;
      ctx.fillStyle = grad;
      ctx.fillRect(0, 0, 152, 152);
    }
    ctx.restore();
    return c;
  }
  createCellImages() {
    const images = [];
    for (let i = 0; i < 8; i++) {
      const cells = this.getGrowthCells(i + 1);
      images.push({
        full: createMipmaps(this.renderGrid(i + 1, 608, cells, 361)),
        flip: Array.from({length: 8}, (_, a) => this.renderGrid(i + 1, 152, cells, Math.ceil((a + 1) / 8 * 361))),
        solid: this.renderSolid(i + 1)
      });
    }
    return images;
  }
  getGrowthCells(seed) {
    const rng = createRng(0x9e3779b1 * seed >>> 0);
    const visited = new Set();
    const cells = [];
    const add = (x, y) => {
      const idx = 19 * y + x;
      if (!visited.has(idx)) { visited.add(idx); cells.push({gx: x, gy: y}); return true; }
      return false;
    };
    add(9, 9);
    for (let i = 0; i < 5 + (3 * rng() | 0); i++) {
      let x = 9, y = 9;
      for (let j = 0; j < 38; j++) {
        const opts = [[1, 0], [-1, 0], [0, 1], [0, -1]].filter(([dx, dy]) => {
          const nx = x + dx, ny = y + dy;
          return nx >= 0 && ny >= 0 && nx < 19 && ny < 19 && !visited.has(19 * ny + nx);
        });
        if (!opts.length) break;
        let best = opts[0], maxScore = -Infinity;
        for (const o of opts) {
          const s = Math.hypot(x + o[0] - 9, y + o[1] - 9) + 1.2 * rng();
          if (s > maxScore) { maxScore = s; best = o; }
        }
        x += best[0]; y += best[1];
        add(x, y);
        if (x === 0 || y === 0 || x === 18 || y === 18) break;
      }
    }
    const frontier = [];
    const addFrontier = (x, y) => {
      for (const [dx, dy] of [[1, 0], [-1, 0], [0, 1], [0, -1]]) {
        const nx = x + dx, ny = y + dy;
        if (nx >= 0 && ny >= 0 && nx < 19 && ny < 19 && !visited.has(19 * ny + nx)) {
          frontier.push({gx: nx, gy: ny});
        }
      }
    };
    for (const c of cells) addFrontier(c.gx, c.gy);
    while (cells.length < 361) {
      let cell;
      do {
        const idx = rng() * frontier.length | 0;
        cell = frontier[idx];
        frontier[idx] = frontier[frontier.length - 1];
        frontier.pop();
      } while (cell && visited.has(19 * cell.gy + cell.gx));
      if (!cell) break;
      add(cell.gx, cell.gy);
      addFrontier(cell.gx, cell.gy);
    }
    return cells;
  }
  createFractalSolidMipmaps() {
    const c = document.createElement("canvas");
    c.width = c.height = 608;
    const ctx = c.getContext("2d");
    const size = 32, inner = 24.96;
    for (const {gx, gy, idx} of this.sparkCells) {
      ctx.drawImage(this.cellImages[idx % 8].solid, gx * size + (size - inner) / 2, gy * size + (size - inner) / 2, inner, inner);
    }
    return createMipmaps(c);
  }
  resize() {
    this.dpr = Math.min(2, window.devicePixelRatio || 1);
    this.width = this.canvas.clientWidth;
    this.height = this.canvas.clientHeight;
    this.canvas.width = Math.round(this.width * this.dpr);
    this.canvas.height = Math.round(this.height * this.dpr);
    this.ctx.setTransform(this.dpr, 0, 0, this.dpr, 0, 0);
    this.ctx.imageSmoothingEnabled = true;
    this.ctx.imageSmoothingQuality = "high";
    const isDesktop = this.width >= 992;
    this.centerX = isDesktop ? 0.66 * this.width : 0.5 * this.width;
    this.centerY = isDesktop ? 0.5 * this.height : 0.62 * this.height;
    this.maxScale = isDesktop ? 0.58 * Math.min(0.6 * this.width, this.height) : 0.5 * Math.min(0.92 * this.width, 0.62 * this.height);
  }
  setupIntersectionObserver() {
    this.observer = new IntersectionObserver(([e]) => e.isIntersecting ? this.start() : this.stop(), { threshold: 0.1 });
    this.observer.observe(this.canvas);
  }
  start() {
    if (this.running) return;
    this.running = true;
    this.reset();
    this.lastTimestamp = performance.now();
    this.animate(this.lastTimestamp);
  }
  stop() { 
    this.running = false; 
    if (this.animationId) cancelAnimationFrame(this.animationId); 
  }
  reset() {
    const now = performance.now();
    this.cells = new Map();
    this.agents = [{ idx: 180, prev: 180, since: now, bornAt: now, dieAt: 0 }];
    this.lastTick = now; this.ticks = 0; this.doneAt = 0; this.phase = 0; this.velocity = 0;
    this.position = -0.04 * Math.log(19 / 0.78);
  }
  update(timestamp) {
    const dt = Math.min(50, timestamp - this.lastTimestamp);
    this.lastTimestamp = timestamp;
    this.updateCellularAutomaton(timestamp);
    this.updatePhysics(dt);
    const isFinal = this.phase + 1 >= 3;
    if (this.doneAt && timestamp - this.doneAt >= 500 && !isFinal) {
      const now = performance.now();
      this.cells = new Map();
      this.agents = [{ idx: 180, prev: 180, since: now, bornAt: -Infinity, dieAt: 0 }];
      this.lastTick = now; 
      this.ticks = 0; 
      this.doneAt = 0; 
      this.phase++;
    }
    if (isFinal && this.doneAt && timestamp - this.doneAt >= 2300 && Math.abs(this.velocity) < 1e-5) { this.stop(); return false; }
    return true;
  }
  updateCellularAutomaton(timestamp) {
    if (this.doneAt) return;
    const rng = this.rng;
    while (timestamp - this.lastTick >= 450) {
      this.lastTick += 450; this.ticks++;
      const tickTime = this.lastTick;
      const aliveSet = new Set(this.agents.filter(a => !a.dieAt).map(a => a.idx));
      const newAgents = [];
      for (const agent of this.agents) {
        if (agent.dieAt) { if (tickTime - agent.dieAt < 350) newAgents.push(agent); continue; }
        this.cells.set(agent.idx, tickTime);
        const neighbors = (this.neighborMap.get(agent.idx) || []).filter(n => !this.cells.has(n) && !aliveSet.has(n));
        if (!neighbors.length) { agent.dieAt = tickTime; newAgents.push(agent); continue; }
        const nextIdx = neighbors[rng() * neighbors.length | 0];
        agent.prev = agent.idx; agent.idx = nextIdx; agent.since = tickTime;
        aliveSet.add(nextIdx); newAgents.push(agent);
      }
      this.agents = newAgents;
      const available = this.getAvailableCells();
      const aliveCount = this.agents.filter(a => !a.dieAt).length;
      if (!available.length && !aliveCount) { this.doneAt = tickTime; break; }
      const target = Math.min(48, Math.max(1, Math.floor((this.ticks / (7 / (1 + 0.25 * this.phase))) ** 2)));
      let current = aliveCount;
      while (current < target && available.length) {
        const idx = rng() * available.length | 0;
        const cellIdx = available[idx];
        available[idx] = available[available.length - 1]; available.pop();
        this.agents.push({ idx: cellIdx, prev: cellIdx, since: tickTime, bornAt: tickTime, dieAt: 0 });
        aliveSet.add(cellIdx); current++;
      }
    }
  }
  getAvailableCells() {
    const available = [], visited = new Set();
    for (const cellIdx of this.cells.keys()) {
      for (const n of (this.neighborMap.get(cellIdx) || [])) {
        if (!this.cells.has(n) && !visited.has(n)) { visited.add(n); available.push(n); }
      }
    }
    return available;
  }
  updatePhysics(dt) {
    const T = Math.log(19 / 0.78);
    const cellRatio = this.cells.size / this.totalSparkCells;
    const isFinal = this.phase + 1 >= 3;
    const baseTarget = this.doneAt && !isFinal ? 1 : -0.04 + 0.34 * cellRatio;
    const target = (this.phase + baseTarget) * T;
    const steps = Math.max(1, Math.ceil(dt / 12));
    const subDt = dt / steps;
    for (let i = 0; i < steps; i++) {
      const force = 81e-8 * (target - this.position) - 0.00153 * this.velocity;
      this.velocity += force * subDt;
      this.position += this.velocity * subDt;
    }
  }
  draw(timestamp) {
    const ctx = this.ctx;
    const scale = this.maxScale * Math.exp(this.phase * Math.log(19 / 0.78) - this.position);
    ctx.fillStyle = "#FAF9F5";
    ctx.fillRect(0, 0, this.width, this.height);
    this.drawGrid(scale);
    this.drawCells(timestamp, scale);
    this.drawAgents(timestamp, scale);
  }
  drawGrid(scale) {
    const ctx = this.ctx;
    const drawDots = (gridSize) => {
      if (gridSize < 2 || (this.width / gridSize + 2) * (this.height / gridSize + 2) > 12000) return;
      const alpha = Math.max(0, Math.min(1, (gridSize - 2) / 7)) * (0.45 + 0.45 * Math.min(1, gridSize / 90));
      if (alpha < 0.01) return;
      ctx.fillStyle = "#C9C6BC"; ctx.globalAlpha = alpha;
      const startX = ((this.centerX + gridSize / 2) % gridSize + gridSize) % gridSize;
      const startY = ((this.centerY + gridSize / 2) % gridSize + gridSize) % gridSize;
      for (let y = startY; y <= this.height; y += gridSize) {
        for (let x = startX; x <= this.width; x += gridSize) {
          ctx.fillRect(x - 1, y - 1, 2, 2);
        }
      }
      ctx.globalAlpha = 1;
    };
    const mainGrid = scale / 19 * 2;
    drawDots(mainGrid);
    drawDots(0.78 * mainGrid / 19);
  }
  drawCells(timestamp, scale) {
    const ctx = this.ctx;
    const cellScale = scale / 19;
    if (cellScale < 0.4) return;
    const r = 2 * cellScale * 0.78;
    for (const [cellIdx, cellTime] of this.cells) {
      const gx = cellIdx % 19, gy = Math.floor(cellIdx / 19);
      const x = this.centerX + 2 * cellScale * (gx - 9);
      const y = this.centerY + 2 * cellScale * (gy - 9);
      const age = timestamp - cellTime;
      const o = this.cellImages[cellIdx % 8];
      if (age < 700) {
        const e = Math.min(7, age / 700 * 8 | 0);
        ctx.drawImage(o.flip[e], x - r / 2, y - r / 2, r, r);
      } else {
        const fadeProgress = Math.min(1, (age - 700) / 1600);
        if (fadeProgress < 1) {
          ctx.globalAlpha = 1 - fadeProgress;
          ctx.drawImage(getMipmap(o.full, r * this.dpr), x - r / 2, y - r / 2, r, r);
          ctx.globalAlpha = 1;
        }
        if (fadeProgress > 0) {
          ctx.globalAlpha = fadeProgress;
          ctx.drawImage(o.solid, x - r / 2, y - r / 2, r, r);
          ctx.globalAlpha = 1;
        }
      }
    }
  }
  drawAgents(timestamp, scale) {
    const ctx = this.ctx;
    const cellScale = scale / 19;
    if (cellScale < 0.4) return;
    for (const agent of this.agents) {
      const progress = Math.min(1, (timestamp - agent.since) / 450);
      const eased = 1 - (1 - progress) * (1 - progress);
      const prevX = this.centerX + 2 * cellScale * (agent.prev % 19 - 9);
      const prevY = this.centerY + 2 * cellScale * (Math.floor(agent.prev / 19) - 9);
      const currX = this.centerX + 2 * cellScale * (agent.idx % 19 - 9);
      const currY = this.centerY + 2 * cellScale * (Math.floor(agent.idx / 19) - 9);
      const x = prevX + (currX - prevX) * eased;
      const y = prevY + (currY - prevY) * eased;
      let opacity = agent.dieAt ? (1 - Math.min(1, (timestamp - agent.dieAt) / 350)) ** 2 : 1 - (1 - Math.min(1, (timestamp - agent.bornAt) / 350)) ** 3;
      if (opacity <= 0) continue;
      const u = cellScale * opacity;
      const size = 2 * u * 0.78;
      if (size < 0.4) continue;
      ctx.drawImage(getMipmap(this.fractalSolidMipmaps, 2 * size * 0.78 * this.dpr), x - size / 2, y - size / 2, size, size);
    }
  }
  animate(timestamp) {
    if (!this.running) return;
    if (this.update(timestamp)) {
      this.draw(timestamp);
      this.animationId = requestAnimationFrame(t => this.animate(t));
    }
  }
  destroy() {
    this.stop();
    window.removeEventListener('resize', this.resizeHandler);
    if (this.observer) {
      this.observer.disconnect();
    }
  }
}
onMounted(() => {
  if (canvasRef.value) {
    animationInstance = new SparkCellularAnimation(canvasRef.value);
    if (window.matchMedia("(prefers-reduced-motion: reduce)").matches) {
      animationInstance.stop();
    }
  }
});
onUnmounted(() => {
  if (animationInstance) {
    animationInstance.destroy();
  }
});
</script>
<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Lora:wght@400;700&display=swap');
.spark-animation-container {
  margin: 0;
  padding: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  width: 100vw;
  background-color: #FAF9F5;
  position: relative;
  overflow: hidden;
}
canvas {
  display: block;
  width: 100vw;
  height: 100vh;
}
.headline {
  position: absolute;
  top: 49%;
  left: 26%;
  transform: translate(-50%, -50%);
  text-align: left;
  pointer-events: none;
  z-index: 10;
  color: #1F1E1B;
  width: 40%;
}
.headline h1 {
  font-size: 5rem;
  margin: 0 0 0.5rem 0;
  font-weight: 500;
  font-family: 'Lora', serif;
  letter-spacing: -0.02em;
  line-height: 1.0; /* 数值越小行间距越小，按需调整 1.0~1.3 */
}
.headline p {
  font-size: 1.1rem;
  margin: 0.9rem 0 0 0; /* 增加上方间距，可改 0.2rem/0.4rem 微调 */
  opacity: 0.6;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  font-weight: 300;
}
@media (max-width: 991px) {
  .headline {
    left: 50%;
    top: 80%;
    transform: translate(-50%, -100%);
    text-align: center;
    width: 90%;
  }
  .headline h1 {
    font-size: 2rem;
  }
  .headline p {
    font-size: 1rem;
  }
}
</style>