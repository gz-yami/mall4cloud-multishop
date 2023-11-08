<template>
  <div class="app-container">
    <!-- 搜索相关区域 -->
    <div class="filter-container">
      <el-button
        v-permission="['multishop:indexImg:save']"
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
      stripe
      fit
      row-key="imgId"
      highlight-current-row
      style="width: 100%;"
    >
      <!-- 图片 -->
      <el-table-column
        label="图片"
        prop="imgUrl"
        align="center"
        width="300px"
      >
        <template #default="{row}">
          <img
            :src="(row.imgUrl).indexOf('http')===-1 ? resourcesUrl + row.imgUrl : row.imgUrl"
            class="img"
          >
        </template>
      </el-table-column>
      <!-- 顺序 -->
      <el-table-column
        label="顺序"
        prop=""
        align="center"
      >
        <template #default="{row}">
          <span>{{ row.seq }}</span>
        </template>
      </el-table-column>
      <!-- 状态 -->
      <el-table-column
        label="状态"
        prop="status"
        align="center"
      >
        <template #default="{row}">
          <el-tag
            v-if="row.status === 0"
            
            type="danger"
          >
            {{ $t("product.category.offline") }}
          </el-tag>
          <el-tag
            v-else
            
          >
            {{ $t("product.category.normal") }}
          </el-tag>
        </template>
      </el-table-column>
      <!-- 平台 -->
      <el-table-column
        label="平台"
        prop="icon"
        align="center"
      >
        <template #default="{row}">
          <span>{{ ['移动端', 'PC端'][row.imgType] }}</span>
        </template>
      </el-table-column>
      <!-- 操作 -->
      <el-table-column
        :label="$t('table.actions')"
        align="center"
        class-name="small-padding fixed-width"
      >
        <template #default="{row}">
          <el-button
            v-permission="['multishop:indexImg:update']"
            type="text"
            @click="onAddOrUpdate(row.imgId)"
          >
            {{ $t('table.edit') }}
          </el-button>
          <el-button
            v-permission="['multishop:indexImg:delete']"
            type="text"
            @click="onDelete(row.imgId)"
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
import * as api from '@/api/multishop/index-img'


</script>

<style lang="scss" scoped>
.app-container {
  .img {
    width: auto;
    height: auto;
    max-width: 100%;
    max-height: 300px;
  }
}
</style>
