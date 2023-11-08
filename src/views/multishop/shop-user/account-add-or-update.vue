<template>
  <el-dialog
    v-model:visible="visible"
    :title="dataForm.hasAccount? $t('table.edit'): $t('table.create')"
  >
    <el-form
      ref="dataFormRef"
      :rules="rules"
      :model="dataForm"
      label-position="left"
      label-width="70px"
      style="width: 400px; margin-left:50px;"
    >
      <!-- 用户名 -->
      <el-form-item
        :label="$t('multishop.shopUserAccount.username')"
        prop="username"
      >
        <el-input v-model="dataForm.username" />
      </el-form-item>
      <!-- 密码 -->
      <el-form-item
        :label="$t('multishop.shopUserAccount.password')"
        prop="password"
      >
        <el-input
          v-model="dataForm.password"
          show-password
        />
      </el-form-item>
      <!-- 状态 -->
      <el-form-item
        :label="$t('multishop.shopUserAccount.status')"
        prop="status"
      >
        <el-radio-group v-model="dataForm.status">
          <el-radio :label="1">
            启用
          </el-radio>
          <el-radio :label="0">
            禁用
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
import * as api from '@/api/multishop/shop-user-account'
import { USER_NAME_REGEXP } from '@/utils/validate'


  emits: ['refreshDataList'],


var visible = ref(false)
var dataForm = reactive({
  hasAccount: 0,
  userId: 0,
  email: null,
  phone: null,
  username: null,
  password: null,
  status: 1
})
var rules = {
  username: [
    { required: true, message: '请输入用户名', trigger: 'blur' },
    { pattern: USER_NAME_REGEXP, message: '用户名不能为纯数字且由数字字母下划线 4-16位组成' }
  ],
  password: [
    { pattern: /^(?!\d+$)([a-zA-Z0-9_]{4,16})$/, message: '密码不能为纯数字且由数字字母下划线 4-16位组成' }
  ]
}


const init  = (userId, hasAccount) => {
  dataForm.userId = userId
  dataForm.hasAccount = hasAccount
  visible = true
  nextTick(() => {
    dataFormRef.value?.resetFields()
    if (!dataForm.hasAccount) {
      return
    }
    api.get(userId).then(data => {
      dataForm = data
      dataForm.hasAccount = hasAccount
    })
  })
}
// 表单提交
const onSubmit  = () => {
  dataFormRef.value?.validate(valid => {
    if (!valid) {
      return
    }
    console.log(dataForm)
    const request = dataForm.hasAccount ? api.update(dataForm) : api.save(dataForm)
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
