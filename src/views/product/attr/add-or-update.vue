<template>
  <div class="attr-manage">
    <el-dialog
      v-model:visible="visible"
      :title="dataForm.attrId? $t('table.edit'): $t('table.create')"
      :close-on-click-modal="false"
      :destroy-on-close="true"
      width="750px"
      @close="closeDialog"
    >
      <el-form
        ref="dataFormRef"
        :rules="rules"
        :model="dataForm"
        label-position="left"
        label-width="80px"
        style="min-width: 600px; width: 600px; margin-left:35px;"
      >
        <!-- 属性名称 -->
        <el-form-item
          :label="$t('product.attr.name')"
          prop="name"
        >
          <el-input
            v-model="dataForm.name"
            maxlength="20"
            show-word-limit
            placeholder="请输入属性名称"
          />
        </el-form-item>
        <!-- 属性描述 -->
        <el-form-item
          :label="$t('product.attr.desc')"
          prop="desc"
        >
          <el-input
            v-model="dataForm.desc"
            type="textarea"
            :rows="2"
            maxlength="30"
            show-word-limit
            placeholder="请输入新增属性的相关描述，30字以内"
          />
        </el-form-item>
        <el-form-item
          :label="$t('product.attr.attrValue')"
          prop="attrValues"
        >
          <div class="attr-int">
            <el-input
              v-model="attrValue"
              clearable
              maxlength="20"
              show-word-limit
              placeholder="请输入要添加的属性值，20字以内"
            />
            <el-button
              icon="el-icon-plus"
              class="add-btn"
              @click="addAttrValue(attrValue)"
            >
              添加
            </el-button>
          </div>
          <div
            v-if="dataForm.attrValues.length"
            class="attr-val"
          >
            <div class="attr-scr">
              <div
                v-for="(attr,idx) in dataForm.attrValues"
                :key="attr.attrValueId"
                :class="['attr-val-item',attr.error?'error':'']"
              >
                <el-input
                  v-model="attr.value"
                  maxlength="20"
                  show-word-limit
                  placeholder="请输入要添加的属性值，20字以内"
                  @blur="checkIntValue(idx)"
                />
                <el-button
                  type="text"
                  class="txt-del-btn"
                  @click="deleteAttrCalue(idx)"
                >
                  删除
                </el-button>
              </div>
            </div>
          </div>
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
    </el-dialog>
    <!-- <category-selector v-if="categorySelectorVisible" ref="categorySelectorRef" @getCategorySelected="getCategorySelected" /> -->
  </div>
</template>

<script setup>
import * as api from '@/api/product/attr'
// import categorySelector from '@/components/CategorySelector'
// import categoryGroup from '@/components/CategoryGroup'


  emits: ['refreshDataList'],


var visible = ref(false)
var dataForm = reactive({
  attrId: '',
  attrType: 0, // 属性类型 0:销售属性，1:基本属性
  attrValues: [], // 属性值列表
  categoryIds: [], // 分类id列表
  desc: '', // 属性描述
  name: '', // 属性名称
  searchType: 1 // 作为搜索参数 0:不需要，1:需要
})
var rules = reactive({
  name: [
    { required: true, message: '请输入属性名称', trigger: 'blur' }
  ]
})
let attrValue = null
var selectedCategorys = ref([])
var categorySelectorVisible = ref(false) // 分类弹窗显隐





const init  = (attrId) => {
  dataForm.attrId = attrId || 0
  visible = true
  nextTick(() => {
    dataFormRef.value?.resetFields()
    if (!dataForm.attrId) {
      attrValue = null
      dataForm.attrValues = []
      return
    }
    api.get(attrId).then(data => {
      dataForm = data
      dataForm.attrValues = data.attrValues ? data.attrValues : []
      // categoryShow(data.categories) // 分类回显
    })
  })
}

/**
 * 分类回显
 */
const categoryShow  = (categorys) => {
  if (categorys.length) {
    let categoryObj = {}
    const categoryIds = []
    let catagorys = []
    categorys.forEach((el, index) => {
      categoryIds.push(el.categoryId)
      if (el.pathNames && el.pathNames.length) {
        if (el.pathNames[0]) catagorys.push(el.pathNames[0])
        if (el.pathNames[1]) catagorys.push(el.pathNames[1])
      }
      if (el.name) catagorys.push(el.name)
      categoryObj.firstCategoryName = catagorys[0]
      categoryObj.secondCategoryName = catagorys[1]
      categoryObj.threeCategoryName = catagorys[2]
      selectedCategorys.push(categoryObj)
      catagorys = []
      categoryObj = {}
    })
    dataForm.categoryIds = categoryIds
  }
}

// 关闭dialog时
const closeDialog  = () => {
  dataForm = {
    attrId: 0,
    attrValues: [], // 属性值列表
    desc: '', // 属性描述
    name: '' // 属性名称
    // attrType: 0, // 属性类型 0:销售属性，1:基本属性
    // categoryIds: [], // 分类id列表
    // searchType: 1 // 作为搜索参数 0:不需要，1:需要
  }
  selectedCategorys = []
}

/**
 * 选择分类弹窗
 */
const selectOrReviseCategory  = () => {
  categorySelectorVisible = true
  nextTick(() => {
    categorySelectorRef.value?.init() // 1代表从创建分类进入
  })
}

/**
 * 获取子组件返回数据
 */
const getCategorySelected  = (selectedCategorys, categoryId) => {
  // console.log('获取子组件返回数据：selectedCategorys:', selectedCategorys, '；parentId:', categoryId)
  categorySelectorVisible = false
  const categoryObj = {}
  categoryObj.firstCategoryName = selectedCategorys[0]
  categoryObj.secondCategoryName = selectedCategorys[1]
  categoryObj.threeCategoryName = selectedCategorys[2]
  // 去重
  if (selectedCategorys.length) {
    dataForm.categoryIds.forEach((item, index) => {
      if (item === categoryId) {
        selectedCategorys.splice(index, 1)
        dataForm.categoryIds.splice(index, 1)
      }
    })
  }
  dataForm.categoryIds.push(categoryId)
  selectedCategorys.push(categoryObj)
}

// 删除选中的某一项分类
const deleteCategoryItemOfSelected  = (index) => {
  selectedCategorys.splice(index, 1)
  dataForm.categoryIds.splice(index, 1)
}

/**
 * 添加属性值
 */
const addAttrValue  = (attrVal) => {
  if (!attrVal) {
    return
  }
  dataForm.attrValues.forEach((el, idx) => {
    if (el.value === attrVal) {
      dataForm.attrValues.splice(idx, 1)
    }
  })
  dataForm.attrValues.push({ attrValueId: null, attrId: null, value: attrVal })
  attrValue = ''
  // console.log('添加属性值dataForm.attrValues：', dataForm.attrValues)
}

/**
 * 删除属性值
 */
const deleteAttrCalue  = (idx) => {
  dataForm.attrValues.splice(idx, 1)
  // console.log('删除属性值dataForm.attrValues：', dataForm.attrValues)
}

/**
 * 失焦时属性值是否为空
 */
const checkIntValue  = (idx) => {
  if (!dataForm.attrValues[idx].value) {
    ElMessage({
      message: '已创建的属性值不能为空！请重新创建',
      type: 'warining',
      duration: 1500
    })
    dataForm.attrValues.splice(idx, 1)
  }
}

/**
 * 表单提交
 */
const onSubmit  = () => {
  dataFormRef.value?.validate(valid => {
    if (!valid) {
      return
    }
    // if (!dataForm.categoryIds.length) {
    //   ElMessage({
    //     message: '请至少选择一个分类',
    //     type: 'warining',
    //     duration: 1500
    //   })
    //   return
    // }
    if (!dataForm.attrValues.length) {
      ElMessage({
        message: '请至少创建一个属性值',
        type: 'warining',
        duration: 1500
      })
      return
    }
    const request = dataForm.attrId ? api.update(dataForm) : api.save(dataForm)
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
p {
  margin: 0;
}
.attr-manage {
  .attr-int {
    display: flex;
    align-items: center;
    justify-content: space-between;
    .add-btn {
      margin-left: 10px;
    }
    .add-btn.el-button--medium {
      padding: 10px 10px !important;
    }
  }
  .attr-val {
    background: #f8f8f8;
    margin-top: 20px;
    padding: 20px 0;
    box-sizing: border-box;
    .attr-scr {
      min-height: 40px;
      max-height: 220px;
      overflow-y: auto;
      padding: 0 20px;
    }
    // .attr-scr::-webkit-scrollbar { // 隐藏滚动条
    //   display: none;
    // }
    .attr-val-item {
      display: flex;
      align-items: center;
      justify-content: space-between;
    }
    .attr-val-item:not(:last-child) {
      margin-bottom: 10px;
    }
    .attr-val-item .int-con {
      display: block;
      width: 100%;
    }
    .attr-val-item .txt-del-btn {
      margin-left: 15px;
    }
  }
}
</style>
