<template>
    <div id="app">
        <h1>AuroraLive Test</h1>
        <div v-if="testGroupsLoaded">
            <h2>可用测试用例</h2>
            <div v-for="testCase in testCasePool" :key="testCase.id" class="test-case-item">
                <input type="checkbox" :value="testCase" v-model="selectedTestCases">
                <span>{{ testCase.name }}</span>
            </div>
            <div class="test-group-input">
                <input v-model="newTestGroupName" placeholder="输入测试组名称">
                <button @click="createTestGroup" :disabled="selectedTestCases.length === 0 || !newTestGroupName">创建测试组</button>
            </div>
            <hr>
            <div v-for="group in testGroups" :key="group.id">
                <TestGroup :group="group" :sharedState="sharedState"/>
            </div>
        </div>
        <div v-else>
            加载中...
        </div>
    </div>
</template>

<script>
import { ref, onMounted } from 'vue';
import axios from 'axios';
import TestGroup from './components/TestGroup.vue';

export default {
    name: 'App',
    components: {
        TestGroup
    },
    setup() {
        const testCasePool = ref([]);
        const selectedTestCases = ref([]);
        const testGroups = ref([]);
        const testGroupsLoaded = ref(false);
        const newTestGroupName = ref('');
        const sharedState = ref({});  // 初始化 sharedState

        const loadTestCases = async () => {
            // 模拟异步加载数据
            await new Promise(resolve => setTimeout(resolve, 1000));  // 模拟延迟
            testCasePool.value = [
                { id: 1, name: "创建流", status: "idle", testFunction: createStream },
                { id: 2, name: "推流", status: "idle", testFunction: publish },
                { id: 3, name: "Pre拉流", status: "idle", testFunction: prePull },
                { id: 4, name: "拉流", status: "idle", testFunction: pull },
                { id: 5, name: "断流", status: "idle", testFunction: unPublish },
                { id: 6, name: "End拉流", status: "idle", testFunction: endPull }
            ];
            testGroupsLoaded.value = true;
        };

        const createTestGroup = () => {
            const newGroupId = testGroups.value.length + 1;
            testGroups.value.push({
                id: newGroupId,
                name: newTestGroupName.value || `测试组 ${newGroupId}`,
                testCases: selectedTestCases.value.map(tc => ({ ...tc }))
            });
            selectedTestCases.value = [];
            newTestGroupName.value = '';
        };

        const createStream = async (state) => {
            try {
                const response = await axios.post('/api/stream/create');
                if (response.status === 201) {
                    state.streamData = response.data;  // 将响应数据保存到 sharedState
                    return 'success';
                }
                return 'failure';
            } catch {
                return 'failure';
            }
        };

        const publish = async (state) => {
            try {
                const response = await axios.post('/api/stream/publish', {
                    streamId: state.streamData.id  // 从 sharedState 中获取 streamId
                });
                return response.status === 200 ? 'success' : 'failure';
            } catch {
                return 'failure';
            }
        };

        const prePull = async (state) => {
            try {
                const response = await axios.get('/api/stream/prepull', {
                    params: { streamId: state.streamData.id }  // 从 sharedState 中获取 streamId
                });
                return response.status === 200 ? 'success' : 'failure';
            } catch {
                return 'failure';
            }
        };

        const pull = async (state) => {
            try {
                const response = await axios.get('/api/stream/pull', {
                    params: { streamId: state.streamData.id }  // 从 sharedState 中获取 streamId
                });
                return response.status === 200 ? 'success' : 'failure';
            } catch {
                return 'failure';
            }
        };

        const unPublish = async (state) => {
            try {
                const response = await axios.post('/api/stream/unpublish', {
                    streamId: state.streamData.id  // 从 sharedState 中获取 streamId
                });
                return response.status === 200 ? 'success' : 'failure';
            } catch {
                return 'failure';
            }
        };

        const endPull = async (state) => {
            try {
                const response = await axios.post('/api/stream/endpull', {
                    streamId: state.streamData.id  // 从 sharedState 中获取 streamId
                });
                return response.status === 200 ? 'success' : 'failure';
            } catch {
                return 'failure';
            }
        };

        onMounted(loadTestCases);

        return {
            testCasePool,
            selectedTestCases,
            testGroups,
            testGroupsLoaded,
            newTestGroupName,
            createTestGroup,
            sharedState  // 确保返回 sharedState
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

h1, h2 {
    color: #333;
    text-align: center;
}

.test-case-item {
    display: flex;
    align-items: center;
    margin: 5px 0;
}

.test-case-item input {
    margin-right: 10px;
}

.test-group-input {
    display: flex;
    align-items: center;
    justify-content: center;
    margin: 20px 0;
}

.test-group-input input {
    padding: 10px;
    font-size: 16px;
    margin-right: 10px;
    width: 300px;
    border: 1px solid #ccc;
    border-radius: 4px;
}

button {
    padding: 10px 20px;
    border: none;
    background-color: #4CAF50;
    color: white;
    cursor: pointer;
    border-radius: 4px;
    transition: background-color 0.3s ease;
}

button:disabled {
    background-color: #cccccc;
    cursor: not-allowed;
}
</style>