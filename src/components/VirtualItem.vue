<template>
  <div>
    <span style="color: #f00">{{ item.id }}</span>
    &nbsp;
    <span>{{ item.value }}</span>
  </div>
</template>

<script setup>
const props = defineProps({
  item: {
    type: Object,
    default: () => {}
  }
})
</script>
