<template>
  <!-- 虚拟列表的容器 -->
  <div ref="containerRef" class="virtual-list-container" :style="{ height }" @scroll="handleScroll">
    <!-- 占位容器，用来撑开滚动条，高度为列表总高度 -->
    <div ref="phantomRef" class="virtual-list-phantom"></div>
    <!-- 可视区列表 -->
    <div ref="listRef" class="virtual-list">
      <!-- 列表项 -->
      <div
        v-for="item in visibleData"
        :id="item.id"
        :key="item.index"
        ref="itemsRef"
        class="virtual-list-item"
      >
        <slot :item="item.item"></slot>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUpdated, nextTick } from 'vue'

const props = defineProps({
  // 列表总数据
  listData: {
    type: Array,
    default: () => []
  },
  // 列表中每一项的预估高度
  estimatedItemHeight: {
    type: Number,
    default: 50
  },
  // 虚拟列表容器高度
  height: {
    type: String,
    default: '100%'
  },
  // 不可视的缓冲区数据与可视区数据的比例
  bufferScale: {
    type: Number,
    default: 1
  }
})

// refs
const containerRef = ref()
const phantomRef = ref()
const listRef = ref()
const itemsRef = ref()

// 可视区高度
let screenHeight = ref(0)
// 可视区开始索引
let startIndex = ref(0)
// 可视区结束索引
let endIndex = ref(0)
// 列表位置信息
let positions = ref([
  // {
  //   index,
  //   height: 100,
  //   top: 0,
  //   bottom: 100
  // }
])

// 修改数据方便在当前组件中使用
const _listData = computed(() => {
  return props.listData.map((item, index) => {
    return {
      index,
      id: '_' + index,
      item
    }
  })
})

// 可视区数据的数目
const visibleCount = computed(() => {
  return Math.ceil(screenHeight.value / props.estimatedItemHeight)
})
// 上方缓冲区数据数目
const aboveCount = computed(() => {
  return Math.min(startIndex.value, visibleCount.value * props.bufferScale)
})
// 下方缓冲区数据数目
const blowCount = computed(() => {
  return Math.min(endIndex.value, visibleCount.value * props.bufferScale)
})
// 可视区数据
const visibleData = computed(() => {
  let start = startIndex.value - aboveCount.value
  let end = endIndex.value + blowCount.value
  return _listData.value.slice(start, end)
})

// 初始化位置信息
function initPositions() {
  positions.value = _listData.value.map((item, index) => {
    return {
      index,
      height: props.estimatedItemHeight,
      top: index * props.estimatedItemHeight,
      bottom: (index + 1) * props.estimatedItemHeight
    }
  })
}

// 设置可视区的偏移量
function setStartOffset() {
  let startOffset = null
  if (startIndex.value >= 1) {
    let dValue =
      positions.value[startIndex.value].top -
      (positions.value[startIndex.value - aboveCount.value]
        ? positions.value[startIndex.value - aboveCount.value].top
        : 0)
    startOffset = positions.value[startIndex.value].top - dValue
  } else {
    startOffset = 0
  }
  listRef.value.style.transform = `translate3d(0, ${startOffset}px, 0)`
}

function binarySearch(list, value) {
  let start = 0
  let end = list.length - 1
  let tempIndex = null

  while (start <= end) {
    let midIndex = parseInt((start + end) / 2)
    let midValue = list[midIndex].bottom
    if (midValue === value) {
      return midIndex + 1
    } else if (midValue < value) {
      start = midIndex + 1
    } else {
      if (!tempIndex || tempIndex > midIndex) {
        tempIndex = midIndex
      }
      end = end - 1
    }
  }

  return tempIndex
}
// 计算起始索引
function setStartIndex(scrollTop = 0) {
  // let item = positions.value.find(v => v && v.bottom > scrollTop)
  // return item.index
  // 二分法优化计算次数
  return binarySearch(positions.value, scrollTop)
}

function handleScroll() {
  let scrollTop = containerRef.value.scrollTop
  startIndex.value = setStartIndex(scrollTop)
  endIndex.value = startIndex.value + visibleCount.value
  setStartOffset()
}

// 计算列表项实际高度
function updateItemHeight() {
  let nodes = itemsRef.value
  nodes.forEach((node) => {
    let rect = node.getBoundingClientRect()
    let height = rect.height
    let index = +node.id.slice(1)
    let dValue = positions.value[index].height - height
    // 如果存在差值，重新计算当前索引及其后所有数据的位置
    if (dValue) {
      positions.value[index].height = height
      positions.value[index].bottom = positions.value[index].bottom - dValue
      for (let k = index + 1; k < positions.value.length; k++) {
        positions.value[k].top = positions.value[k - 1].bottom
        positions.value[k].bottom = positions.value[k].bottom - dValue
      }
    }
  })
}

onMounted(() => {
  initPositions()
  screenHeight.value = containerRef.value.clientHeight
  startIndex.value = 0
  endIndex.value = startIndex.value + visibleCount.value
  setStartOffset()
})

// 渲染完成后，获取列表项的位置并缓存
onUpdated(() => {
  nextTick(() => {
    if (!itemsRef.value || !itemsRef.value.length) return
    updateItemHeight()
    // 获取列表总高度
    let height = positions.value[positions.value.length - 1].bottom
    phantomRef.value.style.height = height + 'px'
    setStartOffset()
  })
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
