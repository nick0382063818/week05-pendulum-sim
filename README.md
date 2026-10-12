<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Pendulum Simulator - Week 05</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    background: #1a1a24;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100vh;
    overflow: auto;
    font-family: 'Consolas', 'Courier New', monospace;
    color: #8888aa;
  }
  canvas {
    display: block;
    border-radius: 8px;
  }
  .info {
    position: fixed;
    bottom: 20px;
    font-size: 12px;
    letter-spacing: 1px;
    opacity: 0.5;
  }
  .params {
    position: fixed;
    top: 20px;
    right: 24px;
    font-size: 13px;
    line-height: 1.8;
    text-align: right;
    opacity: 0.6;
  }
  .params span { color: #66ccff; }
  .controls {
    display: flex;
    gap: 28px;
    align-items: center;
    margin-top: 16px;
    padding: 14px 24px;
    background: rgba(255, 255, 255, 0.03);
    border: 1px solid rgba(102, 204, 255, 0.1);
    border-radius: 8px;
  }
  .control-group {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 6px;
  }
  .control-group label {
    font-size: 11px;
    letter-spacing: 1px;
    opacity: 0.5;
  }
  .control-group .val {
    font-size: 13px;
    color: #66ccff;
    min-width: 52px;
    text-align: center;
  }
  input[type="range"] {
    -webkit-appearance: none;
    appearance: none;
    width: 120px;
    height: 4px;
    background: rgba(102, 204, 255, 0.15);
    border-radius: 2px;
    outline: none;
  }
  input[type="range"]::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    width: 14px;
    height: 14px;
    border-radius: 50%;
    background: #66ccff;
    cursor: pointer;
    box-shadow: 0 0 6px rgba(102, 204, 255, 0.5);
  }
  input[type="range"]::-moz-range-thumb {
    width: 14px;
    height: 14px;
    border-radius: 50%;
    background: #66ccff;
    cursor: pointer;
    border: none;
    box-shadow: 0 0 6px rgba(102, 204, 255, 0.5);
  }
  .btn-reset {
    padding: 8px 20px;
    font-family: 'Consolas', 'Courier New', monospace;
    font-size: 12px;
    letter-spacing: 1px;
    color: #1a1a24;
    background: #66ccff;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    transition: background 0.2s;
  }
  .btn-reset:hover {
    background: #88ddff;
  }
</style>
</head>
<body>
<canvas id="canvas"></canvas>
<div class="controls">
  <div class="control-group">
    <label>摆长 L (px)</label>
    <input type="range" id="sliderL" min="100" max="350" value="200" step="1">
    <span class="val" id="valL">200</span>
  </div>
  <div class="control-group">
    <label>重力 g (m/s²)</label>
    <input type="range" id="sliderG" min="1" max="25" value="9.8" step="0.1">
    <span class="val" id="valG">9.8</span>
  </div>
  <div class="control-group">
    <label>阻尼</label>
    <input type="range" id="sliderD" min="0" max="0.010" value="0.001" step="0.001">
    <span class="val" id="valD">0.001</span>
  </div>
  <button class="btn-reset" id="btnReset">重置摆动</button>
</div>
<div class="params">
  L = <span>200 px</span><br>
  g = <span>9.8 m/s²</span><br>
  θ₀ = <span>45°</span>
</div>
<div class="info">PENDULUM SIM · WEEK 05 · 肖惜文 7904126027</div>

<script>
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');

function resize() {
  canvas.width = window.innerWidth;
  canvas.height = Math.floor(window.innerHeight * 0.68);
}
resize();
window.addEventListener('resize', resize);

let L = 200;
let G = 9.8;
let DAMPING = 0.999;
const DT = 0.05;

let theta = (45 * Math.PI) / 180;
let omega = 0;

const pivotX = () => canvas.width / 2;
const pivotY = () => canvas.height * 0.28;

function physicsStep() {
  const alpha = -(G / L) * Math.sin(theta);
  omega += alpha * DT;
  omega *= DAMPING;
  theta += omega * DT;
}

function drawPivot(px, py) {
  ctx.beginPath();
  ctx.arc(px, py, 6, 0, Math.PI * 2);
  ctx.fillStyle = '#555566';
  ctx.fill();

  ctx.beginPath();
  ctx.moveTo(px - 30, py - 2);
  ctx.lineTo(px + 30, py - 2);
  ctx.strokeStyle = '#444455';
  ctx.lineWidth = 3;
  ctx.stroke();
}

function drawRod(px, py, bx, by) {
  const grad = ctx.createLinearGradient(px, py, bx, by);
  grad.addColorStop(0, '#555566');
  grad.addColorStop(1, '#66ccff');
  ctx.beginPath();
  ctx.moveTo(px, py);
  ctx.lineTo(bx, by);
  ctx.strokeStyle = grad;
  ctx.lineWidth = 2;
  ctx.stroke();
}

function drawBob(bx, by) {
  const radius = 18;

  for (let i = 4; i >= 1; i--) {
    ctx.beginPath();
    ctx.arc(bx, by, radius + i * 8, 0, Math.PI * 2);
    ctx.fillStyle = `rgba(102, 204, 255, ${0.02 * i})`;
    ctx.fill();
  }

  const glow = ctx.createRadialGradient(bx, by, 0, bx, by, radius * 2.5);
  glow.addColorStop(0, 'rgba(102, 204, 255, 0.35)');
  glow.addColorStop(0.5, 'rgba(102, 204, 255, 0.08)');
  glow.addColorStop(1, 'rgba(102, 204, 255, 0)');
  ctx.beginPath();
  ctx.arc(bx, by, radius * 2.5, 0, Math.PI * 2);
  ctx.fillStyle = glow;
  ctx.fill();

  const ballGrad = ctx.createRadialGradient(bx - 5, by - 5, 2, bx, by, radius);
  ballGrad.addColorStop(0, '#aaeeff');
  ballGrad.addColorStop(0.6, '#44aadd');
  ballGrad.addColorStop(1, '#226688');
  ctx.beginPath();
  ctx.arc(bx, by, radius, 0, Math.PI * 2);
  ctx.fillStyle = ballGrad;
  ctx.fill();

  ctx.beginPath();
  ctx.arc(bx - 5, by - 6, 5, 0, Math.PI * 2);
  ctx.fillStyle = 'rgba(255, 255, 255, 0.25)';
  ctx.fill();
}

function drawTrail(px, py) {
  const bx = px + L * Math.sin(theta);
  const by = py + L * Math.cos(theta);

  ctx.beginPath();
  ctx.arc(px, py + L, L, 0, Math.PI * 2);
  ctx.setLineDash([4, 8]);
  ctx.strokeStyle = 'rgba(102, 204, 255, 0.06)';
  ctx.lineWidth = 1;
  ctx.stroke();
  ctx.setLineDash([]);
}

function drawAngleArc(px, py) {
  const arcR = 40;
  const startAngle = Math.PI / 2;
  const endAngle = Math.PI / 2 - theta;

  ctx.beginPath();
  ctx.arc(px, py, arcR, Math.min(startAngle, endAngle), Math.max(startAngle, endAngle));
  ctx.strokeStyle = 'rgba(102, 204, 255, 0.2)';
  ctx.lineWidth = 1;
  ctx.stroke();

  const deg = ((theta * 180) / Math.PI).toFixed(1);
  const labelX = px + (arcR + 14) * Math.sin(theta / 2);
  const labelY = py + (arcR + 14) * Math.cos(theta / 2);
  ctx.font = '11px Consolas, monospace';
  ctx.fillStyle = 'rgba(102, 204, 255, 0.4)';
  ctx.fillText(deg + '°', labelX, labelY);
}

function render() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  const px = pivotX();
  const py = pivotY();
  const bx = px + L * Math.sin(theta);
  const by = py + L * Math.cos(theta);

  drawTrail(px, py);
  drawAngleArc(px, py);
  drawRod(px, py, bx, by);
  drawPivot(px, py);
  drawBob(bx, by);
}

function loop() {
  physicsStep();
  render();
  requestAnimationFrame(loop);
}

loop();

const sliderL = document.getElementById('sliderL');
const sliderG = document.getElementById('sliderG');
const sliderD = document.getElementById('sliderD');
const valL = document.getElementById('valL');
const valG = document.getElementById('valG');
const valD = document.getElementById('valD');
const btnReset = document.getElementById('btnReset');
const paramSpans = document.querySelectorAll('.params span');

function syncParams() {
  paramSpans[0].textContent = L + ' px';
  paramSpans[1].textContent = G + ' m/s²';
}

sliderL.addEventListener('input', () => {
  L = Number(sliderL.value);
  valL.textContent = L;
  syncParams();
});

sliderG.addEventListener('input', () => {
  G = Number(sliderG.value);
  valG.textContent = G;
  syncParams();
});

sliderD.addEventListener('input', () => {
  const d = Number(sliderD.value);
  DAMPING = 1 - d;
  valD.textContent = d.toFixed(3);
});

btnReset.addEventListener('click', resetAnimation);

function resetAnimation() {
  theta = (45 * Math.PI) / 180;
  omega = 0;
}
</script>
</body>
</html>
