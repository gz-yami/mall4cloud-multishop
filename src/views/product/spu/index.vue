<template>
  <div class="app-container">
    <!-- 搜索相关区域 -->
    <div class="filter-container">
      <!-- <el-button size="mini" icon="el-icon-search" class="filter-item" @click="getPage()">{{ $t('table.search') }}</el-button> -->
      <el-button
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
      <el-table-column
        :label="$t('table.createTime')"
        prop="createTime"
        align="center"
      >
        <template #default="{row}">
          <span>{{ row.createTime }}</span>
        </template>
      </el-table-column>
      <!-- 更新时间 -->
      <el-table-column
        :label="$t('table.updateTime')"
        prop="updateTime"
        align="center"
      >
        <template #default="{row}">
          <span>{{ row.updateTime }}</span>
        </template>
      </el-table-column>
      <!-- 品牌ID -->
      <el-table-column
        :label="$t('product.spu.brandId')"
        prop="brandId"
        align="center"
      >
        <template #default="{row}">
          <span>{{ row.brandId }}</span>
        </template>
      </el-table-column>
      <!-- 分类ID -->
      <el-table-column
        :label="$t('product.spu.categoryId')"
        prop="categoryId"
        align="center"
      >
        <template #default="{row}">
          <span>{{ row.categoryId }}</span>
        </template>
      </el-table-column>
      <!-- spu名称 -->
      <el-table-column
        :label="$t('product.spu.name')"
        prop="name"
        align="center"
      >
        <template #default="{row}">
          <span>{{ row.name }}</span>
        </template>
      </el-table-column>
      <!-- 卖点 -->
      <el-table-column
        :label="$t('product.spu.sellingPoint')"
        prop="sellingPoint"
        align="center"
      >
        <template #default="{row}">
          <span>{{ row.sellingPoint }}</span>
        </template>
      </el-table-column>
      <!-- 商品介绍主图 多个图片逗号分隔 -->
      <el-table-column
        :label="$t('product.spu.imgUrls')"
        prop="imgUrls"
        align="center"
      >
        <template #default="{row}">
          <img :src="(row.imgUrls).indexOf('http')===-1 ? resourcesUrl + row.imgUrls : row.imgUrls">
        </template>
      </el-table-column>
      <!-- 售价，整数方式保存 -->
      <el-table-column
        :label="$t('product.spu.priceFee')"
        prop="priceFee"
        align="center"
      >
        <template #default="{row}">
          <span>{{ row.priceFee }}</span>
        </template>
      </el-table-column>
      <!-- 市场价，整数方式保存 -->
      <el-table-column
        :label="$t('product.spu.marketPriceFee')"
        prop="marketPriceFee"
        align="center"
      >
        <template #default="{row}">
          <span>{{ row.marketPriceFee }}</span>
        </template>
      </el-table-column>
      <!-- 状态 1:enable, 0:disable, -1:deleted -->
      <el-table-column
        :label="$t('product.spu.status')"
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
      <el-table-column
        :label="$t('table.actions')"
        align="center"
        width="230"
        class-name="small-padding fixed-width"
      >
        <template #default="{row}">
          <!-- 编辑 -->
          <el-button
            type="text"
            @click="onAddOrUpdate(row.spuId)"
          >
            {{ $t('table.edit') }}
          </el-button>
          <!-- 上架 -->
          <el-button
            v-if="row.status === 0"
            type="text"
            @click="enableOrDisable(row.categoryId,row.status)"
          >
            {{ $t('action.putOnShelf') }}
          </el-button>
          <!-- 下架 -->
          <el-button
            v-if="row.status === 1"
            type="text"
            @click="enableOrDisable(row.categoryId,row.status)"
          >
            {{ $t('action.offShelf') }}
          </el-button>
          <el-button
            type="text"
            @click="onDelete(row.spuId)"
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
import Pagination from '@/components/Pagination'
import AddOrUpdate from './add-or-update.vue'
import * as api from '@/api/product/spu'


  name: '',

</script>
