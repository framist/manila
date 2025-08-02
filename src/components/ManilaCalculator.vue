<template>
  <div class="manila-calculator">
    <div class="instructions-wrapper">
      <div class="instructions">
        <h3>使用说明</h3>
        <div class="instruction-content">
          <p>此计算器帮助玩家计算《马尼拉》桌游中各放置帮手位置的期望收益，辅助决策。
            <br />注意：计算的期望是「如选择这个位置」下的期望
          </p>
          <p><strong>假设与村规：</strong></p>
          <ol>
            <li><strong>假设</strong>：海盗劫掠第一艘能上的船</li>
            <li><strong>假设</strong>：基于后期不会有改变价值的位置占据状况被改变</li>
            <li><strong>村规</strong>：海盗对无法影响第二轮移动结束时位于 13 号格子的平底船</li>
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
        <div class="section boat-settings">
          <h3>游戏状态设置</h3>
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
                <div class="boat-toggles">
                  <label class="checkbox-label">
                    <input type="checkbox" v-model="boat.isSelected" />
                    <span class="boat-name" :class="boat.type">{{ boat.name }}</span>
                  </label>
                </div>
                <div class="occupancy-slots">
                  位置：
                  <label v-for="(pos, posIndex) in getBoatPositions(boat.type)" :key="`pos-${posIndex}`"
                    class="slot-label">
                    <input type="checkbox" v-model="pos.occupied" />
                    {{ pos.cost }}比索
                  </label>
                </div>
              </div>

              <div class="track-header">
                <span class="track-info">船只位置：{{ boat.position }}</span>
                <span class="track-info">P(>13)={{ (calculateOver13Probability(boat) * 100).toFixed(1) }}%</span>
                <span class="track-info">P(=13)={{ (calculateStopAt13Probability(boat) * 100).toFixed(1) }}%</span>
                <span class="track-info">P(>12)={{ (calculateOver12Probability(boat) * 100).toFixed(1) }}%</span>
              </div>

              <div class="track-slots">
                <div class="track-slot" v-for="i in 14" :key="i - 1" :class="{ 'boat-here': i - 1 === boat.position }">
                  {{ i - 1 }}
                </div>
              </div>
              <div class="position-control">
                <input type="range" v-model.number="boat.position" :min="0" :max="13" />
              </div>
            </div>
          </div>
        </div>

        <div class="section special-settings">

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
            <h4>特殊位置已占用位置</h4>
            <div class="occupancy-group">
              <div class="occupancy-slots special-grid">
                <label v-for="(pos, posIndex) in specialPositions" :key="`special-${posIndex}`" class="slot-label">
                  <input type="checkbox" v-model="pos.occupied" />
                  {{ pos.name }} ({{ pos.cost }}比索)
                </label>
              </div>
            </div>


            <div class="boats-setup">
              <div class="position-control">
                <label for="pirate-probability">设置可能存在海盗的概率概率：</label>
                <span>{{ (pirateProbabilityForced * 100).toFixed(0) }}%</span>
                <span v-if="hasPiratesActive()">（实际海盗已存在）</span>
                <input type="range" v-model.number="pirateProbability" :min="0" :max="1" step="0.01"
                  :disabled="hasPiratesActive()" />
              </div>
            </div>

          </div>
        </div>
      </div>

      <!-- 输出区域 - 右侧 -->
      <div class="output-panel">
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
                    <th>期望净收益</th>
                    <th>状态</th>
                  </tr>
                </thead>
                <tbody>
                  <template v-for="(boat, boatIndex) in visibleBoats" :key="boatIndex">
                    <tr v-for="(pos, posIndex) in getBoatPositions(boat.type)" :key="`${boatIndex}-${posIndex}`" :class="{
                      'high-profit': !pos.occupied && calculateExpectedProfit(boat, pos) > 0,
                      'higher-profit': !pos.occupied && calculateExpectedProfit(boat, pos) > 2,
                      'isAvailable': pos.occupied
                    }">
                      <td>{{ boat.name }}</td>
                      <td>{{ posIndex + 1 }}</td>
                      <td>{{ pos.cost }}比索</td>
                      <td>{{ calculateMaxProfit(boat, pos).toFixed(1) }}比索</td>
                      <td>{{ (calculateArrivalProbability(boat) * 100).toFixed(0) }}%</td>
                      <td>{{ calculateExpectedProfit(boat, pos).toFixed(1) }}比索</td>
                      <td><input type="checkbox" v-model="pos.occupied" /></td>
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
                    <th>期望净收益</th>
                    <th>状态</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="(pos, portIndex) in portYardPositions" :key="portIndex" :class="{
                    'high-profit': !pos.occupied && calculatePortExpectedProfit(pos) > 0,
                    'higher-profit': !pos.occupied && calculatePortExpectedProfit(pos) > 2,
                    'isAvailable': pos.occupied
                  }">
                    <td>{{ pos.name }}</td>
                    <td>{{ pos.cost }}比索</td>
                    <td>{{ pos.profit }}比索</td>
                    <td>{{ (calculatePortArrivalProbability(pos) * 100).toFixed(0) }}%</td>
                    <td>{{ calculatePortExpectedProfit(pos).toFixed(1) }}比索</td>
                    <td><input type="checkbox" v-model="pos.occupied" /></td>
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
                    <th>期望净收益</th>
                    <th>状态</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="(pos, index) in specialPositions" :key="index" :class="{
                    'high-profit': !pos.occupied && calculateSpecialExpectedProfit(pos) > 0,
                    'higher-profit': !pos.occupied && calculateSpecialExpectedProfit(pos) > 2,
                    'isAvailable': pos.occupied
                  }">
                    <td>{{ pos.name }}</td>
                    <td>{{ pos.cost }}比索</td>
                    <td>{{ calculateSpecialExpectedProfit(pos).toFixed(1) }}比索</td>
                    <td><input type="checkbox" v-model="pos.occupied" /></td>
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

// 无海盗时可能存在海盗的概率
const pirateProbability = ref(0.5);

// 如果海盗存在，强制设置为 1
const pirateProbabilityForced = computed(() => {
  return hasPiratesActive() ? 1 : pirateProbability.value;
});

// 计算剩余骰子次数
const remainingDiceThrows = computed(() => {
  return 4 - currentRound.value;  // 1,2,3 轮对应 3,2,1 次投掷机会
});

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
  { name: '大领航员', type: 'pilot', position: 'major', cost: 5, profit: 0, occupied: false },
  { name: '小领航员', type: 'pilot', position: 'minor', cost: 2, profit: 0, occupied: false },
  { name: '保险员', type: 'insurance', position: 'insurance', cost: 0, profit: 10, occupied: false },
  { name: '海盗船长', type: 'pirate', position: 'captain', cost: 5, profit: 0, occupied: false },
  { name: '海盗船员', type: 'pirate', position: 'crew', cost: 5, profit: 0, occupied: false }
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

// todo 限制船只上的位置只能从前往后占领

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

const calculateOver13Probability = (boat: Boat): number => {
  const throwsLeft = remainingDiceThrows.value;
  let pos = boat.position;
  return getDpbyN(13)[pos][throwsLeft - 1]
};

const calculateOver12Probability = (boat: Boat): number => {
  const throwsLeft = remainingDiceThrows.value;
  let pos = boat.position;
  if (pos >= 12) return 1; // 已经在 12 号格以上
  return getDpbyN(12)[pos][throwsLeft - 1]
};

// 计算停在 13 号格概率（用于海盗收益/船员收益）
const calculateStopAt13Probability = (boat: Boat): number => {
  const throwsLeft = remainingDiceThrows.value;
  let pos = boat.position;
  if (pos >= 13) return 0;
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

// 计算货物平底船的最大分成收益
const calculateMaxProfit = (boat: Boat, pos: Position): number => {
  // 获取此平底船已占用的位置数
  const occupied = countOccupiedPositions(boat.type);
  const shareBase = occupied + 1;  // +1 是因为将计算将占用的位置收益
  // shareBase 最大不能超过该船位置数
  const maxShareBase = getBoatPositions(boat.type).length;
  const finalShareBase = Math.min(shareBase, maxShareBase);

  return boat.profit / finalShareBase;
};

// 考虑可能存在海盗下的到达概率
const calculateArrivalProbability = (boat: Boat): number => {
  if (hasPiratesActive()) {
    return calculateOver13Probability(boat);
  } else {
    return calculateOver12Probability(boat) * (1 - pirateProbabilityForced.value)
      + pirateProbabilityForced.value * calculateOver13Probability(boat);
  }
};

// 计算货物平底船的期望净收益
const calculateExpectedProfit = (boat: Boat, pos: Position): number => {
  if (!boat.isSelected) return -pos.cost;
  let profit = calculateMaxProfit(boat, pos);
  const probability = calculateArrivalProbability(boat);
  return profit * probability - pos.cost;
};

/*
有以下独立二元事件的概率 P_1 P_2 ... P_n
如何计算有 m 个事件发生的概率？
*/
/**
 * 计算 n 个独立事件中恰好有 m 个发生的概率
 * @param probs  事件发生概率数组 P_1 … P_n
 * @param m      目标发生事件个数 (0 ≤ m ≤ n)
 * @returns      恰有 m 个事件发生的概率
 */
function probExactlyM(probs: number[], m: number): number {
  if (m < 0 || m > probs.length) return 0;
  // dp[k] 表示已处理到当前事件时，恰有 k 个发生的概率
  const dp = new Array(m + 1).fill(0);
  dp[0] = 1;

  for (const p of probs) {
    // 逆序遍历防止覆盖上一轮值
    for (let k = Math.min(m, probs.length); k >= 0; k--) {
      dp[k] = dp[k] * (1 - p) + (k ? dp[k - 1] * p : 0);
    }
  }
  return dp[m];
}


// 计算 n 个船能安全到达终点的期望
const calculateSafeArrivalProbability = (n: number): number => {
  const probs = visibleBoats.value.map(boat => calculateArrivalProbability(boat));
  return probExactlyM(probs, n);
};

// 计算港口/造船厂到达概率
const calculatePortArrivalProbability = (port: PortYardPosition): number => {
  if (visibleBoats.value.length === 0) return 0;

  // 港口的概率是到达终点的概率，造船厂的概率是无法到达终点的概率
  // 船只优先停入靠前的港口/造船厂
  let p = [
    calculateSafeArrivalProbability(0),
    calculateSafeArrivalProbability(1),
    calculateSafeArrivalProbability(2),
    calculateSafeArrivalProbability(3)
  ];

  if (port.type === 'port') {
    if (port.position === 'A') {
        return p[1] + p[2] + p[3];
    }
    if (port.position === 'B') {
        return p[2] + p[3];
    }
    if (port.position === 'C') {
        return p[3];
    }
  }
  if (port.type === 'yard') {
    if (port.position === 'A') {
        return p[0] + p[1] + p[2];
    }
    if (port.position === 'B') {
        return p[0] + p[1];
    }
    if (port.position === 'C') {
        return p[0];
    }
  }
  return 0;
};

// 计算港口/造船厂期望收益（固定收益）
const calculatePortExpectedProfit = (port: PortYardPosition): number => {
  const probability = calculatePortArrivalProbability(port);
  return port.profit * probability - port.cost;
};


// 计算特殊位置期望收益
const calculateSpecialProfit = (pos: SpecialPosition): number => {
  if (pos.type === 'pirate') {
    // 统计所有停在 13 号的船的收益
    let totalBooty = 0;

    visibleBoats.value.forEach(boat => {
      // prob *= 1 - calculateStopAt13Probability(boat);
      totalBooty += calculateStopAt13Probability(boat) * boat.profit;
    });

    // 海盗船长和船员平分
    const pirates = specialPositions.value.filter(p => p.type === 'pirate' && p.occupied);
    const pirateCount = pirates.length;
    // 如果选择下的期望
    return totalBooty / Math.min(pirateCount + 1, 2);
  }
  if (pos.type === 'insurance') {
    // todo
    let booty = pos.profit;
    booty -= calculateSafeArrivalProbability(2) * 6
    booty -= calculateSafeArrivalProbability(1) * 8
    booty -= calculateSafeArrivalProbability(0) * 15
    return booty;
  }
  if (pos.type === 'pilot') {
    return 0;
  }
  return pos.profit || 0;
};

// 计算特殊位置期望净收益
const calculateSpecialExpectedProfit = (pos: SpecialPosition): number => {
  const profit = calculateSpecialProfit(pos);
  return profit - pos.cost;
};
</script>

<style>
:root {
  --bg-primary: #f9f9f9;
  --bg-secondary: #ffffff;
  --text-primary: #2c3e50;
  --text-secondary: #777;
  --border-color: #e0e0e0;
  --border-light: #eee;
  --shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
  --shadow-sm: 0 1px 3px rgba(0, 0, 0, 0.1);
  --highlight-low: rgba(46, 204, 113, 0.1);
  --highlight-high: rgba(46, 204, 113, 0.2);

  /* 船只颜色 */
  --ginseng-color: #b69100;
  --silk-color: #3498DB;
  --nutmeg-color: #8f4c00;
  --jade-color: #2ECC71;

  /* 交互元素 */
  --input-bg: #f5f5f5;
  --input-hover: #e9e9e9;
  --disabled-color: #ccc;
  --boat-highlight: #ffeb3b;
}

/* 深色模式适配 - 使用 data-theme 属性而不是媒体查询 */
[data-theme="dark"] {
  --bg-primary: #1e1e1e;
  --bg-secondary: #2d2d2d;
  --text-primary: #e0e0e0;
  --text-secondary: #aaa;
  --border-color: #444;
  --border-light: #555;
  --shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  --shadow-sm: 0 1px 3px rgba(0, 0, 0, 0.2);
  --highlight-low: rgba(46, 204, 113, 0.15);
  --highlight-high: rgba(46, 204, 113, 0.25);

  /* 深色模式下调整船只颜色以提高可见性 */
  --ginseng-color: #ffd54f;
  --silk-color: #64b5f6;
  --nutmeg-color: #ffab91;
  --jade-color: #81c784;

  /* 交互元素 */
  --input-bg: #333;
  --input-hover: #444;
  --disabled-color: #666;
  --boat-highlight: #ffea0070;
  /* 深色模式下船只位置高亮色 */
}

/* 确保深色模式下文本可见 */
[data-theme="dark"] .boat-here {
  color: #111;
  /* 深色模式下确保船位置的文字颜色可见 */
}

/* 深色模式下调整表单元素 */
[data-theme="dark"] input[type="checkbox"],
[data-theme="dark"] input[type="radio"] {
  filter: invert(80%) hue-rotate(180deg);
}

/* 其余样式保持不变 */
.manila-calculator {
  width: 100%;
  max-width: 1280px;
  margin: 0 auto;
  box-sizing: border-box;
  color: var(--text-primary);
}

/* 布局结构 */
.calculator-layout {
  display: flex;
  width: 100%;
  gap: 20px;
}

.input-panel {
  flex: 0 0 280px;
  min-width: 500px;
}

.output-panel {
  flex: 1;
  min-width: 500px;
}

/* 通用卡片样式 */
.section,
.instructions,
.profit-section {
  margin-bottom: 20px;
  background: var(--bg-primary);
  border-radius: 8px;
  padding: 15px;
  box-shadow: var(--shadow);
}

.instructions {
  position: relative;
  border: 1px solid var(--border-color);
}

.instructions-wrapper {
  width: 100%;
  margin-bottom: 20px;
}

.instruction-content {
  line-height: 1.6;
  font-size: 0.9em;
}

/* 通用内部容器样式 */
.dice-round,
.boat-item,
.occupancy-group,
.profit-table {
  background: var(--bg-secondary);
  border-radius: 6px;
  padding: 10px;
  border: 1px solid var(--border-light);
  margin-bottom: 12px;
}

/* 标题样式 */
h2,
h3,
h4 {
  color: var(--text-primary);
  margin-top: 0;
}

h3 {
  margin-bottom: 12px;
  font-size: 1.2rem;
}

h4 {
  margin-bottom: 10px;
  font-size: 1rem;
}

/* GitHub 链接样式 */
.github-link {
  position: absolute;
  top: 0;
  right: 0;
  width: 40px;
  height: 40px;
  background-color: var(--bg-secondary);
  border-bottom-left-radius: 4px;
  text-decoration: none;
}

.github-icon {
  position: absolute;
  top: 8px;
  right: 8px;
  fill: var(--text-primary);
  display: flex;
  justify-content: center;
  align-items: center;
}

.github-icon svg {
  width: 24px;
  height: 24px;
}

/* 轮次选择器 */
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
  background-color: var(--input-bg);
  cursor: pointer;
}

.round-option:hover {
  background-color: var(--input-hover);
}

/* 船只样式 */
.boats-setup {
  display: flex;
  flex-direction: column;
  gap: 12px;
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

/* 船只颜色 */
.boat-name.ginseng,
.track-label.ginseng {
  color: var(--ginseng-color);
}

.boat-name.silk,
.track-label.silk {
  color: var(--silk-color);
}

.boat-name.nutmeg,
.track-label.nutmeg {
  color: var(--nutmeg-color);
}

.boat-name.jade,
.track-label.jade {
  color: var(--jade-color);
}

/* 表单元素 */
.checkbox-label,
.slot-label {
  display: flex;
  align-items: center;
  gap: 5px;
  font-size: 0.85em;
}

.position-control {
  margin-bottom: 5px;
}

.position-control input[type="range"] {
  width: 100%;
  margin: 5px 0;
}

/* 位置网格布局 */
.occupancy-slots {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.port-yard-grid,
.special-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
}

/* 轨道样式 */
.track-header {
  display: flex;
  margin-bottom: 4px;
}

.track-info {
  width: 100px;
  font-size: small;
  color: var(--text-primary);
}

.track-slots {
  display: flex;
  width: 100%;
}

.track-slot {
  flex: 1;
  border: 1px solid var(--border-color);
  text-align: center;
  padding: 5px 2px;
  font-size: 11px;
}

.boat-here {
  background-color: var(--boat-highlight);
  font-weight: bold;
  color: #333;
  /* 确保深色背景下文字可见 */
}

/* 收益表格 */
.profit-tables {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.profit-table table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.9em;
}

.profit-table th,
.profit-table td {
  border: 1px solid var(--border-light);
  padding: 6px 8px;
  text-align: center;
}

.profit-table th {
  background-color: var(--bg-primary);
  font-weight: bold;
}

/* 高亮样式 */
.high-profit {
  background-color: var(--highlight-low);
}

.higher-profit {
  background-color: var(--highlight-high);
}

.isAvailable {
  color: var(--disabled-color);
}

/* 响应式调整 */
@media (max-width: 1280px) {
  .calculator-layout {
    flex-direction: column;
  }

  .input-panel,
  .output-panel {
    flex: none;
    width: 100%;
  }
}

@media (min-width: 1024px) {
  .manila-calculator {
    padding: 0 20px;
  }
}
</style>
