<script setup>
import { onMounted, ref, nextTick } from 'vue'

// Board settings
const canvas = ref(null)
const ctx = ref(null)
const scoreText = ref("0 : 0")
const gameWidth = 900
const gameHeight = 500

// Color settings
const boardBackground = "forestgreen"
const paddle1Color = "lightblue"
const paddle2Color = "red"
const paddleBorder = "black"
const ballColor = "yellow"
const ballBorderColor = "black"

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
  y: 0
}
let paddle2 = {
  width: 25,
  height: 100,
  x: gameWidth - 25,
  y: 0
}

const gameStart = async () => {
  createBall()
  await nextTick(() => {
    intervalID = setInterval(() => {
      clearBoard()
      drawPaddles()
      moveBall()
      drawBall(ballX, ballY)
      checkCollision()
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
    return
  }
  if(ballX >= gameWidth) {
    player1Score+=1
    updateScore()
    createBall()
    return
  }
  if(ballX <= (paddle1.x + paddle1.width + ballRadius)) {
    if((ballY >= paddle1.y) && (ballY <= (paddle1.y + paddle1.height))) {
      ballX = (paddle1.x + paddle1.width) + ballRadius // prevent ball stuck
      ballXDirection *= -1
      ballSpeed += 1
    }
  }
  if(ballX >= (paddle2.x - ballRadius)) {
    if((ballY >= paddle2.y) && (ballY <= (paddle2.y + paddle1.height))) {
      ballX = paddle2.x - ballRadius // prevent ball stuck
      ballXDirection *= -1
      ballSpeed += 1
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

function updateScore() {
  scoreText.value = player1Score + " : " + player2Score
}

function resetGame() {
  player1Score = 0
  player2Score = 0
  paddle1 = {
    width: 25,
    height: 100,
    x: 0,
    y: 0
  }
  paddle2 = {
    width: 25,
    height: 100,
    x: gameWidth - 25,
    y: 0
  }
  ballX = 0
  ballY = 0
  ballXDirection = 0
  ballYDirection = 0
  clearInterval(intervalID)
  gameStart()
}



onMounted(() => {
  // Variable initialization
  // Using onMounted lifecycle hook because 
  // we need to wait for the HTML components to be ready before calling them
  canvas.value = document.getElementById("gameBoard")
  ctx.value = canvas.value.getContext("2d")
  drawPaddles()
  gameStart()
  window.addEventListener('keydown', changeDirection);
})  

</script>

<template>
  <div id="gameContainer">
    <h1>Pong Game</h1>
    <canvas id="gameBoard" width="900" height="500" style="border:1px solid #FFF;"></canvas>
    <div id="scoreboard">{{ scoreText }}</div>
    <button id="resetBtn" @onclick="resetGame()">Reset Game</button>
  </div>
</template>

<style scoped>
#gameContainer {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

#resetBtn {
  background-color: #4b75ff;
  border: none;
  border-radius: 8px;
  color: white;
  padding: 15px 32px;
  text-align: center;
  font-size: 16px;
  margin: 4px 2px;
  cursor: pointer;
}
</style>