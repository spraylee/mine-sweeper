<script setup lang="ts">

const colors: Record<number, string> = {
  1: '#3366FF',
  2: '#FF9933',
  3: '#FFCC33',
  4: '#33FF33',
  5: '#33CCFF',
  6: '#FF33CC',
  7: '#FF3366',
  8: '#FFCCCC',
}

interface Box {
  x: number;
  y: number;
  mine: boolean;
  opened: boolean;
  flagged: boolean;
  nearMines: number;
}
class Game {
  stage = ([] as Box[][])
  // 是否生成地雷了
  isMinePlanted = false
  state: 'playing' | 'win' | 'lose' = 'playing'
  constructor(public size: number, public mines: number) {
    this.size = size
    this.reset()
  }
  reset() {
    this.isMinePlanted = false
    this.stage = []
    for (let i = 0; i < this.size; i++) {
      this.stage[i] = []
      for (let j = 0; j < this.size; j++) {
        this.stage[i][j] = {
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
      const mine = this.stage[y][x]
      if (!mine.mine) {
        mine.mine = true
        mines.push(mine)
      }
    }
    for (const box of mines) {
      const { x, y } = box
      for (let xx = x - 1; xx <= x + 1; xx++) {
        for (let yy = y - 1; yy <= y + 1; yy++) {
          if (xx < 0 || xx >= this.size || yy < 0 || yy >= this.size) continue
          const near = this.stage[yy][xx]
          near.nearMines++
        }
      }
    }
  }
  onBoxClick(x: number, y: number) {
    if (this.state !== 'playing') return
    if (!this.isMinePlanted) {
      this.plantMines(this.mines, x, y)
    }
    const box = this.stage[y][x]
    if (box.opened) return
    // if (box.flagged) return

    if (box.mine) {
      this.state = 'lose'
      this.showAllMines()
      nextTick(() => {
        alert('lose')
      })
      return
    } else {
      box.opened = true
      if (box.nearMines === 0) {
        for (let xx = x - 1; xx <= x + 1; xx++) {
          for (let yy = y - 1; yy <= y + 1; yy++) {
            if (xx < 0 || xx >= this.size || yy < 0 || yy >= this.size) continue
            const near = this.stage[yy][xx]
            if (!near.opened) {
              this.onBoxClick(xx, yy)
            }
          }
        }
      } else if (this.stage.flat().every(i => i.opened || i.mine)) {
        this.state = 'win'
        nextTick(() => {
          alert('win')
        })
      }
    }

  }
  showAllMines() {
    for (const row of this.stage) {
      for (const box of row) {
        if (box.mine) {
          box.opened = true
        }
      }
    }
  }
}
const game = ref(new Game(10, 10))

const getBoxClass = (box: Box) => {
  const classList = ['w-8 h-8 grid place-items-center']
  classList.push('text-xs font-bold')
  if (box.opened) {
    classList.push('cursor-default bg-gray-400/10')
  } else {
    classList.push('cursor-pointer bg-gray-600 hover:bg-gray-300/20')
  }
  return classList
}

const stage = computed(() => game.value.stage)

</script>

<template>
  <main>
    <h1>Mine Sweeper</h1>
    <div class="game">
      <div class="flex justify-center m10">
        <!-- {{ game }} -->
        <div class>
          <div class="flex" v-for="(row, y) in stage" :key="y">
            <div
              class="box m-0.5"
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
              <div v-if="box.mine && box.opened" class="bg-red-500">
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


<style scoped >
.box > * {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
}
</style>