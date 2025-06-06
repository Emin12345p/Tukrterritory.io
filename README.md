<!DOCTYPE html>
<html lang="tr">
<head>
  <meta charset="UTF-8">
  <title>TürkoTerritory - Gerçek Harita</title>
  <style>
    body { margin: 0; overflow: hidden; font-family: sans-serif; }
    canvas { display: block; background: #f0f0f0; }
    #menu {
      position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
      background: white; padding: 20px; box-shadow: 0 0 20px rgba(0,0,0,0.3);
      text-align: center; border-radius: 10px;
    }
    #controls {
      position: absolute; top: 10px; left: 10px;
      background: white; padding: 10px; border-radius: 8px;
    }
    button { margin: 5px; padding: 5px 10px; }
  </style>
</head>
<body>
<div id="menu">
  <h2>Ülke Seç</h2>
  <input id="nameInput" placeholder="Ülke adı" value="TürkoDev" /><br><br>
  <input type="color" id="colorInput" value="#007bff" /><br><br>
  <button onclick="startGame()">Başla</button>
</div>

<div id="controls" style="display:none">
  <b>Asker Gönder:</b>
  <button onclick="setPercent(0.25)">%25</button>
  <button onclick="setPercent(0.5)">%50</button>
  <button onclick="setPercent(1)">%100</button>
</div>

<canvas id="game"></canvas>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");
canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

let sendPercent = 0.5;
function setPercent(p) {
  sendPercent = p;
}

let player = {};
let bots = [];
let units = [];

function startGame() {
  const name = document.getElementById("nameInput").value;
  const color = document.getElementById("colorInput").value;
  document.getElementById("menu").style.display = "none";
  document.getElementById("controls").style.display = "block";

  player = { x: 400, y: 300, size: 50, color: color, name: name };
  bots = [];
  for (let i = 0; i < 5; i++) {
    bots.push({
      x: Math.random() * canvas.width,
      y: Math.random() * canvas.height,
      size: 30 + Math.random() * 20,
      color: "red",
      name: "Bot" + (i + 1)
    });
  }
  gameLoop();
}

function drawMap() {
  ctx.strokeStyle = "#333";
  ctx.lineWidth = 1;

  // Basit dünya haritası (Afrika, Avrupa, Amerika vs - basit çizim)
  ctx.beginPath();
  ctx.moveTo(100, 400); ctx.lineTo(300, 300); ctx.lineTo(500, 350); ctx.lineTo(700, 200);
  ctx.lineTo(900, 250); ctx.lineTo(800, 500); ctx.lineTo(600, 450); ctx.lineTo(400, 500);
  ctx.lineTo(200, 550); ctx.closePath();
  ctx.stroke();
}

function drawCircle(obj) {
  ctx.beginPath();
  ctx.arc(obj.x, obj.y, obj.size, 0, Math.PI * 2);
  ctx.fillStyle = obj.color;
  ctx.fill();
  ctx.fillStyle = "#000";
  ctx.font = "12px Arial";
  ctx.fillText(obj.name, obj.x - obj.size / 2, obj.y - obj.size - 5);
}

function update() {
  player.size += 0.03;
  bots.forEach(bot => {
    bot.size += 0.02;
    bot.x += (Math.random() - 0.5) * 1;
    bot.y += (Math.random() - 0.5) * 1;

    let dx = player.x - bot.x;
    let dy = player.y - bot.y;
    let dist = Math.sqrt(dx * dx + dy * dy);
    if (dist < player.size + bot.size) {
      if (player.size > bot.size) {
        player.size += bot.size * 0.2;
        bot.size = 0; bot.x = -9999;
      } else {
        alert("Kaybettin!");
        location.reload();
      }
    }
  });
}

function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  drawMap();
  drawCircle(player);
  bots.forEach(bot => {
    if (bot.size > 0) drawCircle(bot);
  });
}

function updateUnits() {
  units.forEach((u, i) => {
    let dx = u.tx - u.x;
    let dy = u.ty - u.y;
    let dist = Math.sqrt(dx * dx + dy * dy);
    let speed = 2;
    if (dist > speed) {
      u.x += (dx / dist) * speed;
      u.y += (dy / dist) * speed;
    } else {
      bots.forEach(bot => {
        let ddx = u.x - bot.x;
        let ddy = u.y - bot.y;
        let ddd = Math.sqrt(ddx * ddx + ddy * ddy);
        if (ddd < bot.size + u.size) {
          bot.size -= u.power * 0.3;
          if (bot.size <= 0) bot.size = 0;
        }
      });
      units.splice(i, 1);
    }
  });
}

function drawUnits() {
  units.forEach(u => {
    ctx.beginPath();
    ctx.arc(u.x, u.y, u.size, 0, Math.PI * 2);
    ctx.fillStyle = u.color;
    ctx.fill();
  });
}

canvas.addEventListener("click", (e) => {
  let rect = canvas.getBoundingClientRect();
  let tx = e.clientX - rect.left;
  let ty = e.clientY - rect.top;
  let amount = player.size * sendPercent;
  player.size *= (1 - sendPercent);
  units.push({
    x: player.x, y: player.y,
    tx: tx, ty: ty,
    size: amount * 0.2,
    power: amount,
    color: "cyan"
  });
});

function gameLoop() {
  update();
  updateUnits();
  draw();
  drawUnits();
  requestAnimationFrame(gameLoop);
}
</script>
</body>
</html>
