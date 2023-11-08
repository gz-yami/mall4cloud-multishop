<template>
  <div
    :class="{ hidden: hidden }"
    class="pagination-container"
  >
    <el-pagination
      v-model:current-page="currentPage"
      v-model:page-size="pageSize"
      :background="background"
      :layout="layout"
      :page-sizes="pageSizes"
      :total="total"
      v-bind="$attrs"
      @size-change="onPageSizeChange"
      @current-change="onPageChange"
    />
  </div>
</template>

<script setup>
import { scrollTo } from '@/utils/scroll-to'



const props = defineProps({
  total: {
    required: true,
    type: Number
  },
  page: {
    type: Number,
    default: 1
  },
  limit: {
    type: Number,
    default: 20
  },
  pageSizes: {
    type: Array,
    default () {
      return [limit, 20, 30, 50]
    }
  },
  layout: {
    type: String,
    default: 'total, sizes, prev, pager, next, jumper'
  },
  background: {
    type: Boolean,
    default: true
  },
  autoScroll: {
    type: Boolean,
    default: true
  },
  hidden: {
    type: Boolean,
    default: false
  }
})
  emits: ['update:page', 'update:limit', 'pagination', 'pagination'],

  computed: {
    currentPage: {
      get () {
        return page
      },
      set (val) {
        emit('update:page', val)
      }
    },
    pageSize: {
      get () {
        return limit
      },
      set (val) {
        emit('update:limit', val)
      }
    }
  },


const onPageSizeChange  = (val) => {
  emit('pagination', { pageNum: currentPage, pageSize: val })
  if (autoScroll) {
    scrollTo(0, 800)
  }
}
const onPageChange  = (val) => {
  emit('pagination', { pageNum: val, pageSize: page })
  if (autoScroll) {
    scrollTo(0, 800)
  }
}

</script>

<style scoped>
.pagination-container {
  background: #fff;
  padding: 10px 16px 30px;
  text-align: right;
}
.pagination-container.hidden {
  display: none;
}

.el-pagination {
  font-weight: normal;
}
</style>
