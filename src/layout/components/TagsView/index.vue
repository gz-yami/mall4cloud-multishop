<template>
  <div
    id="tags-view-container"
    class="tags-view-container"
  >
    <scroll-pane
      ref="scrollPaneRef"
      class="tags-view-wrapper"
      @scroll="handleScroll"
    >
      <!-- native modifier has been removed, please confirm whether the function has been affected  -->
      <!-- native modifier has been removed, please confirm whether the function has been affected  -->
      <router-link
        v-for="tag in visitedViews"
        ref="tagRef"
        :key="tag.path"
        v-slot="{ navigate }"
        :class="isActive(tag)?'active':''"
        :to="{ path: tag.path, query: tag.query, fullPath: tag.fullPath }"
        class="tags-view-item"
        custom
        @click.middle="!isAffix(tag)?closeSelectedTag(tag):''"
@contextmenu.prevent="openMenu(tag,$event)"
      >
        <span @click="navigate" />{{ generateTitle(tag.title) }}
        <span
          v-if="!isAffix(tag)"
          class="el-icon-close"
          @click.prevent.stop="closeSelectedTag(tag)"
        />
      </router-link>
    </scroll-pane>
    <ul
      v-show="visible"
      :style="{left:left+'px',top:top+'px'}"
      class="contextmenu"
    >
      <li @click="refreshSelectedTag(selectedTag)">
        {{ $t('tagsView.refresh') }}
      </li>
      <li
        v-if="!isAffix(selectedTag)"
        @click="closeSelectedTag(selectedTag)"
      >
        {{ $t('tagsView.close') }}
      </li>
      <li @click="closeOthersTags">
        {{ $t('tagsView.closeOthers') }}
      </li>
      <li @click="closeAllTags(selectedTag)">
        {{ $t('tagsView.closeAll') }}
      </li>
    </ul>
  </div>
</template>

<script setup>
import ScrollPane from './ScrollPane'
import { generateTitle } from '@/utils/i18n'
import path from 'path'



var visible = ref(false)
var top = ref(0)
var left = ref(0)
var selectedTag = reactive({})
var affixTags = ref([])
  computed: {
    visitedViews () {
      return $store.state.tagsView.visitedViews
    },
    routes () {
      return $store.state.permission.routes
    }
  },
  watch: {
    $route () {
      addTags()
      moveToCurrentTag()
    },
    visible (value) {
      if (value) {
        document.body.addEventListener('click', closeMenu)
      } else {
        document.body.removeEventListener('click', closeMenu)
      }
    }
  },
onMounted(() => {
  initTags()
  addTags()
})

generateTitle, // generateTitle by vue-i18n
const isActive  = (route) => {
  return route.path === useRoute().path
}
const isAffix  = (tag) => {
  return tag.meta && tag.meta.affix
}
const filterAffixTags  = (routes, basePath = '/') => {
  let tags = []
  routes.forEach(route => {
    if (route.meta && route.meta.affix) {
      const tagPath = path.resolve(basePath, route.path)
      tags.push({
        fullPath: tagPath,
        path: tagPath,
        name: route.name,
        meta: { ...route.meta }
      })
    }
    if (route.children) {
      const tempTags = filterAffixTags(route.children, route.path)
      if (tempTags.length >= 1) {
        tags = [...tags, ...tempTags]
      }
    }
  })
  return tags
}
const initTags  = () => {
  const affixTags = affixTags = filterAffixTags(routes)
  for (const tag of affixTags) {
    // Must have tag name
    if (tag.name) {
      $store.dispatch('tagsView/addVisitedView', tag)
    }
  }
}
const addTags  = () => {
  const { name } = useRoute()
  if (name) {
    $store.dispatch('tagsView/addView', useRoute())
  }
  return false
}
const moveToCurrentTag  = () => {
  const tags = $refs.tag
  nextTick(() => {
    for (const tag of tags) {
      if (tag.to.path === useRoute().path) {
        scrollPaneRef.value?.moveToTarget(tag)
        // when query is different then update
        if (tag.to.fullPath !== useRoute().fullPath) {
          $store.dispatch('tagsView/updateVisitedView', useRoute())
        }
        break
      }
    }
  })
}
const refreshSelectedTag  = (view) => {
  $store.dispatch('tagsView/delCachedView', view).then(() => {
    const { fullPath } = view
    nextTick(() => {
      useRouter().replace({
        path: '/redirect' + fullPath
      })
    })
  })
}
const closeSelectedTag  = (view) => {
  $store.dispatch('tagsView/delView', view).then(({ visitedViews }) => {
    if (isActive(view)) {
      toLastView(visitedViews, view)
    }
  })
}
const closeOthersTags  = () => {
  useRouter().push(selectedTag)
  $store.dispatch('tagsView/delOthersViews', selectedTag).then(() => {
    moveToCurrentTag()
  })
}
const closeAllTags  = (view) => {
  $store.dispatch('tagsView/delAllViews').then(({ visitedViews }) => {
    if (affixTags.some(tag => tag.path === view.path)) {
      return
    }
    toLastView(visitedViews, view)
  })
}
const toLastView  = (visitedViews, view) => {
  const latestView = visitedViews.slice(-1)[0]
  if (latestView) {
    useRouter().push(latestView.fullPath)
  } else {
    // now the default is to redirect to the home page if there is no tags-view,
    // you can adjust it according to your needs.
    if (view.name === 'order/order') {
      // to reload home page
      useRouter().replace({ path: '/redirect' + view.fullPath })
    } else {
      useRouter().push('/')
    }
  }
}
const openMenu  = (tag, e) => {
  const menuMinWidth = 105
  const offsetLeft = $el.getBoundingClientRect().left // container margin left
  const offsetWidth = $el.offsetWidth // container width
  const maxLeft = offsetWidth - menuMinWidth // left boundary
  const left = e.clientX - offsetLeft + 15 // 15: margin right

  if (left > maxLeft) {
    left = maxLeft
  } else {
    left = left
  }

  top = e.clientY
  visible = true
  selectedTag = tag
}
const closeMenu  = () => {
  visible = false
}
const handleScroll  = () => {
  closeMenu()
}

</script>

<style lang="scss" scoped>
.tags-view-container {
  height: 34px;
  width: 100%;
  background: #fff;
  border-bottom: 1px solid #d8dce5;
  box-shadow: 0 1px 3px 0 rgba(0, 0, 0, .12), 0 0 3px 0 rgba(0, 0, 0, .04);
  .tags-view-wrapper {
    .tags-view-item {
      display: inline-block;
      position: relative;
      cursor: pointer;
      height: 26px;
      line-height: 26px;
      border: 1px solid #d8dce5;
      color: #495060;
      background: #fff;
      padding: 0 8px;
      font-size: 12px;
      margin-left: 5px;
      margin-top: 4px;
      &:first-of-type {
        margin-left: 15px;
      }
      &:last-of-type {
        margin-right: 15px;
      }
      &.active {
        background-color: #42b983;
        color: #fff;
        border-color: #42b983;
        &::before {
          content: '';
          background: #fff;
          display: inline-block;
          width: 8px;
          height: 8px;
          border-radius: 50%;
          position: relative;
          margin-right: 2px;
        }
      }
    }
  }
  .contextmenu {
    margin: 0;
    background: #fff;
    z-index: 3000;
    position: absolute;
    list-style-type: none;
    padding: 5px 0;
    border-radius: 4px;
    font-size: 12px;
    font-weight: 400;
    color: #333;
    box-shadow: 2px 2px 3px 0 rgba(0, 0, 0, .3);
    li {
      margin: 0;
      padding: 7px 16px;
      cursor: pointer;
      &:hover {
        background: #eee;
      }
    }
  }
}
</style>

<style lang="scss">
//reset element css of el-icon-close
.tags-view-wrapper {
  .tags-view-item {
    .el-icon-close {
      width: 16px;
      height: 16px;
      vertical-align: 2px;
      border-radius: 50%;
      text-align: center;
      transition: all .3s cubic-bezier(.645, .045, .355, 1);
      transform-origin: 100% 50%;
      &:before {
        transform: scale(.6);
        display: inline-block;
        vertical-align: -3px;
      }
      &:hover {
        background-color: #b4bccc;
        color: #fff;
      }
    }
  }
}
</style>
