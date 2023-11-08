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
        v-permission="['multishop:shopUser:save']"
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
      <!-- 昵称 -->
      <el-table-column
        :label="$t('multishop.shopUser.nickName')"
        prop="nickName"
        align="center"
        width="250"
      >
        <template #default="{row}">
          <span>{{ row.nickName }}</span>
        </template>
      </el-table-column>
      <!-- 头像 -->
      <el-table-column
        :label="$t('multishop.shopUser.avatar')"
        prop="avatar"
        align="center"
        width="100"
      >
        <template #default="{row}">
          <el-image
            :src="row.avatar"
            fit="contain"
          />
        </template>
      </el-table-column>
      <!-- 员工编号 -->
      <el-table-column
        :label="$t('multishop.shopUser.code')"
        prop="code"
        align="center"
      >
        <template #default="{row}">
          <span>{{ row.code }}</span>
        </template>
      </el-table-column>
      <!-- 联系方式 -->
      <el-table-column
        :label="$t('multishop.shopUser.phoneNum')"
        prop="phoneNum"
        align="center"
      >
        <template #default="{row}">
          <span>{{ row.phoneNum }}</span>
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
            v-if="!row.hasAccount"
            v-permission="['account:authAccount:shopSave']"
            type="text"
            @click="addOrUpdateAccountHandle(row.shopUserId,row.hasAccount)"
          >
            设置账号
          </el-button>
          <el-button
            v-if="row.hasAccount"
            v-permission="['account:authAccount:shopUpdate']"
            type="text"
            @click="addOrUpdateAccountHandle(row.shopUserId,row.hasAccount)"
          >
            修改账号
          </el-button>
          <el-button
            v-permission="['multishop:shopUser:update']"
            type="text"
            @click="onAddOrUpdate(row.shopUserId)"
          >
            {{ $t('table.edit') }}
          </el-button>
          <el-button
            v-permission="['multishop:shopUser:delete']"
            type="text"
            @click="onDelete(row.shopUserId)"
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
    <!-- 账户弹窗, 新增 / 修改 -->
    <account-add-or-update
      v-if="accountaddOrUpdateVisible"
      ref="accountAddOrUpdateRef"
      @refresh-data-list="getPage()"
    />
  </div>
</template>

<script setup>
import permission from '@/directive/permission/index.js'
import Pagination from '@/components/Pagination'
import AddOrUpdate from './add-or-update.vue'
import AccountAddOrUpdate from './account-add-or-update.vue'
import * as api from '@/api/multishop/shop-user'


  name: '',

</script>
