# memoria-colores
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Memoria de Colores 🎮</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }
    body {
      min-height: 100vh;
      font-family: Arial, sans-serif;
      background: linear-gradient(135deg, #10172a, #202c4a);
      color: white;
      display: flex;
      justify-content: center;
      align-items: center;
      padding: 20px;
    }
    .game {
      width: 100%;
      max-width: 440px;
      text-align: center;
      background: #172036;
      padding: 25px 20px;
      border-radius: 24px;
      border: 1px solid #35415e;
      box-shadow: 0 15px 40px #0005;
    }
    h1 {
      font-size: 30px;
      margin-bottom: 8px;
    }
    .description {
      color: #bdc9e3;
      font-size: 14px;
      line-height: 1.5;
      margin-bottom: 22px;
    }
    .stats {
      display: flex;
      justify-content: space-around;
      gap: 10px;
      margin-bottom: 18px;
    }
    .stat {
      background: #26324d;
      border-radius: 12px;
      padding: 12px 8px;
      flex: 1;
    }
    .stat span {
      display: block;
      color: #b9c7e3;
      font-size: 12px;
      margin-bottom: 5px;
    }
    .stat strong {
      font-size: 22px;
    }
    #message {
      min-height: 48px;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 16px;
      font-weight: bold;
      margin-bottom: 10px;
    }
    .board {
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      gap: 14px;
      max-width: 330px;
      margin: 0 auto 22px;
    }
    .color {
      width: 100%;
      aspect-ratio: 1;
      border: 4px solid #ffffff20;
      border-radius: 24px;
      cursor: pointer;
      transition: transform 0.12s, filter 0.12s;
      -webkit-tap-highlight-color: transparent;
    }
    .color:active {
      transform: scale(0.94);
    }
    .blue { background: #2684ff; }
    .red { background: #ff405b; }
    .green { background: #20c878; }
    .yellow { background: #ffca3a; }
    .color.lit {
      filter: brightness(1.65);
      transform: scale(1.04);
      border-color: white;
      box-shadow: 0 0 25px #ffffff55;
    }
    .color:disabled {
      cursor: default;
    }
    .main-button {
      width: 100%;
      padding: 15px;
      border: none;
      border-radius: 13px;
      background: #7867ff;
      color: white;
      font-size: 17px;
      font-weight: bold;
      cursor: pointer;
      transition: background 0.2s, transform 0.1s;
    }
    .main-button:hover {
      background: #9285ff;
    }
    .main-button:active {
      transform: scale(0.98);
    }
    .main-button:focus-visible,
    .color:focus-visible {
      outline: 3px solid white;
      outline-offset: 4px;
    }
    .tip {
      color: #9eacc9;
      font-size: 12px;
      margin-top: 17px;
      line-height: 1.5;
    }
    @media (max-width: 360px) {
      h1 { font-size: 25px; }
      .game { padding: 20px 14px; }
      .board { gap: 10px; }
      .color { border-radius: 18px; }
    }
  </style>
</head>
<body>
  <main class="game">
    <h1>🧠 Memoria de Colores</h1>
    <p class="description">
      ¡Mira los colores, recuerda el orden y repite la secuencia!
    </p>
    <div class="stats">
      <div class="stat">
        <span>RONDA</span>
        <strong id="round">0</strong>
      </div>
      <div class="stat">
        <span>RÉCORD</span>
        <strong id="best">0</strong>
      </div>
    </div>
    <div id="message" role="status" aria-live="polite">
      Pulsa comenzar para jugar.
    </div>
    <div class="board" id="board">
      <button class="color blue" data-color="0"
        aria-label="Azul" disabled></button>
      <button class="color red" data-color="1"
        aria-label="Rojo" disabled></button>
      <button class="color green" data-color="2"
        aria-label="Verde" disabled></button>
      <button class="color yellow" data-color="3"
        aria-label="Amarillo" disabled></button>
    </div>
    <button class="main-button" id="start">
      🎮 Comenzar juego
    </button>
    <p class="tip">
      Consejo: concéntrate en el orden de los colores.
      Cada ronda agrega un color más. ¡No te equivoques!
    </p>
  </main>
  <script>
    const colors = [...document.querySelectorAll(".color")];
    const message = document.getElementById("message");
    const roundDisplay = document.getElementById("round");
    const bestDisplay = document.getElementById("best");
    const startButton = document.getElementById("start");
    let sequence = [];
    let playerPosition = 0;
    let round = 0;
    let best = 0;
    let acceptingInput = false;
    let gameOver = false;
    let flashTimers = [];
    function wait(ms) {
      return new Promise(resolve => setTimeout(resolve, ms));
    }
    function clearFlashes() {
      flashTimers.forEach(clearTimeout);
      flashTimers = [];
      colors.forEach(color => color.classList.remove("lit"));
    }
    function lightColor(index, duration = 350) {
      const color = colors[index];
      color.classList.add("lit");
      const timer = setTimeout(() => {
        color.classList.remove("lit");
      }, duration);
      flashTimers.push(timer);
    }
    function enableColors(enabled) {
      colors.forEach(color => {
        color.disabled = !enabled;
      });
    }
    async function showSequence() {
      acceptingInput = false;
      enableColors(false);
      startButton.disabled = true;
      startButton.style.opacity = "0.6";
      message.textContent = "👀 ¡Memoriza los colores!";
      await wait(600);
      for (const index of sequence) {
        lightColor(index);
        await wait(550);
      }
      message.textContent = "👉 ¡Ahora repite la secuencia!";
      playerPosition = 0;
      acceptingInput = true;
      enableColors(true);
      startButton.disabled = false;
      startButton.style.opacity = "1";
      startButton.textContent = "↻ Reiniciar partida";
    }
    async function nextRound() {
      round++;
      roundDisplay.textContent = round;
      sequence.push(Math.floor(Math.random() * colors.length));
      await showSequence();
    }
    function startGame() {
      clearFlashes();
      sequence = [];
      playerPosition = 0;
      round = 0;
      gameOver = false;
      acceptingInput = false;
      roundDisplay.textContent = "0";
      message.textContent = "¡Prepárate!";
      startButton.textContent = "↻ Reiniciar partida";
      nextRound();
    }
    function finishGame() {
      acceptingInput = false;
      gameOver = true;
      enableColors(false);
      if (round > best) {
        best = round;
        bestDisplay.textContent = best;
      }
      message.textContent = `💀 ¡Fin del juego! Llegaste a la ronda ${round}.`;
      startButton.textContent = "🎮 Jugar de nuevo";
    }
    async function handleColorClick(event) {
      if (!acceptingInput || gameOver) return;
      const index = Number(event.currentTarget.dataset.color);
      lightColor(index, 180);
      if (index !== sequence[playerPosition]) {
        finishGame();
        return;
      }
      playerPosition++;
      if (playerPosition === sequence.length) {
        acceptingInput = false;
        enableColors(false);
        message.textContent = "✅ ¡Correcto! Siguiente ronda...";
        await wait(800);
        if (!gameOver) {
          nextRound();
        }
      }
    }
    colors.forEach(color => {
      color.addEventListener("click", handleColorClick);
    });
    startButton.addEventListener("click", startGame);
  </script>
</body>
</html>
