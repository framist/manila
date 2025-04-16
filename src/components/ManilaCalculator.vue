<template>
  <div class="manila-calculator">
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
                  <div class="track-mark" v-for="i in 14" :key="i-1">{{ i-1 }}</div>
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
                <label v-for="(pos, posIndex) in getBoatPositions(boat.type)" :key="`pos-${posIndex}`" class="slot-label">
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
          <h3>船只位置可视化</h3>
          <div class="track-visualization">
            <div class="track" v-for="(boat, index) in visibleBoats" :key="index">
              <div class="track-label" :class="boat.type">{{ boat.name }}</div>
              <div class="track-slots">
                <div class="track-slot" v-for="i in 14" :key="i-1" 
                     :class="{ 'boat-here': i-1 === boat.position }">
                  {{ i-1 }}
                </div>
              </div>
            </div>
          </div>
          <div class="round-display">
            当前设置：{{ getCurrentRoundLabel() }}，剩余骰子次数：{{ remainingDiceThrows }}
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
                    <tr v-for="(pos, posIndex) in getBoatPositions(boat.type)" :key="`${boatIndex}-${posIndex}`"
                        :class="{ 
                          'high-profit': !pos.occupied && calculateExpectedProfit(boat, pos) > 0,
                          'occupied': pos.occupied 
                        }">
                      <td>{{ boat.name }}</td>
                      <td>{{ posIndex + 1 }}</td>
                      <td>{{ pos.cost }}比索</td>
                      <td>{{ calculateProfit(boat, pos).toFixed(1) }}比索</td>
                      <td>{{ (calculateArrivalProbability(boat) * 100).toFixed(0) }}%</td>
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
                  <tr v-for="(port, portIndex) in portYardPositions" :key="portIndex"
                      :class="{ 
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
                  <tr v-for="(pos, index) in specialPositions" :key="index"
                      :class="{ 
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

        <div class="instructions">
          <h3>使用说明</h3>
          <div class="instruction-content">
            <p>此计算器帮助玩家计算《马尼拉》桌游中各放置帮手位置的期望收益，辅助决策。</p>
            <ol>
              <li><strong>当前轮次</strong>：选择当前是第几次骰子投掷前，会影响船只到达目的地的概率计算。</li>
              <li><strong>船只设置</strong>：调整滑块设置各船当前位置 (0-13)。</li>
              <li><strong>已占用位置</strong>：勾选已被占用的位置，系统会据此计算剩余位置的预期收益。</li>
              <li><strong>收益分析</strong>：表格中绿色背景表示正收益的位置，灰色表示已占用位置。</li>
            </ol>
            <p>平底船和海盗船收益由所有参与者（当前及已占用）平分，其他位置收益固定。</p>
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
  { name: '人参', type: 'ginseng', position: 0, isSelected: true },
  { name: '丝绸', type: 'silk', position: 0, isSelected: true },
  { name: '肉豆蔻', type: 'nutmeg', position: 0, isSelected: true },
  { name: '玉石', type: 'jade', position: 0, isSelected: false }
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

// 添加类型定义
interface Boat {
  name: string;
  type: string;
  position: number;
  isSelected: boolean;
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

// 计算船抵达目的地概率（基于骰子 1-6 均等概率和剩余投掷次数）
const calculateArrivalProbability = (boat: Boat): number => {
  const remainingDistance = 13 - boat.position;
  
  if (remainingDistance <= 0) return 1; // 已经到达
  
  // 根据剩余投掷次数计算概率
  const throwsLeft = remainingDiceThrows.value;
  
  if (throwsLeft === 0) return 0; // 没有投掷机会了
  
  // 精确计算单次骰子到达的概率
  const singleThrowProb = Math.min(Math.max(0, (7 - remainingDistance) / 6), 1);
  
  if (throwsLeft === 1) {
    return singleThrowProb; // 只有一次骰子机会
  }
  
  // 计算两次骰子的概率
  if (throwsLeft === 2) {
    // 一次到达的概率 + 第一次未到达但第二次到达的概率
    let secondProb = 0;
    
    // 第一次投掷每个点数的可能性
    for (let firstThrow = 1; firstThrow <= 6; firstThrow++) {
      if (firstThrow >= remainingDistance) {
        // 第一次就到达，概率 1/6
        secondProb += 1/6;
      } else {
        // 第一次未到达，计算第二次到达的概率
        const remainAfterFirst = remainingDistance - firstThrow;
        const secondThrowProb = Math.min(Math.max(0, (7 - remainAfterFirst) / 6), 1);
        secondProb += (1/6) * secondThrowProb;
      }
    }
    
    return secondProb;
  }
  
  // 计算三次骰子的概率（更精确的递归计算）
  if (throwsLeft === 3) {
    let thirdProb = 0;
    
    // 第一次投掷每个点数的可能性
    for (let firstThrow = 1; firstThrow <= 6; firstThrow++) {
      if (firstThrow >= remainingDistance) {
        // 第一次就到达，概率 1/6
        thirdProb += 1/6;
      } else {
        // 第一次未到达，计算余下两次的到达概率
        const newBoat = { ...boat, position: boat.position + firstThrow };
        const twoThrowProb = calculateArrivalProbabilityWithThrows(newBoat, 2);
        thirdProb += (1/6) * twoThrowProb;
      }
    }
    
    return thirdProb;
  }
  
  return 0; // 意外情况
};

// 辅助函数：计算特定投掷次数的到达概率
const calculateArrivalProbabilityWithThrows = (boat: Boat, throws: number): number => {
  const remainingDistance = 13 - boat.position;
  
  if (remainingDistance <= 0) return 1; // 已经到达
  if (throws === 0) return 0; // 没有投掷机会
  
  if (throws === 1) {
    // 单次骰子到达的概率
    return Math.min(Math.max(0, (7 - remainingDistance) / 6), 1);
  }
  
  // 递归计算两次投掷的概率
  let totalProb = 0;
  
  // 考虑第一次投掷的所有可能性
  for (let firstThrow = 1; firstThrow <= 6; firstThrow++) {
    if (firstThrow >= remainingDistance) {
      // 第一次就到达
      totalProb += 1/6;
    } else {
      // 第一次未到达，计算余下次数的到达概率
      const newBoat = { ...boat, position: boat.position + firstThrow };
      totalProb += (1/6) * calculateArrivalProbabilityWithThrows(newBoat, throws - 1);
    }
  }
  
  return totalProb;
};

// 计算有多少个位置被占用
const countOccupiedPositions = (type: string): number => {
  const positions = getBoatPositions(type);
  return positions ? positions.filter(pos => pos.occupied).length : 0;
};

// 计算总位置数
const countTotalPositions = (type: string): number => {
  const positions = getBoatPositions(type);
  return positions ? positions.length : 0;
};

// 判断是否有海盗存在（用于计算平底船收益）
const hasPiratesActive = (): boolean => {
  return specialPositions.value.some(pos => 
    pos.type === 'pirate' && !pos.occupied
  );
};

// 计算货物平底船的收益（考虑已占用和将占用的位置）
const calculateProfit = (boat: Boat, pos: Position): number => {
  if (pos.occupied) return 0; // 已占用位置无收益
  
  // 获取此平底船已占用的位置数
  const occupied = countOccupiedPositions(boat.type);
  
  // 获取分成基数 (已占用 + 当前位置计算)
  const shareBase = occupied + 1;

  // 第三轮投骰子后在 13 号位置会被海盗打劫的风险
  if (remainingDiceThrows.value === 1 && hasPiratesActive()) {
    // 计算停在 13 号格子的概率
    let stop13Prob = 0;
    
    if (boat.position === 13) {
      stop13Prob = 1; // 已在 13 格
    } else if (boat.position >= 7 && boat.position <= 12) {
      stop13Prob = 1/6; // 固定点数概率
    }
    
    // 如果有被海盗打劫的风险，计算收益减少
    if (stop13Prob > 0) {
      const normalProfit = getPotentialProfit(boat.type, shareBase);
      return normalProfit * (1 - stop13Prob); // 根据被打劫概率减少收益
    }
  }
  
  return getPotentialProfit(boat.type, shareBase);
};

// 获取潜在收益
const getPotentialProfit = (boatType: string, shareBase: number): number => {
  switch (boatType) {
    case 'ginseng': return 18 / shareBase;
    case 'silk': return 30 / shareBase;
    case 'nutmeg': return 24 / shareBase;
    case 'jade': return 36 / shareBase;
    default: return 0;
  }
};

// 计算货物平底船的期望收益
const calculateExpectedProfit = (boat: Boat, pos: Position): number => {
  if (pos.occupied) return 0; // 已占用位置无收益
  if (!boat.isSelected) return -pos.cost; // 未选中船只无收益
  
  const profit = calculateProfit(boat, pos);
  const probability = calculateArrivalProbability(boat);
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
      totalProb += calculateArrivalProbability(boat) / portCount;
    } else {
      // 造船厂：船未到达终点的概率
      totalProb += (1 - calculateArrivalProbability(boat)) / yardCount;
    }
  });
  
  return Math.min(totalProb, 1); // 概率最大为 1
};

// 计算港口/造船厂期望收益（固定收益）
const calculatePortExpectedProfit = (port: PortYardPosition): number => {
  // if (port.occupied) return 0; // 已占用位置无收益
  
  const probability = calculatePortArrivalProbability(port);
  return port.profit * probability - port.cost;
};

// 计算特殊位置概率
const calculateSpecialProbability = (pos: SpecialPosition): number => {
  // if (pos.occupied) return 0; // 已占用位置无收益
  
  // 海盗的概率是有船停在 13 格的概率
  if (pos.type === 'pirate') {
    let pirateProb = 0;
    const throwsLeft = remainingDiceThrows.value;
    
    if (throwsLeft === 1) {
      // 对于第三次投掷前，计算停在 13 号格子的确切概率
      visibleBoats.value.forEach(boat => {
        // 当前已在 13 号格子
        if (boat.position === 13) {
          pirateProb += 1;
        } 
        // 计算恰好停在 13 号格子的概率
        else if (boat.position >= 7 && boat.position <= 12) {
          // 需要掷出特定点数才能到达13号格子
          const targetDice = 13 - boat.position;
          pirateProb += 1/6; // 每个目标点数的概率是1/6
        }
      });
    } else if (throwsLeft === 2) {
      // 对于第二次投掷前，计算第二次投掷后停在13号格子的概率
      visibleBoats.value.forEach(boat => {
        if (boat.position === 13) {
          // 已在 13 号格子，但第二次投掷可能会将船移出
          // 停留在 13 号的概率为 0（游戏规则要求必须移动骰子点数）
          pirateProb += 0;
        } else {
          // 计算第一次投掷后可能到达的位置，然后计算从那些位置第二次投掷后恰好到达 13 号格子的概率
          let stopAt13Prob = 0;
          for (let dice1 = 1; dice1 <= 6; dice1++) {
            const intermediatePosition = boat.position + dice1;
            if (intermediatePosition === 13) {
              // 第一次投掷后就到达13号格子，在第二次投掷后必定移动
              // 根据规则，船必须移动，不能停留
              stopAt13Prob += 0;
            } else if (intermediatePosition < 13) {
              // 第一次投掷后未到达13号格子，计算第二次恰好到达的概率
              const targetDice = 13 - intermediatePosition;
              if (targetDice >= 1 && targetDice <= 6) {
                stopAt13Prob += (1/6) * (1/6); // 两次特定点数的概率
              }
            }
            // 如果第一次投掷后超过13，则不可能在第二次结束时在13号格子
          }
          pirateProb += stopAt13Prob;
        }
      });
    } else if (throwsLeft === 3) {
      // 对于第一次投掷前，使用精确计算第三次投掷结束时在13号格子的概率
      visibleBoats.value.forEach(boat => {
        // 这里需要考虑三次骰子投掷的所有可能组合
        let stopAt13Prob = 0;
        
        // 枚举第一次投掷的所有可能性
        for (let dice1 = 1; dice1 <= 6; dice1++) {
          const pos1 = boat.position + dice1;
          
          // 枚举第二次投掷的所有可能性
          for (let dice2 = 1; dice2 <= 6; dice2++) {
            const pos2 = pos1 + dice2;
            
            // 第三次投掷需要恰好到达13号格子
            const targetDice = 13 - pos2;
            if (targetDice >= 1 && targetDice <= 6) {
              stopAt13Prob += (1/6) * (1/6) * (1/6); // 三次特定点数的概率
            }
          }
        }
        
        pirateProb += stopAt13Prob;
      });
    }
    
    return Math.min(pirateProb, 1); // 概率最大为 1
  }
  
  // 领航员的有效性，粗略估计
  if (pos.type === 'pilot') {
    // 领航员只在第三轮投掷前有作用
    return currentRound.value === 3 ? 0.8 : 0.2;
  }
  
  // 保险的收益取决于有船进入造船厂的概率
  if (pos.type === 'insurance') {
    let allArriveProbability = 1;
    visibleBoats.value.forEach(boat => {
      allArriveProbability *= calculateArrivalProbability(boat);
    });
    return 1 - allArriveProbability; // 至少有一艘船未到达港口的概率
  }
  
  return 0;
};

// 计算特殊位置收益
const calculateSpecialProfit = (pos: SpecialPosition): number => {
  if (pos.occupied) return 0; // 已占用位置无收益
  
  // 海盗收益估算（考虑共享收益）
  if (pos.type === 'pirate') {
    // 统计未被占用的海盗位置数量
    const piratePositions = specialPositions.value.filter(p => p.type === 'pirate');
    const occupiedPirates = piratePositions.filter(p => p.occupied).length;
    const totalPirates = piratePositions.length;
    
    // 计算共享基数（当前 + 已占用）
    const shareBase = occupiedPirates + 1;
    
    // 估算可能掠夺的船只的平均收益（第三轮投骰子后）
    let potentialBooty = 0;
    
    if (remainingDiceThrows.value === 1) {
      // 第三轮投骰子后，分析可能停在 13 号格子的船只
      visibleBoats.value.forEach(boat => {
        let stopAt13Prob = 0;
        
        if (boat.position === 13) {
          stopAt13Prob = 1;
        } else if (boat.position >= 7 && boat.position <= 12) {
          stopAt13Prob = 1/6;
        }
        
        // 若船停在13号格子，海盗可获得船上的全部收益
        if (stopAt13Prob > 0) {
          switch (boat.type) {
            case 'ginseng': potentialBooty += 18 * stopAt13Prob; break;
            case 'silk': potentialBooty += 30 * stopAt13Prob; break;
            case 'nutmeg': potentialBooty += 24 * stopAt13Prob; break;
            case 'jade': potentialBooty += 36 * stopAt13Prob; break;
          }
        }
      });
    } else {
      // 其他轮次使用平均估算
      potentialBooty = visibleBoats.value.reduce((total, boat) => {
        switch (boat.type) {
          case 'ginseng': return total + 18;
          case 'silk': return total + 30;
          case 'nutmeg': return total + 24;
          case 'jade': return total + 36;
        }
        return total;
      }, 0) / Math.max(visibleBoats.value.length, 1) * 0.3; // 调整掠夺概率因子
    }
    
    // 海盗船长和船员的收益比例
    if (pos.position === 'captain') {
      const captainOccupied = specialPositions.value.find(p => p.type === 'pirate' && p.position === 'captain')?.occupied;
      if (captainOccupied) return 0;
      
      const crewActive = specialPositions.value.some(p => p.type === 'pirate' && p.position === 'crew' && !p.occupied);
      return crewActive ? (potentialBooty * 0.7) : potentialBooty; // 船长拿更多
    } else {
      const captainActive = specialPositions.value.some(p => p.type === 'pirate' && p.position === 'captain' && !p.occupied);
      return captainActive ? (potentialBooty * 0.3) : potentialBooty; // 船员拿更少
    }
  }
  
  // 保险员固定收益
  if (pos.type === 'insurance') {
    // 计算可能需要赔付的金额
    const potentialPayout = visibleBoats.value.reduce((total, boat) => {
      // 计算船只未到达港口的概率及相应赔付
      const failProb = 1 - calculateArrivalProbability(boat);
      return total + 10 * failProb; // 假设赔付 10 比索
    }, 0);
    
    // 保险收益 = 初始获得 - 预期赔付
    return pos.profit - potentialPayout;
  }
  
  // 领航员无直接收益
  if (pos.type === 'pilot') {
    return 0;
  }
  
  // 其他位置使用固定收益
  return pos.profit || 0;
};

// 计算特殊位置期望收益
const calculateSpecialExpectedProfit = (pos: SpecialPosition): number => {
  if (pos.occupied) return 0; // 已占用位置无收益
  
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

.calculator-layout {
  display: flex;
  width: 100%;
  gap: 20px;
}

.input-panel, .output-panel {
  box-sizing: border-box;
  min-width: 500px;
}

.input-panel {
  flex: 0 0 320px;
  min-width: 280px;
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
  margin-top: 10px;
  padding: 8px;
  background: #f0f8ff;
  border-radius: 4px;
  font-weight: 500;
  text-align: center;
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

.boat-name.ginseng { color: #E74C3C; }
.boat-name.silk { color: #3498DB; }
.boat-name.nutmeg { color: #F39C12; }
.boat-name.jade { color: #2ECC71; }

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

.occupancy-header.ginseng { color: #E74C3C; }
.occupancy-header.silk { color: #3498DB; }
.occupancy-header.nutmeg { color: #F39C12; }
.occupancy-header.jade { color: #2ECC71; }

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

.port-yard-grid, .special-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
}

.visualization-section, .profit-section {
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
  align-items: center;
  margin-bottom: 15px;
}

.track:last-child {
  margin-bottom: 0;
}

.track-label {
  width: 60px;
  font-weight: bold;
}

.track-label.ginseng { color: #E74C3C; }
.track-label.silk { color: #3498DB; }
.track-label.nutmeg { color: #F39C12; }
.track-label.jade { color: #2ECC71; }

.track-slots {
  display: flex;
  flex-grow: 1;
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
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
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

.profit-table th, .profit-table td {
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
  background-color: #f0f0f0;
  color: #999;
}

.instructions {
  background-color: #f8f9fa;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  padding: 15px;
}

.instruction-content {
  line-height: 1.6;
  font-size: 0.9em;
}

/* 响应式调整 */
@media (max-width: 900px) {
  .calculator-layout {
    flex-direction: column;
  }
  
  .input-panel, .output-panel {
    flex: none;
    width: 100%;
  }
}

@media (min-width: 1800px) {
  .manila-calculator {
    padding: 0 20px;
  }
}
</style>
