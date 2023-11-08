<script>
import { h } from 'vue'
export default {
  created () {
    console.log(111)
    this.$router.replace({ path: '/order/order' })
  },
  render: function () {
    return h() // avoid warning message
  }
}
</script>
