<template>
    <div class="test-group">
        <h3 class="group-title">{{ group.name }}</h3>
        <div class="test-cases">
            <div v-for="testCase in group.testCases" :key="testCase.id" class="test-case" :class="testCase.status" @click="runSingleTest(testCase)">
                <div class="test-name">{{ testCase.name }}</div>
                <div class="test-status">状态: {{ getStatusText(testCase.status) }}</div>
                <div class="test-result">结果: {{ testCase.result }}</div>
            </div>
        </div>
        <button class="run-all-btn" @click="runAllTests" :disabled="isRunning">运行所有测试</button>
    </div>
</template>

<script>
import { ref } from 'vue';

export default {
    name: 'TestGroup',
    props: {
        group: {
            type: Object,
            required: true
        },
        sharedState: {
            type: Object,
            required: true
        }
    },
    setup(props) {
        const isRunning = ref(false);

        const getStatusText = (status) => {
            const statusMap = {
                'idle': '未开始',
                'running': '进行中',
                'success': '成功',
                'failure': '失败'
            };
            return statusMap[status] || status;
        };

        const runTest = async (testCase) => {
            testCase.status = 'running';
            const result = await testCase.testFunction(props.sharedState);
            testCase.status = result;
            testCase.result = result === 'success' ? '测试通过' : '测试失败';
            return result === 'success';
        };

        const runAllTests = async () => {
            isRunning.value = true;
            for (let testCase of props.group.testCases) {
                const success = await runTest(testCase);
                if (!success) break;
            }
            isRunning.value = false;
        };

        const runSingleTest = async (testCase) => {
            if (isRunning.value || testCase.status === 'running') {
                return;
            }
            await runTest(testCase);
        };

        return {
            isRunning,
            getStatusText,
            runAllTests,
            runSingleTest
        };
    }
};
</script>

<style scoped>
.test-group {
    background-color: #ffffff;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    padding: 20px;
    margin: 20px 0;
}

.group-title {
    color: #333;
    margin-top: 0;
    margin-bottom: 20px;
    font-size: 1.5em;
}

.test-cases {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
    gap: 15px;
    margin-bottom: 20px;
}

.test-case {
    background-color: #f9f9f9;
    border-radius: 6px;
    padding: 15px;
    cursor: pointer;
    transition: all 0.3s ease;
}

.test-case:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.test-case.running {
    background-color: #fff9c4;
}

.test-case.success {
    background-color: #c8e6c9;
}

.test-case.failure {
    background-color: #ffcdd2;
}

.test-name {
    margin: 0 0 10px 0;
    font-size: 1.1em;
    color: #444;
}

.test-status, .test-result {
    margin: 5px 0;
    font-size: 0.9em;
    color: #666;
}

.run-all-btn {
    background-color: #4CAF50;
    border: none;
    color: white;
    padding: 10px 20px;
    text-align: center;
    text-decoration: none;
    display: inline-block;
    font-size: 16px;
    margin-top: 10px;
    cursor: pointer;
    border-radius: 4px;
    transition: background-color 0.3s ease;
}

.run-all-btn:hover {
    background-color: #45a049;
}

.run-all-btn:disabled {
    background-color: #cccccc;
    cursor: not-allowed;
}
</style>