<script setup lang="ts">
import ManilaCalculator from './components/ManilaCalculator.vue'
</script>

<template>
  <div class="app-container">
    <header>
      <div class="header-content">
        <!-- <img alt="马尼拉期望计算器" class="logo" src="./assets/logo.svg" width="40" height="40" /> -->
        <h1>🏴‍☠️ 只有如此才能打败 Spark 吗？ —— 马尼拉期望计算器</h1>
      </div>
    </header>

    <main>
      <ManilaCalculator />
    </main>
  </div>
</template>

<style scoped>
.app-container {
  width: 100%;
  max-width: 100%;
  margin: 0 auto;
  padding: 0 10px;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  align-items: center; /* 水平居中整个应用 */
}

header {
  width: 100%;
  max-width: 1800px; /* 与计算器组件保持一致 */
  padding: 0.5rem;
  margin-bottom: 0.5rem;
  border-bottom: 1px solid #e0e0e0;
}

.header-content {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.logo {
  height: 40px;
  width: 40px;
}

h1 {
  font-size: 1.5rem;
  color: #2c3e50;
  margin: 0;
}

main {
  width: 100%;
  padding: 0;
  display: flex;
  justify-content: center; /* 水平居中主内容 */
}

@media (max-width: 1000px) {
  h1 {
    font-size: 1.3rem;
  }
  
  .app-container {
    padding: 0 5px;
  }
}
</style>
