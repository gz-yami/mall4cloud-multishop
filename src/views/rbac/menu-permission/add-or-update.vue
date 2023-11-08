<template>
  <el-dialog
    v-model:visible="visible"
    :title="dataForm.menuPermissionId? $t('table.edit'): $t('table.create')"
  >
    <div class="up-wrapper">
      <!-- native modifier has been removed, please confirm whether the function has been affected  -->
      <el-form
        ref="dataFormRef"
        :model="dataForm"
        :rules="dataRule"
        label-width="80px"
        @keyup.enter="onSubmit()"
      >
        <!-- 资源名称 -->
        <el-form-item
          :label="$t('rbac.menuPermission.name')"
          prop="name"
        >
          <el-input v-model="dataForm.name" />
        </el-form-item>
        <!-- 上级菜单 -->
        <el-form-item
          label="上级菜单"
          prop="menuId"
        >
          <el-cascader
            v-model="selectedMenu"
            expand-trigger="hover"
            :options="menuList"
            :props="menuListTreeProps"
            change-on-select
            @change="handleSelectMenuChange"
          />
        </el-form-item>
        <!-- 请求方法 1.GET 2.POST 3.PUT 4.DELETE -->
        <el-form-item
          label="请求方法"
          prop="method"
        >
          <el-radio-group v-model="dataForm.method">
            <el-radio
              v-for="index of 4"
              :key="index"
              :label="index"
            >
              {{ ['', 'GET', 'POST', 'PUT', 'DELETE'][index] }}
            </el-radio>
          </el-radio-group>
        </el-form-item>
        <!-- 资源对应服务器路径 -->
        <el-form-item
          label="资源路径"
          prop="uri"
        >
          <el-input v-model="dataForm.uri" />
        </el-form-item>
        <!-- 权限对应的编码 -->
        <el-form-item
          label="授权标识"
          prop="permission"
        >
          <el-input v-model="dataForm.permission" />
        </el-form-item>
      </el-form>
    </div>
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
import { treeDataTranslate, idList } from '@/utils'
import * as menuApi from '@/api/rbac/menu'
import * as api from '@/api/rbac/menu-permission'


  emits: ['refreshDataList'],


var visible = ref(false)
var dataForm = reactive({
  menuPermissionId: 0,
  menuId: null,
  permission: null,
  name: '',
  uri: '',
  method: 1
})
var dataRule = reactive({
  name: [
    { required: true, message: '资源名称不能为空', trigger: 'blur' }
  ],
  menuId: [
    { required: true, message: '上级菜单不能为空', trigger: 'blur' }
  ],
  method: [
    { required: true, message: '请求方法不能为空', trigger: 'blur' }
  ],
  uri: [
    { required: true, message: '资源路径不能为空', trigger: 'blur' }
  ],
  permission: [
    { required: true, message: '授权标识不能为空', trigger: 'blur' }
  ]
})
var menuList = ref([])
var selectedMenu = ref([])
var menuListTreeProps = {
  value: 'id',
  label: 'name'
}


const init  = (menuPermissionId) => {
  dataForm.menuPermissionId = menuPermissionId || 0
  visible = true
  nextTick(() => {
    dataFormRef.value?.resetFields()
    menuApi.menuList({ ...searchParam }).then(data => {
      menuList = treeDataTranslate(data)
      pageLoading = false
    })
    if (dataForm.menuPermissionId) {
      api.get(menuPermissionId).then(data => {
        dataForm = data
        selectedMenu = idList(menuList, data.menuId).reverse()
      })
    } else {
      selectedMenu = []
    }
  })
}
const handleSelectMenuChange  = (val) => {
  dataForm.menuId = val[val.length - 1]
}
// 表单提交
const onSubmit  = () => {
  dataFormRef.value?.validate(valid => {
    if (valid) {
      const params = {
        menuPermissionId: dataForm.menuPermissionId || undefined,
        menuId: dataForm.menuId || undefined,
        name: dataForm.name,
        permission: dataForm.permission,
        uri: dataForm.uri,
        method: dataForm.method
      }

      const request = dataForm.menuPermissionId ? api.update(params) : api.save(params)
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
