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

</template>

<script setup>
import { ElMessage, ElMessageBox } from "element-plus"
import { reactive, ref, onMounted } from "vue"

import AccountTable from "./components/AccountTable.vue"
import AccountDialog from "./components/AccountDialog.vue"

import MainLayout from "./layouts/MainLayout.vue"
import StatusCards from "./components/StatusCards.vue"
import RuleTable from "./components/RuleTable.vue"
import RuleDialog from "./components/RuleDialog.vue"
import LogPanel from "./components/LogPanel.vue"
import {
  getStatus,
  getRules,
  createRule,
  updateRule,
  removeRule,
  getLogs,
  cloneRule
} from "./api/rules"


import {
  getAccounts,
  createAccount,
  updateAccount,
  removeAccount
} from "./api/accounts"

const status = ref({})

const rules = ref([])

const dialogVisible = ref(false)

const isEdit = ref(false)

const logs = ref([])
const activeMenu = ref("rules")

const currentRule = reactive({

  id: null,

  source: "",

  target: "",

  enabled: true,

  blocked_keywords: "[]",

  replace_words: "{}",

  footer: ""
})

const accounts = ref([])
const accountDialogVisible = ref(false)
const isAccountEdit = ref(false)

const currentAccount = reactive({
  id: null,
  name: "",
  session_path: "",
  proxy: "",
  enabled: true,
  remark: ""
})

async function loadStatus(){

  const res = await getStatus()

  status.value = res.data
}


async function loadRules(){

  const res = await getRules()

  rules.value = res.data
}


async function startClone(rule) {
  await cloneRule(
    rule.id,
    50,
    5
  )

  ElMessage.success("克隆任务已开始，请查看日志")
}

function resetCurrentRule(){

  currentRule.id = null

  currentRule.source = ""

  currentRule.target = ""

  currentRule.enabled = true

  currentRule.blocked_keywords = "[]"

  currentRule.replace_words = "{}"

  currentRule.footer = ""
}


function openAddDialog(){

  resetCurrentRule()

  isEdit.value = false

  dialogVisible.value = true
}


function openEditDialog(row){

  currentRule.id = row.id

  currentRule.source = row.source

  currentRule.target = row.target

  currentRule.enabled = row.enabled

  currentRule.blocked_keywords =
    row.blocked_keywords || "[]"

  currentRule.replace_words =
    row.replace_words || "{}"

  currentRule.footer =
    row.footer || ""

  isEdit.value = true

  dialogVisible.value = true
}


function validateJson(){

  try{

    JSON.parse(
      currentRule.blocked_keywords || "[]"
    )

    JSON.parse(
      currentRule.replace_words || "{}"
    )

    return true

  }catch{

    ElMessage.error(
      "过滤关键词或替换词不是合法 JSON"
    )

    return false
  }
}


async function submitRule(formData){

  Object.assign(
    currentRule,
    formData
  )

  if(
    !currentRule.source ||
    !currentRule.target
  ){

    ElMessage.error(
      "源频道和目标频道不能为空"
    )

    return
  }

  if(!validateJson()) return


  const payload = {

    source: currentRule.source,

    target: currentRule.target,

    enabled: currentRule.enabled,

    blocked_keywords:
      currentRule.blocked_keywords,

    replace_words:
      currentRule.replace_words,

    footer:
      currentRule.footer
  }


  if(isEdit.value){

    await updateRule(
      currentRule.id,
      payload
    )

    ElMessage.success(
      "保存成功"
    )

  }else{

    await createRule({
      source: currentRule.source,
      target: currentRule.target
    })

    ElMessage.success(
      "添加成功"
    )
  }

  dialogVisible.value = false

  await loadStatus()

  await loadRules()
}


async function saveRule(row){

  await updateRule(
    row.id,
    {

      source: row.source,

      target: row.target,

      enabled: row.enabled,

      blocked_keywords:
        row.blocked_keywords,

      replace_words:
        row.replace_words,

      footer:
        row.footer
    }
  )

  ElMessage.success(
    "状态已更新"
  )
}


async function deleteRule(id){

  await ElMessageBox.confirm(
    "确定删除这条规则？",
    "确认删除",
    {
      type:"warning"
    }
  )

  await removeRule(id)

  ElMessage.success(
    "删除成功"
  )

  await loadStatus()

  await loadRules()
}

async function loadLogs(){
  const res = await getLogs()
  logs.value = res.data.logs
}


async function handleMenuChange(menu){
  activeMenu.value = menu

  if(menu === "logs"){
    await loadLogs()
  }
  if (menu === "accounts") {
  await loadAccounts()
}
}


async function loadAccounts() {
  const res = await getAccounts()
  accounts.value = res.data
}

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
      remark: currentAccount.remark
    })
    ElMessage.success("账号保存成功")
  } else {
    await createAccount({
      name: currentAccount.name,
      session_path: currentAccount.session_path,
      proxy: currentAccount.proxy,
      remark: currentAccount.remark
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
    remark: row.remark
  })

  ElMessage.success("账号状态已更新")
}

async function deleteAccount(id) {
  await ElMessageBox.confirm("确定删除这个账号？", "确认删除", {
    type: "warning"
  })

  await removeAccount(id)
  ElMessage.success("账号已删除")
  await loadAccounts()
}

onMounted(async ()=>{

  await loadStatus()

  await loadRules()

})
</script>

<style>
body{
  margin:0;
  background:#f3f4f6;
}
</style>