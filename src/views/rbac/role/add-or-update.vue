<template>
  <el-dialog
    v-model:visible="visible"
    :title="dataForm.roleId? $t('table.edit'): $t('table.create')"
  >
    <el-form
      ref="dataFormRef"
      :rules="dataRule"
      :model="dataForm"
      label-position="right"
      label-width="80px"
    >
      <!-- 角色名称 -->
      <el-form-item
        :label="$t('rbac.role.roleName')"
        prop="roleName"
      >
        <el-input v-model="dataForm.roleName" />
      </el-form-item>
      <!-- 备注 -->
      <el-form-item
        :label="$t('rbac.role.remark')"
        prop="remark"
      >
        <el-input v-model="dataForm.remark" />
      </el-form-item>
      <!-- 权限 -->
      <el-form-item
        size="mini"
        label="授权"
      >
        <el-tree
          ref="menuListTreeRef"
          :data="menuWithPermissions"
          :props="menuListTreeProps"
          node-key="menuId"
          show-checkbox
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
import * as api from '@/api/rbac/role'
import * as menuApi from '@/api/rbac/menu'
import { treeDataTranslate } from '@/utils'


  emits: ['refreshDataList'],


var visible = ref(false)
var dataForm = reactive({
  roleId: 0,
  roleName: null,
  remark: null,
  createUserId: null,
  bizType: null,
  tenantId: null,
  menuIds: [],
  menuPermissionIds: []
})
var dataRule = reactive({
  roleName: [
    { required: true, message: '角色名称不能为空', trigger: 'blur' }
  ]
})
var selectedMenuWithPermissions = ref([])
var iniMenuWithPermissions = ref([])
var menuWithPermissions = ref([])
var menuListTreeProps = reactive({
  value: 'menuId',
  label: 'title'
})
// 用于临时将权限转化为"菜单"的key
var tempPermissionToMenuKey = ref(5498415864)
var rules = {
}


const init  = (roleId) => {
  dataForm.roleId = roleId || 0
  visible = true
  nextTick(() => {
    dataFormRef.value?.resetFields()
    menuListTreeRef.value?.setCheckedKeys([])
    menuApi.listWithPermissions().then(menuWithPermissions => {
      iniMenuWithPermissions = menuWithPermissions
      // 生成树状结构数据
      menuWithPermissionToTree(menuWithPermissions)
    })

    if (dataForm.roleId) {
      api.get(roleId).then(data => {
        dataForm = data

        // 回显选中的选项
        const menuPermissionIdList = []
        let lastMenuIdList = []
        const tempMenuIdList = []
        const tempHaveSubMenuIdList = []
        if (dataForm.menuPermissionIds.length) {
          dataForm.menuPermissionIds.forEach(menuPermissionId => {
            if (menuPermissionId !== tempPermissionToMenuKey) {
              menuPermissionIdList.push(menuPermissionId + tempPermissionToMenuKey)
            }
          })
        }
        // 获取最后一级的菜单（没有子菜单也没有资源）
        if (dataForm.menuIds.length) {
          dataForm.menuIds.forEach(menuId => {
            const iniMenuWithPermissions = iniMenuWithPermissions
            for (const index in iniMenuWithPermissions) {
              if ((iniMenuWithPermissions[index].menuId === menuId && iniMenuWithPermissions[index].menuPermissions.length > 0) || iniMenuWithPermissions[index].parentId === menuId) {
                tempHaveSubMenuIdList.push(menuId)
                break
              }
            }
            tempMenuIdList.push(menuId)
          })
          lastMenuIdList = arrContrDiff(tempHaveSubMenuIdList, tempMenuIdList)
        }
        // 回显，只选最后一级的菜单及资源
        menuListTree.setCheckedKeys([]Ref.value?.concat(menuPermissionIdList, lastMenuIdList))
      })
    }
  })
}
// 把菜单和菜单资源数据生成树状结构（参数为数据menuWithPermissions）
const menuWithPermissionToTree  = (menuWithPermissions) => {
  const tempMenus = []
  const permissionMenuIds = []
  for (const key in menuWithPermissions) {
    const menuWithPermission = menuWithPermissions[key]
    tempMenus.push({
      menuId: menuWithPermission.menuId,
      parentId: menuWithPermission.parentId,
      title: menuWithPermission.title,
      type: 0 // 0 菜单 1按钮
    })
    if (menuWithPermission.menuPermissions.length) {
      const permissions = menuWithPermission.menuPermissions
      for (const pkey in permissions) {
        const permission = permissions[pkey]
        tempMenus.push({
          menuId: permission.menuPermissionId + tempPermissionToMenuKey,
          menuPermissionId: permission.menuPermissionId,
          parentId: permission.menuId,
          title: permission.name,
          type: 1 // 0 菜单 1按钮
        })
        permissionMenuIds.push(permission.menuId)
      }
    }
  }
  menuWithPermissions = treeDataTranslate(tempMenus, 'menuId')
}
// 切割最后一级的菜单
// spliceLastMenu(menus) {
//   for (const key in menus) {
//     const menu = menus[key]
//     if (menu.children && menu.children.length) {
//       spliceLastMenu(menu.children)
//       continue
//     }
//     if (menu.type === 0) {
//       menus.splice(menus.indexOf(menus), 1)
//     }
//   }
// },
// 数组对比，选出两数组不同的元素
const arrContrDiff  = (arr1, arr2) => {
  return arr1.concat(arr2).filter(function (v, i, arr) {
    return arr.indexOf(v) === arr.lastIndexOf(v)
  })
}
// 表单提交
const onSubmit  = () => {
  dataFormRef.value?.validate(valid => {
    if (valid) {
      const menuCheckedKeys = [].concat(menuListTree.getCheckedKeys(), $refs.menuListTreeRef.value?.getHalfCheckedKeys())
      const menuIds = []
      const menuPermissionIds = []
      if (menuCheckedKeys.length) {
        menuCheckedKeys.forEach(menuId => {
          if (menuId < tempPermissionToMenuKey) {
            menuIds.push(menuId)
          } else {
            menuPermissionIds.push(menuId - tempPermissionToMenuKey)
          }
        })
      }

      dataForm.menuIds = menuIds
      dataForm.menuPermissionIds = menuPermissionIds

      const request = dataForm.roleId ? api.update(dataForm) : api.save(dataForm)
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
