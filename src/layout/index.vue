<template>
  <div
    :class="classObj"
    class="app-wrapper"
  >
    <div
      v-if="device==='mobile'&&sidebar.opened"
      class="drawer-bg"
      @click="handleClickOutside"
    />
    <div :class="{'fixed-header':fixedHeader}">
      <navbar />
    </div>
    <div class="main-container">
      <sidebar
        class="sidebar-container"
        :class="{'fixed-sidebar':isCoverHead===true}"
      />
      <app-main />
    </div>
  </div>
</template>

<script setup>
import { AppMain, Navbar, Sidebar } from './components'
import ResizeMixin from './mixin/ResizeHandler'
import { mapState } from 'vuex'



  mixins: [ResizeMixin],

var isCoverHead = ref(false)
  computed: {
    ...mapState({
      sidebar: state => state.app.sidebar,
      device: state => state.app.device,
      showSettings: state => state.settings.showSettings,
      needTagsView: state => state.settings.tagsView,
      fixedHeader: state => state.settings.fixedHeader
    }),
    classObj () {
      return {
        withoutAnimation: sidebar.withoutAnimation,
        mobile: device === 'mobile'
      }
    }
  },
onMounted(() => {
  window.addEventListener('scroll', () => {
    const scrollTop = document.documentElement.scrollTop || document.body.scrollTop || window.pageYOffset
    if (scrollTop > 80) {
      isCoverHead = true
    } else {
      isCoverHead = false
    }
  })
})

const handleClickOutside  = () => {
  $store.dispatch('app/closeSideBar', { withoutAnimation: false })
}

</script>

<style lang="scss" scoped>
  @import "~@/styles/mixin.scss";
  @import "~@/styles/variables.scss";

  .app-wrapper {
    @include clearfix;
    position: relative;
    width: 100%;
    height: 100%;
    background: #f9f9f9;

    &.mobile.openSidebar {
      position: fixed;
      top: 0;
    }
  }

  .drawer-bg {
    background: #000;
    opacity: 0.3;
    width: 100%;
    top: 0;
    height: 100%;
    position: absolute;
    z-index: 999;
  }

  .mobile .fixed-header {
    width: 100%;
  }
</style>
