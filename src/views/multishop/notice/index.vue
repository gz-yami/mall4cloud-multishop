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
        v-permission="['multishop:notice:save']"
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
      <!-- 发布时间 -->
      <el-table-column
        :label="$t('multishop.notice.publishTime')"
        prop="publishTime"
        align="center"
      >
        <template #default="{row}">
          <span>{{ row.publishTime }}</span>
        </template>
      </el-table-column>
      <!-- 公告标题 -->
      <el-table-column
        :label="$t('multishop.notice.title')"
        prop="title"
        align="center"
      >
        <template #default="{row}">
          <span>{{ row.title }}</span>
        </template>
      </el-table-column>
      <!-- 状态(1:公布 0:撤回) -->
      <el-table-column
        :label="$t('multishop.notice.status')"
        prop="status"
        align="center"
      >
        <template #default="{row}">
          <el-tag :type="row.status === 0 ? 'danger' : ''">
            {{ ['撤销','公布'][row.status] }}
          </el-tag>
        </template>
      </el-table-column>
      <!-- 是否置顶 -->
      <el-table-column
        :label="$t('multishop.notice.isTop')"
        prop="isTop"
        align="center"
      >
        <template #default="{row}">
          <el-tag :type="row.isTop === 0 ? 'warning' : ''">
            {{ ['否','是'][row.isTop] }}
          </el-tag>
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
            v-permission="['multishop:notice:update']"
            type="text"
            @click="onAddOrUpdate(row.id)"
          >
            {{ $t('table.edit') }}
          </el-button>
          <el-button
            v-permission="['multishop:notice:delete']"
            type="text"
            @click="onDelete(row.id)"
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
import * as api from '@/api/multishop/notice'


  name: '',

</script>
