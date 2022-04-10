<script setup lang="ts">import { isDark, toggleDark } from '~/composables';


const colors: Record<number, string> = {
  1: '#3366FF',
  2: '#FF9933',
  3: '#FF6666',
  4: '#FF2233',
  5: '#FF1111',
  6: '#FF0000',
  7: '#FF0000',
  8: '#FF0000',
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
  loading = false
  state: 'playing' | 'win' | 'lose' = 'playing'
  constructor(public size: number, public mines: number) {
    this.size = size
    this.reset()
  }
  reset() {
    this.loading = false
    this.state = 'playing'
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
  async onBoxClick(x: number, y: number) {
    if (this.loading) return
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
        let tempList: Box[] = [box]
        this.loading = true
        while (tempList.length) {
          let list: Box[] = []
          for (const { x, y } of tempList) {
            for (let xx = x - 1; xx <= x + 1; xx++) {
              for (let yy = y - 1; yy <= y + 1; yy++) {
                if (xx < 0 || xx >= this.size || yy < 0 || yy >= this.size) continue
                const near = this.stage[yy][xx]
                if (!near.opened) {
                  await sleep(5)
                  if (near.nearMines === 0) list.push(near)
                  near.opened = true
                }
              }
            }
          }
          tempList = list
        }
        this.loading = false
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

const modes = [
  { size: 8, mines: 8 },
  { size: 10, mines: 12 },
  { size: 14, mines: 20 },
]
const mode = useLocalStorage('game-mode', modes[1])
const game = ref(new Game(mode.value.size, mode.value.mines))

useStorage('state', game, undefined, {
  serializer: {
    read: v => v ? Object.assign(game.value, JSON.parse(v)) : game.value,
    write: v => JSON.stringify(v)
  },
})

const changeMode = () => {
  const index = modes.findIndex(i => i.size === game.value.size)
  const nextMode = modes[(index + 1) % modes.length]
  mode.value = nextMode
  game.value.size = nextMode.size
  game.value.mines = nextMode.mines
  game.value.reset()
}

const getBoxClass = (box: Box) => {
  const classList = ['w-8 h-8 grid place-items-center']
  classList.push('text-xs font-bold')
  if (box.opened) {
    classList.push('cursor-default bg-gray-700/20')
  } else {
    classList.push('cursor-pointer bg-gray-600 hover:bg-gray-600/80')
  }
  return classList
}

const stage = computed(() => game.value.stage)

const sleep = (ms: number) => new Promise(resolve => setTimeout(resolve, ms))


</script>

<template>
  <main class="h-full grid justify-center content-center">
    <h1 class="text-lg font-bold">扫雷</h1>
    <div>
      <div class="flex justify-center mt-6 mb-4">
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
                <div text-sm i-mdi:mine></div>
              </div>
              <!-- <div class="cell-inner" v-if="cell.flagged" v-text="'🚩'"></div> -->
            </div>
          </div>
        </div>
      </div>
      <div class="flex items-center gap-4 border-t border-dashed border-gray-500/50 pt-3">
        <span class="cursor-pointer" @click="changeMode">{{ game.size }} x {{ game.size }}</span>
        <span class="flex items-center gap-1">
          <i inline-block text-xs text-red-400 i-mdi:mine></i>
          {{ game.mines }}
        </span>
        <div class="status">
          <span v-if="game.state === 'playing'">游戏中...</span>
          <span text-green v-if="game.state === 'win'">你赢了!</span>
          <span text-red v-if="game.state === 'lose'">你输了!</span>
        </div>
        <div class="cursor-pointer ml-auto" @click="game.reset()" i-carbon:renew></div>
        <button @click="toggleDark()">
          <div v-if="isDark" i-carbon-moon />
          <div v-else i-carbon-sun />
        </button>
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