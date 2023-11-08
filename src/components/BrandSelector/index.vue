<template>
  <el-dialog
    v-model:visible="visible"
    title="选择品牌"
    top="50px"
    :close-on-click-modal="false"
    class="brand-select"
  >
    <el-form
      :inline="true"
      :model="dataForm"
      class="demo-form-inline"
    >
      <el-form-item label="品牌名称">
        <el-input
          v-model.trim="brandName"
          placeholder="品牌名称"
          size="mini"
        />
      </el-form-item>
      <el-form-item>
        <el-button
          type="primary"
          size="mini"
          @click="onSearch"
        >
          查询
        </el-button>
      </el-form-item>
    </el-form>

    <div class="prods-select-body">
      <el-table
        ref="brandTable"
        v-loading="brandListLoading"
        :data="pageVO.list"
        border
        style="width: 100%;"
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
                v-model="singleSelectBrandId"
                :label="row.brandId"
                @change="getSelectBrandRow(row)"
              />
            </div>
          </template>
        </el-table-column>

        <el-table-column
          prop="brandName"
          header-align="center"
          align="center"
          label="品牌名称"
        >
          <template #default="{row}">
            {{ row.name }}
          </template>
        </el-table-column>

        <el-table-column
          prop="firstChar"
          align="center"
          label="品牌首字母"
        >
          <template #default="{row}">
            {{ row.firstLetter }}
          </template>
        </el-table-column>

        <el-table-column
          prop="status"
          align="center"
          label="状态"
        >
          <template #default="{row}">
            <el-tag size="mini">
              {{ row.status===1?'正常':'下线' }}
            </el-tag>
          </template>
        </el-table-column>

        <el-table-column
          align="center"
          width="140"
          label="品牌图片"
        >
          <template #default="{row}">
            <img
              v-if="row.imgUrl"
              :src="(row.imgUrl).indexOf('http')===-1 ? resourcesUrl + row.imgUrl : row.imgUrl"
              class="brand-img"
            >
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页条 -->
      <!-- <pagination v-show="pageVO.total>0" :total="pageVO.total" :page.sync="pageQuery.pageNum" :limit.sync="pageQuery.pageSize" @pagination="getBrandList()" /> -->
    </div>
    <template #footer>
      <span>
        <el-button
          type="primary"
          @click="visible = false"
        >取消</el-button>
        <el-button
          type="primary"
          @click="submitBrand()"
        >提交</el-button>
      </span>
    </template>
  </el-dialog>
</template>

<script>
import { getBrandByCategoryId } from '@/api/product/brand'
import Pagination from '@/components/Pagination'
export default {
  components: { Pagination },

  props: {
    isSingle: {
      default: false,
      type: Boolean
    }
  },
  emits: ['refreshSelectBrand'],

  data () {
    return {
      visible: false,
      brandName: null,
      selectBrand: [],
      dataForm: {
        brand: ''
      },
      resourcesUrl: process.env.VUE_APP_RESOURCES_URL,
      pageQuery: {
        // pageSize: 5,
        // pageNum: 1,
        categoryId: null
      },
      // 返回参数
      pageVO: {
        list: [], // 返回的列表
        total: 0, // 一共多少条数据
        pages: 0 // 一共多少页
      },
      brandListLoading: false,

      singleSelectBrandId: 0,
      selectBrand: [],
      brandListSelections: []
    }
  },

  methods: {
    init (selectBrand, categoryId) {
      this.selectBrand = selectBrand
      this.pageQuery.categoryId = categoryId
      this.visible = true
      this.brandListLoading = true
      if (this.selectBrand) {
        this.selectBrand.forEach(row => {
          this.brandListSelections.push(row)
        })
      }
      this.getBrandList()
    },

    onSearch () {
      this.getBrandList()
    },

    getBrandList () {
      getBrandByCategoryId({ ...this.pageQuery, ...this.searchParam }).then(pageVO => {
        this.pageVO.list = pageVO
        this.brandListLoading = false
      })
    },

    // 单选
    getSelectBrandRow (row) {
      this.brandListSelections = [row]
    },

    // 确定事件
    submitBrand () {
      const brands = []
      this.brandListSelections.forEach(item => {
        const brandIndex = brands.findIndex((brand) => brand.brandId === item.brandId)
        if (brandIndex === -1) {
          brands.push({
            brandId: item.brandId,
            brandName: item.name,
            brandImgUrl: (item.imgUrl).indexOf('http') === -1 ? this.resourcesUrl + item.imgUrl : item.imgUrl
          })
        }
      })
      this.$emit('refreshSelectBrand', brands)
      this.brandListSelections = []
      this.visible = false
    }
  }
}
</script>

<style lang="scss">
.brand-select {
  .prods-select-body {
    height: 600px;
    overflow: auto;
  }
  .el-dialog__header {
    border: 1px solid #eee;
  }
  .el-dialog__body {
    padding: 20px;
  }
  .el-form-item {
    margin-bottom: 20px;
  }
  .pagination-container {
    margin-top: 0;
    text-align: right;
    padding: 15px 0;
  }
  .brand-img {
    width: auto;
    max-width: 100%;
    height: auto;
    max-height: 80px;
  }
}
</style>
