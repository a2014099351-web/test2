# Mini Designer 测试用例验证报告

| 项 | 内容 |
|----|------|
| 项目 | minidesigner |
| 方法 | 手工 UI 验收 + GoogleTest 逻辑层回归 + OpenCppCoverage 行覆盖率 |
| 自动化目标 | `test_md`（GoogleTest） |
| 最近全量结果 | **23 / 23 PASSED**（Debug，2026-07-18） |
| 行覆盖率 | **85.7%**（1047 / 1222），OpenCppCoverage 0.9.9.0 |
| 构建 | VS 2019 / Qt5 / vcpkg |

记录格式：验证项 → 现象 → 处理 → 结果。

---

## 一、手工 UI 验收（进阶篇）

| 用例 ID | 场景 | 预期 | 结果 | 备注 |
|---------|------|------|------|------|
| TC-01 | 创建三种基础控件 | 画布出现 Button/Label/Edit | **Pass** | 点击放置与拖拽均可 |
| TC-02 | 修改属性面板 | 画布即时更新 | **Pass** | 观察者同步 |
| TC-03 | 保存 XML 后重新加载 | 控件与属性一致 | **Pass** | `<ui>` 往返；`XmlSerializer.RoundTripWithParent` |
| TC-04 | Delete / Backspace / 右键删除 | 确认后移除 | **Pass** | 含确认框 |
| TC-05 | Undo/Redo 创建与删除 | 状态可逆 | **Pass** | `Commands.CreateUndoRedo` |
| TC-06 | Undo/Redo 移动与改属性 | 状态可逆 | **Pass** | `Commands.MoveAndSetProperty` |
| TC-07 | Ctrl 多选后批量移动/删除 | 全部生效 | **Pass** | 对齐菜单可用 |
| TC-08 | 网格吸附 | 落点对齐网格 | **Pass** | 默认 10px，可配置 |
| TC-09 | 复制粘贴（含 GroupBox 子树） | 新 id、父子保留 | **Pass** | `Clipboard.DeepCopyGroupBoxSubtree` |
| TC-10 | 扩展 Image/Slider/ProgressBar/RichEdit | 可创建、可序列化 | **Pass** | `Widgets.PaintSaveCloneLoadAllTypes` |
| TC-11 | 插件目录加载 | 启动后列表出现 CheckBox | **Pass** | `plugins/CheckBoxPlugin.dll` |
| TC-12 | GroupBox 组合树 | 拖入/拖出、检查器树形、防环 | **Pass** | `Document.ParentChildTreeAndCycleGuard` |
| TC-13 | 语言切换 | Language 菜单中/英，重启生效 | **Pass** | `mui/zh_CN/minidesigner.qm` |

---

## 二、自动化测试摘要

构建与运行：

```bat
cmake -S . -B build -G "Visual Studio 16 2019" -A x64
cmake --build build --config Debug --target test_md
build\bin\Debug\test_md.exe
```

| 套件 | 覆盖点 | 用例数 |
|------|--------|--------|
| `IdGenerator` | id 递增、noteExisting、reset | 3 |
| `Factory` | 内建类型注册、create、未知类型 | 4 |
| `Document` | 增删查、父子树/防环、containerAt、级联展开、根节点 | 5 |
| `SelectionModel` | 单选/多选/toggle/clear | 1 |
| `Commands` | Create/Delete 级联/Move/Property/Paste 森林 UndoRedo | 4 |
| `Clipboard` | GroupBox 深拷贝子树、空选中 | 2 |
| `XmlSerializer` | parent 往返、未知标签跳过 | 2 |
| `Widgets` | 全类型 paint/save/clone/load、容器标记 | 2 |

**结论**：全量 **23** 项用例通过；逻辑层（工厂/文档树/命令/XML/剪贴板/控件）已自动化回归。

---

## 三、OpenCppCoverage 行覆盖率

工具：[OpenCppCoverage](https://github.com/OpenCppCoverage/OpenCppCoverage) 0.9.9.0  
被测程序：`build\bin\Debug\test_md.exe`（链接 `md_core.dll`）  
统计范围：`src\`（model/factory/command/serialize）+ `widgets\`（排除 autogen / moc / test）  

### 3.1 总体结果（2026-07-18）

| 指标 | 数值 |
|------|------|
| 可执行行 | 1222 |
| 已覆盖行 | 1047 |
| **行覆盖率** | **85.7%** |
| gtest | 23 / 23 PASSED |

说明：`test_md` **不含** Widgets UI（`kmainwindow` / `kdesigncanvas` / dock 等），覆盖率反映可单测逻辑层与内建控件实现，而非整个 GUI。

### 3.2 主要源文件明细（`.cpp`）

| 文件 | 覆盖行 | 总行 | 覆盖率 |
|------|--------|------|--------|
| `src/command/commands.cpp` | 156 | 230 | 67.8% |
| `src/model/document.cpp` | 133 | 144 | 92.4% |
| `src/model/widget.cpp` | 82 | 105 | 78.1% |
| `src/serialize/xmlserializer.cpp` | 59 | 77 | 76.6% |
| `src/model/clipboard.cpp` | 56 | 65 | 86.2% |
| `src/model/selectionmodel.cpp` | 39 | 49 | 79.6% |
| `src/factory/widgetfactory.cpp` | 26 | 33 | 78.8% |
| `src/command/commandstack.cpp` | 24 | 27 | 88.9% |
| `src/model/idgenerator.cpp` | 23 | 24 | 95.8% |
| `widgets/buttonwidget.cpp` | 44 | 44 | 100% |
| `widgets/labelwidget.cpp` | 40 | 40 | 100% |
| `widgets/editwidget.cpp` | 45 | 45 | 100% |
| `widgets/groupboxwidget.cpp` | 52 | 52 | 100% |
| `widgets/sliderwidget.cpp` | 61 | 61 | 100% |
| `widgets/progressbarwidget.cpp` | 62 | 62 | 100% |
| `widgets/richeditwidget.cpp` | 58 | 60 | 96.7% |
| `widgets/imagewidget.cpp` | 45 | 50 | 90.0% |

分层汇总：

| 层 | 覆盖 / 总行 | 覆盖率 |
|----|-------------|--------|
| `widgets` | 431 / 438 | **98.4%** |
| `src/model` | 346 / 404 | **85.6%** |
| `src/factory` | 27 / 34 | **79.4%** |
| `src/serialize` | 59 / 77 | **76.6%** |
| `src/command` | 184 / 268 | **68.7%** |
| 合计（含头文件可执行行） | 1047 / 1222 | **85.7%** |

### 3.3 未覆盖原因（摘要）

- **UI 层**（`src/app`、`src/view`、插件加载 UI）未链入 `test_md`，不在本次统计内。
- `commands.cpp` 约 32% 未覆盖：对齐命令全部分支、部分 resize/reparent 组合冷路径。
- `kxmlserializer` / `kwidget` 错误分支与少用属性路径未全部命中。
- `src/plugin`、`ktranslationloader` 属运行时加载/UI，未纳入本目标。

---

## 四、需求符合性（对照项目要求文档）

| 编号 | 需求 | 结论 | 验证依据 |
|------|------|------|----------|
| 1.3 必选 | 工厂 / 观察者 / 命令 / MVC / 序列化 / 开闭 | 满足 | Factory·Commands·Xml 单测；UI TC-02/05/06；见 ReadMe §1.2 / §5.3 |
| 1.3 可选 | 插件模式、组合模式（控件树） | 满足 | TC-11 插件；TC-12 GroupBox 树 + Object Inspector |
| 3.1～3.6 | 创建 / 选中 / 属性 / 删除 / XML 存取 | 满足 | TC-01～TC-04 |
| 4.1～4.4 | UndoRedo / 多选 / 网格 / 复制粘贴 | 满足 | TC-05～TC-09 |
| 5.1～5.2 | Widget 基类 + Button/Label/Edit | 满足 | Factory / Widgets 单测；TC-01 |
| 5.3 | Image / Slider / ProgressBar | 满足 | TC-10；`widgets/*` |
| 5.3 | 富文本 Edit | 满足 | RichEdit（TC-10 / `richeditwidget`） |
| 6.x | `plugins/` 动态加载 + IWidgetPlugin + DIP | 满足 | TC-11；CheckBox 插件 |
| 7.2 | XML 按工厂扩展、不改解析器 | 满足 | `XmlSerializer` 单测；扩展控件无需改解析器 |
| 八 | 测试报告 +（建议）自动化测试 | 满足 | 本报告；`test_md` 23/23；覆盖率 85.7% |

---
