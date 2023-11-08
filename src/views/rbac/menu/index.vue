<template>
  <div class="mod-menu-home app-container">
    <!-- 搜索相关区域 -->
    <div class="filter-container">
      <el-button
        size="mini"
        icon="el-icon-search"
        class="filter-item"
        @click="getPage()"
      >
        {{ $t('table.search') }}
      </el-button>
      <el-button
        v-permission="['rbac:menu:save']"
        size="mini"
        icon="el-icon-plus"
        type="primary"
        class="filter-item"
        @click="onAddOrUpdate()"
      >
        {{ $t('table.create') }}
      </el-button>
    </div>
    <!-- 列表相关区域 -->
    <el-table
      :data="menulist"
      border
      style="width: 100%;"
      row-key="id"
    >
      <!-- 名称 -->
      <el-table-column
        prop="meta.title"
        header-align="center"
        tree-key="id"
        width="150"
        label="名称"
      />
      <!-- 图标 -->
      <el-table-column
        header-align="center"
        align="center"
        label="图标"
      >
        <template #default="scope">
          <span v-if="scope.row.meta.icon && (scope.row.meta.icon).includes('el-icon')"><i :class="scope.row.meta.icon || ''" /></span>
          <span v-if="scope.row.meta.icon && !(scope.row.meta.icon).includes('el-icon')"><svg-icon :icon-class="scope.row.meta.icon" /></span>
        </template>
      </el-table-column>
      <!-- 类型 -->
      <el-table-column
        prop="type"
        header-align="center"
        align="center"
        label="类型"
      >
        <template #default="scope">
          <el-tag
            v-if="scope.row.component === ''"
            
          >
            {{ '目录' }}
          </el-tag>
          <el-tag
            v-else
            
            type="success"
          >
            {{ '菜单' }}
          </el-tag>
        </template>
      </el-table-column>
      <!-- 排序号 -->
      <el-table-column
        prop="seq"
        header-align="center"
        align="center"
        label="排序号"
      />
      <!-- 菜单Url -->
      <el-table-column
        prop="path"
        header-align="center"
        align="center"
        width="150"
        :show-overflow-tooltip="true"
        label="菜单Url"
      />
      <!-- 组件路径 -->
      <el-table-column
        prop="component"
        header-align="center"
        align="center"
        width="150"
        :show-overflow-tooltip="true"
        label="组件路径"
      />
      <!-- 操作 -->
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作"
      >
        <template #default="scope">
          <el-button
            v-permission="['rbac:menu:update']"
            type="text"
            
            @click="onAddOrUpdate(scope.row.id)"
          >
            {{ '编辑' }}
          </el-button>
          <el-button
            v-permission="['rbac:menu:delete']"
            type="text"
            
            @click="onDelete(scope.row.id)"
          >
            {{ '删除' }}
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update
      v-if="addOrUpdateVisible"
      ref="addOrUpdateRef"
      @refresh-data-list="getPage()"
    />
  </div>
</template>

<script setup>
import permission from '@/directive/permission/index.js'
import * as api from '@/api/rbac/menu'
import { treeDataTranslate } from '@/utils'
import AddOrUpdate from './add-or-update.vue'


  name: '',
  directives: { permission },

var menulist = ref([])
// loading
var pageLoading = ref(true)
// 查询参数
var searchParam = {
}
var addOrUpdateVisible = ref(false)
onMounted(() => {
  getPage()
})

const getPage  = () => {
  pageLoading = true
  api.menuList({ ...searchParam }).then(data => {
    data.forEach(item => {
      if (item.component === 'Layout') {
        item.component = ''
      }
    })
    menulist = treeDataTranslate(data)
    pageLoading = false
  })
}
const onAddOrUpdate  = (menuId) => {
  addOrUpdateVisible = true
  nextTick(() => {
    addOrUpdate.value?.init(menuId)
  })
}
const onDelete  = (menuId) => {
  ElMessageBox.confirm($t('table.sureToDelete'), $t('table.tips'), {
    confirmButtonText: $t('table.confirm'),
    cancelButtonText: $t('table.cancel'),
    type: 'warning'
  }).then(() => deleteById(menuId))
}
const deleteById  = (menuId) => {
  api.deleteById(menuId).then(() => {
    ElMessage({
      message: $t('table.actionSuccess'),
      type: 'success',
      duration: 1500,
      onClose: () => getPage()
    })
  })
}

</script>
