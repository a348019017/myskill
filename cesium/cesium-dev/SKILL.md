---
name: cesium-dev
description: Cesium/NSC Earth 开发注意事项与最佳实践。当在本仓库编写、修改、审查涉及 Cesium 三维地图代码（NSCEarth、GIM 图层、Viewer、Entity、DataSource、Cesium.* 对象）时使用。开发 Cesium 应用、改三维沙盘代码、遇到 Cesium 对象与 Vue/Pinia 结合的问题时调用。
---

# Cesium 开发注意事项（mcms_frontend）

在编写或修改本仓库任何涉及 `Cesium.*`、`NSCEarth`、GIM 图层、`Viewer`/`Entity`/`DataSource` 的三维代码时，先读本文件。核心目标：**不要用 Vue 的响应式系统去包裹 Cesium 对象**。

## 1. 黄金准则：Cesium 对象不允许加入 Vue 响应式

**绝对不要把 `Cesium` 相关的对象（`Viewer`、`Scene`、`Entity`、`DataSource`、`GeojsonMap`、`NSCEarth`、`LayerManagerUtils`、各种 GIM/LineManager）放进 Vue / Pinia 的响应式数据里。**

原因：

- Cesium 对象内部是海量的、高度自引用的图结构（每个 Entity 挂在 DataSource 上，DataSource 挂在 Viewer 上），Vue 2 的 `Object.defineProperty` / Vue 3 的 `Proxy` 会深度递归代理它们，导致：**初始化巨慢**（首帧卡死、构建页面崩溃）、**内存被无限递归代理撑爆**、**循环结构触发堆栈溢出**。
- 你已经踩过的坑见 `git log`：`LayerManagerUtils` 卡死问题（commit `af28ace9` —— 修复 LayerManagerUtils 卡死）。
- 这类"卡死"绝大多数就是 Cesium 对象被拖进响应式导致的。

### 怎么做（按优先级）

1. **优先：根本不放进响应式 store。** 地球实例由 `NSCEarthUI.vue` 等组件持有，`mounted()` 里 `new NSCEarth(...)`，存到 `this.`（非响应式）。这是默认且最干净的做法。
2. **如果非要跨组件共享（gim store 就是这么做的）：用 `markRaw` 包裹再赋值。** 见 [src/store/gim.js:28-29](src/store/gim.js#L28-L29)、[src/components/NSCEarthUI_ST.vue:227](src/components/NSCEarthUI_ST.vue#L227)：
   ```js
   import { markRaw } from "vue"
   state.earth = null           // store 初始化
   this.useGIMStore.earth = markRaw(NSCEarth)   // 赋值时包一层
   ```
3. **Cesium 集合容器用 `markRaw`：** `new Map()` / `new Set()` 里放 Cesium 对象（如 `lineCollectionMap`）也要 `markRaw(new Map())`，否则 Map 内部被代理、塞入的 Entity 全被递归代理。

### 反例（注意，代码库里确实存在，别再新增）:

- ❌ [src/components/NSCEarthUI_3Cross.vue:59](src/components/NSCEarthUI_3Cross.vue#L59)：`useGIMStore.earth = NSCEarth`（没包 `markRaw`）
- ❌ [src/components/tools/LayerManager.vue:58](src/components/tools/LayerManager.vue#L58)：`this.useGIMStore.layermanagerUtil = new LayerManagerUtils(...)`（没包 `markRaw`）

> **排查线索**：页面卡死 / 初始化卡顿 / 内存飙升 / `Maximum call stack size exceeded` / 构建时报奇怪的大对象序列化错误，先怀疑 Cesium 对象进了响应式。

## 2. Cesium 是全局对象，绝不 import

- `Cesium` 作为全局 script 从 `public/Cesium/Cesium.js` 加载（见 `index.html`），直接用裸 `Cesium.*`。
- **仓库里没有任何 `import ... from "cesium"`，也不要新增。** 你的 package.json 里也没有 `cesium` npm 依赖。
- 同理，`GeojsonMap`、`LayerManagerUtils`、GIM 系列都是 `nsc-earth` 提供的，照现有组件的方式引入，别乱 import Cesium。

## 3. 走 nsc-earth 提供的能力，别手搓

能用 `EventManager`（`NSCEarth.eventManager`）、`GIMLineLayerManager`、`GIMSubstationLayerManager`、`GeojsonMap`、`LayerManagerUtils` 就优先用，它们封装了 Viewer/事件/图层的生命周期。直接 new `Cesium.Viewer` 的场景基本只有 `NSCEarth.initViewer()` 一处（[src/utils/NSCEarth.js:48](src/utils/NSCEarth.js#L48)）。

## 4. 生命周期：记得清理

- `NSCEarthUI*.vue` 在 `mounted()` 初始化 viewer、在销毁时/`currentProject.id` 变化时清空地球引用（例：`this.useGIMStore.earth = null`）。
- 自己 new 的 `Cesium.ScreenSpaceEventHandler`、`EventManager` 监听、`DataSource`，在组件 `beforeDestroy`/`onUnmounted` 里移除，否则切项目/切租户会叠监听、泄漏 Entity。
- 销毁时把 store 里的 `earth`/`geomap`/`layermanagerUtil` 置回 `null`（不是 delete key），避免残留对象被再次取值。

## 5. 响应式数据与 Cesium 分离

- 只有**展示用**的普通数据（id、名称、`properties`、选中项的普通字段）才进 Vue 响应式。
- 凡是持有 `Cesium.*` 对象本身、或者"数据源/图层引用"的，一律 markRaw 或放组件非响应式字段。
- 选中态、label 显示态（`labelShow`、`currentCoverId`、`currentFeature`）这类轻量状态可以正常响应式。

## 6. 多租户 / 多版本注意

- 地球 UI 按 prefixed 变体分布：`NSCEarthUI.vue`（通用）、`NSCEarthUI_NX/HB/ST/3Cross.vue`。**改 Cesium 行为先确认改的是哪个变体**（`st*` 是当前分支标准版重点）。改一处记得看同类变体是否需要同步。
- 地貌/地形与租户绑定（`public/terrian/index.js`、`utils/NSCEarth.js` 里 `tenantId` 分支）。

---

**一句话总结**：Cesium 对象是"极重的、自引用的大图对象"，它们是渲染引擎的资产，不是界面数据。`markRaw` 是你的第一道防线，不放 store 才是上策。改完三维代码后，用 `npm run serve` 起服务实测一遍确认不卡死、不报错再算完成。
