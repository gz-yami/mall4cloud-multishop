<script setup>
import { h } from 'vue'

onMounted(() => {
  console.log(111)
  useRouter().replace({ path: '/order/order' })
})
  render: function () {
    return h() // avoid warning message
  }

</script>
