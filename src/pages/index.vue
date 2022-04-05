<script setup lang="ts">

const colors = [
  'red',
  'green',
  'blue',
  'yellow',
  'purple',
  'orange',
  'pink',
  'brown',
]

interface Box {
  x: number;
  y: number;
  mine: boolean;
  opened: boolean;
  flagged: boolean;
  nearMines: number;
}
class Game {
  stage = ref([] as Box[][])
  // 是否生成地雷了
  isMinePlanted = false
  state: 'playing' | 'win' | 'lose' = 'playing'
  constructor(public size: number) {
    this.size = size
    this.reset()
  }
  reset() {
    this.isMinePlanted = false
    this.stage.value = []
    for (let i = 0; i < this.size; i++) {
      this.stage.value[i] = []
      for (let j = 0; j < this.size; j++) {
        this.stage.value[i][j] = {
          y: i,
          x: j,
          mine: false,
          opened: false,
          flagged: false,
          nearMines: 0
        }
      }
    }
  }
  plantMines(count: number, ox: number, oy: number) {
    if (this.isMinePlanted) return
    this.isMinePlanted = true
    const mines = []
    while (mines.length < count) {
      const x = Math.floor(Math.random() * this.size)
      const y = Math.floor(Math.random() * this.size)
      if (Math.abs(x - ox) <= 1 && Math.abs(y - oy) <= 1) continue
      const mine = this.stage.value[x][y]
      if (!mine.mine) {
        mine.mine = true
        mines.push(mine)
      }
    }
    for (const mine of mines) {
      const { x, y } = mine
      for (let i = x - 1; i <= x + 1; i++) {
        for (let j = y - 1; j <= y + 1; j++) {
          if (i < 0 || i >= this.size || j < 0 || j >= this.size) continue
          const nearMine = this.stage.value[i][j]
          if (nearMine.mine) continue
          nearMine.nearMines++
        }
      }
    }
  }
  onBoxClick(x: number, y: number) {
    const box = this.stage.value[y][x]
    if (box.opened) return
    // if (box.flagged) return

    if (box.mine) {
      this.state = 'lose'
      return
    } else {
      box.opened = true
    }
    if (!this.isMinePlanted) {
      this.plantMines(this.size ** 2 / 10, x, y)
    }
    // if (box.nearMines === 0) {
    //   this.openNearBoxes(x, y)
    // } else {
    //   box.opened = true
    // }
    // if (this.isWin()) {
    //   this.state = 'win'
    // }
  }
}
const game = new Game(10)

const getBoxClass = (box: Box) => {
  const classList = ['w-8', 'h-8', 'grid place-items-center']
  classList.push('text-xs font-bold')
  if (box.opened) {
    classList.push('cursor-default bg-gray-400/50')
  } else {
    classList.push('cursor-pointer bg-gray-600 hover:bg-gray-300/20')
  }
  return classList
}

const stage = computed(() => game.stage.value)

</script>

<template>
  <main>
    <h1>Mine Sweeper</h1>
    <div class="game">
      <div class="flex justify-center m10">
        <div class>
          <div class="flex" v-for="(row, y) in stage" :key="y">
            <div
              class="m-0.5"
              v-for="(box, x) in row"
              :key="x"
              :class="getBoxClass(box)"
              @click="game.onBoxClick(box.x, box.y)"
            >
              <div
                v-if="box.opened && !box.mine && box.nearMines"
                v-text="box.nearMines || ''"
                :style="{ color: colors[box.nearMines] }"
              ></div>
              <div v-if="box.mine && box.opened">
                <div i-carbon:close-filled></div>
              </div>
              <!-- <div class="cell-inner" v-if="cell.flagged" v-text="'🚩'"></div> -->
            </div>
          </div>
        </div>
      </div>
      <div class="game-info">
        <div class="status">
          <span v-if="game.state === 'playing'">Playing...</span>
          <span v-if="game.state === 'win'">You Win!</span>
          <span v-if="game.state === 'lose'">You Lose!</span>
        </div>
        <div class="action">
          <button v-if="game.state === 'playing'" @click="game.reset()">Reset</button>
        </div>
      </div>
    </div>
  </main>
</template>
