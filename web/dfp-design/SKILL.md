---
name: dfp-design
description: 数据基础平台设计规范（标准色/标准字/按钮/选择器/输入框）。当需要确认平台配色、字号、按钮/选择器/输入框的尺寸与状态样式，或让页面 UI 与平台设计系统保持一致时使用本 skill。
---

# 数据基础平台设计规范

本 skill 汇总平台设计规范（`uidesgin/` 目录：`标准色.png`、`ui规范.png`）提炼出的标准。平台基于 **Vue 3 + Ant Design Vue（ant-design-vue）** 构建，所有 token 与 Ant Design 官方语义对齐，可直接写入 Vue 组件样式或设计主题变量。页面骨架与组件选型规范见 [[dfp-page]]。

## 设计核心原则

1. **统一语义化设计变量**，颜色一律走下方色板，不随意写死。
2. **品牌主色为「青蓝双色渐变」** `#00D2BE → #4EACFF`；交互实底取蓝色档 `#24B1E7`。功能色（成功/警告/错误）只表达对应语义，禁止用作装饰。
3. **字号按层级取标准档位**，禁止为强调效果使用非规范字号。
4. **组件尺寸三档**：大 40px / 中 32px / 小 24px（高度）。同一界面内同一类型控件保持同一档位。
5. **状态必做全**：default / hover / focus / disabled，输入类还要覆盖校验态（error/warning/success）。
6. **hover → active → disabled 遵循「同色系加白 → 加黑 → 降饱和」**：hover 约加白 30%，active/press 约加黑 10%。

---

## 一、标准色（标准色.png）

### 1.1 品牌与功能色

平台主色为「青蓝双色渐变」`#00D2BE → #4EACFF`，用于 Logo、主视觉、头部强调。交互组件（按钮/链接/选中）采用蓝色实底档 `#24B1E7`。

| 语义 | 色值 |
|---|---|
| 主色（品牌渐变） | `#00D2BE` → `#4EACFF` |
| 主色（实底/交互） | `#24B1E7` |
| 成功 / 在线 | `#34E4C7` |
| 警告 | `#FFAB79` |
| 错误 / 异常 | `#FF5126` |

> 成功 `#34E4C7` 与品牌青同源，但成功仅用于表达语义，不作装饰。主色/成功的 hover/active 真实色板见 1.6；警告/错误的交互态按同一条明暗规则（hover 加白约 30%、active 加黑约 10%）派生即可。

### 1.2 中性文字色

中性色用于文本、背景和边框颜色，通过不同深浅表现层次结构（基于黑色透明度，落成实色如下）：

| 层级 | 色值 | 用途 |
|---|---|---|
| 标题/正文 | `#333333` (Alpha 0.80) | 页面/卡片/表格标题、正文 |
| 装饰色 | `#666666` (Alpha 0.60) | 次要装饰、图标弱化 |
| 辅助文字 | `#979797` (Alpha 0.40) | 描述、次要信息、统计说明、时间戳 |
| 不可点击/占位 | `#CCCCCC` (Alpha 0.20) | placeholder、禁用文字 |

> 平台辅助文字也有 `#808080`（Alpha 0.50）等相邻档位，可混用但推荐优先用 `#979797`。

### 1.3 边框与背景

| Token | 色值 | 用途 |
|---|---|---|
| 边框（常态） | `#E6E6E6` (Alpha 0.10) | 输入框/选择器/卡片描边 |
| 分割线 | `#FAFAFA` (Alpha 0.04) | 列表分割、布局分隔 |
| 填充背景 | `#F0F0F0` (Alpha 0.04) | 禁用态、图表空背景 |
| 页面/布局背景 | `#FAFAFA` | 工作区底 |
| 选中浅底 | `#E4F4FA` | 下拉选中项、标签选中底、蓝色态 hover |

### 1.4 校验状态光环（focus ring）

输入类控件聚焦/校验时使用双色散光：

- 聚焦（信息）：边框 `#24B1E7` + `box-shadow: 0 0 0 2px rgba(36,177,231,0.1)`
- 错误：边框 `#FF5126` + `box-shadow: 0 0 0 2px rgba(255,81,38,0.1)`
- 警告：边框 `#FFAB79` + `box-shadow: 0 0 0 2px rgba(255,171,121,0.1)`
- 成功：边框 `#34E4C7` + `box-shadow: 0 0 0 2px rgba(52,228,199,0.1)`

### 1.5 圆角与阴影

- 组件默认圆角：`6px`；卡片/弹层大圆角：`8px` / `12px`
- 弹层投影：`box-shadow: 0 6px 16px 0 rgba(0,0,0,0.08), 0 3px 6px -4px rgba(0,0,0,0.12)`

### 1.6 交互态明暗规则（按钮色）

按钮/交互控件的状态色由基色按固定规则派生，三档基础色板如右：青（成功同源）/ 蓝 / 中性灰。

| 状态 | 规则 | 青 | 蓝 | 中性 |
|---|---|---|---|---|
| hover | 同色系加白约 30% | `#60E0CC` | `#9DCFF5` | `#E4F4FA` |
| 常态 | 基色 | `#34E4C7` | `#24B1E7` | `#E0E0E0` |
| press | 同色系加黑约 10% | `#23D5BA` | `#189AC3` | `#E8E8E8` |

> 主色交互统一走右侧「蓝」档；成功按钮走「青」档；中性灰用于次按钮/默认态。

---

## 二、标准字（标准字.png）

| 层级 | 字号 | 行高 | 字重 | 颜色 | 用途 |
|---|---|---|---|---|---|
| 大标题 | 20px | 28px | 600 | 标题色 `#333333` | 页面主标题 |
| 标题 | 16px | 24px | 600 | 标题色 `#333333` | 卡片/区块标题 |
| 正文 | 14px | 22px | 400 | 正文色 `#333333` | 常规内容、表格正文 |
| 辅助/说明 | 12px | 20px | 400 | 辅助文字 `#979797` | 提示、次要说明、表格备注 |

> 数字型统计值（如顶部统计卡片数值）允许使用更大字号但保持同一字体族，避免混入非规范字体。

---

## 三、按钮（按钮.png）

### 3.1 类型

| 类型 | 用法 | 样式 |
|---|---|---|
| `primary` 主按钮 | 页面/表单主操作 | 实底 `#24B1E7`，白字；hover 底 `#9DCFF5`，active 底 `#189AC3` |
| `default` 次按钮 | 次要/并列操作 | 白底，边框 `#E6E6E6`，文字 `#333333`，文字与边框跟随主色 hover/active |
| `dashed` 虚线按钮 | 添加/占位操作 | 同 default，但边框为虚线 |
| `text` 文字按钮 | 行内轻操作 | 无边框无底色，hover 底 `rgba(0,0,0,0.04)` |
| `link` 链接按钮 | 跳转/行内链接 | 文字用主色 `#24B1E7`，hover 变 `#189AC3` 并下划线 |
| `danger` 危险按钮 | 删除/危险操作 | 红字红边（`#FF5126` 体系），实底变体为红色 fill |

### 3.2 禁用态

- 底色 `#F0F0F0`，边框 `#E6E6E6`，文字 `#CCCCCC`
- 禁止只变透明度，需整套降饱和

### 3.3 尺寸

| 尺寸 | 高度 | 横向内边距 | 字号 |
|---|---|---|---|
| 大 | 40px | 16px | 16px |
| 中（默认） | 32px | 15px | 14px |
| 小 | 24px | 7px | 14px |

圆角统一 `6px`。带图标按钮图标与文字间距 `8px`，仅图标按钮边长等于高度。

### 3.4 Vue 写法

```vue
<a-button type="primary" :loading="saving" @click="save">保存</a-button>
<a-button @click="reset">重置</a-button>
<a-button type="danger" danger><DeleteOutlined /> 删除</a-button>
<a-button type="link">查看详情</a-button>
```

---

## 四、选择器（选择器.png）

### 4.1 尺寸

同按钮三档：大 40px / 中 32px / 小 24px 高，圆角 `6px`。

### 4.2 状态

| 状态 | 样式 |
|---|---|
| default | 白底，边框 `#E6E6E6` |
| hover | 边框 `#9DCFF5` |
| focus | 边框 `#24B1E7` + `box-shadow 0 0 0 2px rgba(36,177,231,0.1)` |
| disabled | 底色 `#F0F0F0`，边框 `#E6E6E6`，文字 `#CCCCCC` |

### 4.3 图标

- 后缀箭头：默认向下箭头，展开时旋转 180°
- 清除图标：hover 显示，`#24B1E7` 强调
- 加载中：显示 loading 旋转图标，同时禁用点击
- 搜索模式：前缀放大镜图标，搜索同时高亮匹配项

### 4.4 下拉面板与多选

- 面板：白底 `#fff`，圆角 `8px`，使用 1.5 统一弹层投影
- 选项：高 `32px`；hover 底 `rgba(0,0,0,0.04)`；选中底 `#E4F4FA` 且文字转为主色；按下短暂底 `#CDE3F8`；禁用 `#CCCCCC` 且不可点
- 多选：已选项以可关闭标签（tag）换行展示，标签关闭图标 hover 变主色

### 4.5 Vue 写法

```vue
<a-select
  v-model:value="form.sourceType"
  style="width: 220px"
  placeholder="请选择数据源类型"
  :options="sourceTypeOptions"
  allowClear
  @change="handleTypeChange"
/>
<a-select
  v-model:value="form.multi"
  mode="multiple"
  :options="options"
  placeholder="可多选"
  :max-tag-count="3"
/>
<a-select
  v-model:value="form.search"
  show-search
  :filter-option="filterOption"
  :loading="loading"
  placeholder="可搜索选择"
/>
```

---

## 五、输入框（输入框.png）

### 5.1 尺寸

同按钮三档：大 40px / 中 32px / 小 24px 高，圆角 `6px`。

### 5.2 状态

| 状态 | 样式 |
|---|---|
| default | 白底，边框 `#E6E6E6`，placeholder 用占位色 `#CCCCCC` |
| hover | 边框 `#9DCFF5` |
| focus | 边框 `#24B1E7` + 信息色光环（见 1.4） |
| disabled | 底色 `#F0F0F0`，边框 `#E6E6E6`，文字 `#CCCCCC` |
| error / warning / success | 边框 + 对应色光环（见 1.4），配合下方提示信息 |

### 5.3 前后缀与扩展

- 前缀/后缀图标：图标与输入文本间距 `8px`，图标 hover/聚焦时为 `#24B1E7`
- 清除按钮：有值悬停输入框时显示，点击清空
- 密码框：右侧眼睛图标切换明文/密文
- 字数统计：右下角 `{已输入}/{上限}`，达到上限文字变红（`#FF5126`）
- 带搜索/操作按钮的组合框（`a-input-search` / `Group`）：按钮与输入同高同档位

### 5.4 Vue 写法

```vue
<a-input
  v-model:value="form.name"
  placeholder="请输入名称"
  :maxlength="50"
  show-count
  allow-clear
>
  <template #prefix><UserOutlined /></template>
</a-input>
<a-input-password v-model:value="form.pwd" placeholder="请输入密码" />
<a-input
  v-model:value="form.code"
  status="error"
/>
<span v-if="errorMsg" class="form-error">请输入正确的编码</span>
<a-input-search
  v-model:value="searchText"
  placeholder="搜索数据资产..."
  enter-button
  @search="fetchData(true)"
/>
```

---

## 六、树形控件（Tree）

### 6.1 选中态

选中节点背景统一 `#EBFBFA`，文字色不变（仍为正文色 `#333333`）。

全局已在 `src/styles/global.less` 覆盖，无需每个页面单独写：

```less
// 树选中态全局背景（4 个 class 特异性，压过 antd css-in-js 默认样式）
.ant-tree .ant-tree-treenode .ant-tree-node-content-wrapper.ant-tree-node-selected,
.ant-tree .ant-tree-treenode-selected .ant-tree-node-content-wrapper,
.ant-tree .ant-tree-treenode-selected:hover .ant-tree-node-content-wrapper,
.ant-tree .ant-tree-checkbox + span.ant-tree-node-selected {
  background-color: #ebfbfa;
}
```

> antd 默认选中样式形如 `:where(...).ant-tree .ant-tree-node-content-wrapper.ant-tree-node-selected`——`:where()` 不计特异性，实际只有 3 个 class。覆盖规则至少要有 4 个 class 特异性，否则会被压住。

### 6.2 Vue 写法

```vue
<a-tree
  :tree-data="treeData"
  :selected-keys="selectedKeys"
  :expanded-keys="expandedKeys"
  block-node
  @select="handleSelect"
/>
```

选中背景由全局样式自动生效，页面内不要再单独覆盖选中底色。

---

## 七、自查清单（生成/评审页面时逐项核对）

- [ ] 颜色均取自色板：品牌主色走青蓝渐变，交互主色取 `#24B1E7`，功能色语义正确
- [ ] 字号取标准档位，层级与用途匹配
- [ ] 按钮/选择器/输入框尺寸三档统一，圆角 6px
- [ ] 交互组件 default/hover/focus/disabled 状态齐全
- [ ] 输入类组件 error/warning/success 校验态正确
- [ ] 弹层/下拉投影与圆角统一
- [ ] 输入类 placeholder 使用占位色，禁用态整组降饱和
- [ ] 树选中节点背景为 `#EBFBFA`，页面内不要单独覆盖
- [ ] 间距沿用 16px 体系（区块 / 卡片内边距 / 栅格 gutter）