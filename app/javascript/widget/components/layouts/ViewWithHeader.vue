<script>
import Banner from '../Banner.vue';
// import Branding from 'shared/components/Branding.vue';
import ChatHeader from '../ChatHeader.vue';
import ChatHeaderExpanded from '../ChatHeaderExpanded.vue';
import configMixin from '../../mixins/configMixin';
import { mapGetters } from 'vuex';
import { IFrameHelper } from 'widget/helpers/utils';

export default {
  components: {
    Banner,
    // Branding,
    ChatHeader,
    ChatHeaderExpanded,
  },
  mixins: [configMixin],
  data() {
    return {
      showPopoutButton: false,
      scrollPosition: 0,
      ticking: true,
      disableBranding: window.chatwootWebChannel.disableBranding || false,
      requestID: null,
    };
  },
  computed: {
    ...mapGetters({
      appConfig: 'appConfig/getAppConfig',
      availableAgents: 'agent/availableAgents',
    }),
    channelConfig() {
      return window.chatwootWebChannel;
    },
    portal() {
      return window.chatwootWebChannel.portal;
    },
    isHeaderCollapsed() {
      if (!this.hasIntroText) {
        return true;
      }
      return !this.isOnHomeView;
    },
    hasIntroText() {
      return (
        this.channelConfig.welcomeTitle || this.channelConfig.welcomeTagline
      );
    },
    showBackButton() {
      return ['article-viewer', 'messages', 'prechat-form'].includes(
        this.$route.name
      );
    },
    isOnArticleViewer() {
      return ['article-viewer'].includes(this.$route.name);
    },
    isOnHomeView() {
      return ['home'].includes(this.$route.name);
    },
    opacityClass() {
      if (this.isHeaderCollapsed) {
        return {};
      }
      if (this.scrollPosition > 30) {
        return { 'opacity-30': true };
      }
      if (this.scrollPosition > 25) {
        return { 'opacity-40': true };
      }
      if (this.scrollPosition > 20) {
        return { 'opacity-60': true };
      }
      if (this.scrollPosition > 15) {
        return { 'opacity-80': true };
      }
      if (this.scrollPosition > 10) {
        return { 'opacity-90': true };
      }
      return {};
    },
    telegramButtonClass() {
      return 'flex items-center gap-2 px-4 py-2.5 text-white text-sm font-medium rounded-full shadow-lg hover:shadow-xl transform hover:scale-105 transition-all duration-200 active:scale-95 telegram-button';
    },
  },
  mounted() {
    this.$el.addEventListener('scroll', this.updateScrollPosition);
  },
  unmounted() {
    this.$el.removeEventListener('scroll', this.updateScrollPosition);
    cancelAnimationFrame(this.requestID);
  },
  methods: {
    closeWindow() {
      IFrameHelper.sendMessage({ event: 'closeWindow' });
    },
    updateScrollPosition(event) {
      this.scrollPosition = event.target.scrollTop;
      if (!this.ticking) {
        this.requestID = window.requestAnimationFrame(() => {
          this.ticking = false;
        });

        this.ticking = true;
      }
    },
    handleTelegramHover(event) {
      event.target.style.backgroundColor = '#006699';
    },
    handleTelegramLeave(event) {
      event.target.style.backgroundColor = '#0088cc';
    },
  },
};
</script>

<template>
  <div
    class="w-full h-full bg-n-slate-2 dark:bg-n-solid-1"
    :class="{ 'overflow-auto': isOnHomeView }"
    @keydown.esc="closeWindow"
  >
    <div class="relative flex flex-col h-full">
      <div
        :class="{
          expanded: !isHeaderCollapsed,
          collapsed: isHeaderCollapsed,
          'shadow-[0_10px_15px_-16px_rgba(50,50,93,0.08),0_4px_6px_-8px_rgba(50,50,93,0.04)]':
            isHeaderCollapsed,
          ...opacityClass,
        }"
      >
        <ChatHeaderExpanded
          v-if="!isHeaderCollapsed"
          :intro-heading="appConfig.welcomeTitle || channelConfig.welcomeTitle"
          :intro-body="
            appConfig.welcomeDescription || channelConfig.welcomeTagline
          "
          :avatar-url="channelConfig.avatarUrl"
          :show-popout-button="appConfig.showPopoutButton"
        />
        <ChatHeader
          v-if="isHeaderCollapsed"
          :title="channelConfig.websiteName"
          :avatar-url="channelConfig.avatarUrl"
          :show-popout-button="appConfig.showPopoutButton"
          :available-agents="availableAgents"
          :show-back-button="showBackButton"
        />
      </div>
      <Banner />
      <router-view />

      <!-- 添加其他聊天渠道 -->
      <div
        class="px-4 py-4 border-t border-n-slate-4 dark:border-n-slate-8 bg-gradient-to-r from-n-slate-1 to-n-slate-2 dark:from-n-slate-2 dark:to-n-slate-3"
      >
        <div class="text-sm font-medium text-n-slate-12 text-center mb-3">
          {{ $t('WIDGET.MORE_CONTACT_WAYS') }}
        </div>
        <div class="flex justify-center gap-3">
          <a
            v-if="channelConfig.wsLink"
            :href="channelConfig.wsLink"
            target="_blank"
            rel="noopener noreferrer"
            class="flex items-center gap-2 px-4 py-2.5 bg-green-500 hover:bg-green-600 text-white text-sm font-medium rounded-full shadow-lg hover:shadow-xl transform hover:scale-105 transition-all duration-200 active:scale-95"
          >
            <!-- WhatsApp图标 -->
            <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
              <path
                d="M17.472 14.382c-.297-.149-1.758-.867-2.03-.967-.273-.099-.471-.148-.67.15-.197.297-.767.966-.94 1.164-.173.199-.347.223-.644.075-.297-.15-1.255-.463-2.39-1.475-.883-.788-1.48-1.761-1.653-2.059-.173-.297-.018-.458.13-.606.134-.133.298-.347.446-.52.149-.174.198-.298.298-.497.099-.198.05-.371-.025-.52-.075-.149-.669-1.612-.916-2.207-.242-.579-.487-.5-.669-.51-.173-.008-.371-.01-.57-.01-.198 0-.52.074-.792.372-.272.297-1.04 1.016-1.04 2.479 0 1.462 1.065 2.875 1.213 3.074.149.198 2.096 3.2 5.077 4.487.709.306 1.262.489 1.694.625.712.227 1.36.195 1.871.118.571-.085 1.758-.719 2.006-1.413.248-.694.248-1.289.173-1.413-.074-.124-.272-.198-.57-.347m-5.421 7.403h-.004a9.87 9.87 0 01-5.031-1.378l-.361-.214-3.741.982.998-3.648-.235-.374a9.86 9.86 0 01-1.51-5.26c.001-5.45 4.436-9.884 9.888-9.884 2.64 0 5.122 1.03 6.988 2.898a9.825 9.825 0 012.893 6.994c-.003 5.45-4.437 9.884-9.885 9.884m8.413-18.297A11.815 11.815 0 0012.05 0C5.495 0 .16 5.335.157 11.892c0 2.096.547 4.142 1.588 5.945L.057 24l6.305-1.654a11.882 11.882 0 005.683 1.448h.005c6.554 0 11.89-5.335 11.893-11.893A11.821 11.821 0 0020.465 3.516"
              />
            </svg>
            {{ $t('WIDGET.WHATSAPP') }}
          </a>
          <a
            v-if="channelConfig.tgLink"
            :href="channelConfig.tgLink"
            target="_blank"
            rel="noopener noreferrer"
            :class="telegramButtonClass"
            @mouseover="handleTelegramHover"
            @mouseout="handleTelegramLeave"
          >
            <!-- Telegram图标 -->
            <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
              <path
                d="M11.944 0A12 12 0 0 0 0 12a12 12 0 0 0 12 12 12 12 0 0 0 12-12A12 12 0 0 0 12 0a12 12 0 0 0-.056 0zm4.962 7.224c.1-.002.321.023.465.14a.506.506 0 0 1 .171.325c.016.093.036.306.02.472-.18 1.898-.962 6.502-1.36 8.627-.168.9-.499 1.201-.82 1.23-.696.065-1.225-.46-1.9-.902-1.056-.693-1.653-1.124-2.678-1.8-1.185-.78-.417-1.21.258-1.91.177-.184 3.247-2.977 3.307-3.23.007-.032.014-.15-.056-.212s-.174-.041-.249-.024c-.106.024-1.793 1.14-5.061 3.345-.48.33-.913.49-1.302.48-.428-.008-1.252-.241-1.865-.44-.752-.245-1.349-.374-1.297-.789.027-.216.325-.437.893-.663 3.498-1.524 5.83-2.529 6.998-3.014 3.332-1.386 4.025-1.627 4.476-1.635z"
              />
            </svg>
            {{ $t('WIDGET.TELEGRAM') }}
          </a>
        </div>
      </div>

      <!-- <Branding v-if="!isOnArticleViewer" :disable-branding="disableBranding" /> -->
    </div>
  </div>
</template>

<style scoped>
.telegram-button {
  background-color: #0088cc;
}

.telegram-button svg {
  border-radius: 50%;
}

.telegram-button:hover {
  background-color: #006699;
}
</style>
