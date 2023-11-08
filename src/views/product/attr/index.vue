<template>
  <div class="app-container">
    <!-- 搜索相关区域 -->
    <div class="filter-container">
      <el-form
        :inline="true"
        :model="pageQuery"
        class="demo-form-inline"
      >
        <el-form-item label="属性名称">
          <el-input
            v-model="pageQuery.name"
            size="mini"
            placeholder="属性名称"
          />
        </el-form-item>
        <!-- <el-form-item label="属性类型">
          <el-select v-model="pageQuery.attrType" size="mini" placeholder="请选择属性类型">
            <el-option label="销售属性" value="0" />
            <el-option label="基本属性" value="1" />
          </el-select>
        </el-form-item> -->
        <el-form-item>
          <el-button
            type="primary"
            size="mini"
            icon="el-icon-search"
            @click="getPage()"
          >
            查询
          </el-button>
          <el-button
            size="mini"
            icon="el-icon-delete"
            @click="clearSearchInfo()"
          >
            清空
          </el-button>
        </el-form-item>
      </el-form>
      <!-- <el-button size="mini" icon="el-icon-search" class="filter-item" @click="getPage()">{{ $t('table.search') }}</el-button> -->
      <el-button
        v-permission="['product:attr:save']"
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
      <!-- 创建时间 -->
      <!-- <el-table-column :label="$t('table.createTime')" prop="createTime" align="center">
        <template slot-scope="{row}">
          <span>{{ row.createTime }}</span>
        </template>
      </el-table-column> -->
      <!-- 更新时间 -->
      <!-- <el-table-column :label="$t('table.updateTime')" prop="updateTime" align="center">
        <template slot-scope="{row}">
          <span>{{ row.updateTime }}</span>
        </template>
      </el-table-column> -->
      <!-- 属性名称 -->
      <el-table-column
        :label="$t('product.attr.name')"
        prop="name"
        align="center"
      >
        <template #default="{row}">
          <span>{{ row.name }}</span>
        </template>
      </el-table-column>
      <!-- 属性描述 -->
      <!-- <el-table-column :label="$t('product.attr.desc')" prop="desc" align="center">
        <template slot-scope="{row}">
          <span>{{ row.desc }}</span>
        </template>
      </el-table-column> -->
      <!-- 0:不需要，1:需要 -->
      <!-- <el-table-column :label="$t('product.attr.searchType')" prop="searchType" align="center">
        <template slot-scope="{row}">
          <span>{{ ['否', '是'][row.searchType] }}</span>
        </template>
      </el-table-column> -->
      <!-- 0:销售属性，1:基本属性 -->
      <!-- <el-table-column :label="$t('product.attr.attrType')" prop="attrType" align="center">
        <template slot-scope="{row}">
          <span>{{ ['销售属性', '基本属性'][row.attrType] }}</span>
        </template>
      </el-table-column> -->
      <!-- 属性值 -->
      <el-table-column
        label="属性值"
        prop="attrType"
        align="center"
      >
        <template #default="{row}">
          <el-tag
            v-for="(item,index) in row.attrValues"
            :key="index"
          >
            {{ item.value }}
          </el-tag>
        </template>
      </el-table-column>
      <!-- 操作 -->
      <el-table-column
        :label="$t('table.actions')"
        align="center"
        width="230"
        class-name="small-padding fixed-width"
      >
        <template #default="{row}">
          <el-button
            v-permission="['product:attr:update']"
            type="text"
            @click="onAddOrUpdate(row.attrId)"
          >
            {{ $t('table.edit') }}
          </el-button>
          <el-button
            v-permission="['product:attr:delete']"
            type="text"
            @click="onDelete(row.attrId)"
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
import * as api from '@/api/product/attr'


  name: '',

</script>
