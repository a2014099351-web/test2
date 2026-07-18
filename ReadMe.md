# Mini Designer

轻量级可扩展界面设计器（简化版 Qt Designer）。实现 **XML ↔ UI** 双向转换：将 XML 可视化展示，并支持可视化编辑后重新序列化为 XML。

> 需求文档：[Mini Designer 项目要求文档](https://365.kdocs.cn/l/cpLqqRDNxpiv)

## 1. 项目概述

### 1.1 背景与定位

Qt Designer 采用「可视化编辑 + 属性编辑 + XML 序列化 + 插件扩展」的设计思想。本项目实现其简化版，完成：

- XML 文件内容可视化
- 可视化内容编辑并保存回 XML

### 1.2 核心思想（需求 1.3，均已落地）

| 类别 | 模式 / 原则 | 本仓库落点 |
|------|-------------|------------|
| 必选 | 工厂模式（控件创建） | `KWidgetFactory` 按类型名 `create()`；静态注册 + 插件注册 |
| 必选 | 观察者模式（属性面板实时刷新） | `KDocument` / `KSelectionModel` 信号 → 画布 / `KPropertyPanel` / `KObjectInspector` |
| 必选 | 命令模式（Undo/Redo） | `KICommand` + `KCommandStack`（`Ctrl+Z` / `Ctrl+Y`） |
| 必选 | MVC / MVVM（数据与视图分离） | Model：`KDocument` / `KSelectionModel`；View：Canvas / Palette / Inspector / Property；`KMainWindow` 协调 |
| 必选 | 序列化 / 反序列化（XML ↔ 对象） | `KXmlSerializer` + `KWidget::save` / `load` |
| 必选 | 开闭原则（扩展不改旧代码） | 新控件只增 `widgets/*.cpp` 或插件，解析核心无类型 if/else |
| 可选 | 插件模式（动态加载） | `KPluginLoader` 扫描 `plugins/`，`KIWidgetPlugin`（示例 CheckBox） |
| 可选 | 组合模式（控件树） | `KWidget` 父子树（`attachToParent` / `childWidgets`）；容器如 GroupBox；`KObjectInspector` 展示树 |

### 1.3 界面布局

采用 Qt Designer 风格的 `QMainWindow` + Dock 布局（Dock 可拖拽/浮动）：

```
+------------------------------------------------------------------+
|  菜单栏 · File / Edit / Form / View / Language / Help               |
+------------------------------------------------------------------+
|  工具栏 · 新建/打开/保存 · 撤销重做 · 对齐 · 网格/吸附/网格大小       |
+----------+------------------------------------+------------------+
|          |                                    | Object Inspector |
| Widget   |            Form                    |  (对象树)         |
| Box      |     +------------------------+     +------------------+
| (控件库) |     |     设计画布            |     | Property Editor  |
| Button   |     |                        |     |  (属性编辑)       |
| Label    |     |                        |     | x/y/w/h/text...  |
| Edit     |     +------------------------+     |                  |
| ...      |                                    |                  |
+----------+------------------------------------+------------------+
|  状态栏                                                              |
+------------------------------------------------------------------+
```

| 区域 | 实现类 | 说明 |
|------|--------|------|
| 中央 Form | `KDesignCanvas` | 设计画布，带 Form 标题外框 |
| 左侧 Widget Box | `KWidgetPalette` | 控件列表，点击/拖拽创建 |
| 右侧 Object Inspector | `KObjectInspector` | 文档内控件对象树，与选中联动 |
| 右侧 Property Editor | `KPropertyPanel` | 属性面板，修改后立即刷新画布 |

用户流程：创建控件 → 选中 → 修改属性 → 删除 → 保存/加载 XML，并支持后续扩展新控件类型。  
**不做** XML 到 C++ / HTML 的代码生成，仅做可视化编辑与 XML 读写。

## 2. 环境依赖、运行与支持平台

### 依赖

- CMake ≥ 3.10
- C++17 编译器（已用 VS 2019 验证）
- Qt5（Core / Widgets / Gui / Xml）

### 编译

在项目根目录执行兄弟目录脚本一键编译：

```bat
..\script\build_win.bat
```

### 运行

```bat
..\sample\build_x64_Debug\bin\Debug\minidesigner.exe
```
### 支持平台

- 支持平台：Windows、VMware（ubantu）

## 3. 功能对照（需求 → 实现）

### 3.1 基础功能

| 需求项 | 实现说明 |
|--------|----------|
| 3.1 控件创建 | Button / Label / Edit：控件列表点击放置或拖拽到画布 |
| 3.2 控件选中 | 高亮边框与四角控制点；空白处取消选中；支持移动、四角缩放 |
| 3.3 属性编辑 | 属性面板实时显示/编辑 id、x、y、width、height、text 及扩展属性 |
| 3.4 控件删除 | Delete 键 / 右键菜单删除 |
| 3.5 XML 保存 | 设计区全部控件序列化为 XML |
| 3.6 XML 加载 | 解析 XML 并恢复到设计区 |

### 3.2 进阶功能

| 需求项 | 实现说明 |
|--------|----------|
| 4.1 Undo / Redo | `Ctrl+Z` / `Ctrl+Y`：覆盖创建、删除、移动、缩放、改属性、对齐 |
| 4.2 多选 | `Ctrl+点击` 多选；批量移动、对齐、删除 |
| 4.3 网格吸附 | 网格线显示；拖动吸附；工具栏可改网格大小 |
| 4.4 复制粘贴 | `Ctrl+C` / `Ctrl+V`：新 id + 位置偏移（基于 `clone()`） |

### 3.3 扩展系统（需求 5.3）

| 需求项 | 实现 | 文件 |
|--------|------|------|
| Image — 图片控件 | ✓ | `widgets/imagewidget.*`（属性 `src`） |
| Slider — 滑动条 | ✓ | `widgets/sliderwidget.*`（`min` / `max` / `value`） |
| ProgressBar — 进度条 | ✓ | `widgets/progressbarwidget.*`（`min` / `max` / `value`） |
| 支持富文本的 Edit | ✓ RichEdit | `widgets/richeditwidget.*`（HTML，设计态用 `QTextDocument` 绘制） |

约束满足：新增控件无需改 `XmlSerializer` / `DesignCanvas` / `PropertyPanel` 核心逻辑。

### 3.4 插件机制

| 需求项 | 实现说明 |
|--------|----------|
| `plugins/` 动态加载 | 启动扫描并加载 DLL |
| `IWidgetPlugin` | `widgetType()` + `create()`，导出 `md_create_plugin` |
| 运行时注册 | `KWidgetFactory::instance().registerPlugin(...)` |
| 示例 | CheckBox：`plugin_examples/checkbox_plugin/` → `CheckBoxPlugin.dll` |

### 3.5 国际化（中 / 英）

使用 **Qt 国际化**（`tr()` + `QTranslator` + `.ts` / `.qm`）实现界面语言切换：

| 项 | 说明 |
|----|------|
| 源语言 | 英文（源码字符串走 `tr()`） |
| 中文 | `mui/zh_CN/minidesigner.ts`，构建时 `lrelease` 生成 `.qm` |
| 加载 | `KTranslationLoader`（`translationloader.*`）安装应用 / Qt 翻译 |
| 切换入口 | 菜单 **Language**（English / 简体中文 等） |
| 配置 | `QSettings` 键 `ui/language`；切换后需**重启应用**生效 |
| 运行时路径 | exe 旁 `mui/zh_CN/minidesigner.qm` |

### 快捷键

| 快捷键 | 功能 |
|--------|------|
| Ctrl+N / O / S | 新建 / 打开 / 保存 |
| Ctrl+Z / Ctrl+Y | 撤销 / 重做 |
| Ctrl+C / Ctrl+V | 复制 / 粘贴 |
| Delete | 删除选中 |
| Ctrl+Click | 多选切换 |

## 4. XML 格式

```xml
<ui>
  <Button id="btn1" x="100" y="100" width="120" height="40" text="OK" />
  <Label id="label1" x="300" y="100" width="200" height="40" text="Hello" />
  <Image id="img1" x="10" y="10" width="200" height="150" src="logo.png" />
  <Slider id="sld1" x="10" y="180" width="200" height="24" min="0" max="100" value="50" />
  <ProgressBar id="pb1" x="10" y="220" width="200" height="20" min="0" max="100" value="30" />
  <RichEdit id="re1" x="10" y="250" width="240" height="80" html="&lt;b&gt;Hello&lt;/b&gt; &lt;i&gt;rich&lt;/i&gt;" />
  <CheckBox id="chk1" x="10" y="340" width="140" height="28" text="CheckBox" checked="false" />
</ui>
```

**扩展性**：解析器按标签名查 `KWidgetFactory`，再 `create()` + `load()`，**无具体类型 if/else**。新增控件只需注册到工厂，无需修改解析器。

## 5. 设计说明（模块图 / 类图）

> 进阶篇交付要求：设计说明写入 ReadMe.md。

### 5.1 模块关系

```
KMainWindow
  ├── KWidgetPalette          (View · 左侧 Widget Box)
  ├── KDesignCanvas           (View · 中央 Form 画布)
  ├── KObjectInspector        (View · 右侧对象树)
  ├── KPropertyPanel          (View · 右侧属性编辑)
  ├── KDocument               (Model)
  ├── KSelectionModel         (Model)
  ├── KCommandStack           (Command)
  ├── KWidgetFactory          (Factory · 单例 · md_core)
  ├── KXmlSerializer          (Serializer)
  ├── KPluginLoader → KIWidgetPlugin → registerPlugin()
  └── KTranslationLoader      (Qt i18n · 中/英)
```

### 5.2 类关系（简化）

```
KWidget <<abstract>>
  ├── KButtonWidget / KLabelWidget / KEditWidget          (基础)
  ├── KImageWidget / KSliderWidget / KProgressBarWidget  (需求扩展)
  ├── KRichEditWidget                                    (富文本 Edit)
  └── KCheckBoxWidget (plugin DLL)

KWidgetFactory ──create──► KWidget
KIWidgetPlugin ──create──► KWidget
KDocument ──owns──► KWidget*
KCommandStack ──executes──► KICommand
  ├── KCreateWidgetCommand / KDeleteWidgetsCommand
  ├── KMoveWidgetsCommand / KResizeWidgetCommand
  ├── KSetPropertyCommand / KAlignWidgetsCommand
```

### 5.3 设计模式对照（与 §1.2 对应）

| 需求 | 模式 / 原则 | 落点 |
|------|-------------|------|
| 必选 | 工厂 | `KWidgetFactory` |
| 必选 | 观察者 | `KDocument` / `KSelectionModel` 信号 → Canvas / PropertyPanel / ObjectInspector |
| 必选 | 命令 | `KICommand` + `KCommandStack` |
| 必选 | MVC | Model · View · `KMainWindow` 协调 |
| 必选 | 序列化 | `KXmlSerializer` + `KWidget::save` / `load` |
| 必选 | 开闭原则 | 只增 `widgets/*.cpp` 或插件，不改序列化核心 |
| 可选 | 插件 + DIP | `KIWidgetPlugin` + `md_create_plugin`；`KPluginLoader` |
| 可选 | 组合 | `KWidget` 父子树 + 容器（如 GroupBox）+ Object Inspector |

## 6. 如何扩展一个新控件（只增文件）

对应需求「不允许修改任何现有代码，只允许新增 `.h` / `.cpp`」。  
现成样例：`imagewidget` / `sliderwidget` / `progressbarwidget`；富文本见 `richeditwidget`。

1. 在 `widgets/` 新增 `foowidget.h` / `foowidget.cpp`
2. 继承 `KWidget`，实现 `type/paint/save/load/clone/properties`
3. 在 `.cpp` 内用静态 `Registrar` 调用 `KWidgetFactory::instance().registerType(...)`
4. **重新 cmake/build**（`file(GLOB)` 会拾取新 `.cpp`）；无需改 `XmlSerializer` / `DesignCanvas` / `PropertyPanel`

## 7. 插件开发与加载

对应需求 `IWidgetPlugin` + `plugins/` 动态加载：

1. 实现 `KIWidgetPlugin`（`widgetType()` + `create()`）
2. 导出 C 符号：`extern "C" KIWidgetPlugin* md_create_plugin()`
3. 链接 `md_core`，与主程序同 Qt / 同编译器
4. 将 DLL 放到 `minidesigner.exe` 旁的 `plugins/`
5. 参考工程：`plugin_examples/checkbox_plugin/`（类 `KCheckBoxPlugin`，目标 `CheckBoxPlugin`）

## 8. 交付物清单

| 提交物 | 基础篇 | 进阶篇 | 本仓库 |
|--------|--------|--------|--------|
| 源码（可编译） | 必须 | 必须 | ✓ |
| README（运行 + 功能说明） | 必须 | 必须 | ✓ 本文档 |
| 可执行 exe（不上传 git） | 必须 | 必须 | `build/bin/Release/`，按课程要求单独提交 |
| 测试用例验证报告 | 必须 | 必须 | `testreport.md` |
| 设计说明（模块图 / 类图） | 可不做 | 必须（写入 ReadMe） | ✓ 第 5 节 |
| 自动化测试（可选） | 可不做 | 建议 | `test_md` 目标 |

交付目录约定：

```
themproject/minidesigner/
  ├── *.cpp / *.h
  ├── CMakeLists.txt
  ├── ReadMe.md
  └── testreport.md
```

## 9. 已知问题

- 多选时属性面板仅编辑主选中项的共有几何字段；专属属性请单选编辑
- 图片控件需填写可访问的本地路径；路径无效时显示占位文字
- Language 切换后需重启应用；`.qm` 位于 exe 旁 `mui/zh_CN/`
