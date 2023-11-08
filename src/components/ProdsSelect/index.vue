<template>
  <el-dialog
    v-model:visible="visible"
    title="商品选择"
    :modal-append-to-body="false"
    :append-to-body="true"
    top="4vh"
    :close-on-click-modal="false"
    width="800px"
    @close="closeModule"
  >
    <el-form
      :inline="true"
      :model="dataForm"
      class="demo-form-inline"
    >
      <el-form-item label="商品名称">
        <el-input
          v-model.trim="dataForm.name"
          placeholder="商品名称"
          clearable
          style="width: 180px"
        />
      </el-form-item>
      <el-form-item label="上级分类">
        <el-cascader
          v-model="selectedCategory"
          expand-trigger="hover"
          :options="categoryList"
          :props="categoryTreeProps"
          :clearable="true"
          style="width: 180px"
          @change="handleChange"
        />
      </el-form-item>
      <el-form-item>
        <el-button
          type="primary"
          icon="el-icon-search"
          @click="searchProd"
        >
          查询
        </el-button>
      </el-form-item>
      <el-form-item>
        <el-button
          icon="el-icon-delete"
          @click="clean"
        >
          清空
        </el-button>
      </el-form-item>
    </el-form>
    <div class="prods-select-body">
      <el-table
        ref="prodTableRef"
        v-loading="dataListLoading"
        :data="pageVO.list"
        border
        highlight-current-row
        style="width: 100%"
        @selection-change="selectChangeHandle"
      >
        <el-table-column
          v-if="isSingle"
          width="50"
          header-align="center"
          align="center"
        >
          <template #default="{row}">
            <div>
              <!-- native modifier has been removed, please confirm whether the function has been affected  -->
              <el-radio
                v-model="singleSelectspuId"
                :label="row.spuId"
                @change="getSelectProdRow(row)"
              >
&nbsp;
              </el-radio>
            </div>
          </template>
        </el-table-column>
        <el-table-column
          v-if="!isSingle"
          type="selection"
          header-align="center"
          align="center"
          width="50"
        />
        <el-table-column
          align="center"
          width="140"
          label="商品图片"
        >
          <template #default="{row}">
            <img
              :src="(row.mainImgUrl).indexOf('http')===-1 ? resourcesUrl + row.mainImgUrl : row.mainImgUrl"
              width="100"
              height="100"
            >
          </template>
        </el-table-column>
        <el-table-column
          prop="spuName"
          header-align="center"
          align="center"
          label="商品名称"
        />
        <el-table-column
          prop="priceFee"
          header-align="center"
          align="center"
          label="商品价格"
          width="200px"
        />
      </el-table>
    </div>
    <!-- 分页条 -->
    <pagination
      v-show="pageVO.total>0"
      v-model:page="pageQuery.pageNum"
      v-model:limit="pageQuery.pageSize"
      :total="pageVO.total"
      @pagination="getDataList()"
    />
    <template #footer>
      <span>
        <el-button @click="visible = false">取消</el-button>
        <el-button
          type="primary"
          @click="submitProds()"
        >确认</el-button>
      </span>
    </template>
  </el-dialog>
</template>

<script setup>
import { treeDataTranslate, idList } from '@/utils'
import { page } from '@/api/product/list'
import { shopCategoryPage } from '@/api/product/category'
import Pagination from '@/components/Pagination'
import Big from 'big.js'

const props = defineProps({
  isSingle: {
    default: false,
    type: Boolean
  },
  prodType: {
    default: null,
    type: Number
  },
  dataUrl: {
    default: '/prod/prod/page',
    type: String
  }
})
  emits: ['refreshSelectProds'],


var visible = ref(false)
const resourcesUrl = import.meta.env.VITE_APP_RESOURCES_URL
// 查询的参数
var pageQuery = reactive({
  pageSize: 10,
  pageNum: 1,
  spuStatus: 1
})
// 返回参数
var pageVO = reactive({
  list: [], // 返回的列表
  total: 0, // 一共多少条数据
  pages: 0 // 一共多少页
})
// 查询参数
var searchParam = {
}

var dataForm = reactive({
  name: '',
  product: ''
})
var singleSelectspuId = ref(0)
var allData = ref([])
var selectProds = ref([])
let shopCategoryId = null
var pageIndex = ref(1)
var pageSize = ref(10)
var totalPage = ref(0)
var dataListLoading = ref(false)
var addOrUpdateVisible = ref(false)
var dataListSelections = ref([])
var categoryList = ref([])
var selectedCategory = ref([])
var categoryTreeProps = {
  value: 'categoryId',
  label: 'name'
}

onActivated(() => {
  getDataList()
})


// 获取数据列表
const init  = (selectProds) => {
  singleSelectspuId = 0
  selectProds = selectProds
  visible = true
  dataListLoading = true
  clean()
  if (selectProds) {
    selectProds.forEach(row => {
      dataListSelections.push(row)
    })
  }
  getDataList()
  getCategoryList()
}
const getCategoryList  = () => {
  shopCategoryPage().then((data) => {
    categoryList = treeDataTranslate(data, 'categoryId', 'parentId')
  })
}
const getDataList  = () => {
  page({ ...pageQuery, ...searchParam }).then((pageVO) => {
    pageVO = pageVO
    pageVO.list.forEach(prod => {
      prod.priceFee = new Big(prod.priceFee).div(100).toFixed(2)
    })
    dataListLoading = false
    if (selectProds) {
      nextTick(() => {
        selectProds.forEach(row => {
          const index = pageVO.list.findIndex((prodItem) => prodItem.spuId === row.spuId)
          prodTable.toggleRowSelection(pageVORef.value?.list[index])
        })
      })
    }
  })
}
// 每页数
const sizeChangeHandle  = (val) => {
  pageQuery.pageSize = val
  pageQuery.pageNum = 1
  getDataList()
}
// 当前页
const currentChangeHandle  = (val) => {
  pageQuery.pageNum = val
  getDataList()
}
// 单选商品事件
const getSelectProdRow  = (row) => {
  dataListSelections = [row]
}
// 多选点击事件
const selectChangeHandle  = (selection) => {
  pageVO.list.forEach((tableItem) => {
    const selectedProdIndex = selection.findIndex((selectedProd) => {
      if (!selectedProd) {
        return false
      }
      return selectedProd.spuId === tableItem.spuId
    })
    const dataSelectedProdIndex = dataListSelections.findIndex((dataSelectedProd) => dataSelectedProd.spuId === tableItem.spuId)
    if (selectedProdIndex > -1 && dataSelectedProdIndex === -1) {
      dataListSelections.push(tableItem)
    } else if (selectedProdIndex === -1 && dataSelectedProdIndex > -1) {
      dataListSelections.splice(dataSelectedProdIndex, 1)
    }
  })
}
/**
 * 获取分类id
 */
const handleChange  = (val) => {
  shopCategoryId = val[val.length - 1]
}
/**
 * 根据条件搜索商品
 */
const searchProd  = () => {
  pageQuery.pageNum = 1
  searchParam = {
    name: dataForm.name,
    categoryId: shopCategoryId
  }
  getDataList()
}
/**
 * 清空搜索条件
 */
const clean  = () => {
  name = ''
  shopCategoryId = null
  selectedCategory = idList(categoryList, shopCategoryId, 'categoryId', 'children').reverse()
}

const closeModule  = () => {
  name = ''
  shopCategoryId = null
}

// 确定事件
const submitProds  = () => {
  const prods = []
  dataListSelections.forEach(item => {
    const prodIndex = prods.findIndex((prod) => prod.spuId === item.spuId)
    if (prodIndex === -1) {
      prods.push(
        {
          spuId: item.spuId,
          spuName: item.spuName,
          mainImgUrl: item.mainImgUrl,
          activityId: item.activityId,
          prodType: item.prodType
        }
      )
    }
  })
  // var msgInfo = ''
  // // 秒杀活动选择商品的提示
  // if (dataUrl.includes('canSekcillProdPage')) {
  //   msgInfo = $t('components.seckillWhetherToContinue')
  // } else if (dataUrl.includes('getNotGroupProdPage')) {
  //   // 拼团活动选择商品的提示
  //   msgInfo = $t('components.groupWhetherToContinue')
  // }
  // if (msgInfo !== '' && msgInfo !== null) {
  //   prodIsSeckill(prods, msgInfo)
  // }else {
  emit('refreshSelectProds', prods)
  dataListSelections = []
  visible = false
  // }
}
/**
 * 查询商品是否在参与秒杀活动
 */
// prodIsSeckill(prods, msgInfo) {
//   let spuIds = []
//   for (let index = 0; index < prods.length; index++) {
//     spuIds.push(prods[index].spuId)
//   }
//   http({
//     url: http.adornUrl('/admin/discount/prodIsDiscount'),
//     method: 'post',
//     data: spuIds
//   }).then(({ data }) => {
//     var msg = data
//     if (msg !== undefined && msg !== null && msg !== '') {
//       ElMessageBox.confirm(msgInfo, $t('text.tips'), {
//         confirmButtonText: $t('crud.filter.submitBtn'),
//         cancelButtonText: $t('crud.filter.cancelBtn'),
//         type: 'warning'
//       }).then(() => {
//         emit('refreshSelectProds', prods)
//         dataListSelections = []
//         visible = false
//       }).catch(() => { })
//     } else {
//       emit('refreshSelectProds', prods)
//       dataListSelections = []
//       visible = false
//     }
//   })
// }

</script>
<style lang="scss" scope>
.demo-form-inline {
  .el-form-item.el-form-item--medium {
    margin-bottom: 22px;
  }
}
.el-dialog__body {
  padding: 20px;
}
.el-dialog__header {
  border-bottom: 1px solid #eee;
}
.pagination-container {
  padding: 0 !important;
  margin-top: 20px;
}
.prods-select-body {
  height: 600px;
  overflow: auto;
  // border-top: 1px solid #eeeeee;
  // border-right: 1px solid #eeeeee;
  // border-bottom: 1px solid #eeeeee;
}
.el-dialog__footer {
  padding-top: 0;
}
</style>
