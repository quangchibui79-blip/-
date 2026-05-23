# 我是破烂王 - 修改日志

## 版本历史

---

## v1.2 (2026-05-23)

### 变更1：合成区域宽度调整
**文件**: `css/styles.css`

**修改内容**:
```css
.synthesis-area {
    min-width: 400px;  /* 从200px增加 */
    flex: 2;           /* 新增 */
}
```

**变更原因**:
- 提供更好的视觉平衡和用户体验
- 为合成槽提供更多展示空间

---

### 变更2：收购商弹窗拖拽修复
**文件**: 
- `css/styles.css`
- `js/main.js`
- `js/cards.js`

**修改内容**:

1. **css/styles.css** - 调整弹窗层级
```css
.modal-overlay {
    /* 移除 pointer-events: none */
}

.modal-content {
    position: relative;
    z-index: 2001;  /* 新增 */
}
```

2. **js/main.js** - 移除不兼容的HTML5拖拽事件
```javascript
// 移除 setupMerchantSlotEvents 函数
// 移除对 setupMerchantSlotEvents 的调用
```

3. **js/cards.js** - 添加收购商槽位高亮
```javascript
// 新增 highlightMerchantSlot 函数
function highlightMerchantSlot(highlight) {
    const merchantSlot = document.getElementById('merchant-slot');
    if (merchantSlot) {
        if (highlight) {
            merchantSlot.classList.add('slot-active');
        } else {
            merchantSlot.classList.remove('slot-active');
        }
    }
}

// 在 startDrag 中调用 highlightMerchantSlot(true)
// 在 cleanupDrag 中调用 highlightMerchantSlot(false)
```

**变更原因**:
- 修复bug：弹窗打开后无法拖拽卡牌到收购商槽位
- HTML5原生拖拽事件与自定义鼠标事件系统不兼容

---

### 变更3：初始金钱调整
**文件**: `js/state.js`

**修改内容**:
```javascript
const GAME_CONFIG = {
    INITIAL_GOLD: 20,  // 从0改为20
    // ...
}
```

**变更原因**:
- 让玩家可以立即体验垃圾桶盲盒功能
- 降低游戏入门门槛，提高新手友好度

---

## v1.1 (2026-05-23)

### 变更1：合成区域卡牌显示逻辑优化
**文件**: 
- `js/cards.js`
- `index.html`

**修改内容**:

1. **js/cards.js**
```javascript
// renderBackpack 函数 - 跳过已在合成槽中的卡牌
function renderBackpack() {
    gameState.cards.forEach(card => {
        if (synthesisSlot.cardId === card.id) return;
        // ...
    });
}

// 新增 returnCardToBackpack 函数
function returnCardToBackpack() {
    if (synthesisSlot.cardId) {
        showMessage('已取消合成，卡牌返回背包', 'info');
        clearSynthesisSlot();
        renderBackpack();
    }
}

// synthesisSlot 结构更新
let synthesisSlot = {
    cardId: null,
    cardData: null  // 新增：存储卡牌完整数据
};
```

2. **index.html** - 添加提示文字
```html
<div class="synthesis-area">
    <p class="synthesis-hint">连续拖拽两张卡牌进行合成</p>
    <!-- ... -->
</div>
```

**变更原因**:
- 避免玩家混淆，明确显示哪些卡牌正在参与合成
- 防止同一卡牌被重复用于多个合成操作

---

### 变更2：游戏结束判定时机调整
**文件**: 
- `js/state.js`
- `js/synthesis.js`

**修改内容**:

1. **js/state.js**
```javascript
// 添加 checkEnd 参数（默认false）
function addCard(card, checkEnd = false) {
    gameState.cards.push(card);
    renderBackpack();
    if (checkEnd) checkGameOver();
}

function addCards(cards, checkEnd = false) {
    gameState.cards.push(...cards);
    renderBackpack();
    if (checkEnd) checkGameOver();
}

function removeCard(cardId, checkEnd = false) {
    // ...
    if (checkEnd) checkGameOver();
    return removedCard;
}
```

2. **js/synthesis.js**
```javascript
// 合成过程中不检查游戏结束
removeCard(card1.id, false);
removeCard(card2.id, false);
addCard(newCard, false);

// 合成完成后统一检查
checkGameOver();
```

**变更原因**:
- 避免合成过程中因临时卡牌数量减少而误判游戏结束
- 确保游戏结束判定基于稳定的游戏状态

---

## v1.0 (2026-05-23)

### 初始版本
**功能**:
- 基础数据模型（Card类、词库）
- 背包网格布局与拖拽合成
- 收购商售卖弹窗
- 垃圾桶盲盒购买
- 游戏结束检测

**文件结构**:
```
├── index.html
├── css/styles.css
└── js/
    ├── models.js
    ├── state.js
    ├── ai-service.js
    ├── cards.js
    ├── synthesis.js
    ├── merchant.js
    ├── trash-bin.js
    ├── game-over.js
    └── main.js
```

---

## v1.3 (2026-05-23)

### 变更6：收购商弹窗布局优化
**文件**: `css/styles.css`

**修改内容**:
```css
.modal-overlay {
    position: fixed;
    top: 80px;          /* 固定在顶部，不遮挡背包 */
    left: 50%;
    transform: translateX(-50%);
    pointer-events: none; /* 允许点击穿透到下方元素 */
}

.modal-overlay.modal-show {
    pointer-events: auto; /* 弹窗内容可交互 */
}

.modal-content {
    pointer-events: auto;
}
```

**变更原因**:
- 解决弹窗遮挡背包区域的问题
- 玩家可以正常选择要售卖的卡牌

---

### 变更7：卡牌拖拽性能优化
**文件**: `js/cards.js`

**修改内容**:
```javascript
// startDrag 函数
dragState.cloneElement.style.left = '0px';
dragState.cloneElement.style.top = '0px';
dragState.cloneElement.style.transform = `translate(${rect.left}px, ${rect.top}px)`;
dragState.cloneElement.style.willChange = 'transform';

// onDragMove 函数
const newX = clientX - dragState.offsetX;
const newY = clientY - dragState.offsetY;
dragState.cloneElement.style.transform = `translate(${newX}px, ${newY}px)`;
```

**变更原因**:
- 提升拖拽流畅度，使卡牌更跟手
- transform 使用GPU加速，性能更好

---

### 变更8：收购商弹窗拖拽区域隔离
**文件**: `js/cards.js`

**修改内容**:
```javascript
// startDrag 函数
if (!isMerchantModalOpen()) {
    highlightSynthesisArea(true);
} else {
    highlightMerchantSlot(true);
}

// onDragMove 函数
if (!isMerchantModalOpen()) {
    // 检查合成区域
}
```

**变更原因**:
- 避免玩家在售卖卡牌时误触发合成区域
- 明确区分售卖和合成两种操作场景

---

### 变更9：拖拽卡牌图层修复
**文件**: `js/cards.js`

**修改内容**:
```javascript
// startDrag 函数
dragState.cloneElement.style.zIndex = '3000'; /* 从1000提升，高于弹窗的2000 */
```

**变更原因**:
- 修复拖拽卡牌到收购商弹窗卡槽时，卡牌被弹窗遮挡的问题

---

### 变更10：背包区域卡槽背景替换
**文件**: `css/styles.css`

**修改内容**:
```css
.backpack {
    background: url('../image/背包区域卡槽.png') center/cover no-repeat;
}
```

**变更原因**:
- 使用自定义美术资源，提升游戏视觉效果

---

### 变更11：主页背景替换
**文件**: `css/styles.css`

**修改内容**:
```css
body {
    background: url('../image/主页背景.png') center/cover no-repeat fixed;
}
```

**变更原因**:
- 使用自定义美术资源，提升游戏视觉效果

---

### 变更12：背包区域简化
**文件**: `index.html`, `css/styles.css`

**修改内容**:
```html
<!-- index.html - 简化前 -->
<section class="backpack-section">
    <h2 class="backpack-title">🎒 我的破烂</h2>
    <div class="backpack" id="backpack"></div>
</section>

<!-- 简化后 -->
<div class="backpack" id="backpack"></div>
```

```css
/* styles.css - 删除的样式 */
.backpack-section { ... }
.backpack-title { ... }
```

**变更原因**:
- 简化UI，让界面更清爽
- 背景图已经提供了足够的视觉层次

---

### 变更13：背包卡牌位置调整
**文件**: `css/styles.css`

**修改内容**:
```css
.backpack {
    padding-top: 30px;  /* 调整顶部内边距，让卡牌在卡槽内正确位置 */
}
```

**变更原因**:
- 配合卡槽背景图，让卡牌显示在正确的位置

---

### 变更14：交互按钮图片替换
**文件**: `index.html`, `css/styles.css`

**修改内容**:
```html
<!-- 垃圾桶按钮 -->
<button class="trash-bin-btn">
    <img class="action-btn-img" src="image/垃圾桶.png" alt="垃圾桶">
    <span class="trash-bin-text">垃圾桶</span>
    <span class="trash-bin-price">20金币</span>
</button>

<!-- 收购商按钮 -->
<button class="merchant-btn">
    <img class="action-btn-img" src="image/收购商.png" alt="收购商">
    <span class="merchant-text">收购商</span>
</button>

<!-- 帮助按钮 -->
<button class="icon-btn help-btn">
    <img src="image/帮助.png" alt="帮助">
</button>

<!-- 重置游戏按钮 -->
<button class="icon-btn reset-btn">
    <img src="image/重置游戏.png" alt="重置游戏">
</button>
```

```css
/* 按钮背景改为透明，去掉边框 */
.icon-btn { background: transparent; border: none; }
.trash-bin-btn { background: transparent; border: none; }
.merchant-btn { background: transparent; border: none; }
.action-btn-img { width: 60px; height: 60px; object-fit: contain; }
```

**变更原因**:
- 使用自定义美术资源，提升游戏视觉效果

---

### 变更15：扩展垃圾词库更新
**文件**: `js/models.js`, `PRD.md`

**修改内容**:
```javascript
// EXTENDED_TRASH_NAMES 新增18个物品
'假牙',
'香蕉皮',
'放屁垫',
'毛线团',
'狗骨头',
'沃尔玛购物袋',
'邮票',
'弹簧',
'蟑螂',
'老鼠尸体',
'刀柄',
'断剑',
'鸡蛋壳',
'泡面叉子',
'假发',
'单只拖鞋',
'空打火机',
'遥控器'
```

**变更原因**:
- 丰富游戏内容，增加垃圾物品多样性

---

### 变更16：同类卡牌叠放功能
**文件**: `js/cards.js`, `css/styles.css`

**修改内容**:
```javascript
// renderBackpack 函数 - 按名字+合成次数分组
const groupKey = `${card.name}_${card.combineCount}`;

// createCardStack 函数 - 创建叠放容器
function createCardStack(cards) {
    // 最多显示3张卡牌的堆叠效果
    // 添加数量角标
    // 只有最上面的卡牌可拖拽
}
```

```css
.card-stack { position: relative; }
.stacked-card { transform: translate(4px, 4px); } /* 错位堆叠 */
.stack-badge { /* 数量角标样式 */ }
```

**变更原因**:
- 优化背包显示，减少卡牌占用空间
- 提升玩家整理卡牌的体验

---

### 变更17：收购商弹窗图片替换
**文件**: `index.html`, `css/styles.css`

**修改内容**:
```html
<!-- 标题去除emoji -->
<h2 class="modal-title">收购商</h2>

<!-- emoji替换为图片 -->
<img class="merchant-image" src="image/收购商弹窗.png" alt="收购商">
```

```css
.merchant-image {
    width: 100px;
    height: 100px;
    object-fit: contain;
    border-radius: 10px;
}
```

**变更原因**:
- 使用自定义美术资源，提升游戏视觉效果

---

### 变更18：垃圾桶价格与数量调整
**文件**: `js/state.js`, `js/trash-bin.js`, `index.html`

**修改内容**:
```javascript
// GAME_CONFIG
TRASH_BIN_PRICE: 15,     // 从20改为15
MAX_TRASH_BIN_REWARD: 4, // 从5改为4
```

**变更原因**: 降低购买成本，提升游戏体验

---

### 变更19：背包卡牌上限
**文件**: `js/state.js`, `js/trash-bin.js`

**修改内容**:
```javascript
// GAME_CONFIG 新增
MAX_BACKPACK_SIZE: 8

// state.js 新增函数
function getAvailableCardCount() { ... }

// trash-bin.js - 购买前检查背包空间
// trash-bin.js - 开盲盒后溢出退款
```

**变更原因**: 增加策略性，玩家需要合理安排卡牌

---

### 变更20：收购商按钮放大
**文件**: `index.html`, `css/styles.css`

**修改内容**:
```css
.merchant-btn-img { width: 80px; height: 80px; }
```

**变更原因**: 提升收购商按钮的视觉突出度

---

### 变更21：接入扣子AI Agent
**文件**: `js/ai-service.js`

**修改内容**:
```javascript
// 扣子Agent配置
const AI_CONFIG = {
    synthesizeAgent: {
        baseURL: "https://47z92js6s6.coze.site/stream_run",
        token: "...",
        project_id: "7643027786780688427"
    },
    evaluateAgent: {
        baseURL: "https://pjxtrgh8z6.coze.site/stream_run",
        token: "...",
        project_id: "7643038860234113062"
    }
};

// 调用扣子Agent
async function callCozeAgent(agentConfig, promptText) { ... }
```

**变更原因**: 使用扣子平台部署的AI Agent，实现更智能的合成命名和估价

---

## 修改统计

| 版本 | 修改文件数 | 新增功能 | Bug修复 |
|------|-----------|---------|---------|
| v1.0 | 10 | 5 | 0 |
| v1.1 | 4 | 2 | 0 |
| v1.2 | 4 | 1 | 2 |
| v1.3 | 10 | 12 | 4 |

---

## 文档维护说明

### 自动记录机制

**自2026-05-23起，本日志将自动记录所有需求变更和Bug修复：**

1. **每次需求变更** - 自动添加新的变更条目
2. **每次Bug修复** - 记录问题原因和修复方案
3. **同步更新PRD.md** - 确保需求文档与代码实现一致

### 记录内容

每个变更包含：
- 变更版本和日期
- 修改的文件列表
- 具体的代码修改（前后对比）
- 变更原因说明

### 版本号规则

`主版本.次版本.修订号`
- 主版本：重大架构调整
- 次版本：新增功能/需求变更
- 修订号：Bug修复/小型优化

---

*最后更新：2026-05-23*
