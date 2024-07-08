<template>
  <div id="app">
    <h1>测试运行器</h1>
    <div v-if="testGroupsLoaded">
      <TestGroup
          v-for="group in testGroups"
          :key="group.id"
          :group="group"
      />
    </div>
    <div v-else>
      加载中...
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue';
import TestGroup from './components/TestGroup.vue';

export default {
  name: 'App',
  components: {
    TestGroup
  },
  setup() {
    const testGroups = ref([]);
    const testGroupsLoaded = ref(false);

    const loadTestGroups = () => {
      // 模拟异步加载数据
      setTimeout(() => {
        testGroups.value = [
          {
            id: 1,
            name: "功能测试",
            testCases: [
              { id: 1, name: "登录测试", status: "idle" },
              { id: 2, name: "注册测试", status: "idle" },
              { id: 3, name: "搜索测试", status: "idle" }
            ]
          },
          {
            id: 2,
            name: "性能测试",
            testCases: [
              { id: 4, name: "加载速度测试", status: "idle" },
              { id: 5, name: "并发用户测试", status: "idle" }
            ]
          }
        ];
        testGroupsLoaded.value = true;
      }, 1000);
    };

    onMounted(loadTestGroups);

    return {
      testGroups,
      testGroupsLoaded
    };
  }
};
</script>

<style>
body {
  font-family: Arial, sans-serif;
  background-color: #f0f2f5;
  margin: 0;
  padding: 0;
}

#app {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

h1 {
  color: #333;
  text-align: center;
}
</style>