---
name: codecheck
version: 1.0.0
description: "通用代码自查 skill。对新增/修改的代码执行通用工程检查，当前包含：分页列表删除末行后自动回退页码、表单输入长度范围限制、弹窗点击外部不关闭、表格撑满容器等检查细项。当用户要求 code check / 代码自查 / 检查代码是否规范 / 校验某个逻辑有没有漏做时自动触发。不针对任何特定项目或框架。"
---

# 通用代码自查

本 skill 沉淀「写完代码后容易漏掉的通用工程逻辑」检查细项，不绑定任何项目或框架。完成一段功能改动后，按以下细项逐条自查；也可在用户要求 code check 时运行。

## 触发时机

当对话中出现以下任一情况时，主动读取并应用本 skill：

1. 用户要求 "code check" / "代码自查" / "检查代码" / "看看有没有漏掉什么"
2. 用户报告某类功能问题，需要排查同类模块是否都有该问题（如"其它模块有没有没处理这个逻辑的"）
3. 完成列表分页、删除、弹窗表单等通用功能改动后

## 检查细项

### 检查项 1：分页列表删除末行后自动回退页码

**场景**：带分页的列表支持删除时，如果删除的是当前页的最后一条数据（尤其当当前页不是第一页），刷新列表后若仍停留在同一页码，该页将显示为空，用户看到的是空白列表。

**判定标准**：

- ❌ **未处理**：删除成功后直接重新加载当前页数据，停留在空页。
- ✅ **已处理**：删除成功后，先判断「当前页只剩 1 条 && 当前页码 > 1」，满足则页码减一再重新加载。
- ✅ **可接受**：删除后重新加载并主动重置到第 1 页，不会出现空页（与回退到上一页效果等价，只是不保留原页码）。
- ⚠️ **不适用**：列表无分页（一次性加载全部数据，删除只从内存数组移除）；或删除功能尚未实现（接口/TODO 占位），此时不算漏，但要在实现删除时补上该逻辑。

**通用伪代码**：

```js
onDelete(row) {
  deleteRows({ ids: row.id }).then((result) => {
    if (result.ok) {
      showSuccess('删除成功')
      // 删除的是当前页最后一条且不在第一页时，页码回退，避免刷新后停在空页
      if (currentList.length === 1 && currentPage > 1) {
        currentPage -= 1
      }
      reloadList()
    }
  })
}
```

**检查流程**：

1. 定位目标范围内所有「含列表 + 分页 + 删除操作」的模块（先按入口页面找，再扩展到子组件/弹窗内的列表）。
2. 过滤出「分页 + 删除」齐全的模块。
3. 逐个查看删除成功分支，判断是否有页码回退。
4. 报告时区分「已处理 / 可接受（重置第 1 页）/ 未处理 / 不适用」，并列出模块位置（文件路径与行号）。

### 检查项 2：表单输入必须限制长度/范围

**场景**：表单中的文本输入如果不设最大长度，用户可输入任意长度内容，导致超长数据写入存储、界面溢出、校验失效；数值输入若没有范围限制，可能录入明显不合理的值。

**判定标准**：

- ❌ **未处理**：文本输入未设置最大长度（maxlength）；数值输入未设置最小/最大值（min/max）。
- ✅ **已处理**：文本输入设置了 maxlength，且与后端字段长度约定一致；数值输入设置了符合业务语义的 min/max（如面积、数量类通常非负）。
- ⚠️ **例外**：确需长文本的字段（如富文本、大段描述）可以放宽，但仍应有一个合理的上限，而不是完全不限制。

**默认限制值（项目约定）**：

- 单行文本输入（`a-input` / `input` / `eawcs-input`）：`maxLength` 默认 **40**
- 多行文本域（`a-textarea` / `textarea`）：`maxLength` 默认 **80**
- 数值输入（`a-input-number` / `input[type=number]`）：`min`/`max` 按业务语义（面积/金额类通常 `min=0`）
- 特殊输入按业务定：如颜色值（hex）`maxLength=7`、数量类用 `a-input-number` 并设 `:max`

**通用伪代码**：

```js
// 单行文本输入：maxLength 默认 40
<input value={...} maxlength={40} />

// 多行文本域：maxLength 默认 80
<textarea value={...} maxlength={80} />

// 数值输入：必须给合理的范围
<input type="number" min={0} max={100000} step={0.01} />
```

**检查流程**：

1. 找出所有表单输入控件（文本框、数字框、下拉可输入框、富文本等）。
2. 逐个确认是否设置了长度/范围限制。
3. 对文本输入，额外确认 `maxLength` 是否符合默认约定（单行 40、多行 80），明显超标（如仍写 255/500 且非长文本字段）要指出。
4. 对数值类输入，额外确认上下限是否符合业务语义（是否允许负数、是否过大/过小）。
5. 报告时列出「未设置限制 / 限制值不合约定的输入控件 + 位置（文件路径与行号）」。

### 检查项 3：drawer/modal 默认不能点击外部（遮罩）关闭

**场景**：drawer/modal 弹窗内容（尤其含表单）如果允许点击遮罩直接关闭，用户误点外部会导致未保存的输入丢失、正在进行的操作被意外中断。

**判定标准**：

- ❌ **未处理**：drawer/modal 未显式设置禁止遮罩关闭属性，使用默认值（ant 的 a-drawer/a-modal 与 element 的 el-dialog/el-drawer 默认均可点击遮罩关闭）。
- ✅ **已处理**：显式设置 `:maskClosable="false"`（Ant Design Vue）或 `:close-on-click-modal="false"`（Element UI）。
- ⚠️ **例外**：确需点击遮罩关闭的轻量场景（纯提示、无表单、无需要保留的状态）可以允许，但需确认该场景确实不需要防误关。

**组件与属性对照**：

| 组件库 | 组件 | 需设置的属性 | 默认行为 |
|---|---|---|---|
| Ant Design Vue | `a-drawer` | `:maskClosable="false"` | 默认 true，可点遮罩关闭 |
| Ant Design Vue | `a-modal` | `:maskClosable="false"` | 默认 true |
| Element UI | `el-dialog` | `:close-on-click-modal="false"` | 默认 true |
| Element UI | `el-drawer` | `:close-on-click-modal="false"` | 默认 true |

**检查流程**：

1. 找出所有 drawer/modal 组件（`a-drawer`、`a-modal`、`el-dialog`、`el-drawer`，以及项目封装的 `base-drawer` / `base-modal` 等）。
2. 逐个确认是否设置了对应的禁止遮罩关闭属性（`maskClosable=false` / `close-on-click-modal=false`）。
3. 对封装的 drawer/modal 组件：优先在封装组件内部设置默认值；若封装未设置，调用处必须透传或显式设置该属性。
4. 区分「已处理 / 未处理 / 例外」，列出未处理的组件位置（文件路径与行号）。

**通用伪代码**：

```html
<!-- Ant Design Vue：禁止点击遮罩关闭 -->
<a-drawer :maskClosable="false" :visible="visible" @close="onClose">...</a-drawer>
<a-modal :maskClosable="false" :visible="visible" @ok="onOk">...</a-modal>

<!-- Element UI：禁止点击遮罩关闭 -->
<el-dialog :close-on-click-modal="false" :visible.sync="visible">...</el-dialog>
<el-drawer :close-on-click-modal="false" :visible.sync="visible">...</el-drawer>
```

### 检查项 4：table 组件默认撑满容器（动态高度）

**场景**：页面中的表格如果高度固定或不做动态高度计算，容器缩放、搜索区展开收起、内容变化时表格会过短留白或溢出，无法自适应撑满父容器。

**判定标准**：

- ❌ **未处理**：表格未做动态高度（无 `v-auto-table-height` 指令、无 `scroll.y` 动态计算、无高度计算逻辑），按固定高度或内容高度渲染。
- ✅ **已处理（推荐）**：使用 `v-auto-table-height` 指令（main.js 全局注册），指令自动计算父容器剩余高度并设置 `scroll.y`，无需手动维护 `tableH` 与 resize 监听。
- ✅ **已处理（回退）**：指令未注册时，用手动模式（外层 `ref` + `scroll.y` + `mounted`/`beforeDestroy` resize 监听）实现。
- ⚠️ **例外**：确需固定高度的表格（如容器本身无高度约束的嵌入式小表格、纯静态展示）可以放宽。

**实现方式**：

1. **优先：`v-auto-table-height` 指令**。若项目没有该指令，应实现并全局注册（推荐方案），而不是每个页面各自写一套手动高度：

```html
<!-- 外层容器需 flex 布局 + overflow hidden，撑满父容器 -->
<div style="flex: 1; overflow: hidden;">
  <a-table v-auto-table-height ... />
</div>
```

可配置参数：

| 参数 | 说明 | 默认 |
|---|---|---|
| `bottomGap` | 底部留白（分页区域） | 50 |
| `minHeight` | 最小高度 | 160 |

```html
<a-table v-auto-table-height="{ bottomGap: 60, minHeight: 200 }" ... />
```

2. **回退：手动模式**（指令未注册时）：

- 外层容器加 `ref="tableBox"`，样式 `flex: 1; overflow: hidden;`
- `data` 声明 `tableH: 0`
- `a-table` 加 `v-if="tableH > 0"` 和 `:scroll="{ y: tableH }"`
- `mounted` 调 `fetTableHeight()` 并监听 `window.resize`
- `beforeDestroy` 移除监听

```js
fetTableHeight() {
  this.resetHeight().then(() => {
    this.$nextTick(() => {
      this.tableH = this.$refs.tableBox?.getBoundingClientRect()?.height - 50
    })
  })
}
```

**检查流程**：

1. 找出所有表格组件（`a-table`、`ant-table`、`el-table` 及项目封装的表格组件）。
2. 确认表格外层容器是否为 flex 布局且高度链完整（父容器有高度、`flex:1; overflow:hidden`）。
3. 确认是否使用 `v-auto-table-height` 指令，或手动高度计算（`tableH` + `scroll.y` + resize 监听），并确认容器尺寸变化时能重算。
4. 区分「已处理 / 未处理 / 例外」，列出未处理的表格位置（文件路径与行号）。
