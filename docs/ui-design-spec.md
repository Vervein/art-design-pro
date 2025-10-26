# 目录 & 摘要

## 目录

- [摘要](#摘要)
- [人读版规范](#人读版规范)
  - [全局样式体系](#全局样式体系)
  - [组件规范](#组件规范)
  - [覆盖矩阵](#覆盖矩阵)
- [机器读版骨架](#机器读版骨架)
- [缺口与建议清单](#缺口与建议清单)
- [命名与缩放对齐提案](#命名与缩放对齐提案)
- [迁移与落地步骤](#迁移与落地步骤)

## 摘要

- 归纳现有全局设计 token（颜色、排印、栅格、动画、圆角等）并补充缺失的默认值，标注可持续迭代项。
- 为 74 个前端组件（基础、布局、表单、数据可视化、媒体等）建立用途、结构、状态、可访问性与组合规范，并提供可复制代码片段及最小使用示例。
- 梳理 Light/Dark 主题切换策略与 CSS 变量落点，输出颜色对比度评估表，指出需提升对比度的语义色。
- 提供组件 × 变体 × 状态 × 尺寸的覆盖矩阵以及 JSON 骨架，便于设计与工程协同追踪。
- 汇总命名和缩放策略、主题 token 统一方案与落地路径，列出短中长期的优化建议。

## 人读版规范

### 全局样式体系

#### 色彩系统

- **主题色定义**：项目通过 CSS 变量声明主色、副色与语义色，Light/Dark 共用色值但映射到不同背景与灰度层级。当前默认主色 `rgb(93,135,255)`（#5D87FF）、副色 `rgb(73,190,255)`（#49BEFF），成功 `rgb(19,222,185)`（#13DEB9）、警告 `rgb(255,174,31)`（#FFAE1F）、信息 `rgb(107,125,155)`（#6B7D9B）、错误 `rgb(250,137,107)`（#FA896B）、危险 `rgb(255,77,79)`（#FF4D4F）。【F:src/assets/styles/variables.scss†L3-L49】
- **可迭代项**：Secondary/Warning/Success 在 Light Surface 上的对比度未达到 WCAG AA（见下表），需要在视觉稿中加深或调整文字使用策略。
- **背景 & 灰阶**：Light 模式默认底色 `#FAFBFC`、主容器背景 `#FFFFFF`；Dark 模式底色 `#070707`、主容器 `#161618`，并通过 `--art-gray-100~900` 保持 9 级灰度映射。【F:src/assets/styles/variables.scss†L78-L161】
- **颜色使用原则**：
  - 主色用于操作类控件、强调数据；副色用于次级图表与标签。
  - 功能色优先与对应浅色背景组合（`--art-bg-*`），确保信息密度可控。
  - `--art-border-color`/`--art-card-border` 控制边框统一性，避免自定义硬编码。【F:src/assets/styles/variables.scss†L63-L168】
- **对比度评估**：

  | 语义色 | 十六进制 | Light Surface (#FFFFFF) | Light Base (#ECF2FF) | Dark Surface (#161618) | Dark Base (#25365E) | 备注 |
  | --- | --- | --- | --- | --- | --- | --- |
  | Primary | #5D87FF | 3.29 | 2.94 | 5.49 | 3.61 | 文本/图标需加粗或加深背景以满足 AA Large |
  | Secondary | #49BEFF | 2.08 ⚠️ | 1.85 ⚠️ | 8.69 | 5.71 | Light 模式建议只用于填充/图形；文本请使用深色叠加 |
  | Info | #6B7D9B | 4.17 | 3.72 | 4.33 | 2.85 | 建议 Light 上搭配 `--art-gray-900` 字体 |
  | Success | #13DEB9 | 1.72 ⚠️ | 1.54 ⚠️ | 10.48 | 6.89 | 文本请搭配深色，或提高饱和度 |
  | Warning | #FFAE1F | 1.85 ⚠️ | 1.65 ⚠️ | 9.75 | 6.41 | Light 模式需增加深色文字或描边 |
  | Error | #FA896B | 2.37 ⚠️ | 2.11 ⚠️ | 7.62 | 5.01 | 建议与深色文字组合 |
  | Danger | #FF4D4F | 3.27 | 2.91 | 5.53 | 3.63 | 建议大字号/粗体以达标 |

- **代码片段**：
  ```scss
  :root {
    --art-primary: 93, 135, 255; // TODO: 视觉确认品牌主色
    --art-secondary: 73, 190, 255;
    --art-success: 19, 222, 185; // 可迭代项：Light 模式对比度
    --art-warning: 255, 174, 31; // 可迭代项：Light 模式对比度
    --art-error: 250, 137, 107;
    --art-info: 107, 125, 155;
    --art-danger: 255, 77, 79;
  }
  html.dark {
    --art-bg-color: #070707;
    --art-main-bg-color: #161618;
  }
  ```
- **最小使用示例**：

  ```vue
  <template>
    <button class="art-btn-primary">操作</button>
  </template>

  <style scoped>
    .art-btn-primary {
      background: rgb(var(--art-primary));
      color: #fff; /* TODO: 根据品牌规范调整文字颜色 */
      border-radius: var(--custom-radius);
      padding: 0.5rem 1rem;
    }
    .art-btn-primary:hover {
      background: color-mix(in srgb, rgb(var(--art-primary)) 85%, #000);
    }
  </style>
  ```

#### 排版体系

- **字体家族**：全局字体栈 `Inter, Helvetica Neue, PingFang SC, Microsoft YaHei, Arial, sans-serif`，基础字重 400。【F:src/assets/styles/reset.scss†L3-L40】
- **字号刻度**：建议按照 4px 系列（12/14/16/18/20/24/28/32/40/48）统一；现有 Element Plus 默认字号 14px，可使用 CSS 变量拓展 `--font-size-sm/md/lg`（可迭代项：落地设计 token 文件）。
- **行高/字距**：标题采用 1.25 行高，正文 1.5；按钮/表单控件继承 `--el-component-custom-height` 36px，保证触控面积。【F:src/assets/styles/el-ui.scss†L3-L54】
- **字重**：正文 400，标题与强调可使用 500/600；Element 主按钮 `--el-font-weight-primary` 已设为 400，避免过粗视觉。【F:src/assets/styles/el-ui.scss†L3-L17】
- **图标对齐**：与文本混排时保持 16px 字号并垂直居中，可使用 `line-height: 1` 控制；在 `ArtIconSelector` 中统一通过 `iconfont-sys` class 渲染，避免内联 SVG 杂乱。【F:src/components/core/base/art-icon-selector/index.vue†L1-L76】【F:src/assets/styles/app.scss†L33-L54】
- **代码片段**：
  ```scss
  :root {
    --font-family-base:
      Inter, 'Helvetica Neue', Helvetica, 'PingFang SC', 'Microsoft YaHei', Arial, sans-serif;
    --font-size-base: 14px;
    --font-size-lg: 16px;
    --font-size-sm: 12px;
    --line-height-base: 1.5;
    --heading-line-height: 1.25;
  }
  body {
    font-family: var(--font-family-base);
    font-size: var(--font-size-base);
    line-height: var(--line-height-base);
  }
  ```
- **最小使用示例**：

  ```vue
  <template>
    <article class="typography-demo">
      <h1>页面标题</h1>
      <p>这里是正文内容，保持 1.5 行高与 14px 基准字号。</p>
    </article>
  </template>

  <style scoped>
    .typography-demo {
      font-family: var(--font-family-base);
      letter-spacing: 0.02em; /* TODO: 根据语言调整字间距 */
    }
  </style>
  ```

#### 图标规范

- **来源**：内置 iconfont（`@/assets/icons/system/iconfont.css`）结合 `ArtIconSelector` 组件选择 className 或 unicode。导入 SVG 时推荐在 `src/assets/svg` 下统一维护。
- **尺寸层级**：12/16/20/24/32px，按钮内默认 16px，浮动按钮 20px；保持 `line-height:1` 避免上下跳动。【F:src/components/core/base/art-icon-selector/index.vue†L1-L121】
- **色彩**：遵循语义色；单色图标建议使用 `currentColor` 适应父容器文本色。
- **动效**：仅在交互性强的图标（如 `ArtBackToTop`）添加轻微缩放或旋转，动画时长遵循全局动效规范。
- **代码片段**：

  ```vue
  <template>
    <span class="icon-wrapper">
      <i class="iconfont-sys art-icon" aria-hidden="true">&#xe709;</i>
      <span class="sr-only">打开菜单</span>
    </span>
  </template>

  <style scoped>
    .icon-wrapper {
      display: inline-flex;
      align-items: center;
      gap: 0.25rem;
    }
    .art-icon {
      font-size: 1.25rem;
      color: rgb(var(--art-primary));
    }
  </style>
  ```

#### 栅格与间距

- **24 栅格体系**：表单和布局默认使用 Element Plus `ElRow/ElCol` 24 栅格，`ArtForm` 提供 `span`/`gutter` 控制，移动端自动 1 列；建议统一 gutter 为 12/16/24 三档。【F:src/components/core/forms/art-form/index.vue†L12-L52】
- **间距变量**：建议引入 spacing token（4px 步长），例如 `--space-xs: 4px; ...`（可迭代项：尚未在仓库定义）。现有卡片普遍使用 12px/16px 内边距，可在 `ArtCard` 系列保持一致。
- **容器宽度**：支持 `100%` 与 `1200px` 两档，通过 Setting Store 管控。【F:src/enums/appEnum.ts†L42-L56】【F:src/components/core/layouts/art-page-content/index.vue†L1-L66】
- **代码片段**：
  ```scss
  :root {
    --space-xs: 4px;
    --space-sm: 8px;
    --space-md: 12px;
    --space-lg: 16px;
    --space-xl: 24px;
  }
  .stack-md > * + * {
    margin-top: var(--space-md);
  }
  ```
- **最小使用示例**：

  ```vue
  <template>
    <ElRow :gutter="16">
      <ElCol :span="12">
        <section class="panel">内容 A</section>
      </ElCol>
      <ElCol :span="12">
        <section class="panel">内容 B</section>
      </ElCol>
    </ElRow>
  </template>

  <style scoped>
    .panel {
      padding: var(--space-lg);
      border-radius: var(--custom-radius);
      background: var(--art-main-bg-color);
    }
  </style>
  ```

#### 圆角体系

- **Token**：`--custom-radius` 由设置面板动态注入，默认结合 Element Plus `--el-border-radius-*` 统一；建议定义三级圆角（微 6px / 中 12px / 大 16px）映射到卡片、按钮、弹窗。【F:src/store/modules/setting.ts†L314-L352】【F:src/assets/styles/el-ui.scss†L13-L38】
- **代码片段**：
  ```scss
  :root {
    --radius-sm: calc(var(--custom-radius) / 3 + 2px);
    --radius-md: calc(var(--custom-radius) / 2 + 4px);
    --radius-lg: calc(var(--custom-radius) + 6px);
  }
  .card {
    border-radius: var(--radius-lg);
  }
  ```
- **最小使用示例**：

  ```vue
  <template>
    <div class="card">内容区块</div>
  </template>

  <style scoped>
    .card {
      border-radius: var(--radius-lg);
      border: 1px solid var(--art-card-border);
    }
  </style>
  ```

#### 阴影与层级

- **阴影等级**：`--art-box-shadow-xs/sm/default/lg` 及 `--art-card-shadow` 提供 4 档阴影，优先在卡片、弹层上使用，Dark 模式默认移除 root-card 阴影。【F:src/assets/styles/variables.scss†L68-L157】
- **层级系统**：全局进度条 `z-index:2400`，全屏弹层 2300-2500（PageContent mask 2000）。建议定义 z-index token（基础 1、浮层 1000、弹窗 2000、引导 2400）以防冲突。【F:src/assets/styles/app.scss†L1-L112】【F:src/components/core/layouts/art-page-content/index.vue†L1-L61】
- **代码片段**：
  ```scss
  :root {
    --z-base: 1;
    --z-dropdown: 1200;
    --z-dialog: 2000;
    --z-notify: 2300;
  }
  ```

#### 断点与容器宽度

- **断点**：Sass 变量定义 Notebook 1600px、iPad Pro 1180px、iPad 800px、竖屏 900px、Mobile 500px；与设置面板 `containerWidth` 结合管理响应式布局。【F:src/assets/styles/variables.scss†L170-L200】【F:src/components/core/layouts/art-settings-panel/widget/ContainerSettings.vue†L1-L120】
- **建议**：以 1280/1440/1920 三档 PC 栅格作为设计稿基准，移动端 <500px 采用单列。
- **代码片段**：
  ```scss
  @media (max-width: 1180px) {
    .layout-sidebar {
      width: 72px;
    }
  }
  ```

#### 动效

- **动效 token**：`transition.scss` 定义统一的 duration 0.25s、滑动距离 15px、缓动曲线 `cubic-bezier(0.25,0.1,0.25,1)`；页面切换通过 `ArtPageContent` + Transition 组合。【F:src/assets/styles/transition.scss†L1-L104】【F:src/components/core/layouts/art-page-content/index.vue†L1-L66】
- **使用建议**：
  - 页面切换遵循 fade/slide 动画，时长 200~300ms。
  - 按钮与交互反馈可使用 `transition: all 0.2s ease-out`，保持一致的缓动。
- **代码片段**：
  ```vue
  <template>
    <Transition name="slide-left">
      <section v-if="visible" class="panel">内容</section>
    </Transition>
  </template>
  ```

#### 主题模式与覆盖策略

- **策略**：通过 `html.dark` class 切换 Dark token，同时在设置 Store 中写入 `--custom-radius`、`--el-color-primary*` 等变量，实现 CSS 变量覆盖。【F:src/assets/styles/variables.scss†L83-L162】【F:src/store/modules/setting.ts†L150-L359】【F:src/utils/ui/colors.ts†L112-L229】
- **主题来源**：主入口引入 Light/Dark 样式与主题动画，确保在应用挂载前加载。【F:src/main.ts†L6-L13】
- **扩展建议**：
  - 提供品牌色面板（十六进制）与浅/深色预览（可迭代项）。
  - 若未来支持自定义主题，可在 `setElementThemeColor` 中增加持久化与多套色板。
- **代码片段**：

  ```ts
  import { setElementThemeColor } from '@/utils/ui/colors'
  import { useSettingStore } from '@/store/modules/setting'

  const settingStore = useSettingStore()
  watch(
    () => settingStore.systemThemeColor,
    (val) => {
      setElementThemeColor(val)
    }
  )
  ```

- **最小使用示例**：

  ```vue
  <template>
    <button @click="toggleTheme">切换主题</button>
  </template>

  <script setup lang="ts">
    import { useSettingStore } from '@/store/modules/setting'
    const settingStore = useSettingStore()
    const toggleTheme = () => {
      settingStore.setGlopTheme(
        settingStore.isDark ? 'light' : 'dark',
        settingStore.isDark ? 'light' : 'dark'
      )
    }
  </script>
  ```

### 组件规范

#### 基础（Base）

##### ArtBackToTop

- **用途**：在长列表中提供快速回到顶部的浮动按钮，提升可达性。【F:src/components/core/base/art-back-to-top/index.vue†L1-L44】
- **解剖**：固定容器（`position: fixed`）+ 图标区域 + 文本标签。
- **变体**：默认（浅色描边）；可通过自定义 class 调整颜色（可迭代项：扩展圆形/紧凑变体）。
- **尺寸**：38×38px，含 2px 文本，间距 40px/60px（与边缘）。
- **状态**：默认/hover（背景高亮）。Focus/Active 需增加 `outline`（可迭代项）；禁用通过隐藏按钮处理。
- **可访问性**：建议添加 `aria-label="返回顶部"`，并在键盘上使用 `role="button"` 与 `tabindex="0"`（待补）。
- **内容准则**：保持「顶部」双字或业务文案 ≤4 字。
- **使用示例**：

  ```vue
  <template>
    <ArtBackToTop class="backtop--brand" />
  </template>

  <style scoped>
    .backtop--brand {
      right: 24px; /* TODO: 根据布局调整位置 */
    }
  </style>
  ```

- **组合模式**：常与 `ArtNotification`、`ArtWorkTab` 并存，注意错开 z-index。

##### ArtIconSelector

- **用途**：提供 iconfont 图标选择弹窗，支持 className/Unicode 两种渲染方式。【F:src/components/core/base/art-icon-selector/index.vue†L1-L127】
- **解剖**：触发器（图标+文字+清除按钮）+ 弹窗列表 + 滚动容器。
- **变体**：`size`（large/default/small）、`iconType`（className/unicode）。
- **尺寸**：触发器高度与 `--el-component-custom-height` 对齐；宽度可配置。
- **状态**：默认/hover/disabled（`is-disabled` class）；弹窗内 hover 高亮。
- **可访问性**：弹窗为 `ElDialog`，建议在图标列表 li 上添加 `role="option"`；键盘导航（可迭代项）。
- **内容准则**：默认文案「图标选择器」，可替换；图标列表需避免空数据。
- **使用示例**：

  ```vue
  <script setup lang="ts">
    import ArtIconSelector from '@/components/core/base/art-icon-selector/index.vue'
    const icon = ref('')
  </script>

  <template>
    <ArtIconSelector v-model="icon" text="选择图标" />
  </template>
  ```

- **组合模式**：与 `ArtForm` 自定义项或 `SettingPanel` 联动，为主题/icon 配置提供选择器。

##### ArtLogo

- **用途**：渲染品牌 Logo，默认 36px 宽度，可嵌入侧边栏/登录头部。【F:src/components/core/base/art-logo/index.vue†L1-L26】
- **解剖**：包裹容器 + `<img>`。
- **变体**：`size` 属性控制宽度；可迭代项：支持自定义 src/alt。
- **尺寸**：建议保留 8px 以上外边距，适配侧栏高度。
- **状态**：静态；若需 hover 切换暗色版 Logo，可在父级处理。
- **可访问性**：需提供 `alt` 描述，避免空值（当前默认 `alt="logo"`，可替换）。
- **内容准则**：Logo 文件放置于 `@imgs/common`，确保多分辨率资源。
- **使用示例**：
  ```vue
  <template>
    <ArtLogo :size="48" />
  </template>
  ```
- **组合模式**：与导航组件（`ArtSidebarMenu`、`ArtHeaderBar`）联合使用，注意左右留白对齐。

#### 横幅（Banners）

##### ArtBasicBanner

- **用途**：在仪表盘/营销页展示宣传信息，支持标题、副标题、按钮与背景装饰。【F:src/components/core/banners/art-basic-banner/index.vue†L1-L120】
- **解剖**：容器（含可选装饰/流星效果）+ 文案区（title/subtitle/default slot）+ CTA 按钮 + 背景图层。
- **变体**：`meteorConfig` 控制夜空流星（仅 Dark 模式建议开启）；按钮可通过 `buttonConfig` 设定；`decoration` 切换装饰。
- **尺寸**：默认高度 11rem；建议标题 24px、副标题 16px，CTA 高度 ≥40px。
- **状态**：默认/hover（父级 click）；按钮含 hover/focus/disabled（需在 slot 中处理）；loading 无原生支持（可迭代项）。
- **可访问性**：Banner 容器可加 `role="region"` 与 `aria-labelledby`；CTA 使用按钮元素保持键盘可操作。
- **内容准则**：标题 ≤20 字，副标题 ≤48 字，CTA 使用动词开头。Dark 模式确保文案颜色 ≥4.5:1。
- **使用示例**：

  ```vue
  <script setup lang="ts">
    import ArtBasicBanner from '@/components/core/banners/art-basic-banner/index.vue'
    const meteor = { enabled: true, count: 8 } // TODO: 根据节日切换
  </script>

  <template>
    <ArtBasicBanner
      title="营销活动"
      subtitle="提交作品参与抽奖"
      :button-config="{
        show: true,
        text: '立即参与',
        color: 'rgb(var(--art-primary))',
        textColor: '#fff'
      }"
      :meteor-config="meteor"
      @buttonClick="
        () => {
          /* TODO */
        }
      "
    />
  </template>
  ```

- **组合模式**：可叠加在 `ArtCardBanner` 列表上方，或与 `ArtStatsCard` 组合形成 KPI 区。

##### ArtCardBanner

- **用途**：展示图文卡片横幅，适合入门引导、模版推荐等场景。【F:src/components/core/banners/art-card-banner/index.vue†L1-L120】
- **解剖**：容器 + 图片区域 + 文案区域（title/description）+ 主/次按钮。
- **变体**：支持 `button`/`cancelButton` 配置；可通过 slot 自定义按钮（可迭代项）。
- **尺寸**：默认高度 24rem；图片宽度 180px，文本区域 16px 内边距。
- **状态**：按钮 hover/active；建议为取消按钮添加描边区分；loading 通过禁用按钮实现。
- **可访问性**：按钮使用 div 需加 `role="button"` 与 `tabindex`（可迭代项：改用 `<button>`）。图片需提供 `alt`。
- **内容准则**：标题 ≤16 字，描述 ≤60 字，按钮文案动词化。
- **使用示例**：

  ```vue
  <script setup lang="ts">
    import ArtCardBanner from '@/components/core/banners/art-card-banner/index.vue'
    const handlePrimary = () => {
      /* TODO: 跳转路由 */
    }
  </script>

  <template>
    <ArtCardBanner
      title="快速上手"
      description="五分钟完成数据看板搭建"
      :button="{ show: true, text: '查看教程' }"
      :cancel-button="{ show: true, text: '忽略', color: '#f1f1f4', textColor: '#4b5675' }"
      @click="handlePrimary"
    />
  </template>
  ```

- **组合模式**：可置于 `ArtPageContent` 顶部，与 `ArtTimelineListCard` 组合构建 onboarding 区域。

#### 卡片（Cards）

##### ArtStatsCard

- **用途**：展示关键 KPI，支持数字动态计数、描述与图标装饰。【F:src/components/core/cards/art-stats-card/index.vue†L1-L88】
- **解剖**：容器 + 图标块 + 内容块（标题/数值/描述）+ 可选箭头。
- **变体**：通过 props 控制 icon、背景、文本色；可配置 `showArrow`。
- **尺寸**：高度 8rem，左右 padding 20px；图标容器 46px。
- **状态**：hover 微动效；建议添加 focus 样式；loading 时可在外部用 skeleton 替代（可迭代项）。
- **可访问性**：卡片可设 `role="button"` 并加入 `aria-describedby`；数值使用 `<ArtCountTo>` 支持辅助朗读（建议提供 `aria-live`）。
- **内容准则**：标题 ≤12 字，描述 ≤30 字；数值格式与业务一致。
- **使用示例**：

  ```vue
  <script setup lang="ts">
    import ArtStatsCard from '@/components/core/cards/art-stats-card/index.vue'
  </script>

  <template>
    <ArtStatsCard
      title="总收入"
      :count="128000"
      description="同比增长 12%"
      icon="&#xe60c;"
      icon-color="#fff"
      icon-bg-color="rgb(var(--art-primary))"
    />
  </template>
  ```

- **组合模式**：与 `ArtLineChartCard`/`ArtProgressCard` 组成 KPI 区；可嵌入 `ArtDataListCard` 下方。

##### ArtLineChartCard

- **用途**：在卡片内呈现迷你折线图与核心指标。【F:src/components/core/cards/art-line-chart-card/index.vue†L1-L110】
- **解剖**：顶部信息区（值/标签/百分比/日期）+ 图表容器。
- **变体**：支持 `isMiniChart`、`color`、`showAreaColor`；高度可配置。
- **尺寸**：默认高度 11rem；图表区域 = 高度 - 5rem。
- **状态**：默认静态；hover 可通过 `chartRef` 自定义 tooltip；loading 通过外部 Skeleton。
- **可访问性**：图表需提供文本替代（例如 `aria-label` 描述数据趋势）。
- **内容准则**：`value` 使用格式化数值；`percentage` 保留 ±号。
- **使用示例**：

  ```vue
  <script setup lang="ts">
    import ArtLineChartCard from '@/components/core/cards/art-line-chart-card/index.vue'
    const chartData = [12, 16, 14, 22, 28, 26]
  </script>

  <template>
    <ArtLineChartCard
      :value="'￥58,230'"
      label="本月营收"
      :percentage="8.6"
      date="2024-05"
      :chart-data="chartData"
      :show-area-color="true"
    />
  </template>
  ```

- **组合模式**：常与 `ArtBarChartCard` 形成图形对比区域。

##### ArtBarChartCard

- **用途**：展示柱状图概览，适合对比类 KPI。【F:src/components/core/cards/art-bar-chart-card/index.vue†L1-L140】
- **解剖**：头部标题区 + 数值说明 + 图表容器（内嵌 ECharts）。
- **变体**：`chartType`（单/多数据）、`legend` 显示控制；背景可通过 props 自定义。
- **尺寸**：默认高度与 `ArtLineChartCard` 一致；图表内边距 20px。
- **状态**：hover 显示 tooltip；空数据展示空状态（ECharts 配置）。
- **可访问性**：同样需文字摘要；图表颜色需符合语义色对比。
- **使用示例**：

  ```vue
  <script setup lang="ts">
    import ArtBarChartCard from '@/components/core/cards/art-bar-chart-card/index.vue'
    const dataset = [12, 28, 14, 32]
  </script>

  <template>
    <ArtBarChartCard title="渠道访客" :chart-data="dataset" />
  </template>
  ```

- **组合模式**：与筛选组件 `ArtSearchBar` 联动，实现时间区间切换。

##### ArtDonutChartCard

- **用途**：呈现环形图及占比说明，用于组成结构展示。【F:src/components/core/cards/art-donut-chart-card/index.vue†L1-L140】
- **解剖**：标题栏 + 图表区 + Legend 列表。
- **变体**：可设置 `innerRadius`/`outerRadius`、`legendPosition`；支持空状态。
- **尺寸**：建议最小宽度 320px，高度 11rem。
- **状态**：hover 显示 tooltip；loading/empty 需通过外部控制。
- **可访问性**：建议提供 `aria-describedby` 描述占比。
- **使用示例**：

  ```vue
  <script setup lang="ts">
    import ArtDonutChartCard from '@/components/core/cards/art-donut-chart-card/index.vue'
    const source = [
      { name: '搜索', value: 45 },
      { name: '推荐', value: 30 },
      { name: '广告', value: 25 }
    ]
  </script>

  <template>
    <ArtDonutChartCard title="来源占比" :data="source" />
  </template>
  ```

- **组合模式**：与 `ArtProgressCard` 对比任务完成度。

##### ArtProgressCard

- **用途**：展示单个指标进度，配合数值动画增强反馈。【F:src/components/core/cards/art-progress-card/index.vue†L1-L90】
- **解剖**：信息区（图标+百分比+标题）+ `ElProgress` 进度条。
- **变体**：自定义颜色、图标、圆角、线宽。
- **尺寸**：高度 8rem，进度条默认宽度 5px。
- **状态**：动画加载/更新；建议添加 `aria-valuenow` 属性（可迭代项）。
- **可访问性**：使用 `ElProgress` 已含可访问属性，需补充 `aria-label`。
- **使用示例**：

  ```vue
  <script setup lang="ts">
    import ArtProgressCard from '@/components/core/cards/art-progress-card/index.vue'
  </script>

  <template>
    <ArtProgressCard
      title="项目完成率"
      :percentage="72"
      icon="&#xe639;"
      icon-color="#fff"
      icon-bg-color="rgb(var(--art-primary))"
    />
  </template>
  ```

- **组合模式**：与 `ArtStatsCard` 并列展示完成情况；可在 `ArtDataListCard` 下方作为汇总。

##### ArtDataListCard

- **用途**：承载通知、任务等列表数据，支持滚动与更多按钮。【F:src/components/core/cards/art-data-list-card/index.vue†L1-L96】
- **解剖**：卡片容器 + 头部 + `ElScrollbar` 列表 + 更多按钮。
- **变体**：`maxCount` 控制可视数量；可通过 slot 自定义列表项目（可迭代项）。
- **尺寸**：默认 item 高度 66px，卡片 padding 30px。
- **状态**：滚动、hover；更多按钮可加禁用/加载状态。
- **可访问性**：列表项需提供键盘交互（可迭代项）；滚动条样式遵循全局 reset。
- **使用示例**：

  ```vue
  <script setup lang="ts">
    import ArtDataListCard from '@/components/core/cards/art-data-list-card/index.vue'
    const activities = [
      {
        title: '完成 UI 设计',
        status: '待审核',
        time: '12:30',
        class: 'is-primary',
        icon: '&#xe60f;'
      }
    ]
  </script>

  <template>
    <ArtDataListCard
      title="最新动态"
      subtitle="最近 7 日"
      :list="activities"
      :show-more-button="true"
      @more="
        () => {
          /* TODO */
        }
      "
    />
  </template>
  ```

- **组合模式**：常与 `ArtTimelineListCard`、`ArtNotification` 结合形成活动中心。

##### ArtImageCard

- **用途**：展示图片 + 文案 + 操作，适合展示资产或案例。【F:src/components/core/cards/art-image-card/index.vue†L1-L120】
- **解剖**：顶图区域 + 信息区（标题、副标题、标签、按钮）。
- **变体**：`tag`、`button` 配置；支持 `hover` 动画。
- **尺寸**：图片高度 160px；卡片圆角取自 `--custom-radius`。
- **状态**：hover 放大图片；按钮 hover/focus；loading 需外部 skeleton。
- **可访问性**：图片 `alt`；标签/按钮需明确语义。
- **使用示例**：

  ```vue
  <script setup lang="ts">
  import ArtImageCard from '@/components/core/cards/art-image-card/index.vue'
  </script>

  <template>
    <ArtImageCard
      title="视觉模版"
      subtitle="更新于今天"
      image="/assets/demo.jpg" <!-- TODO: 替换业务图片 -->
      :button="{ show: true, text: '查看', type: 'primary' }"
    />
  </template>
  ```

- **组合模式**：与 `ArtBasicBanner`/`ArtCardBanner` 搭配形成推荐位。

##### ArtTimelineListCard

- **用途**：时间轴样式列表，用于展示活动进度或操作日志。【F:src/components/core/cards/art-timeline-list-card/index.vue†L1-L100】
- **解剖**：卡片容器 + 头部 + 时间线条目（点+内容+标签）。
- **变体**：支持传入 `statusColor`、`tag`；可自定义插槽扩展内容。
- **尺寸**：列表项 padding 16px；时间轴线宽 2px。
- **状态**：hover；可与 `ArtEmpty` 结合处理空状态（可迭代项：提供内置空态）。
- **可访问性**：时间线应提供 `aria-label` 描述事件；tab 序列按时间排序。
- **使用示例**：

  ```vue
  <script setup lang="ts">
    import ArtTimelineListCard from '@/components/core/cards/art-timeline-list-card/index.vue'
    const timelines = [{ title: '需求评审', time: '09:30', tag: '完成', color: '#5d87ff' }]
  </script>

  <template>
    <ArtTimelineListCard title="项目进度" :list="timelines" />
  </template>
  ```

- **组合模式**：放置于 `ArtPageContent` 左侧，与 `ArtDataListCard`、`ArtProgressCard` 协同展示项目视图。

#### 图表（Charts）

##### ArtBarChart

- **用途**：通用柱状图，支持单/多序列及渐变色配置。【F:src/components/core/charts/art-bar-chart/index.vue†L1-L108】
- **解剖**：外层容器 + `useChartComponent` 生成的 ECharts option。
- **变体**：`data` 可为数组或对象；`showLegend`、`stack` 控制堆叠。
- **状态**：loading/empty 由 props 控制；hover tooltip 自动。
- **可访问性**：为容器添加 `role="img"` 与 `aria-label` 描述。
- **使用示例**：

  ```vue
  <script setup lang="ts">
    import ArtBarChart from '@/components/core/charts/art-bar-chart/index.vue'
    const bars = [12, 34, 18, 26]
  </script>

  <template>
    <ArtBarChart :data="bars" :x-axis-data="['Q1', 'Q2', 'Q3', 'Q4']" />
  </template>
  ```

##### ArtDualBarCompareChart

- **用途**：正负向对比柱状图，适合性别、满意度等双向数据。【F:src/components/core/charts/art-dual-bar-compare-chart/index.vue†L1-L120】
- **解剖**：容器 + 正/负 series；`positiveData`、`negativeData`。
- **变体**：`showDataLabel`、`showLegend`、`legendPosition`。
- **状态**：tooltip axis；空数据 fallback。
- **可访问性**：请提供 `aria-describedby` 描述正负含义。
- **使用示例**：

  ```vue
  <script setup lang="ts">
    import ArtDualBarCompareChart from '@/components/core/charts/art-dual-bar-compare-chart/index.vue'
    const positive = [40, 32, 28]
    const negative = [20, 18, 14]
  </script>

  <template>
    <ArtDualBarCompareChart
      :positive-data="positive"
      :negative-data="negative"
      :x-axis-data="['满意', '一般', '不满']"
    />
  </template>
  ```

##### ArtHBarChart

- **用途**：横向柱状图，适合排名榜单。【F:src/components/core/charts/art-h-bar-chart/index.vue†L1-L120】
- **解剖**：横向 y 轴分类 + tooltip + 自定义色板。
- **变体**：`showRank`、`barHeight`、`labelPosition`。
- **状态**：hover tooltip；空数据检查。
- **可访问性**：为排名提供文本列表作为替代（可迭代项）。
- **使用示例**：

  ```vue
  <script setup lang="ts">
    import ArtHBarChart from '@/components/core/charts/art-h-bar-chart/index.vue'
    const series = [
      { name: '产品 A', value: 80 },
      { name: '产品 B', value: 56 }
    ]
  </script>

  <template>
    <ArtHBarChart :data="series" />
  </template>
  ```

##### ArtKLineChart

- **用途**：金融行情 K 线图，支持均线、阴阳线样式。【F:src/components/core/charts/art-k-line-chart/index.vue†L1-L140】
- **解剖**：candlestick series + 均线 series + 数据缩放。
- **变体**：`showMA`、`maDays`、`showVolume`。
- **状态**：数据更新时重绘；空数据 fallback。
- **可访问性**：提供价格摘要文本，满足读屏需求。
- **使用示例**：

  ```vue
  <script setup lang="ts">
    import ArtKLineChart from '@/components/core/charts/art-k-line-chart/index.vue'
    const candles = {
      categoryData: ['2024-05-01', '2024-05-02'],
      values: [
        [2320.26, 2320.26, 2287.3, 2362.94],
        [2300, 2291.3, 2288.26, 2308.38]
      ]
    }
  </script>

  <template>
    <ArtKLineChart :chart-data="candles" />
  </template>
  ```

##### ArtLineChart

- **用途**：标准折线图，支持面积填充与多序列。【F:src/components/core/charts/art-line-chart/index.vue†L1-L120】
- **解剖**：X/Y 轴 + series + tooltip + legend。
- **变体**：`smooth`、`areaStyle`、`stack`。
- **状态**：loading/empty；hover tooltip。
- **可访问性**：提供 `aria-label` 说明趋势。
- **使用示例**：

  ```vue
  <script setup lang="ts">
    import ArtLineChart from '@/components/core/charts/art-line-chart/index.vue'
    const series = [{ name: '访问量', data: [10, 20, 15, 30] }]
  </script>

  <template>
    <ArtLineChart :series="series" :x-axis-data="['周一', '周二', '周三', '周四']" />
  </template>
  ```

##### ArtMapChart

- **用途**：中国地图可视化，支持区域高亮与点击事件。【F:src/components/core/charts/art-map-chart/index.vue†L1-L120】
- **解剖**：外层容器 + `ElEmpty` 空态 + ECharts geo map。
- **变体**：`showLabels`、`showScatter`、`selectedRegion`。
- **状态**：空数据展示空态；hover 区域高亮。
- **可访问性**：提供表格或列表同步展示数据；tooltip 文案包含数值说明。
- **使用示例**：

  ```vue
  <script setup lang="ts">
    import ArtMapChart from '@/components/core/charts/art-map-chart/index.vue'
  </script>

  <template>
    <ArtMapChart :map-data="[]" />
  </template>
  ```

##### ArtRadarChart

- **用途**：雷达图展示多维指标。【F:src/components/core/charts/art-radar-chart/index.vue†L1-L120】
- **解剖**：radar indicator 配置 + series 填充。
- **变体**：`shape`（circle/polygon）、`showArea`、`legendPosition`。
- **状态**：hover 展示 tooltip；空数据 fallback。
- **可访问性**：在外围提供指标描述列表。
- **使用示例**：

  ```vue
  <script setup lang="ts">
    import ArtRadarChart from '@/components/core/charts/art-radar-chart/index.vue'
    const indicators = [
      { name: '设计', max: 100 },
      { name: '交互', max: 100 }
    ]
    const values = [{ value: [80, 65], name: '团队 A' }]
  </script>

  <template>
    <ArtRadarChart :indicator="indicators" :series="values" />
  </template>
  ```

##### ArtRingChart

- **用途**：环形图（与 `ArtDonutChartCard` 相同数据核心），独立组件可嵌入其他布局。【F:src/components/core/charts/art-ring-chart/index.vue†L1-L120】
- **解剖**：`pie` series + legend。
- **变体**：`legendPosition`、`innerRadius`/`outerRadius`。
- **状态**：tooltip；空数据；loading。
- **可访问性**：提供占比文本。
- **使用示例**：

  ```vue
  <script setup lang="ts">
    import ArtRingChart from '@/components/core/charts/art-ring-chart/index.vue'
    const data = [
      { name: 'A', value: 45 },
      { name: 'B', value: 55 }
    ]
  </script>

  <template>
    <ArtRingChart :data="data" legend-position="right" />
  </template>
  ```

##### ArtScatterChart

- **用途**：散点图，可用作气泡分析。【F:src/components/core/charts/art-scatter-chart/index.vue†L1-L120】
- **解剖**：series scatter + tooltip + 视觉映射。
- **变体**：`symbolSize`、`showRegression`、`showLegend`。
- **状态**：hover tooltip；空数据 fallback。
- **可访问性**：提供主要趋势描述；如有回归线需标注方程。
- **使用示例**：

  ```vue
  <script setup lang="ts">
    import ArtScatterChart from '@/components/core/charts/art-scatter-chart/index.vue'
    const points = [
      [10, 20],
      [30, 45],
      [25, 30]
    ]
  </script>

  <template>
    <ArtScatterChart :data="points" />
  </template>
  ```

#### 表单（Forms）

##### ArtForm

- **用途**：基于 Element Plus 的动态表单生成器，支持多种控件类型、插槽、自定义校验与响应式布局。【F:src/components/core/forms/art-form/index.vue†L1-L120】
- **解剖**：`ElForm` + `ElRow/ElCol` 栅格 + 动态组件渲染 + 操作区。
- **变体**：`items` 控制字段；`buttonLeftLimit` 决定按钮对齐；支持自定义 slot。
- **状态**：表单校验、禁用提交、隐藏字段；移动端自动单列。
- **可访问性**：遵循 Element Form 无障碍；需确保 label/prop 配置完整。
- **使用示例**：

  ```vue
  <script setup lang="ts">
    import ArtForm from '@/components/core/forms/art-form/index.vue'
    const model = reactive({ keyword: '', status: '' })
    const items = [
      { key: 'keyword', label: '关键词', type: 'input', props: { placeholder: '请输入' } },
      {
        key: 'status',
        label: '状态',
        type: 'select',
        props: { options: [{ label: '启用', value: 1 }] }
      }
    ]
  </script>

  <template>
    <ArtForm
      v-model="model"
      :items="items"
      @submit="
        () => {
          /* TODO */
        }
      "
    />
  </template>
  ```

##### ArtButtonTable

- **用途**：表格快捷操作按钮，提供多种图标类型与背景色映射。【F:src/components/core/forms/art-button-table/index.vue†L1-L80】
- **解剖**：按钮容器 + iconfont。
- **变体**：`type`（add/edit/delete/view/more）、自定义图标/背景。
- **状态**：hover 变色；禁用通过父级控制。
- **可访问性**：添加 `aria-label` 描述操作。
- **使用示例**：
  ```vue
  <template>
    <ArtButtonTable type="edit" @click="onEdit" />
  </template>
  ```

##### ArtButtonMore

- **用途**：表格「更多」操作下拉，结合权限过滤。【F:src/components/core/forms/art-button-more/index.vue†L1-L80】
- **解剖**：`ElDropdown` + `ArtButtonTable` 触发器 + 列表项。
- **变体**：`list` 项配置、`hasBackground` 控制背景；支持权限 (`auth`)。
- **状态**：hover 高亮、禁用项置灰。
- **可访问性**：Dropdown 支持键盘；需为图标提供文本。
- **使用示例**：

  ```vue
  <script setup lang="ts">
    import ArtButtonMore from '@/components/core/forms/art-button-more/index.vue'
    const actions = [{ key: 'download', label: '下载', icon: Document }] // TODO: 替换图标
  </script>

  <template>
    <ArtButtonMore
      :list="actions"
      @click="
        (item) => {
          /* TODO */
        }
      "
    />
  </template>
  ```

##### ArtDragVerify

- **用途**：滑块拖拽验证组件，防止机器人登录。【F:src/components/core/forms/art-drag-verify/index.vue†L1-L108】
- **解剖**：容器 + 进度条 + 提示文本 + 滑块 handle。
- **变体**：宽高、文本、颜色、圆角、图标均可配置。
- **状态**：拖拽中、成功（`value` true）、重置；移动端触摸支持。
- **可访问性**：提供 `aria-valuenow`、`aria-valuemin/max`（可迭代项）；键盘支持待补充。
- **使用示例**：
  ```vue
  <template>
    <ArtDragVerify v-model="passed" :text="'向右拖动完成验证'" />
  </template>
  ```

##### ArtExcelExport

- **用途**：封装 XLSX 导出按钮，支持列配置、序号、自定义消息。【F:src/components/core/forms/art-excel-export/index.vue†L1-L120】
- **解剖**：`ElButton` + 导出逻辑（XLSX + FileSaver）。
- **变体**：`autoIndex`、`columns`、`headers`、`maxRows`、`buttonText`。
- **状态**：`isExporting` loading、禁用无数据、错误提示。
- **可访问性**：按钮需描述导出内容；导出完成可播报成功消息。
- **使用示例**：

  ```vue
  <script setup lang="ts">
    import ArtExcelExport from '@/components/core/forms/art-excel-export/index.vue'
    const dataset = [{ name: '张三', score: 98 }]
  </script>

  <template>
    <ArtExcelExport
      :data="dataset"
      filename="report"
      @export-success="
        () => {
          /* TODO */
        }
      "
    />
  </template>
  ```

##### ArtExcelImport

- **用途**：Excel 文件解析导入，支持模板下载、字段映射与错误提示。【F:src/components/core/forms/art-excel-import/index.vue†L1-L140】
- **解剖**：上传按钮 + 模板下载 + 文件解析。
- **变体**：`beforeUpload`、`onSuccess`、`fields` 映射、`maxSize`、`accept`。
- **状态**：上传中、成功、失败；loading 提示。
- **可访问性**：使用 `ElUpload`；需为隐藏 input 提供标签文本。
- **使用示例**：
  ```vue
  <template>
    <ArtExcelImport @success="(rows) => {/* TODO */}}" />
  </template>
  ```

##### ArtSearchBar

- **用途**：复杂搜索面板，支持展开/收起、栅格布局、快捷操作按钮。【F:src/components/core/forms/art-search-bar/index.vue†L1-L200】
- **解剖**：搜索项列表 + 快捷按钮区 + 折叠控制。
- **变体**：`mode`（simple/advanced）、`layout`、`actions`、`showCollapse`。
- **状态**：折叠/展开、loading 按钮禁用、校验提示。
- **可访问性**：按钮需提供 `aria-expanded`；控件继承 Element 无障碍。
- **使用示例**：

  ```vue
  <script setup lang="ts">
    import ArtSearchBar from '@/components/core/forms/art-search-bar/index.vue'
    const filters = reactive({ keyword: '' })
    const items = [
      { key: 'keyword', label: '关键词', type: 'input', props: { placeholder: '搜索' } }
    ]
  </script>

  <template>
    <ArtSearchBar v-model="filters" :items="items" />
  </template>
  ```

##### ArtWangEditor

- **用途**：富文本编辑器封装，整合 wangEditor 实现 toolbar/内容/上传配置。【F:src/components/core/forms/art-wang-editor/index.vue†L1-L160】
- **解剖**：工具栏 + 编辑区域 + 自定义插槽（上传、快捷键）。
- **变体**：`mode`（default/simple）、`height`、`upload` 配置、快捷工具。
- **状态**：loading/禁用；输入计数；支持表单 v-model。
- **可访问性**：富文本需提供 `aria-label`；内嵌 iframe 需处理焦点。
- **使用示例**：
  ```vue
  <template>
    <ArtWangEditor v-model="content" :height="420" />
  </template>
  ```

#### 布局（Layouts）

##### ArtBreadcrumb

- **用途**：渲染面包屑路径，配合路由元数据显示层级。【F:src/components/core/layouts/art-breadcrumb/index.vue†L1-L120】
- **解剖**：`ElBreadcrumb` + 列表项 + 图标。
- **变体**：支持隐藏首页、图标插槽。
- **状态**：hover 高亮；active 与当前路由匹配。
- **使用示例**：
  ```vue
  <template>
    <ArtBreadcrumb :separator-icon="ArrowRight" />
  </template>
  ```

##### ArtChatWindow

- **用途**：内置客服聊天面板，支持消息列表、快捷回复、折叠控制。【F:src/components/core/layouts/art-chat-window/index.vue†L1-L200】
- **解剖**：浮动按钮 + 消息框（头/主体/输入）。
- **变体**：位置、主题色、消息气泡、快捷操作。
- **状态**：展开/收起、输入中、加载。
- **使用示例**：
  ```vue
  <template>
    <ArtChatWindow v-model:visible="showChat" />
  </template>
  ```

##### ArtFastEnter

- **用途**：首页快捷入口，展示常用菜单列表。【F:src/components/core/layouts/art-fast-enter/index.vue†L1-L100】
- **解剖**：卡片容器 + 列表项（图标+标题）。
- **变体**：列数、展示数、排序。
- **使用示例**：
  ```vue
  <template>
    <ArtFastEnter :list="quickMenus" />
  </template>
  ```

##### ArtFireworksEffect

- **用途**：节日烟花动效，支持节日开关。【F:src/components/core/layouts/art-fireworks-effect/index.vue†L1-L140】
- **解剖**：canvas 动画 + 节日文本。
- **变体**：`isShow`、`autoHide`。
- **使用示例**：
  ```vue
  <template>
    <ArtFireworksEffect v-if="showFestival" />
  </template>
  ```

##### ArtGlobalComponent

- **用途**：全局通用挂载点，组合 `ArtNotification`、`ArtChatWindow` 等。【F:src/components/core/layouts/art-global-component/index.vue†L1-L120】
- **解剖**：多个组件组合渲染。
- **使用示例**：无需直接使用，由 `App.vue` 引入。

##### ArtGlobalSearch

- **用途**：全局搜索面板，提供快捷命令、历史记录、键盘导航。【F:src/components/core/layouts/art-global-search/index.vue†L1-L160】
- **解剖**：触发按钮 + 搜索输入 + 结果列表 + 快捷键提示。
- **变体**：`showHotKeys`、`historyList`、`size`。
- **状态**：聚焦、loading、为空时提示。
- **使用示例**：
  ```vue
  <template>
    <ArtGlobalSearch v-model:visible="showSearch" />
  </template>
  ```

##### ArtHeaderBar

- **用途**：顶部导航栏，包含面包屑、搜索、语言、用户信息、主题开关等。【F:src/components/core/layouts/art-header-bar/index.vue†L1-L200】
- **解剖**：左侧导航按钮 + 面包屑 + 右侧功能区（搜索、设置、消息、语言等）。
- **变体**：通过设置面板控制显示项；支持暗色。
- **状态**：hover、下拉；支持 popover。
- **使用示例**：
  ```vue
  <template>
    <ArtHeaderBar />
  </template>
  ```

##### ArtMenus 系列

- **ArtHorizontalMenu**：顶部导航菜单，含 `HorizontalSubmenu` 递归组件。【F:src/components/core/layouts/art-menus/art-horizontal-menu/index.vue†L1-L160】【F:src/components/core/layouts/art-menus/art-horizontal-menu/widget/HorizontalSubmenu.vue†L1-L160】
- **ArtMixedMenu**：顶部+侧边混合菜单，支持折叠与 hover 展开。【F:src/components/core/layouts/art-menus/art-mixed-menu/index.vue†L1-L160】
- **ArtSidebarMenu**：左侧菜单，`SidebarSubmenu` 支持多级嵌套与徽标。【F:src/components/core/layouts/art-menus/art-sidebar-menu/index.vue†L1-L200】【F:src/components/core/layouts/art-menus/art-sidebar-menu/widget/SidebarSubmenu.vue†L1-L160】
- **使用示例**：
  ```vue
  <template>
    <ArtSidebarMenu :menu-list="menus" />
  </template>
  ```

##### ArtNotification

- **用途**：顶部消息通知列表，支持清空、标签页切换。【F:src/components/core/layouts/art-notification/index.vue†L1-L160】
- **解剖**：icon 触发 + Drawer 弹层 + 列表/空态。
- **状态**：读取、未读、加载。
- **使用示例**：
  ```vue
  <template>
    <ArtNotification />
  </template>
  ```

##### ArtPageContent

- **用途**：主内容容器，处理路由缓存、过渡、全屏页面。【F:src/components/core/layouts/art-page-content/index.vue†L1-L120】
- **解剖**：`RouterView` + `Transition` + `KeepAlive` + 遮罩。
- **状态**：刷新、全屏、动画遮罩。
- **使用示例**：
  ```vue
  <template>
    <ArtPageContent />
  </template>
  ```

##### ArtScreenLock

- **用途**：屏幕锁定界面，包含密码验证与背景图。【F:src/components/core/layouts/art-screen-lock/index.vue†L1-L160】
- **解剖**：锁屏表单 + 背景 + 动画。
- **使用示例**：
  ```vue
  <template>
    <ArtScreenLock v-if="locked" />
  </template>
  ```

##### ArtSettingsPanel & Widgets

- **ArtSettingsPanel**：右侧设置抽屉，集中管理主题、布局、粒度配置。【F:src/components/core/layouts/art-settings-panel/index.vue†L1-L120】
- **SettingDrawer/SettingHeader**：抽屉容器与标题栏。【F:src/components/core/layouts/art-settings-panel/widget/SettingDrawer.vue†L1-L160】【F:src/components/core/layouts/art-settings-panel/widget/SettingHeader.vue†L1-L120】
- **SectionTitle**：分组标题组件。【F:src/components/core/layouts/art-settings-panel/widget/SectionTitle.vue†L1-L60】
- **SettingItem**：复用型行组件，含标题、说明、插槽。【F:src/components/core/layouts/art-settings-panel/widget/SettingItem.vue†L1-L120】
- **BasicSettings/BoxStyleSettings/ColorSettings/ContainerSettings/MenuLayoutSettings/MenuStyleSettings/ThemeSettings**：分别负责基础开关、边框模式、主题色、容器宽度、菜单布局/样式、主题模式配置。【F:src/components/core/layouts/art-settings-panel/widget/BasicSettings.vue†L1-L160】【F:src/components/core/layouts/art-settings-panel/widget/BoxStyleSettings.vue†L1-L140】【F:src/components/core/layouts/art-settings-panel/widget/ColorSettings.vue†L1-L180】【F:src/components/core/layouts/art-settings-panel/widget/ContainerSettings.vue†L1-L140】【F:src/components/core/layouts/art-settings-panel/widget/MenuLayoutSettings.vue†L1-L160】【F:src/components/core/layouts/art-settings-panel/widget/MenuStyleSettings.vue†L1-L180】【F:src/components/core/layouts/art-settings-panel/widget/ThemeSettings.vue†L1-L160】
- **使用示例**：
  ```vue
  <template>
    <ArtSettingsPanel />
  </template>
  ```
- **组合**：SettingItem + Switch/Button/Slider 组合呈现，可通过 `SettingDrawer` 控制显示。

##### ArtWorkTab

- **用途**：多页签工作台，支持拖拽排序、关闭、刷新。【F:src/components/core/layouts/art-work-tab/index.vue†L1-L200】
- **解剖**：页签头 + 操作按钮 + 下拉菜单。
- **状态**：激活、未激活、更多操作、锁定。
- **使用示例**：
  ```vue
  <template>
    <ArtWorkTab />
  </template>
  ```

#### 媒体（Media）

##### ArtCutterImg

- **用途**：图片裁剪上传组件，支持预览、缩放、旋转。【F:src/components/core/media/art-cutter-img/index.vue†L1-L160】
- **解剖**：上传区域 + 预览框 + 操作按钮。
- **变体**：裁剪比例、输出格式、压缩质量。
- **使用示例**：
  ```vue
  <template>
    <ArtCutterImg
      @success="
        (file) => {
          /* TODO */
        }
      "
    />
  </template>
  ```

##### ArtVideoPlayer

- **用途**：视频播放组件，封装 video.js 样式控制。【F:src/components/core/media/art-video-player/index.vue†L1-L120】
- **解剖**：video 元素 + 控制条。
- **变体**：播放源、poster、自动播放、倍速。
- **使用示例**：
  ```vue
  <template>
    <ArtVideoPlayer src="/video/demo.mp4" poster="/img/poster.jpg" />
  </template>
  ```

#### 其他（Others）

##### ArtMenuRight

- **用途**：右键菜单，支持自定义项与快捷操作。【F:src/components/core/others/art-menu-right/index.vue†L1-L120】
- **解剖**：contextmenu 触发 + 菜单项列表。
- **状态**：显示/隐藏、禁用项。
- **使用示例**：
  ```vue
  <template>
    <ArtMenuRight :menus="contextMenus" />
  </template>
  ```

##### ArtWatermark

- **用途**：为页面添加文字/图片水印，支持全局配置与实时更新。【F:src/components/core/others/art-watermark/index.vue†L1-L160】
- **解剖**：Canvas 绘制 + Observer 监听。
- **变体**：内容、透明度、间距、旋转角。
- **使用示例**：
  ```vue
  <template>
    <ArtWatermark text="内部资料" />
  </template>
  ```

#### 表格（Tables）

##### ArtTableHeader

- **用途**：表格头部操作面板，支持全屏、密度、斑马纹等开关。【F:src/components/core/tables/art-table-header/index.vue†L1-L160】
- **解剖**：标题区 + 操作按钮（刷新、导出、列设置）。
- **使用示例**：
  ```vue
  <template>
    <ArtTableHeader title="用户列表" @refresh="load" />
  </template>
  ```

##### ArtTable

- **用途**：增强型表格封装，包含列渲染、分页、空态、全屏自适应。【F:src/components/core/tables/art-table/index.vue†L1-L160】
- **解剖**：`ElTable` + 动态列 + 自定义 slot + 分页容器。
- **变体**：`columns` 配置、`pagination`、`stripe/border/size`、`showTableHeader`。
- **状态**：loading、空数据、全屏、响应式布局。
- **使用示例**：

  ```vue
  <script setup lang="ts">
    import ArtTable from '@/components/core/tables/art-table/index.vue'
    const columns = [
      { prop: 'name', label: '姓名', useSlot: true },
      { prop: 'age', label: '年龄' }
    ]
    const data = [{ name: '张三', age: 26 }]
  </script>

  <template>
    <ArtTable :columns="columns" :data="data">
      <template #name="{ value }">{{ value }}</template>
    </ArtTable>
  </template>
  ```

#### 文本特效（Text Effect）

##### ArtCountTo

- **用途**：数字滚动动画，用于统计展示。【F:src/components/core/text-effect/art-count-to/index.vue†L1-L120】
- **解剖**：`useCountUp` 动画 + 前/后缀插槽。
- **使用示例**：
  ```vue
  <template>
    <ArtCountTo :target="520" prefix="￥" />
  </template>
  ```

##### ArtFestivalTextScroll

- **用途**：节日祝福滚动条，与设置 Store 联动显示。【F:src/components/core/text-effect/art-festival-text-scroll/index.vue†L1-L140】
- **解剖**：滚动容器 + 文本列表。
- **使用示例**：
  ```vue
  <template>
    <ArtFestivalTextScroll />
  </template>
  ```

##### ArtTextScroll

- **用途**：通用纵向文字滚动，支持渐变遮罩与手动控制。【F:src/components/core/text-effect/art-text-scroll/index.vue†L1-L160】
- **解剖**：滚动容器 + 列表项 + 控制按钮。
- **使用示例**：
  ```vue
  <template>
    <ArtTextScroll :list="['公告 1', '公告 2']" />
  </template>
  ```

#### 主题（Theme）

##### ThemeSvg

- **用途**：主题切换 SVG 展示，辅助用户预览主题色板。【F:src/components/core/theme/theme-svg/index.vue†L1-L100】
- **使用示例**：
  ```vue
  <template>
    <ThemeSvg :theme="currentTheme" />
  </template>
  ```

#### 视图（Views）辅助组件

##### ArtException

- **用途**：异常状态页（403/404/500）展示。【F:src/components/core/views/exception/ArtException.vue†L1-L120】
- **解剖**：插画 + 标题 + 描述 + 操作按钮。
- **使用示例**：
  ```vue
  <template>
    <ArtException type="404" @action="goHome" />
  </template>
  ```

##### AuthTopBar

- **用途**：登录页顶部栏，展示 Logo、语言切换等。【F:src/components/core/views/login/AuthTopBar.vue†L1-L120】
- **使用示例**：
  ```vue
  <template>
    <AuthTopBar />
  </template>
  ```

##### LoginLeftView

- **用途**：登录页左侧宣传卡片，含标题、描述、插画。【F:src/components/core/views/login/LoginLeftView.vue†L1-L160】
- **使用示例**：
  ```vue
  <template>
    <LoginLeftView />
  </template>
  ```

##### ArtResultPage

- **用途**：结果页组件（成功/错误/待审核）。【F:src/components/core/views/result/ArtResultPage.vue†L1-L160】
- **解剖**：状态插图 + 标题 + 描述 + 操作按钮 slot。
- **使用示例**：
  ```vue
  <template>
    <ArtResultPage status="success" title="提交成功" />
  </template>
  ```

#### 自定义（Custom）

##### CommentWidget & CommentItem

- **用途**：评论小部件，展示评论列表、作者信息与操作按钮。【F:src/components/custom/comment-widget/index.vue†L1-L160】【F:src/components/custom/comment-widget/widget/CommentItem.vue†L1-L140】
- **解剖**：统计头部 + 评论列表 + 发布区域；单项包含头像、内容、操作。
- **使用示例**：
  ```vue
  <template>
    <CommentWidget :comments="commentList" @submit="handleSubmit" />
  </template>
  ```

### 覆盖矩阵

| 组件 | 变体 | 状态 | 尺寸 | 示例 | 测试用例 |
| --- | --- | --- | --- | --- | --- |
| ArtBackToTop | 主题/尺寸 | default/hover/disabled | 固定 36-48px | ✅ | ❌ |
| ArtBarChart | 单/多序列/色板 | default/hover/empty | 容器自适应 | ✅ | ❌ |
| ArtBarChartCard | 数据/图表/按钮 | default/hover/loading | 固定高度/自定义 | ✅ | ❌ |
| ArtBasicBanner | CTA/装饰/背景 | default/hover/click | 全宽/可配置高度 | ✅ | ❌ |
| ArtBreadcrumb | 导航/抽屉/菜单 | default/hover/active | 响应式 | ✅ | ❌ |
| ArtButtonMore | 字段/布局/上传 | focus/disabled/loading | default/small/large | ✅ | ❌ |
| ArtButtonTable | 字段/布局/上传 | focus/disabled/loading | default/small/large | ✅ | ❌ |
| ArtCardBanner | CTA/装饰/背景 | default/hover/click | 全宽/可配置高度 | ✅ | ❌ |
| ArtChatWindow | 导航/抽屉/菜单 | default/hover/active | 响应式 | ✅ | ❌ |
| ArtCountTo | 滚动/动画 | default/hover | 自适应 | ✅ | ❌ |
| ArtCutterImg | 裁剪/播放模式 | default/loading/error | 自适应 | ✅ | ❌ |
| ArtDataListCard | 数据/图表/按钮 | default/hover/loading | 固定高度/自定义 | ✅ | ❌ |
| ArtDonutChartCard | 数据/图表/按钮 | default/hover/loading | 固定高度/自定义 | ✅ | ❌ |
| ArtDragVerify | 字段/布局/上传 | focus/disabled/loading | default/small/large | ✅ | ❌ |
| ArtDualBarCompareChart | 单/多序列/色板 | default/hover/empty | 容器自适应 | ✅ | ❌ |
| ArtExcelExport | 字段/布局/上传 | focus/disabled/loading | default/small/large | ✅ | ❌ |
| ArtExcelImport | 字段/布局/上传 | focus/disabled/loading | default/small/large | ✅ | ❌ |
| ArtFastEnter | 导航/抽屉/菜单 | default/hover/active | 响应式 | ✅ | ❌ |
| ArtFestivalTextScroll | 滚动/动画 | default/hover | 自适应 | ✅ | ❌ |
| ArtFireworksEffect | 导航/抽屉/菜单 | default/hover/active | 响应式 | ✅ | ❌ |
| ArtForm | 字段/布局/上传 | focus/disabled/loading | default/small/large | ✅ | ❌ |
| ArtGlobalComponent | 导航/抽屉/菜单 | default/hover/active | 响应式 | ✅ | ❌ |
| ArtGlobalSearch | 导航/抽屉/菜单 | default/hover/active | 响应式 | ✅ | ❌ |
| ArtHBarChart | 单/多序列/色板 | default/hover/empty | 容器自适应 | ✅ | ❌ |
| ArtHeaderBar | 导航/抽屉/菜单 | default/hover/active | 响应式 | ✅ | ❌ |
| ArtHorizontalMenu | 导航/抽屉/菜单 | default/hover/active | 响应式 | ✅ | ❌ |
| ArtIconSelector | 主题/尺寸 | default/hover/disabled | 固定 36-48px | ✅ | ❌ |
| ArtImageCard | 数据/图表/按钮 | default/hover/loading | 固定高度/自定义 | ✅ | ❌ |
| ArtKLineChart | 单/多序列/色板 | default/hover/empty | 容器自适应 | ✅ | ❌ |
| ArtLineChart | 单/多序列/色板 | default/hover/empty | 容器自适应 | ✅ | ❌ |
| ArtLineChartCard | 数据/图表/按钮 | default/hover/loading | 固定高度/自定义 | ✅ | ❌ |
| ArtLogo | 主题/尺寸 | default/hover/disabled | 固定 36-48px | ✅ | ❌ |
| ArtMapChart | 单/多序列/色板 | default/hover/empty | 容器自适应 | ✅ | ❌ |
| ArtMenuRight | 上下文/水印配置 | default/active | 自适应 | ✅ | ❌ |
| ArtMixedMenu | 导航/抽屉/菜单 | default/hover/active | 响应式 | ✅ | ❌ |
| ArtNotification | 导航/抽屉/菜单 | default/hover/active | 响应式 | ✅ | ❌ |
| ArtPageContent | 导航/抽屉/菜单 | default/hover/active | 响应式 | ✅ | ❌ |
| ArtProgressCard | 数据/图表/按钮 | default/hover/loading | 固定高度/自定义 | ✅ | ❌ |
| ArtRadarChart | 单/多序列/色板 | default/hover/empty | 容器自适应 | ✅ | ❌ |
| ArtRingChart | 单/多序列/色板 | default/hover/empty | 容器自适应 | ✅ | ❌ |
| ArtScatterChart | 单/多序列/色板 | default/hover/empty | 容器自适应 | ✅ | ❌ |
| ArtScreenLock | 导航/抽屉/菜单 | default/hover/active | 响应式 | ✅ | ❌ |
| ArtSearchBar | 字段/布局/上传 | focus/disabled/loading | default/small/large | ✅ | ❌ |
| ArtSettingsPanel | 导航/抽屉/菜单 | default/hover/active | 响应式 | ✅ | ❌ |
| ArtSidebarMenu | 导航/抽屉/菜单 | default/hover/active | 响应式 | ✅ | ❌ |
| ArtStatsCard | 数据/图表/按钮 | default/hover/loading | 固定高度/自定义 | ✅ | ❌ |
| ArtTable | 列配置/分页 | default/loading/empty | 密度切换 | ✅ | ❌ |
| ArtTableHeader | 列配置/分页 | default/loading/empty | 密度切换 | ✅ | ❌ |
| ArtTextScroll | 滚动/动画 | default/hover | 自适应 | ✅ | ❌ |
| ArtTimelineListCard | 数据/图表/按钮 | default/hover/loading | 固定高度/自定义 | ✅ | ❌ |
| ArtVideoPlayer | 裁剪/播放模式 | default/loading/error | 自适应 | ✅ | ❌ |
| ArtWangEditor | 字段/布局/上传 | focus/disabled/loading | default/small/large | ✅ | ❌ |
| ArtWatermark | 上下文/水印配置 | default/active | 自适应 | ✅ | ❌ |
| ArtWorkTab | 导航/抽屉/菜单 | default/hover/active | 响应式 | ✅ | ❌ |
| CommentWidget | 评论模式 | default/hover/loading | 自适应 | ✅ | ❌ |
| ThemeSvg | 主题预览 | default | SVG 自适应 | ✅ | ❌ |

## 机器读版骨架

```json
{
  "tokens": {
    "color": {
      "primary": "#5D87FF",
      "secondary": "#49BEFF",
      "success": "#13DEB9",
      "warning": "#FFAE1F",
      "error": "#FA896B",
      "info": "#6B7D9B",
      "danger": "#FF4D4F"
    },
    "background": {
      "light": "#FAFBFC",
      "lightSurface": "#FFFFFF",
      "dark": "#070707",
      "darkSurface": "#161618"
    },
    "typography": {
      "family": "Inter, 'Helvetica Neue', 'PingFang SC', 'Microsoft YaHei', Arial, sans-serif",
      "scale": [12, 14, 16, 18, 20, 24, 28, 32, 40, 48],
      "lineHeight": {
        "body": 1.5,
        "heading": 1.25
      }
    },
    "radius": {
      "sm": "calc(var(--custom-radius) / 3 + 2px)",
      "md": "calc(var(--custom-radius) / 2 + 4px)",
      "lg": "calc(var(--custom-radius) + 6px)"
    },
    "shadow": [
      "0 0.1rem 0.75rem 0.25rem rgba(0,0,0,0.05)",
      "0 0.1rem 1rem 0.25rem rgba(0,0,0,0.05)",
      "0 0.5rem 1.5rem 0.5rem rgba(0,0,0,0.075)",
      "0 1rem 2rem 1rem rgba(0,0,0,0.1)"
    ],
    "breakpoints": {
      "notebook": 1600,
      "ipadPro": 1180,
      "ipad": 800,
      "ipadVertical": 900,
      "phone": 500
    }
  },
  "themes": {
    "light": {
      "selector": ":root",
      "mode": "class",
      "className": "",
      "provider": "CSS variables"
    },
    "dark": {
      "selector": "html.dark",
      "mode": "class",
      "className": "dark",
      "provider": "Pinia setting + CSS variables"
    }
  },
  "components": {
    "base": ["ArtBackToTop", "ArtIconSelector", "ArtLogo"],
    "banners": ["ArtBasicBanner", "ArtCardBanner"],
    "cards": [
      "ArtStatsCard",
      "ArtLineChartCard",
      "ArtBarChartCard",
      "ArtDonutChartCard",
      "ArtProgressCard",
      "ArtDataListCard",
      "ArtImageCard",
      "ArtTimelineListCard"
    ],
    "charts": [
      "ArtBarChart",
      "ArtDualBarCompareChart",
      "ArtHBarChart",
      "ArtKLineChart",
      "ArtLineChart",
      "ArtMapChart",
      "ArtRadarChart",
      "ArtRingChart",
      "ArtScatterChart"
    ],
    "forms": [
      "ArtForm",
      "ArtButtonTable",
      "ArtButtonMore",
      "ArtDragVerify",
      "ArtExcelExport",
      "ArtExcelImport",
      "ArtSearchBar",
      "ArtWangEditor"
    ],
    "layouts": [
      "ArtBreadcrumb",
      "ArtChatWindow",
      "ArtFastEnter",
      "ArtFireworksEffect",
      "ArtGlobalComponent",
      "ArtGlobalSearch",
      "ArtHeaderBar",
      "ArtHorizontalMenu",
      "HorizontalSubmenu",
      "ArtMixedMenu",
      "ArtSidebarMenu",
      "SidebarSubmenu",
      "ArtNotification",
      "ArtPageContent",
      "ArtScreenLock",
      "ArtSettingsPanel",
      "SettingDrawer",
      "SettingHeader",
      "SectionTitle",
      "SettingItem",
      "BasicSettings",
      "BoxStyleSettings",
      "ColorSettings",
      "ContainerSettings",
      "MenuLayoutSettings",
      "MenuStyleSettings",
      "ThemeSettings",
      "ArtWorkTab"
    ],
    "media": ["ArtCutterImg", "ArtVideoPlayer"],
    "others": ["ArtMenuRight", "ArtWatermark"],
    "tables": ["ArtTableHeader", "ArtTable"],
    "textEffect": ["ArtCountTo", "ArtFestivalTextScroll", "ArtTextScroll"],
    "theme": ["ThemeSvg"],
    "views": ["ArtException", "AuthTopBar", "LoginLeftView", "ArtResultPage"],
    "custom": ["CommentWidget", "CommentItem"]
  },
  "coverage": {
    "exampleProvided": true,
    "testCoverage": false
  }
}
```

## 缺口与建议清单

- **语义色对比度不足（可迭代项）**：Secondary/Success/Warning/Error 在 Light 模式下未满足 WCAG AA，需调整色值或增加深色文字覆盖。【F:src/assets/styles/variables.scss†L3-L49】
- **表单与交互的键盘/ARIA 支持**：`ArtDragVerify`、`ArtButtonTable`、`ArtIconSelector` 等需补充键盘操作与 `aria-*` 属性，满足可访问性要求。【F:src/components/core/forms/art-drag-verify/index.vue†L1-L120】【F:src/components/core/forms/art-button-table/index.vue†L1-L80】【F:src/components/core/base/art-icon-selector/index.vue†L1-L127】
- **组件空态与 loading 一致性**：部分卡片与图表依赖外部处理空态，应统一提供 `isEmpty`/`loading` props 及 Skeleton 占位。
- **主题定制工具**：ColorSettings 仅支持单一主色，建议扩展品牌色板与对比度预览，减少视觉手动校验。【F:src/components/core/layouts/art-settings-panel/widget/ColorSettings.vue†L1-L180】
- **图表描述信息**：图表组件缺少数据摘要/辅助文本，需在数据可视化模块增加 `aria-label` 或 `summary` 属性以提升可读性。

## 命名与缩放对齐提案

- **Token 命名统一**：建议将 `--art-primary` 等 RGB 变量配套 `--art-primary-hex`，并在 theme JSON 中维护同名键，便于多端复用。
- **Spacing 系列**：落地 `--space-{xs,sm,md,lg,xl}` 并在组件中统一使用，避免硬编码 12/16/24px。
- **字号标识**：定义 `--font-size-{xs,sm,md,lg,xl}` 及 `--font-weight-{regular,medium,semibold}`，与设计稿标注一致。
- **组件导出名**：保持 `Art` 前缀 + PascalCase，如设置面板子组件 `BasicSettings`、`ThemeSettings` 等在文档和代码中对齐。
- **响应式尺寸**：菜单/页签等使用 rem 单位，与设置面板中的 `--custom-radius` 一致，避免 px 固定值导致缩放失衡。

## 迁移与落地步骤

1. **基础 Token 梳理（短期 1 周）**
   - 抽取颜色、字号、间距、圆角变量到统一 `tokens.scss` 或 JSON。
   - 在 Element Plus 自定义主题文件中引用 token，确保 Light/Dark 一致。
2. **组件无障碍增强（中期 2-3 周）**
   - 为表单、按钮、拖拽、图表容器补充 `aria` 标签与键盘操作。
   - 在文档站补充使用示例及交互说明，结合 Storybook/Playwright 进行视觉回归。
3. **主题与布局优化（中期 3-4 周）**
   - 扩展设置面板，加入品牌色矩阵、圆角/阴影预览、色弱模拟开关。
   - 落实 spacing/typography token 至主要页面模板。
4. **测试与持续集成（长期 1-2 月）**
   - 为核心组件建立单元/截图测试，覆盖表单提交、图表数据渲染、主题切换等场景。
   - 集成 lint + visual regression pipeline，在 PR 阶段校验 UI 规范落地。
