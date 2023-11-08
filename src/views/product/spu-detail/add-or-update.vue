<template>
  <el-dialog
    v-model:visible="visible"
    :title="dataForm.spuId? $t('table.edit'): $t('table.create')"
  >
    <el-form
      ref="dataFormRef"
      :rules="rules"
      :model="dataForm"
      label-position="left"
      label-width="70px"
      style="width: 400px; margin-left:50px;"
    >
      <!-- 商品详情 -->
      <el-form-item
        :label="$t('product.spuDetail.detail')"
        prop="detail"
      >
        <el-input v-model="dataForm.detail" />
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
import * as api from '@/api/product/spu-detail'


  emits: ['refreshDataList'],


var visible = ref(false)
var dataForm = reactive({
  spuId: 0,
  detail: null
})
var rules = {
}


const init  = (spuId) => {
  dataForm.spuId = spuId || 0
  visible = true
  nextTick(() => {
    dataFormRef.value?.resetFields()
    if (!dataForm.spuId) {
      return
    }
    api.get(spuId).then(data => {
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
    const request = dataForm.spuId ? api.update(dataForm) : api.save(dataForm)
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
