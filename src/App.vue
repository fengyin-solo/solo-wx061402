<template>
  <div class="game-container">
    <div class="terminal-header">
      <span class="terminal-dot red"></span>
      <span class="terminal-dot yellow"></span>
      <span class="terminal-dot green"></span>
      <span class="terminal-title">🌴 海岛生存模拟器 v1.0</span>
      <span class="save-indicator" :class="{ saving: isSaving }">
        {{ isSaving ? '💾 保存中...' : '💾 已自动保存' }}
      </span>
    </div>

    <div class="game-content">
      <div class="left-panel">
        <div class="section status-section">
          <h2 class="section-title">📊 生存状态</h2>
          <div class="stats-grid">
            <div class="stat-item" v-for="(stat, key) in gameState.stats" :key="key">
              <div class="stat-label">
                <span class="stat-icon">{{ statIcons[key] }}</span>
                <span>{{ statLabels[key] }}</span>
              </div>
              <div class="stat-bar">
                <div 
                  class="stat-fill" 
                  :style="{ width: stat.value + '%' }"
                  :class="getStatClass(stat.value)"
                ></div>
              </div>
              <span class="stat-value">{{ Math.round(stat.value) }}/100</span>
            </div>
          </div>
        </div>

        <div class="section inventory-section">
          <h2 class="section-title">🎒 背包资源</h2>
          <div class="inventory-grid">
            <div class="inventory-item" v-for="(item, key) in gameState.inventory" :key="key">
              <span class="item-icon">{{ itemIcons[key] }}</span>
              <span class="item-name">{{ itemLabels[key] }}</span>
              <span class="item-count">×{{ item }}</span>
            </div>
          </div>
        </div>

        <div class="section tools-section">
          <h2 class="section-title">🛠️ 已制造物品</h2>
          <div class="tools-list">
            <div 
              class="tool-item" 
              v-for="(crafted, key) in gameState.crafted" 
              :key="key"
              :class="{ active: crafted }"
            >
              <span>{{ toolIcons[key] }} {{ toolLabels[key] }}</span>
              <span v-if="crafted" class="tool-bonus">{{ toolBonuses[key] }}</span>
              <span v-else class="not-crafted">未制造</span>
            </div>
          </div>
        </div>
      </div>

      <div class="right-panel">
        <div class="section action-section">
          <h2 class="section-title">⛏️ 收集资源</h2>
          <div class="action-buttons">
            <button 
              class="action-btn collect-btn" 
              @click="collectResource('coconut')"
              :disabled="!canAct"
            >
              <span class="btn-icon">🥥</span>
              <span>采集椰子</span>
              <span class="btn-cost">-5体力</span>
            </button>
            <button 
              class="action-btn collect-btn" 
              @click="collectResource('wood')"
              :disabled="!canAct"
            >
              <span class="btn-icon">🪵</span>
              <span>砍伐木头</span>
              <span class="btn-cost">-8体力</span>
            </button>
            <button 
              class="action-btn collect-btn" 
              @click="collectResource('stone')"
              :disabled="!canAct"
            >
              <span class="btn-icon">🪨</span>
              <span>敲打石头</span>
              <span class="btn-cost">-10体力</span>
            </button>
          </div>
        </div>

        <div class="section action-section">
          <h2 class="section-title">🔨 制造物品</h2>
          <div class="action-buttons">
            <button 
              class="action-btn craft-btn" 
              @click="craftItem('axe')"
              :disabled="!canCraft('axe') || gameState.crafted.axe"
            >
              <span class="btn-icon">🪓</span>
              <span>制造斧头</span>
              <span class="btn-cost">🪵×3 🪨×2</span>
            </button>
            <button 
              class="action-btn craft-btn" 
              @click="craftItem('fishingRod')"
              :disabled="!canCraft('fishingRod') || gameState.crafted.fishingRod"
            >
              <span class="btn-icon">🎣</span>
              <span>制造鱼竿</span>
              <span class="btn-cost">🪵×2 🪨×1</span>
            </button>
            <button 
              class="action-btn craft-btn" 
              @click="craftItem('shelter')"
              :disabled="!canCraft('shelter') || gameState.crafted.shelter"
            >
              <span class="btn-icon">🏠</span>
              <span>建造庇护所</span>
              <span class="btn-cost">🪵×8 🪨×4 🥥×2</span>
            </button>
          </div>
        </div>

        <div class="section action-section">
          <h2 class="section-title">🗺️ 探索区域</h2>
          <div class="action-buttons">
            <button 
              class="action-btn explore-btn" 
              @click="explore('beach')"
              :disabled="!canExplore"
            >
              <span class="btn-icon">🏖️</span>
              <span>探索海滩</span>
              <span class="btn-cost">-15体力</span>
            </button>
            <button 
              class="action-btn explore-btn" 
              @click="explore('jungle')"
              :disabled="!canExplore"
            >
              <span class="btn-icon">🌴</span>
              <span>探索丛林</span>
              <span class="btn-cost">-20体力</span>
            </button>
            <button 
              class="action-btn explore-btn" 
              @click="explore('reef')"
              :disabled="!canExplore"
            >
              <span class="btn-icon">🪸</span>
              <span>探索礁石</span>
              <span class="btn-cost">-25体力</span>
            </button>
          </div>
        </div>

        <div class="section action-section">
          <h2 class="section-title">🍽️ 生存行动</h2>
          <div class="action-buttons">
            <button 
              class="action-btn eat-btn" 
              @click="eatCoconut"
              :disabled="gameState.inventory.coconut < 1"
            >
              <span class="btn-icon">🥥</span>
              <span>吃椰子</span>
              <span class="btn-cost">+20饱食 +15水分</span>
            </button>
            <button 
              class="action-btn rest-btn" 
              @click="rest"
              :disabled="!canRest"
            >
              <span class="btn-icon">😴</span>
              <span>休息恢复</span>
              <span class="btn-cost">+30体力 -5饱食</span>
            </button>
            <button 
              class="action-btn fish-btn" 
              @click="fish"
              :disabled="!gameState.crafted.fishingRod || !canAct"
            >
              <span class="btn-icon">🐟</span>
              <span>钓鱼</span>
              <span class="btn-cost">-10体力</span>
            </button>
          </div>
        </div>
      </div>
    </div>

    <div class="log-section">
      <h2 class="section-title">📜 事件日志</h2>
      <div class="log-container" ref="logContainer">
        <div 
          class="log-entry" 
          v-for="(log, index) in logs" 
          :key="index"
          :class="log.type"
        >
          <span class="log-time">[{{ log.time }}]</span>
          <span class="log-message">{{ log.message }}</span>
        </div>
      </div>
    </div>

    <div v-if="showDangerModal" class="modal-overlay" @click.self="showDangerModal = false">
      <div class="modal danger-modal">
        <h3>⚠️ 遭遇危险！</h3>
        <p class="danger-text">{{ dangerMessage }}</p>
        <div class="danger-effect" v-if="dangerEffect">{{ dangerEffect }}</div>
        <button class="modal-btn" @click="showDangerModal = false">我知道了</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive, computed, onMounted, onUnmounted, ref, nextTick } from 'vue'

const SAVE_KEY = 'island_survival_save'

const initialState = {
  stats: {
    stamina: { value: 100 },
    hunger: { value: 100 },
    thirst: { value: 100 },
    health: { value: 100 },
    sanity: { value: 100 }
  },
  inventory: {
    coconut: 3,
    wood: 5,
    stone: 2,
    fish: 0
  },
  crafted: {
    axe: false,
    fishingRod: false,
    shelter: false
  },
  gameTime: 0
}

const gameState = reactive(loadGame())

function loadGame() {
  try {
    const saved = localStorage.getItem(SAVE_KEY)
    if (saved) {
      const parsed = JSON.parse(saved)
      return {
        stats: {
          stamina: { value: parsed.stats?.stamina ?? 100 },
          hunger: { value: parsed.stats?.hunger ?? 100 },
          thirst: { value: parsed.stats?.thirst ?? 100 },
          health: { value: parsed.stats?.health ?? 100 },
          sanity: { value: parsed.stats?.sanity ?? 100 }
        },
        inventory: {
          coconut: parsed.inventory?.coconut ?? 3,
          wood: parsed.inventory?.wood ?? 5,
          stone: parsed.inventory?.stone ?? 2,
          fish: parsed.inventory?.fish ?? 0
        },
        crafted: {
          axe: parsed.crafted?.axe ?? false,
          fishingRod: parsed.crafted?.fishingRod ?? false,
          shelter: parsed.crafted?.shelter ?? false
        },
        gameTime: parsed.gameTime ?? 0
      }
    }
  } catch (e) {
    console.error('加载存档失败:', e)
  }
  return JSON.parse(JSON.stringify(initialState))
}

function saveGame() {
  isSaving.value = true
  const saveData = {
    stats: {
      stamina: gameState.stats.stamina.value,
      hunger: gameState.stats.hunger.value,
      thirst: gameState.stats.thirst.value,
      health: gameState.stats.health.value,
      sanity: gameState.stats.sanity.value
    },
    inventory: { ...gameState.inventory },
    crafted: { ...gameState.crafted },
    gameTime: gameState.gameTime
  }
  localStorage.setItem(SAVE_KEY, JSON.stringify(saveData))
  setTimeout(() => {
    isSaving.value = false
  }, 500)
}

const statLabels = {
  stamina: '体力',
  hunger: '饱食度',
  thirst: '水分',
  health: '健康度',
  sanity: '理智值'
}

const statIcons = {
  stamina: '⚡',
  hunger: '🍖',
  thirst: '💧',
  health: '❤️',
  sanity: '🧠'
}

const itemLabels = {
  coconut: '椰子',
  wood: '木头',
  stone: '石头',
  fish: '鱼'
}

const itemIcons = {
  coconut: '🥥',
  wood: '🪵',
  stone: '🪨',
  fish: '🐟'
}

const toolLabels = {
  axe: '斧头',
  fishingRod: '鱼竿',
  shelter: '简易庇护所'
}

const toolIcons = {
  axe: '🪓',
  fishingRod: '🎣',
  shelter: '🏠'
}

const toolBonuses = {
  axe: '木头获取 +50%',
  fishingRod: '可以钓鱼',
  shelter: '危险概率 -30%'
}

const craftRecipes = {
  axe: { wood: 3, stone: 2 },
  fishingRod: { wood: 2, stone: 1 },
  shelter: { wood: 8, stone: 4, coconut: 2 }
}

const logs = ref([])
const logContainer = ref(null)
const isSaving = ref(false)
const showDangerModal = ref(false)
const dangerMessage = ref('')
const dangerEffect = ref('')

const canAct = computed(() => gameState.stats.stamina.value > 0)
const canExplore = computed(() => gameState.stats.stamina.value >= 20)
const canRest = computed(() => gameState.stats.hunger.value >= 5)

function getStatClass(value) {
  if (value <= 20) return 'danger'
  if (value <= 50) return 'warning'
  return 'normal'
}

function addLog(message, type = 'info') {
  const now = new Date()
  const time = `${String(now.getHours()).padStart(2, '0')}:${String(now.getMinutes()).padStart(2, '0')}:${String(now.getSeconds()).padStart(2, '0')}`
  logs.value.push({ message, type, time })
  if (logs.value.length > 50) logs.value.shift()
  nextTick(() => {
    if (logContainer.value) {
      logContainer.value.scrollTop = logContainer.value.scrollHeight
    }
  })
}

function clamp(value, min = 0, max = 100) {
  return Math.max(min, Math.min(max, value))
}

function collectResource(type) {
  const costs = { coconut: 5, wood: 8, stone: 10 }
  if (gameState.stats.stamina.value < costs[type]) {
    addLog('体力不足，无法收集！', 'danger')
    return
  }

  let amount = 1
  if (type === 'wood' && gameState.crafted.axe) {
    amount = Math.random() > 0.5 ? 2 : 1
    addLog('🪓 斧头加成！', 'bonus')
  }

  gameState.stats.stamina.value = clamp(gameState.stats.stamina.value - costs[type])
  gameState.inventory[type] += amount
  addLog(`采集到 ${itemIcons[type]} ${itemLabels[type]} ×${amount}`, 'success')

  passiveDecay()
  checkGameOver()
}

function canCraft(item) {
  const recipe = craftRecipes[item]
  if (!recipe) return false
  return Object.entries(recipe).every(
    ([resource, amount]) => gameState.inventory[resource] >= amount
  )
}

function craftItem(item) {
  if (!canCraft(item) || gameState.crafted[item]) return

  const recipe = craftRecipes[item]
  Object.entries(recipe).forEach(([resource, amount]) => {
    gameState.inventory[resource] -= amount
  })
  gameState.crafted[item] = true
  addLog(`🎉 成功制造 ${toolIcons[item]} ${toolLabels[item]}！${toolBonuses[item]}`, 'success')

  passiveDecay()
}

function explore(area) {
  const costs = { beach: 15, jungle: 20, reef: 25 }
  const areaNames = { beach: '海滩', jungle: '丛林', reef: '礁石' }
  const areaIcons = { beach: '🏖️', jungle: '🌴', reef: '🪸' }

  if (gameState.stats.stamina.value < costs[area]) {
    addLog('体力不足，无法探索！', 'danger')
    return
  }
  if (gameState.stats.stamina.value < 20) {
    addLog('体力低于20，太虚弱了，不能探索！', 'danger')
    return
  }

  gameState.stats.stamina.value = clamp(gameState.stats.stamina.value - costs[area])

  let dangerChance = 0.3
  if (gameState.crafted.shelter) dangerChance -= 0.1
  if (area === 'jungle') dangerChance += 0.1

  if (Math.random() < dangerChance) {
    triggerDanger(area)
  } else {
    const rewards = getExploreRewards(area)
    rewards.forEach(reward => {
      gameState.inventory[reward.type] += reward.amount
      addLog(`探索 ${areaIcons[area]} ${areaNames[area]} 获得 ${itemIcons[reward.type]} ${itemLabels[reward.type]} ×${reward.amount}`, 'success')
    })
  }

  passiveDecay()
  checkGameOver()
}

function getExploreRewards(area) {
  const rewards = []
  const rand = Math.random()

  switch (area) {
    case 'beach':
      rewards.push({ type: 'coconut', amount: Math.floor(rand * 3) + 1 })
      if (rand > 0.5) rewards.push({ type: 'wood', amount: Math.floor(rand * 2) + 1 })
      if (rand > 0.8) rewards.push({ type: 'stone', amount: 1 })
      break
    case 'jungle':
      rewards.push({ type: 'wood', amount: Math.floor(rand * 4) + 2 })
      if (rand > 0.4) rewards.push({ type: 'coconut', amount: Math.floor(rand * 2) + 1 })
      if (rand > 0.7) rewards.push({ type: 'stone', amount: Math.floor(rand * 2) + 1 })
      break
    case 'reef':
      rewards.push({ type: 'stone', amount: Math.floor(rand * 3) + 2 })
      if (rand > 0.5) rewards.push({ type: 'fish', amount: Math.floor(rand * 2) + 1 })
      if (rand > 0.8) rewards.push({ type: 'coconut', amount: 1 })
      break
  }
  return rewards.filter(r => r.amount > 0)
}

function triggerDanger(area) {
  const dangers = [
    {
      name: '毒蛇',
      icon: '🐍',
      message: '你被一条毒蛇咬伤了！',
      effect: () => {
        gameState.stats.health.value = clamp(gameState.stats.health.value - 25)
        gameState.stats.stamina.value = clamp(gameState.stats.stamina.value - 15)
        return '健康 -25, 体力 -15'
      }
    },
    {
      name: '中暑',
      icon: '☀️',
      message: '烈日炎炎，你中暑了！',
      effect: () => {
        gameState.stats.thirst.value = clamp(gameState.stats.thirst.value - 30)
        gameState.stats.stamina.value = clamp(gameState.stats.stamina.value - 20)
        return '水分 -30, 体力 -20'
      }
    },
    {
      name: '跌落',
      icon: '💫',
      message: '你在探索时不慎摔倒了！',
      effect: () => {
        gameState.stats.health.value = clamp(gameState.stats.health.value - 15)
        gameState.stats.stamina.value = clamp(gameState.stats.stamina.value - 10)
        return '健康 -15, 体力 -10'
      }
    },
    {
      name: '受惊',
      icon: '😱',
      message: '奇怪的声音让你受到惊吓！',
      effect: () => {
        gameState.stats.sanity.value = clamp(gameState.stats.sanity.value - 20)
        return '理智 -20'
      }
    }
  ]

  const danger = dangers[Math.floor(Math.random() * dangers.length)]
  const effectText = danger.effect()

  dangerMessage.value = `${danger.icon} ${danger.message}`
  dangerEffect.value = effectText
  showDangerModal.value = true

  addLog(`⚠️ 遭遇危险：${danger.icon} ${danger.name}！${effectText}`, 'danger')
}

function eatCoconut() {
  if (gameState.inventory.coconut < 1) {
    addLog('没有椰子可以吃！', 'danger')
    return
  }

  gameState.inventory.coconut -= 1
  gameState.stats.hunger.value = clamp(gameState.stats.hunger.value + 20)
  gameState.stats.thirst.value = clamp(gameState.stats.thirst.value + 15)
  addLog('🥥 吃了一个椰子，饱食 +20, 水分 +15', 'success')

  passiveDecay()
}

function fish() {
  if (!gameState.crafted.fishingRod) {
    addLog('需要先制造鱼竿才能钓鱼！', 'danger')
    return
  }
  if (gameState.stats.stamina.value < 10) {
    addLog('体力不足，无法钓鱼！', 'danger')
    return
  }

  gameState.stats.stamina.value = clamp(gameState.stats.stamina.value - 10)

  const success = Math.random() > 0.3
  if (success) {
    const amount = Math.floor(Math.random() * 3) + 1
    gameState.inventory.fish += amount
    gameState.stats.hunger.value = clamp(gameState.stats.hunger.value + 10 * amount)
    addLog(`🎣 钓上了 ${amount} 条鱼！饱食 +${10 * amount}`, 'success')
  } else {
    addLog('🎣 鱼儿跑了，什么也没钓到...', 'info')
  }

  passiveDecay()
  checkGameOver()
}

function rest() {
  if (gameState.stats.hunger.value < 5) {
    addLog('太饿了，无法休息！', 'danger')
    return
  }

  gameState.stats.hunger.value = clamp(gameState.stats.hunger.value - 5)
  gameState.stats.stamina.value = clamp(gameState.stats.stamina.value + 30)
  if (gameState.crafted.shelter) {
    gameState.stats.health.value = clamp(gameState.stats.health.value + 10)
    gameState.stats.sanity.value = clamp(gameState.stats.sanity.value + 10)
    addLog('😴 在庇护所里休息，体力 +30, 健康 +10, 理智 +10', 'success')
  } else {
    addLog('😴 休息了一会，体力 +30', 'success')
  }

  passiveDecay()
}

function passiveDecay() {
  gameState.stats.hunger.value = clamp(gameState.stats.hunger.value - 2)
  gameState.stats.thirst.value = clamp(gameState.stats.thirst.value - 2)
  gameState.gameTime += 1

  if (gameState.stats.hunger.value <= 0) {
    gameState.stats.health.value = clamp(gameState.stats.health.value - 5)
    addLog('😣 你饿坏了，健康正在下降！', 'danger')
  }
  if (gameState.stats.thirst.value <= 0) {
    gameState.stats.health.value = clamp(gameState.stats.health.value - 8)
    addLog('🥵 你渴坏了，健康正在下降！', 'danger')
  }
  if (gameState.stats.sanity.value <= 30) {
    addLog('🧠 你的精神状态不太好...', 'warning')
  }
}

function checkGameOver() {
  if (gameState.stats.health.value <= 0) {
    addLog('💀 你死了...游戏结束！', 'danger')
    setTimeout(() => {
      if (confirm('你死了！是否重新开始游戏？')) {
        resetGame()
      }
    }, 500)
  }
}

function resetGame() {
  localStorage.removeItem(SAVE_KEY)
  Object.assign(gameState.stats.stamina, { value: 100 })
  Object.assign(gameState.stats.hunger, { value: 100 })
  Object.assign(gameState.stats.thirst, { value: 100 })
  Object.assign(gameState.stats.health, { value: 100 })
  Object.assign(gameState.stats.sanity, { value: 100 })
  gameState.inventory.coconut = 3
  gameState.inventory.wood = 5
  gameState.inventory.stone = 2
  gameState.inventory.fish = 0
  gameState.crafted.axe = false
  gameState.crafted.fishingRod = false
  gameState.crafted.shelter = false
  gameState.gameTime = 0
  logs.value = []
  addLog('🌅 新的一天开始了，你醒来发现自己在一座荒岛上...', 'info')
}

let saveInterval = null
let decayInterval = null

onMounted(() => {
  addLog('🌅 你醒来发现自己在一座荒岛上，必须想办法生存下去...', 'info')
  addLog('💡 提示：体力低于20无法探索，记得及时补充体力！', 'info')

  saveInterval = setInterval(() => {
    saveGame()
    addLog('💾 游戏已自动保存', 'info')
  }, 30000)

  decayInterval = setInterval(() => {
    passiveDecay()
    checkGameOver()
  }, 10000)
})

onUnmounted(() => {
  if (saveInterval) clearInterval(saveInterval)
  if (decayInterval) clearInterval(decayInterval)
})
</script>

<style scoped>
.game-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
  padding: 20px;
  font-family: 'Courier New', 'Consolas', monospace;
}

.terminal-header {
  background: #2d2d44;
  padding: 12px 20px;
  border-radius: 8px 8px 0 0;
  display: flex;
  align-items: center;
  gap: 10px;
  border-bottom: 1px solid #444;
}

.terminal-dot {
  width: 12px;
  height: 12px;
  border-radius: 50%;
}

.terminal-dot.red { background: #ff5f56; }
.terminal-dot.yellow { background: #ffbd2e; }
.terminal-dot.green { background: #27c93f; }

.terminal-title {
  flex: 1;
  color: #00ff88;
  font-weight: bold;
  text-shadow: 0 0 10px rgba(0, 255, 136, 0.5);
}

.save-indicator {
  color: #888;
  font-size: 14px;
  transition: color 0.3s;
}

.save-indicator.saving {
  color: #ffbd2e;
}

.game-content {
  display: grid;
  grid-template-columns: 350px 1fr;
  gap: 20px;
  background: rgba(0, 0, 0, 0.3);
  padding: 20px;
  border-radius: 0 0 8px 8px;
}

.left-panel, .right-panel {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.section {
  background: rgba(45, 45, 68, 0.8);
  border-radius: 8px;
  padding: 15px;
  border: 1px solid #444;
}

.section-title {
  color: #00ff88;
  font-size: 16px;
  margin-bottom: 15px;
  padding-bottom: 8px;
  border-bottom: 1px solid #444;
  text-shadow: 0 0 5px rgba(0, 255, 136, 0.3);
}

.stats-grid {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.stat-item {
  display: grid;
  grid-template-columns: 100px 1fr 60px;
  align-items: center;
  gap: 10px;
}

.stat-label {
  display: flex;
  align-items: center;
  gap: 5px;
  color: #e0e0e0;
  font-size: 14px;
}

.stat-icon {
  font-size: 16px;
}

.stat-bar {
  height: 16px;
  background: #333;
  border-radius: 8px;
  overflow: hidden;
  border: 1px solid #555;
}

.stat-fill {
  height: 100%;
  transition: width 0.3s ease, background-color 0.3s;
}

.stat-fill.normal {
  background: linear-gradient(90deg, #00ff88, #00cc6a);
  box-shadow: 0 0 10px rgba(0, 255, 136, 0.5);
}

.stat-fill.warning {
  background: linear-gradient(90deg, #ffbd2e, #ff9500);
  box-shadow: 0 0 10px rgba(255, 189, 46, 0.5);
}

.stat-fill.danger {
  background: linear-gradient(90deg, #ff5f56, #ff3b30);
  box-shadow: 0 0 10px rgba(255, 95, 86, 0.5);
  animation: pulse 1s infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.7; }
}

.stat-value {
  text-align: right;
  color: #aaa;
  font-size: 12px;
}

.inventory-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
}

.inventory-item {
  background: rgba(0, 0, 0, 0.3);
  padding: 10px;
  border-radius: 6px;
  display: flex;
  align-items: center;
  gap: 8px;
  border: 1px solid #444;
}

.item-icon {
  font-size: 20px;
}

.item-name {
  flex: 1;
  color: #e0e0e0;
  font-size: 14px;
}

.item-count {
  color: #00ff88;
  font-weight: bold;
}

.tools-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.tool-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 12px;
  background: rgba(0, 0, 0, 0.3);
  border-radius: 6px;
  border: 1px solid #444;
  opacity: 0.5;
}

.tool-item.active {
  opacity: 1;
  border-color: #00ff88;
  box-shadow: 0 0 10px rgba(0, 255, 136, 0.2);
}

.tool-bonus {
  color: #00ff88;
  font-size: 12px;
}

.not-crafted {
  color: #666;
  font-size: 12px;
}

.action-buttons {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  gap: 10px;
}

.action-btn {
  background: rgba(0, 0, 0, 0.4);
  border: 1px solid #555;
  border-radius: 8px;
  padding: 12px;
  color: #e0e0e0;
  cursor: pointer;
  transition: all 0.2s;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 5px;
  font-family: inherit;
}

.action-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
}

.action-btn:active:not(:disabled) {
  transform: translateY(0);
}

.action-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

.btn-icon {
  font-size: 24px;
}

.btn-cost {
  font-size: 11px;
  color: #888;
}

.collect-btn:hover:not(:disabled) {
  border-color: #4ecdc4;
  background: rgba(78, 205, 196, 0.2);
}

.craft-btn:hover:not(:disabled) {
  border-color: #ffe66d;
  background: rgba(255, 230, 109, 0.2);
}

.explore-btn:hover:not(:disabled) {
  border-color: #ff6b6b;
  background: rgba(255, 107, 107, 0.2);
}

.eat-btn:hover:not(:disabled) {
  border-color: #00ff88;
  background: rgba(0, 255, 136, 0.2);
}

.rest-btn:hover:not(:disabled) {
  border-color: #a855f7;
  background: rgba(168, 85, 247, 0.2);
}

.fish-btn:hover:not(:disabled) {
  border-color: #3b82f6;
  background: rgba(59, 130, 246, 0.2);
}

.log-section {
  margin-top: 20px;
  background: rgba(45, 45, 68, 0.8);
  border-radius: 8px;
  padding: 15px;
  border: 1px solid #444;
}

.log-container {
  height: 150px;
  overflow-y: auto;
  background: rgba(0, 0, 0, 0.4);
  border-radius: 6px;
  padding: 10px;
  font-size: 13px;
  line-height: 1.8;
}

.log-container::-webkit-scrollbar {
  width: 6px;
}

.log-container::-webkit-scrollbar-track {
  background: #333;
}

.log-container::-webkit-scrollbar-thumb {
  background: #555;
  border-radius: 3px;
}

.log-entry {
  padding: 2px 0;
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
}

.log-time {
  color: #666;
  margin-right: 8px;
}

.log-entry.success {
  color: #00ff88;
}

.log-entry.danger {
  color: #ff5f56;
}

.log-entry.warning {
  color: #ffbd2e;
}

.log-entry.info {
  color: #888;
}

.log-entry.bonus {
  color: #ffe66d;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal {
  background: #2d2d44;
  padding: 30px;
  border-radius: 12px;
  border: 2px solid #ff5f56;
  text-align: center;
  max-width: 400px;
  box-shadow: 0 0 30px rgba(255, 95, 86, 0.5);
  animation: shake 0.5s;
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-10px); }
  75% { transform: translateX(10px); }
}

.modal h3 {
  color: #ff5f56;
  margin-bottom: 15px;
  font-size: 20px;
}

.danger-text {
  color: #e0e0e0;
  font-size: 16px;
  margin-bottom: 10px;
}

.danger-effect {
  color: #ffbd2e;
  font-size: 14px;
  margin-bottom: 20px;
  padding: 10px;
  background: rgba(255, 189, 46, 0.1);
  border-radius: 6px;
}

.modal-btn {
  background: #ff5f56;
  color: white;
  border: none;
  padding: 10px 30px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  font-family: inherit;
  transition: all 0.2s;
}

.modal-btn:hover {
  background: #ff3b30;
  transform: scale(1.05);
}

@media (max-width: 1024px) {
  .game-content {
    grid-template-columns: 1fr;
  }
  
  .action-buttons {
    grid-template-columns: 1fr 1fr;
  }
}

@media (max-width: 640px) {
  .action-buttons {
    grid-template-columns: 1fr;
  }
  
  .stat-item {
    grid-template-columns: 90px 1fr 50px;
  }
}
</style>
