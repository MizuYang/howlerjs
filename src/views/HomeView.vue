<template>
  <template v-for="item in data" :key="item.id">
    <component :is="item.component"></component>
  </template>
</template>

<script setup>
import { reactive, defineAsyncComponent, onMounted } from 'vue'

const data = reactive([
  {
    id: 1,
    name: 'Demo1',
    component: ''
  }
])

onMounted(() => {
  getComponents()
})

function getComponents () {
  data.forEach(item => {
    item.component = defineAsyncComponent(() => import(`@/components/${item.name}.vue`))
  })
}
</script>

<style lang='scss' scope></style>
