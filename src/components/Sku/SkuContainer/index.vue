<template>
  <div class="group-container">
    <div class="spec-name">
      <div
        v-for="(item, index) in sku.leaf"
        :key="index"
        class="spec-name-int"
      >
        <el-tag
          :type="item.type"
          effect="plain"
          closable
          @close="handleRemoveSkuLeaf(index)"
        >
          {{ item[optionText] }}
        </el-tag>
        <div
          v-if="item.imgUrl"
          class="spec-imgbox"
        >
          <img-upload
            v-if="hasSkuImage"
            v-model="item.imgUrl"
            class="spec-img"
            @input="handleUploadSuccess(item, $event)"
          />
          <div
            v-if="hasSkuImage && item.imgUrl"
            class="preview-btn"
            @click="picturePreview(item.imgUrl)"
          >
            预览
          </div>
        </div>
      </div>

      <!-- 新增 -->
      <div
        v-if="sku[optionValue] && sku[optionValue] !== 0"
        class="sku-item"
      >
        <el-popover
          v-model="visiable"
          placement="bottom"
          width="200"
          trigger="click"
        >
          <el-select
            v-model="leafValue"
            class="popover-select"
            size="mini"
            multiple
            filterable
            allow-create
            default-first-option
            :popper-append-to-body="false"
            style="width:100%"
            placeholder="属性值"
            @change="createSkuLeaf"
            @visible-change="status => !status && handleSelectSku()"
          >
            <el-option
              v-for="item in skuOptions"
              :key="item[optionValue]"
              :label="item[optionText]"
              :value="item[optionValue]"
            />
          </el-select>
          <!-- 新增 -->
          <template #reference>
            <el-button
              circle
              type="primary"
              plain
              size="mini"
              class="cursor"
            >
              <i class="el-icon-plus" />
            </el-button>
          </template>
        </el-popover>
      </div>
    </div>
  </div>
</template>

<script setup>
import ImgUpload from '@/components/ImgUpload'
const noop = res => res


  inject: [
    'ease'
  ],
const props = defineProps({
  sku: {
    type: Object,
    default () {
      return {}
    }
  },
  hasSkuImage: {
    type: Boolean,
    default: false
  },
  onSkuLeafChange: {
    type: Function,
    default: noop
  }
})


var visiable = ref(false)
var leafValue = ref([])
var skuOptions = ref([])
var id = ref(0)

  computed: {
    optionValue () {
      return ease.optionValue
    },

    optionText () {
      return ease.optionText
    }
  },

  watch: {
    sku: {
      deep: true,
      immediate: true,
      handler (sku) {
        fetchLeafById(sku[optionValue])
      }
    }
  },

onMounted(() => {
  const { sku, optionValue } = this
  sku[optionValue] && fetchLeafById(sku[optionValue])
})


const handleHideVisiable  = () => {
  visiable = false
}

const handleResetLeafValue  = () => {
  leafValue = []
}

const fetchLeafById  = (id) => {
  if (!id) return
  ease.onFetchSku(id).then(skuOptions => {
    id = id
    skuOptions = skuOptions
    const skuList = []
    // 筛选未选择的属性
    skuOptions.forEach(skuItem => {
      if (!sku.leaf.find(item => item.id === skuItem.id)) {
        skuList.push(skuItem)
      }
    })
    skuOptions = skuList
  })
}

const handleRemoveSkuLeaf  = (index) => {
  const { sku } = this
  sku.leaf.splice(index, 1)

  onSkuLeafChange(sku.leaf)
}

const handleRemoveImage  = (id) => {
  const { sku, optionValue } = this
  sku?.leaf?.forEach(item => {
    if (item[optionValue] === id) {
      item.imgUrl = ''
    }
  })

  onSkuLeafChange(sku.leaf)
}

const filterSkuOptions  = (data) => {
  const oldSellData = []
  const addSelData = []
  const skuOptions = skuOptions
  data.forEach(item => {
    if (skuOptions.find(skuItem => skuItem.id === item)) {
      oldSellData.push(item)
    } else {
      addSelData.push(item)
    }
  })
  return {
    oldSellData, addSelData
  }
}

const createSkuLeaf  = (selVal) => {
  const { sku, optionValue, skuOptions } = this
  // 过滤需要新增的规格值
  // data = data.filter(item => typeof (item) === 'string')
  const { addSelData, oldSellData } = filterSkuOptions(selVal)
  if (!addSelData.length) return
  ease.onCreateSku({
    data: addSelData,
    id: sku[optionValue]
  }).then((processedNewOptions) => {
    if (processedNewOptions[0].text.length > 20) {
      ElMessage({
        message: '属性名长度不可超过20个字符',
        duration: 1500
      })
      processedNewOptions = []
    }
    skuOptions.push(...processedNewOptions)
    nextTick(() => {
      const values = processedNewOptions.map(item => item.id).concat(oldSellData)
      leafValue = values

      // const values = processedNewOptions.map(item => item.id)
      // leafValue = leafValue.filter(item => typeof (item) === 'number')
      // leafValue.push(...values)
    })
  })
}

const handleSelectSku  = (data) => {
  const { sku, hasSkuImage, optionValue, optionText, skuOptions, leafValue } = this
  const skuLeaf = skuOptions.filter(item => leafValue.indexOf(item[optionValue]) >= 0)
  skuLeaf.map(item => {
    item.is_show = hasSkuImage
  })

  const skuLeafIds = sku.leaf.map(item => item[optionValue])

  skuLeaf.forEach(item => {
    if (skuLeafIds.indexOf(item[optionValue]) < 0) {
      sku.leaf.push(item)
    }
  })

  // 过滤同名规格值
  for (let i = 0; i < sku.leaf.length; i++) {
    for (let j = i + 1; j < sku.leaf.length; j++) {
      if (sku.leaf[i][optionText] === sku.leaf[j][optionText]) {
        sku.leaf.splice(i, 1)
        j--
      }
    }
  }

  handleResetLeafValue()
  handleHideVisiable()
  onSkuLeafChange(sku.leaf)
}

const handleUploadSuccess  = (item, urls) => {
  onSkuLeafChange(sku.leaf)
}
// handleUploadSuccess2(response, file, fileList, id) {
//   let { sku, optionValue } = this

//   sku.leaf.forEach(item => {
//     if (item[optionValue] === id) {
//       item.imgUrl = response.imgUrl
//     }
//   })

//   onSkuLeafChange(sku.leaf)
// },

// 图片预览
const picturePreview  = (imgUrl) => {
  ease.onPreviewImg(imgUrl)
}

const created  = () => {
  const { sku, optionValue } = this
  sku[optionValue] && fetchLeafById(sku[optionValue])
}


</script>

<style lang="scss">
.group-container {
  .spec-name {
    margin: 15px 5px;
    .spec-name-int {
      display: inline-block;
      margin-bottom: 15px;
      margin-right: 15px;
      vertical-align: top;
      // 属性名称
      .spec-imgbox {
        display: block;
        width: 60px;
        height: 60px;
        position: relative;
        .preview-btn {
          position: absolute;
          bottom: -1px;
          left: 1px;
          display: none;
          width: 100%;
          background: rgba(0, 0, 0, 0.2);
          line-height: 1.5em;
          text-align: center;
          font-size: 12px;
          border-radius: 0 0 6px 6px;
          cursor: pointer;
        }
        .spec-img {
          // margin-left: 25px;
          margin-top: 10px;
          // sku上传图片组件大小修改
          .plugin-images {
            .el-upload {
              .pic-uploader-icon {
                width: 60px;
                height: 60px;
                line-height: 60px;
                font-size: 18px;
              }
              .pic {
                height: 60px;
              }
            }
          }
        }
      }
      .spec-imgbox:hover .preview-btn {
        display: block;
      }
      .error-tips {
        margin-left: 25px;
        margin-top: 10px;
        color: #e43130;
      }
      .el-tag--plain {
        position: relative;
        min-width: 100px;
        color: #606266;
        border-color: #dcdfe6;
        padding-right: 22px;
      }
      .el-tag--plain .el-tag__close {
        position: absolute;
        top: 50%;
        transform: translateY(-50%);
        right: 4px;
        color: #606266;
      }
      .el-tag--plain .el-tag__close:hover {
        background: #dcdfe6;
      }
    }
    .add-spec-btn {
      display: inline-block;
      font-size: 12px;
      line-height: 1.5em;
      margin-top: 10px;
      color: #02a1e9;
      cursor: pointer;
    }
    .sku-item {
      display: inline-block;
    }
  }
}
</style>
