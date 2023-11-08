<template>
  <div>
    <div class="plugin-images">
      <div
        tabindex="0"
        class="el-upload el-upload--text"
        @click="elxImgboxHandle"
      >
        <img
          v-if="value"
          :src="getImgSrc()"
          class="pic"
        >
        <i
          v-else
          class="el-icon-plus pic-uploader-icon"
        />
      </div>
    </div>
    <!-- 弹窗, 新增图片 -->
    <elx-imgbox
      v-if="elxImgboxVisible"
      ref="elxImgboxRef"
      @refresh-pic="refreshPic"
    />
    <el-dialog
      v-model:visible="visible"
      :append-to-body="visible"
    >
      <img
        width="100%"
        :src="getImgSrc()"
        alt
      >
    </el-dialog>
  </div>
</template>

<script setup>
import ElxImgbox from '@/components/imgbox'


const props = defineProps({
  value: {
    default: '',
    type: String
  },
  disabled: {
    default: false,
    type: Boolean
  }
})
  emits: ['input'],


const resourcesUrl = import.meta.env.VITE_APP_RESOURCES_URL
var elxImgboxVisible = ref(false)
var visible = ref(false)


// 打开图片选择窗
const elxImgboxHandle  = () => {
  if (disabled) {
    openImg()
    return false
  }
  elxImgboxVisible = true
  nextTick(() => {
    elxImgboxRef.value?.init(1)
  })
}
/**
 * 获取图片路径
 */
const getImgSrc  = () => {
  if (!value) {
    return ''
  }
  if (value.indexOf('http://') === 0 || value.indexOf('https://') === 0) {
    return value
  }
  return resourcesUrl + value
}
const openImg  = () => {
  visible = true
}
const refreshPic  = (imagePath) => {
  console.log('imagePath', imagePath)
  emit('update:modelValue', imagePath)
}

</script>

<style lang="scss">
.plugin-images {
  display: inline-block;
  width: auto;
  .el-upload {
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
      width: 120px;
      height: 120px;
      display: block;
    }
    .el-upload:hover {
      border-color: #409eff;
    }
  }
}
</style>
