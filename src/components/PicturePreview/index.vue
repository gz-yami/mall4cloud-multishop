<template>
  <el-dialog
    v-model:visible="visible"
    :close-on-click-modal="false"
    top="10vh"
    :append-to-body="true"
    :lock-scroll="true"
    class="img-preview"
    width="650px"
  >
    <div class="img-box">
      <img
        :src="(imgUrl).indexOf('http')===-1 ? resourcesUrl + imgUrl : imgUrl"
        class="img"
      >
    </div>
  </el-dialog>
</template>

<script setup>



var imgUrl = ref('')
var visible = ref(false)
var resourcesUrl = import.meta.env.VITE_APP_RESOURCES_URL


const init  = (imgUrl) => {
  imgUrl = imgUrl
  visible = true
}


</script>

<style lang="scss" scoped>
.img-preview {
  .img-box {
    display: block;
    width: 100%;
    max-height: 700px;
    overflow: auto;
  }
  .img-box::-webkit-scrollbar {
    display: none;
  }
  .img {
    display: block;
    width: 100%;
    height: auto;
  }
}
</style>
