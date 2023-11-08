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
          ref="elxImgbox"
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

<script>
import VueDraggable from 'vuedraggable'
import ElxImgbox from '@/components/imgbox'
export default {

  components: {
    VueDraggable,
    ElxImgbox
  },

  props: {
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
  },
  emits: ['input', 'input', 'input'],

  data () {
    return {
      dialogImageUrl: '',
      dialogVisible: false,
      elxImgboxVisible: false,
      resourcesUrl: process.env.VUE_APP_RESOURCES_URL,
      imageList: []
    }
  },

  watch: {
    value: function (newVal, oldVal) {
      const res = []
      if (this.value) {
        const imageArray = this.value.split(',')
        for (let i = 0; i < imageArray.length; i++) {
          if (imageArray[i]) {
            res.push({ url: this.getImgSrc(imageArray[i]), response: imageArray[i] })
          }
        }
      }
      this.imageList = res
    },
    imageList: function (newVal, oldVal) {
      const pics = this.imageList.map(file => {
        return file.response
      }).join(',')
      this.$emit('input', pics)
    }
  },

  methods: {
    /**
     * 获取图片路径
     */
    getImgSrc (img) {
      if (!img) {
        return ''
      }
      if (img.indexOf('http://') === 0 || img.indexOf('https://') === 0) {
        return img
      }
      return this.resourcesUrl + img
    },
    /**
     * 删除图片
     */
    handleRemove (index) {
      this.imageList.splice(index, 1)
      const pics = this.imageList.map(file => {
        return file.response
      }).join(',')
      this.$emit('input', pics)
    },
    /**
     * 放大图片
     */
    handlePictureCardPreview (imgUrl) {
      this.dialogImageUrl = imgUrl
      this.dialogVisible = true
    },
    onDragStart (e) {
      e.target.classList.add('hideShadow')
    },
    onDragEnd (e) {
      e.target.classList.remove('hideShadow')
    },
    /**
     * 打开图片选择窗
     */
    elxImgboxHandle () {
      const num = this.limit - this.imageList.length
      if (num < 1) {
        this.$message.error('可选择照片数量已达上限')
        return
      }
      this.elxImgboxVisible = true
      this.$nextTick(() => {
        this.$refs.elxImgbox.init(0, num)
      })
    },
    /**
     * 接收回调的图片数据
     */
    refreshPic (imagePath) {
      const imageArray = imagePath.split(',')
      let pics = imageArray.map(img => {
        return img
      }).join(',')
      if (this.value) {
        // let picArray = imagePath.split(',')
        // console.log(picArray.length, this.value, !this.value)
        pics = this.value + ',' + pics
      }
      this.$emit('input', pics)
    }
  }
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
