<script setup>
import { ref, reactive } from 'vue'
import CronGenerator from './components/CronGenerator.vue'

// 用于独立使用示例的 ref
const standaloneConfig = ref({
  scheduleType: '',
  cron: ''
});

// 用于表单集成示例的 reactive 对象
const form = reactive({
    name: '我的定时任务',
    description: '这是一个任务描述',
    scheduleConfig: {
        scheduleType: 'periodic',
        cron: '0 0 * * * ?' // 默认值
    }
})

const submitForm = () => {
    // 在这里，你可以将 `form` 对象提交到后端
   console.log('Submitting form:', form)
    alert('表单数据已打印到控制台，请查看！')
}
</script>

<template>
  <div id="app-container">
    <h1 class="main-title">Cron 表达式生成器</h1>

    <!-- 独立使用示例 -->
    <div class="standalone-container">
      <h2 class="sub-title">独立使用示例 (带回显功能)</h2>
       <el-input v-model="standaloneConfig.cron" placeholder="在此输入或粘贴Cron表达式进行测试" class="cron-input-tester">
         <template #prepend>
            <el-select v-model="standaloneConfig.scheduleType" style="width: 115px">
                <el-option label="周期性" value="periodic" />
                <el-option label="一次性" value="onetime" />
                <el-option label="自定义" value="custom" />
            </el-select>
         </template>
       </el-input>
      <cron-generator v-model:schedule-config="standaloneConfig" />
    </div>

    <!-- 集成示例 -->
    <div class="form-example">
      <h2 class="sub-title">集成示例</h2>
      <el-form label-width="100px" :model="form">
        <el-form-item label="任务名称">
          <el-input v-model="form.name"></el-input>
        </el-form-item>
        <el-form-item label="任务描述">
          <el-input v-model="form.description" type="textarea"></el-input>
        </el-form-item>
         <el-form-item label="调度配置">
            <cron-generator v-model:schedule-config="form.scheduleConfig" />
         </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="submitForm">提交</el-button>
        </el-form-item>
      </el-form>

      <div class="result-display">
        <h3 class="result-title">最终表单数据:</h3>
        <pre class="code-block">{{ form }}</pre>
      </div>
    </div>

  </div>
</template>

<style scoped>
#app-container {
  padding: 20px;
  max-width: 900px;
  margin: 0 auto;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
}

.main-title {
  text-align: center;
  margin-bottom: 2rem;
  color: #333;
}

.sub-title {
  margin-bottom: 1rem;
  color: #444;
  border-bottom: 1px solid #eee;
  padding-bottom: 0.5rem;
}

.standalone-container,
.form-example {
  margin-bottom: 2.5rem;
}

.result-display {
  margin-top: 1rem;
  padding: 20px;
  border: 1px solid #ebeef5;
  border-radius: 4px;
  background-color: #fafafa;
}

.result-title {
  margin: 0 0 0.5rem 0;
  font-size: 1rem;
  font-weight: normal;
  color: #666;
}

.code-block {
  color: #666;
  background-color: #f5f5f5;
  border: 1px solid #ddd;
  padding: 15px;
  border-radius: 4px;
  white-space: pre-wrap;
  word-wrap: break-word;
  font-family: 'Courier New', Courier, monospace;
}

.cron-input-tester {
  margin-bottom: 1rem;
}
</style>
