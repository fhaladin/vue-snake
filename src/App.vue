<template>
  <div id="app">
    <GlobalEvents
      @keydown="arrowUp"
      @keydown.down="arrowDown"
      @keydown.left="arrowLeft"
      @keydown.right="arrowRight"
    />
    <canvas ref="game-area" :width="game.w" :height="game.h"></canvas>
  </div>
</template>

<script>
import GlobalEvents from 'vue-global-events'

export default {
  name: 'App',
  components: {
    GlobalEvents
  },
  data () {
    return {
      isGameOver: false,
      game: {
        area: null,
        w: 400,
        h: 400
      },
      snake: {
        head: {
          x: 0,
          y: 0
        },
        body: [],
        w: 10,
        h: 10
      },
      food: {
        x: 0,
        y: 0,
        w: 10,
        h: 10
      },
      move: {
        interval: null,
        time: 0.1
      }
    }
  },
  watch: {
    'snake.head.x' (val) {
      if (val === 0 || val === this.game.w - this.snake.w) {
        clearInterval(this.move.interval)
        this.isGameOver = true
      }
    },
    'snake.head.y' (val) {
      if (val === 0 || val === this.game.h - this.snake.h) {
        clearInterval(this.move.interval)
        this.isGameOver = true
      }
    },
    isGameOver (val) {
      if (val) {
        this.game.area.fillStyle = 'red'
        this.game.area.textAlign = 'center'
        this.game.area.font = '30px Arial';
        this.game.area.fillText('Game Over', this.game.w/2, this.game.h/2)
      }
    }
  },
  mounted () {
    const canvas = this.$refs['game-area']
    const game = this.game
    const snake = this.snake
    const food = this.food

    this.game.area = canvas.getContext("2d")
    this.snake.head.x = this.randomCoor(0, game.w - snake.w)
    this.snake.head.y = this.randomCoor(0, game.h - snake.h)

    this.food.x = this.randomCoor(0, game.w - food.w)
    this.food.y = this.randomCoor(0, game.h - food.h)

    this.dSnake()
    this.dFood()
  },
  methods: {
    /** Control */
    arrowUp () {
      clearInterval(this.move.interval)
      this.sMove('y', -10)
    },
    arrowDown () {
      clearInterval(this.move.interval)
      this.sMove('y', 10)
    },
    arrowLeft () {
      clearInterval(this.move.interval)
      this.sMove('x', -10)
    },
    arrowRight () {
      clearInterval(this.move.interval)
      this.sMove('x', 10)
    },

    /** Draw */
    dSnake () {
      const game = this.game
      const snake = this.snake

      game.area.fillStyle = '#000'
      game.area.fillRect(snake.head.x, snake.head.y, snake.w, snake.h)

      snake.body.forEach((b) => {
        game.area.fillRect(b.x, b.y, snake.w, snake.h)
      })
    },
    dFood () {
      const game = this.game
      const food = this.food

      game.area.fillStyle = '#1cfc03'
      game.area.fillRect(food.x, food.y, food.w, food.h)
    },

    /** Snake */
    removeSnake () {
      const game = this.game
      const snake = this.snake

      game.area.clearRect(
        snake.head.x,
        snake.head.y,
        snake.w,
        snake.h
      )

      snake.body.forEach((b) => {
        game.area.clearRect(
          b.x,
          b.y,
          snake.w,
          snake.h
        )
      })
    },
    eat () {
      const game = this.game
      const food = this.food

      this.snake.body.push({
        x: food.x,
        y: food.y
      })

      this.food.x = this.randomCoor(0, game.w - food.w)
      this.food.y = this.randomCoor(0, game.h - food.h)
      this.dFood()
    },

    /** Movement */
    sMove (dir, dis) {
      this.move.interval = setInterval(() => {
        if (!this.isGameOver) {
          this.removeSnake()
          if (this.snake.body.length !== 0) {
            this.snake.body.pop()
            this.snake.body.unshift({...this.snake.head})
          }
  
          this.snake.head[dir] += dis
          
          if (this.snake.head.x === this.food.x && this.snake.head.y === this.food.y) {
            this.eat()
          }
  
          this.dSnake()
        }
      }, this.move.time * 1000)
    },

    /** Utils */
    randomCoor (min, max) {
      let numb = Math.round(Math.random() * (max - min) + min)
      numb -= numb % this.snake.w
      
      return numb
    }
  }
}
</script>

<style>
  canvas {
    border: 1px solid black;
  }
</style>
