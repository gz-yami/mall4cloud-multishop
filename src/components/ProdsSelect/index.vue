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
        ref="prodTable"
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

<script>
import { treeDataTranslate, idList } from '@/utils'
import { page } from '@/api/product/list'
import { shopCategoryPage } from '@/api/product/category'
import Pagination from '@/components/Pagination'
import Big from 'big.js'
export default {
  components: { Pagination },

  props: {
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
  },
  emits: ['refreshSelectProds'],

  data () {
    return {
      visible: false,
      resourcesUrl: process.env.VUE_APP_RESOURCES_URL,
      // 查询的参数
      pageQuery: {
        pageSize: 10,
        pageNum: 1,
        spuStatus: 1
      },
      // 返回参数
      pageVO: {
        list: [], // 返回的列表
        total: 0, // 一共多少条数据
        pages: 0 // 一共多少页
      },
      // 查询参数
      searchParam: {
      },

      dataForm: {
        name: '',
        product: ''
      },
      singleSelectspuId: 0,
      allData: [],
      selectProds: [],
      shopCategoryId: null,
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      addOrUpdateVisible: false,
      dataListSelections: [],
      categoryList: [],
      selectedCategory: [],
      categoryTreeProps: {
        value: 'categoryId',
        label: 'name'
      }
    }
  },

  activated () {
    this.getDataList()
  },

  methods: {
    // 获取数据列表
    init (selectProds) {
      this.singleSelectspuId = 0
      this.selectProds = selectProds
      this.visible = true
      this.dataListLoading = true
      this.clean()
      if (this.selectProds) {
        this.selectProds.forEach(row => {
          this.dataListSelections.push(row)
        })
      }
      this.getDataList()
      this.getCategoryList()
    },
    getCategoryList () {
      shopCategoryPage().then((data) => {
        this.categoryList = treeDataTranslate(data, 'categoryId', 'parentId')
      })
    },
    getDataList () {
      page({ ...this.pageQuery, ...this.searchParam }).then((pageVO) => {
        this.pageVO = pageVO
        this.pageVO.list.forEach(prod => {
          prod.priceFee = new Big(prod.priceFee).div(100).toFixed(2)
        })
        this.dataListLoading = false
        if (this.selectProds) {
          this.$nextTick(() => {
            this.selectProds.forEach(row => {
              const index = this.pageVO.list.findIndex((prodItem) => prodItem.spuId === row.spuId)
              this.$refs.prodTable.toggleRowSelection(this.pageVO.list[index])
            })
          })
        }
      })
    },
    // 每页数
    sizeChangeHandle (val) {
      this.pageQuery.pageSize = val
      this.pageQuery.pageNum = 1
      this.getDataList()
    },
    // 当前页
    currentChangeHandle (val) {
      this.pageQuery.pageNum = val
      this.getDataList()
    },
    // 单选商品事件
    getSelectProdRow (row) {
      this.dataListSelections = [row]
    },
    // 多选点击事件
    selectChangeHandle (selection) {
      this.pageVO.list.forEach((tableItem) => {
        const selectedProdIndex = selection.findIndex((selectedProd) => {
          if (!selectedProd) {
            return false
          }
          return selectedProd.spuId === tableItem.spuId
        })
        const dataSelectedProdIndex = this.dataListSelections.findIndex((dataSelectedProd) => dataSelectedProd.spuId === tableItem.spuId)
        if (selectedProdIndex > -1 && dataSelectedProdIndex === -1) {
          this.dataListSelections.push(tableItem)
        } else if (selectedProdIndex === -1 && dataSelectedProdIndex > -1) {
          this.dataListSelections.splice(dataSelectedProdIndex, 1)
        }
      })
    },
    /**
     * 获取分类id
     */
    handleChange (val) {
      this.shopCategoryId = val[val.length - 1]
    },
    /**
     * 根据条件搜索商品
     */
    searchProd () {
      this.pageQuery.pageNum = 1
      this.searchParam = {
        name: this.dataForm.name,
        categoryId: this.shopCategoryId
      }
      this.getDataList()
    },
    /**
     * 清空搜索条件
     */
    clean () {
      this.name = ''
      this.shopCategoryId = null
      this.selectedCategory = idList(this.categoryList, this.shopCategoryId, 'categoryId', 'children').reverse()
    },

    closeModule () {
      this.name = ''
      this.shopCategoryId = null
    },

    // 确定事件
    submitProds () {
      const prods = []
      this.dataListSelections.forEach(item => {
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
      // if (this.dataUrl.includes('canSekcillProdPage')) {
      //   msgInfo = this.$i18n.t('components.seckillWhetherToContinue')
      // } else if (this.dataUrl.includes('getNotGroupProdPage')) {
      //   // 拼团活动选择商品的提示
      //   msgInfo = this.$i18n.t('components.groupWhetherToContinue')
      // }
      // if (msgInfo !== '' && msgInfo !== null) {
      //   this.prodIsSeckill(prods, msgInfo)
      // }else {
      this.$emit('refreshSelectProds', prods)
      this.dataListSelections = []
      this.visible = false
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
    //   this.$http({
    //     url: this.$http.adornUrl('/admin/discount/prodIsDiscount'),
    //     method: 'post',
    //     data: spuIds
    //   }).then(({ data }) => {
    //     var msg = data
    //     if (msg !== undefined && msg !== null && msg !== '') {
    //       this.$confirm(msgInfo, this.$i18n.t('text.tips'), {
    //         confirmButtonText: this.$i18n.t('crud.filter.submitBtn'),
    //         cancelButtonText: this.$i18n.t('crud.filter.cancelBtn'),
    //         type: 'warning'
    //       }).then(() => {
    //         this.$emit('refreshSelectProds', prods)
    //         this.dataListSelections = []
    //         this.visible = false
    //       }).catch(() => { })
    //     } else {
    //       this.$emit('refreshSelectProds', prods)
    //       this.dataListSelections = []
    //       this.visible = false
    //     }
    //   })
    // }
  }
}
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
