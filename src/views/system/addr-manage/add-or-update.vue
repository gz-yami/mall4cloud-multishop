<template>
  <div class="addr-manage">
    <el-dialog
      v-model:visible="visible"
      :title="dataForm.areaId? $t('table.edit'): $t('table.create')"
      :close-on-click-modal="false"
      :destroy-on-close="true"
      width="650px"
      @close="closeDialog"
    >
      <el-form
        ref="dataFormRef"
        :rules="rules"
        :model="dataForm"
        label-position="left"
        label-width="90px"
        style="width: 500px; margin-left:50px;"
      >
        <el-form-item
          class="addr-item"
          prop="areaName"
          label="地区名称"
        >
          <el-input
            v-model="dataForm.areaName"
            placeholder="请输入名称"
          />
        </el-form-item>
        <el-form-item
          class="addr-item"
          label="上级地区"
          prop=""
        >
          <el-cascader
            v-model="selectedOptions"
            expand-trigger="hover"
            :options="areaList"
            :props="categoryTreeProps"
            :disabled="disableSelector"
            change-on-select
            filterable
            @change="handleChange"
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
    </el-dialog>
  </div>
</template>

<script setup>
import * as api from '@/api/system/addr-manage'
import { treeDataTranslate } from '@/utils'

  emits: ['refreshDataList'],


var visible = ref(false)
var disableSelector = ref(false) // 是否禁用选择上级分类
var dataForm = reactive({
  areaId: '',
  areaName: null,
  parentId: null,
  level: null
})
var areaList = ref([])
var categoryTreeProps = reactive({
  value: 'areaId',
  label: 'areaName'
})
var selectedOptions = ref([])
var value = ref('')
var rules = {
  name: [
    { required: true, message: '请填写地区名称', trigger: 'blur' }
  ]
}


const init  = (areaId, level) => {
  visible = true
  dataForm.areaId = areaId || 0
  visible = true
  isSubmit = false
  selectedOptions = []
  nextTick(() => {
    dataFormRef.value?.resetFields()
    if (dataForm.areaId) {
      api.get(areaId).then((data) => {
        dataForm = data
        selectedOptions = dataForm.parentId
        categoryTreeProps.areaId = dataForm.areaId
        categoryTreeProps.areaName = dataForm.areaName
      })
    }
    level === 0 ? disableSelector = true : disableSelector = false
    api.page().then((data) => {
      areaList = treeDataTranslate(data, 'areaId', 'parentId')
      console.log('areaList:', areaList)
    })
  })
}

const handleChange  = (val) => {
  dataForm.parentId = val[val.length - 1]
}

// 关闭弹窗时
const closeDialog  = () => {
  dataForm = {
    areaId: '',
    areaName: null,
    parentId: null,
    level: null
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
    if (!dataForm.areaName) {
      ElMessage({
        message: '地区名称不能为空！',
        type: 'warning',
        duration: 1500
      })
    }
    if (!dataForm.areaId) {
      if (!dataForm.parentId) {
        dataForm.parentId = 0
        dataForm.level = 0
      } else {
        dataForm.level = selectedOptions.length
      }
    }
    const request = dataForm.areaId ? api.update(dataForm) : api.save(dataForm)
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

<style lang="scss" scoped>
.addr-manage {
  .addr-item {
    .el-cascader--medium {
      width: 220px;
    }
  }
}
</style>
