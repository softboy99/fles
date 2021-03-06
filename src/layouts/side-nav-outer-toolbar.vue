<template>
  <div class="side-nav-outer-toolbar">
    <header-toolbar
      class="layout-header"
      :menu-toggle-enabled="true"
      :toggle-menu-func="toggleMenu"
      :title="title"
    />
    
    <!-- eslint-disable vue/valid-v-model -->
      <dx-drawer
        class="layout-body"
        position="before"
        template="default"
        v-model:opened="menuOpened"
        :opened-state-mode="drawerOptions.menuMode"
        :reveal-mode="drawerOptions.menuRevealMode"
        :min-size="drawerOptions.minMenuSize"
        :shading="drawerOptions.shaderEnabled"
        :close-on-outside-click="drawerOptions.closeOnOutsideClick"
      >
    
    <!-- eslint-enabled -->
      <dx-scroll-view ref="scrollViewRef" class="with-footer">
        <slot />
        <slot name="footer" />
      </dx-scroll-view>   
           <Suspense>
          <template #default>         
                  <side-nav-menu
                  :compact-mode="!menuOpened"
                  @click="handleSideBarClick"
                />
          </template>
          </Suspense>
      <!-- eslint-enable -->
    </dx-drawer>
     
  </div>
</template>

<script language ="ts">
import DxDrawer from "devextreme-vue/drawer";
import DxScrollView from "devextreme-vue/scroll-view";
import menuItems from "../app-navigation";
import HeaderToolbar from "../components/header-toolbar";
import SideNavMenu from "../components/side-nav-menu";
import { computed, ref, watch } from 'vue';
import { useRoute } from 'vue-router';
export default {
  props: {
    title: String,
    isXSmall: Boolean,
    isLarge: Boolean
  },
  setup(props) {
    const route = useRoute();
    const routePath = ref(route.path);
    const scrollViewRef = ref(null);
    const menuOpened = ref(props.isLarge);
    const menuTemporaryOpened = ref(false);
    function toggleMenu(e) {
      const pointerEvent = e.event;
      pointerEvent.stopPropagation();
      if (menuOpened.value) {
        menuTemporaryOpened.value = false;
      }
      menuOpened.value = !menuOpened.value;
    }
    function handleSideBarClick() {
      if (menuOpened.value === false) {
        menuTemporaryOpened.value = true;
      }
      menuOpened.value = true;
    }
    const drawerOptions = computed(() => {
      const shaderEnabled = !props.isLarge;
      return {
        menuMode: props.isLarge ? "shrink" : "overlap",
        menuRevealMode: props.isXSmall ? "slide" : "expand",
        minMenuSize: props.isXSmall ? 0 : 60,
        menuOpened:props.isLarge,
        closeOnOutsideClick: shaderEnabled,
        shaderEnabled
      };
    });
    const headerMenuTogglerEnabled = computed(() => {
      return props.isXSmall;
    });
    watch(
      () => props.isLarge,
      () => {
        if (!menuTemporaryOpened.value) {
          menuOpened.value = props.isLarge;
        }
    });
    watch(
      routePath,
      () => {
      if (menuTemporaryOpened.value || !props.isLarge) {
        menuOpened.value = false;
        menuTemporaryOpened.value = false;
      }
      scrollViewRef.value.instance.scrollTo(0);
      }
    );
    return {
      menuOpened,
      menuTemporaryOpened,
      menuItems,
      toggleMenu,
      handleSideBarClick,
      drawerOptions,
      headerMenuTogglerEnabled,
      scrollViewRef
    };
  },
  components: {
    DxDrawer,
    DxScrollView,
    HeaderToolbar,
    SideNavMenu
  }
};
</script>

<style lang="scss">
.side-nav-outer-toolbar {
  flex-direction: column;
  display: flex;
  height: 100%;
  width: 100%;
}
.layout-header {
  z-index: 1501;
}
.layout-body {
  flex: 1;
  min-height: 0;
}
.content {
  flex-grow: 1;
}
</style>