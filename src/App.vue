<script setup>
import { ref, computed, onMounted } from 'vue';
import DeviceList from './components/DeviceList.vue';
import GiscusComments from './components/GiscusComments.vue';
import Footer from "./components/Footer.vue";
import DateSelector from "./components/DateSelector.vue";
import Header from "./components/Header.vue";
import Toast from "./components/Toast.vue";
import LoadingSkeleton from "./components/LoadingSkeleton.vue";
import DeviceCountCard from "./components/DeviceCountCard.vue";
import StatsPanel from "./components/StatsPanel.vue";
import DarkModeToggle from "./components/DarkModeToggle.vue";
import { pageConfig } from "./composables/pageConfig.js";
import { useToast } from "./composables/useToast.js";
import { useDevices } from "./composables/useDevices.js";
import { useStatsState } from "./composables/useStatsState.js";

// ===== 配置管理 =====
const { pageConfigs, isLoading: configLoading, fetchFlags } = pageConfig();

// 页面加载状态
const isPageReady = ref(false);

// 计算配置项
const showDeviceCount = computed(() =>
    isPageReady.value && (pageConfigs.value?.config?.WEB_DEVICE_COUNT ?? false)
);

const showComments = computed(() =>
    isPageReady.value && (pageConfigs.value?.config?.WEB_COMMENT ?? true)
);

const showAISummary = computed(() =>
    isPageReady.value && (pageConfigs.value?.config?.WEB_AI_SUMMARY ?? true)
);

// ===== 音乐卡片跳转方法 =====
const openMusicPage = () => {
  window.open('https://yourspotify.shimmerl.top:12998/?gname=Last+day&amp;token=52839df5-892d-4b42-b4ba-a42a777ce5d6')
};

const serverTzOffset = computed(() =>
    pageConfigs.value?.tzOffset ?? 8
);

const showSummary = computed(() =>
    isPageReady.value && (pageConfigs.value?.config?.WEB_SUMMARY ?? true)
);

// ===== Toast =====
const { toast, showToast } = useToast();

// ===== 设备管理 =====
const {
    devices,
    selectedDevice,
    clientIp,
    isRefreshing,
    statsKey,
    hasRealDevices,
    fetchClientIp,
    fetchDevices,
    refreshStats,
    setupAutoRefresh,
    getSelectedDevice
} = useDevices(showToast, { showSummary });

// ===== 统计状态 =====
const {
    selectedDate,
    statsType,
    timeOffset,
    stats,
    getDateRangeText,
    handleStatsUpdate
} = useStatsState();

// ===== 生命周期 =====
onMounted(async () => {
    try {
        await fetchFlags();
        isPageReady.value = true;

        // 并行加载数据
        await Promise.all([
            fetchDevices(),
            fetchClientIp()
        ]);
    } catch (err) {
        console.error('❌ 初始化失败:', err);
        isPageReady.value = true;
    }

    setupAutoRefresh();
});
</script>

<template>
  <!-- Toast 提示 -->
  <Toast :show="toast.show" :message="toast.message" :type="toast.type" />

  <!-- 暗黑模式切换 -->
  <DarkModeToggle />

  <!-- 加载骨架屏 -->
  <LoadingSkeleton v-if="!isPageReady" />

  <!-- 实际内容 -->
  <div v-else class="bg-gray-100 min-h-screen rounded-lg dark:bg-[#1e2022]">
    <div class="max-w-7xl mx-auto px-4">
      <Header />

      <div class="flex flex-col lg:flex-row gap-6 pb-6">
        <!-- 左侧模块区 -->
        <div class="space-y-6">
          <!-- 设备统计卡片 -->
          <DeviceCountCard v-if="showDeviceCount" :devices="devices" />

          <!-- 评论区组件 -->
          <GiscusComments
              v-if="showComments"
              :repo="pageConfigs.config.GISCUS_REPO"
              :repo-id="pageConfigs.config.GISCUS_REPOID"
              :category="pageConfigs.config.GISCUS_CATEGORY"
              :category-id="pageConfigs.config.GISCUS_CATEGORYID"
              :mapping="pageConfigs.config.GISCUS_MAPPING"
              :reactions-enabled="pageConfigs.config.GISCUS_REACTIONSENABLED ? '1' : '0'"
              :emit-metadata="pageConfigs.config.GISCUS_EMITMETADATA ? '1' : '0'"
              :input-position="pageConfigs.config.GISCUS_INPUTPOSITION"
              :theme="pageConfigs.config.GISCUS_THEME"
              :lang="pageConfigs.config.GISCUS_LANG"
          />

          <!-- 设备列表 -->
          <div class="sticky top-4">
            <DeviceList
                :devices="devices"
                v-model:selected-device="selectedDevice"
                @refresh-devices="fetchDevices"
            />


            <div class="my-4 bg-white rounded-lg not-dark:shadow-md p-4 dark:bg-[#181a1b]">
            <!-- 音乐天地标题（粗体样式统一，无语法错误） -->
            <div class="item-headline-music flex items-center gap-2 mb-3 font-bold text-gray-800 dark:text-gray-200 text-base">
              <i class="haofont hao-icon-music"></i>
              <span>音乐天地</span>
              </div>

              <!-- 简化后的音乐卡片（无翻转容器，仅单个背景图） -->
              <div class="card-widget card-music cursor-pointer" @click="openMusicPage">
                <!-- 仅保留单个正面背景图，删除翻转和背面相关内容 -->
                <div class="msimg w-full rounded-lg" style="background:url(https://spotify.shimmerl.top:3000/api?user=9uj82h9nbmtxex14xy22ndxnz&amp;unique=true&amp;count=6) center center/100% no-repeat; padding-top: 100%;"></div>
              </div>
            </div>

            <!-- 日期筛选组件 -->
            <Transition name="slide-fade">
              <DateSelector
                  v-show="selectedDevice !== null"
                  v-model="statsType"
                  v-model:offset="timeOffset"
                  v-model:selected-date="selectedDate"
                  :date-range-text="getDateRangeText()"
                  :server-tz-offset="serverTzOffset"
              />
            </Transition>
          </div>
        </div>

        <!-- 右侧统计 -->
        <div class="flex-1 min-w-0">
          <StatsPanel
              :selected-device="selectedDevice"
              :device-info="getSelectedDevice()"
              :selected-date="selectedDate"
              :stats-type="statsType"
              :time-offset="timeOffset"
              :show-ai-summary="showAISummary"
              :stats-key="statsKey"
              :is-refreshing="isRefreshing"
              :has-real-devices="hasRealDevices"
              @stats-update="handleStatsUpdate"
              @refresh="refreshStats"
          />
        </div>
      </div>

      <Footer :client-ip="clientIp" />
    </div>
  </div>
</template>

<style scoped>
/* DateSelector 动画 */
.slide-fade-enter-active {
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  overflow: hidden;
}

.slide-fade-leave-active {
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  overflow: hidden;
}

.slide-fade-enter-from {
  opacity: 0;
  max-height: 0;
  transform: translateY(-10px);
}

.slide-fade-enter-to {
  opacity: 1;
  max-height: 220px;
  transform: translateY(0);
}

.slide-fade-leave-from {
  opacity: 1;
  max-height: 300px;
  transform: translateY(0);
}

.slide-fade-leave-to {
  opacity: 0;
  max-height: 0;
  transform: translateY(-10px);
}
</style>
