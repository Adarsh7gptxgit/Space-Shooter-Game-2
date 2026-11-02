
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Space Shooter Game 2</title>

<style>
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  background: #030817;
  color: white;
  font-family: Arial, sans-serif;
  text-align: center;
  overflow: hidden;
}

h1 {
  color: #00eaff;
  text-shadow: 0 0 15px #00eaff;
}

#game {
  display: block;
  margin: auto;
  background: radial-gradient(#101d42, #020611);
  border: 2px solid #00d9ff;
  max-width: 100%;
  touch-action: none;
}

.info {
  display: flex;
  justify-content: space-around;
  padding: 10px;
  font-size: 18px;
}

button {
  background: #00cfff;
  border: none;
  padding: 12px 24px;
  margin: 8px;
  border-radius: 10px;
  color: #00101c;
  font-weight: bold;
  font-size: 16px;
}

.controls {
  display: flex;
  justify-content: center;
  gap: 30px;
}

#message {
  font-size: 20px;
  color: #ffcc00;
}
</style>
</head>

<body>

<h1>🚀 SPACE SHOOTER</h1>

<div class="info">
  <span>Score: <b id="score">0</b></span>
  <span>Lives: <b id="lives">3</b></span>
  <span>Level: <b id="level">1</b></span>
</div>

<canvas id="game" width="400" height="600"></canvas>

<div id="message">Defend the Galaxy!</div>

<button onclick="startGame()">START / RESTART</button>

<div class="controls">
  <button id="left">⬅ LEFT</button>
  <button id="fire">🔥 FIRE</button>
  <button id="right">RIGHT ➡</button>
</div>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

let score = 0;
let lives = 3;
let level = 1;
let running = false;

let player;
let bullets = [];
let enemies = [];
let stars = [];

let left = false;
let right = false;
let firing = false;
let fireCooldown = 0;
let spawnTimer = 0;

function resetGame() {
  score = 0;
  lives = 3;
  level = 1;

  player = {
    x: 180,
    y: 520,
    width: 40,
    height: 40,
    speed: 5
  };

  bullets = [];
  enemies = [];
  stars = [];

  for (let i = 0; i < 70; i++) {
    stars.push({
      x: Math.random() * 400,
      y: Math.random() * 600,
      speed: Math.random() * 3 + 1
    });
  }

  updateInfo();
}

function updateInfo() {
  document.getElementById("score").textContent = score;
  document.getElementById("lives").textContent = lives;
  document.getElementById("level").textContent = level;
}

function drawPlayer() {
  ctx.save();
  ctx.shadowBlur = 15;
  ctx.shadowColor = "#00ffff";

  ctx.fillStyle = "#00eaff";

  ctx.beginPath();
  ctx.moveTo(player.x + 20, player.y);
  ctx.lineTo(player.x + 40, player.y + 40);
  ctx.lineTo(player.x + 20, player.y + 30);
  ctx.lineTo(player.x, player.y + 40);
  ctx.closePath();
  ctx.fill();

  ctx.fillStyle = "white";
  ctx.fillRect(player.x + 17, player.y + 15, 6, 12);

  ctx.restore();
}

function shoot() {
  if (fireCooldown <= 0) {
    bullets.push({
      x: player.x + 18,
      y: player.y,
      speed: 8
    });

    fireCooldown = 12;
  }
}

function spawnEnemy() {
  enemies.push({
    x: Math.random() * 360,
    y: -40,
    size: 30,
    speed: 2 + level * 0.5
  });
}

function drawEnemy(enemy) {
  ctx.fillStyle = "#ff4055";
  ctx.shadowBlur = 12;
  ctx.shadowColor = "red";

  ctx.beginPath();
  ctx.arc(
    enemy.x + 15,
    enemy.y + 15,
    enemy.size / 2,
    0,
    Math.PI * 2
  );
  ctx.fill();

  ctx.shadowBlur = 0;
}

function collision(a, b) {
  return (
    a.x < b.x + b.size &&
    a.x + 40 > b.x &&
    a.y < b.y + b.size &&
    a.y + 40 > b.y
  );
}

function update() {
  if (!running) return;

  ctx.clearRect(0, 0, 400, 600);

  // Stars
  ctx.fillStyle = "white";

  stars.forEach(star => {
    star.y += star.speed;

    if (star.y > 600) {
      star.y = 0;
      star.x = Math.random() * 400;
    }

    ctx.fillRect(star.x, star.y, 2, 2);
  });

  // Player movement
  if (left && player.x > 0) {
    player.x -= player.speed;
  }

  if (right && player.x < 360) {
    player.x += player.speed;
  }

  if (firing) shoot();

  if (fireCooldown > 0) fireCooldown--;

  drawPlayer();

  // Bullets
  bullets.forEach((bullet, index) => {
    bullet.y -= bullet.speed;

    ctx.fillStyle = "#ffff00";
    ctx.fillRect(bullet.x, bullet.y, 5, 15);

    if (bullet.y < 0) {
      bullets.splice(index, 1);
    }
  });

  // Enemy spawning
  spawnTimer++;

  if (spawnTimer > Math.max(20, 60 - level * 4)) {
    spawnEnemy();
    spawnTimer = 0;
  }

  // Enemies
  enemies.forEach((enemy, ei) => {
    enemy.y += enemy.speed;
    drawEnemy(enemy);

    if (enemy.y > 600) {
      enemies.splice(ei, 1);
      lives--;
      updateInfo();

      if (lives <= 0) endGame();
      return;
    }

    if (collision(player, enemy)) {
      enemies.splice(ei, 1);
      lives--;
      updateInfo();

      if (lives <= 0) endGame();
      return;
    }

    // Bullet collision
    bullets.forEach((bullet, bi) => {
      if (
        bullet.x > enemy.x &&
        bullet.x < enemy.x + enemy.size &&
        bullet.y < enemy.y + enemy.size &&
        bullet.y > enemy.y
      ) {
        enemies.splice(ei, 1);
        bullets.splice(bi, 1);

        score += 10;
        level = Math.floor(score / 100) + 1;

        updateInfo();
      }
    });
  });

  if (running) {
    requestAnimationFrame(update);
  }
}

function startGame() {
  resetGame();
  running = true;

  document.getElementById("message").textContent =
    "Destroy the enemy spaceships!";

  update();
}

function endGame() {
  running = false;

  document.getElementById("message").textContent =
    "💥 GAME OVER! Final Score: " + score;
}

// Keyboard controls
document.addEventListener("keydown", e => {
  if (e.key === "ArrowLeft" || e.key === "a") left = true;
  if (e.key === "ArrowRight" || e.key === "d") right = true;
  if (e.code === "Space") firing = true;
});

document.addEventListener("keyup", e => {
  if (e.key === "ArrowLeft" || e.key === "a") left = false;
  if (e.key === "ArrowRight" || e.key === "d") right = false;
  if (e.code === "Space") firing = false;
});

// Mobile controls
function holdButton(id, action) {
  const btn = document.getElementById(id);

  btn.addEventListener("pointerdown", e => {
    e.preventDefault();
    action(true);
  });

  ["pointerup", "pointercancel", "pointerleave"].forEach(event => {
    btn.addEventListener(event, () => action(false));
  });
}

holdButton("left", value => left = value);
holdButton("right", value => right = value);
holdButton("fire", value => firing = value);

resetGame();
</script>

</body>
</html>
