<template>
  <div id="game-container">
    <div class="game-info">
      <h1>🎮 Super Mario Game</h1>
      <div class="controls">
        <p>Use Arrow Keys or WASD to move</p>
        <p>Position: X: {{ Math.round(player.x) }}, Y: {{ Math.round(player.y) }}</p>
        <p>Score: {{ score }}</p>
      </div>
    </div>
    <canvas 
      ref="gameCanvas" 
      :width="gameWidth" 
      :height="gameHeight"
      @keydown="handleKeyDown"
      @keyup="handleKeyUp"
      tabindex="0"
    ></canvas>
  </div>
</template>

<script>
import { ref, onMounted, onUnmounted } from 'vue'

export default {
  name: 'App',
  setup() {
    const gameCanvas = ref(null)
    const gameWidth = 1200
    const gameHeight = 600
    const score = ref(0)
    
    let ctx = null
    let animationId = null
    
    const player = ref({
      x: 100,
      y: 400,
      width: 48,
      height: 48,
      velocityX: 0,
      velocityY: 0,
      speed: 5,
      jumpPower: 15,
      gravity: 0.8,
      onGround: false,
      direction: 'right',
      isMoving: false
    })
    
    const keys = ref({
      left: false,
      right: false,
      up: false,
      down: false
    })
    
    const platforms = [
      { x: 0, y: 500, width: 400, height: 100, color: '#8B4513' },
      { x: 500, y: 450, width: 200, height: 150, color: '#8B4513' },
      { x: 800, y: 400, width: 400, height: 200, color: '#8B4513' },
      { x: 350, y: 350, width: 100, height: 20, color: '#CD853F' },
      { x: 600, y: 250, width: 100, height: 20, color: '#CD853F' },
      { x: 900, y: 200, width: 100, height: 20, color: '#CD853F' }
    ]
    
    const buildings = [
      { x: 50, y: 420, width: 60, height: 80, color: '#D2691E', type: 'house' },
      { x: 200, y: 440, width: 50, height: 60, color: '#A0522D', type: 'house' },
      { x: 850, y: 320, width: 70, height: 80, color: '#8B4513', type: 'house' }
    ]
    
    const coins = ref([
      { x: 400, y: 300, width: 24, height: 24, collected: false },
      { x: 650, y: 200, width: 24, height: 24, collected: false },
      { x: 950, y: 150, width: 24, height: 24, collected: false }
    ])
    
    const clouds = [
      { x: 100, y: 80, width: 80, height: 40 },
      { x: 400, y: 50, width: 100, height: 50 },
      { x: 800, y: 100, width: 90, height: 45 },
      { x: 1000, y: 60, width: 85, height: 42 }
    ]
    
    const handleKeyDown = (e) => {
      const key = e.key.toLowerCase()
      if (key === 'arrowleft' || key === 'a') {
        keys.value.left = true
        e.preventDefault()
      }
      if (key === 'arrowright' || key === 'd') {
        keys.value.right = true
        e.preventDefault()
      }
      if (key === 'arrowup' || key === 'w' || key === ' ') {
        if (player.value.onGround) {
          player.value.velocityY = -player.value.jumpPower
          player.value.onGround = false
        }
        e.preventDefault()
      }
      if (key === 'arrowdown' || key === 's') {
        keys.value.down = true
        e.preventDefault()
      }
    }
    
    const handleKeyUp = (e) => {
      const key = e.key.toLowerCase()
      if (key === 'arrowleft' || key === 'a') keys.value.left = false
      if (key === 'arrowright' || key === 'd') keys.value.right = false
      if (key === 'arrowdown' || key === 's') keys.value.down = false
    }
    
    const drawMario = () => {
      const p = player.value
      
      ctx.save()
      
      if (p.direction === 'left') {
        ctx.translate(p.x + p.width, p.y)
        ctx.scale(-1, 1)
        ctx.translate(-p.x, -p.y)
      }
      
      ctx.fillStyle = '#FF0000'
      ctx.fillRect(p.x + 12, p.y, 24, 12)
      
      ctx.fillStyle = '#FFD700'
      ctx.fillRect(p.x + 8, p.y + 12, 32, 16)
      
      ctx.fillStyle = '#0000FF'
      ctx.fillRect(p.x + 8, p.y + 28, 32, 12)
      
      ctx.fillStyle = '#8B4513'
      ctx.fillRect(p.x + 12, p.y + 40, 10, 8)
      ctx.fillRect(p.x + 26, p.y + 40, 10, 8)
      
      ctx.fillStyle = '#000000'
      ctx.fillRect(p.x + 16, p.y + 16, 4, 4)
      ctx.fillRect(p.x + 28, p.y + 16, 4, 4)
      
      ctx.fillStyle = '#000000'
      ctx.beginPath()
      ctx.arc(p.x + 24, p.y + 24, 2, 0, Math.PI)
      ctx.fill()
      
      ctx.restore()
    }
    
    const drawPlatforms = () => {
      platforms.forEach(platform => {
        ctx.fillStyle = platform.color
        ctx.fillRect(platform.x, platform.y, platform.width, platform.height)
        
        ctx.strokeStyle = '#654321'
        ctx.lineWidth = 2
        ctx.strokeRect(platform.x, platform.y, platform.width, platform.height)
        
        for (let i = 0; i < platform.width; i += 20) {
          ctx.strokeStyle = '#4a3319'
          ctx.beginPath()
          ctx.moveTo(platform.x + i, platform.y)
          ctx.lineTo(platform.x + i, platform.y + platform.height)
          ctx.stroke()
        }
      })
    }
    
    const drawBuildings = () => {
      buildings.forEach(building => {
        ctx.fillStyle = building.color
        ctx.fillRect(building.x, building.y, building.width, building.height)
        
        ctx.strokeStyle = '#000'
        ctx.lineWidth = 2
        ctx.strokeRect(building.x, building.y, building.width, building.height)
        
        ctx.fillStyle = '#87CEEB'
        const windowSize = 10
        const windowSpacing = 15
        for (let i = 0; i < 2; i++) {
          for (let j = 0; j < 2; j++) {
            ctx.fillRect(
              building.x + 10 + i * windowSpacing,
              building.y + 10 + j * windowSpacing,
              windowSize,
              windowSize
            )
          }
        }
        
        ctx.fillStyle = '#654321'
        ctx.fillRect(
          building.x + building.width / 2 - 8,
          building.y + building.height - 20,
          16,
          20
        )
      })
    }
    
    const drawCoins = () => {
      coins.value.forEach(coin => {
        if (!coin.collected) {
          ctx.fillStyle = '#FFD700'
          ctx.beginPath()
          ctx.arc(coin.x + coin.width / 2, coin.y + coin.height / 2, coin.width / 2, 0, Math.PI * 2)
          ctx.fill()
          
          ctx.strokeStyle = '#FFA500'
          ctx.lineWidth = 2
          ctx.stroke()
          
          ctx.fillStyle = '#FFA500'
          ctx.font = 'bold 16px Arial'
          ctx.textAlign = 'center'
          ctx.fillText('$', coin.x + coin.width / 2, coin.y + coin.height / 2 + 5)
        }
      })
    }
    
    const drawClouds = () => {
      ctx.fillStyle = 'rgba(255, 255, 255, 0.8)'
      clouds.forEach(cloud => {
        ctx.beginPath()
        ctx.arc(cloud.x, cloud.y + cloud.height / 2, cloud.height / 2, 0, Math.PI * 2)
        ctx.arc(cloud.x + cloud.width / 3, cloud.y, cloud.height / 1.5, 0, Math.PI * 2)
        ctx.arc(cloud.x + cloud.width * 2 / 3, cloud.y, cloud.height / 1.5, 0, Math.PI * 2)
        ctx.arc(cloud.x + cloud.width, cloud.y + cloud.height / 2, cloud.height / 2, 0, Math.PI * 2)
        ctx.fill()
      })
    }
    
    const drawBackground = () => {
      const gradient = ctx.createLinearGradient(0, 0, 0, gameHeight)
      gradient.addColorStop(0, '#87CEEB')
      gradient.addColorStop(1, '#E0F6FF')
      ctx.fillStyle = gradient
      ctx.fillRect(0, 0, gameWidth, gameHeight)
    }
    
    const checkCollision = (rect1, rect2) => {
      return rect1.x < rect2.x + rect2.width &&
             rect1.x + rect1.width > rect2.x &&
             rect1.y < rect2.y + rect2.height &&
             rect1.y + rect1.height > rect2.y
    }
    
    const updatePlayer = () => {
      const p = player.value
      
      p.isMoving = false
      
      if (keys.value.left) {
        p.velocityX = -p.speed
        p.direction = 'left'
        p.isMoving = true
      } else if (keys.value.right) {
        p.velocityX = p.speed
        p.direction = 'right'
        p.isMoving = true
      } else {
        p.velocityX *= 0.8
        if (Math.abs(p.velocityX) < 0.1) p.velocityX = 0
      }
      
      if (!p.onGround) {
        p.velocityY += p.gravity
      }
      
      if (p.velocityY > 20) p.velocityY = 20
      
      p.x += p.velocityX
      p.y += p.velocityY
      
      if (p.x < 0) p.x = 0
      if (p.x + p.width > gameWidth) p.x = gameWidth - p.width
      
      p.onGround = false
      
      platforms.forEach(platform => {
        if (checkCollision(p, platform)) {
          if (p.velocityY > 0) {
            p.y = platform.y - p.height
            p.velocityY = 0
            p.onGround = true
          } else if (p.velocityY < 0) {
            p.y = platform.y + platform.height
            p.velocityY = 0
          }
        }
      })
      
      buildings.forEach(building => {
        if (checkCollision(p, building)) {
          if (p.velocityY > 0 && p.y + p.height - p.velocityY <= building.y) {
            p.y = building.y - p.height
            p.velocityY = 0
            p.onGround = true
          } else if (p.velocityX > 0) {
            p.x = building.x - p.width
            p.velocityX = 0
          } else if (p.velocityX < 0) {
            p.x = building.x + building.width
            p.velocityX = 0
          } else if (p.velocityY < 0) {
            p.y = building.y + building.height
            p.velocityY = 0
          }
        }
      })
      
      coins.value.forEach(coin => {
        if (!coin.collected && checkCollision(p, coin)) {
          coin.collected = true
          score.value += 100
        }
      })
      
      if (p.y > gameHeight) {
        p.x = 100
        p.y = 400
        p.velocityX = 0
        p.velocityY = 0
        score.value = Math.max(0, score.value - 50)
      }
    }
    
    const gameLoop = () => {
      updatePlayer()
      
      drawBackground()
      drawClouds()
      drawPlatforms()
      drawBuildings()
      drawCoins()
      drawMario()
      
      animationId = requestAnimationFrame(gameLoop)
    }
    
    onMounted(() => {
      ctx = gameCanvas.value.getContext('2d')
      gameCanvas.value.focus()
      
      window.addEventListener('keydown', handleKeyDown)
      window.addEventListener('keyup', handleKeyUp)
      
      gameLoop()
    })
    
    onUnmounted(() => {
      if (animationId) {
        cancelAnimationFrame(animationId)
      }
      window.removeEventListener('keydown', handleKeyDown)
      window.removeEventListener('keyup', handleKeyUp)
    })
    
    return {
      gameCanvas,
      gameWidth,
      gameHeight,
      player,
      score,
      handleKeyDown,
      handleKeyUp
    }
  }
}
</script>

<style scoped>
#game-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  background: linear-gradient(to bottom, #5c94fc, #4a7bd4);
  padding: 20px;
}

.game-info {
  background: rgba(255, 255, 255, 0.9);
  padding: 20px;
  border-radius: 10px;
  margin-bottom: 20px;
  text-align: center;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.game-info h1 {
  margin: 0 0 10px 0;
  color: #e60012;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.2);
}

.controls {
  color: #333;
  line-height: 1.6;
}

.controls p {
  margin: 5px 0;
  font-size: 14px;
}

canvas {
  border: 4px solid #333;
  border-radius: 8px;
  background: #5c94fc;
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.3);
  cursor: pointer;
  display: block;
}

canvas:focus {
  outline: 3px solid #ffd700;
}
</style>
