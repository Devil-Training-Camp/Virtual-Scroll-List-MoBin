<template>
  <!-- 容器 -->
  <div
    class="virtual-scroll-container"
    ref="vListRef"
    :style="{ height }"
    @scroll="handleScroll"
  >
    <!-- 不可视占位，撑开容器滚动条，高度为列表总高度 -->
    <div class="virtual-scroll-phantom" ref="vPhantomRef"></div>
    <!-- 最终渲染出的列表 -->
    <div class="virtual-scroll-list" ref="vContentRef">
      <div
        class="virtual-scroll-item"
        v-for="item in visibleData"
        :key="item.index"
        :id="item.index"
        ref="vItemsRefs"
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
    default: []
  },
  // 预估高度
  estimatedItemHeight: {
    type: Number,
    required: true
  },
  // 缓冲区比例：接收缓冲区数据与可视区数据的比例
  bufferScale: {
    type: Number,
    default: 1
  },
  // 容器高度
  height: {
    type: String,
    default: '100%'
  }
})

let vListRef = ref(null)
let vContentRef = ref(null)
let vItemsRefs = ref(null)
let vPhantomRef = ref(null)
// 可视区高度
let screenHeight = ref(0)
// 起始索引
let startIndex = ref(0)
// 结束索引
let endIndex = ref(0)
// 列表项渲染后存储每一项的高度以及位置信息
let positions = ref([
  // {
  //   top: 0,
  //   bottom: 100,
  //   height: 100
  // }
])

// 复制一份全量数据，避免修改到原始数据
const _listData = computed(() => {
  return props.listData.map((item, index) => {
    return {
      index,
      item
    }
  })
})
// 可视区数据数目
const visibleCount = computed(() => {
  return Math.ceil(screenHeight.value / props.estimatedItemHeight)
})
// 可视区上方数据数目
const aboveCount = computed(() => {
  return Math.min(startIndex.value, props.bufferScale * visibleCount.value)
})
// 可视区下方数据数目
const blowCount = computed(() => {
  return Math.min(
    _listData.value.length - endIndex.value,
    props.bufferScale * visibleCount.value
  )
})
// 可视区数据数目
const visibleData = computed(() => {
  let start = startIndex.value - aboveCount.value
  let end = endIndex.value + blowCount.value
  return _listData.value.slice(start, end)
})

// 初始化位置信息
function initPositions() {
  positions.value = props.listData.map((item, index) => {
    return {
      index,
      height: props.estimatedItemHeight,
      top: index * props.estimatedItemHeight,
      bottom: (index + 1) * props.estimatedItemHeight
    }
  })
}

// 获取列表起始索引
function getStartIndex(scrollTop = 0) {
  // let item = positions.value.find(i => i && i.bottom > scrollTop)
  // return item.index
  // 二分法减少搜索次数
  return binarySearch(positions.value, scrollTop)
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
    } else if (midValue > value) {
      if (tempIndex === null || tempIndex > midIndex) {
        tempIndex = midIndex
      }
      end = end - 1
    }
  }
  return tempIndex
}

// 获取列表项的当前尺寸
function updateItemHeight() {
  let nodes = vItemsRefs.value
  nodes.forEach(node => {
    let rect = node.getBoundingClientRect()
    let height = rect.height
    let index = +node.id.slice(0)
    let oldHeight = positions.value[index].height
    let dValue = oldHeight - height
    // 存在差值
    if (dValue) {
      positions.value[index].bottom = positions.value[index].bottom - dValue
      positions.value[index].height = height
      for (let k = index + 1; k < positions.value.length; k++) {
        positions.value[k].top = positions.value[k - 1].bottom
        positions.value[k].bottom = positions.value[k].bottom - dValue
      }
    }
  })
}

// 获取当前的偏移量
function setStartOffset() {
  let startOffset = null
  if (startIndex.value >= 1) {
    let size =
      positions.value[startIndex.value].top -
      (positions.value[startIndex.value - aboveCount.value]
        ? positions.value[startIndex.value - aboveCount.value].top
        : 0)
    startOffset = positions.value[startIndex.value - 1].bottom - size
  } else {
    startOffset = 0
  }
  vContentRef.value.style.transform = `translate3d(0, ${startOffset}px, 0)`
}

function handleScroll() {
  // 滚动后的位置
  let scrollTop = vListRef.value.scrollTop
  // 滚动后的索引
  startIndex.value = getStartIndex(scrollTop)
  endIndex.value = startIndex.value + visibleCount.value
  // 滚动后的偏移量
  setStartOffset()
}

onMounted(() => {
  // dom挂载后获取可视区高度
  screenHeight.value = vListRef.value.clientHeight
  // 初始化数据
  initPositions()
  startIndex.value = 0
  endIndex.value = startIndex.value + visibleCount.value
})

// 需要在渲染完成后，获取列表项每一项的位置并缓存
onUpdated(() => {
  nextTick(() => {
    if (!vItemsRefs.value || !vItemsRefs.value.length) {
      return
    }
    // 获取真实元素大小，修改对应尺寸缓存
    updateItemHeight()
    // 更新占位列表总高度
    let height = positions.value[positions.value.length - 1].bottom
    vPhantomRef.value.style.height = height + 'px'
    // 更新偏移量
    setStartOffset()
  })
})
</script>

<style lang="scss" scoped>
.virtual-scroll-container {
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
