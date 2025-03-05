<script setup lang="ts">
import ScrollToTop from '@core/components/ScrollToTop.vue';
import { useThemeConfig } from '@core/composable/useThemeConfig';
import { hexToRgb } from '@layouts/utils';
import { useTheme } from 'vuetify';
import Chatbot from './components/Chatbot.vue';

const { syncInitialLoaderTheme, syncVuetifyThemeWithTheme: syncConfigThemeWithVuetifyTheme, isAppRtl, handleSkinChanges } = useThemeConfig()

const { global } = useTheme()

// Add chatbot control
const isChatbotOpen = ref(false)
const toggleChatbot = () => {
  isChatbotOpen.value = !isChatbotOpen.value
}

// ℹ️ Sync current theme with initial loader theme
syncInitialLoaderTheme()
syncConfigThemeWithVuetifyTheme()
handleSkinChanges()
</script>

<template>
  <VLocaleProvider :rtl="isAppRtl">
    <!-- ℹ️ This is required to set the background color of active nav link based on currently active global theme's primary -->
    <VApp :style="`--v-global-theme-primary: ${hexToRgb(global.current.value.colors.primary)}`">
      <RouterView />
      <ScrollToTop />
      <!-- Add Chatbot Widget -->
      <div class="chatbot-widget">
        <VScaleTransition>
          <div v-if="isChatbotOpen" class="chatbot-container">
            <Chatbot />
          </div>
        </VScaleTransition>

        <VBtn @click="toggleChatbot" color="primary" :icon="!isChatbotOpen" class="chatbot-toggle" elevation="2">
          <VIcon v-if="!isChatbotOpen">mdi-message-text</VIcon>
          <VIcon v-else>mdi-close</VIcon>
        </VBtn>
      </div>
    </VApp>
  </VLocaleProvider>
</template>

<style scoped>
.chatbot-widget {
  position: fixed;
  z-index: 999;
  inset-block-end: 20px;
  inset-inline-end: 20px;
}

.chatbot-container {
  position: absolute;
  inset-block-end: 70px;
  inset-inline-end: 0;
  margin-block-end: 10px;
}

.chatbot-toggle {
  position: relative;
  z-index: 1000;
}

/* Ensure chatbot doesn't overlap with scroll-to-top button */
:deep(.scroll-to-top) {
  inset-block-end: 90px;
}
</style>
