<template>
  <div class="app-container">
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
        v-permission="['rbac:menuPermission:save']"
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
      v-loading="pageLoading"
      :data="pageVO.list"
      border
      fit
      highlight-current-row
      style="width: 100%;"
    >
      <!-- 资源关联菜单 -->
      <el-table-column
        :label="$t('rbac.menuPermission.menuTitle')"
        prop="menuTitle"
        align="center"
      >
        <template #default="{row}">
          <span>{{ row.menuTitle }}</span>
        </template>
      </el-table-column>
      <!-- 资源名称 -->
      <el-table-column
        :label="$t('rbac.menuPermission.name')"
        prop="name"
        align="center"
      >
        <template #default="{row}">
          <span>{{ row.name }}</span>
        </template>
      </el-table-column>
      <!-- 权限对应的编码 -->
      <el-table-column
        :label="$t('rbac.menuPermission.permission')"
        prop="permission"
        align="center"
      >
        <template #default="{row}">
          <span>{{ row.permission }}</span>
        </template>
      </el-table-column>
      <!-- 资源对应服务器路径 -->
      <el-table-column
        :label="$t('rbac.menuPermission.uri')"
        prop="uri"
        align="center"
      >
        <template #default="{row}">
          <span>{{ row.uri }}</span>
        </template>
      </el-table-column>
      <!-- 请求方法 -->
      <el-table-column
        :label="$t('rbac.menuPermission.method')"
        prop="method"
        align="center"
      >
        <template #default="{row}">
          <span>{{ ['', 'GET', 'POST', 'PUT', 'DELETE'][row.method] }}</span>
        </template>
      </el-table-column>
      <el-table-column
        :label="$t('table.actions')"
        align="center"
        width="230"
        class-name="small-padding fixed-width"
      >
        <template #default="{row}">
          <el-button
            v-permission="['rbac:menuPermission:update']"
            type="text"
            @click="onAddOrUpdate(row.menuPermissionId)"
          >
            {{ $t('table.edit') }}
          </el-button>
          <el-button
            v-permission="['rbac:menuPermission:delete']"
            type="text"
            @click="onDelete(row.menuPermissionId)"
          >
            {{ $t('table.delete') }}
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页条 -->
    <pagination
      v-show="pageVO.total>0"
      v-model:page="pageQuery.pageNum"
      v-model:limit="pageQuery.pageSize"
      :total="pageVO.total"
      @pagination="getPage()"
    />
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
import Pagination from '@/components/Pagination'
import AddOrUpdate from './add-or-update.vue'
import * as api from '@/api/rbac/menu-permission'


  name: '',

</script>
