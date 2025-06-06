<script setup lang="ts">
import useMenuStore from '@/store/modules/menu'
import useSettingsStore from '@/store/modules/settings'
import { compile } from 'path-to-regexp'
import Breadcrumb from '../../../Breadcrumb/index.vue'
import BreadcrumbItem from '../../../Breadcrumb/item.vue'

defineOptions({
  name: 'Breadcrumb',
})

const route = useRoute()

const settingsStore = useSettingsStore()
const menuStore = useMenuStore()

const { generateI18nTitle } = useMenu()

// 面包屑备份
let breadcrumbListBackup: any = []
const breadcrumbList = computed(() => {
  // 如果是刷新页面，则返回备份的面包屑，效果相当于刷新页面时面包屑不闪烁
  if (route.name === 'reload') {
    return breadcrumbListBackup
  }
  const list = []
  // 是否展示主页
  if (settingsStore.settings.home.enable) {
    list.push({
      path: settingsStore.settings.home.fullPath,
      title: settingsStore.settings.home.title,
    })
  }
  // 是否展示主导航
  if (route.fullPath !== settingsStore.settings.home.fullPath && settingsStore.settings.breadcrumb.enableMainMenu && !['single'].includes(settingsStore.settings.menu.mode)) {
    const index = menuStore.allMenus.findIndex(item => item.children.some(r => route.fullPath.indexOf(`${r.path}/`) === 0 || route.fullPath === r.path))
    menuStore.allMenus[index]?.meta && list.push({
      path: '',
      title: menuStore.allMenus[index].meta?.title,
    })
  }
  route.matched.forEach((item) => {
    if (item.meta?.breadcrumb !== false) {
      list.push({
        path: item.path,
        title: item.meta?.title,
      })
    }
  })
  const findItem = settingsStore.customTitleList.find(item => item.fullPath === route.fullPath)
  if (findItem && list.length > 0) {
    list[list.length - 1].title = findItem.title
  }
  breadcrumbListBackup = list
  return list
})

function pathCompile(path: string) {
  const toPath = compile(path)
  return toPath(route.params)
}
</script>

<template>
  <Breadcrumb
    v-if="settingsStore.mode === 'pc' && settingsStore.settings.app.routeBaseOn !== 'filesystem'" class="breadcrumb whitespace-nowrap px-2" :class="{
      [`breadcrumb-${settingsStore.settings.breadcrumb.style}`]: settingsStore.settings.breadcrumb.style !== '',
    }"
  >
    <TransitionGroup name="breadcrumb">
      <BreadcrumbItem v-for="(item, index) in breadcrumbList" :key="`${index}_${item.path}_${item.title}`" :to="index < breadcrumbList.length - 1 && item.path !== '' ? pathCompile(item.path) : ''">
        {{ generateI18nTitle(item.title) }}
      </BreadcrumbItem>
    </TransitionGroup>
  </Breadcrumb>
</template>

<style scoped>
.breadcrumb {
  &.breadcrumb-modern {
    :deep(.breadcrumb-item) {
      .text {
        --uno: bg-muted;

        padding: 6px 16px;
        clip-path: polygon(0 0, calc(100% - 8px) 0, 100% 50%, calc(100% - 8px) 100%, 0 100%, 8px 50%);

        [dir="rtl"] & {
          clip-path: polygon(8px 0, 100% 0, calc(100% - 8px) 50%, 100% 100%, 8px 100%, 0 50%);
        }

        &.is-link:hover {
          --uno: bg-muted;
        }
      }

      &:first-child .text {
        padding-inline-start: 12px;
        border-radius: 6px 0 0 6px;
        clip-path: polygon(0 0, calc(100% - 8px) 0, 100% 50%, calc(100% - 8px) 100%, 0 100%);

        [dir="rtl"] & {
          border-radius: 0 6px 6px 0;
          clip-path: polygon(8px 0, 100% 0, 100% 100%, 8px 100%, 0 50%);
        }
      }

      &:last-child:not(:first-child) .text {
        --uno: bg-muted;

        padding-inline-end: 12px;
        border-radius: 0 6px 6px 0;
        clip-path: polygon(0 0, 100% 0, 100% 100%, 0 100%, 8px 50%);

        [dir="rtl"] & {
          border-radius: 6px 0 0 6px;
          clip-path: polygon(0 0, 100% 0, calc(100% - 8px) 50%, 100% 100%, 0 100%);
        }
      }

      .separator {
        display: none;
      }
    }
  }
}

/* 面包屑动画 */
.breadcrumb-enter-active {
  transition: transform 0.3s, opacity 0.3s;
}

.breadcrumb-enter-from {
  opacity: 0;
  transform: translateX(30px) skewX(-50deg);
}
</style>
