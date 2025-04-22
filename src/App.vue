<script setup lang="ts">
import ManilaCalculator from './components/ManilaCalculator.vue'
</script>

<template>
  <div class="app-container">
    <header>
      <div class="header-content">
        <h1>🏴‍☠️ 只有如此才能打败 Spark 吗？ —— 马尼拉期望计算器</h1>
      </div>
    </header>

    <main>
      <ManilaCalculator />
    </main>
  </div>
</template>

<style>
/* 这里移除 scoped 以便可以影响全局 */
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
  max-width: 1800px;
  padding: 0.5rem;
  margin-bottom: 0.5rem;
  border-bottom: 1px solid var(--app-border);
}

.header-content {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

h1 {
  font-size: 1.5rem;
  color: var(--app-text);
  margin: 0;
}

main {
  width: 100%;
  padding: 0;
  display: flex;
  justify-content: center;
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
