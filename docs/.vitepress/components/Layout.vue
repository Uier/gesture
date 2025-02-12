<template>
  <div ref="main" class="theme" :class="pageClasses">
    <NavBar v-if="showNavbar" @toggle="toggleSidebar" />

    <SideBar :open="openSideBar">
      <template #sidebar-top>
        <slot name="sidebar-top" />
      </template>
      <template #sidebar-bottom>
        <slot name="sidebar-bottom" />
      </template>
    </SideBar>

    <div class="sidebar-mask" @click="toggleSidebar(false)" />

    <Content v-if="isCustomLayout" />

    <Home v-else-if="enableHome">
      <template #hero>
        <slot name="home-hero" />
      </template>
      <template #features>
        <slot name="home-features" />
      </template>
      <template #footer>
        <slot name="home-footer" />
      </template>
    </Home>

    <Page v-else>
      <template #top>
        <slot name="page-top" />
      </template>
      <template #bottom>
        <slot name="page-bottom" />
      </template>
    </Page>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, defineAsyncComponent, provide } from 'vue'
import { useRoute, useSiteData, useSiteDataByRoute } from 'vitepress'
import type { DefaultTheme } from 'vitepress/dist/client/theme-default/config.js'

// Components
import NavBar from 'vitepress/dist/client/theme-default/components/NavBar.vue'
import SideBar from 'vitepress/dist/client/theme-default/components/SideBar.vue'
import Page from 'vitepress/dist/client/theme-default/components/Page.vue'

const Home = defineAsyncComponent(() => import('./Home.vue'))

// Generic State
const route = useRoute()
const siteData = useSiteData<DefaultTheme.Config>()
const siteRouteData = useSiteDataByRoute()

// Custom Layout
const isCustomLayout = computed(() => !!route.data.frontmatter.customLayout)
// home
const enableHome = computed(() => !!route.data.frontmatter.home)

// Inject main
const main = ref()
const interacting = ref(false)
provide('main', main)
provide('interacting', interacting)

// Navbar
const showNavbar = computed(() => {
  const { themeConfig } = siteRouteData.value
  const { frontmatter } = route.data
  if (frontmatter.navbar === false || themeConfig.navbar === false) {
    return false
  }
  return (
    siteData.value.title ||
    themeConfig.logo ||
    themeConfig.repo ||
    themeConfig.nav
  )
})

// Sidebar
const openSideBar = ref(false)

const showSidebar = computed(() => {
  const { frontmatter } = route.data
  const { themeConfig } = siteRouteData.value
  return (
    !frontmatter.home &&
    frontmatter.sidebar !== false &&
    ((typeof themeConfig.sidebar === 'object' &&
      Object.keys(themeConfig.sidebar).length != 0) ||
      (Array.isArray(themeConfig.sidebar) && themeConfig.sidebar.length != 0))
  )
})

const toggleSidebar = (to?: boolean) => {
  openSideBar.value = typeof to === 'boolean' ? to : !openSideBar.value
}

const hideSidebar = toggleSidebar.bind(null, false)
// Close the sidebar when navigating to a different location
watch(route, hideSidebar)
// TODO: route only changes when the pathname changes
// Listening to hashchange does nothing because it's prevented in router

// Page Classes
const pageClasses = computed(() => {
  return [
    {
      'no-navbar': !showNavbar.value,
      'sidebar-open': openSideBar.value,
      'no-sidebar': !showSidebar.value,
      interacting: interacting.value,
    },
  ]
})

watch(interacting, (newVal) => {
  const body = document.querySelector('body')
  const html = document.querySelector('html')

  if (newVal) {
    html.classList.add('interacting')
    body.classList.add('interacting')
    return
  }

  html.classList.remove('interacting')
  body.classList.remove('interacting')
})
</script>

<style>
#ads-container {
  margin: 0 auto;
}

@media (min-width: 420px) {
  #ads-container {
    position: relative;
    right: 0;
    float: right;
    margin: -8px -8px 24px 24px;
    width: 146px;
  }
}

@media (max-width: 420px) {
  #ads-container {
    /* Avoid layout shift */
    height: 105px;
  }
}

@media (min-width: 1400px) {
  #ads-container {
    position: fixed;
    right: 8px;
    bottom: 8px;
  }
}

.demo-box {
  margin-top: 1rem;
  height: 320px;
  width: 100%;
  background-color: #f5f3ff;
  border-radius: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.demo-element {
  width: 6rem;
  height: 6rem;
  background-color: #8b5cf6;
  border-radius: 16px;
}

.interacting {
  overflow: hidden;
}
</style>
