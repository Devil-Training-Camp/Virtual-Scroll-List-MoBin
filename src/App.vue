<template>
  <!-- 模拟一个窄一点的列表 -->
  <div style="width: 450px; height: 100%">
    <VirtualScroll
      :list-data="data"
      :estimated-item-height="100"
      #default="slotProps"
    >
      <VirtualItem :item="slotProps.item" />
    </VirtualScroll>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { faker } from '@faker-js/faker'
import VirtualScroll from '@/components/VirtualScroll.vue'
import VirtualItem from '@/components/VirtualItem.vue'

let data = ref([])

// 生成随机数据
function initData() {
  for (let i = 0; i < 1000; i++) {
    data.value.push({
      id: i,
      value: faker.lorem.sentences()
    })
  }
}
initData()
</script>
