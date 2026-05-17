<template>
  <el-card>
    <template #header>
      <div class="card-header">
        <span>克隆任务</span>
        <el-button type="primary" @click="$emit('add')">
          新增任务
        </el-button>
      </div>
    </template>

    <el-table :data="tasks" border>
      <el-table-column prop="id" label="ID" width="70" />
      <el-table-column prop="name" label="任务名" />
      <el-table-column prop="source_channel" label="源频道" />
      <el-table-column prop="target_channels" label="目标频道" />
      <el-table-column prop="status" label="状态" width="100" />
      <el-table-column prop="last_message_id" label="进度" width="100" />

      <el-table-column label="操作" width="360">
        <template #default="{ row }">
          <el-button size="small" type="success" @click="$emit('start', row.id)">
            开始
          </el-button>

          <el-button size="small" type="warning" @click="$emit('pause', row.id)">
            暂停
          </el-button>

          <el-button size="small" type="primary" @click="$emit('resume', row.id)">
            继续
          </el-button>
            <el-button
                v-if="row.status === 'running' || row.status === 'paused'"
                size="small"
                type="danger"
                plain
                @click="$emit('stop', row)"
                >
                停止
        </el-button>
          <el-button size="small" @click="$emit('edit', row)">
            编辑
          </el-button>

          <el-button size="small" type="danger" @click="$emit('delete', row.id)">
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
  </el-card>
</template>

<script setup>
defineProps({
  tasks: {
    type: Array,
    required: true
  }
})

defineEmits([
  "add",
  "edit",
  "delete",
  "start",
  "pause",
  "resume"
])
</script>

<style scoped>
.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
</style>