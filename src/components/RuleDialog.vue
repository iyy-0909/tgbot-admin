<template>
  <el-dialog
    :model-value="visible"
    @update:model-value="$emit('update:visible', $event)"
    :title="isEdit ? '编辑规则' : '新增规则'"
    width="600px"
  >
    <el-form label-width="120px">

      <el-form-item label="源频道">
        <el-input
          v-model="localForm.source"
          placeholder="例如 csaze888"
        />
      </el-form-item>

      <el-form-item label="目标频道">
        <el-input
          v-model="localForm.target"
          placeholder="例如 textclonel"
        />
      </el-form-item>

      <el-form-item label="启用">
        <el-switch v-model="localForm.enabled" />
      </el-form-item>

      <el-form-item label="过滤关键词">
        <el-input
          v-model="localForm.blocked_keywords"
          type="textarea"
          :rows="3"
          placeholder='例如 ["广告","诈骗"]'
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
        <el-input
          v-model="localForm.footer"
          type="textarea"
          :rows="3"
        />
      </el-form-item>

    </el-form>

    <template #footer>

      <el-button
        @click="$emit('update:visible', false)"
      >
        取消
      </el-button>

      <el-button
        type="primary"
        @click="submit"
      >
        保存
      </el-button>

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

const emit = defineEmits([
  "update:visible",
  "submit"
])

const localForm = reactive({
  id: null,
  source: "",
  target: "",
  enabled: true,
  blocked_keywords: "[]",
  replace_words: "{}",
  footer: ""
})


watch(
  () => props.form,

  (newVal) => {

    if (!newVal) return

    Object.assign(
      localForm,
      newVal
    )
  },

  {
    immediate: true,
    deep: true
  }
)


function submit(){

  emit(
    "submit",
    { ...localForm }
  )
}
</script>