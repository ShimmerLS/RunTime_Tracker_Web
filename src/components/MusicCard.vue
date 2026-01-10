<script setup>
const props = defineProps({
  title: {
    type: String,
    required: true
  },
  coverUrl: {
    type: String,
    required: true
  },
  link: {
    type: String,
    required: true
  }
})

const open = () => {
  window.open(props.link, '_blank')
}
</script>

<template>
  <div class="bg-white rounded-lg not-dark:shadow-md p-4 dark:bg-[#181a1b]">
    <!-- 标题 -->
    <div class="flex items-center gap-2 mb-3 text-base font-semibold text-gray-800 dark:text-gray-200">
      <i class="haofont hao-icon-music text-lg"></i>
      <span>{{ title }}</span>
    </div>

    <!-- 卡片本体 -->
    <div
      class="cursor-pointer rounded-lg overflow-hidden hover:opacity-90 transition"
      @click="open"
    >
      <div
        class="w-full aspect-square bg-center bg-no-repeat bg-cover"
        :style="{ backgroundImage: `url(${coverUrl})` }"
      />
    </div>
  </div>
</template>