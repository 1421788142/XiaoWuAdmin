<template>
    <div class="w-screen h-screen overflow-hidden bg-slate-500">
        <!-- 移动端菜单 -->
        <div v-if="getConfigState('sysMode') === sysModeEnum.phone">
            <a-drawer
                width="220px"
                v-model:visible="wapMenuVisible"
                placement="left"
                :closable="false"
                class="wap-menu"
            >   
                <div class="!h-screen w-full">
                    <menuVue menuTheme="vertical" class="w-full" />
                </div>
            </a-drawer>
        </div>
        <!-- 页面菜单 -->
        <a-layout class="w-full h-full">
            <!-- 侧边栏 -->
            <a-layout-sider 
                :collapsedWidth="50" 
                v-if="getConfigState('sysMode') === sysModeEnum.web && !getConfigState('isHasFull') && menuType !== 'horizontal'" 
                width="230px" 
                :theme="menuType === 'verticalDark' ? 'dark' : 'light'" 
                v-model:collapsed="isHasCollapsed" 
                :trigger="null"
            >
                <div class="flex flex-col h-screen">
                    <logo :collapsed="isHasCollapsed" :menuType="menuType" v-if="getConfigState('isHasLogo')"/>
                    <menuVue ref="leftMenu" :collapsed="isHasCollapsed" menuTheme="dark" />
                </div>
            </a-layout-sider>
            <!-- 右侧 -->
            <a-layout>
                <!-- 顶部 -->
                <a-layout-header class="md:!px-4 !px-1 !bg-white dark:!bg-[#141414] !h-max">
                    <div class="flex items-center justify-between" v-if="!getConfigState('isHasFull')">
                        <div class="flex items-center" v-if="menuType === 'verticalDark' || menuType === 'verticalLight' || getConfigState('sysMode') === sysModeEnum.phone">
                            <div class="flex items-center" v-if="getConfigState('sysMode') === sysModeEnum.web">
                                <menu-unfold-outlined v-if="isHasCollapsed" class="text-xl" @click="setConfigState('isHasCollapsed',false)"/>
                                <menu-fold-outlined v-else class="text-xl" @click="setConfigState('isHasCollapsed',true)" />
                                <breadcrumb />
                            </div>
                            <div class="flex items-center" v-if="getConfigState('sysMode') === sysModeEnum.phone">
                                <menu-unfold-outlined v-if="wapMenuVisible" class="text-xl" @click="wapMenuVisible = !wapMenuVisible"/>
                                <menu-fold-outlined v-else class="text-xl" @click="wapMenuVisible = !wapMenuVisible" />
                            </div>
                        </div>
                        <div v-if="menuType === 'horizontal' && getConfigState('sysMode') === sysModeEnum.web" class="flex flex-1">
                            <logo :collapsed="isHasCollapsed" :menuType="menuType" v-if="getConfigState('isHasLogo')"/>
                            <menuVue menuTheme="light" />
                        </div>
                        <!-- 混合模式顶部菜单 -->
                        <div v-if="menuType === 'mix' && getConfigState('sysMode') === sysModeEnum.web" class="flex flex-1 max-w-[80%]">
                            <a-menu
                                mode="horizontal"
                                class="!justify-center flex-1 max-w-[100%]"
                                :selectedKeys="selectedKeys"
                            >
                                <a-menu-item @click="toPage(item)" :key="item.path" v-for="item in menuMixList">
                                    <span class="nav-text">{{item.meta.title || '未命名'}}</span>
                                    <template #icon>
                                        <component v-if="item.icon" :is="item.icon"></component>
                                    </template>
                                </a-menu-item>
                            </a-menu>
                        </div>
                        <userInfo />
                    </div>
                    <!-- 历史菜单 -->
                    <navTag v-if="getConfigState('isHasHistory')" @change="(isHasReset)=>{
                        setViewShow(isHasReset)
                    }" />
                </a-layout-header>
                <layoutPage ref="pageContent" />
            </a-layout>
        </a-layout>
        <!-- 系统设置 -->
        <div id="set-up" :class="{'!right-0':!getConfigState('isHasSystem')}" class="set-up">
            <a-button type="primary" class="flex items-center justify-center cursor-pointer" @click="openSet">
                <setting-outlined :style="{fontSize:'20px'}" />
            </a-button>
        </div>
        <systemConfig ref="systemDrawer"/>
    </div>
</template>

<script setup lang="ts">
import { ref, nextTick, onMounted, provide, computed } from 'vue';
import { guide, getFirstMenu } from './index'
import { debounce } from '@/utils/util'

import menuVue from '@/layout/components/sidebar/menu.vue'
import layoutPage from '@/layout/components/page/index.vue'
import navTag from './components/tag/index.vue'
import breadcrumb from './components/breadcrumb/index.vue'
import userInfo from './components/user/index.vue'
import systemConfig from './components/systemConfig/index.vue'
import logo from './components/logo/index.vue'

import { useLayout } from './hooks/useLayout';
import { sysModeEnum } from '@/enums/sys'

import { useRoute, useRouter } from 'vue-router';
const route = useRoute()
const router = useRouter()

const isHasCollapsed = computed(()=>getConfigState('isHasCollapsed'))
const menuType = computed(()=>getConfigState('menuType'))
const {  
    wapMenuVisible,
    crumbsList,
    menuMixList,
    selectedKeys,
    switchDark,
    setupColorblind,
    setupGrey,
    getConfigState,
    setConfigState,
    resetConfig
} = useLayout(route)
let sysConfig = {
    crumbsList,
    switchDark,
    setupColorblind,
    setupGrey,
    getConfigState,
    setConfigState,
    resetConfig
}
provide('sysConfig',sysConfig)

const leftMenu = ref<ComponentRef>()

// 混合菜单跳转菜单
const toPage = (menuItem:menuListType)=>{
    if(menuItem.isFrame && !menuItem.children) return window.open(menuItem.path)
    let firstMenu = getFirstMenu(menuItem.path)
    if(firstMenu.isFrame){
        window.open(firstMenu.path)
        leftMenu.value.setMenuList(menuItem.path)
        return
    } 
    selectedKeys.value = [firstMenu.path,menuItem.path]
    router.push({ path: firstMenu.path || '/' })
}

// 防抖函数监听页面尺寸
const newSize = ()=> {
    /**  width <= 900 系统为手机模式
     *  width > 900 系统为web模式
     */
    if(document.body.clientWidth<=990){
        setConfigState('sysMode',sysModeEnum.phone)
    } else {
        setConfigState('sysMode',sysModeEnum.web)
    }
}

let debounceFn = debounce(newSize,200)
onMounted(()=>{
    nextTick( async()=>{
        await guide(getConfigState('isHasSystem'))
        setConfigState('isHasSystem',true)
    })
    window.onresize = debounceFn
})
newSize()

// 刷新页面
const pageContent = ref(null)
const setViewShow =(setViewShow:boolean)=>{
    pageContent.value.setViewShow(setViewShow)
}

// 获取配置弹窗
const systemDrawer = ref(null)
const openSet = ()=>{
    systemDrawer.value.show()
}
</script>

<style lang="scss">
.wap-menu .ant-drawer-body{
    padding: 0 !important;
}
  //设置按钮
.set-up{
  width: 50px;
  height: 50px;
  position: fixed;
  right: -35px;
  top: 40vh;
  z-index: 9;
  transition: all .3s;
  border-radius: 15px 0 0 15px !important;
  overflow: hidden;
  .ant-btn{
    height: 100% !important;
  }
}
.set-up:hover{
    right: 0;
}
</style>