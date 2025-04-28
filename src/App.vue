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
  <div class="app-container">
    <header>
      <div class="header-content">
        <h1>🏴‍☠️ 桌游《马尼拉》期望计算器</h1>
        <button
          class="theme-toggle"
          @click="toggleTheme"
          :title="darkMode ? '切换到浅色模式' : '切换到深色模式'"
        >
          {{ darkMode ? "☀️" : "🌙" }}
        </button>
      </div>
    </header>

    <main>
      <ManilaCalculator />
    </main>
  </div>
</template>

<style>
/* 为深色/浅色模式定义颜色变量 */
:root {
  --app-bg: #ffffff;
  --app-text: #2c3e50;
  --app-border: #e0e0e0;
}

[data-theme="dark"] {
  --app-bg: #121212;
  --app-text: #e0e0e0;
  --app-border: #444444;
}

.app-container {
  width: 100%;
  max-width: 100%;
  margin: 0 auto;
  padding: 0 10px;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  align-items: center;
  color: var(--app-text);
  background-color: var(--app-bg);
}

header {
  width: 100%;
  max-width: 1280px;
  padding: 0.5rem;
  margin-bottom: 0.5rem;
  border-bottom: 1px solid var(--app-border);
}

.header-content {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.75rem;
}

h1 {
  font-size: 1.5rem;
  color: var(--app-text);
  margin: 0;
}

/* 主题切换按钮样式 */
.theme-toggle {
  background: none;
  border: none;
  cursor: pointer;
  font-size: 1.5rem;
  padding: 0.3rem;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background-color 0.2s;
}

.theme-toggle:hover {
  background-color: rgba(128, 128, 128, 0.2);
}

main {
  width: 100%;
  padding: 0;
  display: flex;
  justify-content: center;
}

@media (max-width: 1280px) {
  h1 {
    font-size: 1.3rem;
  }

  .app-container {
    padding: 0 5px;
  }
}
</style>
