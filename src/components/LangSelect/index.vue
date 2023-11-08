<template>
  <el-dropdown
    trigger="click"
    class="international"
    @command="handleSetLanguage"
  >
    <div>
      <svg-icon
        class-name="international-icon"
        icon-class="language"
      />
    </div>
    <template #dropdown>
      <el-dropdown-menu>
        <el-dropdown-item
          :disabled="language==='zh'"
          command="zh"
        >
          中文
        </el-dropdown-item>
        <el-dropdown-item
          :disabled="language==='en'"
          command="en"
        >
          English
        </el-dropdown-item>
      </el-dropdown-menu>
    </template>
  </el-dropdown>
</template>

<script setup>

  computed: {
    language () {
      return $store.getters.language
    }
  },

const handleSetLanguage  = (lang) => {
  $i18n.locale = lang
  $store.dispatch('app/setLanguage', lang)
  ElMessage({
    message: 'Switch Language Success',
    type: 'success'
  })
}

</script>
