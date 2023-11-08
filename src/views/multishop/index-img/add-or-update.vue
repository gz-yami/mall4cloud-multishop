<template>
  <div class="index-img">
    <el-dialog
      v-model:visible="visible"
      :title="dataForm.attrId? $t('table.edit'): $t('table.create')"
      :close-on-click-modal="false"
      :destroy-on-close="true"
      top="12vh"
      width="750px"
      @close="closeDialog"
    >
      <el-form
        ref="dataFormRef"
        :rules="rules"
        :model="dataForm"
        label-position="left"
        label-width="90px"
        style="min-width: 600px; width: 600px; margin-left:35px;"
      >
        <!-- 图片 -->
        <el-form-item
          label="轮播图片"
          prop="imgUrl"
        >
          <img-upload v-model="dataForm.imgUrl" />
          <div class="">
            建议图片尺寸为710*780
          </div>
        </el-form-item>
        <!-- 平台 -->
        <el-form-item
          label="展示平台"
          prop="imgType"
        >
          <el-radio
            v-model="dataForm.imgType"
            :label="0"
          >
            移动端
          </el-radio>
        </el-form-item>
        <!-- 状态 -->
        <el-form-item
          label="轮播图状态"
          prop="status"
        >
          <el-radio
            v-model="dataForm.status"
            :label="0"
          >
            禁用
          </el-radio>
          <el-radio
            v-model="dataForm.status"
            :label="1"
          >
            正常
          </el-radio>
        </el-form-item>
        <!-- 排序 -->
        <el-form-item
          label="轮播图排序"
          prop="seq"
        >
          <el-input-number
            v-model="dataForm.seq"
            :min="0"
            controls-position="right"
            :precision="0"
          />
        </el-form-item>
        <!-- 类型 -->
        <el-form-item
          label="轮播图类型"
          prop=""
        >
          <el-radio
            v-model="relatedSpu"
            :label="0"
          >
            无
          </el-radio>
          <el-radio
            v-model="relatedSpu"
            :label="1"
          >
            商品
          </el-radio>
          <div v-if="relatedSpu === 1 && dataForm.spu && dataForm.spu.spuId">
            <el-card
              :body-style="{ padding: '0px' }"
              style="height: 160px;width: 120px"
            >
              <img
                :src="(dataForm.spu.mainImgUrl).indexOf('http')===-1 ? resourcesUrl + dataForm.spu.mainImgUrl : dataForm.spu.mainImgUrl"
                style="height:104px;width:100%"
              >
              <div class="card-prod-bottom">
                <span class="card-prod-name">{{ dataForm.spu.spuName }}</span>
                <el-button
                  type="text"
                  class="card-prod-name-button"
                  @click="deleteCurrentProd"
                >
                  删除
                </el-button>
              </div>
            </el-card>
          </div>
          <div
            v-if="relatedSpu === 1 && !dataForm.spuId"
            class="select-pro"
          >
            <el-button @click="selectProds">
              选择商品
            </el-button>
          </div>
          <prods-select
            v-if="prodSelectVisible"
            ref="prodSelectRef"
            :is-single="true"
            @refresh-select-prods="getSelectedProd"
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
import * as api from '@/api/multishop/index-img'
import ImgUpload from '@/components/ImgUpload'
import ProdsSelect from '@/components/ProdsSelect'


  emits: ['refreshDataList'],


var visible = ref(false)
var dataForm = reactive({
  imgId: 0,
  imgUrl: null,
  seq: 0,
  imgType: 0,
  status: 1,
  spuId: null,
  spu: {} // 商品列表
})
var relatedSpu = ref(0)
const resourcesUrl = import.meta.env.VITE_APP_RESOURCES_URL
// 关联数据
var card = reactive({
  id: 0,
  pic: '',
  name: '',
  realData: {
    prod: [],
    shop: [],
    activity: []
  }
})
var prodSelectVisible = ref(false)
var prods = reactive({})
var rules = {
  imgUrl: [
    { required: true, message: '请选择轮播图图片', trigger: 'blur' }
  ]
}


const init  = (imgId) => {
  visible = true
  dataForm.imgId = imgId || 0
  nextTick(() => {
    dataFormRef.value?.resetFields()
    if (!dataForm.imgId) {
      return
    }
    api.get(imgId).then(data => {
      dataForm = data
      dataForm.spu.spuName = dataForm.spu.name
      spus = data.spus || []
      relatedSpu = data.spuId ? 1 : 0
    })
  })
}

const selectProds  = () => {
  prodSelectVisible = true
  nextTick(() => {
    prodSelectRef.value?.init()
  })
}

const getSelectedProd  = (prods) => {
  console.log(prods)
  dataForm.spu = prods[0]
  dataForm.spuId = prods[0].spuId
  dataForm.spuName = prods[0].spuName
}

const deleteCurrentProd  = () => {
  dataForm.spu = {}
  dataForm.spuId = ''
}

const closeDialog  = () => {
  dataForm = {
    imgId: 0,
    imgUrl: null,
    seq: 0,
    imgType: 0,
    status: 1,
    spuId: null,
    spu: {}
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
    if (!dataForm.imgUrl) {
      ElMessage({
        message: '请选择轮播图片',
        type: 'warning',
        duration: 1500
      })
      return
    }
    if (relatedSpu === 1 && !dataForm.spuId) {
      ElMessage({
        message: '请选择商品',
        type: 'warning',
        duration: 1500
      })
      return
    }
    if (relatedSpu === 0) {
      dataForm.spuId = ''
    }
    const dataForm = JSON.parse(JSON.stringify(dataForm))
    dataForm.spu = undefined
    const request = dataForm.imgId ? api.update(dataForm) : api.save(dataForm)
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
.index-img {
  .el-form--label-left {
    .el-form-item__label {
      text-align: right !important;
    }
  }
  .select-pro {
    margin-top: 10px;
  }
  .card-prod-bottom {
    position: relative;
    text-align: left;
    top: -15px;
    .card-prod-name {
      margin: auto;
      padding: 0 4px;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
      width: 118px;
      display: inline-block;
    }
    .card-prod-name-button {
      position: absolute;
      top: 30px;
      right: 0;
    }
    .el-button--medium {
      padding: 2px 6px !important;
    }
  }
}
</style>
