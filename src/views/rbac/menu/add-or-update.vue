<template>
  <el-dialog
    v-model:visible="visible"
    :title="dataForm.menuId? $t('table.edit'): $t('table.create')"
  >
    <!-- 目录和菜单框 -->
    <!-- native modifier has been removed, please confirm whether the function has been affected  -->
    <el-form
      ref="dataFormRef"
      :model="dataForm"
      :rules="dataRule"
      label-width="80px"
      @keyup.enter="onSubmit()"
    >
      <!-- 类型选择 0：目录 1：菜单 -->
      <el-form-item
        label="类型"
        prop="type"
      >
        <el-radio-group v-model="dataForm.type">
          <el-radio
            v-for="(type, index) in dataForm.typeList"
            :key="index"
            :label="index"
          >
            {{ type }}
          </el-radio>
        </el-radio-group>
      </el-form-item>
      <!-- 目录/菜单名称 -->
      <el-form-item
        :label="dataForm.typeList[dataForm.type] + '名称'"
        prop="name"
      >
        <el-input
          v-model="dataForm.name"
          :placeholder="dataForm.typeList[dataForm.type] + '名称'"
        />
      </el-form-item>
      <!-- 上级菜单 -->
      <el-form-item label="上级菜单">
        <el-cascader
          v-model="selectedMenu"
          clearable
          expand-trigger="hover"
          :options="menuList"
          :props="menuListTreeProps"
          change-on-select
          @change="handleSelectMenuChange"
        />
      </el-form-item>
      <!-- 菜单路由 -->
      <el-form-item
        label="菜单路由"
        prop="url"
      >
        <el-input
          v-model="dataForm.url"
          placeholder="菜单路由"
        />
      </el-form-item>
      <!-- 组件路径 -->
      <el-form-item
        v-if="dataForm.type === 1"
        label="组件路径"
        prop="componentUrl"
      >
        <el-input
          v-model="dataForm.componentUrl"
          placeholder="组件路径"
        />
      </el-form-item>
      <!-- 排序号 -->
      <el-form-item
        v-if="dataForm.type !== 2"
        label="排序号"
        prop="seq"
      >
        <el-input-number
          v-model="dataForm.seq"
          controls-position="right"
          :min="0"
        />
      </el-form-item>
      <!-- 图标 -->
      <el-form-item
        label="icon"
        prop="icon"
      >
        <el-input
          v-model="dataForm.icon"
          v-popover:iconListPopover
          :readonly="true"
        />
        <el-popover
          ref="iconListPopoverRef"
          placement="bottom"
          trigger="click"
          popper-class="mod-menu-home__icon-popover"
        >
          <icons-select @select-icon="selectIcon" />
        </el-popover>
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
import * as api from '@/api/rbac/menu'
import { treeDataTranslate, idList } from '@/utils'
import IconsSelect from '@/components/IconsSelect'


  emits: ['refreshDataList'],


var visible = ref(false)
var dataForm = reactive({
  menuId: 0,
  type: 1,
  typeList: ['目录', '菜单'],
  name: '',
  parentId: 0,
  url: '',
  componentUrl: '',
  seq: 0,
  icon: '',
  iconList: []
})
var dataRule = reactive({
  name: [
    { required: true, validator: validateName, trigger: 'blur' }
  ],
  url: [
    { required: true, validator: validateUrl, trigger: 'blur' }
  ],
  componentUrl: [
    { required: true, validator: validateComponent, trigger: 'blur' }
  ]
})
var menuList = ref([])
var selectedMenu = ref([])
var menuListTreeProps = reactive({
  value: 'id',
  label: 'name'
})
var iconPopoverVisible = ref([])

  watch: {
    // 如果弹窗关闭，dataForm里的值初始化
    visible (val) {
      if (val === false) {
        initFrameCon()
      }
    }
  },


const init  = (menuId) => {
  dataForm.menuId = menuId || 0
  visible = true
  nextTick(() => {
    // $refs['dataForm'].resetFields()
    api.menuList({ ...searchParam }).then(data => {
      menuList = treeDataTranslate(data)
      pageLoading = false
    })
    if (dataForm.menuId) {
      api.get(menuId).then(data => {
        // 不是目录，才显示组件路径（data.component返回Layout就为目录）
        if (data.component !== 'Layout') {
          dataForm.type = 1
          dataForm.componentUrl = data.component
        } else {
          dataForm.type = 0
          dataForm.componentUrl = ''
        }
        dataForm.menuId = data.menuId
        dataForm.name = data.title
        dataForm.parentId = data.parentId
        dataForm.url = data.path
        dataForm.seq = data.seq
        dataForm.icon = data.icon
        selectedMenu = idList(menuList, data.parentId).reverse()
      })
    } else {
      selectedMenu = []
    }
  })
}
// 弹框内容初始化
const initFrameCon  = () => {
  dataForm = {
    menuId: 0,
    type: 1,
    typeList: ['目录', '菜单'],
    name: '',
    parentId: 0,
    url: '',
    componentUrl: '',
    seq: 0,
    icon: '',
    iconList: []
  }
}
// 监听上级菜单的改动
const handleSelectMenuChange  = (val) => {
  dataForm.parentId = val[val.length - 1]
  if (dataForm.parentId === undefined) {
    dataForm.parentId = 0
  }
}
// 选择图标
const selectIcon  = (iconName) => {
  dataForm.icon = iconName
}
// 表单提交
const onSubmit  = () => {
  dataFormRef.value?.validate(valid => {
    if (valid) {
      // 提交为目录，component为Layout
      let component = ''
      if (parseInt(dataForm.type) === 0) {
        component = 'Layout'
      } else {
        component = dataForm.componentUrl
      }
      const params = {
        menuId: dataForm.menuId || undefined,
        name: dataForm.name,
        title: dataForm.name,
        parentId: dataForm.parentId,
        component,
        path: dataForm.url,
        seq: dataForm.seq,
        icon: dataForm.icon
      }
      const request = dataForm.menuId ? api.update(params) : api.save(params)
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

<style lang="scss">
.mod-menu-home{
  &__icon-popover {
    max-width: 420px;
    max-height: 280px;
  }
}
</style>
