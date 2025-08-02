<script setup lang="ts">
import { ref, onMounted, watch } from "vue";
import ManilaCalculator from "./components/ManilaCalculator.vue";

// 主题状态控制
const darkMode = ref(false);

// 初始化主题
onMounted(() => {
  // 优先使用用户保存的偏好，否则使用系统偏好
  const savedTheme = localStorage.getItem("theme");
  if (savedTheme) {
    darkMode.value = savedTheme === "dark";
  } else {
    darkMode.value = window.matchMedia("(prefers-color-scheme: dark)").matches;
  }
  applyTheme(darkMode.value);
});

// 监听主题变化
watch(darkMode, (isDark) => {
  applyTheme(isDark);
  localStorage.setItem("theme", isDark ? "dark" : "light");
});

// 应用主题
const applyTheme = (isDark: boolean) => {
  document.documentElement.setAttribute(
    "data-theme",
    isDark ? "dark" : "light"
  );
};

// 切换主题
const toggleTheme = () => {
  darkMode.value = !darkMode.value;
};
</script>

<template>
  <main class="container">
    <header>
      <h1>🏴‍☠️ 桌游《马尼拉》期望计算器</h1>
      <button
        class="contrast"
        @click="toggleTheme"
        :title="darkMode ? '切换到浅色模式' : '切换到深色模式'"
        style="width: auto; padding: 0.5rem;"
      >
        {{ darkMode ? "☀️" : "🌙" }}
      </button>
    </header>

    <ManilaCalculator />
  </main>
</template>

<style>
/* 只保留少量自定义样式 */
header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
}

h1 {
  margin: 0;
}

@media (max-width: 768px) {
  header {
    flex-direction: column;
    gap: 1rem;
    text-align: center;
  }
  
  h1 {
    font-size: 1.25rem;
  }
}
</style>
