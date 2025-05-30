<template>
  <div class="gradient-picker">
    <!-- 顶部渐变条 -->
    <div class="gradient-bar-wrapper">
      <div
          class="gradient-bar"
          :style="{ background: gradientCSS }"
          @click="addColorStop"
      >
        <div
            v-for="(stop, index) in internalStops"
            :key="index"
            class="color-stop"
            :style="{ left: stop.position + '%' }"
            @mousedown="onDragStart($event, index)"
            @click.stop="selectStop(index)"
            @contextmenu.prevent="removeStopByIndex(index)"
        >
          <div class="color-stop-marker" :class="{ selected: index === selectedIndex }">
            <div class="default-marker" :style="{ backgroundColor: stop.color }"></div>
          </div>
        </div>
      </div>
    </div>

    <!-- 列表头部 -->
    <div class="stop-row stop-header">
      <span style="width: 140px">位置</span>
      <span style="width: 30px"></span>
      <span style="width: 140px"></span>
      <span style="text-align: left">
        <el-button @click="addColorStopManually" style="border: none; background: transparent;">
          <svg t="1748571155200" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="5019" width="20" height="20"><path d="M469.312 469.312v-256h85.376v256h256v85.376h-256v256H469.312v-256h-256V469.312z" fill="#8a8a8a" p-id="5020"></path></svg>
        </el-button>
      </span>
    </div>

    <!-- 颜色点列表 -->
    <div class="stop-list">
      <div class="stop-row" v-for="(stop, index) in internalStops" :key="index">
        <el-input-number v-model="stop.position" :min="0" :max="100" controls-position="right" style="width: 140px" @focus="selectStop(index)">
          <template #suffix>
            <span>%</span>
          </template>
        </el-input-number>
        <el-color-picker v-model="stop.color" style="width: 30px" @focus="selectStop(index)"/>
        <el-input v-model="stop.color" style="width: 140px" @focus="selectStop(index)"/>
        <div style="display: flex; align-items: center; justify-content: center; height: 100%;">
          <el-button @click="removeStopByIndex(index)" :disabled="internalStops.length <= 2" style="border: none; background: transparent;">
            <svg t="1748571216087" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="3964" width="20" height="20"><path d="M802.909091 488.727273h-558.545455a34.909091 34.909091 0 1 0 0 69.818182h558.545455a34.909091 34.909091 0 1 0 0-69.818182" fill="#797979" p-id="3965"></path></svg>
          </el-button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted, onUnmounted } from 'vue'

// Props
const props = defineProps({
  modelValue: {
    type: Array,
    default: () => [
      { color: '#ff0000', position: 0 },
      { color: '#00ff00', position: 100 }
    ]
  }
})
const emit = defineEmits(['update:modelValue'])

const internalStops = ref([...props.modelValue])
const selectedIndex = ref(null)

// 用于拖拽的索引
const draggingIndex = ref(null)

// 渐变背景
const gradientCSS = computed(() => {
  return `linear-gradient(to right, ${internalStops.value
      .map(stop => `${stop.color} ${stop.position}%`)
      .join(', ')})`
})

// 监听外部更新
watch(
    () => props.modelValue,
    (val) => {
      const sorted = [...val].sort((a, b) => a.position - b.position)
      const current = internalStops.value[selectedIndex.value]
      internalStops.value = sorted
      if (current) {
        const idx = sorted.findIndex(
            s => s.color === current.color && s.position === current.position
        )
        selectedIndex.value = idx !== -1 ? idx : null
      }
    }
)

// 同步内部状态
watch(
    internalStops,
    (val) => {
      const sorted = [...val].sort((a, b) => a.position - b.position)
      if (JSON.stringify(sorted) !== JSON.stringify(props.modelValue)) {
        internalStops.value = sorted
        emit('update:modelValue', sorted)
      }
    },
    { deep: true }
)

// 点击条添加颜色点
function addColorStop(event) {
  const rect = event.target.getBoundingClientRect()
  const clickX = event.clientX - rect.left
  const percent = Math.round((clickX / rect.width) * 100)
  const newStop = { color: '#FACD91', position: percent }
  internalStops.value.push(newStop)
  internalStops.value.sort((a, b) => a.position - b.position)
  selectedIndex.value = internalStops.value.findIndex(
      s => s.color === newStop.color && s.position === newStop.position
  )
}

// 手动添加颜色点（默认 50%）
function addColorStopManually() {
  const newStop = { color: '#FACD91', position: 50 }
  internalStops.value.push(newStop)
}

// 删除颜色点
function removeStopByIndex(index) {
  if (internalStops.value.length <= 2) return
  internalStops.value.splice(index, 1)
  if (selectedIndex.value === index) {
    selectedIndex.value = null
  } else if (selectedIndex.value > index) {
    selectedIndex.value -= 1
  }
}

// 选择颜色点
function selectStop(index) {
  selectedIndex.value = index
}

// 拖拽相关事件
function onDragStart(event, index) {
  draggingIndex.value = index
  event.preventDefault()
}

function onDragMove(event) {
  if (draggingIndex.value === null) return

  const gradientBar = document.querySelector('.gradient-bar')
  if (!gradientBar) return

  const rect = gradientBar.getBoundingClientRect()
  let pos = ((event.clientX - rect.left) / rect.width) * 100
  pos = Math.min(100, Math.max(0, pos)) // 限制范围在0-100

  internalStops.value[draggingIndex.value].position = Math.round(pos)
}

function onDragEnd() {
  draggingIndex.value = null
}

onMounted(() => {
  window.addEventListener('mousemove', onDragMove)
  window.addEventListener('mouseup', onDragEnd)
})

onUnmounted(() => {
  window.removeEventListener('mousemove', onDragMove)
  window.removeEventListener('mouseup', onDragEnd)
})
</script>

<style scoped>
.gradient-picker {
  border: 1px dashed #dcdfe6;
  padding: 20px;
  width: 100%;
}
.gradient-bar-wrapper {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.gradient-bar {
  position: relative;
  height: 20px;
  width: 100%;
  margin-right: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  cursor: pointer;
}

.color-stop {
  position: absolute;
  bottom: 90%;
  transform: translateX(-50%);
  cursor: pointer;
  user-select: none;
  display: flex;
  flex-direction: column;
  align-items: center;
}

/* 外层白底矩形容器 */
.color-stop-marker {
  position: relative; /* 关键：为伪元素定位提供参考 */
  background-color: rgba(230, 233, 240, 0.30);
  border-radius: 4px;
  padding: 3px;
  display: flex;
  flex-direction: column;
  align-items: center;
}
.color-stop-marker::after {
  content: '';
  position: absolute;
  bottom: -4px; /* 距离上层元素底部的距离 */
  left: 50%;
  transform: translateX(-50%);
  width: 0;
  height: 0;
  border-left: 4px solid transparent;
  border-right: 4px solid transparent;
  border-top: 5px solid rgba(230, 233, 240, 0.30); /* 颜色与背景一致 */
}
/* 被选中时边框变蓝 */
.color-stop-marker.selected {
  background-color: #409EFF;
}
.color-stop-marker.selected::after {
  border-top-color: #409EFF; /* 与 .selected 背景色一致 */
}


.default-marker {
  width: 12px;
  height: 12px;
  border-radius: 3px;
  border: 1px solid #666;
}

.stop-list {
  margin-top: 8px;
  max-height: 100px;
  overflow-y: auto;
  padding-right: 4px; /* 防止滚动条遮住内容 */
}

.stop-row {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 8px;
}
.stop-header {
  margin-top: 12px;
  display: flex;
  align-items: center;
  font-weight: bold;
  margin-bottom: 0;
}
</style>
