<template>
  <!-- 动态高度的虚拟列表 -->
  <DynamicList :list-data="data" :estimated-item-height="100">
    <template #default="slotProps">
      <DynamicListItem :item="slotProps.item" />
    </template>
  </DynamicList>

  <!-- 静态高度的虚拟列表 -->
  <!-- <StaticList :list-data="data" :item-height="100"></StaticList> -->
</template>

<script setup>
import { ref } from 'vue'
import { faker } from '@faker-js/faker'
import DynamicList from '@/components/DynamicList/VirtualScroll.vue'
import DynamicListItem from '@/components/DynamicList/VirtualItem.vue'
// import StaticList from '@/components/StaticList/VirtualScroll.vue'

let data = ref([])

// 生成随机数据
function initData() {
  for (let i = 0; i < 1000; i++) {
    data.value.push({
      id: i,
      value: faker.lorem.sentences()
      // value: i
    })
  }
}
initData()
</script>
