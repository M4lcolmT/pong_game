<script setup>
import { onMounted, ref, nextTick } from 'vue'

// Board settings
const canvas = ref(null)
const ctx = ref(null)
const scoreText = ref("0 : 0")
const gameWidth = 900
const gameHeight = 500

// Color settings
const boardBackground = "#0B1026"
const paddle1Color = "#4F8BFF"
const paddle2Color = "#db5656"       
const paddleBorder = "#6E9FFF"
const ballColor = "#FFFFFF"
const ballBorderColor = "#E2E8FF"

// Game configurations
let intervalID
let player1Score = 0
let player2Score = 0

// Ball configurations
const ballRadius = 12.5
let ballSpeed = 1
let ballX = gameWidth / 2
let ballY = gameHeight / 2
let ballXDirection = 0
let ballYDirection = 0

// Paddle configurations
const paddleSpeed = 50
let paddle1 = {
  width: 25,
  height: 100,
  x: 0,
  y: 200
}
let paddle2 = {
  width: 25,
  height: 100,
  x: gameWidth - 25,
  y: 200
}

function setupBoard() {
  clearBoard()
  drawPaddles()
  createBall()
}

const startGame = async () => {
  createBall()
  await nextTick(() => {
    intervalID = setInterval(() => {
      clearBoard()
      drawPaddles()
      moveBall()
      drawBall(ballX, ballY)
      checkCollision()
      checkWinner()
    }, 10)
  })
}

function clearBoard() {
  ctx.value.fillStyle = boardBackground
  ctx.value.fillRect(0, 0, gameWidth, gameHeight)
}

const drawPaddles = () => {
  ctx.value.strokeStyle = paddleBorder;
  ctx.value.fillStyle = paddle1Color;
  ctx.value.fillRect(paddle1.x, paddle1.y, paddle1.width, paddle1.height);

  ctx.value.fillStyle = paddle2Color;
  ctx.value.fillRect(paddle2.x, paddle2.y, paddle2.width, paddle2.height);
}

function createBall() {
  ballSpeed = 1
  if(Math.round(Math.random()) == 1) {
    ballXDirection = 1
  } else {
    ballXDirection = -1
  }
  if(Math.round(Math.random()) == 1){
    ballYDirection = Math.random() * 1; //more random directions
  }
  else{
    ballYDirection = Math.random() * -1; //more random directions
  }
  ballX = gameWidth / 2
  ballY = gameHeight / 2
  drawBall(ballX, ballY)
}

function moveBall() {
  ballX += (ballSpeed * ballXDirection)
  ballY += (ballSpeed * ballYDirection)
}

function drawBall(ballX, ballY) {
  ctx.value.fillStyle = ballColor
  ctx.value.strokeStyle = ballBorderColor
  ctx.value.lineWidth = 2
  ctx.value.beginPath()
  ctx.value.arc(ballX, ballY, ballRadius, 0, 2 * Math.PI)
  ctx.value.stroke()
  ctx.value.fill()
}

function checkCollision() {
  if(ballY <= 0 + ballRadius) {
    ballYDirection *= -1
  }
  if(ballY >= gameHeight - ballRadius) {
    ballYDirection *= -1
  }
  if(ballX <= 0) {
    player2Score++
    updateScore()
    createBall()
    resetPaddlePosition()
    return
  }
  if(ballX >= gameWidth) {
    player1Score+=1
    updateScore()
    createBall()
    resetPaddlePosition()
    return
  }
  if(ballX <= (paddle1.x + paddle1.width + ballRadius)) {
    if((ballY >= paddle1.y) && (ballY <= (paddle1.y + paddle1.height))) {
      ballX = (paddle1.x + paddle1.width) + ballRadius // prevent ball stuck
      ballXDirection *= -1
      ballSpeed += .5
    }
  }
  if(ballX >= (paddle2.x - ballRadius)) {
    if((ballY >= paddle2.y) && (ballY <= (paddle2.y + paddle1.height))) {
      ballX = paddle2.x - ballRadius // prevent ball stuck
      ballXDirection *= -1
      ballSpeed += .5
    }
  }
}

function changeDirection(event) {
  const keyPressed = event.keyCode
  const paddle1Up = 87
  const paddle1Down = 83
  const paddle2Up = 38
  const paddle2Down = 40

  switch(keyPressed) {
    case paddle1Up:
      if(paddle1.y > 0) {
        paddle1.y -= paddleSpeed
      }
      break
    case paddle1Down:
      if(paddle1.y < gameHeight - paddle1.height) {
        paddle1.y += paddleSpeed
      }
      break
    case paddle2Up:
      if(paddle2.y > 0) {
        paddle2.y -= paddleSpeed
      }
      break
    case paddle2Down:
      if(paddle2.y < gameHeight - paddle2.height) {
        paddle2.y += paddleSpeed
      }
      break
  }
}

function checkWinner() {
  if(player1Score == 5) {
    alert("Player 1 wins!")
    resetGame()
  }
  if(player2Score == 5) {
    alert("Player 2 wins!")
    resetGame()
  }
}

function updateScore() {
  scoreText.value = player1Score + " : " + player2Score
}

function resetPaddlePosition() {
  paddle1 = {
    width: 25,
    height: 100,
    x: 0,
    y: 200
  }
  paddle2 = {
    width: 25,
    height: 100,
    x: gameWidth - 25,
    y: 200
  }
}

function resetGame() {
  player1Score = 0
  player2Score = 0
  updateScore() // reset score
  resetPaddlePosition()
  ballX = 0
  ballY = 0
  ballXDirection = 0
  ballYDirection = 0
  clearInterval(intervalID)
  setupBoard()
}

onMounted(() => {
  // Variable initialization
  // Using onMounted lifecycle hook because 
  // we need to wait for the HTML components to be ready before calling them
  canvas.value = document.getElementById("gameBoard")
  ctx.value = canvas.value.getContext("2d")
  setupBoard()
  window.addEventListener('keydown', changeDirection);
})  

</script>

<template>  
  <div class="game-container">
    <h1 class="game-title">Pong Game</h1>
    
    <div class="canvas-container">
      <canvas 
        id="gameBoard" 
        :width="gameWidth" 
        :height="gameHeight"
      ></canvas>
      
      <div class="score-display">
        {{ scoreText }}
      </div>
    </div>

    <div class="controls-container">
      <div class="buttons">
        <button class="game-button" @click="startGame()">
          Start
        </button>
        <button class="game-button" @click="resetGame()">
          Reset
        </button>
      </div>

      <div class="controls-info">
        <div class="player-controls">
          <span>P1: W / S</span>
          <span>P2: ↑ / ↓</span>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.game-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2rem;
  z-index: 1;
}

.game-title {
  color: #fff;
  font-size: 1.5rem;
  font-weight: 600;
  margin: 0;
  text-align: center;
  opacity: 0.9;
}

.canvas-container {
  position: relative;
  padding: 1rem;
}

#gameBoard {
  border: 2px solid rgba(79, 139, 255, 0.2);
  border-radius: 4px;
  background-color: #0B1026;
  box-shadow: 0 0 40px rgba(79, 139, 255, 0.1);
}

.score-display {
  position: absolute;
  top: -40px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 2rem;
  color: #fff;
  font-weight: 200;
  letter-spacing: 0.2em;
  opacity: 0.9;
}

.controls-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: .5rem;
  width: 100%;
}

.buttons {
  display: flex;
  gap: 2rem;
}

.game-button {
  background: transparent;
  border: 1px solid rgba(79, 139, 255, 0.6);
  color: #4F8BFF;
  padding: 0.8rem 2rem;
  font-size: 0.9rem;
  font-weight: 600;
  letter-spacing: 0.2em;
  text-transform: uppercase;
  border-radius: 2px;
  cursor: pointer;
  transition: all 0.2s ease;
  border-radius: 8px;
  border-width: 2.5px;
}

.game-button:hover {
  background: rgba(79, 139, 255, 0.1);
  border-color: #4F8BFF;
  box-shadow: 0 0 20px rgba(79, 139, 255, 0.2);
}

.controls-info {
  margin-top: 1rem;
}

.player-controls {
  display: flex;
  gap: 2rem;
  color: rgba(255, 255, 255, 0.6);
  font-size: 0.8rem;
  letter-spacing: 0.1em;
}

@media (max-width: 1024px) {
  .game-title {
    font-size: 1.8rem;
  }

  .score-display {
    font-size: 2rem;
    top: -40px;
  }

  .game-button {
    padding: 0.6rem 1.5rem;
  }

  .player-controls {
    flex-direction: column;
    gap: 0.5rem;
    align-items: center;
  }
}
</style>