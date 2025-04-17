<template>
  <div class="manila-calculator">
    <div class="instructions-wrapper">
      <div class="instructions">
        <h3>使用说明</h3>
        <div class="instruction-content">
          <p>此计算器帮助玩家计算《马尼拉》桌游中各放置帮手位置的期望收益，辅助决策。</p>
          <ol>
            <li><strong>当前轮次</strong>：选择当前是第几次骰子投掷前，会影响船只到达目的地的概率计算。</li>
            <li><strong>船只设置</strong>：调整滑块设置各船当前位置 (0-13)。</li>
            <li><strong>已占位置</strong>：勾选已被占用的位置，系统会据此计算剩余位置的预期收益。</li>
            <li><strong>收益分析</strong>：表格中绿色背景表示正收益的位置，灰色表示已占用位置。</li>
            <li><strong>假重要设</strong>：基于后期不会有改变价值的位置占据状况被改变</li>
            <li><strong>额外村规</strong>：海盗对无法影响第二轮移动结束时位于 13 号格子的平底船</li>
          </ol>
        </div>
        <a href="https://github.com/framist/manila" target="_blank" class="github-link" title="查看GitHub仓库">
          <span class="github-icon">
            <svg viewBox="0 0 16 16" width="16" height="16" aria-hidden="true">
              <path fill-rule="evenodd"
                d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z">
              </path>
            </svg>
          </span>
        </a>
      </div>
    </div>

    <div class="calculator-layout">
      <!-- 输入区域 - 左侧 -->
      <div class="input-panel">
        <h2>游戏状态设置</h2>

        <div class="section boat-settings">
          <h3>船只位置设置</h3>
          <div class="dice-round">
            <h4>当前轮次</h4>
            <div class="round-selector">
              <label v-for="(round, index) in diceRounds" :key="index" class="round-option">
                <input type="radio" :value="round.value" v-model="currentRound" />
                {{ round.label }}
              </label>
            </div>
          </div>

          <div class="boats-setup">
            <div v-for="(boat, index) in boats" :key="index" class="boat-item">
              <div class="boat-header">
                <span class="boat-name" :class="boat.type">{{ boat.name }}</span>
                <div class="boat-toggles">
                  <label class="checkbox-label">
                    <input type="checkbox" v-model="boat.isSelected" />
                    本轮使用
                  </label>
                </div>
              </div>

              <div class="position-control">
                <span class="position-label">位置：{{ boat.position }}</span>
                <input type="range" v-model.number="boat.position" :min="0" :max="13" />
                <div class="position-track">
                  <div class="track-mark" v-for="i in 14" :key="i - 1">{{ i - 1 }}</div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div class="section special-settings">
          <h3>已占用位置设置</h3>

          <div class="boat-occupancy">
            <h4>平底船已占用位置</h4>
            <div v-for="(boat, boatIndex) in visibleBoats" :key="`boat-${boatIndex}`" class="occupancy-group">
              <h5 class="occupancy-header" :class="boat.type">{{ boat.name }}</h5>
              <div class="occupancy-slots">
                <label v-for="(pos, posIndex) in getBoatPositions(boat.type)" :key="`pos-${posIndex}`"
                  class="slot-label">
                  <input type="checkbox" v-model="pos.occupied" />
                  位置{{ posIndex + 1 }} ({{ pos.cost }}比索)
                </label>
              </div>
            </div>
          </div>

          <div class="port-yard-occupancy">
            <h4>港口与造船厂已占用位置</h4>
            <div class="occupancy-group">
              <div class="occupancy-slots port-yard-grid">
                <label v-for="(pos, posIndex) in portYardPositions" :key="`port-${posIndex}`" class="slot-label">
                  <input type="checkbox" v-model="pos.occupied" />
                  {{ pos.name }} ({{ pos.cost }}比索)
                </label>
              </div>
            </div>
          </div>

          <div class="special-occupancy">
            <h4>特殊位置已占用</h4>
            <div class="occupancy-group">
              <div class="occupancy-slots special-grid">
                <label v-for="(pos, posIndex) in specialPositions" :key="`special-${posIndex}`" class="slot-label">
                  <input type="checkbox" v-model="pos.occupied" />
                  {{ pos.name }} ({{ pos.cost }}比索)
                </label>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- 输出区域 - 右侧 -->
      <div class="output-panel">
        <h2>期望收益分析</h2>

        <div class="visualization-section">
          <div class="round-display">
            当前设置：{{ getCurrentRoundLabel() }}，剩余骰子次数：{{ remainingDiceThrows }}
          </div>

          <h3>船只位置可视化</h3>
          <div class="track-visualization">
            <div class="track" v-for="(boat, index) in visibleBoats" :key="index">
              <div class="track-header">
                <span class="track-label" :class="boat.type">{{ boat.name }}
                  <span v-for="(pos, posIndex) in getBoatPositions(boat.type)" :key="`occupied-${posIndex}`"
                    :class="{ 'occupied': pos.occupied }">●</span>
                </span>
                <span class="track-info">P(>13)={{ (calculateOver13Probability(boat) * 100).toFixed(1) }}%</span>
                <span class="track-info">P(=13)={{ (calculateStopAt13Probability(boat) * 100).toFixed(1) }}%</span>
                <span class="track-info">P(>12)={{ (calculateOver12Probability(boat) * 100).toFixed(1) }}%</span>
              </div>

              <div class="track-slots">
                <div class="track-slot" v-for="i in 14" :key="i - 1" :class="{ 'boat-here': i - 1 === boat.position }">
                  {{ i - 1 }}
                </div>
              </div>


            </div>
          </div>

        </div>

        <div class="profit-section">
          <h3>收益期望分析</h3>

          <div class="profit-tables">
            <div class="profit-table" v-if="visibleBoats.length > 0">
              <h4>平底船位置收益</h4>
              <table>
                <thead>
                  <tr>
                    <th>货物</th>
                    <th>位置</th>
                    <th>成本</th>
                    <th>分成收益</th>
                    <th>到达概率</th>
                    <th>期望收益</th>
                    <th>状态</th>
                  </tr>
                </thead>
                <tbody>
                  <template v-for="(boat, boatIndex) in visibleBoats" :key="boatIndex">
                    <tr v-for="(pos, posIndex) in getBoatPositions(boat.type)" :key="`${boatIndex}-${posIndex}`" :class="{
                      'high-profit': !pos.occupied && calculateExpectedProfit(boat, pos) > 0,
                      'occupied': pos.occupied
                    }">
                      <td>{{ boat.name }}</td>
                      <td>{{ posIndex + 1 }}</td>
                      <td>{{ pos.cost }}比索</td>
                      <td>{{ calculateProfit(boat, pos).toFixed(1) }}比索</td>
                      <td>{{ (calculateOver13Probability(boat) * 100).toFixed(0) }}%</td>
                      <td>{{ calculateExpectedProfit(boat, pos).toFixed(1) }}比索</td>
                      <td>{{ pos.occupied ? '已占用' : '可选择' }}</td>
                    </tr>
                  </template>
                </tbody>
              </table>
            </div>

            <div class="profit-table">
              <h4>港口和造船厂收益</h4>
              <table>
                <thead>
                  <tr>
                    <th>位置</th>
                    <th>成本</th>
                    <th>收益</th>
                    <th>到达概率</th>
                    <th>期望收益</th>
                    <th>状态</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="(port, portIndex) in portYardPositions" :key="portIndex" :class="{
                    'high-profit': !port.occupied && calculatePortExpectedProfit(port) > 0,
                    'occupied': port.occupied
                  }">
                    <td>{{ port.name }}</td>
                    <td>{{ port.cost }}比索</td>
                    <td>{{ port.profit }}比索</td>
                    <td>{{ (calculatePortArrivalProbability(port) * 100).toFixed(0) }}%</td>
                    <td>{{ calculatePortExpectedProfit(port).toFixed(1) }}比索</td>
                    <td>{{ port.occupied ? '已占用' : '可选择' }}</td>
                  </tr>
                </tbody>
              </table>
            </div>

            <div class="profit-table">
              <h4>特殊位置收益</h4>
              <table>
                <thead>
                  <tr>
                    <th>位置</th>
                    <th>成本</th>
                    <th>收益</th>
                    <th>触发概率</th>
                    <th>期望收益</th>
                    <th>状态</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="(pos, index) in specialPositions" :key="index" :class="{
                    'high-profit': !pos.occupied && calculateSpecialExpectedProfit(pos) > 0,
                    'occupied': pos.occupied
                  }">
                    <td>{{ pos.name }}</td>
                    <td>{{ pos.cost }}比索</td>
                    <td>{{ calculateSpecialProfit(pos).toFixed(1) }}比索</td>
                    <td>{{ (calculateSpecialProbability(pos) * 100).toFixed(0) }}%</td>
                    <td>{{ calculateSpecialExpectedProfit(pos).toFixed(1) }}比索</td>
                    <td>{{ pos.occupied ? '已占用' : '可选择' }}</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, reactive } from 'vue';

// 定义骰子轮次选项
const diceRounds = [
  { label: '第一次投掷前', value: 1 },
  { label: '第二次投掷前', value: 2 },
  { label: '第三次投掷前', value: 3 }
];
const currentRound = ref(1);

// 计算剩余骰子次数
const remainingDiceThrows = computed(() => {
  return 4 - currentRound.value;  // 1,2,3 轮对应 3,2,1 次投掷机会
});

// 获取当前轮次标签
const getCurrentRoundLabel = () => {
  return diceRounds.find(round => round.value === currentRound.value)?.label || '第一次投掷前';
};

// 定义船只数据
const boats = ref([
  { name: '人参', type: 'ginseng', position: 0, isSelected: true, profit: 18 },
  { name: '丝绸', type: 'silk', position: 0, isSelected: true, profit: 30 },
  { name: '肉豆蔻', type: 'nutmeg', position: 0, isSelected: true, profit: 24 },
  { name: '玉石', type: 'jade', position: 0, isSelected: false, profit: 36 }
]);

// 定义各船位置及成本，加入是否已占用的状态
const boatPositionsData = reactive({
  ginseng: [
    { cost: 1, occupied: false },
    { cost: 2, occupied: false },
    { cost: 3, occupied: false }
  ],
  silk: [
    { cost: 3, occupied: false },
    { cost: 4, occupied: false },
    { cost: 5, occupied: false }
  ],
  nutmeg: [
    { cost: 2, occupied: false },
    { cost: 3, occupied: false },
    { cost: 4, occupied: false }
  ],
  jade: [
    { cost: 3, occupied: false },
    { cost: 4, occupied: false },
    { cost: 5, occupied: false },
    { cost: 5, occupied: false }
  ]
});

// 定义港口和造船厂位置
const portYardPositions = ref([
  { name: 'A 港口', type: 'port', position: 'A', cost: 4, profit: 6, occupied: false },
  { name: 'B 港口', type: 'port', position: 'B', cost: 3, profit: 8, occupied: false },
  { name: 'C 港口', type: 'port', position: 'C', cost: 2, profit: 15, occupied: false },
  { name: 'A 造船厂', type: 'yard', position: 'A', cost: 4, profit: 6, occupied: false },
  { name: 'B 造船厂', type: 'yard', position: 'B', cost: 3, profit: 8, occupied: false },
  { name: 'C 造船厂', type: 'yard', position: 'C', cost: 2, profit: 15, occupied: false }
]);

// 定义特殊位置
const specialPositions = ref([
  { name: '海盗船长', type: 'pirate', position: 'captain', cost: 5, profit: 0, occupied: false },
  { name: '海盗船员', type: 'pirate', position: 'crew', cost: 5, profit: 0, occupied: false },
  { name: '大领航员', type: 'pilot', position: 'major', cost: 5, profit: 0, occupied: false },
  { name: '小领航员', type: 'pilot', position: 'minor', cost: 2, profit: 0, occupied: false },
  { name: '保险员', type: 'insurance', position: 'insurance', cost: 0, profit: 10, occupied: false }
]);


interface Boat {
  name: string;
  type: string;
  position: number;
  isSelected: boolean;
  profit: number;
}

interface Position {
  cost: number;
  occupied: boolean;
}

interface PortYardPosition {
  name: string;
  type: string;
  position: string;
  cost: number;
  profit: number;
  occupied: boolean;
}

interface SpecialPosition {
  name: string;
  type: string;
  position: string;
  cost: number;
  profit: number;
  occupied: boolean;
}

// 计算可见的船只（已选择的）
const visibleBoats = computed<Boat[]>(() => {
  return boats.value.filter(boat => boat.isSelected);
});

// 获取船只位置设置
const getBoatPositions = (type: string): Position[] => {
  return boatPositionsData[type as keyof typeof boatPositionsData] || [];
};

// 船只超过位置 n 概率动态规划表
// dp[i][t]: 从 i 位置剩 t 次骰子 > n 的概率
const getDpbyN = (n: number) => {
  const dp: number[][] = Array.from(
    { length: n + 1 },
    () => Array.from({ length: 3 }, () => 0)
  );

  // 填充动态规划表
  for (let t = 0; t < 3; t++) {
    for (let i = 0; i <= n; i++) {
      for (let d = 1; d <= 6; d++) {
        let next = i + d;
        if (next > n) {
          // > n，到达终点
          dp[i][t] += 1 / 6;
        } else {
          // 小于 n，继续移动
          if (t > 0) {
            dp[i][t] += dp[next][t - 1] / 6; // 后续到达终点
          }
        }
      }
    }
  }
  return dp
}

// 原有的到达终点概率函数
const calculateOver13Probability = (boat: Boat): number => {
  const throwsLeft = remainingDiceThrows.value;
  let pos = boat.position;
  return getDpbyN(13)[pos][throwsLeft - 1]
};

const calculateOver12Probability = (boat: Boat): number => {
  const throwsLeft = remainingDiceThrows.value;
  let pos = boat.position;
  return getDpbyN(12)[pos][throwsLeft - 1]
};

// 计算停在 13 号格概率（用于海盗收益/船员收益）
const calculateStopAt13Probability = (boat: Boat): number => {
  const throwsLeft = remainingDiceThrows.value;
  let pos = boat.position;
  return getDpbyN(12)[pos][throwsLeft - 1]
    - getDpbyN(13)[pos][throwsLeft - 1] // 计算停在 13 的概率 
};

// 计算有多少个位置被占用
const countOccupiedPositions = (type: string): number => {
  const positions = getBoatPositions(type);
  return positions ? positions.filter(pos => pos.occupied).length : 0;
};

// 判断是否有海盗存在（用于计算平底船收益）
const hasPiratesActive = (): boolean => {
  return specialPositions.value.some(pos =>
    pos.type === 'pirate' && pos.occupied
  );
};

// 获取潜在收益
const getPotentialProfit = (boat: Boat, shareBase: number): number => {
  return boat.profit / shareBase;
};

// 计算货物平底船的收益（考虑海盗劫掠）
const calculateProfit = (boat: Boat, pos: Position): number => {
  // 获取此平底船已占用的位置数
  const occupied = countOccupiedPositions(boat.type);
  const shareBase = occupied + 1;  // +1 是因为将计算将占用的位置收益

  // 第三轮有海盗时，停在 13 号的船员无收益
  if (remainingDiceThrows.value === 1 && hasPiratesActive()) {
    const stop13Prob = calculateStopAt13Probability(boat);
    const normalProfit = getPotentialProfit(boat, shareBase);
    // 停在 13 号概率部分收益归 0，其余正常
    return normalProfit * (1 - stop13Prob);
  }
  return getPotentialProfit(boat, shareBase);
};

// 计算货物平底船的期望收益
const calculateExpectedProfit = (boat: Boat, pos: Position): number => {
  if (!boat.isSelected) return -pos.cost;
  // 若第三轮有海盗，停在 13 号的概率部分收益归 0
  const profit = calculateProfit(boat, pos);
  const probability = calculateOver13Probability(boat);
  return profit * probability - pos.cost;
};

// 计算港口/造船厂到达概率
const calculatePortArrivalProbability = (port: PortYardPosition): number => {
  if (visibleBoats.value.length === 0) return 0;

  // 港口的概率是到达终点的概率，造船厂的概率是无法到达终点的概率
  let totalProb = 0;

  // 统计港口总数和造船厂总数
  const portCount = Math.max(
    portYardPositions.value.filter(p => p.type === 'port' && !p.occupied).length,
    1
  );
  const yardCount = Math.max(
    portYardPositions.value.filter(p => p.type === 'yard' && !p.occupied).length,
    1
  );

  visibleBoats.value.forEach(boat => {
    if (port.type === 'port') {
      // 港口：船到达终点的概率
      totalProb += calculateOver13Probability(boat) / portCount;
    } else {
      // 造船厂：船未到达终点的概率
      totalProb += (1 - calculateOver13Probability(boat)) / yardCount;
    }
  });

  return Math.min(totalProb, 1); // 概率最大为 1
};

// 计算港口/造船厂期望收益（固定收益）
const calculatePortExpectedProfit = (port: PortYardPosition): number => {
  const probability = calculatePortArrivalProbability(port);
  return port.profit * probability - port.cost;
};

// 计算特殊位置概率
const calculateSpecialProbability = (pos: SpecialPosition): number => {
  if (pos.type === 'pirate') {
    // 只在第三轮有概率
    if (remainingDiceThrows.value !== 1) return 0;
    // 只要有船停在 13 号就触发
    let prob = 0;
    visibleBoats.value.forEach(boat => {
      prob += calculateStopAt13Probability(boat);
    });
    return Math.min(prob, 1);
  }
  if (pos.type === 'pilot') return 0;
  if (pos.type === 'insurance') {
    let allArriveProbability = 1;
    visibleBoats.value.forEach(boat => {
      allArriveProbability *= calculateOver13Probability(boat);
    });
    return 1 - allArriveProbability;
  }
  return 0;
};

// 计算特殊位置收益
const calculateSpecialProfit = (pos: SpecialPosition): number => {
  if (pos.occupied) return 0;
  if (pos.type === 'pirate') {
    // 只在第三轮有收益
    if (remainingDiceThrows.value !== 1) return 0;
    // 统计所有停在 13 号的船的收益
    let totalBooty = 0;
    visibleBoats.value.forEach(boat => {
      const stop13Prob = calculateStopAt13Probability(boat);
      let booty = 0;
      switch (boat.type) {
        case 'ginseng': booty = 18; break;
        case 'silk': booty = 30; break;
        case 'nutmeg': booty = 24; break;
        case 'jade': booty = 36; break;
      }
      totalBooty += booty * stop13Prob;
    });
    // 海盗船长和船员平分
    const pirates = specialPositions.value.filter(p => p.type === 'pirate' && !p.occupied);
    const pirateCount = pirates.length;
    if (pirateCount === 0) return 0;
    return totalBooty / pirateCount;
  }
  if (pos.type === 'insurance') {
    let potentialPayout = visibleBoats.value.reduce((total, boat) => {
      const failProb = 1 - calculateOver13Probability(boat);
      return total + 10 * failProb;
    }, 0);
    return pos.profit - potentialPayout;
  }
  if (pos.type === 'pilot') {
    return 0;
  }
  return pos.profit || 0;
};

// 计算特殊位置期望收益
const calculateSpecialExpectedProfit = (pos: SpecialPosition): number => {
  const probability = calculateSpecialProbability(pos);
  const profit = calculateSpecialProfit(pos);
  return profit * probability - pos.cost;
};
</script>

<style scoped>
.manila-calculator {
  width: 100%;
  max-width: 1800px;
  margin: 0 auto;
  box-sizing: border-box;
}

/* 添加顶部使用说明样式 */
.instructions-wrapper {
  width: 100%;
  margin-bottom: 20px;
}

.instructions {
  position: relative;
  background-color: #f8f9fa;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  padding: 15px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
}

.instruction-content {
  line-height: 1.6;
  font-size: 0.9em;
}

/* GitHub 角落链接样式 */
.github-link {
  position: absolute;
  top: 0;
  right: 0;
  border: 0;
  display: block;
  width: 40px;
  height: 40px;
  background-color: #f1f1f1;
  border-bottom-left-radius: 4px;
  text-decoration: none;
}

.github-icon {
  position: absolute;
  top: 8px;
  right: 8px;
  fill: #151513;
  display: flex;
  justify-content: center;
  align-items: center;
}

.github-icon svg {
  width: 24px;
  height: 24px;
}

.calculator-layout {
  display: flex;
  width: 100%;
  gap: 20px;
}

.input-panel,
.output-panel {
  box-sizing: border-box;
  min-width: 500px;
}

.input-panel {
  flex: 0 0 280px;
}

.output-panel {
  flex: 1;
}

.section {
  margin-bottom: 20px;
  background: #f9f9f9;
  border-radius: 8px;
  padding: 15px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
}

.dice-round {
  margin-bottom: 15px;
  background: white;
  border-radius: 6px;
  padding: 10px;
  border: 1px solid #eee;
}

.dice-round h4 {
  margin-top: 0;
  margin-bottom: 8px;
}

.round-selector {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.round-option {
  display: flex;
  align-items: center;
  gap: 5px;
  padding: 4px 8px;
  border-radius: 4px;
  background-color: #f5f5f5;
  cursor: pointer;
}

.round-option:hover {
  background-color: #e9e9e9;
}

.round-display {
  padding: 8px;
  background: #f0f8ff;
  border-radius: 4px;
  font-weight: 500;
  text-align: center;
  margin-bottom: 15px;
}

h2 {
  margin-top: 0;
  margin-bottom: 15px;
  color: #2c3e50;
  font-size: 1.5rem;
}

h3 {
  margin-top: 0;
  margin-bottom: 12px;
  color: #2c3e50;
  font-size: 1.2rem;
}

h4 {
  margin-bottom: 10px;
  font-size: 1rem;
}

.boats-setup {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.boat-item {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 10px;
  background-color: white;
}

.boat-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
  align-items: center;
  flex-wrap: wrap;
}

.boat-name {
  font-weight: bold;
  font-size: 1.1em;
}

.boat-toggles {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}

.checkbox-label {
  display: flex;
  align-items: center;
  gap: 5px;
  font-size: 0.9em;
}

.boat-name.ginseng {
  color: #E74C3C;
}

.boat-name.silk {
  color: #3498DB;
}

.boat-name.nutmeg {
  color: #F39C12;
}

.boat-name.jade {
  color: #2ECC71;
}

.position-control {
  margin-bottom: 5px;
}

.position-label {
  display: block;
  margin-bottom: 5px;
  font-size: 0.9em;
}

.position-control input[type="range"] {
  width: 100%;
  margin: 5px 0;
}

.position-track {
  display: flex;
  justify-content: space-between;
  font-size: 0.75em;
  color: #777;
}

.occupancy-group {
  margin-bottom: 15px;
  padding: 10px;
  border: 1px solid #eee;
  border-radius: 5px;
  background-color: white;
}

.occupancy-header {
  margin: 0 0 8px 0;
  font-size: 1em;
  font-weight: bold;
}

.occupancy-header.ginseng {
  color: #E74C3C;
}

.occupancy-header.silk {
  color: #3498DB;
}

.occupancy-header.nutmeg {
  color: #F39C12;
}

.occupancy-header.jade {
  color: #2ECC71;
}

.occupancy-slots {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.slot-label {
  display: flex;
  align-items: center;
  gap: 5px;
  font-size: 0.85em;
}

.port-yard-grid,
.special-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
}

.visualization-section,
.profit-section {
  background: #f9f9f9;
  border-radius: 8px;
  padding: 15px;
  margin-bottom: 20px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
}

.track-visualization {
  margin: 15px 0;
  border: 1px solid #eee;
  border-radius: 8px;
  padding: 15px;
  background-color: white;
}

.track {
  display: flex;
  flex-direction: column;
  /* 改为列式布局 */
  margin-bottom: 12px;
}

.track-header {
  display: flex;
  margin-bottom: 4px;
  grid-template-columns: auto 1fr auto;
  /* 头尾固定大小，中间伸展 */
}

.track-label {
  width: 100px;
  font-weight: bold;
}


.track-label.ginseng {
  color: #E74C3C;
}

.track-label.silk {
  color: #3498DB;
}

.track-label.nutmeg {
  color: #F39C12;
}

.track-label.jade {
  color: #2ECC71;
}

.track-info {
  width: 100px;
  font-size: small;
  color: #2c3e50;
}

.occupied {
  color: #ccc;
}

/* 调整 track-slots 使其占满整行 */
.track-slots {
  display: flex;
  width: 100%;
}

.track-slot {
  flex: 1;
  border: 1px solid #ddd;
  text-align: center;
  padding: 5px 2px;
  font-size: 11px;
}

.boat-here {
  background-color: #ffeb3b;
  font-weight: bold;
}

.profit-tables {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.profit-table {
  background: white;
  border-radius: 8px;
  padding: 12px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.profit-table h4 {
  margin-top: 0;
  color: #2c3e50;
  font-size: 1.1em;
  margin-bottom: 10px;
}

.profit-table table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.9em;
}

.profit-table th,
.profit-table td {
  border: 1px solid #eee;
  padding: 6px 8px;
  text-align: center;
}

.profit-table th {
  background-color: #f5f5f5;
  font-weight: bold;
}

.high-profit {
  background-color: rgba(46, 204, 113, 0.15);
}

.occupied {
  color: #ccc;
}

/* 响应式调整 */
@media (max-width: 900px) {
  .calculator-layout {
    flex-direction: column;
  }

  .input-panel,
  .output-panel {
    flex: none;
    width: 100%;
  }
}

@media (min-width: 1800px) {
  .manila-calculator {
    padding: 0 20px;
  }
}



.probability-table-container {
  margin: 10px 0 20px 0;
  overflow-x: auto;
}

.probability-table {
  width: 100%;
  border-collapse: collapse;
  background-color: white;
  border-radius: 6px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.probability-table th,
.probability-table td {
  padding: 8px 12px;
  border: 1px solid #eee;
  text-align: center;
}

.probability-table th {
  background-color: #f5f5f5;
  font-weight: bold;
}

.probability-table td.boat-name {
  font-weight: bold;
}

.probability-table td.boat-name.ginseng {
  color: #E74C3C;
}

.probability-table td.boat-name.silk {
  color: #3498DB;
}

.probability-table td.boat-name.nutmeg {
  color: #F39C12;
}

.probability-table td.boat-name.jade {
  color: #2ECC71;
}
</style>
