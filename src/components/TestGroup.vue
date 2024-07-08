<template>
  <div class="test-group">
    <h2>{{ group.name }}</h2>
    <div class="test-cases">
      <div 
        v-for="(testCase, index) in group.testCases" 
        :key="index" 
        class="test-case"
        :class="{ 
          'running': testCase.status === 'running',
          'success': testCase.status === 'success',
          'failure': testCase.status === 'failure'
        }"
        @click="runSingleTest(index)"
      >
        <h3>{{ testCase.name }}</h3>
        <p>状态: {{ getStatusText(testCase.status) }}</p>
        <p v-if="testCase.result">结果: {{ testCase.result }}</p>
      </div>
    </div>
    <button @click="runAllTests" :disabled="isRunning">运行所有测试</button>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'TestGroup',
  data() {
    return {
      group: {
        name: '拉流测试',
        testCases: [
          { name: '推流', status: 'idle', result: null },
          { name: '拉流', status: 'idle', result: null },
          { name: '拉流探测', status: 'idle', result: null },
          { name: '断流', status: 'idle', result: null },
          { name: '重新拉流', status: 'idle', result: null },
        ]
      },
      isRunning: false
    }
  },
  methods: {
    getStatusText(status) {
      const statusMap = {
        'idle': '未开始',
        'running': '进行中',
        'success': '成功',
        'failure': '失败'
      };
      return statusMap[status] || status;
    },
    async runTest(testCase) {
      testCase.status = 'running';
      try {
        const response = await axios.post('http://localhost:8080/runTest', { testName: testCase.name });
        testCase.status = response.data.success ? 'success' : 'failure';
        testCase.result = response.data.result;
        return response.data.success;
      } catch (error) {
        testCase.status = 'failure';
        testCase.result = error.message;
        return false;
      }
    },
    async runAllTests() {
      this.isRunning = true;
      for (let testCase of this.group.testCases) {
        const success = await this.runTest(testCase);
        if (!success) break;
      }
      this.isRunning = false;
    },
    async runSingleTest(index) {
      if (this.isRunning || this.group.testCases[index].status === 'running') {
        return;
      }
      await this.runTest(this.group.testCases[index]);
    }
  }
}
</script>

<style scoped>
.test-group {
  margin: 20px;
  padding: 20px;
  border: 1px solid #ddd;
}
.test-cases {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}
.test-case {
  width: 200px;
  padding: 10px;
  border: 1px solid #eee;
  cursor: pointer;
  transition: all 0.3s ease;
}
.test-case.running {
  background-color: #f0f0f0;
  cursor: not-allowed;
}
.test-case.success {
  background-color: #e6ffe6;
}
.test-case.failure {
  background-color: #ffe6e6;
}
button {
  margin-top: 20px;
  padding: 10px 20px;
  font-size: 16px;
}
</style>
