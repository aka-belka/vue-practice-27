<template>
  <div id="app">
    <header class="app-header">
      <h1>Генератор цветовых палитр</h1>
    </header>

    <!-- Основной контент -->
    <main class="main-content">
      <PaletteGenerator />
    </main>
    
    <footer class="app-footer">
      <p>Vue 3 + Vite • Практика 27</p>
    </footer>
  </div>
</template>

<script>
import { defineComponent } from 'vue'
import PaletteGenerator from './components/PaletteGenerator.vue'

export default defineComponent({
  name: 'App',
  components: {
    PaletteGenerator
  }
})
</script>

<style>
/* Глобальные стили */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
html, body {
  margin: 0;
  padding: 0;
  width: 100%;
}
body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: #6f334fff;
  color: #333;
  line-height: 1.6;
}

#app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  background-color: #cd96b0ff;
}


/* Шапка */
.app-header {
  background: linear-gradient(135deg, #ea6666ff 0%, #a24b74ff 100%);
  color: white;
  padding: 2rem;
  width: 100%;
  text-align: center;
  margin: 0;
}

.app-header h1 {
  margin-bottom: 0.5rem;
  font-size: 2.5rem;
}

.app-header p {
  opacity: 0.9;
  font-size: 1.1rem;
}

/* Основное содержимое */
.main-content {
  flex: 1;
  padding: 2rem;
  max-width: 1200px;
  margin: 0 auto;
  width: 100%;
}

/* Подвал */
.app-footer {
  background-color: #472b33ff;
  color: white;
  text-align: center;
  padding: 1rem;
  margin-top: auto;
  width: 100%;
  margin: 0;
}
</style>