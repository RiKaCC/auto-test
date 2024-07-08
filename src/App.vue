<template>
  <div id="app">
    <h1>服务测试系统</h1>
    <TestGroup />
  </div>
</template>

<script>
import TestGroup from './components/TestGroup.vue'

export default {
  name: 'App',
  components: {
    TestGroup
  }
}
</script>
