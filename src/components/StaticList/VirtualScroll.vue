<template>
  <!-- 虚拟列表的容器 -->
  <div ref="containerRef" class="virtual-list-container" :style="{ height }" @scroll="handleScroll">
    <!-- 占位容器，用来撑开滚动条，高度为列表总高度 -->
    <div class="virtual-list-phantom" :style="{ height: listDataHeight + 'px' }"></div>
    <!-- 可视区列表 -->
    <div class="virtual-list" :style="{ transform: getStartOffsetTransform }">
      <div
        v-for="item in visibleData"
        :key="item.id"
        class="virtual-list-item"
        :style="{ height: itemHeight + 'px' }"
      >
        {{ item.value }}
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const props = defineProps({
  // 列表总数据
  listData: {
    type: Array,
    default: () => []
  },
  // 列表项高度
  itemHeight: {
    type: Number,
    default: 100
  },
  // 虚拟列表容器高度
  height: {
    type: String,
    default: '100%'
  }
})

const containerRef = ref()

// 可视区高度
let screenHeight = ref(0)
// 可视区开始索引
let startIndex = ref(0)
// 可视区结束索引
let endIndex = ref(0)
// 可视区偏移量
let startOffset = ref(0)

// 列表总高度
const listDataHeight = computed(() => {
  return props.listData.length * props.itemHeight
})

// 可视区数据数目
const visibleCount = computed(() => {
  return Math.ceil(screenHeight.value / props.itemHeight)
})
// 可视区数据
const visibleData = computed(() => {
  return props.listData.slice(startIndex.value, Math.min(endIndex.value, props.listData.length))
})

// 设置可视区偏移量
const getStartOffsetTransform = computed(() => {
  return `translate3d(0, ${startOffset.value}px, 0)`
})

function handleScroll() {
  let scrollTop = containerRef.value.scrollTop
  startIndex.value = Math.floor(scrollTop / props.itemHeight)
  endIndex.value = startIndex.value + visibleCount.value
  startOffset.value = startIndex.value * props.itemHeight
}

onMounted(() => {
  screenHeight.value = containerRef.value.clientHeight
  startIndex.value = 0
  endIndex.value = startIndex.value + visibleCount.value
})
</script>

<style lang="scss" scoped>
.virtual-list-container {
  overflow: auto;
  position: relative;

  .virtual-list-phantom {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    z-index: -1;
  }

  .virtual-list {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;

    .virtual-list-item {
      box-sizing: border-box;
      color: #555;
      border-bottom: 1px solid #ccc;
      padding: 5px;
    }
  }
}
</style>
