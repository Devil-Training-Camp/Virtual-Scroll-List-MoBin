<template>
  <VirtualScroll :list-data="data" :item-height="itemHeight" />
</template>

<script setup>
import { ref } from 'vue'
import VirtualScroll from '@/components/VirtualScroll.vue'

let data = ref([])
let itemHeight = ref(100)

// 生成随机数据
function initData() {
  for (let i = 0; i < 1000; i++) {
    data.value.push({
      id: i,
      value: 'test - ' + i
    })
  }
}
initData()
</script>
