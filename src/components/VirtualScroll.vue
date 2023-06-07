<template>
  <!-- 容器 -->
  <div class="virtual-scroll-container" ref="vListRef" @scroll="handleScroll">
    <!-- 不可视占位，撑开容器滚动条，高度为列表总高度 -->
    <div
      class="virtual-scroll-phantom"
      :style="{ height: listDataHeight + 'px' }"
    ></div>
    <!-- 最终渲染出的列表 -->
    <div
      class="virtual-scroll-list"
      :style="{ transform: getStartOffsetTransform }"
    >
      <div
        class="virtual-scroll-item"
        :style="{ height: itemHeight + 'px' }"
        v-for="item in visibleData"
        :key="item.id"
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
    default: []
  },
  // 列表每一项item的高度
  itemHeight: {
    type: Number,
    default: 50
  }
})

let vListRef = ref(null)
// 可视区高度
let scrollHeight = ref(0)
// 可视区偏移量，使滚动条滚动高度没有超出itemHeight时过渡平滑
let startOffset = ref(0)
// 起始索引
let startIndex = ref(0)
// 结束索引
let endIndex = ref(0)

// 列表总高度
const listDataHeight = computed(() => {
  return props.listData.length * props.itemHeight
})
// 偏移量作用到可视区dom的样式
const getStartOffsetTransform = computed(() => {
  return `translate3d(0, ${startOffset.value}px, 0)`
})
// 可视区列表数目
const visibleCount = computed(() => {
  return Math.ceil(scrollHeight.value / props.itemHeight)
})
// 可视区数据
const visibleData = computed(() => {
  return props.listData.slice(
    startIndex.value,
    Math.min(endIndex.value, props.listData.length)
  )
})

onMounted(() => {
  initialize()
})

function initialize() {
  scrollHeight.value = vListRef.value.clientHeight
  startIndex.value = 0
  endIndex.value = startIndex.value + visibleCount.value
}

function handleScroll() {
  // 当前滚动位置
  let scrollTop = vListRef.value.scrollTop
  // 滚动后的索引
  startIndex.value = Math.floor(scrollTop / props.itemHeight)
  endIndex.value = startIndex.value + visibleCount.value
  // 此时的偏移量
  startOffset.value = startIndex.value * props.itemHeight
}
</script>

<style lang="scss" scoped>
.virtual-scroll-container {
  height: 100%;
  position: relative;
  overflow: auto;

  .virtual-scroll-phantom {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    z-index: -1;
  }

  .virtual-scroll-list {
    position: absolute;
    left: 0;
    right: 0;
    top: 0;

    .virtual-scroll-item {
      box-sizing: border-box;
      padding: 5px;
      color: #555;
      border-bottom: 1px solid #ccc;
    }
  }
}
</style>
