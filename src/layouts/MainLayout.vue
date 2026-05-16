<template>
  <el-container class="layout">

    <el-aside
      width="220px"
      class="aside"
    >

      <div class="logo">
        CloneBot
      </div>

      <el-menu
        :default-active="activeMenu"
        background-color="#111827"
        text-color="#cbd5e1"
        active-text-color="#ffffff"
        @select="$emit('change-menu', $event)"
      >

        <el-menu-item index="rules">
          频道规则
        </el-menu-item>

        <el-menu-item index="logs">
          运行日志
        </el-menu-item>

        <el-menu-item index="settings">
          系统设置
        </el-menu-item>
        <el-menu-item index="accounts">
          账号管理
        </el-menu-item>
      </el-menu>

    </el-aside>


    <el-container>

      <el-header class="header">

        <div>
          Telegram 自动化后台
        </div>

        <el-tag type="success">
          {{ status || "unknown" }}
        </el-tag>

      </el-header>


      <el-main>

        <slot />

      </el-main>

    </el-container>

  </el-container>
</template>

<script setup>

defineProps({

  status: {
    type: String,
    default: "unknown"
  },

  activeMenu: {
    type: String,
    default: "rules"
  }
})

defineEmits([
  "change-menu"
])

</script>

<style scoped>

.layout{
  min-height:100vh;
}

.aside{
  background:#111827;
}

.logo{
  height:60px;
  line-height:60px;

  color:white;

  font-size:22px;

  font-weight:bold;

  text-align:center;
}

.header{
  background:white;

  display:flex;

  align-items:center;

  justify-content:space-between;

  font-size:18px;

  border-bottom:1px solid #e5e7eb;
}

</style>