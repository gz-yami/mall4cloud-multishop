<template>
  <section class="app-main">
    <router-view
      :key="key"
      v-slot="{ Component }"
    >
      <transition
        name="fade-transform"
        mode="out-in"
      >
        <keep-alive :include="cachedViews">
          <component :is="Component" />
        </keep-alive>
      </transition>
    </router-view>
  </section>
</template>

<script setup>


  computed: {
    cachedViews () {
      return $store.state.tagsView.cachedViews
    },
    key () {
      return useRoute().path
    }
  }

</script>

<style lang="scss" scoped>
.app-main {
  position: relative;
  width: 1034px;
  background: #FFFFFF;
  overflow: hidden;
  margin-left: 20px;
}

.hasTagsView {
  .app-main {
    /* 134 = navbar + tags-view + margin-bottom = 94 + 20 + 20 */
    min-height: calc(100vh - 134px);
  }
}
</style>

<style lang="scss">
// fix css style bug in open el-dialog
.el-popup-parent--hidden {
  .fixed-header {
    padding-right: 15px;
  }
}
</style>
