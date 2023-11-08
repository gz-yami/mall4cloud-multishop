<template>
  <div class="ease-sku">
    <sku-group
      v-for="(item, index) in data"
      :key="index"
      :index="index"
      :sku="item"
      :sku-tree="skuTreeData"
      :on-change-has-img="onChangeHasImg"
      :on-sku-change="rebuildSku"
      :on-sku-remove="handleSkuRemove"
    />
    <!-- 添加属性 -->
    <div class="part-form-item">
      <el-button
        class="add-btn"
        size="mini"
        @click="addSku"
      >
        <i class="el-icon-plus" />添加属性
      </el-button>
      <div
        v-if="!categoryId && warning"
        class="warning"
      >
        请先选择分类！
      </div>
      <div
        v-if="!hasSkuValue"
        class="warning"
      >
        请先选择已添加属性的属性值！
      </div>
      <div
        v-if="!hasSkuId"
        class="warning"
      >
        请先选择已添加属性！
      </div>
    </div>
  </div>
</template>

<script setup>
import skuGroup from '@/components/Sku/SkuGroup'
import { ElMessage as Message } from 'element-plus'

const noop = res => res
const noopPromise = () => Promise.resolve(noop)





  provide () {
    return {
      ease: this
    }
  },
const props = defineProps({
  maxSize: {
    type: Number,
    default: 2
  },
  hasSkuImg: {
    type: Boolean,
    default: false
  },
  showAddSkuImage: {
    type: Boolean,
    default: true
  },
  salesAttrs: {
    type: Array,
    default () {
      return []
    }
  },
  saveSalesAttrs: {
    type: Array,
    default () {
      return []
    }
  },
  action: {
    type: String,
    default: ''
  },
  headers: {
    type: Object,
    default () {
      return {}
    }
  },
  accept: {
    type: String,
    default: ''
  },
  uploadName: {
    type: String,
    default: 'resource'
  },
  // 可选规格列表
  skuTree: {
    type: Array,
    default () {
      return []
    }
  },
  sku: {
    type: Object,
    default () {
      return {}
    }
  },
  categoryId: {
    type: Number,
    default: null
  },
  // 自定义sku的id key
  optionValue: {
    type: String,
    default: 'id'
  },
  // 自定义sku的text key
  optionText: {
    type: String,
    default: 'text'
  },
  // 异步获取规格列表
  onFetchGroup: {
    type: Function,
    default: noopPromise
  },
  // 异步获取规格可选值
  onFetchSku: {
    type: Function,
    default: noopPromise
  },
  // 创建新的规格名
  onCreateGroup: {
    type: Function,
    default: noopPromise
  },
  // 创建新的规格值
  onCreateSku: {
    type: Function,
    default: noopPromise
  },
  // 图片预览
  onPreviewImg: {
    type: Function,
    default: noopPromise
  },
  hasSkuVal: {
    type: Boolean,
    default: null
  }
})
  emits: ['input', 'on-change', 'on-change'],


var data = reactive(salesAttrs)
var skuTreeData = reactive(skuTree)
var warning = ref(false)
var hasSkuValue = reactive(hasSkuVal)
var hasSkuId = ref(true)

  watch: {
    data: {
      deep: true,
      immediate: true,
      handler (salesAttrs) {
        emit('update:modelValue', salesAttrs)
        // 已选属性值，隐藏错误提示
        if (data?.some((item) => item.id && item.leaf?.length > 0)) {
          hasSkuValue = true
        }
        // 已选属性，隐藏错误提示
        if (data?.some((item) => item.id)) {
          hasSkuId = true
        }
      }
    },

    skuTree (skuTree) {
      skuTreeData = skuTree
    },

    categoryId () {
      if (!categoryId) {
        data.splice(0, data.length)
      }
    }
  },

onBeforeMount(() => {
  const { onFetchGroup } = this
  if (typeof (onFetchGroup) == 'function') {
    onFetchGroup().then(skuTree => {
      skuTreeData = skuTree
    })
  }
})


const onChangeHasImg  = (data) => {
  hasSkuImg = data
}

const addSku  = () => {
  hasSkuValue = true
  hasSkuId = true
  if (!categoryId) {
    warning = true
    return
  }
  // 未选属性值，提示用户并禁继续选属性
  if (data?.some((item) => item.id && item.leaf && !item.leaf.length)) {
    hasSkuValue = false
    return
  }
  // 未选已添加属性，提示用户并禁止继续加选属性
  if (data?.some((item) => !item.id)) {
    hasSkuId = false
    return
  }
  data.push({
    leaf: []
  })
}

const rebuildSku  = (sku, index) => {
  const { skuTreeData, optionValue, optionText } = this
  if (
    data.some(
      (item, idx) => item[optionText] === sku[optionText] && index !== idx
    )
  ) {
    ElMessage({
      message: '规格名不能重复',
      duration: 800
    })
    return false
  }
  data[index] = Object.assign({}, sku)

  emit('on-change', data)
}

const handleSkuRemove  = (sku, index) => {
  const { data, skuTreeData, optionValue } = this
  data.splice(index, 1)

  if (index === 0 && data.length > 0) {
    data[0].leaf.map((item) => {
      item.is_show = sku?.leaf[0].is_show
    })
    // hasSkuImg = false
  }
  // 属性选择列表中不存在被删除的已选属性
  if (!skuTreeData?.some(item => item[optionValue] === sku[optionValue])) {
    // 被删除的已选属性id为数字类型（即非手动创建的属性）
    if (typeof (sku[optionValue]) === 'number') {
      // 删除某个已选属性后，往属性选项列表中增加被删除的已选属性
      skuTreeData.push(sku)
    }
  }
  // data[index].leaf[0].is_show = false
  emit('on-change', data)
}

</script>

<style lang="scss">
.ease-sku {
  .add-btn.el-button {
    border-color: #02a1e9;
    color: #02a1e9;
  }
  .warning {
    display: inline-block;
    margin-left: 15px;
    color: #e43130;
    font-size: 13px;
  }
}
</style>
