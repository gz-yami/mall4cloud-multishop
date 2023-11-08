<template>
  <el-dialog
    v-model:visible="visible"
    :title="dataForm.shopUserId? $t('table.edit'): $t('table.create')"
  >
    <el-form
      ref="dataFormRef"
      :rules="dataRule"
      :model="dataForm"
      label-position="right"
      label-width="80px"
    >
      <!-- 昵称 -->
      <el-form-item
        :label="$t('multishop.shopUser.nickName')"
        prop="nickName"
      >
        <el-input v-model="dataForm.nickName" />
      </el-form-item>
      <!-- 头像 -->
      <el-form-item
        :label="$t('multishop.shopUser.avatar')"
        prop="avatar"
        label-width="80px"
      >
        <img-upload v-model="dataForm.avatar" />
        <!-- <pic-upload v-model="dataForm.avatar" /> -->
      </el-form-item>
      <!-- 员工编号 -->
      <el-form-item
        :label="$t('multishop.shopUser.code')"
        prop="code"
      >
        <el-input v-model="dataForm.code" />
      </el-form-item>
      <!-- 联系方式 -->
      <el-form-item
        :label="$t('multishop.shopUser.phoneNum')"
        prop="phoneNum"
      >
        <el-input v-model="dataForm.phoneNum" />
      </el-form-item>
      <!-- 选择角色 -->
      <el-form-item
        label="选择角色"
        prop="roleIds"
      >
        <el-select
          v-model="dataForm.roleIds"
          multiple
          placeholder="请选择"
          style="width: 60%"
        >
          <el-option
            v-for="item in roleOpts"
            :key="item.roleId"
            :label="item.roleName"
            :value="item.roleId"
          />
        </el-select>
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
import * as api from '@/api/multishop/shop-user'
// import PicUpload from '@/components/PicUpload'
import * as roleApi from '@/api/rbac/role'
import ImgUpload from '@/components/ImgUpload'



  emits: ['refreshDataList'],


var visible = ref(false)
var dataForm = reactive({
  shopUserId: 0,
  shopId: null,
  nickName: null,
  avatar: null,
  code: null,
  phoneNum: null,
  hasAccount: null,
  roleIds: []
})
var roleOpts = ref([])
var dataRule = {
  nickName: [
    { required: true, message: '昵称不能为空', trigger: 'blur' }
  ],
  avatar: [
    { required: true, message: '头像不能为空', trigger: 'blur' }
  ]
}


const init  = (shopUserId) => {
  dataForm.shopUserId = shopUserId || 0
  visible = true
  nextTick(() => {
    dataFormRef.value?.resetFields()
    roleApi.list().then(data => {
      roleOpts = data
    })
    if (dataForm.shopUserId) {
      api.get(shopUserId).then(data => {
        dataForm = data
      })
    }
  })
}
// 表单提交
const onSubmit  = () => {
  dataFormRef.value?.validate(valid => {
    if (valid) {
      const request = dataForm.shopUserId ? api.update(dataForm) : api.save(dataForm)
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
    }
  })
}

</script>
