
# Cron 表达式生成器组件 (`CronGenerator.vue`)

这是一个基于 Vue 3 和 Element Plus 的高级 Cron 表达式生成与解析组件。它提供了一个直观的用户界面，帮助用户轻松创建复杂的 Cron 调度规则，同时也支持解析已有的 Cron 表达式并回显到 UI 界面上。

## 功能特性

- **多种调度类型**：支持"周期性"、"一次性"和"自定义"三种模式。
- **丰富的配置选项**：
  - **简单重复**：按分钟、小时、天、周进行重复执行。
  - **周期性执行**：可精确到按天或按周，并指定特定月份。
  - **一次性执行**：选择一个精确的未来时间点执行。
  - **自定义**：直接输入或使用预置模板。
- **双向数据绑定**：通过 `v-model:scheduleConfig` 与父组件通信，数据流清晰。
- **智能回显**：能解析传入的配置对象，自动更新 UI 至对应状态。

## 依赖

- Vue 3
- Element Plus

## 如何使用

1.  **引入组件**:
    在您的父组件中，引入 `CronGenerator.vue`。

    ```vue
    <script setup>
    import { ref } from 'vue';
    import CronGenerator from './components/CronGenerator.vue';

    const scheduleConfig = ref({
      scheduleType: 'periodic',
      cron: '0 0 * * * ?'
    });
    </script>
    ```

2.  **在模板中使用**:
    使用 `v-model:schedule-config` 将组件与您的数据进行绑定。

    ```vue
    <template>
      <cron-generator v-model:schedule-config="scheduleConfig" />
    </template>
    ```

## `v-model` 数据结构

组件通过 `v-model:scheduleConfig` 与父组件进行数据交互。这个 `scheduleConfig` 是一个对象，其结构如下：

```javascript
{
  scheduleType: String, // 调度类型
  cron: String          // Cron 表达式
}
```

- **`scheduleType`** (必填): 用于告知组件当前应该展示哪种调度界面。
  - `'periodic'`: 周期性
  - `'onetime'`: 一次性
  - `'custom'`: 自定义

- **`cron`** (必填): 标准的 Cron 表达式字符串。

## 参数与回显说明

要实现 Cron 表达式的回显功能，您只需要向组件传递一个符合上述结构的 `scheduleConfig` 对象即可。组件内部会监听这个对象的变化，并自动完成 UI 的更新。

#### 示例 1: 回显一个"周期性"任务

**父组件数据**:
```javascript
const scheduleConfig = ref({
  scheduleType: 'periodic',
  cron: '0 15 10 ? 6 1,2' // 每月6月的周一和周二，上午10点15分执行
});
```
组件会自动切换到"周期性" -> "周期性执行"，并填充好对应的"执行日"、"执行月"和"执行时间"。

#### 示例 2: 回显一个"一次性"任务

**父组件数据**:
```javascript
const scheduleConfig = ref({
  scheduleType: 'onetime',
  cron: '0 30 8 1 1 ? 2025' // 2025年1月1日，早上8点30分执行
});
```
组件会自动切换到"一次性"模式，并填充好日期时间选择器。

#### 示例 3: 回显一个"自定义"任务

**父组件数据**:
```javascript
const scheduleConfig = ref({
  scheduleType: 'custom',
  cron: '0 0 12 LW * ?' // 每月最后一个工作日中午12点执行
});
```
组件会自动切换到"自定义"模式，并在输入框中填入该表达式。

## 返回值

当用户在组件 UI 上进行操作时，组件会**实时地**通过 `update:scheduleConfig` 事件向父组件同步最新的配置对象。返回的数据结构与传入的参数结构完全一致。您可以监听这个数据变化，并将其提交到后端或用于其他业务逻辑。
