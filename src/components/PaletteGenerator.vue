<template>
  <div class="demo-container">

    <!-- Панель управления -->
    <div class="controls">
      <button @click="generateRandomPalette" class="control-btn primary">
         Случайная палитра
      </button>

      <div class="range-group">
        <label>Количество цветов: {{ colorsCount }}</label>
        <input
          type="range"
          v-model="colorsCount"
          min="3"
          max="7"
          step="2"
          class="range-slider"
        />
      </div>

      <div class="toggle-group">
        <span>Формат:</span>
        <button
          @click="colorFormat = 'hex'"
          :class="{ active: colorFormat === 'hex' }"
          class="format-btn"
        >
          HEX
        </button>
        <button
          @click="colorFormat = 'rgb'"
          :class="{ active: colorFormat === 'rgb' }"
          class="format-btn"
        >
          RGB
        </button>
      </div>
    </div>

    <!-- Отображение палитры -->
    <div class="palette-display">
      <div
        v-for="color in palette"
        :key="color.id"
        class="color-card"
        :style="{ backgroundColor: color.hex }"
        @click="copyToClipboard(color)"
      >
        <div class="color-info" :class="getTextColorClass(color.hex)">
          <strong>{{ getFormattedValue(color) }}</strong>
          <span v-if="color.locked" class="lock-icon">🔒</span>
        </div>
        <div class="color-actions">
          <button
            @click.stop="toggleLock(color.id)"
            class="action-btn small"
            :title="color.locked ? 'Разблокировать' : 'Заблокировать'"
          >
            {{ color.locked ? '🔓' : '🔒' }}
          </button>
        </div>
      </div>
    </div>

    <!-- Уведомление о копировании -->
    <transition name="fade">
      <div v-if="showCopiedNotification" class="notification">
        Цвет {{ copiedColorValue }} скопирован в буфер обмена!
      </div>
    </transition>

    <!-- Превью палитры в UI-элементах -->
    <div class="preview-section">
      <h3>Превью интерфейса</h3>
      <div class="preview-controls">
        <button @click="toggleBackground" class="bg-toggle">
          {{ useDarkBg ? '🌙 Тёмный фон' : '☀️ Светлый фон' }}
        </button>
      </div>
      <div
        class="ui-preview"
        :class="useDarkBg ? 'dark-bg' : 'light-bg'"
      >
        <!-- Mockup элементов интерфейса -->
        <div class="mockup-header" :style="{ backgroundColor: getPreviewColor(0) }">
          <h4>Заголовок приложения</h4>
        </div>
        <div class="mockup-card">
          <p>Пример карточки с контентом</p>
          <button
            class="mockup-btn"
            :style="{
              backgroundColor: getPreviewColor(1),
              color: getContrastColor(getPreviewColor(1))
            }"
          >
            Основная кнопка
          </button>
          <button
            class="mockup-btn "
            :style="{
              backgroundColor: getPreviewColor(2),
              color: getContrastColor(getPreviewColor(2))
            }"
          >
            Вторая кнопка
          </button>
        </div>
        <div
          class="mockup-footer"
          :style="{ backgroundColor:  getPreviewColor(3),
          color: getContrastColor(getPreviewColor(3))
          }"
        >
          <p>Нижний колонтитул</p>
        </div>
      </div>
    </div>
  </div>
</template>



<script setup>
import { ref, computed, onMounted, watch } from 'vue'

// 1. Реактивное состояние палитры
const palette = ref([])
const colorsCount = ref(5)
const colorFormat = ref('hex')
const showCopiedNotification = ref(false)
const copiedColorValue = ref('')
const useDarkBg = ref(false)

// 2. Генерация случайного цвета в формате HEX
const generateRandomColor = () => {
  const letters = '0123456789ABCDEF'
  let color = '#'
  for (let i = 0; i < 6; i++) {
    color += letters[Math.floor(Math.random() * 16)]
  }
  return color
}
const getPreviewColor = (index) => {
  if (!palette.value.length) return '#f0f0f0'
  
  // Циклически используем цвета палитры
  const colorIndex = index % palette.value.length
  return palette.value[colorIndex]?.hex || '#f0f0f0'
}
// 3. Генерация гармоничной палитры
const generateRandomPalette = () => {
  const newColors = []
  const baseHue = Math.floor(Math.random() * 360)

  for (let i = 0; i < colorsCount.value; i++) {
    // Проверяем, не заблокирован ли уже существующий цвет на этой позиции
    const existingColor = palette.value[i]
    
    if (existingColor && existingColor.locked) {
      // Оставляем заблокированный цвет на месте
      newColors.push({ ...existingColor })
    } else {
      // Генерируем новый гармоничный цвет
      const hue = (baseHue + (i * 60)) % 360
      const saturation = 60 + Math.random() * 30
      const lightness = 40 + Math.random() * 30
      
      // Преобразуем HSL в HEX (упрощённый вариант)
      const hex = hslToHex(hue, saturation, lightness)
      
      newColors.push({
        id: Date.now() + i,
        hex: hex,
        locked: false
      })
    }
  }
  
  palette.value = newColors
}

// 4. Преобразование HSL в HEX
const hslToHex = (h, s, l) => {
  h /= 360
  s /= 100
  l /= 100
  
  let r, g, b
  
  if (s === 0) {
    r = g = b = l
  } else {
    const hue2rgb = (p, q, t) => {
      if (t < 0) t += 1
      if (t > 1) t -= 1
      if (t < 1/6) return p + (q - p) * 6 * t
      if (t < 1/2) return q
      if (t < 2/3) return p + (q - p) * (2/3 - t) * 6
      return p
    }
    
    const q = l < 0.5 ? l * (1 + s) : l + s - l * s
    const p = 2 * l - q
    
    r = hue2rgb(p, q, h + 1/3)
    g = hue2rgb(p, q, h)
    b = hue2rgb(p, q, h - 1/3)
  }
  
  const toHex = x => {
    const hex = Math.round(x * 255).toString(16)
    return hex.length === 1 ? '0' + hex : hex
  }
  
  return `#${toHex(r)}${toHex(g)}${toHex(b)}`
}

// 5. Копирование цвета в буфер обмена с использованием современного API[citation:1]
const copyToClipboard = async (color) => {
  const textToCopy = colorFormat.value === 'hex' 
    ? color.hex 
    : hexToRgbString(color.hex)
  
  try {
    // Используем современный Clipboard API[citation:1]
    await navigator.clipboard.writeText(textToCopy)
    
    // Показываем уведомление
    copiedColorValue.value = textToCopy
    showCopiedNotification.value = true
    
    // Скрываем уведомление через 2 секунды
    setTimeout(() => {
      showCopiedNotification.value = false
    }, 2000)
    
  } catch (err) {
    // Fallback для старых браузеров
    console.error('Ошибка копирования:', err)
    const textArea = document.createElement('textarea')
    textArea.value = textToCopy
    document.body.appendChild(textArea)
    textArea.select()
    document.execCommand('copy')
    document.body.removeChild(textArea)
    
    // Всё равно показываем уведомление
    copiedColorValue.value = textToCopy
    showCopiedNotification.value = true
    setTimeout(() => {
      showCopiedNotification.value = false
    }, 2000)
  }
}

// 6. Преобразование HEX в строку RGB
const hexToRgbString = (hex) => {
  const result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex)
  if (!result) return 'rgb(0, 0, 0)'
  
  return `rgb(${parseInt(result[1], 16)}, ${parseInt(result[2], 16)}, ${parseInt(result[3], 16)})`
}

// 7. Переключение блокировки цвета
const toggleLock = (colorId) => {
  const colorIndex = palette.value.findIndex(color => color.id === colorId)
  if (colorIndex !== -1) {
    palette.value[colorIndex].locked = !palette.value[colorIndex].locked
  }
}

// 8. Получение форматированного значения цвета
const getFormattedValue = (color) => {
  return colorFormat.value === 'hex' 
    ? color.hex.toUpperCase() 
    : hexToRgbString(color.hex)
}

// 9. Вычисляемые свойства
const lockedColorsCount = computed(() => {
  return palette.value.filter(color => color.locked).length
})

// 10. Определение цвета текста для контраста
const getTextColorClass = (hexColor) => {
  // Простой расчёт яркости цвета
  const hex = hexColor.replace('#', '')
  const r = parseInt(hex.substr(0, 2), 16)
  const g = parseInt(hex.substr(2, 2), 16)
  const b = parseInt(hex.substr(4, 2), 16)
  const brightness = (r * 299 + g * 587 + b * 114) / 1000
  
  return brightness > 128 ? 'dark-text' : 'light-text'
}

const getContrastColor = (hexColor) => {
  if (!hexColor) return '#000'
  const hex = hexColor.replace('#', '')
  const r = parseInt(hex.substr(0, 2), 16)
  const g = parseInt(hex.substr(2, 2), 16)
  const b = parseInt(hex.substr(4, 2), 16)
  const brightness = (r * 299 + g * 587 + b * 114) / 1000
  
  return brightness > 128 ? '#000' : '#FFF'
}

// 11. Работа с localStorage для сохранения палитры
const localStorageEnabled = ref(true)

const saveToLocalStorage = () => {
  if (!localStorageEnabled.value) return
  
  const dataToSave = {
    palette: palette.value,
    colorsCount: colorsCount.value,
    colorFormat: colorFormat.value,
    timestamp: new Date().toISOString()
  }
  
  try {
    localStorage.setItem('vuePaletteGenerator', JSON.stringify(dataToSave))
  } catch (err) {
    console.warn('Не удалось сохранить в localStorage:', err)
  }
}

const loadFromLocalStorage = () => {
  try {
    const savedData = localStorage.getItem('vuePaletteGenerator')
    if (savedData) {
      const parsedData = JSON.parse(savedData)
      
      // Проверяем, не устарели ли данные (больше 7 дней)
      const savedDate = new Date(parsedData.timestamp)
      const daysDiff = (new Date() - savedDate) / (1000 * 60 * 60 * 24)
      
      if (daysDiff < 7) {
        palette.value = parsedData.palette || []
        colorsCount.value = parsedData.colorsCount || 5
        colorFormat.value = parsedData.colorFormat || 'hex'
      }
    }
  } catch (err) {
    console.warn('Не удалось загрузить из localStorage:', err)
  }
}

const clearLocalStorage = () => {
  localStorage.removeItem('vuePaletteGenerator')
  palette.value = []
  colorsCount.value = 5
  colorFormat.value = 'hex'
}

// 12. Переключение фона превью
const toggleBackground = () => {
  useDarkBg.value = !useDarkBg.value
}

// 13. Наблюдатели для автосохранения
watch([palette, colorsCount, colorFormat], () => {
  saveToLocalStorage()
}, { deep: true })

// 14. Инициализация при монтировании
onMounted(() => {
  loadFromLocalStorage()
  
  // Если в localStorage ничего нет, генерируем начальную палитру
  if (palette.value.length === 0) {
    generateRandomPalette()
  }
})
</script>







<style scoped>
.demo-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Segoe UI', system-ui, sans-serif;
}

/* Панель управления */
.controls {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  margin: 30px 0;
  padding: 20px;
  background: #f8f9fa;
  border-radius: 12px;
  align-items: center;
}

.control-btn {
  padding: 12px 24px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s ease;
}

.control-btn.primary {
  background: linear-gradient(135deg, #ea6666ff 0%, #a24b69ff 100%);
  color: white;
}

.control-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.range-group {
  flex: 1;
  min-width: 200px;
}

.range-slider {
  width: 100%;
  margin-top: 8px;
  height: 6px;
  border-radius: 3px;
  background: #ddd;
  outline: none;
  -webkit-appearance: none;
}

.range-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #ea6666ff;
  cursor: pointer;
}

.toggle-group {
  display: flex;
  gap: 10px;
  align-items: center;
}

.format-btn {
  padding: 8px 16px;
  border: 2px solid #ea6666ff;
  background: transparent;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s;
}

.format-btn.active {
  background: #ea6666ff;
  color: white;
}

/* Отображение палитры */
.palette-display {
  display: flex;
  gap: 10px;
  margin: 30px 0;
  height: 150px;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
}

.color-card {
  flex: 1;
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding: 15px;
  cursor: pointer;
  transition: transform 0.3s ease;
}

.color-card:hover {
  transform: scale(1.05);
  z-index: 1;
}

.color-info {
  font-size: 14px;
  font-family: 'Consolas', monospace;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.color-info.dark-text {
  color: #000;
}

.color-info.light-text {
  color: #fff;
}

.lock-icon {
  font-size: 12px;
}

.color-actions {
  display: flex;
  justify-content: flex-end;
}

.action-btn {
  background: rgba(255, 255, 255, 0.2);
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 4px;
  color: white;
  cursor: pointer;
  padding: 4px 8px;
  backdrop-filter: blur(10px);
}

.action-btn.small {
  padding: 2px 6px;
  font-size: 12px;
}

/* Уведомление */
.notification {
  position: fixed;
  top: 20px;
  right: 20px;
  background: #28a745;
  color: white;
  padding: 12px 24px;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
  z-index: 1000;
  animation: slideIn 0.3s ease;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

@keyframes slideIn {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

/* Превью интерфейса */
.preview-section {
  margin: 40px 0;
  padding: 25px;
  background: #f8f9fa;
  border-radius: 12px;
}

.preview-controls {
  margin-bottom: 20px;
}

.bg-toggle {
  padding: 10px 20px;
  background: #8b969fff;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: background 0.3s;
}

.bg-toggle:hover {
  background: #5a6268;
}

.ui-preview {
  padding: 30px;
  border-radius: 12px;
  transition: background 0.3s;
}

.ui-preview.light-bg {
  background: #ffffff;
  border: 1px solid #dee2e6;
}

.ui-preview.dark-bg {
  background: #212529;
  border: 1px solid #495057;
  color: white;
}

.mockup-header {
  padding: 20px;
  border-radius: 8px 8px 0 0;
  color: white;
  text-align: center;
}

.mockup-card {
  padding: 30px;
  background: rgba(0, 0, 0, 0.05);
  border-radius: 0 0 8px 8px;
  margin: 0;
}

.mockup-btn {
  padding: 10px 20px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-weight: 600;
  margin-right: 10px;
  transition: opacity 0.3s;
}

.mockup-btn:hover {
  opacity: 0.9;
}


.mockup-footer {
  padding: 15px;
  margin-top: 20px;
  border-radius: 8px;
  text-align: center;
  color: white;
}

/* Отладочная информация */
.debug-info {
  margin-top: 30px;
  padding: 20px;
  background: #e9ecef;
  border-radius: 8px;
}

.state-info {
  display: flex;
  gap: 20px;
  align-items: center;
  flex-wrap: wrap;
}

.clear-btn {
  padding: 8px 16px;
  background: #dc3545;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: background 0.3s;
}

.clear-btn:hover {
  background: #c82333;
}

/* Адаптивность */
@media (max-width: 768px) {
  .controls {
    flex-direction: column;
    align-items: stretch;
  }
  
  .palette-display {
    flex-direction: column;
    height: auto;
  }
  
  .color-card {
    height: 80px;
  }
  
  .state-info {
    flex-direction: column;
    align-items: flex-start;
  }
}
</style>