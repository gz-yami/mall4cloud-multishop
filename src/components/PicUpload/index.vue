<template>
  <div>
    <el-upload
      class="pic-uploader-component"
      :action="resourcesUrl"
      :data="dataForm"
      :show-file-list="false"
      :on-success="handleUploadSuccess"
      :before-upload="beforeUpload"
    >
      <img
        v-if="value"
        :src="getImgSrc"
        class="pic"
      >
      <i
        v-else
        class="el-icon-plus pic-uploader-icon"
      />
    </el-upload>
  </div>
</template>

<script setup>
import { ossInfo } from '@/api/biz/oss'
import { getUUID } from '@/utils/index'

const props = defineProps({
  value: {
    default: '',
    type: String
  }
})
  emits: ['input'],


var dataForm = reactive({
  policy: '',
  signature: '',
  key: '',
  ossaccessKeyId: '',
  dir: '',
  host: ''
})
var resourcesUrl = import.meta.env.VITE_APP_RESOURCES_URL

  computed: {
    getImgSrc () {
      if (!value) {
        return ''
      }
      if (value.indexOf('http://') === 0 || value.indexOf('https://') === 0) {
        return value
      }
      return resourcesUrl + value
    }
  },


// 图片上传
const handleUploadSuccess  = (response, file, fileList) => {
  dataForm.key
  emit('update:modelValue', '/' + dataForm.key)
}
// 限制图片上传大小
const beforeUpload  = (file) => {
  const isLt2M = file.size / 1024 / 1024 < 2
  if (!isLt2M) {
    ElMessage.error('上传图片大小不能超过 2MB!')
    return false
  }

  const _self = this
  return new Promise((resolve, reject) => {
    ossInfo().then(response => {
      _self.dataForm.policy = response.policy
      _self.dataForm.signature = response.signature
      _self.dataForm.ossaccessKeyId = response.accessid
      _self.dataForm.key = response.dir + getUUID()
      _self.dataForm.dir = response.dir
      _self.dataForm.host = response.host
      resolve(true)
    }).catch(err => {
      console.log(err)
      reject(false)
    })
  })
}

</script>

<style lang="scss">
  .pic-uploader-component .el-upload {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
    .pic-uploader-icon {
      font-size: 28px;
      color: #8c939d;
      width: 120px;
      height: 120px;
      line-height: 120px;
      text-align: center;
    }
    .pic {
      width: auto;
      height: 120px;
      display: block;
    }
  }
  .pic-uploader-component .el-upload:hover {
    border-color: #409EFF;
  }

</style>
