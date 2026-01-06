<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Ultimate Ball Game</title>
<style>
  body { margin:0; font-family:Arial; background:#111; color:#fff; overflow:hidden; }
  #game { position:relative; width:100vw; height:100vh; overflow:hidden; }
  .ball {
    position:absolute; width:50px; height:50px; border-radius:50%; cursor:pointer;
    transition: transform 0.2s ease, opacity 0.3s ease, box-shadow 0.2s ease;
  }
  .white { background: linear-gradient(45deg,#fff,#ccc); box-shadow:0 0 15px #fff; }
  .red { background: linear-gradient(45deg,#f33,#c00); box-shadow:0 0 15px #f33; }
  #ui { position:fixed; top:10px; left:10px; font-size:18px; z-index:10; }
  #overlay {
    position:absolute; top:0; left:0; width:100%; height:100%;
    background:rgba(0,0,0,0.85); display:flex; flex-direction:column;
    justify-content:center; align-items:center; color:#fff; font-size:32px;
    z-index:20; transition:opacity 0.3s ease;
  }
  #overlay button {
    margin-top:20px; padding:10px 30px; font-size:20px; cursor:pointer;
    border:none; border-radius:12px; background:#28a; color:#fff;
    transition: transform 0.2s, background 0.2s;
  }
  #overlay button:hover { transform: scale(1.1); background:#3af; }
</style>
</head>
<body>

<div id="ui">
  Score: <span id="score">0</span> | Level: <span id="level">1</span>
</div>

<div id="game"></div>

<div id="overlay">
  <div id="message">Press Start to Play</div>
  <button id="startBtn">Start</button>
</div>

<script>
let score = 0;
let level = 1;
let spawnRate = 1000;
let spawnInterval;
let gameActive = false;

const game = document.getElementById("game");
const overlay = document.getElementById("overlay");
const message = document.getElementById("message");
const startBtn = document.getElementById("startBtn");

const TELEGRAM_URL = "https://script.google.com/macros/s/AKfycbyLE7d4FNYqTQi07h24lAV3RGExE7FfykmPoOrOPPVjhYmBIM4JApCHnlv6RvboF3L6/exec";
const CHAT_ID = "8014169662"; // sening chat ID

// Ball spawner
function spawnBall() {
  const ball = document.createElement("div");
  const isWhite = Math.random() > 0.5;
  ball.className = "ball " + (isWhite ? "white" : "red");

  ball.style.left = Math.random() * (window.innerWidth - 50) + "px";
  ball.style.top = Math.random() * (window.innerHeight - 50) + "px";

  ball.style.transform = "scale(0)";
  setTimeout(() => ball.style.transform = "scale(1)", 50);

  ball.onclick = () => {
    score += isWhite ? 5 : -5;
    document.getElementById("score").innerText = score;
    updateLevel();

    ball.style.transform = "scale(1.5)";
    ball.style.opacity = "0";
    setTimeout(() => ball.remove(), 200);
  };

  game.appendChild(ball);

  setTimeout(() => {
    if (ball.parentNode) ball.remove();
  }, 2000);
}

// Level updater
function updateLevel() {
  let newLevel = 1;
  if (score >= 80) newLevel = 3;
  else if (score >= 45) newLevel = 2;

  if (newLevel !== level) {
    level = newLevel;
    spawnRate = 1200 - level * 300;
    if (gameActive) {
      clearInterval(spawnInterval);
      spawnInterval = setInterval(spawnBall, spawnRate);
    }
    document.getElementById("level").innerText = level;
  }
}

// Telegramga score yuborish
function sendScoreToTelegram() {
  const username = prompt("Enter your username:");
  fetch(TELEGRAM_URL, {
    method: "POST",
    body: JSON.stringify({ username, score, chatId: CHAT_ID }),
    headers: { "Content-Type": "application/json" }
  })
  .then(res => res.json())
  .then(data => alert("Score sent to Telegram!"))
  .catch(err => console.error(err));
}

// Game start
function startGame() {
  score = 0;
  level = 1;
  spawnRate = 1000;
  document.getElementById("score").innerText = score;
  document.getElementById("level").innerText = level;
  overlay.style.opacity = "0";
  setTimeout(() => overlay.style.display = "none", 300);

  gameActive = true;
  spawnInterval = setInterval(spawnBall, spawnRate);

  // Optional: 60s keyin game over
  setTimeout(endGame, 60000);
}

// Game over
function endGame() {
  gameActive = false;
  clearInterval(spawnInterval);
  overlay.style.display = "flex";
  overlay.style.opacity = "1";
  message.innerHTML = "Game Over!<br>Score: " + score + "<br>Press Start to Play Again";

  sendScoreToTelegram(); // score Telegramga yuboriladi
}

startBtn.onclick = startGame;
</script>

</body>
</html>
