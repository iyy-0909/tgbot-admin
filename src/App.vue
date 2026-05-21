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

    <div v-if="activeMenu === 'bots'" class="bot-page">
      <BotTable
        :bots="bots"
        @add="openAddBotDialog"
        @edit="openEditBotDialog"
        @delete="deleteBotHandler"
        @toggle="saveBotStatus"
        @test="testBotHandler"
      />

      <BotBindingTable
        :bindings="botBindings"
        :bots="bots"
        @add="openAddBotBindingDialog"
        @edit="openEditBotBindingDialog"
        @delete="deleteBotBindingHandler"
        @toggle="saveBotBindingStatus"
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

    <BotDialog
      :visible="botDialogVisible"
      :form="currentBot"
      :is-edit="isBotEdit"
      @update:visible="botDialogVisible = $event"
      @submit="submitBot"
    />

    <BotBindingDialog
      :visible="botBindingDialogVisible"
      :form="currentBotBinding"
      :bots="bots"
      :is-edit="isBotBindingEdit"
      @update:visible="botBindingDialogVisible = $event"
      @submit="submitBotBinding"
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

import BotTable from "./components/BotTable.vue"
import BotDialog from "./components/BotDialog.vue"
import BotBindingTable from "./components/BotBindingTable.vue"
import BotBindingDialog from "./components/BotBindingDialog.vue"

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

import {
  getBots,
  createBot,
  updateBot,
  deleteBot,
  getBotBindings,
  createBotBinding,
  updateBotBinding,
  deleteBotBinding,
  testBot,
  sendBotTest
} from "./api/bots"



const status = ref({})
const rules = ref([])
const logs = ref([])
const accounts = ref([])
const bots = ref([])
const botBindings = ref([])
const cloneTasks = ref([])

const activeMenu = ref("rules")

const dialogVisible = ref(false)
const isEdit = ref(false)

const accountDialogVisible = ref(false)
const isAccountEdit = ref(false)

const botDialogVisible = ref(false)
const isBotEdit = ref(false)

const botBindingDialogVisible = ref(false)
const isBotBindingEdit = ref(false)

const cloneTaskDialogVisible = ref(false)
const isCloneTaskEdit = ref(false)

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


const currentAccount = reactive({
  id: null,
  name: "",
  session_path: "",
  proxy: "",
  enabled: true,
  remark: "",
})


const currentBot = reactive({
  id: null,
  name: "",
  token: "",
  enabled: true,
  remark: "",
  last_error: "",
})


const currentBotBinding = reactive({
  id: null,
  target_channel: "",
  bot_id: null,
  enabled: true,
  remark: "",
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


async function loadAccounts() {
  const res = await getAccounts()
  accounts.value = res.data
}


async function loadBots() {
  const res = await getBots()
  bots.value = res.data
}


async function loadBotBindings() {
  const res = await getBotBindings()
  botBindings.value = res.data
}


async function loadBotPage() {
  await loadBots()
  await loadBotBindings()
}


async function loadCloneTasks() {
  const res = await getCloneTasks()
  cloneTasks.value = res.data
}


async function handleMenuChange(menu) {
  activeMenu.value = menu

  if (menu === "rules") {
    await loadStatus()
    await loadRules()
  }

  if (menu === "logs") {
    await loadLogs()
  }

  if (menu === "accounts") {
    await loadAccounts()
  }

  if (menu === "bots") {
    await loadBotPage()
  }

  if (menu === "clone") {
    await loadCloneTasks()
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
  await cloneRule(rule.id, 50, 5)
  ElMessage.success("克隆任务已开始，请查看日志")
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
// Bot 管理
// =========================

function resetCurrentBot() {
  currentBot.id = null
  currentBot.name = ""
  currentBot.token = ""
  currentBot.enabled = true
  currentBot.remark = ""
  currentBot.last_error = ""
}


function openAddBotDialog() {
  resetCurrentBot()
  isBotEdit.value = false
  botDialogVisible.value = true
}


function openEditBotDialog(row) {
  Object.assign(currentBot, {
    id: row.id,
    name: row.name || "",
    token: row.token || "",
    enabled: row.enabled ?? true,
    remark: row.remark || "",
    last_error: row.last_error || "",
  })

  isBotEdit.value = true
  botDialogVisible.value = true
}


async function submitBot(formData) {
  Object.assign(currentBot, formData)

  if (!currentBot.name || !currentBot.token) {
    ElMessage.error("Bot 名称和 Token 不能为空")
    return
  }

  const payload = {
    name: currentBot.name,
    token: currentBot.token,
    enabled: currentBot.enabled,
    remark: currentBot.remark || "",
  }

  if (isBotEdit.value) {
    await updateBot(currentBot.id, payload)
    ElMessage.success("Bot 保存成功")
  } else {
    await createBot(payload)
    ElMessage.success("Bot 添加成功")
  }

  botDialogVisible.value = false

  await loadBots()
}


async function saveBotStatus(row, value) {
  await updateBot(row.id, {
    enabled: value,
  })

  ElMessage.success(value ? "Bot 已启用" : "Bot 已停用")

  await loadBots()
}

async function testBotHandler(row) {
  try {
    const res = await testBot(row.id)

    if (res.data.ok) {
      ElMessage.success(`Bot 正常：@${res.data.bot.username}`)
    } else {
      ElMessage.error(res.data.message || "Bot 测试失败")
    }
  } catch (e) {
    console.error(e)
    ElMessage.error("Bot 测试失败")
  }
}

async function deleteBotHandler(id) {
  await ElMessageBox.confirm(
    "确定删除这个 Bot？删除后会同时删除相关目标频道绑定。",
    "确认删除",
    {
      type: "warning",
    },
  )

  await deleteBot(id)

  ElMessage.success("Bot 已删除")

  await loadBotPage()
}


// =========================
// 目标频道绑定 Bot
// =========================

function resetCurrentBotBinding() {
  currentBotBinding.id = null
  currentBotBinding.target_channel = ""
  currentBotBinding.bot_id = null
  currentBotBinding.enabled = true
  currentBotBinding.remark = ""
}


function openAddBotBindingDialog() {
  if (!bots.value.length) {
    ElMessage.warning("请先添加 Bot")
    return
  }

  resetCurrentBotBinding()
  isBotBindingEdit.value = false
  botBindingDialogVisible.value = true
}


function openEditBotBindingDialog(row) {
  Object.assign(currentBotBinding, {
    id: row.id,
    target_channel: row.target_channel || "",
    bot_id: row.bot_id || null,
    enabled: row.enabled ?? true,
    remark: row.remark || "",
  })

  isBotBindingEdit.value = true
  botBindingDialogVisible.value = true
}


async function submitBotBinding(formData) {
  Object.assign(currentBotBinding, formData)

  if (!currentBotBinding.target_channel) {
    ElMessage.error("目标频道不能为空")
    return
  }

  if (!currentBotBinding.bot_id) {
    ElMessage.error("请选择 Bot")
    return
  }

  const payload = {
    target_channel: currentBotBinding.target_channel,
    bot_id: currentBotBinding.bot_id,
    enabled: currentBotBinding.enabled,
    remark: currentBotBinding.remark || "",
  }

  if (isBotBindingEdit.value) {
    await updateBotBinding(currentBotBinding.id, payload)
    ElMessage.success("绑定保存成功")
  } else {
    await createBotBinding(payload)
    ElMessage.success("绑定添加成功")
  }

  botBindingDialogVisible.value = false

  await loadBotBindings()
}


async function saveBotBindingStatus(row, value) {
  await updateBotBinding(row.id, {
    enabled: value,
  })

  ElMessage.success(value ? "绑定已启用" : "绑定已停用")

  await loadBotBindings()
}


async function deleteBotBindingHandler(id) {
  await ElMessageBox.confirm(
    "确定删除这个绑定？",
    "确认删除",
    {
      type: "warning",
    },
  )

  await deleteBotBinding(id)

  ElMessage.success("绑定已删除")

  await loadBotBindings()
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

.bot-page {
  display: flex;
  flex-direction: column;
  gap: 20px;
}
</style>