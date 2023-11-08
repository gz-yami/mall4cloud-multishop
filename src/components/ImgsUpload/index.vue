<template>
  <div class="mul-pic-upload">
    <vue-draggable
      v-model="imageList"
      class="el-upload-list el-upload-list--picture-card"
      @start="onDragStart"
      @end="onDragEnd"
    >
      <!--拷贝上传图片组件生成的预览图元素代码，用绑定的model循环生成可拖拽元素-->
      <li
        v-for="(item,index) in imageList"
        :key="index"
        tabindex="0"
        class="el-upload-list__item is-success"
      >
        <img
          :src="item.url"
          alt
          class="el-upload-list__item-thumbnail"
        >
        <a class="el-upload-list__item-name">
          <i class="el-icon-document" />
        </a>
        <label class="el-upload-list__item-status-label">
          <i class="el-icon-upload-success el-icon-check" />
        </label>
        <span class="el-upload-list__item-actions">
          <span class="el-upload-list__item-preview">
            <i
              class="el-icon-zoom-in"
              @click="handlePictureCardPreview(item.url)"
            />
          </span>
          <span
            v-if="!disabled"
            class="el-upload-list__item-delete"
            @click="handleRemove(index)"
          >
            <i class="el-icon-delete" />
          </span>
        </span>
      </li>
      <li
        v-if="!disabled && imageList.length < limit"
        class="el-upload-list__item"
        @click="elxImgboxHandle"
      >
        <div
          tabindex="0"
          class="el-upload el-upload--picture-card"
        >
          <i class="el-icon-plus" />
        </div>
        <!-- 弹窗, 新增图片 -->
        <elx-imgbox
          v-if="elxImgboxVisible"
          ref="elxImgboxRef"
          @refresh-pic="refreshPic"
        />
      </li>
    </vue-draggable>
    <!-- <div v-if="prompt">{{$t("biz.imgbox.PicMaxQuantity")}}{{limit}}</div> -->
    <el-dialog
      v-model:visible="dialogVisible"
      :modal="modal"
      top="7vh"
    >
      <img
        width="100%"
        :src="dialogImageUrl"
        alt
      >
    </el-dialog>
  </div>
</template>

<script setup>
import VueDraggable from 'vuedraggable'
import ElxImgbox from '@/components/imgbox'


const props = defineProps({
  value: {
    default: '',
    type: String
  },
  // 最大上传数量
  limit: {
    default: 9,
    type: Number
  },
  // false: 能对图片进行操作  true: 不能对图片进行操作
  disabled: {
    default: false,
    type: Boolean
  },
  modal: {
    default: true,
    type: Boolean
  },
  prompt: {
    default: true,
    type: Boolean
  }
})
  emits: ['input', 'input', 'input'],


var dialogImageUrl = ref('')
var dialogVisible = ref(false)
var elxImgboxVisible = ref(false)
const resourcesUrl = import.meta.env.VITE_APP_RESOURCES_URL
var imageList = ref([])

  watch: {
    value: function (newVal, oldVal) {
      const res = []
      if (value) {
        const imageArray = value.split(',')
        for (let i = 0; i < imageArray.length; i++) {
          if (imageArray[i]) {
            res.push({ url: getImgSrc(imageArray[i]), response: imageArray[i] })
          }
        }
      }
      imageList = res
    },
    imageList: function (newVal, oldVal) {
      const pics = imageList.map(file => {
        return file.response
      }).join(',')
      emit('update:modelValue', pics)
    }
  },


/**
 * 获取图片路径
 */
const getImgSrc  = (img) => {
  if (!img) {
    return ''
  }
  if (img.indexOf('http://') === 0 || img.indexOf('https://') === 0) {
    return img
  }
  return resourcesUrl + img
}
/**
 * 删除图片
 */
const handleRemove  = (index) => {
  imageList.splice(index, 1)
  const pics = imageList.map(file => {
    return file.response
  }).join(',')
  emit('update:modelValue', pics)
}
/**
 * 放大图片
 */
const handlePictureCardPreview  = (imgUrl) => {
  dialogImageUrl = imgUrl
  dialogVisible = true
}
const onDragStart  = (e) => {
  e.target.classList.add('hideShadow')
}
const onDragEnd  = (e) => {
  e.target.classList.remove('hideShadow')
}
/**
 * 打开图片选择窗
 */
const elxImgboxHandle  = () => {
  const num = limit - imageList.length
  if (num < 1) {
    ElMessage.error('可选择照片数量已达上限')
    return
  }
  elxImgboxVisible = true
  nextTick(() => {
    elxImgboxRef.value?.init(0, num)
  })
}
/**
 * 接收回调的图片数据
 */
const refreshPic  = (imagePath) => {
  const imageArray = imagePath.split(',')
  let pics = imageArray.map(img => {
    return img
  }).join(',')
  if (value) {
    // let picArray = imagePath.split(',')
    // console.log(picArray.length, value, !value)
    pics = value + ',' + pics
  }
  emit('update:modelValue', pics)
}

</script>

<style lang="scss" scope>
.mul-pic-upload {
  .upload-component {
    display: inline;
  }
  .el-upload-list--picture-card .el-upload-list__item {
    width: 120px;
    height: 120px;
    border: 1px dashed #d9d9d9;
  }
  .el-upload--picture-card {
    border: 0;
    font-size: 28px;
    color: #8c939d;
    background: #fff;
    width: 120px;
    height: 120px;
    line-height: 120px;
    text-align: center;
  }
  .hideShadow {
    .el-upload-list__item-actions {
      display: none;
    }
  }
}
</style>
