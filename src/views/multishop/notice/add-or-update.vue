<template>
  <el-dialog
    v-model:visible="visible"
    :title="dataForm.id? $t('table.edit'): $t('table.create')"
    :close-on-click-modal="false"
    top="5vh"
    class="natice-dialog"
  >
    <el-form
      ref="dataFormRef"
      :rules="rules"
      :model="dataForm"
      label-position="left"
      label-width="70px"
      style="margin: 0 20px;"
    >
      <!-- 公告标题 -->
      <el-form-item
        :label="$t('multishop.notice.title')"
        prop="title"
      >
        <el-input
          v-model="dataForm.title"
          placeholder="请输入公告标题"
        />
      </el-form-item>
      <!-- 状态(1:公布 0:撤回) -->
      <el-form-item
        :label="$t('multishop.notice.status')"
        prop="status"
      >
        <el-radio-group v-model="dataForm.status">
          <el-radio :label="0">
            撤销
          </el-radio>
          <el-radio :label="1">
            公布
          </el-radio>
        </el-radio-group>
      </el-form-item>
      <!-- 是否置顶 -->
      <el-form-item
        :label="$t('multishop.notice.isTop')"
        prop="isTop"
      >
        <el-radio-group v-model="dataForm.isTop">
          <el-radio :label="0">
            否
          </el-radio>
          <el-radio :label="1">
            是
          </el-radio>
        </el-radio-group>
      </el-form-item>
      <!-- 公告内容 -->
      <el-form-item
        :label="$t('multishop.notice.content')"
        prop="content"
      >
        <tiny-mce
          v-if="visible"
          v-model="dataForm.content"
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
</template>

<script setup>
import * as api from '@/api/multishop/notice'
import TinyMce from '@/components/Tinymce'


  emits: ['refreshDataList'],


var visible = ref(false)
var dataForm = reactive({
  // id: 1, // 公告id
  // shopId: 1, // 店铺id
  title: '', // 公告标题
  content: '', // 公告内容
  status: 1, // 状态(1:公布 0:撤回)
  isTop: 0, // 是否置顶
  type: 2 // 类型(1:平台公告 2:商城公告)
  // publishTime: '' // 发布时间
})
var rules = {
}


const init  = (id) => {
  dataForm.id = id || 0
  visible = true
  nextTick(() => {
    dataFormRef.value?.resetFields()
    if (!dataForm.id) {
      return
    }
    api.get(id).then(data => {
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
    const request = dataForm.id ? api.update(dataForm) : api.save(dataForm)
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
.natice-dialog {
  .el-dialog {
    min-width: 740px
  }
  .el-dialog__body {
    padding: 10px 20px;
  }
}
</style>
