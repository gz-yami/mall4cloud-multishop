<template>
  <el-dialog
    v-if="visible"
    v-model:visible="visible"
    :title="$t('order.order.orderShipping')"
    :close-on-click-modal="false"
    width="50%"
  >
    <!-- :rules="dataRule" -->
    <!-- native modifier has been removed, please confirm whether the function has been affected  -->
    <el-form
      ref="dataFormRef"
      :model="dataForm"
      label-width="100px"
      @keyup.enter="onSubmit()"
    >
      <div class="detail-cont">
        <div class="detail01">
          <div class="text-width">
            <el-form-item :label="$t('order.order.waitForDelivery')">
              <span>{{ $t("order.order.delType") }}:{{
                $t("order.order.expressDelivery")
              }}</span>
            </el-form-item>
            <el-form-item label>
              <span>{{ $t("constant.consignee") }}：{{
                dataForm.orderAddr.consignee
              }}</span>
            </el-form-item>
            <el-form-item label>
              <span>{{ $t("constant.mobilePhone") }}：{{
                dataForm.orderAddr.mobile
              }}</span>
            </el-form-item>
            <el-form-item label>
              <span>{{ $t("constant.deliveryAddr") }}：{{
                dataForm.orderAddr.province
              }}{{ dataForm.orderAddr.city
              }}{{ dataForm.orderAddr.area
              }}{{ dataForm.orderAddr.addr }}</span>
            </el-form-item>
          </div>
        </div>
      </div>
      <el-form-item :label="$t('order.order.deliveryMethod') + ':'">
        <el-radio-group
          v-model="dataForm.deliveryType"
          @change="clear()"
        >
          <el-radio
            :label="1"
            disabled
          >
            {{
              $t("order.order.selfConOrd")
            }}
          </el-radio>
          <el-radio :label="3">
            {{
              $t("order.order.noNeedRequired")
            }}
          </el-radio>
          <el-radio
            :label="4"
            disabled
          >
            {{
              $t("order.order.sameCityDelivery")
            }}
          </el-radio>
        </el-radio-group>
      </el-form-item>
    </el-form>
    <template #footer>
      <span>
        <el-button
          type="primary"
          @click="visible = false"
        >{{
          $t("table.cancel")
        }}</el-button>
        <el-button
          type="primary"
          @click="submitProds()"
        >{{
          $t("table.confirm")
        }}</el-button>
      </span>
    </template>
  </el-dialog>
</template>

<script setup>
import * as deliveryCompanyApi from '@/api/delivery/delivery-company'
import * as orderApi from '@/api/order/order'


  emits: ['refreshOrderDeliveryUpdate'],


var visible = ref(false)
var dataForm = reactive({
  deliveryCompanyId: '',
  orderAddr: {},
  deliveryNo: 0,
  names: [],
  orderId: 0,
  deliveryType: 1
})
var dataList = ref([])
var selectOrderItems = ref([])
var isSubmit = ref(false)
var orderItems = ref([])
var orderId = ref(0)
let order = null

onMounted(() => { },

methods: {
  processingStr (str) {
    // str = str.replace(/\u200B/g,'');
    // return str
  },

  /**
   * 获取数据列表
   */
  init (order) {
    isSubmit = false
    dataForm.orderAddr = {}
    visible = true
    orderId = order.orderId
    clear()
    // 修改
    orderApi.getOrderItemAndAddress(order.orderId).then(data => {
      dataForm.orderAddr = data.orderAddr
      dataList = data.orderItems
      dataForm.deliveryType = parseInt(data.deliveryType)
      console.log(dataList)
      dataList.forEach(element => {
        element.changeNum = element.count
        selectOrderItems.push({
          changeNum: element.count,
          orderItemId: element.orderItemId,
          pic: element.pic,
          spuName: element.spuName
        })
      })
    })
    // getDeliveryList()
  },
  getDeliveryList () {
    deliveryCompanyApi.list().then((data) => {
      dataForm.names = data
    })
  },
  clear () {
    dataForm.deliveryNo = 0
    dataForm.deliveryCompanyId = ''
  },
  /**
   * 确定事件
   */
  submitProds () {
    if (isSubmit) {
      return
    }
    isSubmit = true
    // let param = dataForm
    orderApi.delivery({
      orderId: orderId,
      selectOrderItems: selectOrderItems,
      deliveryType: dataForm.deliveryType
    }).then((data) => {
      ElMessage({
        message: $t('table.actionSuccess'),
        type: 'success',
        duration: 1500,
        onClose: () => {
          visible = false
          setTimeout(() => {
            emit('refreshOrderDeliveryUpdate')
          }, 1000)
        }
      })
    }).catch(({ e }) => {
      isSubmit = false
    })
  },
  errorMsg (message) {
    ElMessage({
      message,
      type: 'error',
      duration: 1500
    })
  }
})

</script>

<style>
.el-input__inner {
  border: 1px solid #dcdfe6 !important;
}
</style>
<style scoped>
.mod-order-orderInfo {
  height: 100%;
  width: 100%;
  font: 14px Arial, "PingFang SC", "Hiragino Sans GB", STHeiti,
    "Microsoft YaHei", "WenQuanYi Micro Hei", sans-serif;
  color: #495060;
}
.element.style {
  border-top: 1px solid #eeeeee;
  border-right: 1px solid #eeeeee;
  border-bottom: 1px solid #eeeeee;
}
.prods-select-body {
  height: auto;
  overflow: auto;
  border-top: 1px solid #eeeeee;
  border-right: 1px solid #eeeeee;
  border-bottom: 1px solid #eeeeee;
}
.mod-order-order .contprod-:last-child {
  border-bottom: 0;
  border-left: 0;
}
.mod-order-order .prod-cont.prod-bottom,
.mod-order-order .prod-cont {
  border-bottom: 1px solid #dddee1;
  border-left: 1px solid #dddee1;
}
</style>
