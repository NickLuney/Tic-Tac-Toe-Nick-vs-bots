<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Tic Tac Toe – Vs Bot</title>

<style>
body {
  font-family: system-ui;
  background: #0f172a;
  color: white;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  margin: 0;
}

.game-container {
  background: #020617;
  padding: 20px;
  border-radius: 16px;
  text-align: center;
  width: 320px;
}

.scoreboard {
  display: flex;
  justify-content: space-between;
  font-size: 14px;
  margin-bottom: 8px;
}

.controls {
  margin: 10px 0;
}

select {
  background: #111827;
  color: white;
  border: none;
  padding: 6px;
  border-radius: 6px;
}

.board {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 6px;
  margin: 15px 0;
}

.cell {
  width: 90px;
  height: 90px;
  font-size: 32px;
  background: #111827;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
}

/* ✅ Win banner */
#announcement {
  position: fixed;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  background: #22c55e;
  color: #022c22;
  padding: 15px 20px;
  border-radius: 10px;
  font-weight: bold;
  display: none;
  z-index: 10;
}

canvas {
  position: fixed;
  top: 0;
  left: 0;
  pointer-events: none;
}
</style>
</head>

<body>

<div id="announcement">
  Nick's last day in ABI will Be June 26. I will be joining the Supervisor Team in the Victoria Contact Center
</div>

<canvas id="confetti"></canvas>

<div class="game-container">
  <h2>Tic Tac Toe</h2>

  <div class="scoreboard">
    <div>You: <strong id="score-x">0</strong></div>
    <div>Bot: <strong id="score-o">0</strong></div>
    <div>Draws: <strong id="score-d">0</strong></div>
  </div>

  <div class="controls">
    Difficulty:
    <select id="difficulty">
      <option value="easy">Easy</option>
      <option value="medium" selected>Medium</option>
      <option value="hard">Impossible</option>
    </select>
  </div>

  <div id="status">Your turn</div>
  <div class="board" id="board"></div>

  <button onclick="resetBoard()">Reset Board</button>
</div>

<script>
const boardEl = document.getElementById("board");
const statusEl = document.getElementById("status");
const banner = document.getElementById("announcement");
const difficultyEl = document.getElementById("difficulty");

const scoreXEl = document.getElementById("score-x");
const scoreOEl = document.getElementById("score-o");
const scoreDEl = document.getElementById("score-d");

let board = Array(9).fill(null);
let gameActive = true;
let scores = { X: 0, O: 0, D: 0 };

const wins = [
  [0,1,2],[3,4,5],[6,7,8],
  [0,3,6],[1,4,7],[2,5,8],
  [0,4,8],[2,4,6]
];

// Build board
for (let i = 0; i < 9; i++) {
  const cell = document.createElement("div");
  cell.className = "cell";
  cell.onclick = () => handleMove(i);
  boardEl.appendChild(cell);
}

function handleMove(i){
  if (!gameActive || board[i]) return;

  play(i, "X");
  if (!gameActive) return;

  setTimeout(() => {
    play(getBotMove(), "O");
  }, 300);
}

function play(i, player){
  board[i] = player;
  boardEl.children[i].textContent = player;

  if (checkWin(board, player)){
    gameActive = false;

    if (player === "X"){
      scores.X++;
      statusEl.textContent = "🎉 YOU WIN!";
      banner.style.display = "block";
      startConfetti();
    } else {
      scores.O++;
      statusEl.textContent = "🤖 Bot wins!";
    }
    updateScores();
    return;
  }

  if (board.every(c => c)){
    gameActive = false;
    scores.D++;
    statusEl.textContent = "Draw!";
    updateScores();
  }
}

function checkWin(b, p){
  return wins.some(([a,b1,c]) => b[a]===p && b[b1]===p && b[c]===p);
}

/* 🤖 BOT LOGIC */
function getBotMove(){
  const level = difficultyEl.value;

  if (level === "easy") return randomMove();

  if (level === "medium"){
    return Math.random() < 0.7 ? smartMove() : randomMove();
  }

  return minimaxMove(); // impossible
}

function randomMove(){
  const free = board.map((v,i)=>v?null:i).filter(v=>v!==null);
  return free[Math.floor(Math.random()*free.length)];
}

function smartMove(){
  // win or block
  for (let p of ["O","X"]){
    for (let i=0;i<9;i++){
      if (!board[i]){
        board[i]=p;
        if (checkWin(board,p)){
          board[i]=null;
          return i;
        }
        board[i]=null;
      }
    }
  }
  return randomMove();
}

/* ✅ MINIMAX (IMPOSSIBLE) */
function minimaxMove(){
  let bestScore = -Infinity;
  let move;

  board.forEach((c,i)=>{
    if (!c){
      board[i]="O";
      let score = minimax(board,0,false);
      board[i]=null;
      if (score>bestScore){
        bestScore=score;
        move=i;
      }
    }
  });
  return move;
}

function minimax(b, depth, isMax){
  if (checkWin(b,"O")) return 10-depth;
  if (checkWin(b,"X")) return depth-10;
  if (b.every(c=>c)) return 0;

  if (isMax){
    return Math.max(...b.map((c,i)=>{
      if (!c){
        b[i]="O";
        let v=minimax(b,depth+1,false);
        b[i]=null;
        return v;
      }
      return -Infinity;
    }));
  } else {
    return Math.min(...b.map((c,i)=>{
      if (!c){
        b[i]="X";
        let v=minimax(b,depth+1,true);
        b[i]=null;
        return v;
      }
      return Infinity;
    }));
  }
}

function updateScores(){
  scoreXEl.textContent = scores.X;
  scoreOEl.textContent = scores.O;
  scoreDEl.textContent = scores.D;
}

function resetBoard(){
  board.fill(null);
  gameActive = true;
  statusEl.textContent = "Your turn";
  banner.style.display = "none";
  stopConfetti();
  [...boardEl.children].forEach(c=>c.textContent="");
}

/* 🎉 CONFETTI */
const canvas = document.getElementById("confetti");
const ctx = canvas.getContext("2d");
let confetti=[], active=false;

function resize(){
  canvas.width=innerWidth;
  canvas.height=innerHeight;
}
resize(); onresize=resize;

function startConfetti(){
  active=true;
  confetti=Array.from({length:120},()=>({
    x:Math.random()*canvas.width,
    y:Math.random()*canvas.height-canvas.height,
    r:Math.random()*6+2,
    d:Math.random()*5+2,
    c:`hsl(${Math.random()*360},100%,50%)`
  }));
  animate();
}

function stopConfetti(){
  active=false;
  ctx.clearRect(0,0,canvas.width,canvas.height);
}

function animate(){
  if(!active)return;
  ctx.clearRect(0,0,canvas.width,canvas.height);
  confetti.forEach(p=>{
    ctx.beginPath();
    ctx.arc(p.x,p.y,p.r,0,Math.PI*2);
    ctx.fillStyle=p.c;
    ctx.fill();
    p.y+=p.d;
    if(p.y>canvas.height)p.y=-10;
  });
  requestAnimationFrame(animate);
}
</script>

</body>
</html>

