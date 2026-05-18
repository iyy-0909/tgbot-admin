<template>
  <el-card class="clone-card">
    <template #header>
      <div class="card-header">
        <div>
          <div class="card-title">克隆任务</div>
          <div class="card-subtitle">频道历史克隆任务管理</div>
        </div>

        <el-button type="primary" @click="emit('add')">
          新增任务
        </el-button>
      </div>
    </template>

    <el-table
      :data="tasks"
      border
      stripe
      class="clone-table"
      empty-text="暂无克隆任务"
    >
      <el-table-column prop="id" label="ID" width="70" align="center" />

      <el-table-column prop="name" label="任务名" min-width="150" />

      <el-table-column prop="source_channel" label="源频道" min-width="170">
        <template #default="{ row }">
          <el-tag type="info" effect="plain">
            {{ row.source_channel }}
          </el-tag>
        </template>
      </el-table-column>

      <el-table-column label="目标频道" min-width="240">
        <template #default="{ row }">
          <el-tooltip
            effect="dark"
            :content="formatTargets(row.target_channels)"
            placement="top"
          >
            <span class="target-text">
              {{ formatTargets(row.target_channels) }}
            </span>
          </el-tooltip>
        </template>
      </el-table-column>

      <el-table-column label="状态" width="100" align="center">
        <template #default="{ row }">
          <el-tag
            :type="getStatusType(row.status)"
            size="small"
          >
            {{ row.status || "idle" }}
          </el-tag>
        </template>
      </el-table-column>

      <el-table-column label="Worker" width="110" align="center">
        <template #default="{ row }">
          <el-tag
            v-if="row.worker_running"
            type="success"
            size="small"
          >
            运行中
          </el-tag>

          <el-tag
            v-else
            type="info"
            size="small"
          >
            未运行
          </el-tag>
        </template>
      </el-table-column>

      <el-table-column
        prop="last_message_id"
        label="进度"
        width="100"
        align="center"
      />

      <el-table-column label="操作" width="420" fixed="right">
        <template #default="{ row }">
          <div class="action-buttons">
            <el-button
              size="small"
              type="success"
              :disabled="row.worker_running"
              @click="emit('start', row.id)"
            >
              开始
            </el-button>

            <el-button
              size="small"
              type="warning"
              :disabled="!row.worker_running"
              @click="emit('pause', row.id)"
            >
              暂停
            </el-button>

            <el-button
              size="small"
              type="primary"
              :disabled="row.worker_running"
              @click="emit('resume', row.id)"
            >
              继续
            </el-button>

            <el-button
              size="small"
              type="danger"
              plain
              :disabled="!row.worker_running"
              @click="emit('stop', row.id)"
            >
              停止
            </el-button>

            <el-button
              size="small"
              @click="emit('edit', row)"
            >
              编辑
            </el-button>

            <el-button
              size="small"
              type="danger"
              @click="emit('delete', row)"
            >
              删除
            </el-button>
          </div>
        </template>
      </el-table-column>
      
        <el-table-column label="监听" width="100" align="center">
            <template #default="{ row }">
                <el-switch
                :model-value="row.enable_listener"
                size="small"
                active-text="开"
                inactive-text="关"
                @change="value => emit('toggle-listener', row, value)"
                />
            </template>
        </el-table-column>

    </el-table>
  </el-card>
</template>

<script setup>
defineProps({
  tasks: {
    type: Array,
    required: true,
  },
})

const emit = defineEmits([
  "add",
  "edit",
  "delete",
  "start",
  "pause",
  "resume",
  "stop",
  "toggle-listener",
])

const getStatusType = (status) => {
  const map = {
    idle: "info",
    running: "success",
    paused: "warning",
    stopped: "danger",
    done: "primary",
    error: "danger",
  }

  return map[status] || "info"
}

const formatTargets = (value) => {
  if (!value) {
    return "-"
  }

  try {
    const parsed = typeof value === "string" ? JSON.parse(value) : value

    if (Array.isArray(parsed)) {
      return parsed.join("，")
    }

    return String(parsed)
  } catch (e) {
    return String(value)
  }
}
</script>

<style scoped>
.clone-card {
  border-radius: 12px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.card-title {
  font-size: 16px;
  font-weight: 600;
  color: #303133;
}

.card-subtitle {
  margin-top: 4px;
  font-size: 12px;
  color: #909399;
}

.clone-table {
  width: 100%;
}

.target-text {
  display: inline-block;
  max-width: 220px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  vertical-align: middle;
}

.action-buttons {
  display: flex;
  align-items: center;
  gap: 6px;
  flex-wrap: wrap;
}

.action-buttons .el-button {
  margin-left: 0;
}
</style>