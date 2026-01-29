# water-cycle-simulation
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Water Cycle with Ice and Snow</title>
<style>
  body {
    margin: 0;
    background: linear-gradient(#87ceeb, #e0f7ff);
    font-family: Arial, sans-serif;
    text-align: center;
  }
  canvas {
    display: block;
    margin: auto;
    background: linear-gradient(#87ceeb, #ffffff);
    max-width: 100%;
  }
  #controls {
    background: #ffffffcc;
    padding: 10px;
  }
  label {
    margin: 0 10px;
  }
  button {
    padding: 6px 12px;
    margin: 5px;
    cursor: pointer;
  }
</style>
</head>
<body>
<h2>🌎 Water Cycle Simulation with Ice & Snow ❄️</h2>
<canvas id="canvas" width="900" height="500"></canvas>
<div id="controls">
  <button onclick="toggleSimulation()">▶️ Start / ⏸️ Pause</button>
  <button onclick="forcePrecipitation()">🌧️/❄️ Force Precipitation</button>
  <label>🌡️ Temperature
    <input type="range" min="0" max="15" value="5" id="temp" />
  </label>
  <label>💨 Wind
    <input type="range" min="-5" max="5" value="0" id="wind" />
  </label>
</div>
<p>
☀️ Evaporation → ☁️ Condensation → 🌧️ Rain / ❄️ Snow → 🌊 Collection / 🧊 Ice
</p>

<script>
const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d");

let running = false;
let vapor = [];
let precipitation = [];
let clouds = [{ x: 300, y: 80 }];

function toggleSimulation() {
  running = !running;
}

function forcePrecipitation() {
  createPrecipitation(80);
}

function createVapor() {
  vapor.push({
    x: Math.random() * canvas.width,
    y: canvas.height - 60,
    alpha: 1,
  });
}

function createPrecipitation(amount = 30) {
  for (let i = 0; i < amount; i++) {
    precipitation.push({
      x: clouds[0].x + Math.random() * 200,
      y: clouds[0].y + 40,
      type: getPrecipitationType(),
    });
  }
}

function getPrecipitationType() {
  const temp = Number(document.getElementById("temp").value);
  return temp <= 4 ? "snow" : "rain";
}

function drawSun() {
  ctx.beginPath();
  ctx.arc(800, 80, 40, 0, Math.PI * 2);
  ctx.fillStyle = "gold";
  ctx.fill();
}

function drawWater() {
  const temp = Number(document.getElementById("temp").value);
  ctx.fillStyle = temp <= 2 ? "#a9d0f5" : "#1e90ff"; // Ice = lighter blue
  ctx.fillRect(0, canvas.height - 60, canvas.width, 60);
}

function drawCloud() {
  ctx.fillStyle = "#eee";
  ctx.beginPath();
  ctx.arc(clouds[0].x, clouds[0].y, 40, 0, Math.PI * 2);
  ctx.arc(clouds[0].x + 50, clouds[0].y, 50, 0, Math.PI * 2);
  ctx.arc(clouds[0].x + 100, clouds[0].y, 40, 0, Math.PI * 2);
  ctx.fill();
}

function drawSnowflake(x, y) {
  ctx.fillStyle = "white";
  ctx.beginPath();
  ctx.moveTo(x, y);
  for (let i = 0; i < 6; i++) {
    ctx.lineTo(x + 5 * Math.cos((i * Math.PI) / 3), y + 5 * Math.sin((i * Math.PI) / 3));
    ctx.moveTo(x, y);
  }
  ctx.fill();
}

function update() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  drawSun();
  drawCloud();
  drawWater();

  const temp = Number(document.getElementById("temp").value);
  const wind = Number(document.getElementById("wind").value);

  if (running && Math.random() < temp * 0.03) {
    createVapor();
  }

  vapor.forEach((v, i) => {
    v.y -= 1 + temp * 0.2;
    v.x += wind * 0.3;
    v.alpha -= 0.003;

    ctx.fillStyle = `rgba(255,255,255,${v.alpha})`;
    ctx.beginPath();
    ctx.arc(v.x, v.y, 5, 0, Math.PI * 2);
    ctx.fill();

    if (v.alpha <= 0) vapor.splice(i, 1);
  });

  clouds[0].x += wind * 0.2;
  if (clouds[0].x > canvas.width) clouds[0].x = -150;
  if (clouds[0].x < -150) clouds[0].x = canvas.width;

  if (running && vapor.length > 80) {
    createPrecipitation();
    vapor = [];
  }

  precipitation.forEach((p, i) => {
    p.y += 5;
    p.x += wind * 0.4;

    if (p.type === "rain") {
      ctx.fillStyle = "#0077ff";
      ctx.fillRect(p.x, p.y, 3, 10);
    } else {
      drawSnowflake(p.x, p.y);
    }

    if (p.y > canvas.height - 60) {
      precipitation.splice(i, 1);
    }
  });

  ctx.fillStyle = "black";
  ctx.font = "16px Arial";
  ctx.fillText("☀️ Evaporation", 50, canvas.height - 80);
  ctx.fillText("☁️ Condensation", clouds[0].x, clouds[0].y - 30);
  ctx.fillText("🌧️ Precipitation (Rain/Snow)", clouds[0].x + 10, clouds[0].y + 90);
  ctx.fillText("🌊 Collection / 🧊 Ice", 50, canvas.height - 20);

  requestAnimationFrame(update);
}

update();
</script>
</body>
</html>
