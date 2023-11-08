<template>
  <el-dialog
    v-model:visible="visible"
    :title="dataForm.categoryId? $t('table.edit'): $t('table.create')"
    :close-on-click-modal="false"
    top="10vh"
    width="750px"
    @close="closeDialog"
  >
    <el-form
      ref="dataFormRef"
      :rules="rules"
      :model="dataForm"
      label-position="left"
      label-width="90px"
      class="category-form"
      style="width: 500px; margin-left:50px;"
    >
      <!-- 分类名称 -->
      <el-form-item
        :label="$t('product.category.name')"
        prop="name"
      >
        <el-input
          v-model="dataForm.name"
          :placeholder="$t('product.category.enterCateName')"
          maxlength="255"
        />
      </el-form-item>
      <!-- 分类图片 -->
      <el-form-item
        :label="$t('product.category.imgUrl')"
        prop="imgUrl"
      >
        <img-upload v-model="dataForm.imgUrl" />
        <span v-if="dataForm.parentId === 0">{{ $t('product.category.recommImgSize') + '510*80' }}</span>
      </el-form-item>
      <!-- 分类图标 -->
      <el-form-item
        :label="$t('product.category.icon')"
        prop="icon"
      >
        <img-upload v-model="dataForm.icon" />
      </el-form-item>
      <!-- 上级分类 -->
      <el-form-item
        v-if="showSelectColumnOfCategory"
        class="higher-category"
        :label="$t('product.category.categoryParent')"
      >
        <category-group
          :selected-categorys="selectedCategorys"
          :show-category-select-btn="showCategorySelectBtn"
          :single="true"
          @select-or-revise-category="selectOrReviseCategory"
        />
      </el-form-item>
      <!-- 排序 -->
      <el-form-item
        :label="$t('product.category.seq')"
        prop="seq"
      >
        <el-input-number
          v-model="dataForm.seq"
          :min="0"
          controls-position="right"
          :precision="0"
        />
      </el-form-item>
    </el-form>

    <template #footer>
      <div class="dialog-footer">
        <el-button @click="visible = false">
          {{ $t('table.cancel') }}
        </el-button>
        <el-button
          type="primary"
          @click="onSubmit()"
        >
          {{ $t('table.confirm') }}
        </el-button>
      </div>
    </template>
    <category-selector
      v-if="categorySelectorVisible"
      ref="categorySelectorRef"
      @get-category-selected="getCategorySelected"
    />
  </el-dialog>
</template>

<script setup>
import * as api from '@/api/product/category'
import ImgUpload from '@/components/ImgUpload'
import categorySelector from '@/components/CategorySelector'
import categoryGroup from '@/components/CategoryGroup'


  emits: ['refreshDataList'],


var visible = ref(false)
var dataForm = reactive({
  categoryId: 0,
  shopId: null,
  parentId: 0,
  name: null,
  desc: null,
  path: null,
  status: '1',
  icon: '',
  imgUrl: '',
  level: null,
  seq: 1
})
var showCategorySelectBtn = ref(true)
var categoryList = ref([])
var categorySelectorVisible = ref(false)
var selectedCategorys = ref([])
var showSelectColumnOfCategory = ref(true) // 是否显示上级分类栏
var rules = {
  name: [
    { required: true, message: $t('product.category.categoryNoNull'), trigger: 'blur' }
  ],
  imgUrl: [
    { required: true, message: $t('product.category.imageNoNull'), trigger: 'blur' }
  ]
}


const init  = (categoryId) => {
  visible = true
  dataForm.categoryId = categoryId || 0
  nextTick(() => {
    dataFormRef.value?.resetFields()
    if (!dataForm.categoryId) {
      return
    }
    if (categoryId) {
      api.get(categoryId).then(data => {
        dataForm = data
        selectedCategorys = data.pathNames ? data.pathNames : []
        showCategorySelectBtn = false
        data.level === 0 ? showSelectColumnOfCategory = false : showSelectColumnOfCategory = true
      })
    }
  })
}

// 关闭dialog时
const closeDialog  = () => {
  dataForm = {
    categoryId: 0,
    shopId: null,
    parentId: 0,
    name: null,
    desc: null,
    path: null,
    status: '1',
    icon: '',
    imgUrl: '',
    level: null,
    seq: 1
  }
  selectedCategorys = []
  showCategorySelectBtn = true
  showSelectColumnOfCategory = true
}

/**
 * 选择分类弹窗
 */
const selectOrReviseCategory  = () => {
  categorySelectorVisible = true
  nextTick(() => {
    categorySelectorRef.value?.init(1) // 1代表从创建分类进入
  })
}

// handleChange(val) {
//   dataForm.parentId = val[val.length - 1]
//   console.log('selectedCategory:', selectedCategory)
// },

/**
 * 获取子组件返回数据
 */
const getCategorySelected  = (selectedCategorys, parentId) => {
  console.log('父组件接收子组件数据：selectedCategorys:', selectedCategorys, '；parentId:', parentId)
  categorySelectorVisible = false
  selectedCategorys = selectedCategorys
  dataForm.parentId = parentId
}

// 表单提交
const onSubmit  = () => {
  dataFormRef.value?.validate(valid => {
    if (!valid) {
      return
    }
    // 层级 0第一级
    dataForm.level = !selectedCategorys.length ? 0 : selectedCategorys.length
    const request = dataForm.categoryId ? api.update(dataForm) : api.save(dataForm)
    request.then(data => {
      ElMessage({
        message: $t('table.actionSuccess'),
        type: 'success',
        duration: 1500,
        onClose: () => {
          visible = false
          emit('refreshDataList')
          dataFormRef.value?.resetFields()
        }
      })
    })
  })
}

</script>

<style lang="scss">
.category-form {
  .category-group {
    display: flex;
    align-items: center;
    height: 36px;
    line-height: 36px;
    .category {
      display: flex;
      align-items: center;
      .classify {
        height: 30px;
        line-height: 30px;
        padding: 0 8px;
      }
    }
  }
}
</style>
