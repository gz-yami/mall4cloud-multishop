<template>
  <div class="category-selector">
    <el-dialog
      v-model:visible="visible"
      :title="$t('components.selector.categorySelector')"
      :append-to-body="visible"
      width="1000px"
      @close="onClose"
    >
      <div class="prod-category clearfix">
        <div class="category">
          <!-- 分类 -->
          <div
            v-if="firstCategorys.dataList.length"
            class="category-box"
          >
            <el-input
              v-model="firstCategorys.name"
              :placeholder="$t('components.selector.chooseProdCateg')"
              :disabled="true"
            />
            <ul class="category-list">
              <li
                v-for="(item,index) in firstCategorys.dataList"
                :key="item.categoryId"
                class="category-item"
                :class="item.categoryId==firstCategorys.id?'active':''"
                @click="selectFirstCategorys(item.categoryId, index)"
              >
                {{ item.name }}
              </li>
            </ul>
          </div>
          <!-- 分类 -->
          <div
            v-if="secondCategorys.dataList.length && firstCategorys.id!=0"
            class="category-box"
          >
            <el-input
              v-model="secondCategorys.name"
              :placeholder="$t('components.selector.chooseProdCateg')"
              :disabled="true"
            />
            <ul class="category-list">
              <li
                v-for="(item,index) in secondCategorys.dataList"
                :key="item.categoryId"
                class="category-item"
                :class="isCreateCategory?'prohibit-sel':item.categoryId==secondCategorys.id?'active':''"
                @click="selectSecondCategorys(item.categoryId,index)"
              >
                {{ item.name }}
              </li>
            </ul>
          </div>
          <!-- 分类 -->
          <div
            v-if="showthreeCategorys && threeCategorys.dataList.length > 0 && secondCategorys.id!=0"
            class="category-box"
          >
            <el-input
              v-model="threeCategorys.name"
              :placeholder="$t('components.selector.chooseProdCateg')"
              :disabled="true"
            />
            <ul class="category-list">
              <li
                v-for="(item,index) in threeCategorys.dataList"
                :key="item.categoryId"
                class="category-item"
                :class="[isCreateCategory?'prohibit-sel':item.categoryId==threeCategorys.id?'active':'']"
                @click="selectThreeCategorys(item.categoryId,index)"
              >
                {{ item.name }}
              </li>
            </ul>
          </div>
        </div>
        <!-- 当前选择 -->
        <div class="current-selected">
          <span class="blod">{{ $t("components.selector.currCho") }}：</span>
          <span class="select-item">{{ firstCategorys.name }}</span>
          <span
            v-if="!isCreateCategory && secondCategorys.id"
            class="select-item"
          >&nbsp;>&nbsp;&nbsp;{{ secondCategorys.name }}</span>
          <span
            v-if="showthreeCategorys && threeCategorys.id"
            class="select-item"
          >&nbsp;>&nbsp;&nbsp;{{ threeCategorys.name }}</span>
        </div>
        <!-- 确认 -->
        <div class="read-rule">
          <div
            class="read-rule-txt"
            :class="buttonHighlight?'todo':''"
            @click="optionsConfirm"
          >
            {{ $t("components.selector.haveReadFol") }}
          </div>
        </div>
      </div>
    </el-dialog>
  </div>
</template>

<script setup>
import * as api from '@/api/product/category'

const props = defineProps({
  // categoryList: Array
})
  emits: ['getCategorySelected'],


var visible = ref(false)
var allDataList = ref([])
// 第一个分类
var firstCategorys = reactive({
  id: 0,
  name: '',
  dataList: []
})
// 第二个分类
var secondCategorys = reactive({
  id: 0,
  name: '',
  dataList: []
})
// 第三个分类
var threeCategorys = reactive({
  id: 0,
  name: '',
  dataList: []
})
var parentId = ref(0)
var isCreateCategory = ref(false) // 是否创建(店铺)分类选择
var showthreeCategorys = ref(false) // 是否显示第三级分类
var buttonHighlight = ref(false) // 按钮高亮


/**
 * 初始化
 */
const init  = (type, key) => {
  visible = true
  console.log('分类选择器key:', key)
  if (key === 'platform') {
    api.platformCategoryPage().then(data => {
      allDataList = data
      firstCategorys.dataList = allDataList.filter(item => item.level === 0)
    })
    showthreeCategorys = true // 平台分类，最高三级
  } else {
    api.shopCategoryPage().then(data => {
      allDataList = data
      firstCategorys.dataList = allDataList.filter(item => item.level === 0)
    })
    showthreeCategorys = false // 店铺分类，最高二级
  }

  isCreateCategory = !!(type && type === 1)
}
const show  = () => {
  visible = true
}
const hide  = () => {
  visible = false
}

// 选中第一个分类
const selectFirstCategorys  = (categoryId, index) => {
  secondCategorys.dataList = allDataList.filter(item => item.parentId === categoryId)
  firstCategorys.name = firstCategorys.dataList[index].name
  parentId = firstCategorys.id = categoryId
  secondCategorys.id = 0
  if (showthreeCategorys) threeCategorys.id = 0

  if (isCreateCategory || (!isCreateCategory && secondCategorys.dataList.length == 0)) { // 创建分类
    buttonHighlight = true
  } else {
    buttonHighlight = false
  }
}
// 选中第二个分类
const selectSecondCategorys  = (categoryId, index) => {
  if (isCreateCategory) {
    return
  }
  threeCategorys.dataList = allDataList.filter(item => item.parentId === categoryId)
  parentId = secondCategorys.id = categoryId
  secondCategorys.name = secondCategorys.dataList[index].name
  if (showthreeCategorys) threeCategorys.id = 0

  if (!isCreateCategory && !buttonHighlight && !showthreeCategorys) { // 非创建分类&&店铺分类
    buttonHighlight = true
  } else {
    buttonHighlight = false
  }
  console.log('buttonHighlight：', buttonHighlight)
}
// 选中第三个分类
const selectThreeCategorys  = (categoryId, index) => {
  if (isCreateCategory) {
    return
  }
  parentId = threeCategorys.id = categoryId
  threeCategorys.name = threeCategorys.dataList[index].name
  buttonHighlight = true
}
// 新增 / 修改
const optionsConfirm  = () => {
  // 平台分类 & 没有第三级分类
  if (showthreeCategorys && !threeCategorys.id) {
    return
  }
  // 店铺分类 & 创建分类 & 没有第一级分类
  if (!showthreeCategorys && isCreateCategory && !firstCategorys.id) {
    return
  }
  // 店铺分类 & 非创建分类 & 没有第二级分类
  if (!showthreeCategorys && !isCreateCategory && secondCategorys.dataList.length > 0 && !secondCategorys.id) {
    return
  }
  // $store.commit('common/removeMainActiveTab')
  const selectedCategorys = []
  if (firstCategorys.id) {
    selectedCategorys.push(firstCategorys.name)
  }
  if (!isCreateCategory && secondCategorys.id) {
    selectedCategorys.push(secondCategorys.name)
  }
  if (showthreeCategorys && !isCreateCategory && threeCategorys.id) {
    selectedCategorys.push(threeCategorys.name)
  }
  emit('getCategorySelected', selectedCategorys, parentId)
}

// 关闭
const onClose  = () => {
  allDataList = []
  // 第一个分类
  firstCategorys = {
    id: 0,
    name: '',
    dataList: []
  },
  // 第二个分类
  secondCategorys = {
    id: 0,
    name: '',
    dataList: []
  },
  // 第三个分类
  threeCategorys = {
    id: 0,
    name: '',
    dataList: []
  }
}


</script>

<style lang="scss">
.prod-category {
  .clearfix::after {
    display: block;
    content: "";
    visibility: hidden;
    height: 0;
    clear: both;
  }
  width: 90%;
  margin: 0 auto;
  ul,
  li {
    margin: 0;
    padding: 0;
    list-style: none;
  }
  .tips {
    // display: inline-block;
    float: right;
    padding: 10px 12px;
    background: #02a1e9;
    color: #fff;
    font-size: 13px;
    margin: 0;
  }
  .category-box {
    box-sizing: border-box;
    width: 32%;
    box-sizing: border-box;
    display: inline-block;
    border: 1px solid #e4e4e4;
    padding: 20px;
    background: #fff;
    box-shadow: 0px 5px 8px -4px #e2e2e2;
  }
  .int {
    width: 100%;
    height: 2.5em;
    line-height: 2.5em;
    text-indent: 1em;
    border: 1px solid #e2e2e2;
    border-radius: 5px;
    outline: none;
    /* box-shadow: 0px 5px 8px -4px #e2e2e2; */
  }
  input.int::placeholder {
    color: #bdbdbd;
  }
  .category-list {
    margin-top: 10px;
    height: 250px;
    margin-left: 10px;
    overflow-y: auto;
  }
  .category-item {
    padding: 6px;
    cursor: pointer;
  }
  .category-item:hover,
  .category-item.active {
    // background: #e9f9ff;
    background: #f5f5f5;
    border-radius: 3px;
  }
  .current-selected {
    margin-top: 20px;
    border: 2px solid #fedbab;
    padding: 6px;
    background: #fffaf2;
    line-height: 1.5em;
  }
  .read-rule {
    width: 100%;
    margin-top: 20px;
  }
  .read-rule-txt {
    text-align: center;
    color: #fff;
    background: #999999;
    line-height: 3em;
    width: 50%;
    margin: 0 auto;
  }
  .rule {
    border: 1px solid #b9b9b9;
    padding: 0 10px;
    font-size: 13px;
    color: #777;
    margin-top: 20px;
  }
  .notice {
    line-height: 3em;
    border-bottom: 1px dashed #b9b9b9;
    margin: 0;
  }
  .blod {
    font-weight: bold;
  }
  .rule-tit {
    text-align: center;
  }
  .rule-txt {
    line-height: 1.5em;
    padding-bottom: 30px;
  }
  .todo {
    background: #02a1e9;
    cursor: pointer;
  }
  .prohibit-sel {
    cursor: not-allowed;
  }
}
</style>
