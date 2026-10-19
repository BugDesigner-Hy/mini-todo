# 威力待办 · Mi Todo

一个基于 Vue 3 + Vite 的**极简待办清单**，以中国传统色为设计语言：十套主题配色、流畅的勾选与列表动效、数据全部留在本地浏览器。可当作网页使用，也可打包成 Chrome 扩展的弹窗小工具。

> 掌控时间，掌控人生。

## ✨ 功能特性

- **十套中国传统色主题**：赭石、朱砂、青碧、黛蓝、藤黄、胭脂、石青、藕荷、松花、秋香，一键切换，全局渐变色实时联动。
- **完整待办流程**：回车或点击按钮即可添加（新任务自动置顶），支持勾选完成、双击/按钮编辑、单条删除、清除已完成。
- **三态筛选**：待办 / 已办 / 全部，带滑动指示器与实时数量统计。
- **时间记录**：保存创建时间与完成时间，点击条目可展开查看；也可在设置中开启「始终显示时间」。
- **个性签名**：自定义副标题（20 字以内），随主题一起持久化。
- **随机换色**：可开启「新增待办颜色随机变化」，每次添加自动切换主题。
- **空状态与动效**：空列表插画、列表增删的弹性动画、`IntersectionObserver` 滚动入场、勾选打钩旋转动画、头部流光效果。
- **纯本地存储**：基于 `localStorage` 持久化，无需登录、无后端、数据不上传。
- **移动端适配**：≤640px 时切换为全屏布局，操作按钮常驻显示。
- **可作浏览器扩展**：`popup.html` 为 400×600 的弹窗入口，配合 `public/manifest.json`（MV3）即可加载为 Chrome 扩展。

## 🛠 技术栈

| 分类 | 选型 |
| --- | --- |
| 框架 | Vue 3（Composition API + `<script setup>`） |
| 构建 | Vite 5 |
| 样式 | 原生 CSS：scoped style + CSS 变量主题系统 |
| 状态 | `ref` / `computed` / `watch`，无额外状态管理库 |
| 依赖 | 仅 `vue`，零 UI 组件库、零后端 |

## 🚀 快速开始

```bash
# 安装依赖
npm install

# 启动开发服务器（默认 http://localhost:5173）
npm run dev

# 构建产物
npm run build

# 本地预览构建结果
npm run preview
```

`npm run build` 会一次产出两套入口，均位于 `dist/`：

- `dist/index.html` —— 网页版（可直接部署到任意静态托管，含子路径）；
- `dist/popup.html` —— Chrome 扩展弹窗版（400×600）。

产物使用相对路径（`base: './'`），因此同一份 `dist/` 既能放到服务器子目录、也能直接当作扩展加载。

### 作为 Chrome 扩展加载

1. `npm run build`，产物输出到 `dist/`；
2. 打开 `chrome://extensions/`，开启「开发者模式」；
3. 点击「加载已解压的扩展程序」，选择 `dist/` 目录（`manifest.json` 已随 `public/` 一同复制）。

## 📁 目录结构

```
mi-todo/
├─ index.html              # 网页入口
├─ popup.html              # 浏览器扩展弹窗入口（400×600）
├─ vite.config.js          # Vite 配置（多入口：index + popup，base 为相对路径）
├─ LICENSE                 # MIT 许可证
├─ public/
│  ├─ manifest.json        # Chrome 扩展清单（MV3）
│  ├─ favicon.svg
│  ├─ icons.svg
│  └─ icon128.svg
└─ src/
   ├─ main.js
   ├─ App.vue              # 主题 / 筛选 / 设置 / 本地存储
   ├─ style.css            # 传统色变量、全局重置与基础样式
   ├─ assets/
   └─ components/
      ├─ TodoInput.vue     # 输入框与添加按钮
      ├─ TodoList.vue      # 列表容器、空状态、入场观察
      └─ TodoItem.vue      # 单条待办：勾选、编辑、删除、时间
```

## 📝 使用说明

- **添加**：输入内容后按 `Enter` 或点击右侧箭头按钮。
- **完成**：点击左侧圆形勾选框，完成后文字划线并记录完成时间。
- **编辑**：双击任务文字或点击铅笔图标；`Enter` 保存，`Esc` 取消。
- **删除**：点击垃圾桶图标；底部「清除已完成」批量清理。
- **设置**：右上角齿轮 → 修改签名、切换主题、开启随机换色/常显时间、一键清除全部记录（二次确认）。

## ⚠️ 说明与待办

- 数据仅存于当前浏览器的 `localStorage`，清理浏览器数据或更换设备后不会自动同步。
- 计划中：任务拖拽排序、截止日期与提醒、导出/导入 JSON、深色模式。

## 📄 许可

本项目基于 [MIT License](LICENSE) 开源，Copyright (c) 2026 BugDesigner-Hy。
