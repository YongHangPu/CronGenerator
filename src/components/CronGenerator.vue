<template>
  <div class="cron-generator-container">
    <el-form label-width="100px">
      <el-form-item label="调度类型">
        <el-radio-group v-model="scheduleType">
          <el-radio value="periodic">周期性</el-radio>
          <el-radio value="onetime">一次性</el-radio>
          <el-radio value="custom">自定义</el-radio>
        </el-radio-group>
      </el-form-item>

      <!-- 一次性 -->
      <div v-if="scheduleType === 'onetime'">
        <el-form-item label="执行时间">
          <el-date-picker
            v-model="onetime.dateTime"
            type="datetime"
            placeholder="选择日期和时间"
            value-format="YYYY-MM-DD HH:mm:ss"
            class="full-width" />
        </el-form-item>
      </div>

      <!-- 周期性 -->
      <div v-if="scheduleType === 'periodic'">
        <el-form-item label="执行频率">
          <el-radio-group v-model="periodic.frequencyType">
            <el-radio value="simple">简单重复执行</el-radio>
            <el-radio value="periodic">周期性执行</el-radio>
          </el-radio-group>
        </el-form-item>

        <!-- 简单重复执行 -->
        <template v-if="periodic.frequencyType === 'simple'">
          <el-form-item label="执行间隔">
            <div class="cron-row">
              <el-input-number v-model="periodic.simple.interval" :min="1" class="interval-input" />
              <el-select v-model="periodic.simple.unit" placeholder="请选择" class="interval-unit-select">
                <el-option label="分钟" value="minute"></el-option>
                <el-option label="小时" value="hour"></el-option>
                <el-option label="天" value="day"></el-option>
                <el-option label="周" value="week"></el-option>
              </el-select>
            </div>
          </el-form-item>
          <el-form-item label="执行时间" v-if="['day', 'week'].includes(periodic.simple.unit)">
            <el-time-picker v-model="periodic.simple.time" format="HH:mm" value-format="HH:mm" placeholder="选择时间" />
          </el-form-item>
        </template>

        <!-- 周期性执行 -->
        <template v-if="periodic.frequencyType === 'periodic'">
          <el-form-item label="执行日">
            <div class="cron-row">
              <el-select v-model="periodic.periodic.dayType" class="day-type-select">
                <el-option label="日" value="day"></el-option>
                <el-option label="周" value="week"></el-option>
              </el-select>
              <el-select
                v-if="periodic.periodic.dayType === 'day'"
                v-model="periodic.periodic.daysOfMonth"
                multiple
                placeholder="请选择日"
                class="day-value-select">
                <el-option
                  v-for="item in dayOfMonthOptions"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value" />
              </el-select>
              <el-select
                v-if="periodic.periodic.dayType === 'week'"
                v-model="periodic.periodic.daysOfWeek"
                multiple
                placeholder="请选择周"
                class="day-value-select">
                <el-option v-for="item in weekOptions" :key="item.value" :label="item.label" :value="item.value" />
              </el-select>
            </div>
          </el-form-item>
          <el-form-item label="执行月">
            <el-select v-model="periodic.periodic.months" multiple placeholder="请选择月" class="full-width">
              <el-option v-for="item in monthOptions" :key="item.value" :label="item.label" :value="item.value" />
            </el-select>
          </el-form-item>
          <el-form-item label="执行时间">
            <el-time-picker
              v-model="periodic.periodic.time"
              format="HH:mm"
              value-format="HH:mm"
              placeholder="选择时间" />
          </el-form-item>
        </template>
      </div>

      <!-- 自定义 -->
      <div v-if="scheduleType === 'custom'">
        <el-form-item label="Cron表达式">
          <div class="cron-row">
            <el-input v-model="custom.expression" placeholder="请输入Cron表达式"></el-input>
            <el-dropdown class="preset-dropdown">
              <el-button>
                预置表达式
                <el-icon class="el-icon--right"><arrow-down /></el-icon>
              </el-button>
              <template #dropdown>
                <el-dropdown-menu>
                  <el-dropdown-item v-for="item in presetCrons" :key="item.name" @click="custom.expression = item.cron">
                    {{ item.name }}
                  </el-dropdown-item>
                </el-dropdown-menu>
              </template>
            </el-dropdown>
          </div>
        </el-form-item>
      </div>
    </el-form>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import { ArrowDown } from '@element-plus/icons-vue'

// --- 组件通信 ---
const props = defineProps({
  scheduleConfig: {
    type: Object,
    default: () => ({
        scheduleType: 'periodic',
        cron: '* * * * * ?'
    })
  },
})

const emit = defineEmits(['update:scheduleConfig'])

// --- 响应式状态定义 ---

// 核心状态：当前选择的调度类型
const scheduleType = ref('periodic')

// '一次性' 调度类型的状态
const onetime = ref({
  dateTime: '' // 执行日期和时间
})

// '周期性' 调度类型的状态
const periodic = ref({
  // 频率类型: 'simple'-简单重复执行, 'periodic'-周期性执行
  frequencyType: 'simple',

  // '简单重复执行' 的配置
  simple: {
    interval: 1, // 执行间隔
    unit: 'minute', // 间隔单位: 'minute', 'hour', 'day', 'week'
    time: '00:00' // 执行时间（当单位为天或周时）
  },

  // '周期性执行' 的配置
  periodic: {
    dayType: 'week', // 执行日类型: 'day'-按天, 'week'-按周
    daysOfWeek: [], // 选择的周 (1-7)
    daysOfMonth: [], // 选择的日 (1-31)
    months: [], // 选择的月 (1-12)
    time: '00:00' // 执行时间
  }
})

// '自定义' 调度类型的状态
const custom = ref({
  expression: '* * * * * ?' // 自定义的Cron表达式
})

// --- 选项常量 ---

// 周选项 (星期一=1, 星期日=7)
const weekOptions = [
  { value: 1, label: '星期一' },
  { value: 2, label: '星期二' },
  { value: 3, label: '星期三' },
  { value: 4, label: '星期四' },
  { value: 5, label: '星期五' },
  { value: 6, label: '星期六' },
  { value: 7, label: '星期日' }
]

// 月份选项
const monthOptions = Array.from({ length: 12 }, (_, i) => ({
  value: i + 1,
  label: `${i + 1}月`
}))

// 日期选项
const dayOfMonthOptions = Array.from({ length: 31 }, (_, i) => ({
  value: i + 1,
  label: `${i + 1}日`
}))

// 预置的常用Cron表达式
const presetCrons = ref([
  { name: '每分钟执行', cron: '0 * * * * ?' },
  { name: '每5分钟执行', cron: '0 0/5 * * * ?' },
  { name: '每小时执行', cron: '0 0 * * * ?' },
  { name: '每天零点执行', cron: '0 0 0 * * ?' },
  { name: '每个工作日上午9点及下午6点', cron: '0 0 9,18 ? * 1-5' },
  { name: '每月最后一天23点59分', cron: '0 59 23 L * ?' },
  { name: '每月最后一个工作日19点50分', cron: '0 50 19 LW * ?' },
  { name: '每月第四个周五19点50分', cron: '0 50 19 ? * 5#4' },
  { name: '每月15日9点15分', cron: '0 15 9 15 * ?' }
])

// --- Cron 生成逻辑 ---

const generatedCron = computed(() => {
  // Cron 表达式各字段的默认值
  const second = '0'
  let minute = '*'
  let hour = '*'
  let dayOfMonth = '*'
  let month = '*'
  let dayOfWeek = '?'
  let year = ''

  switch (scheduleType.value) {
    // --- 一次性任务 ---
    case 'onetime': {
      if (!onetime.value.dateTime) return '' // 未选择时间则返回空
      const d = new Date(onetime.value.dateTime)
      minute = d.getMinutes()
      hour = d.getHours()
      dayOfMonth = d.getDate()
      month = d.getMonth() + 1
      year = d.getFullYear()
      // 返回一个精确到年月日时分的完整Cron表达式
      return `${second} ${minute} ${hour} ${dayOfMonth} ${month} ? ${year}`
    }
    // --- 周期性任务 ---
    case 'periodic': {
      // 简单重复执行
      if (periodic.value.frequencyType === 'simple') {
        const { interval, unit, time } = periodic.value.simple
        if (unit === 'minute') {
          minute = `0/${interval}`
        } else if (unit === 'hour') {
          minute = '0'
          hour = `*/${interval}`
        } else if (unit === 'day' || unit === 'week') {
          // 将'周'的间隔转换为'天'
          const finalInterval = unit === 'week' ? interval * 7 : interval
          const [h, m] = time.split(':')
          minute = parseInt(m, 10)
          hour = parseInt(h, 10)
          dayOfMonth = `*/${finalInterval}`
          dayOfWeek = '?' // 按天重复时，星期字段应为'?'
        }
        // 周期性执行
      } else {
        const { dayType, daysOfWeek, daysOfMonth, months, time } = periodic.value.periodic

        const [h, m] = (time || '00:00').split(':')
        minute = parseInt(m, 10)
        hour = parseInt(h, 10)

        // 处理'日'和'周'的互斥关系
        if (dayType === 'day') {
          dayOfMonth = daysOfMonth.length > 0 ? daysOfMonth.join(',') : '*'
          dayOfWeek = '?'
        } else {
          // dayType === 'week'
          dayOfMonth = '?'
          dayOfWeek = daysOfWeek.length > 0 ? daysOfWeek.join(',') : '*'
        }

        month = months.length > 0 ? months.join(',') : '*'

        // 当'日'和'周'都未指定时，Cron规范要求将一个设为'?'
        if (dayOfMonth === '*' && dayOfWeek === '*') {
          dayOfWeek = '?'
        }

        // 组合成最终的表达式
        return `0 ${minute} ${hour} ${dayOfMonth} ${month} ${dayOfWeek}`
      }
      break
    }
    // --- 自定义任务 ---
    case 'custom':
      return custom.value.expression
    default:
      // 默认情况下返回一个不执行的表达式
      return ''
  }

  // 组合简单重复执行的最终表达式
  return [second, minute, hour, dayOfMonth, month, dayOfWeek, year].filter(Boolean).join(' ')
})

// --- 完整配置对象生成逻辑 ---

const generatedConfig = computed(() => {
    return {
        scheduleType: scheduleType.value,
        cron: generatedCron.value,
    }
});

/**
 * 解析传入的 cron 字符串并更新 UI 状态
 * @param {string} cronString - 要解析的 cron 表达式
 */
const parseCron = (cronString) => {
  if (!cronString) {
    return
  }

  const parts = cronString.split(' ')
  // 对于无法解析的、或不符合基本格式的，直接归为"自定义"
  if (parts.length < 6 || parts.length > 7) {
    if(scheduleType.value === 'custom') {
      custom.value.expression = cronString;
    }
    return;
  }

  const [second, minute, hour, dayOfMonth, month, dayOfWeek, year] = parts

  // 根据 scheduleType 来决定如何解析
  if (scheduleType.value === 'onetime') {
    if (year && year !== '*') {
      onetime.value.dateTime = `${year}-${String(month).padStart(2,'0')}-${String(dayOfMonth).padStart(2,'0')} ${String(hour).padStart(2,'0')}:${String(minute).padStart(2,'0')}:${String(second).padStart(2,'0')}`;
    }
    return;
  }

  if (scheduleType.value === 'periodic') {
    // 解析周期性执行
    if ((dayOfWeek !== '?' && dayOfWeek !== '*') || month !== '*') {
      periodic.value.frequencyType = 'periodic';
      const pState = periodic.value.periodic;

      pState.time = `${String(hour).padStart(2, '0')}:${String(minute).padStart(2, '0')}`;
      pState.months = month === '*' ? [] : month.split(',').map(Number);

      if (dayOfWeek !== '?' && dayOfWeek !== '*') {
        pState.dayType = 'week';
        pState.daysOfWeek = dayOfWeek.split(',').map(Number);
        pState.daysOfMonth = [];
      } else {
        pState.dayType = 'day';
        pState.daysOfMonth = dayOfMonth === '*' ? [] : dayOfMonth.split(',').map(Number);
        pState.daysOfWeek = [];
      }
      return;
    }

    // 解析简单重复执行
    periodic.value.frequencyType = 'simple';
    const sState = periodic.value.simple;
    if (minute.startsWith('0/')) {
      sState.unit = 'minute';
      sState.interval = parseInt(minute.split('/')[1] || 1, 10);
      return;
    }
    if (hour.startsWith('*/')) {
      sState.unit = 'hour';
      sState.interval = parseInt(hour.split('/')[1] || 1, 10);
      sState.time = '00:00';
      return;
    }
    if (dayOfMonth.startsWith('*/')) {
      const isWeekly = parseInt(dayOfMonth.split('/')[1] || 1, 10) % 7 === 0;
       if (isWeekly) {
          sState.unit = 'week';
          sState.interval = parseInt(dayOfMonth.split('/')[1], 10) / 7;
       } else {
          sState.unit = 'day';
          sState.interval = parseInt(dayOfMonth.split('/')[1], 10);
       }
      sState.time = `${String(hour).padStart(2, '0')}:${String(minute).padStart(2, '0')}`;
      return;
    }
  }

  // 其他所有无法精确匹配的，都归为"自定义"
  if(scheduleType.value === 'custom') {
    custom.value.expression = cronString;
  }
};

// --- 监听与生命周期 ---

// 监听内部UI变化，生成新的配置对象并通知父组件
watch(generatedConfig, (newConfig) => {
  // 避免因外部传入props导致的循环更新
  if (newConfig.cron !== props.scheduleConfig.cron || newConfig.scheduleType !== props.scheduleConfig.scheduleType) {
      emit('update:scheduleConfig', newConfig);
  }
}, { deep: true });

// 监听外部传入的配置对象变化，并更新UI
watch(() => props.scheduleConfig, (newConfig) => {
  if (!newConfig) return;

  // 如果外部传入的配置和内部生成的一致，则不进行解析，避免循环更新
  if (newConfig.cron === generatedConfig.value.cron && newConfig.scheduleType === generatedConfig.value.scheduleType) {
    return;
  }

  // 首先，根据外部配置更新调度类型
  scheduleType.value = newConfig.scheduleType || 'periodic';

  // 然后，根据新的调度类型解析 cron 表达式来回显UI
  parseCron(newConfig.cron);

}, { immediate: true, deep: true });

</script>

<style scoped>
.cron-generator-container {
  border: 1px solid #dcdfe6;
  padding: 20px;
  border-radius: 4px;
  width: 100%; /* 确保组件在父表单项中能完全展开 */
  box-sizing: border-box; /* 避免padding和border影响总宽度 */
}

/* 用于将多个组件放在一行的容器 */
.cron-row {
  display: flex;
  width: 100%;
  gap: 10px; /* 元素间距 */
}

/* 占满父容器宽度 */
.full-width {
  width: 100%;
}

/* 间隔输入框 */
.interval-input {
  width: 120px;
}

/* 间隔单位选择器 */
.interval-unit-select {
  width: 100px;
}

/* 预置表达式下拉按钮 */
.preset-dropdown {
  margin-left: auto; /* 推动到最右边 */
  flex-shrink: 0; /* 防止被压缩 */
}

/* 执行日类型选择器（日/周） */
.day-type-select {
  width: 100px;
  flex-shrink: 0;
}

/* 执行日具体值选择器 */
.day-value-select {
  flex-grow: 1; /* 占据剩余空间 */
}

.el-icon--right {
  margin-left: 4px;
}
</style>
