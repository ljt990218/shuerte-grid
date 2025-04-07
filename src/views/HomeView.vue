<script setup lang="ts">
import { ref, onMounted } from 'vue'

interface GridItem {
  id: number
  number: number
  isClick: boolean
}

const totalTime = ref(0)
const startTime = ref(0)
const endTime = ref(0)
const isGameOver = ref(false)
const currentItemID = ref(0)

// 添加网格大小配置
const gridSizes = {
  small: { rows: 5, cols: 5, total: 25 },
  middle: { rows: 7, cols: 7, total: 49 },  
  large: { rows: 10, cols: 10, total: 100 }
} as const

const currentSize = ref<keyof typeof gridSizes>('small')

function shuffleArray<T>(array: T[]): T[] {
  const shuffled = [...array]
  for (let i = shuffled.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
      ;[shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]]
  }
  return shuffled
}

const grid = ref<GridItem[]>(
  shuffleArray(
    Array.from({ length: gridSizes[currentSize.value].total }, (_, i) => ({
      id: i + 1,
      number: i + 1,
      isClick: false,
    }))
  )
)

// 添加切换网格大小的方法
function switchGridSize(size: keyof typeof gridSizes) {
  currentSize.value = size
  resetGame()
}

function handleClick(item: GridItem, index: number) {
  if (item.isClick) return
  if (item.id !== currentItemID.value + 1) return

  currentItemID.value = item.id
  grid.value[index].isClick = !grid.value[index].isClick

  if (currentItemID.value === gridSizes[currentSize.value].total) {
    endTime.value = Date.now()
    totalTime.value = (endTime.value - startTime.value) / 1000
    isGameOver.value = true
  }
}

function resetGame() {
  grid.value = shuffleArray(
    Array.from({ length: gridSizes[currentSize.value].total }, (_, i) => ({
      id: i + 1,
      number: i + 1,
      isClick: false,
    }))
  )
  currentItemID.value = 0
  isGameOver.value = false
  startTime.value = Date.now()
}

onMounted(() => {
  startTime.value = Date.now()
})

const ageGroup = ref('adult')

const referenceStandards = {
  child: [
    { level: '优秀水平', time: '26秒以内' },
    { level: '中等水平', time: '42秒以内' },
    { level: '需要提升', time: '50秒以上' },
  ],
  teen: [
    { level: '优良水平', time: '16秒以内' },
    { level: '中等水平', time: '26秒以内' },
    { level: '需要提升', time: '36秒以上' },
  ],
  adult: [
    { level: '专业水平', time: '8秒以内' },
    { level: '中等水平', time: '20秒以内' },
    { level: '需要提升', time: '20秒以上' },
  ],
}

function getTimeLevel(time: number, group: string) {
  if (group === 'child') {
    if (time <= 26) return 0
    if (time <= 42) return 1
    return 2
  }
  if (group === 'teen') {
    if (time <= 16) return 0
    if (time <= 26) return 1
    return 2
  }
  // adult
  if (time <= 8) return 0
  if (time <= 20) return 1
  return 2
}
</script>

<template>
  <h1 class="text-3xl pt-4 mb-4 text-center font-bold">Shuerte Grid</h1>
  <div class="mb-4">
    <div class="flex gap-2 justify-center">
      <button 
        v-for="(config, size) in gridSizes" 
        :key="size" 
        @click="switchGridSize(size)" 
        :class="[
          'px-4 py-2 rounded-full transition-all duration-200',
          currentSize === size
            ? 'bg-blue-400 text-white shadow-md transform scale-105'
            : 'bg-gray-100 text-gray-600 hover:bg-gray-200'
        ]"
      >
        {{ `${config.rows}x${config.cols}` }}
      </button>
    </div>
  </div>
  <div class="flex justify-center items-center p-2 touch-manipulation">
    <div v-if="!isGameOver" :class="[
      'grid gap-2 p-4 bg-gray-100 rounded-lg',
      {
        'grid-cols-5': currentSize === 'small',
        'grid-cols-7': currentSize === 'middle',
        'grid-cols-10': currentSize === 'large'
      }
    ]">
      <div v-for="(item, index) in grid" :key="item.id" @click="handleClick(item, index)" :class="[
        'rounded flex items-center justify-center font-bold cursor-pointer transition-colors',
        {
          'w-16 h-16 text-xl': currentSize === 'small',
          'w-14 h-14 text-lg': currentSize === 'middle',
          'w-12 h-12 text-base': currentSize === 'large'
        },
        item.isClick ? 'bg-gray-200 text-blue rounded' : 'bg-white text-blue-500 hover:bg-gray-200',
      ]">
        {{ item.number }}
      </div>
    </div>

    <div v-if="isGameOver" class="text-center bg-white px-8 py-8 rounded-lg shadow-lg">
      <h1 class="text-4xl font-bold mb-6 text-blue-600">恭喜完成!</h1>
      <p class="text-2xl mb-4 text-gray-700">
        用时: <span class="font-bold text-blue-500">{{ totalTime }}</span> 秒
      </p>
      <div class="mb-4">
        <div class="flex gap-2 justify-center">
          <button v-for="(option, value) in [
            { value: 'child', label: '7-12岁' },
            { value: 'teen', label: '13-17岁' },
            { value: 'adult', label: '18岁及以上' }
          ]" :key="value" @click="ageGroup = option.value" :class="[
              'px-4 py-2 rounded-full transition-all duration-200',
              ageGroup === option.value
                ? 'bg-blue-400 text-white shadow-md transform scale-105'
                : 'bg-gray-100 text-gray-600 hover:bg-gray-200'
            ]">
            {{ option.label }}
          </button>
        </div>
      </div>
      <div class="text-sm text-gray-500 mb-6">
        <p class="mb-1">5*5 参考成绩：</p>
        <p v-for="(standard, index) in referenceStandards[ageGroup]" :key="index" class="mb-1"
          :class="{ 'text-blue-600 font-bold': getTimeLevel(totalTime, ageGroup) === index }">
          {{ standard.level }}：{{ standard.time }}
        </p>
      </div>
      <button @click="resetGame"
        class="bg-blue-500 text-white px-8 py-2 rounded-lg text-xl font-semibold hover:bg-blue-600 transition-colors duration-200 shadow-md">
        再来一次
      </button>
    </div>
  </div>
</template>
