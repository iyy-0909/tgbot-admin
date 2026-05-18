<template>
  <MainLayout
    :status="status.status"
    :active-menu="activeMenu"
    @change-menu="handleMenuChange"
  >
    <div v-if="activeMenu === 'rules'">
      <StatusCards :status="status" />

      <RuleTable
        :rules="rules"
        @add="openAddDialog"
        @edit="openEditDialog"
        @delete="deleteRule"
        @toggle="saveRule"
        @clone="startClone"
      />
    </div>

    <div v-if="activeMenu === 'logs'">
      <LogPanel
        :logs="logs"
        @refresh="loadLogs"
      />
    </div>

    <div v-if="activeMenu === 'accounts'">
      <AccountTable
        :accounts="accounts"
        @add="openAddAccountDialog"
        @edit="openEditAccountDialog"
        @delete="deleteAccount"
        @toggle="saveAccount"
      />
    </div>

    <div v-if="activeMenu === 'clone'">
      <CloneTaskTable
        :tasks="cloneTasks"
        @add="openAddCloneTaskDialog"
        @edit="openEditCloneTaskDialog"
        @delete="removeCloneTaskHandler"
        @start="startCloneTaskHandler"
        @pause="pauseCloneTaskHandler"
        @resume="resumeCloneTaskHandler"
        @stop="handleStopCloneTask"
        @toggle-listener="handleToggleCloneListener"
      />
    </div>

    <RuleDialog
      :visible="dialogVisible"
      :form="currentRule"
      :is-edit="isEdit"
      @update:visible="dialogVisible = $event"
      @submit="submitRule"
    />

    <AccountDialog
      :visible="accountDialogVisible"
      :form="currentAccount"
      :is-edit="isAccountEdit"
      @update:visible="accountDialogVisible = $event"
      @submit="submitAccount"
    />
  </MainLayout>

  <CloneTaskDialog
    :visible="cloneTaskDialogVisible"
    :form="currentCloneTask"
    :is-edit="isCloneTaskEdit"
    @update:visible="cloneTaskDialogVisible = $event"
    @submit="submitCloneTask"
  />
</template>

<script setup>
import { ElMessage, ElMessageBox } from "element-plus"
import { ref, reactive, onMounted, onUnmounted } from "vue"

import MainLayout from "./layouts/MainLayout.vue"
import StatusCards from "./components/StatusCards.vue"
import RuleTable from "./components/RuleTable.vue"
import RuleDialog from "./components/RuleDialog.vue"
import LogPanel from "./components/LogPanel.vue"

import AccountTable from "./components/AccountTable.vue"
import AccountDialog from "./components/AccountDialog.vue"

import CloneTaskTable from "./components/CloneTaskTable.vue"
import CloneTaskDialog from "./components/CloneTaskDialog.vue"

import {
  getCloneTasks,
  createCloneTask,
  updateCloneTask,
  deleteCloneTask,
  startCloneTask,
  pauseCloneTask,
  resumeCloneTask,
} from "./api/cloneTasks"

import {
  getStatus,
  getRules,
  createRule,
  updateRule,
  removeRule,
  getLogs,
  cloneRule,
  stopCloneTask,
} from "./api/rules"

import {
  getAccounts,
  createAccount,
  updateAccount,
  removeAccount,
} from "./api/accounts"


const status = ref({})
const rules = ref([])
const logs = ref([])
const accounts = ref([])
const cloneTasks = ref([])

const activeMenu = ref("rules")

const dialogVisible = ref(false)
const isEdit = ref(false)

const cloneTaskDialogVisible = ref(false)
const isCloneTaskEdit = ref(false)

const accountDialogVisible = ref(false)
const isAccountEdit = ref(false)

let cloneRefreshTimer = null


const currentRule = reactive({
  id: null,
  source: "",
  target: "",
  enabled: true,
  blocked_keywords: "[]",
  replace_words: "{}",
  footer: "",
  remove_contact_lines: true,
  clone_task_id: null,
})


const currentCloneTask = reactive({
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
  remove_contact_lines: true,
  enable_listener: false,
  enabled: true,
  status: "idle",
  last_message_id: 0,
})


const currentAccount = reactive({
  id: null,
  name: "",
  session_path: "",
  proxy: "",
  enabled: true,
  remark: "",
})


async function loadStatus() {
  const res = await getStatus()
  status.value = res.data
}


async function loadRules() {
  const res = await getRules()
  rules.value = res.data
}


async function loadLogs() {
  const res = await getLogs()
  logs.value = res.data.logs
}


async function loadCloneTasks() {
  const res = await getCloneTasks()
  cloneTasks.value = res.data
}


async function loadAccounts() {
  const res = await getAccounts()
  accounts.value = res.data
}


async function handleMenuChange(menu) {
  activeMenu.value = menu

  if (menu === "logs") {
    await loadLogs()
  }

  if (menu === "accounts") {
    await loadAccounts()
  }

  if (menu === "clone") {
    await loadCloneTasks()
  }

  if (menu === "rules") {
    await loadStatus()
    await loadRules()
  }
}


// =========================
// 频道规则
// =========================

function resetCurrentRule() {
  currentRule.id = null
  currentRule.source = ""
  currentRule.target = ""
  currentRule.enabled = true
  currentRule.blocked_keywords = "[]"
  currentRule.replace_words = "{}"
  currentRule.footer = ""
  currentRule.remove_contact_lines = true
  currentRule.clone_task_id = null
}


function openAddDialog() {
  resetCurrentRule()
  isEdit.value = false
  dialogVisible.value = true
}


function openEditDialog(row) {
  currentRule.id = row.id
  currentRule.source = row.source || ""
  currentRule.target = row.target || ""
  currentRule.enabled = row.enabled ?? true
  currentRule.blocked_keywords = row.blocked_keywords || "[]"
  currentRule.replace_words = row.replace_words || "{}"
  currentRule.footer = row.footer || ""
  currentRule.remove_contact_lines = row.remove_contact_lines ?? true
  currentRule.clone_task_id = row.clone_task_id || null

  isEdit.value = true
  dialogVisible.value = true
}


function validateRuleJson() {
  try {
    JSON.parse(currentRule.blocked_keywords || "[]")
    JSON.parse(currentRule.replace_words || "{}")
    return true
  } catch {
    ElMessage.error("过滤关键词或替换词不是合法 JSON")
    return false
  }
}


async function submitRule(formData) {
  Object.assign(currentRule, formData)

  if (!currentRule.source || !currentRule.target) {
    ElMessage.error("源频道和目标频道不能为空")
    return
  }

  if (!validateRuleJson()) return

  const payload = {
    source: currentRule.source,
    target: currentRule.target,
    enabled: currentRule.enabled,
    blocked_keywords: currentRule.blocked_keywords || "[]",
    replace_words: currentRule.replace_words || "{}",
    footer: currentRule.footer || "",
    remove_contact_lines: currentRule.remove_contact_lines,
  }

  if (isEdit.value) {
    await updateRule(currentRule.id, payload)
    ElMessage.success("规则保存成功")
  } else {
    await createRule(payload)
    ElMessage.success("规则添加成功")
  }

  dialogVisible.value = false

  await loadStatus()
  await loadRules()
}


async function saveRule(row) {
  await updateRule(row.id, {
    source: row.source,
    target: row.target,
    enabled: row.enabled,
    blocked_keywords: row.blocked_keywords || "[]",
    replace_words: row.replace_words || "{}",
    footer: row.footer || "",
    remove_contact_lines: row.remove_contact_lines ?? true,
  })

  ElMessage.success("规则状态已更新")

  await loadStatus()
  await loadRules()
}


async function deleteRule(id) {
  await ElMessageBox.confirm(
    "确定删除这条规则？",
    "确认删除",
    {
      type: "warning",
    },
  )

  await removeRule(id)

  ElMessage.success("删除成功")

  await loadStatus()
  await loadRules()
}


async function startClone(rule) {
  await cloneRule(
    rule.id,
    50,
    5,
  )

  ElMessage.success("克隆任务已开始，请查看日志")
}


// =========================
// 克隆任务
// =========================

function resetCurrentCloneTask() {
  Object.assign(currentCloneTask, {
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
    remove_contact_lines: true,
    enable_listener: false,
    enabled: true,
    status: "idle",
    last_message_id: 0,
  })
}


function openAddCloneTaskDialog() {
  resetCurrentCloneTask()
  isCloneTaskEdit.value = false
  cloneTaskDialogVisible.value = true
}


function openEditCloneTaskDialog(row) {
  Object.assign(currentCloneTask, {
    id: row.id,
    name: row.name || "",
    source_channel: row.source_channel || "",
    target_channels: row.target_channels || "[]",
    account_id: row.account_id || 1,
    clone_limit: row.clone_limit || 100,
    single_delay: row.single_delay || 3,
    album_delay: row.album_delay || 8,
    target_delay: row.target_delay || 2,
    blocked_keywords: row.blocked_keywords || "[]",
    replace_words: row.replace_words || "{}",
    footer: row.footer || "",
    remove_contact_lines: row.remove_contact_lines ?? true,
    enable_listener: row.enable_listener ?? false,
    enabled: row.enabled ?? true,
    status: row.status || "idle",
    last_message_id: row.last_message_id || 0,
  })

  isCloneTaskEdit.value = true
  cloneTaskDialogVisible.value = true
}


function validateCloneTaskJson() {
  try {
    JSON.parse(currentCloneTask.target_channels || "[]")
    JSON.parse(currentCloneTask.blocked_keywords || "[]")
    JSON.parse(currentCloneTask.replace_words || "{}")
    return true
  } catch {
    ElMessage.error("目标频道、过滤关键词或替换词 JSON 格式错误")
    return false
  }
}


async function submitCloneTask(formData) {
  Object.assign(currentCloneTask, formData)

  if (!currentCloneTask.name || !currentCloneTask.source_channel) {
    ElMessage.error("任务名称和源频道不能为空")
    return
  }

  if (!validateCloneTaskJson()) return

  const payload = {
    name: currentCloneTask.name,
    source_channel: currentCloneTask.source_channel,
    target_channels: currentCloneTask.target_channels || "[]",
    account_id: currentCloneTask.account_id,
    clone_limit: currentCloneTask.clone_limit,
    single_delay: currentCloneTask.single_delay,
    album_delay: currentCloneTask.album_delay,
    target_delay: currentCloneTask.target_delay,
    blocked_keywords: currentCloneTask.blocked_keywords || "[]",
    replace_words: currentCloneTask.replace_words || "{}",
    footer: currentCloneTask.footer || "",
    remove_contact_lines: currentCloneTask.remove_contact_lines,
    enable_listener: currentCloneTask.enable_listener,
    enabled: currentCloneTask.enabled,
  }

  if (isCloneTaskEdit.value) {
    await updateCloneTask(currentCloneTask.id, payload)
    ElMessage.success("克隆任务保存成功")
  } else {
    await createCloneTask(payload)
    ElMessage.success("克隆任务添加成功")
  }

  cloneTaskDialogVisible.value = false

  await loadCloneTasks()
  await loadRules()
  await loadStatus()
}


async function removeCloneTaskHandler(id) {
  await ElMessageBox.confirm(
    "确定删除这个克隆任务？",
    "确认删除",
    {
      type: "warning",
    },
  )

  await deleteCloneTask(id)

  ElMessage.success("克隆任务已删除")

  await loadCloneTasks()
  await loadRules()
  await loadStatus()
}


async function startCloneTaskHandler(id) {
  await startCloneTask(id)
  ElMessage.success("克隆已开始")
  await loadCloneTasks()
}


async function pauseCloneTaskHandler(id) {
  await pauseCloneTask(id)
  ElMessage.success("克隆已暂停")
  await loadCloneTasks()
}


async function resumeCloneTaskHandler(id) {
  await resumeCloneTask(id)
  ElMessage.success("克隆已继续")
  await loadCloneTasks()
}


async function handleStopCloneTask(id) {
  await stopCloneTask(id)
  ElMessage.success("已停止克隆任务")
  await loadCloneTasks()
}


const handleToggleCloneListener = async (row, value) => {
  try {
    await updateCloneTask(row.id, {
      enable_listener: value,
    })

    ElMessage.success(value ? "已开启实时监听" : "已关闭实时监听")

    await loadCloneTasks()
    await loadRules()
    await loadStatus()
  } catch (e) {
    console.error(e)
    ElMessage.error("切换实时监听失败")
    await loadCloneTasks()
  }
}


// =========================
// 账号管理
// =========================

function resetCurrentAccount() {
  currentAccount.id = null
  currentAccount.name = ""
  currentAccount.session_path = ""
  currentAccount.proxy = ""
  currentAccount.enabled = true
  currentAccount.remark = ""
}


function openAddAccountDialog() {
  resetCurrentAccount()
  isAccountEdit.value = false
  accountDialogVisible.value = true
}


function openEditAccountDialog(row) {
  Object.assign(currentAccount, row)
  isAccountEdit.value = true
  accountDialogVisible.value = true
}


async function submitAccount(formData) {
  Object.assign(currentAccount, formData)

  if (!currentAccount.name || !currentAccount.session_path) {
    ElMessage.error("账号名称和 Session 路径不能为空")
    return
  }

  if (isAccountEdit.value) {
    await updateAccount(currentAccount.id, {
      name: currentAccount.name,
      session_path: currentAccount.session_path,
      proxy: currentAccount.proxy,
      enabled: currentAccount.enabled,
      remark: currentAccount.remark,
    })

    ElMessage.success("账号保存成功")
  } else {
    await createAccount({
      name: currentAccount.name,
      session_path: currentAccount.session_path,
      proxy: currentAccount.proxy,
      remark: currentAccount.remark,
    })

    ElMessage.success("账号添加成功")
  }

  accountDialogVisible.value = false
  await loadAccounts()
}


async function saveAccount(row) {
  await updateAccount(row.id, {
    name: row.name,
    session_path: row.session_path,
    proxy: row.proxy,
    enabled: row.enabled,
    remark: row.remark,
  })

  ElMessage.success("账号状态已更新")
}


async function deleteAccount(id) {
  await ElMessageBox.confirm(
    "确定删除这个账号？",
    "确认删除",
    {
      type: "warning",
    },
  )

  await removeAccount(id)

  ElMessage.success("账号已删除")

  await loadAccounts()
}


// =========================
// 生命周期
// =========================

onMounted(async () => {
  await loadStatus()
  await loadRules()
  await loadCloneTasks()

  cloneRefreshTimer = setInterval(async () => {
    try {
      await loadCloneTasks()
    } catch (e) {
      console.error("自动刷新克隆任务失败", e)
    }
  }, 5 * 60 * 1000)
})


onUnmounted(() => {
  if (cloneRefreshTimer) {
    clearInterval(cloneRefreshTimer)
    cloneRefreshTimer = null
  }
})
</script>

<style>
body {
  margin: 0;
  background: #f3f4f6;
}
</style>