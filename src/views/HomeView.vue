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
    Array.from({ length: 25 }, (_, i) => ({
      id: i + 1,
      number: i + 1,
      isClick: false,
    }))
  )
)

function handleClick(item: GridItem, index) {
  if (item.isClick) return
  if (item.id !== currentItemID.value + 1) return

  currentItemID.value = item.id
  grid.value[index].isClick = !grid.value[index].isClick

  if (currentItemID.value === 25) {
    endTime.value = Date.now()
    totalTime.value = (endTime.value - startTime.value) / 1000
    isGameOver.value = true
  }
}

function resetGame() {
  grid.value = shuffleArray(
    Array.from({ length: 25 }, (_, i) => ({
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
  <h1 class="text-3xl my-4 text-center font-bold">Shuerte Grid</h1>
  <div class="flex justify-center items-center p-2 touch-manipulation">
    <div v-if="!isGameOver" class="grid grid-cols-5 gap-2 p-4 bg-gray-100 rounded-lg">
      <div
        v-for="(item, index) in grid"
        :key="item.id"
        @click="handleClick(item, index)"
        :class="[
          'w-16 h-16 rounded flex items-center justify-center text-xl font-bold cursor-pointer transition-colors',
          item.isClick ? 'bg-gray-200 text-blue rounded' : 'bg-white text-blue-500 hover:bg-gray-200',
        ]"
      >
        {{ item.number }}
      </div>
    </div>

    <div v-if="isGameOver" class="text-center bg-white p-16 rounded-lg shadow-lg">
      <h1 class="text-4xl font-bold mb-6 text-blue-600">恭喜完成!</h1>
      <p class="text-2xl mb-4 text-gray-700">
        用时: <span class="font-bold text-blue-500">{{ totalTime }}</span> 秒
      </p>
      <div class="mb-4">
        <select v-model="ageGroup" class="px-4 py-2 rounded border border-gray-300">
          <option value="child">7-12岁</option>
          <option value="teen">13-17岁</option>
          <option value="adult">18岁及以上</option>
        </select>
      </div>
      <div class="text-sm text-gray-500 mb-6">
        <p class="mb-1">参考成绩：</p>
        <p
          v-for="(standard, index) in referenceStandards[ageGroup]"
          :key="index"
          class="mb-1"
          :class="{ 'text-blue-600 font-bold': getTimeLevel(totalTime, ageGroup) === index }"
        >
          {{ standard.level }}：{{ standard.time }}
        </p>
      </div>
      <button
        @click="resetGame"
        class="bg-blue-500 text-white px-8 py-3 rounded-lg text-xl font-semibold hover:bg-blue-600 transition-colors duration-200 shadow-md"
      >
        再来一次
      </button>
    </div>
  </div>
</template>
