<template>
  <el-dialog
    :model-value="visible"
    @update:model-value="$emit('update:visible', $event)"
    :title="isEdit ? '编辑克隆任务' : '新增克隆任务'"
    width="720px"
  >
    <el-form label-width="130px">
      <el-form-item label="任务名称">
        <el-input v-model="localForm.name" placeholder="例如 杭州频道克隆" />
      </el-form-item>

      <el-form-item label="源频道">
        <el-input v-model="localForm.source_channel" placeholder="例如 @HZKTV_XZ" />
      </el-form-item>

      <el-form-item label="目标频道">
        <el-input
          v-model="localForm.target_channels"
          type="textarea"
          :rows="3"
          placeholder='例如 ["@target1","@target2"]'
        />
      </el-form-item>

      <el-form-item label="账号ID">
        <el-input-number v-model="localForm.account_id" :min="1" />
      </el-form-item>

      <el-form-item label="每次克隆数量">
        <el-input-number v-model="localForm.clone_limit" :min="1" />
      </el-form-item>

      <el-form-item label="单条间隔秒">
        <el-input-number v-model="localForm.single_delay" :min="1" />
      </el-form-item>

      <el-form-item label="相册间隔秒">
        <el-input-number v-model="localForm.album_delay" :min="1" />
      </el-form-item>

      <el-form-item label="目标间隔秒">
        <el-input-number v-model="localForm.target_delay" :min="1" />
      </el-form-item>

      <el-form-item label="过滤关键词">
        <el-input
          v-model="localForm.blocked_keywords"
          type="textarea"
          :rows="3"
          placeholder='例如 ["广告","加微信"]'
        />
      </el-form-item>

      <el-form-item label="替换词">
        <el-input
          v-model="localForm.replace_words"
          type="textarea"
          :rows="3"
          placeholder='例如 {"旧词":"新词"}'
        />
      </el-form-item>

      <el-form-item label="尾巴 Footer">
        <el-input v-model="localForm.footer" type="textarea" :rows="3" />
      </el-form-item>

      <el-form-item label="启用">
        <el-switch v-model="localForm.enabled" />
      </el-form-item>
    </el-form>

    <template #footer>
      <el-button @click="$emit('update:visible', false)">取消</el-button>
      <el-button type="primary" @click="submit">保存</el-button>
    </template>
  </el-dialog>
</template>

<script setup>
import { reactive, watch } from "vue"

const props = defineProps({
  visible: Boolean,
  form: Object,
  isEdit: Boolean
})

const emit = defineEmits(["update:visible", "submit"])

const localForm = reactive({
  id: null,
  name: "",
  source_channel: "",
  target_channels: "[]",
  account_id: 1,
  clone_limit: 100,
  single_delay: 3,
  album_delay: 8,
  target_delay: 2,
  blocked_keywords: "[]",
  replace_words: "{}",
  footer: "",
  enabled: true,
  status: "idle",
  last_message_id: 0
})

watch(
  () => props.form,
  (val) => {
    if (!val) return
    Object.assign(localForm, val)
  },
  { immediate: true, deep: true }
)

function submit() {
  emit("submit", { ...localForm })
}
</script>