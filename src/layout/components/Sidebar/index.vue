<template>
  <div>
    <el-scrollbar wrap-class="scrollbar-wrapper">
      <el-menu
        :default-openeds="openeds"
        :default-active="activeMenu"
        :background-color="variables.menuBg"
        :text-color="variables.menuText"
        :unique-opened="false"
        :active-text-color="variables.menuActiveText"
      >
        <sidebar-item
          v-for="route in permission_routes"
          :key="route.path"
          :item="route"
          :base-path="route.path"
        />
      </el-menu>
    </el-scrollbar>
  </div>
</template>

<script setup>
import { mapGetters } from 'vuex'
import SidebarItem from './SidebarItem'
import variables from '@/styles/variables.scss'


  computed: {
    ...mapGetters([
      'permission_routes'
    ]),
    activeMenu () {
      const route = useRoute()
      const { meta, path } = route
      // if set path, the sidebar will highlight the path you set
      if (meta.activeMenu) {
        return meta.activeMenu
      }
      return path
    },
    // 所有sub-menu的index的数组(用来默认展开所有菜单)
    openeds () {
      const permission_routes = permission_routes
      const ids = []
      permission_routes.forEach(route => {
        ids.push(route.path)
      })
      return ids
    },
    variables () {
      return variables
    }
  }

</script>
