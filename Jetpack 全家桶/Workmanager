# 面试者视角：WorkManager 核心总结（Android 高频考点）

## 一、核心定位与适用场景（必背）

### 1. 核心定义
**WorkManager** 是 Android Jetpack 官方提供的后台任务调度框架，用于处理**可延迟、需保证最终执行**的后台任务，兼容 **Android 5.0（API 21）及以上版本**。

### 2. 核心特性
- **任务持久化**：应用退出、设备重启后，满足条件的任务仍能执行；
- **智能约束**：支持网络、充电、电量、设备空闲等约束条件，节省资源；
- **任务编排**：支持链式调度（串行 / 并行）、周期性执行、任务去重；
- **线程管理**：`doWork()` 自动运行在子线程，无需手动切换。

### 3. 典型适用场景（面试必答）

| 场景类型       | 具体案例 |
|----------------|---------|
| 数据同步       | 日志上报、本地缓存与服务器增量同步、离线表单提交 |
| 媒体处理       | 视频压缩、图片批量处理、大文件下载 / 上传 |
| 定时任务       | 每日数据备份、周期性数据拉取（最小周期 15 分钟） |
| 复杂调度       | 多步骤任务依赖（下载→解压→导入）、任务优先级控制 |

### 4. 不适用场景（避坑点）
- **实时任务**（如即时消息、实时定位）：需用 FCM / 定位 SDK；
- **前台持续任务**（如音乐播放、导航）：需用 Foreground Service；
- **页面内短时任务**（如网络请求）：用协程 + Retrofit 更高效。

---

## 二、核心组件与工作流程（面试重点）

### 1. 三大核心组件

| 组件           | 作用                         | 关键要点 |
|----------------|------------------------------|----------|
| `Worker`       | 定义后台任务逻辑             | 继承 `Worker` 类，重写 `doWork()` 方法，返回 `Result.success()`/`failure()`；可通过 `inputData` 传参 |
| `WorkRequest`  | 任务请求配置                 | 分 `OneTimeWorkRequest`（单次）和 `PeriodicWorkRequest`（周期性）；可通过 `Constraints` 配置约束 |
| `WorkManager`  | 任务调度入口                 | 调用 `getInstance()` 获取实例，通过 `enqueue()` 提交任务；支持任务监听、取消 |

### 2. 核心工作流程
1. 定义 `Worker` 子类，实现 `doWork()` 业务逻辑；
2. 构建 `WorkRequest`（配置参数、约束、调度规则）；
3. 通过 `WorkManager` 提交任务，系统根据设备状态智能调度；
4. 监听任务状态（成功 / 失败 / 运行中），处理结果。

---

## 三、关键 API 与代码实现（面试实操）

### 1. 基础使用（单次任务）

```kotlin
// 1. 自定义 Worker
class LogUploadWorker(context: Context, params: WorkerParameters) : Worker(context, params) {
    override fun doWork(): Result {
        val userId = inputData.getString("user_id") ?: "unknown"
        uploadLog(userId) // 业务逻辑
        return Result.success() // 成功
    }
}

// 2. 提交任务
val inputData = Data.Builder().putString("user_id", "10086").build()
val workRequest = OneTimeWorkRequestBuilder<LogUploadWorker>()
    .setInputData(inputData)
    .build()
WorkManager.getInstance(this).enqueue(workRequest)
