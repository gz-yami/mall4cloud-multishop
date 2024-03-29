<template>
  <div class="page-category">
    <!-- 搜索相关区域 -->
    <div class="filter-container">
      <el-button
        v-permission="['product:category:save']"
        type="primary"
        class="filter-item"
        @click="addOrUpdateHandle()"
      >
        {{ $t('table.create') }}
      </el-button>
    </div>

    <!-- 分类列表 -->
    <div :class="['category-list', threeList.length ? 'box-border' : '']">
      <div
        v-for="(renderList, level) in renderListData"
        :key="level"
        :class="['addr-list-box', renderListData.length?'active':'']"
      >
        <div class="addr-list">
          <!-- <div v-for="(item, index) in renderList" :key="index" :class="['addr-item', selectedsData.length > 0 && selectedsData[selectedsData.length-1].categoryId == item.categoryId ? 'active' : '']"> -->
          <div
            v-for="(item, index) in renderList"
            :key="index"
            :class="['addr-item', currentSelectId == item.categoryId ? 'active' : '']"
          >
            <div
              class="dot"
              @click="handleAddrListSelect(item, level)"
            >
              {{ item.name }}
            </div>
            <div class="btn">
              <div class="tag">
                <el-tag
                  v-if="item.status == 0"
                  type="danger"
                >
                  下架
                </el-tag>
                <el-tag
                  v-else
                >
                  正常
                </el-tag>
              </div>
              <el-button
                v-permission="['product:category:update']"
                type="primary"
                link
                @click="addOrUpdateHandle(item.categoryId)"
              >
                编辑
              </el-button>
              <!-- 上架 -->
              <el-button
                v-if="item.status === 0"
                type="primary"
                link
                @click="enableOrDisable(item.categoryId, item.status, index)"
              >
                {{ $t('action.putOnShelf') }}
              </el-button>
              <!-- 下架 -->
              <el-button
                v-if="item.status === 1"
                type="primary"
                link
                @click="enableOrDisable(item.categoryId, item.status, index)"
              >
                {{ $t('action.offShelf') }}
              </el-button>
              <el-button
                v-permission="['product:category:delete']"
                type="primary"
                link
                @click="deleteHandle(item, level)"
              >
                删除
              </el-button>
            </div>
          </div>
        </div>
      </div>

      <div
        v-if="!renderListData.length"
        class="nodata"
      >
        暂无数据
      </div>
    </div>
    <!-- 分页条 -->
    <!-- <pagination v-show="pageVO.total>0" :total="pageVO.total" :page.sync="pageQuery.pageNum" :limit.sync="pageQuery.pageSize" @pagination="getPage()" /> -->
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update
      v-if="addOrUpdateVisible"
      ref="addOrUpdateRef"
      @refresh-data-list="getPage()"
    />
  </div>
</template>

<script setup>
import * as api from '@/api/product/category'
import { onMounted, reactive } from 'vue'
import AddOrUpdate from './components/add-or-update.vue'

const Data = reactive({
  selectedsData: [], // 记录选中值
  renderListData: [], // 地址列表
  resourcesUrl: import.meta.env.VITE_APP_RESOURCES_URL,
  // 查询的参数
  pageQuery: {
    pageSize: 10,
    pageNum: 1
  },
  // 返回参数
  pageVO: {
    list: [] // 返回的列表
    // total: 0, // 一共多少条数据
    // pages: 0 // 一共多少页
  },
  secondList: [],
  threeList: [],
  secondItemId: null,
  firstItemId: null,
  pageLoading: true,
  // 查询参数
  searchParam: {
  },
  addOrUpdateVisible: false,
  disable: false,
  currentSelectId: '' // 当前选中的id
})
const { renderListData, threeList, addOrUpdateVisible, currentSelectId } = toRefs(Data)

onMounted(() => {
  getPage()
})

const getPage = () => {
  Data.pageLoading = true
  api.shopCategoryPage({ ...Data.pageQuery, ...Data.searchParam }).then(pageVO => {
    Data.secondList = Data.threeList = []
    Data.pageVO.list = treeDataTranslate(pageVO, 'categoryId', 'parentId')
    Data.pageLoading = false
    setInitRenderListData(Data.pageVO.list)
  })
}

// 设置初始数据
const setInitRenderListData = (treeData) => {
  Data.selectedsData.splice(0, Data.selectedsData.length) // 1. 删除旧已选中数据
  // 删除旧的渲染列表并添加新的第一级渲染列表
  if (treeData.length > 0) {
    changeRenderListData({
      startLevel: 0,
      changeLength: Data.renderListData.length, // 删除的长度
      changeValue: [treeData] // 添加新值
    })
  }
}

// 获取渲染列表
const getRenderListData = (data) => {
  return (data && data.children) || []
}

/**
     * 监听列表项选择
     */
const handleAddrListSelect = (selectData, level) => {
  Data.selectedsData.splice(level, 1, selectData) // 记录选中值
  Data.currentSelectId = selectData.categoryId
  // 生成下一级的列表渲染数据
  const childRenderData = getRenderListData(selectData)
  // 删除所有子孙级渲染数据, 并添加下级渲染数据
  if (childRenderData.length > 0) {
    changeRenderListData({
      startLevel: level + 1,
      changeLength: Data.renderListData.length,
      changeValue: [childRenderData]
    })
  }
}

// 更改渲染列表数据
const changeRenderListData = ({ startLevel, changeLength, changeValue }) => {
  Data.renderListData.splice(startLevel, changeLength, ...changeValue)
}

/**
     * 更新
     */
const addOrUpdateRef = ref()
const addOrUpdateHandle = (categoryId) => {
  Data.addOrUpdateVisible = true
  nextTick(() => {
    addOrUpdateRef.value.init(categoryId)
  })
}

/**
     * 删除
     */
const deleteHandle = (item, level) => {
  ElMessageBox.confirm($t('table.sureToDelete'), $t('table.tips'), {
    confirmButtonText: $t('table.confirm'),
    cancelButtonText: $t('table.cancel'),
    type: 'warning'
  }).then(() => {
    deleteById(item.categoryId)
    changeRenderListData({
      startLevel: level,
      changeLength: Data.renderListData.length
    })
  })
}

const deleteById = (categoryId) => {
  api.deleteById(categoryId).then(() => {
    ElMessage({
      message: $t('table.actionSuccess'),
      type: 'success',
      duration: 1500,
      onClose: () => getPage()
    })
  })
}

/**
     * 上架/下架分类
     */
const enableOrDisable = (categoryId, sts, idx) => {
  const categoryDTO = {}
  categoryDTO.categoryId = categoryId
  categoryDTO.status = sts === 0 ? 1 : 0
  ElMessageBox.confirm(sts === 1 ? '下架当前分类后，当前分类下的所有商品也将被下架，是否确认下架当前分类?' : '是否确认上架当前分类?', '提示', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning'
  }).then(() => {
    api.enableOrDisable(categoryDTO).then((data) => {
      ElMessage({
        message: data ? '操作成功' : data,
        type: data ? 'success' : 'error',
        duration: 1500,
        onClose: () => getPage()
      })
    })
  }).catch(() => { })
}
</script>

<style lang="scss" scoped>
.page-category {
  // 地址列表区域
  .category-list {
    // margin: 0 auto;
    display: flex;
    margin: 20px 0;
    overflow-x: auto;
    .addr-list-box {
      width: 28%;
      min-width: 380px;
      padding: 15px 0;
      .addr-list {
        height: 650px;
        overflow: auto;
        .addr-item {
          font-size: 14px;
          color: #444;
          line-height: 2.5em;
          display: flex;
          align-items: center;
          justify-content: space-between;
          .dot {
            position: relative;
            padding-left: 20px;
            cursor: pointer;
            width: 40%;
            min-width: 100px;
            box-sizing: border-box;
            white-space: nowrap;
            overflow: hidden;
            text-overflow: ellipsis;
          }
          .dot::before {
            content: "";
            position: absolute;
            left: 10px;
            top: 50%;
            transform: translateY(-50%);
            width: 4px;
            height: 4px;
            border-radius: 50%;
            background: #bbb;
          }
          .btn {
            .tag {
              display: inline-block;
              margin-right: 10px;
            }
            min-width: 98px;
            margin-right: 10px;
          }
        }
        .addr-item:hover,
        .addr-item.active {
          background: #f5f7fa;
        }
      }
    }
    .addr-list-box.active {
      border: 1px solid #eee;
    }
    .addr-list-box.active:not(:first-child) {
      border-left: 0;
    }
    .nodata {
      width: 100%;
      text-align: center;
      font-size: 13px;
      color: #aaa;
      margin-top: 50px;
    }
  }
  .category-list.box-border {
    border-left: 1px solid #eee;
    border-right: 1px solid #eee;
  }
}
</style>
