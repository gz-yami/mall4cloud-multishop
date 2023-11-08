<template>
  <el-dialog
    v-model:visible="visible"
    :title="dataForm.hotSearchId? $t('table.edit'): $t('table.create')"
  >
    <el-form
      ref="dataFormRef"
      :rules="rules"
      :model="dataForm"
      label-position="right"
      label-width="80px"
    >
      <!-- 热搜标题 -->
      <el-form-item
        :label="$t('multishop.hotSearch.title')"
        prop="title"
      >
        <el-input v-model="dataForm.title" />
      </el-form-item>
      <!-- 内容 -->
      <el-form-item
        :label="$t('multishop.hotSearch.content')"
        prop="content"
      >
        <el-input v-model="dataForm.content" />
      </el-form-item>
      <!-- 顺序 -->
      <el-form-item
        :label="$t('multishop.hotSearch.seq')"
        prop="seq"
      >
        <el-input-number
          v-model="dataForm.seq"
          controls-position="right"
          :precision="0"
        />
      </el-form-item>
      <!-- 状态 0下线 1上线 -->
      <el-form-item
        :label="$t('multishop.hotSearch.status')"
        prop="status"
      >
        <el-radio-group v-model="dataForm.status">
          <el-radio :label="0">
            {{ '下线' }}
          </el-radio>
          <el-radio :label="1">
            {{ '启用' }}
          </el-radio>
        </el-radio-group>
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
</template>

<script setup>
import * as api from '@/api/multishop/hot-search'


  emits: ['refreshDataList'],


var visible = ref(false)
var dataForm = reactive({
  hotSearchId: 0,
  content: null,
  seq: null,
  status: 1,
  title: null
})
var rules = {
  title: [
    { required: true, message: '标题不能为空', trigger: 'blur' }
  ],
  content: [
    { required: true, message: '内容不能为空', trigger: 'blur' }
  ]
}


const init  = (hotSearchId) => {
  dataForm.hotSearchId = hotSearchId || 0
  visible = true
  nextTick(() => {
    dataFormRef.value?.resetFields()
    if (!dataForm.hotSearchId) {
      return
    }
    api.get(hotSearchId).then(data => {
      dataForm = data
    })
  })
}
// 表单提交
const onSubmit  = () => {
  dataFormRef.value?.validate(valid => {
    if (!valid) {
      return
    }
    const request = dataForm.hotSearchId ? api.update(dataForm) : api.save(dataForm)
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
